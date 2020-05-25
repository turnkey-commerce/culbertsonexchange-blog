---
title: JQuery UI Autocomplete and MVC3 Remote Validation Conflict–Update
author: James
type: post
date: 2011-12-19T01:59:53+00:00
url: /?p=229
categories:
  - Software Development

---
I [previously posted][1] about a fix to a problem when the JQuery UI Autocomplete doesn’t play well with the ASP.Net MVC3 Remote validation. Part of the solution included a way to ensure that the remote validation isn’t fired for each key click or change of focus in the text box.

The method for doing this is to change the settings of the validation to not fire onkeyup or onfocusout, but to get there the validator settings must be obtained from the form in question.  The following snippet was given to grab the settings (<span style="color: #ff0000">do not use this method</span>):

<pre class="brush: jscript; title: ; notranslate" title="">var validatorSettings = $.data($('form')[0], 'validator')
</pre>

The problem is that this method of getting the settings will not work correctly if a new form is inserted before the form in question, because that will case the original form to have a counter of [1] and the code will no longer work. Better to use the following method to get the form by named id:

<pre class="brush: jscript; title: ; notranslate" title="">var validatorSettings = $('#searchForm').validate().settings;</pre>

Here&#8217;s how it looks in the whole snippet of Javascript:

<pre class="brush: jscript; title: ; notranslate" title="">&lt;script type="text/javascript"&gt;
    window.onload = function () {
        var validatorSettings = $('#searchForm').validate().settings;
        validatorSettings.onkeyup = false;
        validatorSettings.onfocusout = false;
    }
&lt;/script&gt;
</pre>

Then the form can be given an id so that JQuery will select the correct form:

<pre class="brush: csharp; title: ; notranslate" title="">@using (Html.BeginForm("Index", "Services", FormMethod.Get, new { id = "searchForm" })) {
</pre>

For the full code [refer to the previous article][1] (where this issue has been corrected).

 [1]: http://www.culbertsonexchange.com/wp/?p=190