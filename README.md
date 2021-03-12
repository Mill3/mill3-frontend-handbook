# Mill3 Studio - The frontend handbook 📙 🐉 🧙‍♂️

Documenting our good practices and toolsets.
## Topics

* UI/Components/Modules/Utilities structure
* Classname naming convention (BEM)
* Our utility class system (@mill3/system-ui-sass)
* Webpack chunks loader explained
* Mill3 WP Boilerplate
* ARIA good practices

## We split our modules using the following convention

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

## Classname naming convention

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

### SCSS structure :

```css
.site-header {
  /* save for later */
  $self: &;

  /* basic style */

  &__wrap {
    color: red;
  }

  &__column {
    color: blue;

    /* site-header > column > child element */
    &__elem {
      opacity: 0.5;
    }

    /* a modifier class example */
    &.--modifier {
      color: rebeccapurple;
    }
  }

  /* State changes rules : */

  &:hover,
  &.--active {
    /* refer to $self variable to avoid retyping .site-header on state change */
    #{$self}__column {
      color: gold;
    }
  }
}
```

## @mill3/system-ui-sass



https://www.npmjs.com/package/@mill3-packages/system-ui-sass

https://mill3-system-ui-sass-demo-site.netlify.app/

Exemple :

https://codepen.io/Coderesting/pen/yLyaJMz
## JS UI/Components/Modules/Utilities structure

All JS classes should be stored in a directory representing its name. More on that later, but that structure is important when used with our BarbaJS Webkapack chunkloading plugin.

TODO : explain the factory patern

> factory function is any function which is not a class or constructor that returns a (presumably new) object.
## BarbaWebpackChunks plugin

TODO

```html
<header
  class="site-header"
  data-ui="site-header"
>
```

https://github.com/Mill3/mill3-wp-theme-boilerplate/blob/master/src/js/core/barba.webpack-chunks.js

## Mill3 WP Boilerplate

https://github.com/Mill3/mill3-wp-theme-boilerplate
## ARIA good practices

TODO

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