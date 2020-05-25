---
title: Code First EF 4.1 with the Altairis Membership/Role Provider
author: James
type: post
date: 2011-04-10T20:48:14+00:00
url: /?p=120
categories:
  - Internet
  - Software Development
tags:
  - Entity Framework
  - MVC

---
### Problem with Current Membership/Role Provider

I’ve always thought the default membership/role provider for ASP.Net is a bit heavy in that it is targeted by default to a different database than the main application database and takes several additional steps to set up and deploy. I found the <a href="http://altairiswebsecurity.codeplex.com/" target="_blank">Altairis Web Security Toolkit</a> on CodePlex and it has a nice, simple schema that is easy to integrate that into your application database. It is also available from Nuget as _<a href="http://www.nuget.org/List/Packages/Altairis.Web.Security" target="_blank">Altairis.Web.Security</a>_. 

### Working with the EF Magic Unicorn Edition

[<img style="border-right-width: 0px;margin: 3px 10px 3px 0px;padding-left: 0px;padding-right: 0px;border-top-width: 0px;border-bottom-width: 0px;border-left-width: 0px;padding-top: 0px" border="0" alt="unicorn07" src="http://www.culbertsonexchange.com/wp/wp-content/uploads/2011/04/unicorn07_thumb.gif" width="80" height="72" />][1]The new Entity Framework Code First, also known as the “<a href="http://www.hanselman.com/blog/SimpleCodeFirstWithEntityFramework4MagicUnicornFeatureCTP4.aspx" target="_blank">EF Magic Unicorn Edition</a>” provides a nice way to generate the database and the ORM from POCO (Plain Old CLR Objects) classes that you write up front. This gives a nice “<a href="http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx" target="_blank">code first</a>” approach where the domain objects drive the database rather than vice-versa (also known as “persistence ignorance”). 

With the current edition of Magic Unicorn the database is constantly being regenerated during development, which interrupts the workflow to do the additional step of adding the membership/role tables. And even if that’s not too much trouble the membership/role tables aren’t accessible to the code first EF context for access by the application.

### Code First Integration with the Drop-in Altairis Provider

One solution, besides writing a provider from scratch, is to create code first classes that generate the database schema that is expected by the drop-in membership provider.&#160; That way the membership provider still works as expected without change, and the user/role classes are available to the application for doing administration and easily adding additional columns for the User table.

To implement this I did the following steps for an MVC3 project:

  1. Start a new MVC3 web application project. 
  2. Use the NuGet console to download the following packages:   
    Altairis.Web.Security   
    EntityFramework 
  3. Create User and Role classes and include them in a class that defines the EF context. In the context class I did a few special things to make sure that the many-to-many table for the roles was generated correctly: 

<pre class="brush: csharp; title: ; notranslate" title="">public class ApplicationDB : DbContext
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
</pre>

Data may also be seeded into the initial database, for example to create some default roles. The commented out class definition can be utilized during development to recreate the database if the model changes: 

<pre class="brush: csharp; title: ; notranslate" title="">// Change the base class as follows if you want to drop and create the database during development:
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
</pre>

The classes themselves are straightforward and can be decorated with attributes that integrate well with MVC:

<pre class="brush: csharp; title: ; notranslate" title="">public class User {
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
</pre>

<pre class="brush: csharp; title: ; notranslate" title="">public class Role {
    [Key]
    [Display(Name = "Role Name")]
    [Required(ErrorMessage = "Role Name is required")]
    [MaxLength(100)]
    public string RoleName { get; set; }

    public virtual ICollection&lt;User&gt; Users { get; set; }
}
</pre>

Additional properties could be added to the User class and they wouldn’t interfere with the operation of the membership provider as long a they are not “Required” properties. 

I wrote an example ASP.Net MVC3 application and it is available for download at Github: <a href="https://github.com/turnkey-commerce/CodeFirstAltairis/archives/master" target="_blank">Download Code</a>

**Update:** I found an issue with database initialization if the model is changed and the EF data access is not initialized before the classic ADO.Net access provided by Altairis. See [this post][2] for more information.

 [1]: http://www.culbertsonexchange.com/wp/wp-content/uploads/2011/04/unicorn07.gif
 [2]: http://www.culbertsonexchange.com/wp/?p=148