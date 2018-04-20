---
showToc: false
---

# Formatting

## Hints

You can now provide hints in various ways using the hint tag.
```
{% hint style='info' %}
Important info: this note needs to be highlighted
{% endhint %}
```
The above example will produce a styled alert, with an icon:

Available styles are:

- info (default)
- tip
- danger
- working

{% hint style='info' %}
Important info: this note needs to be highlighted
{% endhint %}

{% hint style='tip' %}
Tip: this note needs to be highlighted
{% endhint %}

{% hint style='danger' %}
Danger: this note needs to be highlighted
{% endhint %}

{% hint style='working' %}
Working: this note needs to be highlighted
{% endhint %}

## Code Blocks 

Plugin: [codetabs](https://github.com/GitbookIO/plugin-codetabs)
This is a code block with tabs for each languages:

```
{% codetabs name="Python", type="py" -%}
msg = "Hello World"
print msg
{% language name="JavaScript", type="js" -%}
var msg = "Hello World";
console.log(msg);
{% language name="HTML", type="html" -%}
<b>Hello World</b>
<p>Wassup</p>
{% endcodetabs %}
```

translates to: 

{% codetabs name="Python", type="py" -%}
msg = "Hello World"
print msg
{% language name="JavaScript", type="js" -%}
var msg = "Hello World";
console.log(msg);
{% language name="HTML", type="html" -%}
<b>Hello World</b>
<p>Wassup</p>
{% endcodetabs %}

For languages using syntax like `{{`, `{%`  we have to escape the content using `{% raw %}<h1>Hello {{yourName}}!</h1>{% endraw %}` for example:

{% codetabs name="Python", type="py" -%}
{% raw %}<h1>Hello {{yourName}}!</h1>{% endraw %}
{% language name="React", type="js" -%}
var React = require('react')
{% endcodetabs %}