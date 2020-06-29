---
title: Using SimpleBrowser with SpecFlow for BDD with ASP.NET MVC
author: James
type: post
date: 2012-01-20T02:45:00+00:00
aliases:
  - /permalink/p249
categories:
  - Software Development
tags:
  - asp.net
  - simplebrowser
  - specflow

---
The [SpecFlow][1] project provides a useful way to integrate BDD (Behavior Driven Development) testing into .NET projects using the Gherkin language. The main idea is that the project specs or user stories are written in user language and then are linked to actual tests that are automatically executed as the project is developed. There are several possible ways to implement this with ASP.NET MVC to automate the testing with the browser.

### WatiN

Steve Sanderson wrote a nice article about applying [SpecFlow with ASP.Net MVC][2] In this case he used the [WatiN][3] tool to drive the tests through a browser. I implemented a similar testing setup on a project using SpecFlow and WatiN but found that driving an actual browser made the tests too slow.

### HtmlUnit

A later article from Sanderson describes a way to use [HtmlUnit for a “headless browser automation”][4] approach to testing This approach is faster but requires using the IKVM method of linking Java within .NET. To ease this aspect of it there is a Nuget package called [NHtmlUnit][5] I did a small spike with NHtmlUnit but found it slower than I expected but faster than the WatiN approach.

### SimpleBrowser

I found a project on GitHub called [SimpleBrowser][6] by Nathan Ridley which offers a headless browser testing approach written in directly in .NET. I found the interface to be easy to use and best of all the tests written with SimpleBrowser are very fast.

For example a typical SpecFlow scenario for user login:

{{< highlight "C#" >}}
    Feature: Log On
       In order to access the features of the website
       As a user
       I want to be able to log On.

    Scenario: Log On User
       Given I am on the Login Page
       When I enter my username and password
       And I click the Log On button
       Then I am on the Home Page Logged In
{{< / highlight >}}

The tooling that SpecFlow provides for Visual Studio creates a method stub for the tests steps, which can then be filled in to use the SimpleBrowser methods. For example for the first step of being on the Registration page, after filling in the test method with the SimpleBrowser calls:

{{< highlight "C#" >}}
[Given(@"I am on the Login Page")]
public void GivenIAmOnTheLoginPage() {
     WebBrowser.Current.Navigate("http://localhost:51044/Account/LogOn");
     Assert.IsTrue(WebBrowser.Current.Find("h2", FindBy.Text, "Log On").Exists);
}
{{< / highlight >}}

The method is linked to the scenario step via the attribute [Given(…)] and the body of the method uses the SimpleBrowser “Navigate” method to load the page Then the “Assert” tests that the SimpleBrowser has indeed loaded the log on page by finding an “h2” tag where its inner text is “Log On”.

The “WebBrowser.Current” is a static class that holds the SimpleClass browser instance within the SpecFlow ScenarioContext so that it can maintain state between steps, especially after a login:

{{< highlight "C#" >}}
[Binding]
public static class WebBrowser {
   public static Browser Current {
       get {
          if (!ScenarioContext.Current.ContainsKey("browser"))
             ScenarioContext.Current["browser"] = new Browser();
          return (Browser)ScenarioContext.Current["browser"];
       }
   }
}
{{< / highlight >}}

The &#8220;new Browser();” is where the SimpleBrowser Browser class is created.

Steps such as clicking on the Log On button are easy to accomplish with the Click() method:

{{< highlight "C#" >}}
[When(@"I click the Log On button")]
public void WhenIClickTheLogOnButton() {
    WebBrowser.Current.Find("input", FindBy.Value, "Log On").Click();
}
{{< / highlight >}}

There is also an overload for the Find() method that allows a shortcut if there is an “id” attribute within the html element. For example if there is are “id” elements with UserName and Password for those input elements, then the step for filling in the form fields is simple:

{{< highlight "C#" >}}
[When(@"I enter my username and password")]
public void WhenIEnterMyUsernameAndPassword() {
    WebBrowser.Current.Find("UserName").Value = "user";
    WebBrowser.Current.Find("Password").Value = "password";
}
{{< / highlight >}}

### Conclusion

I found using the SimpleBrowser easy to use and capable for most testing required for BDD testing with a “headless” browser. The resulting tests run very fast and aren’t fragile to minor changes in the HTML. The only limitation is that it doesn’t support Javascript but test that require Javascript can be done using WatiN or Selenium as part of automated integration tests rather than BDD.

****

5 thoughts on “Using SimpleBrowser with SpecFlow for BDD with ASP.NET MVC”

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}Ryan Svihla  
_January 20, 2012 at 8:23 am_
Very nice write up.

>As for SimpleBrowser, even without javascript support it will still let me make assertions against content and against anything the javascript would be consuming. I look at it as integration testing for views/json/xml responses.

****

{{< figure src="https://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_January 20, 2012 at 8:46 am_

>Thanks Ryan. Good point on checking the views created for the dynamic pages. Then for the next step you could skip to the results page by using the “Navigate()” method to go to the page by inserting the appropriate url and query params for the routing to consume.

****

{{< figure src="https://1.gravatar.com/avatar/bbb93ca63414691dd427a80c4f50d9e2?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}Karan  
_July 6, 2012 at 3:10 am_

>Thanks for the article. I tried SimpleBrowser, and its significantly faster than running Watin tests. Unfortunately, SimpleBrowser does not support Ajax calls. Is Watin the best option for running javascript code? I read around that Selenium has a headless browser that supports javascript – maybe worth having a look at?

****

{{< figure src="https://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_July 6, 2012 at 7:52 am_

>@Karan,  
Watin will definitely work OK for Javascript code, but it is much slower as it is calling through the actual browser. I think Selenium is worth a look for this purpose, I’ll take a look at it and write an article about that, thanks for the comments.

 [1]: http://specflow.org/
 [2]: http://blog.stevensanderson.com/2010/03/03/behavior-driven-development-bdd-with-specflow-and-aspnet-mvc/
 [3]: http://watin.org/
 [4]: http://blog.stevensanderson.com/2010/03/30/using-htmlunit-on-net-for-headless-browser-automation/
 [5]: http://nuget.org/packages/NHtmlUnit
 [6]: https://github.com/axefrog/SimpleBrowser
