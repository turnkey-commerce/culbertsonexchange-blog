---
title: User Initialization in Code First/Altairis
author: James
type: post
slug: user-initialization-in-code-first-altairis
date: 2011-04-27T00:46:45+00:00
aliases:
  - /permalink/p166
categories:
  - Software Development
comments: false

---
In a [previous post]({{< relref "2011-04-10-code-first-ef-4-1-with-the-altairis-membershiprole-provider.md" >}}) I showed how Entity Framework Code First ORM could be integrated with the [Altairis Web Security](https://archive.codeplex.com/?p=altairiswebsecurity) simple membership provider to create a simple way to add a membership provider with an application using the EF 4.1 framework. One of the issues is that when the database is regenerated during development the users would have to be manually re-registered.&#160; This is a hassle if you want to create an initial admin account or user account for testing.

[RWBrad asked in the comments]({{< relref "2011-04-10-code-first-ef-4-1-with-the-altairis-membershiprole-provider.md#rwbrad" >}}) if there was a way to seed an initial user in the DBInitializer routine. I did some experimenting with the Seed method and found that it can be done by calling the membership provider directly to create a new user and then adding the appropriate roles via the EF context as shown below:

{{< highlight "C#" >}}
// Create a user.
MembershipCreateStatus status = new MembershipCreateStatus();
Membership.CreateUser("admin", "password", "admin@user.com");
if (status == MembershipCreateStatus.Success) {
    // Add the role.
    User admin = context.Users.Find("admin");
    Role adminRole = context.Roles.Find("Administrator");
    admin.Roles = new List&lt;Role&gt;();
    admin.Roles.Add(adminRole);
}
{{< / highlight >}}

Then [Adam Nagle then showed that the user can also be created more directly]({{< relref "2011-04-10-code-first-ef-4-1-with-the-altairis-membershiprole-provider.md#nagle" >}}) by the context when it’s preferred to keep the data access independent of the provider assembly.

These ideas make this concept more useful by allowing the seeding of initial users without having to go to SQL to add the admin role.

The example ASP.Net MVC3 application has been updated with these changes and is available for download at Github: [Download Code][1]

 [1]: https://github.com/turnkey-commerce/CodeFirstAltairis
