---
title: Using SpecFlow to Test the ASP.Net Web API
author: James
type: post
date: 2013-05-13T01:39:00+00:00
url: /?p=293
categories:
  - Software Development
tags:
  - asp.net
  - specflow
  - Web API

---
[I previously wrote about the use of SpecFlow][1] in an ASP.Net MVC application with the SimpleBrowser library. I’ve recently been building <a href="https://github.com/turnkey-commerce/ResearchLinks" target="_blank">a project</a> to utilize the ASP.Net Web API to study the best practices of building a “RESTful” API for an application to save research notes and links about multiple projects.

### Approach

Since the Web API is intended to be utilized by a number of types of client applications that support the HTTP protocol (smart phones, tablets, desktop applications, as well as browsers), it is convenient to use a library that is well-suited to use the HTTP verbs used by the Web API. In this case I used the <a href="http://msdn.microsoft.com/en-us/library/system.net.http.httpclient.aspx" target="_blank">HTTPClient</a> class for interacting with the Web API. 

For example in the case of posting a new project, and the client has been setup with the proper post parameters, then the client can post to the Web API as follows:

<pre>_responseContent = client.PostAsync("http://localhost:55301/api/projects", content).Result;</pre>

The “.Result” property causes the results to be returned to the _responseContent synchronously, which is desired for the tests.

Similarly to test an update (“put” verb) then the client can post to the Web API as follows:

<pre>_responseContent = client.PutAsync("http://localhost:55301/api/projects/" + _projectSaved.ProjectId, content).Result;</pre>

### Defining SpecFlow Feature Files

The feature files are defined to exercise the various features of the API, for example the post verb to create a new project:

<font face="Consolas"><font color="#008000"><strong>&nbsp; Feature</strong></font>: Projects API<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; In order to perform CRUD operations on the projects<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; As a client of the Web Api<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; I want to be able to Create, Update, Delete, and List projects</font>

<font face="Consolas"><strong><font color="#008000">&nbsp; Scenario</font></strong>: Create a new project saves posted values.<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <strong><font color="#008000">Given</font></strong> the following project inputs and authentication<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Field&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Value&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Test Project&nbsp;&nbsp;&nbsp;&nbsp; |<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Description | Test Description |<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | UserName&nbsp;&nbsp;&nbsp; | james&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Password&nbsp;&nbsp;&nbsp; | james2013&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <font color="#008000"><strong>When</strong></font> the client posts the inputs to the website<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <strong><font color="#008000">Then</font></strong> a Created status should be returned<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <strong><font color="#008000">When</font></strong> the client gets the project by header location<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <strong><font color="#008000">Then</font></strong> the saved project matches the inputs</font>

This scenario gathers the inputs, then posts the inputs to the Web API. It then checks that a “Created” status is returned. Then from the “location” header returned, the client will use that information to get the project that was saved and than verify that it matches the inputs.

Similar tests are setup for the other verbs for the Project API.

### Defining Step Details

For each of the steps in the scenario a corresponding C# step file will be defined. The details for each step will be discussed below.

#### **<font color="#008000">Given</font>** the following project inputs and authentication

This step takes the Fields and Values defined in the scenario and uses a SpecFlow helper “FillInsance” to load the values into a “ProjectTestModel” DTO class that was defined for holding values associated with the tests:

<div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:0cb918a7-086d-449b-992d-150f8f918cec" class="wlWriterEditableSmartContent">
  <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
[Given(@"the following project inputs and authentication")]
public void GivenTheFollowingProjectInputsAndAuthentication(Table table)
{
     table.FillInstance&lt;ProjectTestModel&gt;(_projectTestModel);
}
</pre>
</div>


<p>
  This properties of the _projectTestModel member variables will be used downstream in the other steps.
</p>


<h4>
  <font color="#008000"><strong>When</strong></font> the client posts the inputs to the website
</h4>


<p>
  This step takes the inputs loaded in the previous step and uses them to post to the Web API to create a new project:
</p>


<div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:1657eb33-9d03-40ae-92a7-6802f2561448" class="wlWriterEditableSmartContent">
  <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
[When(@"the client posts the inputs to the website")]
public void WhenTheClientPostsTheInputsToTheWebsite()
{

     var client = StepHelpers.SetupHttpClient(_projectTestModel.UserName, _projectTestModel.Password);

     var postData = StepHelpers.SetPostData&lt;ProjectTestModel&gt;(_projectTestModel);
            HttpContent content = new FormUrlEncodedContent(postData);

     _responseContent = client.PostAsync("http://localhost:55301/api/projects", content).Result;
            client.Dispose();
}
</pre>
</div>


<p>
  This step uses two custom helpers to set up the HttpClient with the authentication header and to set up the post data from the ProjectTestModel DTO. These helpers reduce the amount of repetitive code in the steps. Finally the client posts the data to the Web API and captures the response.
</p>


<h4>
  <strong><font color="#008000">Then</font></strong> a Created status should be returned
</h4>


<p>
  This step does an assert to check that the expected Status code is returned in the response:
</p>


<p>
  <div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:4e440753-bda3-4055-abf9-4180a1f33aee" class="wlWriterEditableSmartContent">
    <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
[Then(@"a (.*) status should be returned")]
public void ThenAStatusShouldBeReturned(string statusCode)
{
     Assert.AreEqual(statusCode, _responseContent.StatusCode.ToString());
}
</pre>
</div>


<p>
  This step is coded in a way that the status code that is expected is passed in the step definition with the (.*) wildcard and passed to the code as “statusCode”. Then it compares the expected statusCode to the one that is returned in the _responseContent returned from the previous step.
</p>


<h4>
  <strong><font color="#008000">When</font></strong> the client gets the project by header location
</h4>


<p>
  This step gets the URL path for the new project by examining the “Location” header parameter that was returned by the response.&nbsp; This will give the path and project ID needed to get the created project:
</p>


<div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:ad00fac3-337b-456e-8d32-276f72feba4c" class="wlWriterEditableSmartContent">
  <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
[When(@"the client gets the project by header location")]
public void WhenTheClientGetsTheProjectByHeaderLocation()
{
      var client = StepHelpers.SetupHttpClient(_projectTestModel.UserName, _projectTestModel.Password);
      _responseContent = client.GetAsync(_responseContent.Headers.Location).Result;
      _projectSaved = JsonConvert.DeserializeObject&lt;Project&gt;(_responseContent.Content.ReadAsStringAsync().Result);
            client.Dispose();
}
</pre>
</div>


<p>
  This step does a setup of a new HttpClient similar to the post. However, in this case it does a “get” verb using the URL that was returned in the “Location” header. The JSON.Net library is then used to deserialize the response into the “_projectSaved” Project object so that it can be used to compare to the expected values.
</p>


<h4>
  <strong><font color="#008000">Then</font></strong> the saved project matches the inputs
</h4>


<p>
  This step will then do asserts to compare the properties of the inputs in the “_projectTestModel” DTO to the “_projectSaved” obtained from the previous step:
</p>


<div style="padding-bottom: 0px;margin: 0px;padding-left: 0px;padding-right: 0px;float: none;padding-top: 0px" id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:670c4d4d-72f0-4c3a-9708-b9d045ec35df" class="wlWriterEditableSmartContent">
  <pre>


<pre class="brush: csharp; title: ; notranslate" title="">
[Then(@"the saved project matches the inputs")]
public void ThenTheSavedProjectMatchesTheInputs()
{
     Assert.AreEqual(_projectTestModel.Name, _projectSaved.Name);
     Assert.AreEqual(_projectTestModel.Description, _projectSaved.Description);
     Assert.AreEqual(_projectTestModel.UserName, _projectSaved.UserName);
}
</pre>
</div>


<h3>
  Other Scenarios
</h3>


<p>
  Other scenarios were set up to complete the testing of the Project API to test for all of the verbs (“get”, “put”, “post”, and “delete”) as well as testing for aspects such as invalid authorization or testing that users may not access or modify other user’s projects. These scenarios are defined in the following file: <a href="https://github.com/turnkey-commerce/ResearchLinks/blob/master/ResearchLinks.SpecTests/ProjectsApi.feature" target="_blank">ProjectsApi.feature</a>.
</p>


<h4>
  Other Testing Considerations
</h4>


<p>
  It’s important that the tests are run with a clean environment so that they don’t depend on other tests.&nbsp; Therefore a “[BeforeScenario]” method is used to ensure that the projects are removed prior to each test, in the following file: <a href="https://github.com/turnkey-commerce/ResearchLinks/blob/master/ResearchLinks.SpecTests/Helpers/DatabaseHelpers.cs" target="_blank">DatabaseHelpers.cs</a>.
</p>


<p>
  Also to ensure that the web server is available for the test run, the IISExpress module is started when the tests are run via the “[SetUp]” and “[TearDown]” attributes on methods that start and shutdown IISExpress. This setup and teardown is found in the following file: <a href="https://github.com/turnkey-commerce/ResearchLinks/blob/master/ResearchLinks.SpecTests/Startup.cs" target="_blank">Startup.cs</a>.
</p>


<p>
  The SpecFlow code should be set up in a separate project within the solution, also separate from unit tests.&nbsp; This allows it to be omitted from some deployments. For example this is useful for AppHarbor deployments where these type of tests won’t run within the production environment. 
</p>


<h4>
  Full Source code
</h4>


<p>
  The full source code for this project can be found at the <a href="https://github.com/turnkey-commerce/ResearchLinks" target="_blank">ResearchLinks</a> solution on GitHub and the code related to SpecFlow is contained in the <a href="https://github.com/turnkey-commerce/ResearchLinks/tree/master/ResearchLinks.SpecTests" target="_blank">ResearchLinks.SpecTests</a> sub-project.
</p>


<p>
  &nbsp;
</p>


<p>
  <strong><font color="#008000" face="Consolas"></font></strong>
</p>

 [1]: http://www.culbertsonexchange.com/wp/?p=249