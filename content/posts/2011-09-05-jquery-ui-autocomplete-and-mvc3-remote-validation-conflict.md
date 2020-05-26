---
title: JQuery UI Autocomplete and MVC3 Remote Validation Conflict
author: James
type: post
date: 2011-09-05T18:44:36+00:00
url: /?p=190
categories:
  - Software Development
tags:
  - JQuery
  - MVC
  - Validation

---
On a recent project I had a great deal of trouble to get [JQuery UI Autocomplete][1] and [ASP.Net MVC3 Remote Validation][2] to play nicely with each other. The goal was to have a type-in for selecting a city for a search in which the pull-down would auto-complete based on the available cities in the database:

![Autocomplete in action](http://www.culbertsonexchange.com/wp/wp-content/uploads/2011/09/selection_thumb.png)

Once a city is selected from the autocomplete it should do a validation as to whether the city is populated with actual data in the database (in case the user types in an invalid city:

![Validation for missing city](http://www.culbertsonexchange.com/wp/wp-content/uploads/2011/09/selection2_thumb.png)

This is accomplished using the MVC 3 remote validation which makes it easy to do by adding an annotation to the ViewModel to indicate that the attribute will be remotely validated:

{{< highlight "C#" >}}
public class SearchFormViewModel {
    ...
    [Remote("IsCityAvailable","Listings",ErrorMessage="Selected city has no listings, please reselect.")]
    public string City { get; set; }
}
{{< / highlight >}}

This of course requires an &#8220;IsCityAvailable&#8221; action on the &#8220;Listings&#8221; controller that will return false if there are no database entries for the given city:

{{< highlight "C#" >}}
    //Check that a city exists on the search selector.
    public JsonResult IsCityAvailable(string city) {
            City findCity = cityRepository.FindByCityName(city);
            if (findCity == null) {
                return Json(false, JsonRequestBehavior.AllowGet);
            } else {
                return Json(true, JsonRequestBehavior.AllowGet);
            }
     }
{{< / highlight >}}

### Conflict with Autocomplete

The problem comes in when the user types in a part of the city name and then selects from the drop-down autocomplete, it causes the remote validation to fire but it only includes the text that was typed, e.g. “A” rather than the selected drop-down text.  This causes the validation to fail when it should indicate success.

This has to do with the way the JQuery validation fires based on key-up and blur events. Researching the web shows that there are ways to turn this part off, but it is complicated by the layer that MVC3 puts on top of the JQuery validation.

### Solution

The solution was to add some Javascript to the partial view that defined the search inputs so that the “onkeyup” and “onfocusout” settings are set to false. This is done in the “window.onload” so that it they are turned off early enough in the process:

{{< highlight "C#" >}}
<script type="text/javascript">
    window.onload = function () {
        var validatorSettings = $('#searchForm').validate().settings;
        validatorSettings.onkeyup = false;
        validatorSettings.onfocusout = false;
    }
</script>
{{< / highlight >}}

Then a JQuery handler is added to the Search button click event so that the validate is forced when the search button is pressed (also works when the form is submitted via a return):

{{< highlight "Javascript" >}}
$("#searchSubmit").click(function (event) {
    $("#searchForm").validate().element("input#City");
});
{{< / highlight >}}

 [1]: http://jqueryui.com/demos/autocomplete/
 [2]: http://msdn.microsoft.com/en-us/library/gg508808(v=vs.98).aspx