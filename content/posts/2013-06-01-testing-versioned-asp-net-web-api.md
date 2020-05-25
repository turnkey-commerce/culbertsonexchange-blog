---
title: Testing Versioned API’s in ASP.Net Web API
author: James
type: post
date: 2013-06-01T18:33:38+00:00
url: /?p=345
categories:
  - Software Development
tags:
  - asp.net
  - specflow
  - unit tests
  - versioning
  - Web API

---
I recently wrote about <a href="http://www.culbertsonexchange.com/wp/?p=318" target="_blank">a framework for Versioning ASP.Net Web API</a>. Continuing on the theme of versioning API’s, I researched how to test the different versions of a Web API, both for unit tests and SpecFlow acceptance tests.

### Overview

As a quick review, the [**SDammann.WebApi.Versioning**][1] package was used for versioning the API and provides methods for selecting the controller that is active for a particular version of the API. At the code level the controller is selected by the name space of the particular version of the controller, e.g. **_Appname_.Controllers.Version1**. As mentioned in the last post, care must be taken to avoid unnecessary duplication in the controllers. Likewise the same duplication concerns apply for testing and relevant techniques will be reviewed in this post.

### SpecFlow Tests

I <a href="http://www.culbertsonexchange.com/wp/?p=293" target="_blank">previously covered using SpecFlow for testing the Web API</a>.&nbsp; When new versions of a Web API are introduced the following items should be considered:

  * Continue testing the previous versions that are still being supported.
  * Ensure that the features of the new version are being tested.

The challenge is to extend the test coverage across multiple versions of the Web API without duplicating features or steps. Fortunately SpecFlow provides a syntax to run the same scenario multiple times via a feature called “Scenario Outline”. This feature also allows the scenario to pass variables for each run of the scenario. 

#### Running a Test for Multiple Versions

The ability to run a test for both versions, V1 and V2, can be set up as follows:

<div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:4f767eb6-877e-410b-8897-c3208dffde75" class="wlWriterEditableSmartContent">
  <pre>


<pre class="brush: csharp; pad-line-numbers: true; title: ; notranslate" title="">
Scenario Outline: Create a new project saves posted values - All Versions
	Given the following project inputs and authentication
		| Field       | Value            |
		| Name        | Test Project     |
		| Description | Test Description |
		| IsUrgent    | True             |
		| UserName    | james            |
		| Password    | james2013        |
	When the client posts the inputs to the website for &lt;Version&gt;
	Then a Created status should be returned
	When the client gets the project by header location
	Then the saved project matches the inputs for &lt;Version&gt;
Scenarios: 
	| Version |
	| V1      |
	| V2      |    
</pre>
</div>


<p>
  The “Scenarios:” table at the bottom of the Scenario Outline provides a variable called <strong>Version</strong> that gets passed to the tests.&nbsp; Since there are two instances of Version in the table it will get passed twice to run the tests for each version. The <strong><Version></strong> variable within the steps will get passed to the step definition.&nbsp; For example the step definition for the first <strong>When</strong> statement above is written as follows:
</p>


<p>
  <div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:477144bd-3069-44cf-8ac7-0d3347a2b887" class="wlWriterEditableSmartContent">
    <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
[When(@&quot;the client posts the inputs to the website for V(.*)&quot;)]
public void WhenTheClientPostsTheInputsToTheWebsiteForV(int version)
{
  var client = StepHelpers.SetupHttpClient(_projectTestModel.UserName, _projectTestModel.Password);

  var postData = StepHelpers.SetPostData&lt;ProjectTestModel&gt;(_projectTestModel);
  HttpContent content = new FormUrlEncodedContent(postData);

  _responseContent = client.PostAsync(&quot;http://localhost:55301/api/v&quot; + version.ToString() + &quot;/projects&quot;, content).Result;
  client.Dispose();
}
</pre>
</div>


<p>
  The V(.*) construct within the step causes the integer part of the “Vn” to be passed to the step. Therefore for the case where “V1” is defined in the Scenario Outline then “1” will be passed via the method argument as “version”.&nbsp; Then the “version” variable will be used to call the correct version of the API from within the body of the method in client.PostAsync() for each run.
</p>


<p>
  Likewise for the second “Then” statement that validates the saved inputs, the “<Version>” can be passed so that any inputs that are not available in V1 are only checked for newer versions:
</p>


<div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:efa41e17-a758-46bf-b317-434c13245475" class="wlWriterEditableSmartContent">
  <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
[Then(@&quot;the saved project matches the inputs for V(.*)&quot;)]
public void ThenTheSavedProjectMatchesTheInputs(int version)
{ 
  Assert.AreEqual(_projectTestModel.Name, _projectSaved.Name);
  Assert.AreEqual(_projectTestModel.Description, _projectSaved.Description);  
  Assert.AreEqual(_projectTestModel.UserName, _projectSaved.UserName);
  if (version &gt; 1) 
  {
    Assert.AreEqual(_projectTestModel.IsUrgent, _projectSaved.IsUrgent);
  }
}
</pre>
</div>


<h4>
  Running a Test for a Specific Version
</h4>


<p>
  Some tests might be applicable for a specific version. For example to test that the lack of “IsUrgent” input should return an error only for V2.&nbsp; Therefore the Scenario Outline can be set up to only run V2:
</p>


<p>
  <div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:b3b609cd-1dfe-4d52-a950-59115afb4ace" class="wlWriterEditableSmartContent">
    <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
Scenario Outline: Create a new project where IsUrgent is missing returns bad request - V2
	Given the following project inputs and authentication
		| Field       | Value            |
		| Name        | Test Project     |
		| Description | Test Description |
		| UserName    | james            |
		| Password    | james2013        |
	When the client posts the inputs to the website for &lt;Version&gt;
	Then a BadRequest status should be returned
Scenarios: 
	| Version |
	| V2      |
</pre>
</div>


<p>
  This scenario will only run once for V2.&nbsp; It is still useful to put it in the scenario outline so that when V3 is created it will be easy to extend the test to run for that version by adding a V3 row to the table at the bottom.
</p>


<h4>
  Running the Scenarios in Nunit
</h4>


<p>
  When the versions are tested in this fashion in NUnit, they will be shown as nested under each Scenario Outline:
</p>


<p>
  <a href="http://www.culbertsonexchange.com/wp/wp-content/uploads/2013/06/image.png"><img style="border-bottom: 0px;border-left: 0px;padding-left: 0px;padding-right: 0px;border-top: 0px;border-right: 0px;padding-top: 0px" title="image" border="0" alt="image" src="http://www.culbertsonexchange.com/wp/wp-content/uploads/2013/06/image_thumb.png" width="542" height="181" /></a>
</p>


<p>
  The value of the variable that was passed for each run is shown and is helpful for identifying the test that was run.
</p>


<h3>
  Unit Tests
</h3>


<p>
  Unit test can be handled via a similar pattern to run the same test case multiple times by version. For example with NUnit there is an attribute called “[TestCase()]” that can be attached to a test instead of the simple “[Test]” attribute. The TestCase attribute can pass a variable to the test method, in this case it will pass “version”:
</p>


<p>
  <div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:d8c6b1c4-2a0e-4f6b-9e86-55c6d73a2ba1" class="wlWriterEditableSmartContent">
    <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
[TestCase(&quot;V1&quot;)]
[TestCase(&quot;V2&quot;)]
public void Get_Projects_Returns_Expected_Projects_For_John(string version)
{
  //Arrange
  _projectRepository = _mockRepositories.GetProjectsRepository(ReturnType.Normal);
  var projectsController = SetupController(_projectRepository.Object, &quot;john&quot;, HttpMethod.Get, version);

  //Act
  var response = projectsController.Get();
  var responseContent = JsonConvert.DeserializeObject&lt;ProjectDto&gt;(response.Content.ReadAsStringAsync().Result);

  //Assert 
  Assert.AreEqual(HttpStatusCode.OK, response.StatusCode, &quot;Expecting an OK Status Code&quot;); 
  Assert.AreEqual(1, responseContent.Projects.Count);
  Assert.AreEqual(1, responseContent.Meta.NumberProjects); 
  Assert.AreEqual(&quot;john&quot;, responseContent.Projects[0].UserName);
  Assert.AreEqual(&quot;Test Project 3&quot;, responseContent.Projects[0].Name);
}
</pre>
</div>


<p>
  The version variable is then used in the creation of the projects controller in the “Arrange” portion of the test. In the helper method that creates the controller the version variable is used as follows:
</p>


<div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:06557216-9007-464d-9f64-2ba200f87166" class="wlWriterEditableSmartContent">
  <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
private dynamic SetupController(IProjectRepository mockRepository, string userName, HttpMethod method, string version)
{  
  dynamic projectsController = null;
            
  if (version == &quot;V1&quot;)  
  {
    projectsController = new ResearchLinks.Controllers.Version1.ProjectsController(mockRepository);
  }  
  else if (version == &quot;V2&quot;) 
  {
    projectsController = new ResearchLinks.Controllers.Version2.ProjectsController(mockRepository);
  }
  // other setup code ommitted...
            
  return projectsController;
}
</pre>
</div>


<p>
  In this case the purpose is to ensure that the correct controller is constructed according to the name space partitioning by version that is used by the <strong>SDammann.WebApi.Versioning</strong> package.
</p>


<h4>
  Running Unit Tests in NUnit
</h4>


<p>
  When the NUnit tests that are decorated with the “[TestCase()]” attribute are run in the NUnit Test Runner they appear similar to the SpecFlow tests:
</p>


<p>
  <a href="http://www.culbertsonexchange.com/wp/wp-content/uploads/2013/06/image1.png"><img style="border-bottom: 0px;border-left: 0px;padding-left: 0px;padding-right: 0px;border-top: 0px;border-right: 0px;padding-top: 0px" title="image" border="0" alt="image" src="http://www.culbertsonexchange.com/wp/wp-content/uploads/2013/06/image_thumb1.png" width="521" height="204" /></a>
</p>


<p>
  As with the SpecFlow tests the version variable passed to the methods within each test case help identify when there are problems with an individual test case.
</p>


<h4>
  Summary
</h4>


<p>
  To fully test all versions of a Web API the “<strong>Scenario Outline</strong>” feature in SpecFlow and the “<strong>TestCase</strong>” feature in NUnit may be used to ensure that all versions are being tested without duplication of code or tests.
</p>


<p>
  The full source for this project can be found at <a href="https://github.com/turnkey-commerce/ResearchLinks">https://github.com/turnkey-commerce/ResearchLinks</a>.&nbsp; The unit tests are in the <a href="https://github.com/turnkey-commerce/ResearchLinks/tree/master/ResearchLinks.Tests" target="_blank">ResearchLinks.Tests</a> folder and the Specflow tests are in the <a href="https://github.com/turnkey-commerce/ResearchLinks/tree/master/ResearchLinks.SpecTests" target="_blank">ResearchLinks.SpecTests</a> folder.
</p>

 [1]: http://nuget.org/packages/SDammann.WebApi.Versioning/