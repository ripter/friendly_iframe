# friendly_iframe
create a friendly iframe with an API run on the parent.

# Iceberg loading.
The friendly iframe can be used to load iceberg code onto the page. This is code that we want to run inside of the iframe, but we want to provide a surface API that it can use to work with the parent frame.

`surface.js`:
```
const $ = require('jquery');
const fif = require('friendly_iframe');

// Load the rest of the code.
fif('js/iceberg.bundle.js', {
  $: $
});
```

## The Embeded Problem
When creating a library or component that is loaded onto other people's pages, we have to be careful to not break anything else that might exist on the page, and to prevent things on the page from affecting our code.

The friendly iframe allows us to load most of our code in an iframe, where we can control every library on the page. This even protects us from hacks that change the native library.

## Real world example.
At Tout we had an issue with a page that used an old library [scriptaculous](https://script.aculo.us/). This issue is that this library implements it's own `Object.keys()` method that does not use `hasOwnProperty`. This resulted in the ToutPlayer not working because `Object.keys` would walk up the prototype and return unexpected results.

Our code was already wrapped in an [IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) by webpack. This provides a lot of protection and allows us to safely use our version of jQuery. But it doesn't protect us from changes to that native library.

The first option might be to stop using `Object.keys()` and instead use something like [lodash](https://lodash.com/docs) or [underscore](http://underscorejs.org/). The issue is that these libraries use a  polyfill for native functions like `Object.keys()`. This means that they will use the *broken* version when loaded with a library like scriptaculous.

Our best hope is Iceberg loading. In this process we have two Javascript files. `surface.js` and `iceberg.js`.

`surface.js` is an IIFE that contains everything we need to manipulate the page. This is our risk surface. It will still get broken or hacked native functions on the page. But we can make surface as small as possible.

`iceberg.js` is the rest of our code. It doesn't matter if it is IIFE or not, because it is the only script in the iframe. The risk is reduced to just iframe origin issues. And since it's a friendly iframe, the origin is already set to the page.
