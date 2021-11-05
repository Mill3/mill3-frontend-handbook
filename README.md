- [Mill3 Studio - The frontend handbook 📙 🐉 🧙‍♂️](#mill3-studio---the-frontend-handbook---️)
  - [Modules structre convention](#modules-structre-convention)
  - [Classname naming convention (BEM)](#classname-naming-convention-bem)
    - [SCSS structure example :](#scss-structure-example-)
  - [@mill3/system-ui-sass](#mill3system-ui-sass)
  - [JS UI/Components/Modules/Utilities structure](#js-uicomponentsmodulesutilities-structure)
    - [Anatomy of a class](#anatomy-of-a-class)
  - [BarbaWebpackChunks plugin - DOM-Controller](#barbawebpackchunks-plugin---dom-controller)
  - [Event Emitters](#event-emitters)
  - [Mill3 WP Boilerplate](#mill3-wp-boilerplate)
  - [ARIA good practices](#aria-good-practices)
    - [Hiding element from screen readers](#hiding-element-from-screen-readers)
    - [ARIA-label](#aria-label)
    - [Using button -vs- link](#using-button--vs--link)
  - [Locomotive-scroll recipes](#locomotive-scroll-recipes)
    - [Fixed panel reveal](#fixed-panel-reveal)


# Mill3 Studio - The frontend handbook 📙 🐉 🧙‍♂️

Documenting our good practices and toolsets.

## Modules structre convention

We split our modules using the following convention

* ```UI:``` anything that is present at all time on the UI: Site-header, Site-footer, Site-Nav, Site-Transition, Site-Loader. Preferably, prefix the element with ```Site``` before it.
* ```Modules:``` anything that is model related (post type) or repeated usage in page. Invoked on demand and in context : Recipe, Article, TextTicker, 3rd party elements parsing lib, Player, etc.
* ```Components:``` reusable components used inside Modules and UI (Accordions, Toogling system, Proxy binding)
* ```Utils:``` any utility modules with a specific role used in conjonction with UI, Modules or components (Math utils, events binding, DOM manipulation, transformation classnames for generic HTML elements, etc)

The same logic should be applied to JS or CSS modules. **Very important :** make sure your JS module file structure is **always** matching its related CSS file :

**JS module :**

```
src/js/modules/site-nav/index.js
```

**CSS module**

```
src/scss/modules/_Site-Nav.scss
```

## Classname naming convention (BEM)

We use a loose interpretation the [BEM](https://en.bem.info/methodology/quick-start/) naming convention.

* Mark child elements with ```__```
* Modifier classname with ```--```
* Do not over scope child elements !

```html
<header class="site-header">
  <hgroup class="site-header__wrap">

    <aside class="site-header__column">
      <p>Column section in site-header, not a child of __wrap</p>
      <span class="site-header__column__elem">Site header column's child element</span>
    </aside>

    <aside class="site-header__column --modifier">
      ...
    </aside>

  </hgroup>
</header>
```

### SCSS structure example :

```css
.site-header {
  /* save for later */
  $self: &;

  /* basic style */

  &__wrap {
    /* do stuff outside utility-class */

    /* media query mixin */
    @include media-breakpoint-up(lg) {
      /* do stuff for lg breakpoint */
    }
  }

  &__column {
    color: blue;

    /* site-header > column > child element */
    &__elem {
      opacity: 0.5;
    }

    /* a modifier class example */
    &.--modifier {
      opacity: 0.25;
    }
  }

  /* State changes rules : */

  &:hover,
  &.--active {
    /* refer to $self variable to avoid retyping .site-header on state change */
    #{$self}__elem {
      opacity: 1;
    }
  }
}
```
## @mill3/system-ui-sass

We developed our own utility class framework for layout structure. It's inspired by Bootstrap and TailwindCSS.

**NPM :**

https://www.npmjs.com/package/@mill3-packages/system-ui-sass

**Source :**

https://github.com/Mill3/mill3-packages/tree/master/packages/system-ui-sass/src

All available classes are documented in our Storybook site :

https://mill3-system-ui-sass-demo-site.netlify.app/

Real world examples here :

https://mill3-system-ui-sass-demo-site.netlify.app/?path=/story/examples-grids--advanced-grid-layout
## JS UI/Components/Modules/Utilities structure

All JS classes should be stored in a directory representing its name. More on that later, but that structure is important when used with our BarbaJS Webkapack chunkloading plugin.

### Anatomy of a class

```modules/my-module-name/index.js```

```js
import FooBar from 'foo-bar-npm-package';

// const on top, you can export when needed
const MY_CONST_SELECTOR = `[data-selector]`
export const EXPORTABLE_CONST = `[data-selector]`

class MyModuleName {
  constructor() {
    this.initialized = false;
    this.foo = null;
    this._methodForBinding = this._methodForBinding.bind(this);
  }

  get name() {
    return `MyModuleName`;
  }

  // init stuff
  init() {
    // prevent double init
    if(this.initialized) return;

    // do some smart stuff

    // set class as initialized
    this.initialized = true;
  }

  // destroy stuff
  destroy() {
    // undo smart stuff
    this.initialized = false;
  }

  // create events
  _bindEvents() {
    this.foo.addEventListener('click', this._methodForBinding);
  }

  // destroy events
  _unbindEvents() {
    this.foo.removeEventListener('click', this._methodForBinding);
  }

  // underscore prefix means private
  _privateMethod() {}

  // binding method, must be binded to class in constructor method
  _methodForBinding() {}

}
```
## BarbaWebpackChunks plugin - DOM-Controller

We use the ```dom-controller``` approach with our JS module. When the page is ready, the app scan the source and search for UI and Module classes :

```html
<div data-ui="site-nav"></div>
<div data-module="my-module"></div>
```

An element can cast 1 or multiple class each seperated by a coma.

```html
<div data-ui="site-nav,foo-bar"></div>
```

The data-module or data-ui value ```my-module-name``` is transformed to PascalCase ```MyModuleName```, this should match your class static name :

```html
<div data-module="my-module-name"></div>
```

Which point to a file in ```modules/my-module-name/index.js```

Inside the same class, you must use the ```PascalCase``` format or the class **won't** init :

```js
get name() {
  return `MyModuleName`;
}

init() {
  // init stuff
}

destroy() {
  // destroy stuff
}
```

**Barba plugins reference :**

https://github.com/Mill3/mill3-wp-theme-boilerplate/blob/master/src/js/core/barba.webpack-chunks.js
https://github.com/Mill3/mill3-wp-theme-boilerplate/blob/master/src/js/core/barba.dom-controller.js

## Event Emitters

Un traversal système est souvent nécessaire pour permettre un ou plusieurs module de communiquer entre eux.

Exemple type, module ```SiteNav.js``` et ```MegaMenu.js```, les éléments du menu doivent ouvrir le MenuMenu en ```:hover```, donc envoyer un event ```Emitter.emit("MegaMenu.open")```.

Chaque module reçoit le même ```EventEmitter2``` global, à partir duquel il pourra attacher des events.

https://www.npmjs.com/package/eventemitter2

```js
\\ app.js

const emitter = new EventEmitter2()
const FoobarClassInstance = new FoobarClass()
FoobarClassInstance.emitter = emitter

```

Ensuite dans la class

```js
// @modules/foo/FoobarClass.js
_registerEvents() {
  if(!this.emitter) return
  this.emitter.once('Foobar.dummy', this._dummy)
}

_dummy() {
  console.log(`Act Act !`)
}

```

Finalement dans un autre module :

```js
// @modules/site-nav/SiteNav.js
button.addEventListener(`mouseenter`, this.emitter.emit('Foobar.dummy'))

```


## Mill3 WP Boilerplate

https://github.com/Mill3/mill3-wp-theme-boilerplate

TODO: write a good and smart tutorial about this :)
## ARIA good practices

### Hiding element from screen readers

Add aria-hidden="true" to all DOM elements that you want to hide from screen readers.
This includes:
- design related elements
- elements that do not add any value from a SEO standpoint
- duplicated content for some responsive layouts

### ARIA-label

When creating an HTML structure for a button or link, you may need to create inner element in order to perform animations or honor designers work. In these cases, these extra DOM elements may confuse screen readers.

In order to prevent screen reader confusion, add aria-hidden="true" to your button's internal HTML structure. Then, force button label by adding aria-label="My button" to your button or link. This way, screen readers will ignore internal HTML structure and read button or link as it should be.

```html
<button class="btn" aria-label="My button">
  <span class="btn__label" aria-hidden="true">My button</span>
  <span class="btn__underline" aria-hidden="true"></span>
</button>
```

### Using button -vs- link

It's always tempting to use link for everything that received mouse/touch input. Unfortunately, it's bad pratice. Screen readers assume that a link will, by design, redirect to another URL. Then end result has been read like button text (or aria-label), then href attribute. If no href is provided or href="#", it will give wrong information to end user that may leave him from the website.

You must use button for actions other than url navigation.


## Locomotive-scroll recipes

### Fixed panel reveal

**Concept**

The idea behind this concept is to have a fixed element, which is revealed by his parent.  
Imagine this as you are seeing the world through a shoe box. The world is your fixed element and the shoe box is your viewport.

As the viewport is scrolling down, the world reveal himself from the bottom. In a normal scrolling experience, you would see the top of your fixed element appearing from the bottom and moving up to viewport's top. 

In a fixed panel reveal, when viewport is scrolling down, your fixed element's bottom would stick to viewport's bottom and reveal his content from bottom to top. 


**Example**

We first use this technique in [MILL3 website](https://mill3.studio).

<video src="./assets/locomotive-scroll-fixed-panel-reveal.mp4" width="720" height="446" controls></video>

![Langmobile](../../blob/master/assets/00_Illustration04_00000.png "Langmobile")

**Naming convention**

Viewport: DOM element containing everything. Scroll normally on the page (smooth-scroll or not). 
Panel: Fixed DOM element containing all visual content for user (text/image/video/etc..). 
Sticky Target: DOM element setting panel's limits.

**Code**

```html
<section class="viewport position-relative overflow-hidden" data-scroll-section>
  <div class="sticky-target position-absolute t-0 l-0 w-100">
    <div 
      class="panel d-flex flex-column justify-content-center align-items-center" 
      data-scroll 
      data-scroll-target=".sticky-target" 
      data-scroll-sticky
    >
      <h1 class="m-0 mb-4">Hello World</h1>
      <p>This is a fixed panel.</p>
    </div>
  </div>
</section>
```
```css
.viewport {
  height: calc(var(--vh) * 100);
}
.sticky-target {
  top: calc(var(--vh) * -100);
  bottom: calc(var(--vh) * -100);
}
.panel {
  height: calc(var(--vh) * 100);
}
```

Your ``.viewport``, ``.sticky-target`` and ``.panel`` will, most of the time, be the same height.  
If you want your panel to stay fixed longer in viewport, increase viewport's height to 200vh or 300vh.  


**Panel smaller than 100vh**

This technique required some adjustment to work with a panel smaller than 100vh.  
The problem is that smaller panel need to stick to viewport's bottom until they are fully revealed. Then they need to scroll normally with the flow of the page. This little details implies some modifications to our previous code.

```html
<section class="viewport position-relative overflow-hidden" data-scroll-section>
  <div class="sticky-target position-absolute t-0 l-0 w-100">
    <div 
      class="panel d-flex align-items-end" 
      data-scroll 
      data-scroll-target=".sticky-target" 
      data-scroll-sticky
    >
      <div class="wrapper w-100 d-flex flex-column justify-content-center align-items-center">
        <h1 class="m-0 mb-4">Hello World</h1>
        <p>This is a fixed panel.</p>
      </div>
    </div>
  </div>
</section>
```
```css
.viewport {
  height: calc(var(--vh) * 50);
}
.sticky-target {
  top: calc(var(--vh) * -100);
  bottom: 0;
}
.panel {
  height: calc(var(--vh) * 100);
}
.wrapper {
  height: calc(var(--vh) * 50);
}
```

1. Wrap your content inside another DOM element.
2. ``.wrapper`` height is equal to ``.viewport``.
3. ``.panel`` needs to align his children to bottom.
4. ``.sticky-target`` bottom equal 0.

**Notes:** Smaller panel technique has not been tested on a lots of project. It may not work as expected on your project. If so, ask Dominic for some help.