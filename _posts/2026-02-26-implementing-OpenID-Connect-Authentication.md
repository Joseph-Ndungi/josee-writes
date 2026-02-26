---
title: "OpenID Connect Authentication"
date: 2026-02-26
categories: [Angular, Authentication, Security]
tags: [OpenID Connect, OAuth2, Authentication, Security]
canonical_url: "https://dev.to/josephndungi/implementing-openid-connect-authentication-in-angular-19-without-ngmodules-ia8"
image: https://images.unsplash.com/photo-1614064641938-3bbee52942c7?q=80&w=1170&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
description: "Beginners guide on OpenID Connect Authentication"
---

# Implementing OpenID Connect Authentication in Angular 19 Without NgModules

Authentication is one of those things that looks simple from the outside and then quickly humbles you once you start wiring it properly.

Over the past day, I implemented OpenID Connect authentication in an Angular 19 application using Node v20.18.3. I am not using NgModules. The entire app is built using standalone APIs, which makes things cleaner but also slightly different from older tutorials you may find online.

This post walks through the setup, the structure, the issues I ran into, and how I fixed them.

Tech stack:

* Node v20.18.3
* Angular 19.2.20
* angular auth oidc client
* Identity Server as the OpenID provider

No NgModules. Everything is standalone.

---

## Step 1. Install the OIDC client

```bash
npm install angular-auth-oidc-client
```

This library handles the heavy lifting. Token exchange. Storage. Refresh. State handling. You should not manually handle tokens.

---

## Step 2. Configure authentication in app.config.ts

Because this is a standalone Angular app, everything is configured using ApplicationConfig.

```ts
import { ApplicationConfig, provideAppInitializer, inject } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { AuthModule, StsConfigLoader } from 'angular-auth-oidc-client';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),

    importProvidersFrom(
      AuthModule.forRoot({
        loader: {
          provide: StsConfigLoader,
          useFactory: OidcConfigLoaderFactory,
        },
      })
    ),

    provideAppInitializer(() => {
      const oidcSecurityService = inject(OidcSecurityService);
      const router = inject(Router);

      return oidcSecurityService.checkAuth().toPromise().then((result) => {
          if (result?.isAuthenticated) {
            router.navigate(['/dashboard']);
          }
        });
    })
  ]
};
```

Important detail.

Calling checkAuth on app startup is mandatory. Without it, the redirect from Identity Server will not be processed and login will look broken.

This was the first major issue I faced.

Login seemed to work. Tokens were returned. But Angular never picked them up. The fix was adding checkAuth in provideAppInitializer.

---

## Step 3. OIDC configuration

In your config loader:

```ts
{
  authority: 'https://your-identity-server',
  redirectUrl: window.location.origin,
  postLogoutRedirectUri: window.location.origin,
  clientId: 'your-client-id',
  scope: 'openid profile api',
  responseType: 'code',
  silentRenew: true,
  useRefreshToken: true
}
```

Always use responseType code with PKCE.

---

## Step 4. Implement login

Your login service can be simple:

```ts
login(): void {
      this.oidcSecurityService.authorize(undefined, {
      customParams: { prompt: 'login' }
    });
}
```

Why prompt equals login?

This was another issue I ran into.

The app was not redirecting to the Identity Server login page. It was silently authenticating and returning tokens.

The reason was that I already had an active SSO session. The Identity Server detected the session and immediately redirected back without showing the login screen.

Adding prompt equals login forces the login page to appear.

---

## Step 5. Guard your routes properly

Routes:

```ts
export const routes: Routes = [
  { path: 'login', component: LoginComponent },

  {
    path: 'dashboard',
    canActivate: [AuthorizationGuard],
    component: DashboardComponent ,
  },

  { path: '**', redirectTo: '/login' }
];
```

Guard:

```ts
canActivate(): Observable<boolean | UrlTree> {
  return this.oidcSecurityService.checkAuth().pipe(
    map(({ isAuthenticated }) => {
      if (isAuthenticated) {
        return true;
      }
      return this.router.parseUrl('/login');
    })
  );
}
```

Another issue I encountered here was destructuring incorrectly.

My custom isAuthenticated method was already returning a boolean, but in the guard I destructured it as if it was an object. That silently broke navigation and kept redirecting back to login.

Fixing the observable type fixed the routing loop.

---

## Step 6. Redirect after login

After authentication, Angular returns to whatever URL was used as redirectUrl.

If that is login, you will land back on login.

The fix is simple.

Inside LoginComponent:

```ts
ngOnInit() {
  this.securityService.isAuthenticated().subscribe(isAuth => {
    if (isAuth) {
      this.router.navigate(['/dashboard']);
    }
  });
}
```

Now authenticated users never stay on login.

---

## Step 7. Do not manually store tokens

The library already stores tokens in localStorage using DefaultLocalStorageService.

Do not decode and persist tokens manually.

If you need profile info:

```ts
this.oidcSecurityService.getAccessToken().subscribe(token => {
  const decoded = jwtDecode(token);
  console.log(decoded.sub);
});
```

Keep it clean.

---

## Bonus. Lazy loading

You can lazy load the authenticated area:

```ts
{
  path: '',
  component: LayoutComponent,
  canActivate: [AuthorizationGuard],
  loadChildren: () =>
    import('./features/app.routes').then(m => m.APP_ROUTES)
}
```

This improves performance and keeps login lightweight.

---

## Challenges I Faced

1. Login appeared to do nothing
   Root cause was active SSO session
   Fix was prompt equals login

2. Tokens were returned but Angular did not recognize authentication
   Root cause was missing checkAuth on startup
   Fix was adding provideAppInitializer

3. After login it kept returning to login page
   Root cause was incorrect guard observable typing
   Fix was correcting the guard implementation

4. Redirect URL landed on login instead of dashboard
   Fix was redirecting inside LoginComponent when authenticated

---

## Final Thoughts

Implementing OpenID Connect in Angular 19 without NgModules is actually clean once you understand the lifecycle.

The key ideas are:

* Always call checkAuth on startup
* Let the library manage tokens
* Keep guards simple
* Understand how redirect URLs affect routing
* Force login prompt only when needed

Authentication is less about writing code and more about understanding flow.

Once you get the flow right, everything becomes predictable.

Below is a sample Angular app repository.

👉[GitHub Repository](https://github.com/Joseph-Ndungi/OpenID-Connect-Authentication.git)

If you are implementing this on Angular 19 with Node 20 and standalone APIs, this structure should save you a few hours of debugging.

And probably a bit of frustration too.

Happy Coding!
