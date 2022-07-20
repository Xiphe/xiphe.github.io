---
layout: post
title: Building company internal ui component library in 2022
categories: systems
image: /assets/posts/2022-07-20/alfons-morales-YLSwjSy7stw-unsplash.jpeg
tags: component-library, web platform
author: Hannes Diercks
imageBy: '<span>Photo by <a href="https://unsplash.com/@alfonsmc10?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Alfons Morales</a> on <a href="https://unsplash.com/s/photos/component-library?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>'
description: Why and how I'd create a component library in 2022
---

Hi I'm Hannes and a few days ago I found myself looking into a rabbit-whole
I've been in quite a few times.

{% capture content %}

#### Sidetrack: Why I'm writing this

About 7 years ago when component libraries were just about to become a thing
I was part of a small team that drastically increased development speed and unified
the UI/UX of [Jimdo](https://www.jimdo.com/) by creating a UI library that
got adopted by all product teams. After that I was able to take what we've
learned there and apply it to a bunch of other companies.

In the past 2 Years I've been more on the product side of things but now
[my employer musicube](https://www.musicu.be/en/) was acquired by [songtradr](https://www.songtradr.com/)
and the design and development here is just on the verge of creating
a component library.

That's the reason I thought it might be cool to take everything I've learned
about these systems, mix it with my current assessment of the web platform,
sprinkle a little flat-hierarchy ideology in there and write down how I'd create
a component library today, mid 2022.

Ready? Ok let's go 🚀

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

## 1. Why?

> Why should I build a component library in 2022?

Communication between designers and developers is essential when building
software products in a bigger company. And a component library is on of the
most powerful tools they can utilize to make communication as easy as possible.

### 💡 Document Decisions

**Each component is a documentation of all design and development decisions leading to it.**

The creation process may vary, but in the end there is one component
that has been approved by both development and design.
Thereby creating confidence in users of the library.

When someone is not entirely happy with a component they can use the current
state of the library as a reference point and propose improvements
based on that.

### 🥳 Unify UI/UX of the product

**Users of your product will subconsciously learn its UX patterns and apply it to other parts of the product. When these patterns match, the user is effective and happy.**

A widely adopted component library is the best way to get there.

Think destructive buttons. They should have the same alarming color
everywhere in the product. If their color would differ a lot this
would require more cognitive affords from the user.

### 🏎 Speed up development

**Users of the library should be faster when using it then they'd be without it.**

"Fast" is a pretty diffuse term in product development... what I mean is:

1. Getting a decent state of the companies UI/UX in any prototyping environment
   should be faster using the library then writing it by hand
2. Round-Trips between design and development should be reduced since both
   speak the same language and can refer to the library
3. Applying new UI/UX developments/fixes to old code should be a simple task
   that can be done by someone who does not know the old code at all.

## How?

> How should I approach building a library to be effective?

### 🐙 Treat it as if it were open source

I saw the greatest success when people work with a company internal library as if
they were creating, contributing to, or using an open source project
_(only that this one is "just" open to the whole company instead of the whole world)_.

With that in mind it comes natural to document everything properly, to be
inclusive to everyone, to work with issues and pull requests, to be mindful with
API changes... I'll go into a few of these in a more detail later.

{% capture content %}

Don't worry in case you never maintained or contributed to an open source project, that's fine.  
I cover the most important parts later in this article. Just behave sensible,
remember [why the project exists](#1-why) and be respectful to users and co-maintainers.

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>


### 👮 Don't get too fixated with nomenclature

The most famous methodology for design systems is probably [Atomic Design](https://bradfrost.com/blog/post/atomic-web-design/). And though I think there is a great amount of guidance
you can get out of that remember that a component library is a living
system.

Everything can be changed, especially in the beginning.  
<small><em>Later on be careful when renaming API in order to keep the amount of breaking changes low<a href="-thoughtful-release-management"><sup>*</sup></em></small>

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

All of these use HTML/CSS in the end and most likely support serving css file.

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

#### Sidetrack: Component State

⚠️ Make sure that every state is accessible and changeable from outside the
component. Most state is best managed in the products using the library rather
then in the components even when it requires a little more boilerplate on the
consumer side.

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


{% capture content %}

ℹ️ [Figma](https://figma.com/) is also a workshop

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>


So with that in mind, and the fact that the library is of little worth when
nobody uses it I advise to have an up to date storefront. 
_(Doesn't mean you can't also have a workshop)_.

### 👜 Minimize dependencies

From my personal experience managing external dependencies of a modern web project
is one of the most painful an challenging tasks. It's of high worth when
your library is sleek, disrupts the dependency choice of consumers as little
as possible and has little to ask when dependants want to bring their packages
up to date.

There is no golden rule here but I'd recommend using as little dependencies
that might interfere with consuming projects as possible.

### 🚀 Thoughtful release management

Ideally whenever you release an update to the library, users can simply
install the latest version and ship it. 🚀

In order to achieve that be aware of the APIs users rely on.
When you've chosen to go with the integration points mentioned earlier your APIs
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

For the latter approach it's good to know that [PurgeCSS](https://purgecss.com/) needs
to read complete css classes in your sourcefiles to know what to keep and what not

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

## What?

Given your company mostly codes in ReactJS here's what id use:

- A dedicated slack channel where everyone is invited to ask questions and connect
- A [github](https://github.com/new) _(gitlab, ...)_ repo to organize and communicate
- [scss](https://sass-lang.com/) to create a [variable file similar to bootstraps](https://github.com/twbs/bootstrap/blob/main/scss/_variables.scss) (but strapped down a lot)
- also scss to create one or more stylesheets that hold all the **looks**
- [docz](https://www.docz.site/) to create user facing documentation with example html (and later React) for all components
- [TypeScript](https://www.typescriptlang.org/) to implement React components for the **behaviour**
- [esbuild](https://esbuild.github.io/) to create tree shakable bundles of the javascript code
- [semantic release](https://github.com/semantic-release/semantic-release) to create
  a nice changelog, incorporate [Semver](https://semver.org/) and automatically release a package to either npm or some private registry.
- A PullRequest based workflow that creates a preview of the updated documentation
  as well as a preview release of the package for each change.
