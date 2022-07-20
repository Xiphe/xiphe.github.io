---
layout: post
title: Building a company internal ui component library in 2022
categories: systems
image: /assets/posts/2022-07-20/alfons-morales-YLSwjSy7stw-unsplash.jpeg
tags: component-library, web platform
author: Hannes Diercks
imageBy: '<span>Photo by <a href="https://unsplash.com/@alfonsmc10?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Alfons Morales</a> on <a href="https://unsplash.com/s/photos/component-library?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>'
description: Why and how I'd create a component library today
---

Hi I'm Hannes and a few days ago I found myself looking into a rabbit-whole
I've been in quite a few times.

{% capture content %}

#### Sidetrack: Why I'm writing this

About 7 years ago when component libraries were just about to become a thing
I was part of a small team that drastically increased development speed and unified
the UI/UX of [Jimdo](https://www.jimdo.com/) by creating a component library along
with a strategy on how to get adoption and support from all product teams.  
After that I was able to take what we've learned at Jimdo and apply it to 
new and existing design systems a bunch of other companies.

In the past 2 Years I've been more on the product side of things but now
[my employer musicube](https://www.musicu.be/en/) was acquired by [songtradr](https://www.songtradr.com/)
and the design and development teams here are just on the verge of creating
a component library.

That's the reason I thought it might be cool to take everything I've learned
about these systems, mix it with my current assessment of the web platform,
sprinkle a little flat-hierarchy ideology in there and write down how I'd create
a component library today, mid 2022.

Ready? _Ok let's go!_ 🚀

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

&nbsp;

## 1. Why?

> Why should I build a component library in 2022?

Communication between designers and developers is essential when building
software products in a bigger company. And a component library is on of the
most powerful tools they can utilize to make communication as easy as possible.

### 💡 Document Decisions

**Each component documents all design and development decisions leading to it.**

The creation process may vary, but in the end there is one component
that has been approved by both development and design.
So users can confidently use it, knowing that they use the latest state and
when needed also can find out which decisions lead to it.

When someone is not entirely happy with a component they can use the current
state of the library as a reference point and propose improvements
based on that.

### 🥳 Unify UI/UX of your product

**Users of your product will subconsciously learn its UX patterns and apply it to other parts of the product. When these patterns match, the user is effective and happy.**

A widely adopted component library is the best way to get there.

If you fail to deliver a coherent experience, for example when the colors
of your destructive buttons vary a lot across the system, navigating requires
more cognitive affords from the user which can be exhausting and lead to deep sadness 😥.

### 🏎 Speed up development

**Users of the library should be faster when using it then they'd be without it.**

_..."Fast" is a pretty diffuse term in product development... what I mean is_:

1. Getting a decent state of the companies UI/UX in any prototyping environment
   should be a no-brainer.
2. Round-Trips between design and development should be reduced since both
   speak the same language and can refer to the library
3. Applying new UI/UX developments/fixes to old code should be **simple**.

&nbsp;

## How?

> How should I approach building a library to be effective?

### 🐙 Treat it as if it were open source

I saw the greatest success when people maintain, contribute and use a company 
internal library as if were an open source project
_(only that this one is "just" open to the whole company instead of the whole world)_.

With that in mind it comes natural to document everything properly, to be
inclusive to everyone, to work with issues and pull requests, to be mindful with
API changes, etc...

{% capture content %}

🧑‍🏫 Don't worry in case you never maintained or contributed to an open source project, that's fine.  
I cover the most important parts [later in this article](#-write-documentation-for-users). Just behave sensible,
remember [why the project exists](#1-why) and be respectful to users and co-maintainers.

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>


### 🚓 Don't get too fixated with nomenclature

The most famous methodology for design systems is probably [Atomic Design](https://bradfrost.com/blog/post/atomic-web-design/). And though I think there is a great amount of guidance
you can get out of that remember that a component library is a living
system.

Everything can be changed, especially in the beginning.  
<small><em>Later on be careful when renaming API in order to keep the amount of breaking changes low<a href="-thoughtful-release-management"><sup>*</sup></a></em></small>

Dare giving components a suboptimal name or put them in the _wrong_ category.
The most important thing is that everybody (design and development) is on the
same page and uses a common language.

### ❄️ Don't require everything to be in the library

You want to allow projects to have their own components and maybe even
variants of elements in the library. In a living system, evolution must also occur.
And a process that is too strict hinders product development instead of helping it.

> I still like to call these one-of components "snowflakes", a label coined by the team behind [XINGs](https://www.xing.com/en) ui system.

### 🔌 Create different integration levels

In order to support rapid prototyping and to prevent locking your company in
a narrow tech choice I strongly recommend maintaining different integration
points with the library.

#### Level 1: CSS Preprocessor

When you have a common css preprocessor ([sass](https://sass-lang.com/) for example)
that is widely adopted in your development this makes for a great low level
integration.

Provide Colors, Spacings, Typography settings and maybe also mixins on this level

#### Level 2: CSS + HTML

**🤔 Think plain [Bootstrap](https://getbootstrap.com/).**

Level 2 uses the values provided by the CSS Preprocessor and
provides static components that implement the **look** of your UI design.

The web platform uses CSS and HTML for UI, there's no way around it and
therefore integrating here is tremendously valuable.

Create a stylesheet that unlocks your components in any page by simply adding a
`<link />`.  
Even when your development writes no single line of plain/html and uses
ReactJS only, Marketing & Support teams use low-/no-code solutions, Growth teams want to
try out some crazy new stuff, Companies get acquired that had build their
awesome product with Rails and now should adopt your look fast...  

All of these use HTML/CSS in the end and most likely support serving a css file.

#### Level 3: Framework

This level abstracts specific html structure and detailed class names and
should get really close to the language that is actually spoken between
design and development.  
When someone says "Can you please create a centered modal with a destructive button saying 'Delete!'?",
on the framework integration level it should boil down to something like

```tsx
<Modal align="modal-center">
  <Button variant="button-destructive">Delete!</Button>
</Modal>
```

It uses the static components provided by the HTML + CSS level and arguments
them with **behaviour**, makes them accessible and easy to work with in your
most common product development environments.

{% capture content %}

#### Sidetrack: ⚠️ Component State

Make sure that every state is accessible and changeable from outside the
component. Most state is best managed in the products using the library rather
then in the components even when it requires a little more boilerplate on the
consumer side.  

> routing is also a state

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

### 📜 Write documentation for users

There is this [great article from Brad Frost about the difference between storefront and workshop documentation](https://bradfrost.com/blog/post/the-workshop-and-the-storefront/).

Let's comparing the most popular UI library tools 

- [docz](https://www.docz.site/)
  Helps you build a **storefront**: A place where consumers can learn about the
  library concepts and browse components without needing to understand all underlying details.
- [Storybook](https://storybook.js.org/)
  This is a **workshop**: It supports developers at building great components.

<small>ℹ️ <a href="https://figma.com/">Figma</a> is also a workshop</small>

So with that in mind, and the fact that the library is of little worth when
nobody uses it I advise to have an up to date storefront. 
_(Doesn't mean you can't also have a workshop)_.

### 👜 Minimize dependencies

I personally feel like managing external dependencies is one of the most painful 
and challenging tasks in modern web development.  
Therefore it's of high worth when your library is sleek, doesn't disrupt the dependency choice
of consumers and has little to ask when dependants want to bring their packages up to date.

There is no golden rule here but I'd recommend using as little dependencies
that might interfere with consuming projects as possible.

{% capture content %}

#### Sidetrack: 🧰 external css utilities

Users of your library will inevitably need custom styling. 
They should be free to choose if they use inline styes, css in js or 
[tailwind.css](https://tailwindcss.com/) for that. 

The libraries job is to provide component styles, not to serve all css needs.

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

### 🛳 Thoughtful release management

Ideally whenever you release an update to the library, consumers can simply
install the latest version and ship it to users. 

In order to achieve that be aware of the APIs consumers rely on.  
When you went with [the integration points mentioned earlier](#-create-different-integration-levels) your APIs
are now:

- variable names of css preprocessor integration
- class names and element structures of CSS + HTML integration
- exports and signatures (props) of framework integration

Whenever one of these is modified or the look / behavior is changed significantly
you should consider it a breaking change – meaning users of the library
must be informed and provided with straight forward instructions
on how to migrate.

### 🌳 Consider Tree Shaking

Your library will most likely include more components and helpers
than any one project will need to use. So there should be tooling at hand
to either only pick the parts needed for a project or to remove code that is not required.

{% capture content %}

#### Sidetrack: 🧹 Supporting PurgeCSS

[PurgeCSS](https://purgecss.com/) needs to statically find the css classes 
you're using in your sourcefiles in order to know what css to keep and what to throw away 🚮

so its usually more effective to use class names in api's instead of magically
creating them, at least for less common variants.

```tsx
// Good:
<Button color="button-blue" />
// => <button class="button button-blue">

// Problematic:
<Button blue />
// => <button class="button button-blue">
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

&nbsp;

## What?

> What tech stack should I go with today?

Here's my personal list of solutions that stood the test of time,
have huge adoption rates and support the requirements I've set out above.

- A dedicated [Slack](https://slack.com/) channel where everyone is invited to ask questions and connect
- A [GitHub](https://github.com/new) repo to organize and communicate in a structured way
- [Scss](https://sass-lang.com/) to create a [variable file similar to bootstraps](https://github.com/twbs/bootstrap/blob/main/scss/_variables.scss) (but strapped down a lot) and one or more stylesheets for the component styles
- [docz](https://www.docz.site/) to create user facing documentation with example html (and later React) for all components
- [TypeScript](https://www.typescriptlang.org/) to implement typed components for [your framework here]
- [esbuild](https://esbuild.github.io/) or [Vite](https://vitejs.dev/) to create tree shakable bundles of the javascript code
- [semantic release](https://github.com/semantic-release/semantic-release) to create
  a nice changelog, incorporate [Semver](https://semver.org/) and automatically release a package to either npm or some private registry.
- A PullRequest based workflow that creates a preview of the updated documentation
  as well as a preview release of the package for each change.

There are lots of other puzzle pieces required but I think these are the most important ones 
to get right in the beginning.

&nbsp;

&nbsp;

Alright, now let's go build a library! 📚

&nbsp;
