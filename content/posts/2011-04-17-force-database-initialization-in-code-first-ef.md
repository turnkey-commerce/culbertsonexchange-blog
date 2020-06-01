---
title: Force Database Initialization in Code First EF
author: James
type: post
date: 2011-04-17T15:45:48+00:00
aliases:
  - /permalink/p148
categories:
  - Internet
  - Software Development
tags:
  - Entity Framework
  - MVC

---
Continuing on the [previous post]({{< relref "2011-04-10-code-first-ef-4-1-with-the-altairis-membershiprole-provider.md" >}}) about using the drop-in Altairis.Web.Security membership provider, I ran into an issue where the database may not be recreated in a timely fashion on a change to the model classes. For example if you add the following property to the User class:

{{< highlight "C#" >}}
public string FullName { get; set; }
{{< / highlight >}}

And change the initializer in the ApplicationDB.cs class to drop and create the database on model change:

{{< highlight "C#" >}}
public class DBInitializer : DropCreateDatabaseIfModelChanges&lt;ApplicationDB&gt; 
{{< / highlight >}}

The problem is if you then access the database by creating a new user in the “Account/Register” controller, the database will not drop and create because the Altairis membership provider uses non-EF (ADO.Net classic) methods of data access. Therefore, the Entity Framework access methods haven’t been touched yet and won’t drop/create the DB until that time.

The solution is to add a section to the Application_Start() method in the Global.asax:

{{< highlight "C#" >}}
protected void Application_Start() {
     AreaRegistration.RegisterAllAreas();

     // Initializes and seeds the database.
     Database.SetInitializer(new DBInitializer());

     // Forces initialization of database on model changes.
     using (var context = new ApplicationDB()) {
          context.Database.Initialize(force: true);
     }

     RegisterGlobalFilters(GlobalFilters.Filters);
     RegisterRoutes(RouteTable.Routes);
}
{{< / highlight >}}

The code in lines 8-10 forces the context to initialize the database and if there is a model change then it will drop and recreate the database on start of the application, before it can be accessed by the non-EF methods.

The example ASP.Net MVC3 application has been updated with these changes and is available for download at Github: [Download Code][1]

****

3 thoughts on “Force Database Initialization in Code First EF”

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}Ciwan  
_December 1, 2011 at 7:26 am_

>Hello James  
I have followed your steps from Previous related post and this one, but it isn’t working for me. I am getting the [following error](http://tinyurl.com/cwa34dm).  
Can you please help me resolve the error ? :( I would greatly appreciate it.  
Thank You.

****

{{< figure src="http://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_December 1, 2011 at 8:28 am_

>Hi Ciwan,  
I responded to the post that you linked with some suggestions as it looks like a possible DB permissions problem.  
-James

****

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}Dan  
_November 29, 2012 at 11:23 pm_

>Thanks James, this was a great help.

 [1]: https://github.com/turnkey-commerce/CodeFirstAltairis/
