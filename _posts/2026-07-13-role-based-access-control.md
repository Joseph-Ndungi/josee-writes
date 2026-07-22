---
layout: post
title: "Understanding Role-Based Access Control (RBAC)"
date: 2026-06-16
author: Joseph Ndungi
tags: [security, authentication, authorization, RBAC, software engineering]
categories: [Software Engineering]
canonical_url: "https://blogs.innova.co.ke/understanding-role-based-access-control/"
description: A practical guide to Role-Based Access Control (RBAC), how it works, why it matters, and how to implement it effectively in modern applications.
image: https://images.unsplash.com/photo-1516321318423-f06f85e504b3?q=80&w=1170&auto=format&fit=crop&ixlib=rb-4.1.0
---

*By Joseph Ndungi*


# Building a RoleBasedAccessControlService 

Every serious application eventually hits the same wall. You built the login. You built the dashboard. Everyone is happy. Then someone in management asks the question that ruins your week: "Can we make sure only admins see the delete button?"

Congratulations. You now need access control.

Most engineers reach for the nearest if statement and call it a day. `if (user.role == "Admin")` sprinkled across forty components. It works, right up until someone adds a new role called Supervisor and now you are hunting through the entire codebase like it owes you money. This is where a proper RoleBasedAccessControlService earns its keep.

## What It Actually Does

At its core, this service answers one question consistently: does this user have permission to do this thing? Not "is this user an Admin," but "can this user perform this action." That distinction matters more than it sounds like it should. Roles are labels. Permissions are the actual currency. A role is just a bundle of permissions wearing a name tag.

Once you separate roles from permissions, adding a new role stops being a redeploy everything event. You just assign a different bundle.

## The Backend, ASP.NET Core

On the server, keep the service dumb and predictable. It should not care about UI, routing, or anyone's feelings. It takes a user and a required permission, and it returns yes or no.

```csharp
public interface IRoleBasedAccessControlService
{
    bool HasPermission(ClaimsPrincipal user, string permission);
    IEnumerable<string> GetPermissions(ClaimsPrincipal user);
}

public class RoleBasedAccessControlService : IRoleBasedAccessControlService
{
    private readonly Dictionary<string, HashSet<string>> _rolePermissions = new()
    {
        ["Admin"] = new HashSet<string> { "users.delete", "users.edit", "reports.view" },
        ["Supervisor"] = new HashSet<string> { "users.edit", "reports.view" },
        ["Viewer"] = new HashSet<string> { "reports.view" }
    };

    public bool HasPermission(ClaimsPrincipal user, string permission)
    {
        var roles = user.Claims
            .Where(c => c.Type == ClaimTypes.Role)
            .Select(c => c.Value);

        return roles.Any(role =>
            _rolePermissions.TryGetValue(role, out var permissions)
            && permissions.Contains(permission));
    }

    public IEnumerable<string> GetPermissions(ClaimsPrincipal user)
    {
        return user.Claims
            .Where(c => c.Type == ClaimTypes.Role)
            .SelectMany(role => _rolePermissions.TryGetValue(role.Value, out var perms)
                ? perms
                : Enumerable.Empty<string>())
            .Distinct();
    }
}
```

In a real system this dictionary lives in a database table, not hardcoded in a class, but the shape of the logic does not change. Wire it into a policy based authorization handler so your controllers stay clean:

```csharp
public class PermissionRequirement : IAuthorizationRequirement
{
    public string Permission { get; }
    public PermissionRequirement(string permission) => Permission = permission;
}

public class PermissionHandler : AuthorizationHandler<PermissionRequirement>
{
    private readonly IRoleBasedAccessControlService _rbac;

    public PermissionHandler(IRoleBasedAccessControlService rbac) => _rbac = rbac;

    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, PermissionRequirement requirement)
    {
        if (_rbac.HasPermission(context.User, requirement.Permission))
            context.Succeed(requirement);

        return Task.CompletedTask;
    }
}
```

Now a controller action just says `[Authorize(Policy = "users.delete")]` and moves on with its life.

## The Frontend, Angular

The frontend version of this service should never be trusted as the actual gatekeeper. It exists purely to make the UI honest, not secure. Hiding a button is not security, it is politeness. The real enforcement lives on the server, always.

```typescript
@Injectable({ providedIn: 'root' })
export class RoleBasedAccessControlService {
  private permissions = new Set<string>();

  setPermissions(permissions: string[]): void {
    this.permissions = new Set(permissions);
  }

  hasPermission(permission: string): boolean {
    return this.permissions.has(permission);
  }
}
```

Populate `permissions` once at login, straight from the token or a profile endpoint, and never recompute it from role names in the UI. Then a structural directive makes it usable in templates without repeating logic everywhere:

```typescript
@Directive({ selector: '[appHasPermission]' })
export class HasPermissionDirective implements OnInit {
  @Input('appHasPermission') permission!: string;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef,
    private rbac: RoleBasedAccessControlService
  ) {}

  ngOnInit(): void {
    if (this.rbac.hasPermission(this.permission)) {
      this.viewContainer.createEmbeddedView(this.templateRef);
    } else {
      this.viewContainer.clear();
    }
  }
}
```

Usage in a template is now boring in the best way:

```html
<button *appHasPermission="'users.delete'">Delete User</button>
```

No component knows or cares what a Supervisor is. It just asks if the permission exists, same question, every time, everywhere.

## The Part People Skip

The temptation once this is built is to relax. Don't. Two things quietly wreck RBAC systems over time.

First, permission sprawl. Every feature adds two or three new permission strings until nobody remembers what `reports.export.legacy` actually gates. Review and prune this list the same way you would review dead code, because that is what it is.

Second, caching staleness. If a user's role changes mid session and your frontend cached the old permission set, they either see things they shouldn't or lose access to things they should still have. Refresh the permission set on token renewal, not just on login.

Get those two things right and this pattern will quietly hold up an entire application's worth of feature gates without anyone thinking about it again. Which is the actual goal. Good infrastructure is invisible.

Happy Coding!