---
layout: post
title: How to programmatically test visual design of components
categories: testing
image: /assets/posts/2020-09-28/vitor-pinto-M_fFGxg8Zhk-unsplash.jpg
tags: testing, components, design, UI, test-real-styles
author: Hannes Diercks
imageBy: '<span>Photo by <a href="https://unsplash.com/@vdapinto?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Vitor Pinto</a> on <a href="https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>'
description: Learn to write solid tests for the appearance of components
---


## The Problem

When writing component based applications or libraries we often create components
where part of their API only changes the appearance.


<div id="ex1" class="sidetrack">
  <style>
  .ex1-hidden {
    display: none;
  }
  #ex1 label {
    user-select: none;
  }
  #ex1-button {
    font-weight: bold;
    pointer-events: none;
    border: none;
    border-radius: 3px;
    padding: 0.5em 1em;
    background: lightgray;
  }
  #ex1-button.ex1-primary {
    background: fuchsia;
    color: white;
  }
  </style>

  <div id="ex1-code" class="language-tsx highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">&lt;</span><span class="nc">Button</span> <span class="nt">primary</span><span class="p">&gt;</span>Hello World<span class="p">&lt;/</span><span class="nc">Button</span><span class="p">&gt;</span></code></pre></div></div>

  <template id="ex1-default-code"><div class="highlight"><pre class="highlight"><code><span class="p">&lt;</span><span class="nc">Button</span><span class="p">&gt;</span>Hello World<span class="p">&lt;/</span><span class="nc">Button</span><span class="p">&gt;</span></code></pre></div></template>

  <button id="ex1-button" class="ex1-primary">Hello World</button>
  &nbsp; | <label>
    <input type="checkbox" id="ex1-toggle" checked>
    toggle primary prop
  </label>

  <script type="text/javascript">
    const toggle = document.getElementById('ex1-toggle');
    const codeContainer = document.getElementById('ex1-code');
    const primaryCode = codeContainer.innerHTML;
    const defaultCode = document.getElementById('ex1-default-code').innerHTML;
    const button = document.getElementById('ex1-button');
    let isPrimary = toggle.checked;
    function update() {
      requestAnimationFrame(() => {
        codeContainer.innerHTML = isPrimary ? primaryCode : defaultCode;
        button.classList[isPrimary ? 'add' : 'remove']('ex1-primary');
      });
    };
    toggle.addEventListener('change', function(ev) {
      isPrimary = ev.target.checked;
      update();
    });
  </script>
</div>

How should we best test this?
Lets find out!

&nbsp;

## First Identify the interface(s)

Before writing any test we should be clear about the interfaces of an abstraction and their consumers.  
_This is a big topic for another post... jumping to conclusions:_

- The **abstraction** is the `Button` component. 
- It's **interfaces** are:
  1. The programming Interface.  
     _(In react: props + context + return value)_
  2. It's UI + UX
- Their **consumers** are:
  1. Fellow programmers using the component in their app
  2. The users of this app

> Most abstractions in frontend applications have two kinds of consumers: Engineers & App-Users

&nbsp;

## Then find the cause + effect you want to test

For our `Button`:  
 - When the `primary` prop is set to true by an engineer [_via complex hooks and stuff_] **(cause)**  
 - Then the background of the app should appear `fuchsia` to the app-user **(effect)**

> `fuchsia` is the main brand color!

&nbsp;

## Now decide on the best tooling to write an optimal programmatic test with

This is where things get tricky! Most tools I see being used today don't support writing optimal tests. So let's compare them!

{% capture content %}

#### Sidetrack: Characteristics of an optimal test

1. The test **acts** on the **cause** and **asserts** on the **effect** (see [AAA](https://duckduckgo.com/?q=arrange+act+assert) & [previous section](#then-find-the-cause--effect-you-want-to-test))
2. The test does not cover implementation details (see: [Testing Implementation Details](https://kentcdodds.com/blog/testing-implementation-details))
3. It **only** breaks when the covered effect changes.

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

&nbsp;

So what are the options?

### Option 0: Just don't test

I think it's fully valid to just not test visual design programmatically. Manual QA will 
most likely be enough.

Still there are cases where automated tests are really valuable. Most primarily
in **design systems**/**component libraries** where the mission is to provide
abstractions that are easy to use for engineers and look as intended to the user.

&nbsp;

As a rule of thumb ask yourself: _"Am I (or my team) the only one using this component?"_

**Yes**: Only Add tests if you feel the component is unstable.  
**No**: Add tests unless the component is dead simple.

&nbsp;

### Option 1: Snapshot Testing
   
A simplified snapshot test would look like this:

{% capture content %}

```tsx
expect(renderToHTML(<Button primary>Hello</Button>))
  .toMatchInlineSnapshot(`
    "<button class="primary">Hello<button>"
  `);
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>


From my experience tests like this have little to no value for appearance testing
because they **assert** on an **implementation detail** (namely the class name).

- 👎 the `primary` class could set `background: olive;` but the test would be green.
- 👎 refactoring to `<button style="background: fuchsia;">` would have the same **effect** but the test would fail.
- 👎 refactoring the class name or adding others would make the test fail.

> This does not mean snapshot tests are bad. I recommend reading [Effective Snapshot Testing](https://kentcdodds.com/blog/effective-snapshot-testing).

&nbsp;

### Option 2: Visual Regression Testing

A simplified visual regression test would create a screenshot of the button rendered in a browser
and compare it to a stored screenshot. If any pixels have changed, the test would fail.

<figure><img src="/assets/posts/2020-09-28/screenshot-difference.png">
<figcaption class="center">Image Source: <a href="https://qanish.wordpress.com/2018/12/12/visual-regression-testing-tools/">Visual Regression Testing Tools</a></figcaption></figure>

&nbsp;

While this perfectly covers the **effect** without depending on implementation
details, it fails on focussing on the single **effect** we want to test.
Thereby [violating point 3 of an optimal test](#sidetrack-characteristics-of-an-optimal-test).

- 👎 **Any** change to the looks of the button will now cause the test to fail.
  All buttons now have a border? All button tests fail!
- 👎 Even changing the button **label** will fail the test.
- 👎 Visual regression tests also fully cover **the browsers rendering/painting process.**

  **Assertions** have the highest value when they cover **effects** as short as possible after they
  leave your **abstraction**. If we let the effects flow though a bunch of other systems, chances are they get blurry.  

  For example we will not test that the primary background color is actually _perceived_ as "fuchsia" by the **user**. (_That should have been done beforehand by user- and a11y-tests_).

  Why should we test that the browser actually renders a DOM-node with a `backgroundColor` of `fuchsia` accordingly? That is something that the maintainers of the browser should test, not us.

> This also does not mean visual regression tests are bad in general. I recommend reading [Visual Regression Testing](https://medium.com/loftbr/visual-regression-testing-eb74050f3366).

&nbsp;

### Option 3: Style testing

The closes point where the **effect** leaves our domain (the point after which we can not accidentally introduce bugs anymore) is the [computed styles](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle) of the DOM-Node in the browser. 

Let's test that!

{% capture content %}

```tsx
import { getRealStyles, toCss } from 'test-real-styles';

describe('primary prop', () => {
  it('makes the background fuchsia', async () => {
    const button = renderToDomNode(<Button primary>Hello</Button>);

    const styles = await getRealStyles({
      css: readFileSync('button.css'), 
      doc: button,
      getStyles: ['backgroundColor'],
    });
    expect(toCss(styles)).toMatchInlineSnapshot(`
      "background-color: fuchsia;"
    `);
  });
});
```

{% endcapture %}


<div class="sidetrack">
{{ content | markdownify }}
</div>


- 👍 Acts on the **cause** (primary prop) and asserts on the **effect** (background-color)
- 👍 Doesn't cover implementation details on how the style is applied to the element 
- 👎 Still covers implementation details when it comes to element composition.
  (refactoring the button to be wrapped in a `span` will fail the test)
- 👍 Adding other styles or changing the label will not break the test.

I shamelessly recommend using [`test-real-styles`](https://github.com/Xiphe/test-real-styles)
because it uses real browsers and supports using your full page css.  

There is also [`toHaveStyle` of `jest-dom`](https://github.com/testing-library/jest-dom#tohavestyle) but it did not work for me because it keeps the element in [jsdom](https://github.com/jsdom/jsdom) which has [lots of issues emulating CSSOM](https://github.com/jsdom/jsdom/labels/css)

Most css-in-js solutions provide something like [`jest-styled-components`](https://github.com/styled-components/jest-styled-components) which is pretty neat, too. 
(_A downside is that these will also not evaluate the styles in a real browser and therefore will not capture bugs caused by the cascade._)

&nbsp;

## Finally write the actual test

Given we decided on a **style-testing** solution, I want to share a few thoughts 
that might make the tests even more valuable.

Writing `.button { background: fuchsia; }` in the implementation and
`expect(getStyle('background'))` `.toBe('fuchsia')` in the test feels tedious.

Though in bigger components this will be more meaningful and stabilizing, there
is a potential gem to be created:

#### A machine-readable spec of low level design patterns

Sit down with your designers and not only talk about how the button should
look like but also which underlying **patterns** lead to this look.

This might surface things like:

"We want all intractable elements to have a `border-radius` of `5px`"  
"The main solid interaction element should use our brand-color as background" 

Which can be translated to:

{% capture content %}

```js
// designIntentions.js
import { brandColor } from './atoms';

export const interactable = {
  borderRadius: '5px';
};
export const primaryInteractable = {
  ...interactable,
  backgroundColor: brandColor
}
```

Which then can be used in style-tests like:

```jsx
import { getRealStyles } from 'test-real-styles';
import { primaryInteractable } from 'designIntentions.js';

describe('primary prop', () => {
  it('applies primary interactable styles', async () => {
    const buttonElement = renderToHtmlElement(<Button primary>Hello</Button>);

    const styles = await getRealStyles({
      css: readFileSync('button.css'), 
      doc: buttonElement,
      getStyles: Object.keys(primaryIntractable),
    });
    expect(styles).toEqual(primaryIntractable);
  });
});
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

And I think that's **beautiful**.

&nbsp;

### Do you think your project could benefit from style testing?  

I'm available to help you set everything up technically, help dev and design
teams to get the most value of this and also [write tests for you](/lmttfy/).

[get in touch ❣️](#footer-contact)


&nbsp;


### Was this post valuable for you?

Cool! Here is how you can give back if you want to: (_only pick a few_ 😉)

 - [Fix typos or improve my phrasing 💖](https://github.com/Xiphe/xiphe.github.io/edit/master/_posts/2020-09-28-component-design-testing.md)
 - Share the article to people that might also like it
 - [Follow me on Twitter](https://twitter.com/XipheHH)
 - [Sponsor me on GitHub](https://github.com/sponsors/Xiphe)
 - Hire me to help your frontend dev & design or recommend me
 - Spread love, peace and sanity
 - Act to save our planet
 - Be inclusive to everyone but the intolerant (because [the paradox of tolerance](https://en.wikipedia.org/wiki/Paradox_of_tolerance) is a thing).

Thanks for reading! Be safe ✌️
