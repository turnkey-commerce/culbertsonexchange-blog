---
title: JQuery UI Autocomplete and MVC3 Remote Validation Conflict–Update
author: James
type: post
date: 2011-12-19T01:59:53+00:00
aliases:
  - /permalink/p229
categories:
  - Software Development
comments: false

---
I [previously posted][1] about a fix to a problem when the JQuery UI Autocomplete doesn’t play well with the ASP.Net MVC3 Remote validation. Part of the solution included a way to ensure that the remote validation isn’t fired for each key click or change of focus in the text box.

The method for doing this is to change the settings of the validation to not fire onkeyup or onfocusout, but to get there the validator settings must be obtained from the form in question.  The following snippet was given to grab the settings (**do not use this method**):

{{< highlight "Javascript" >}}
var validatorSettings = $.data($('form')[0], 'validator')
{{< / highlight >}}

The problem is that this method of getting the settings will not work correctly if a new form is inserted before the form in question, because that will case the original form to have a counter of **1** and the code will no longer work. Better to use the following method to get the form by named id:

{{< highlight "Javascript" >}}
var validatorSettings = $('#searchForm').validate().settings;
{{< / highlight >}}

Here's how it looks in the whole snippet of Javascript:

{{< highlight "Javascript" >}}
<script type="text/javascript">
    window.onload = function () {
        var validatorSettings = $('#searchForm').validate().settings;
        validatorSettings.onkeyup = false;
        validatorSettings.onfocusout = false;
    }
</script>
{{< / highlight >}}

Then the form can be given an id so that JQuery will select the correct form:

{{< highlight "C#" >}}
@using (Html.BeginForm("Index", "Services", FormMethod.Get, new { id = "searchForm" })) {
{{< / highlight >}}

For the full code [refer to the previous article][1] (where this issue has been corrected).

 [1]: {{< relref "2011-09-05-jquery-ui-autocomplete-and-mvc3-remote-validation-conflict.md" >}}
