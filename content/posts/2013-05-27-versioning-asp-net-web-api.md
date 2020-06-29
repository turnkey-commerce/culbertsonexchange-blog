---
title: Versioning ASP.Net Web API
author: James
type: post
date: 2013-05-28T03:10:11+00:00
aliases:
  - /permalink/p318
categories:
  - Software Development
tags:
  - asp.net
  - versioning
  - Web API
comments: false

---
One of the important things to consider when building an API is a strategy for versioning the API to manage changes. There are several reasons this is important:

* To support users (developers) of the API that are using an existing version so as not to force breaking changes on them. 
* To prevent breaking existing versions of the client applications that are using an existing version of the API.

Previous versions of the API will normally be maintained for either a period to allow developers and client applications to upgrade or indefinitely.

### Strategies for Versioning

There are multiple ways to indicate the version of the API:
  
* Specify the version as part of the URI address or URL query parameters, e.g.  
  > http[]()://www.example.com/api/**v1**/project  
  > http[]()://www.example.com/api/projects?**version=1**  
* Specify the version as part of the Accept request header, for example:  
  > Accept: application/vnd.company.myapp.customer-**v3**+xml
* Specify the version as a special request header, for example:  
  > **X-API-VERSION: 3**  

There are pros and cons of each approach, and have been [extensively discussed on the Internet](http://www.lexicalscope.com/blog/2012/03/12/how-are-rest-apis-versioned/) with religious fervor. I recently attended a [nice talk by Michael Pratt on how to version or not version your API][1] (be sure to use down arrows for pros and cons for each strategy). The best approach is to pick a strategy that best fits your use cases and to stick with it for consistency.

### Adding Versioning to ASP.Net Web API

There are no versioning capabilities provided by the ASP.Net Web API out of the box. However there is a nice Nuget package called [SDammann.WebApi.Versioning](http://nuget.org/packages/SDammann.WebApi.Versioning/) by [Sebastiaan Dammann][2] that extends the Web API with a versioning framework. The package is flexible in that it allows the API developer to choose one of the strategies mentioned above for the API versioning. This article will show how to set up the framework on a project.

#### Setting Up the Project

The first step is to get the WebApi.Versioning from Nuget by bringing up the Package Manager Console and getting the Nuget package:

{{< highlight "C#" >}}
  PM> Install-Package SDammann.WebApi.Versioning`
{{< / highlight >}}

In this example we are going to use the the URI address strategy to indicate the versioning. 

The next step is to indicate in the Global.Asax.cs\Application_Start method to use the versioning package for the controller selector and to provide the strategy to use for the controller selection:

{{< highlight "C#" >}}
// enable API versioning
    GlobalConfiguration.Configuration.Services.Replace(typeof(IHttpControllerSelector),
      new RouteVersionedControllerSelector(GlobalConfiguration.Configuration));
{{< / highlight >}}

In this case the **new RouteVersionControllerSelector(…)** indicates to the package which strategy to use (in this case selecting by routes in the URI). For that to work the default route (and likewise any custom routes) must be modified to include the version in *App_Start\WebApiConfig.cs*:

{{< highlight "C#" >}}
  config.Routes.MapHttpRoute(
      name: "DefaultApi",
      routeTemplate: "api/v{version}/{controller}/{id}",
      defaults: new { id = RouteParameter.Optional }
  );
{{< / highlight >}}

The **{version}** integer variable after the “v” will be utilized by the package to determine which controller to use based on the value. For example “v1” will indicate to use the controllers that are in the namespace **Appname.Controllers.Version1**.

#### Managing Controllers to Avoid Duplication

One of the downsides of this approach is that it could cause a lot of code duplication if the entire controllers are copied to a new namespace each time a version is added. One approach to reduce besides keeping the controllers “skinny” is to add a base class for each controller and implement overrides for the version where needed. For example the Controllers folder would look something like this:

![Code example](/uploads/2013/05/image.png)

The base controllers would contain the initial implementation for version 1 with virtual methods:

{{< highlight "C#" >}}
namespace ResearchLinks.Controllers
{
    [Authorize]
    public class ProjectResearchItemsControllerBase : ApiController
    {
        protected IResearchItemRepository _researchItemRepository;
        protected IProjectRepository _projectRepository;

        // GET /api/projects/4/researchItems
        public virtual HttpResponseMessage Get(int projectId)
        {
            var researchItems = new List<ResearchItem>();
            var project = new Project();
            // Rest of implementation ommitted...
        }
...
{{< / highlight >}}

Then for the Version1 implementation it would not need to override any methods but only provide services such as dependency injection and to expose the methods to the Version1 class:

{{< highlight "C#" >}}
namespace ResearchLinks.Controllers.Version1
{
    [Authorize]
    public class ProjectsController : ProjectsControllerBase
    {
        // Constructor here to inject dependencies in concrete class.
        public ProjectsController(IProjectRepository projectRepository)
        {
            _projectRepository = projectRepository;
        }

        // Nothing to override in Version1
    }
}
{{< / highlight >}}


Then for Version2 we can override the methods that will be affected by a change and ignore the other methods that will be inherited from the base class. For example say we need to add a new Boolean property to the Project model called “IsUrgent”. We don’t want it to affect Version1 users but we do want it to be mandatory for Version2. We can add this property to the model as a nullable Boolean property and not add any Required attributes so as not to disturb Version1 usage:

{{< highlight "C#" >}}
    public class Project
    {
        [Key]
        public int ProjectId { get; set; }

        [Required, MaxLength(50)]
        public string Name { get; set; }

        // Added for version 2 and higher
        public bool? IsUrgent { get; set; }

        // Other properties omitted...

    }
{{< / highlight >}}


This change will affect the “Post” and “Put” verbs for the Projects controller and we will need to override those methods.For example to override the “Post” method and to ensure the required flag is present we can do the following:

{{< highlight "C#" >}}
namespace ResearchLinks.Controllers.Version2
{
    [Authorize]
    public class ProjectsController : ProjectsControllerBase
    {
    
        //VERSION 2!

        // In version 2 we need to add an additional bool property IsUrgent on the post and put.
        // We will catch it in the controller since it is not required in the data model for backward compatibility in V1.

        // Constructor here to inject dependencies in concrete class.
        public ProjectsController(IProjectRepository projectRepository)
        {
            _projectRepository = projectRepository;
        }

        // POST api/v2/projects (Insert)
        public override HttpResponseMessage Post(Project project)
        {
            // Check for presence of V2 "IsUrgent" flag.
            if (project.IsUrgent == null)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, "The IsUrgent indicator is required.");
            }
           // Rest of implementation ommitted...
        }
{{< / highlight >}}

### Conclusion

The [**SDammann.WebApi.Versioning**](http://nuget.org/packages/SDammann.WebApi.Versioning) package provides an easy and flexible framework for versioning ASP.Net Web API. It covers the most common strategies to version a web API and could be extended to support less common methods as it is open source.

In the next blog post I will discuss strategies for testing the versioning of the Web API.

The full source for this project can be found at https://github.com/turnkey-commerce/ResearchLinks.

 [1]: http://slid.es/michaelpratt/how-to-not-version-your-api
 [2]: http://nuget.org/packages?q=Author%3A%22Sebastiaan%20Dammann%22