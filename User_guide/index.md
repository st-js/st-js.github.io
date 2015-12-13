---
layout: userguide
---

{% comment %}
=============================================
TODO: Create an issue to support @FunctionalInterface on
TODO: top of @JavascriptFunction

TODO: Double check that it is not possible for a non-anonymous
TODO: class to implement an @JavaScript function

TODO: Double check what happens to the type of this with:
TODO:   - Lambdas nested inside anonymous classes
TODO:   - anonymous classes nested inside lambdas

TODO: Can we support automatic `this` binding with anonymous inner classes?
=============================================
{% endcomment %}

# User Guide index for ST-JS {{site.last_stjs_version}}

{% comment %}Extracting documentation pages{% endcomment %}
{% assign userguidePages = site.pages | where: "isUserGuide", true | sort: "url" %}
{% assign userguideRoot = "User_guide" %}

{% comment %}This will print all documentation hierachy{% endcomment %}
{% include show-children.html dir=userguideRoot docs=userguidePages level=2 %}
