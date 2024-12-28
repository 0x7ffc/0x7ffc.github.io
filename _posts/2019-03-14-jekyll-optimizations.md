---
layout: post
title: "Jekyll Optimizations"
description: "Some easy Jekyll optimizations such as local fonts, mathjax 3 and CSS inlining which make your site loads faster."
mathjax: true
---

Things we'll do:

- Host fonts locally.
- Mathjax version 3.
- CSS inlining.

## Local Font

Host fonts locally speeds things up a lot, especially when the font CDN is slow in your region.

[Google webfonts helper](https://google-webfonts-helper.herokuapp.com/fonts) provides easy download interface. I put all my fonts inside `assets/fonts` folder, and inside "_scss/_fonts.scss":

```css
@font-face {
  font-family: 'Roboto Mono';
  font-style: normal;
  font-weight: 400;
  src: url('/assets/fonts/roboto-mono-v12-latin-ext_latin-regular.eot'); /* IE9 Compat Modes */
  src: local(''),
       url('/assets/fonts/roboto-mono-v12-latin-ext_latin-regular.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
       url('/assets/fonts/roboto-mono-v12-latin-ext_latin-regular.woff2') format('woff2'), /* Super Modern Browsers */
       url('/assets/fonts/roboto-mono-v12-latin-ext_latin-regular.woff') format('woff'), /* Modern Browsers */
       url('/assets/fonts/roboto-mono-v12-latin-ext_latin-regular.ttf') format('truetype'), /* Safari, Android, iOS */
       url('/assets/fonts/roboto-mono-v12-latin-ext_latin-regular.svg#RobotoMono') format('svg'); /* Legacy iOS */
}
```

Don't forget the `@import "fonts"` statement in your main stylesheet.

## Mathjax 3

Mathjax version 3 is lot faster than version 2. However there is a catch.

In version 2.7, MathJax ran a preprocessor on HTML files that converted math delimiters into `<script>` tags:

- inline math `$..$` becomes: `<script type="math/tex">...</script>`
- display math `$$..$$` becomes: `<script type="math/tex; mode=display">...</script>`

MathJax 3 on the other hand directly parses math delimiters in HTML documents and renders math expressions which makes the `math_engine` option in Jekyll's `_config.yml` obsolete. However, GitHub Pages currently always sets `math_engine: mathjax`, overriding user configuration.

So to fix this we convert all `<script>` back to plain math Tex.

```html
{{ "{% if page.mathjax " }}%}
<script type="text/javascript">
  MathJax = { tex: { inlineMath: [['$', '$'], ['\\(', '\\)']] } };
  document.addEventListener('DOMContentLoaded', function() {
    function stripcdata(x) {
      if (x.startsWith('% <![CDATA[') && x.endsWith('%]]>'))
	return x.substring(11,x.length-4);
      return x;
    }
    for (node of document.querySelectorAll('script[type^="math/tex"]')) {
      const inline = !node.type.match(/; *mode=display/);
      if (inline) {
	node.outerHTML = "\\(" + stripcdata(node.textContent) + "\\)";
      } else {
        node.outerHTML = "\\[" + stripcdata(node.textContent) + "\\]";
      }
    }
    var script = document.createElement('script');
    script.async = true;
    script.id = "MathJax-script";
    script.src = "https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js";
    document.head.appendChild(script);
  }, false);
</script>
{{ "{% endif " }}%}
```

I use `page.mathjax` to optionally render mathjax: inside my blog post front matter, use `mathjax: true` to control the behavior.

Now you can write inline math: $J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha}$, as well as display math:

$$
J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha}
$$

If you want to speed up things a little further, you can host Mathjax fonts locally as we did before.

(There is a [bug](https://github.com/mathjax/MathJax-src/issues/335) with amsCd)

## CSS Inlining

First, move your `main.scss` file to `_includes/main.scss`, now you can use `{{ "{% include main.scss " }}%}` in your template. Also remove the front matter inside `main.scss` since you don't need Jekyll to process it anymore. You can still use `@import` to import files inside `_scss`. In the end, add these to your `head.html`:

```xml
{{ "{% capture styles " }}%}
{{ "{% include main.scss " }}%}
{{ "{% endcapture " }}%}
<style>
  {{ "{{ styles | scssify " }}}}
</style>
```
