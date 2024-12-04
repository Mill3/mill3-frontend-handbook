- [Mill3 Studio - The frontend handbook 📙 🐉 🧙‍♂️](#mill3-studio---the-frontend-handbook---️)
  - [Modules structre convention](#modules-structre-convention)
  - [Classname naming convention (BEM)](#classname-naming-convention-bem)
    - [SCSS structure example :](#scss-structure-example-)
  - [MILL3 Sass Library](#mill3-sass-library)
  - [JS UI/Components/Modules/Utilities structure](#js-uicomponentsmodulesutilities-structure)
    - [Anatomy of a class](#anatomy-of-a-class)
  - [Windmill](#windmill)
  - [Event Emitters](#event-emitters)
  - [Mill3 WP Boilerplate](#mill3-wp-boilerplate)
  - [ARIA good practices](#aria-good-practices)
    - [Hiding element from screen readers](#hiding-element-from-screen-readers)
    - [ARIA-label](#aria-label)
    - [Using button -vs- link](#using-button--vs--link)
  - [Smooth scroll recipes](#smooth-scroll-recipes)
    - [Fixed panel reveal](#fixed-panel-reveal)
  - [Converting font for the web](#converting-font-for-the-web)
    - [Variable Fonts](#variable-fonts)
  - [Fixing webfont line-height problem](#fixing-webfont-line-height-problem)


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
src/js/modules/pb-row-medias/index.js
src/js/ui/site-nav/index.js
```

**CSS module**

```
src/scss/modules/_accordions.scss
src/scss/page-builder/_pb-row-medias.scss
src/scss/ui/_site-nav.scss
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
@use "@mill3-sass-mixins/breakpoints";

.site-header {
  /* save for later */
  $self: &;

  /* basic style */

  &__wrap {
    /* do stuff outside utility-class */

    /* media query mixin */
    @include breakpoints.media-breakpoint-up(xl) {
      /* do stuff for xl breakpoint */
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

## MILL3 Sass Library

We developed our own utility class framework for layout structure. It's inspired by Bootstrap and TailwindCSS.

All available classes are documented in our Storybook site :

https://mill3-system-ui-sass-demo-site.netlify.app/

Real world examples here :

https://mill3-system-ui-sass-demo-site.netlify.app/?path=/story/examples-grids--advanced-grid-layout

## JS UI/Modules structure

All JS modules should be stored in a directory representing its name. That structure is important when used with Webpack chunkloading plugin.

### Anatomy of a module

```modules/my-module-name/index.js``` or ```ui/my-module-name/index.js```

```js
import FooBar from 'foo-bar-npm-package';
import { $ } from '@utils/dom';
import { on, off } from '@utils/listener';

// constants on top, you can export when needed
const MY_CONST = 'MY_CONST';
export const EXPORTABLE_CONST = 'EXPORTABLE_CONST';

class MyModuleName {
  constructor(el, emitter) {
    this.el = el; // DOM element referenced with [data-module="my-module-name"]
    this.emitter = emitter; // global emitter for communication between modules and other site wise architecture (js/core/emitter.js)
    this.foo = $('.foo', this.el);

    // methods binding
    // sometimes, it's required to bind class methods to make sure "this" is referenced to the correct object
    this._methodForBinding = this._methodForBinding.bind(this);
  }

  // REQUIRED METHOD
  // invoked by Windmill (js/core/windmill.js) before module is visible to user
  init() {
    // do some smart stuff
    this._bindEvents();
  }

  // REQUIRED METHOD
  // invoked by Windmill (js/core/windmill.js) just before DOM element is removed from page
  destroy() {
    // unbind UI related events
    this._unbindEvents();

    // delete references to all properties and methods binding
    this.el = null;
    this.emitter = null;
    this.foo = null;

    this._methodForBinding = null;
  }

  // OPTIONAL METHOD
  // invoked by Windmill (js/core/windmill.js) after all modules are initialized and SiteTransition is completed
  start() {}

  // OPTIONAL METHOD
  // invoked by Windmill (js/core/windmill.js) right after window's History API change
  stop() {}

  _bindEvents() {
    on(this.foo, 'click', this._methodForBinding)
  }
  _unbindEvents() {
    off(this.foo, 'click', this._methodForBinding)
  }

  // methods prefixed with _ (underscore) = private method
  _privateMethod() {}

  // binded method, reference to this will be correct
  _methodForBinding() {}

}
```

## Windmill

MILL3's Windmill is our internal solution to SPA page transition. Windmill is responsible for listening and triggering window's History API, loading requested URL via AJAX and replacing the new page's content at runtime. For more documentation about Windmill, see ```js/core/windmill.js```.

### Modules loading using Windmill ###

We prioritize asynchronous modules loading by default. This way of thinking allow us to load only the minimum JS during initial page loading.  
All of this rely on Webpack Chunk plugin. To explain shortly, Webpack bundle all folders in ```js/modules/``` and ```js/ui``` into a small independant package.  
When the DOM is ready, we scan all ```[data-module]``` and ```[data-ui]``` DOM nodes and load their specific modules.  
The ```[data-module]``` or ```[data-ui]``` value should match your module filepath : ```modules/my-module/index.js```

```html
<div data-ui="site-nav"></div>
<div data-module="my-module"></div>
```

An element can cast multiple modules at the same time. Seperated each module by a coma.  
In example below, Webpack will load ```js/ui/site-nav/index.js``` AND ```js/ui/foo-bar/index.js```.  

```html
<div data-ui="site-nav,foo-bar"></div>
```

## Event Emitters

Un système d'évènements centralisés est souvent nécessaire pour permettre un ou plusieurs module de communiquer entre eux.

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


## Smooth scroll recipes

### Fixed panel reveal

**Concept**

The idea behind this concept is to have a fixed element, which is revealed by his parent.  
Imagine this as you are seeing the world through a shoe box. The world is your fixed element and the shoe box is your viewport.

As the viewport is scrolling down, the world reveal himself from the bottom. In a normal scrolling experience, you would see the top of your fixed element appearing from the bottom and moving up to viewport's top. 

In a fixed panel reveal, when viewport is scrolling down, your fixed element's bottom would stick to viewport's bottom and reveal his content from bottom to top. 

**Example**

We first use this technique in [MILL3 website](https://mill3.studio).

![MILL3 Fixed Panel Reveal](/assets/images/fixed-panel-reveal-v2.gif "MILL3 Fixed Panel Reveal")

**Naming convention**

Viewport: DOM element containing everything. Scroll normally on the page (smooth-scroll or not).  
Panel: Fixed DOM element containing all visual content for user (text/image/video/etc..).  
Sticky Target: DOM element setting panel's limits.

**Code**

```html
<section class="viewport">
  <div class="sticky-target w-100">
    <div class="panel w-100 d-flex flex-column justify-content-center align-items-center">
      <h1 class="m-0 mb-4">Hello World</h1>
      <p class="m-0">This is a fixed panel.</p>
    </div>
  </div>
</section>
```
```css
.viewport {
  clip-path: inset(0);
  height: 100vh;
  position: relative;
}
.sticky-target {
  position: absolute;
  top: -100vh;
  bottom: -100vh;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
}
.panel {
  height: 100vh;
  position: sticky;
  bottom: 0;
}
```

[See code in action](https://codepen.io/ebhoren/full/RwzBONV)  

Your ``.viewport`` and ``.panel`` will, most of the time, be the same height.  
If you want your panel to stay fixed longer in viewport, increase viewport and panel height to 200vh or 300vh.  
``.sticky-target`` height will be always be ``.panel`` + 200vh (top: -100vh; bottom: -100vh).  
These top/bottom offset will allow ``.panel`` to stick into viewport during the full scrolling.  


**Panel smaller than 100vh**

![Panel smaller than 100vh](/assets/images/fixed-panel-small.gif "Panel smaller than 100vh")

This technique required some adjustment to work with a panel smaller than 100vh.  
The problem is that smaller panel need to stick to viewport's bottom until they are fully revealed. Then they need to scroll normally with the flow of the page. This little details implies some modifications to our previous css code.

```css
.viewport {
  height: 50vh;
}
.sticky-target {
  top: -50vh;
  bottom: 0;
}
.panel {
  height: 50vh;
}
```

[See code in action](https://codepen.io/ebhoren/full/QWXBRwL)  

**Notes:** Smaller panel technique has not been tested on a lots of project. It may not work as expected on your project. If so, ask Dominic for some help.



## Converting font for the web

To begin, you need to install [FontTools](https://formulae.brew.sh/formula/fonttools) via Homebrew. 
When it's done, run this command in Terminal:

```bash
pyftsubset "input.ttf" --flavor=woff2 --output-file="ouput.woff2" --unicodes="U+0020-007F,U+00A0-00FF,U+20AC,U+20BC,U+2010,U+2013,U+2014,U+2018,U+2019,U+201A,U+201C,U+201D,U+201E,U+2039,U+203A,U+2026,U+2022"
```

What it does it that it take a .ttf or .otf file and used only a subset of characters (specified in the --unicodes parameter).  
Then, it compress and save output in .woff2 format.  
Technique was inspired by [Walter Bert](https://walterebert.com/blog/subsetting-web-fonts/).  

**Notes:** To know which unicodes to use for your project, see [https://www.zachleat.com/unicode-range-interchange/](https://www.zachleat.com/unicode-range-interchange/). Paste unicodes in left textarea, and it will show all characters that will be included in font. If you need more character, add them in the middle textarea. If right textarea's size doesn't change after adding characters in middle textarea, it means that this character is already included in left textarea unicodes ranges.  

```css
@font-face {
  font-family: 'My-Font-Family';
  src: url('#{$font-directory}My-Font-Family.woff2') format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: block;
  unicode-range: U+0020-007F,U+00A0-00FF,U+20AC,U+2010,U+2013,U+2014,U+2018,U+2019,U+201A,U+201C,U+201D,U+201E,U+2039,U+203A,U+2026,U+2022;
}
```

### Variable Fonts

For variable fonts you should add woff2 supports variations and woff2-variations format declarations for better browser compatibility. 

```css
@font-face {
  font-family: 'My-Font-Family';
  src: url('#{$font-directory}My-Font-Family.woff2') format("woff2 supports variations"),
       url('#{$font-directory}My-Font-Family.woff2') format("woff2-variations");
  font-weight: 400;
  font-style: normal;
  font-display: block;
  unicode-range: U+0020-007F,U+00A0-00FF,U+20AC,U+2010,U+2013,U+2014,U+2018,U+2019,U+201A,U+201C,U+201D,U+201E,U+2039,U+203A,U+2026,U+2022;
}
```


## Fixing webfont line-height problem

1. Install XCode Font Tools
2. run in Terminal: ftxdumperfuser -t hhea -A d my-font-name.extension
3. Open my-font-name.hhea.xml in your favorite text editor
4. Check value of ascender property
5. Go to [https://www.fontsquirrel.com/tools/webfont-generator]
6. Upload your font and choose "expert" mode.
7. In the Vertical Metrics tab, choose Custom Adjustment. Increase value from ascender value you have seen before. You may need to test various value before getting the correct one.
8. No need to change X-Height Matching or anything else. Download your kit and install it in your project.