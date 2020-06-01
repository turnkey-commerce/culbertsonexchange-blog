---
title: Testing Versioned API’s in ASP.Net Web API
author: James
type: post
date: 2013-06-01T18:33:38+00:00
aliases: 
  - /permalink/p345
categories:
  - Software Development
tags:
  - asp.net
  - specflow
  - unit tests
  - versioning
  - Web API

---
I recently wrote about [a framework for Versioning ASP.Net Web API]({{< relref "2013-05-27-versioning-asp-net-web-api.md" >}}). Continuing on the theme of versioning API’s, I researched how to test the different versions of a Web API, both for unit tests and SpecFlow acceptance tests.

### Overview

As a quick review, the [**SDammann.WebApi.Versioning**][1] package was used for versioning the API and provides methods for selecting the controller that is active for a particular version of the API. At the code level the controller is selected by the name space of the particular version of the controller, e.g. **_Appname_.Controllers.Version1**. As mentioned in the last post, care must be taken to avoid unnecessary duplication in the controllers. Likewise the same duplication concerns apply for testing and relevant techniques will be reviewed in this post.

### SpecFlow Tests

I [previously covered using SpecFlow for testing the Web API]({{< relref "2013-05-12-using-specflow-to-test-the-asp-net-web-api.md" >}}). When new versions of a Web API are introduced the following items should be considered:

* Continue testing the previous versions that are still being supported.
* Ensure that the features of the new version are being tested.

The challenge is to extend the test coverage across multiple versions of the Web API without duplicating features or steps. Fortunately SpecFlow provides a syntax to run the same scenario multiple times via a feature called “Scenario Outline”. This feature also allows the scenario to pass variables for each run of the scenario.

#### Running a Test for Multiple Versions

The ability to run a test for both versions, V1 and V2, can be set up as follows:

{{< highlight "C#" >}}
Scenario Outline: Create a new project saves posted values - All Versions
	Given the following project inputs and authentication
		| Field       | Value            |
		| Name        | Test Project     |
		| Description | Test Description |
		| IsUrgent    | True             |
		| UserName    | james            |
		| Password    | james2013        |
	When the client posts the inputs to the website for <Version>
	Then a Created status should be returned
	When the client gets the project by header location
	Then the saved project matches the inputs for <Version>
Scenarios: 
	| Version |
	| V1      |
	| V2      |    
{{< / highlight >}}


The “Scenarios:” table at the bottom of the Scenario Outline provides a variable called **Version** that gets passed to the tests. Since there are two instances of Version in the table it will get passed twice to run the tests for each version. The **<Version>** variable within the steps will get passed to the step definition. For example the step definition for the first **When** statement above is written as follows:


{{< highlight "C#" >}}
[When(@"the client posts the inputs to the website for V(.*)")]
public void WhenTheClientPostsTheInputsToTheWebsiteForV(int version)
{
  var client = StepHelpers.SetupHttpClient(_projectTestModel.UserName, _projectTestModel.Password);

  var postData = StepHelpers.SetPostData<ProjectTestModel>(_projectTestModel);
  HttpContent content = new FormUrlEncodedContent(postData);

  _responseContent = client.PostAsync("http://localhost:55301/api/v" + version.ToString() + "/projects", content).Result;
  client.Dispose();
}
{{< / highlight >}}

The V(.*) construct within the step causes the integer part of the “Vn” to be passed to the step. Therefore for the case where “V1” is defined in the Scenario Outline then “1” will be passed via the method argument as “version”. Then the “version” variable will be used to call the correct version of the API from within the body of the method in client.PostAsync() for each run.

Likewise for the second “Then” statement that validates the saved inputs, the “<Version>” can be passed so that any inputs that are not available in V1 are only checked for newer versions:

{{< highlight "C#" >}}
[Then(@"the saved project matches the inputs for V(.*)")]
public void ThenTheSavedProjectMatchesTheInputs(int version)
{ 
  Assert.AreEqual(_projectTestModel.Name, _projectSaved.Name);
  Assert.AreEqual(_projectTestModel.Description, _projectSaved.Description);  
  Assert.AreEqual(_projectTestModel.UserName, _projectSaved.UserName);
  if (version > 1) 
  {
    Assert.AreEqual(_projectTestModel.IsUrgent, _projectSaved.IsUrgent);
  }
}
{{< / highlight >}}


#### Running a Test for a Specific Version


Some tests might be applicable for a specific version. For example to test that the lack of “IsUrgent” input should return an error only for V2. Therefore the Scenario Outline can be set up to only run V2:

{{< highlight "C#" >}}
Scenario Outline: Create a new project where IsUrgent is missing returns bad request - V2
	Given the following project inputs and authentication
		| Field       | Value            |
		| Name        | Test Project     |
		| Description | Test Description |
		| UserName    | james            |
		| Password    | james2013        |
	When the client posts the inputs to the website for <Version>
	Then a BadRequest status should be returned
Scenarios: 
	| Version |
	| V2      |
{{< / highlight >}}

This scenario will only run once for V2. It is still useful to put it in the scenario outline so that when V3 is created it will be easy to extend the test to run for that version by adding a V3 row to the table at the bottom.

 #### Running the Scenarios in Nunit

When the versions are tested in this fashion in NUnit, they will be shown as nested under each Scenario Outline:

![code example](http://www.culbertsonexchange.com/wp/wp-content/uploads/2013/06/image.png)

The value of the variable that was passed for each run is shown and is helpful for identifying the test that was run.

#### Unit Tests

Unit test can be handled via a similar pattern to run the same test case multiple times by version. For example with NUnit there is an attribute called “[TestCase()]” that can be attached to a test instead of the simple “[Test]” attribute. The TestCase attribute can pass a variable to the test method, in this case it will pass “version”:

{{< highlight "C#" >}}
[TestCase("V1")]
[TestCase("V2")]
public void Get_Projects_Returns_Expected_Projects_For_John(string version)
{
  //Arrange
  _projectRepository = _mockRepositories.GetProjectsRepository(ReturnType.Normal);
  var projectsController = SetupController(_projectRepository.Object, "john", HttpMethod.Get, version);

  //Act
  var response = projectsController.Get();
  var responseContent = JsonConvert.DeserializeObject<ProjectDto>(response.Content.ReadAsStringAsync().Result);

  //Assert 
  Assert.AreEqual(HttpStatusCode.OK, response.StatusCode, "Expecting an OK Status Code"); 
  Assert.AreEqual(1, responseContent.Projects.Count);
  Assert.AreEqual(1, responseContent.Meta.NumberProjects); 
  Assert.AreEqual("john", responseContent.Projects[0].UserName);
  Assert.AreEqual("Test Project 3", responseContent.Projects[0].Name);
}
{{< / highlight >}}

The version variable is then used in the creation of the projects controller in the “Arrange” portion of the test. In the helper method that creates the controller the version variable is used as follows:


{{< highlight "C#" >}}
private dynamic SetupController(IProjectRepository mockRepository, string userName, HttpMethod method, string version)
{  
  dynamic projectsController = null;
            
  if (version == "V1")  
  {
    projectsController = new ResearchLinks.Controllers.Version1.ProjectsController(mockRepository);
  }  
  else if (version == "V2") 
  {
    projectsController = new ResearchLinks.Controllers.Version2.ProjectsController(mockRepository);
  }
  // other setup code ommitted...
            
  return projectsController;
}
{{< / highlight >}}

In this case the purpose is to ensure that the correct controller is constructed according to the name space partitioning by version that is used by the **SDammann.WebApi.Versioning** package.


#### Running Unit Tests in NUnit

When the NUnit tests that are decorated with the “[TestCase()]” attribute are run in the NUnit Test Runner they appear similar to the SpecFlow tests:

![Code Example](http://www.culbertsonexchange.com/wp/wp-content/uploads/2013/06/image1.png)

As with the SpecFlow tests the version variable passed to the methods within each test case help identify when there are problems with an individual test case.

#### Summary

To fully test all versions of a Web API the “**Scenario Outline**” feature in SpecFlow and the “**TestCase**” feature in NUnit may be used to ensure that all versions are being tested without duplication of code or tests.

The full source for this project can be found at https://github.com/turnkey-commerce/ResearchLinks. The unit tests are in the https://github.com/turnkey-commerce/ResearchLinks/tree/master/ResearchLinks.Tests folder and the Specflow tests are in the https://github.com/turnkey-commerce/ResearchLinks/tree/master/ResearchLinks.SpecTests folder.

 [1]: http://nuget.org/packages/SDammann.WebApi.Versioning/