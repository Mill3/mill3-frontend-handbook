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
      <span class="site-header__column__elem">Column child element</span>
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

TODO : explain the factory patern

> factory function is any function which is not a class or constructor that returns a (presumably new) object.
## Webpack chunks loader

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

### Hiding element from screen readers
Add aria-hidden="true" to all DOM elements that you want to hide from screen readers.  
This includes:
- design related elements 
- elements that do not add any value from a SEO standpoint
- duplicated content for some responsive layouts