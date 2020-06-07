---
title: Code First EF 4.1 with the Altairis Membership/Role Provider
author: James
type: post
date: 2011-04-10T20:48:14+00:00
aliases:
  - /permalink/p120
categories:
  - Internet
  - Software Development
tags:
  - Entity Framework
  - MVC

---
### Problem with Current Membership/Role Provider

I’ve always thought the default membership/role provider for ASP.Net is a bit heavy in that it is targeted by default to a different database than the main application database and takes several additional steps to set up and deploy. I found the [Altairis Web Security Toolkit](http://altairiswebsecurity.codeplex.com/) on CodePlex and it has a nice, simple schema that is easy to integrate that into your application database. It is also available from Nuget as [Altairis.Web.Security](http://www.nuget.org/List/Packages/Altairis.Web.Security).

### Working with the EF Magic Unicorn Edition

![1]The new Entity Framework Code First, also known as the [EF Magic Unicorn Edition](http://www.hanselman.com/blog/SimpleCodeFirstWithEntityFramework4MagicUnicornFeatureCTP4.aspx)” provides a nice way to generate the database and the ORM from POCO (Plain Old CLR Objects) classes that you write up front. This gives a nice “[code first](http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx)” approach where the domain objects drive the database rather than vice-versa (also known as “persistence ignorance”).

With the current edition of Magic Unicorn the database is constantly being regenerated during development, which interrupts the workflow to do the additional step of adding the membership/role tables. And even if that’s not too much trouble the membership/role tables aren’t accessible to the code first EF context for access by the application.

### Code First Integration with the Drop-in Altairis Provider

One solution, besides writing a provider from scratch, is to create code first classes that generate the database schema that is expected by the drop-in membership provider.&#160; That way the membership provider still works as expected without change, and the user/role classes are available to the application for doing administration and easily adding additional columns for the User table.

To implement this I did the following steps for an MVC3 project:

  1. Start a new MVC3 web application project.
  2. Use the NuGet console to download the following packages:
    Altairis.Web.Security
    EntityFramework
  3. Create User and Role classes and include them in a class that defines the EF context. In the context class I did a few special things to make sure that the many-to-many table for the roles was generated correctly:

{{< highlight "C#" >}}
public class ApplicationDB : DbContext
{
    public DbSet&lt;User&gt; Users { get; set; }
    public DbSet&lt;Role&gt; Roles { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder) {
        // Maps to the expected many-to-many join table name for roles to users.
        modelBuilder.Entity&lt;User&gt;()
        .HasMany(u =&gt; u.Roles)
        .WithMany(r =&gt; r.Users)
        .Map(m =&gt;
        {
            m.ToTable("RoleMemberships");
            m.MapLeftKey("UserName");
            m.MapRightKey("RoleName");
        });
    }
}
{{< / highlight >}}

Data may also be seeded into the initial database, for example to create some default roles. The commented out class definition can be utilized during development to recreate the database if the model changes:

{{< highlight "C#" >}}
// Change the base class as follows if you want to drop and create the database during development:
// public class DBInitializer : DropCreateDatabaseIfModelChanges&lt;ApplicationDB&gt;
public class DBInitializer : CreateDatabaseIfNotExists&lt;ApplicationDB&gt;
{
    protected override void Seed(ApplicationDB context)
    {
        var roles = new List&lt;Role&gt;{
            new Role{RoleName = "Administrator"},
            new Role{RoleName = "User"},
            new Role{RoleName = "PowerUser"}
        };

        roles.ForEach(r =&gt; context.Roles.Add(r));
    }
}
{{< / highlight >}}

The classes themselves are straightforward and can be decorated with attributes that integrate well with MVC:

{{< highlight "C#" >}}
public class User {
    [Key]
    [Required(ErrorMessage = "User Name is required")]
    [Display(Name="User Name")]
    [MaxLength(100)]
    public string UserName { get; set; }

    [Required]
    [MaxLength(64)]
    public byte[] PasswordHash { get; set; }

    [Required]
    [MaxLength(128)]
    public byte[] PasswordSalt { get; set; }

    [Required(ErrorMessage = "Email is required")]
    [MaxLength(200)]
    public string Email { get; set; }

    [MaxLength(200)]
    public string Comment { get; set; }

    [Display(Name = "Approved?")]
    public bool IsApproved { get; set; }

    [Display(Name = "Crate Date")]
    public DateTime DateCreated { get; set; }

    [Display(Name = "Last Login Date")]
    public DateTime? DateLastLogin { get; set; }

    [Display(Name = "Last Activity Date")]
    public DateTime? DateLastActivity { get; set; }

    [Display(Name = "Last Password Change Date")]
    public DateTime DateLastPasswordChange { get; set; }

    public virtual ICollection&lt;Role&gt; Roles { get; set; }

}
{{< / highlight >}}

{{< highlight "C#" >}}
public class Role {
    [Key]
    [Display(Name = "Role Name")]
    [Required(ErrorMessage = "Role Name is required")]
    [MaxLength(100)]
    public string RoleName { get; set; }

    public virtual ICollection&lt;User&gt; Users { get; set; }
}
{{< / highlight >}}

Additional properties could be added to the User class and they wouldn’t interfere with the operation of the membership provider as long a they are not “Required” properties.

I wrote an example ASP.Net MVC3 application and it is available for download at Github: [Download Code](https://github.com/turnkey-commerce/CodeFirstAltairis)

**Update:** I found an issue with database initialization if the model is changed and the EF data access is not initialized before the classic ADO.Net access provided by Altairis. See [this post][2] for more information.

****

14 thoughts on “Code First EF 4.1 with the Altairis Membership/Role Provider”

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}raj  
_April 14, 2011 at 12:56 am_

>How to identify the User table from the Membership provider?

****

{{< figure src="http://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_April 14, 2011 at 7:10 am_

>The membership provider in this example (Altairis) will automatically identify the User table as that’s how it has been defined. It’s possible that a different provider will look for a different table and columns so please note that this one is specific to Altairis. The Membership provider itself is defined in the web.config (see download code).

****
<!-- markdownlint-disable MD033 -->
<span id="rwbrad"><span>{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}RWBrad  
_April 24, 2011 at 9:51 pm_
<!-- markdownlint-enable MD033 -->
>How would you create some initial user in the DBInitializer routine?

****

{{< figure src="http://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_April 24, 2011 at 10:02 pm_

>@RWBrad: That’s a great question and it is definitely a chicken/egg thing as you’d like to create a user with an admin role to get started. I was working on a related issue on the follow-up post on how to force the DB re-creation before the membership provider is accessed as it can cause an error otherwise. It could be done by calling the membership provider directly in that method. I’ll try that and do a follow-up on it.  
>**EDIT:** I did find that you could do this in the “Seed” class for the initializer. Here is an example:

{{< highlight "C#" >}}
MembershipCreateStatus status = new MembershipCreateStatus();
Membership.CreateUser(“admin”, “password”, “admin@user.com”);
if (status == MembershipCreateStatus.Success) {
    // Add the role to it.
    User user = context.Users.Find(“admin”);
    Role role = context.Roles.Find(“Administrator”);
    user.Roles = new List();
    user.Roles.Add(role);
}
{{< / highlight >}}

****
<!-- markdownlint-disable MD033 -->
<span id="nagle"><span>{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}Adam Nagle  
_April 26, 2011 at 8:29 am_
<!-- markdownlint-enable MD033 -->
>I dug into the Altairis TableMembershipProvider Source Code and found another way to Seed user data. I needed to do it this way because my initializer lives down in my data access layer.

{{< highlight "C#" >}}
using (var hmac = new System.Security.Cryptography.HMACSHA512())
{
    passwordSalt = hmac.Key;
    passwordHash = hmac.ComputeHash(System.Text.Encoding.UTF8.GetBytes(password));

    var users = new List
    {
        new User
        { 
            UserName = “testuser”, Email = “testuser@gmail.com”,
            PasswordHash = passwordHash,
            PasswordSalt = passwordSalt,
            IsApproved = true,
            DateCreated = DateTime.Now,
            DateLastPasswordChange = DateTime.Now,
            Roles = context.Roles.Where(x=>x.RoleName == “User”).ToList()
        }
    };
    users.ForEach(x => context.Users.Add(x));
{{< / highlight >}}

****

{{< figure src="http://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_April 26, 2011 at 8:36 am_

>@Adam,  
Thanks for sharing that more direct way to create the user!

****

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}dc  
_May 11, 2011 at 4:49 am_

>Can we have add extra properties with Required attribute onto the User entity?

****

{{< figure src="http://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_May 12, 2011 at 8:45 pm_

>@dc: Not on the User entity because it would cause the Membership provider to break on the create method. However, a workaround could be to add the properties as not required and then use a ViewModel for the controller/view to make the property required with an attribute in the ViewModel for the purpose of validation in the view. The ViewModel could then be mapped to the User Entity in the controller. There are tools such as Automapper that can help automate that mapping.

****

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}Michiel  
_August 3, 2011 at 8:23 am_

>Great code. I made a NuGet package based on this code, check it out here:

[http://www.nuget.org/List/Packages/quickstart.mvc3.unity.ef.altairiswebsecurity](http://www.nuget.org/List/Packages/quickstart.mvc3.unity.ef.altairiswebsecurity)

****

{{< figure src="http://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_August 3, 2011 at 1:31 pm_

>@Michiel, Thanks for packaging that up, looks very cool! Great to have nice packages like that to start up a new project.

****

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}Stuart  
_November 3, 2011 at 1:07 am_

>Adding role(s) to user(s) when seeding the DB…. stuck on this one for a bit.  
Just what I was after:  
user.Roles = new List();  
Thanks.

****

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}Tim  
_December 31, 2011 at 3:13 pm_

>finally i found your post. it does help me a lot, thanks a lot, but i have one more question? how to init the database with .mdf format instead of scf. i played around with this few hours, can’t figure out, please help me.

****

{{< figure src="/images/user.png" alt="Commenter" class="commenter">}}Tim  
_December 31, 2011 at 3:56 pm_
>dudu, i figured out myself, thank ya

****

{{< figure src="http://1.gravatar.com/avatar/7d7f3a3ae79c647242de191255ce6a36?s=44&d=http%3A%2F%2F1.gravatar.com%2Favatar%2Fad516503a11cd5ca435acc9bb6523536%3Fs%3D44&r=G" alt="Commenter" class="commenter">}}James Culbertson  
_December 31, 2011 at 7:55 pm_

>Glad you got it figured out OK. You can control the database it creates or connects to in the connection string.

 [1]: /uploads/2011/04/unicorn07.gif
 [2]: {{< relref "2011-04-17-force-database-initialization-in-code-first-ef.md" >}}
