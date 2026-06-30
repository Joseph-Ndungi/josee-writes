---
title: "Adding Google Login to IdentityServer the Right Way"
date: 2026-06-29
categories: [Software Engineering, Angular]
tags: [IdentityServer, Google OAuth, OpenID Connect, Authentication, Angular]
canonical_url: ""
image: https://images.unsplash.com/photo-1573804633927-bfcbcd909acd?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
description: Learn how to integrate Google Sign-In with IdentityServer using ASP.NET Core authentication. This step-by-step guide covers Google OAuth configuration, external login setup, callback handling, and common pitfalls to help you add secure social authentication to your applications.
---

**Adding Google Login to IdentityServer: A Comedy of Errors**

I recently added Google login to an Angular app backed by IdentityServer. Simple enough on paper. Took me almost two days.


---

**The Setup**

Stack: Angular 20 (standalone), `angular-auth-oidc-client` v21, IdentityServer8, ASP.NET Core. Google OAuth configured on the backend via `.AddGoogle()`.

Goal: Add a "Continue with Google" button alongside existing credentials login. One button. How hard could it be.

---

**Mistake 1: Wrong endpoint**

First attempt pointed at `/Account/ExternalLogin`. 404. The actual controller was `ExternalController` with a `Challenge` action. Check your backend before assuming standard IdentityServer scaffolding.

---

**Mistake 2: Wrong returnUrl**

Passing `http://localhost:4200/auth/callback` as the `returnUrl` to the challenge endpoint. This made the backend sign the user in and redirect straight to Angular with absolutely no authorization code attached. Token: nowhere. Dashboard: unreachable.

The `returnUrl` must be the full OIDC authorize URL, not your Angular callback. IdentityServer needs to complete the code flow first, then redirect to Angular with `?code=XXX`.

The fix that cracked this open was `urlHandler`:

```typescript
this.oidcSecurityService.authorize(undefined, {
  customParams: {
    acr_values: 'idp:Google',
    prompt: 'login',
  },
  urlHandler: (authorizeUrl) => {
    const challengeUrl =
      `${environment.config.STSURL}/External/Challenge?` +
      `scheme=Google&` +
      `returnUrl=${encodeURIComponent(authorizeUrl)}`;
    window.location.href = challengeUrl;
  },
});
```

`urlHandler` intercepts the authorize URL the library builds and lets you redirect somewhere else first. In this case, the IdentityServer challenge endpoint. IdentityServer handles Google, comes back, signs the user in, then hits `/connect/authorize` with an active session and issues the code. Angular gets `?code=XXX`. Library exchanges it. Token arrives. Dashboard loads.

---

**Mistake 3: `checkAuth()` everywhere**

Calling `checkAuth()` in `AppComponent`, the callback component, and the guard simultaneously caused a race condition. The library was trying to process the auth code in multiple places at once and losing.

Rule: call `checkAuth()` once at app init, skip it on the callback route, let the callback component handle its own:

```typescript
// AppComponent
ngOnInit(): void {
  const currentUrl = window.location.pathname;
  if (currentUrl === '/auth/callback') return;  // Let CallbackComponent handle this

  this.oidcSecurityService.checkAuth().subscribe({
    next: ({ isAuthenticated }) => {
      if (isAuthenticated && (currentUrl === '/' || currentUrl === '/login')) {
        this.router.navigateByUrl('/dashboard');
      }
    }
  });
}
```

```typescript
// CallbackComponent
ngOnInit(): void {
  this.oidcSecurityService.checkAuth().subscribe(({ isAuthenticated }) => {
    this.router.navigate([isAuthenticated ? '/dashboard' : '/login']);
  });
}
```

---

**Mistake 4: `startCheckSession: true`**

This loads IdentityServer in a hidden iframe to poll session state. IdentityServer has `frame-ancestors 'none'` in its CSP. The iframe gets blocked. Turn it off unless you specifically need session monitoring:

```typescript
startCheckSession: false,
silentRenew: false,
triggerRefreshWhenIdTokenExpired: false,
```

---

**Mistake 5: `silentRenewUrl` defaulting to `https://please_set`**

With `silentRenew: false` you'd expect this not to matter. It does. The library was still firing silent authorize requests with `redirect_uri: https://please_set`. IdentityServer rejected every one. Explicitly set it to your actual callback URL:

```typescript
silentRenewUrl: environment.config.AuthCallbackUrl,
```

---

**Mistake 6: Config registered too late**

Calling `setupModule()` inside `AppComponent.ngOnInit()` means the config isn't ready when the first OIDC calls fire. Register at bootstrap:

```typescript
// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptorsFromDi()),
    provideAnimations(),
    importProvidersFrom(
      AuthModule.forRoot(oidcAuthConfig),
      ToastrModule.forRoot()
    )
  ]
});
```

```typescript
// oidc-auth.config.ts
export const oidcAuthConfig: PassedInitialConfig = {
  config: {
    authority: environment.config.STSURL,
    authWellknownEndpointUrl: `${environment.config.STSURL}/.well-known/openid-configuration`,
    redirectUrl: environment.config.AuthCallbackUrl,
    postLogoutRedirectUri: environment.config.MatrixUIURL,
    clientId: environment.config.MatrixUIClientIdAsInSTS,
    scope: 'openid profile email api1',
    responseType: 'code',
    silentRenew: false,
    useRefreshToken: false,
    startCheckSession: false,
    triggerRefreshWhenIdTokenExpired: false,
    silentRenewUrl: environment.config.AuthCallbackUrl,
    disableIatOffsetValidation: false,
    maxIdTokenIatOffsetAllowedInSeconds: 600,
    logLevel: LogLevel.Debug,
  },
};
```

---

**The auth guard: keep it simple**

Use `isAuthenticated$` directly, not `checkAuth()`. Calling `checkAuth()` in the guard fires another authorize request:

```typescript
canActivate(): Observable<boolean | UrlTree> {
  return this.oidcSecurityService.isAuthenticated$.pipe(
    take(1),
    map(({ isAuthenticated }) =>
      isAuthenticated ? true : this.router.createUrlTree(['/login'])
    )
  );
}
```

---

**What the full flow looks like when it works**

```
Click "Login with Google"
  → library builds authorize URL
  → urlHandler sends it to /External/Challenge?scheme=Google&returnUrl=<authorize url>
  → IdentityServer challenges Google
  → user authenticates
  → ExternalController.Callback signs user into IdentityServer session
  → redirects to /connect/authorize (the returnUrl)
  → IdentityServer sees active session, issues authorization code
  → redirects to http://localhost:4200/auth/callback?code=XXX
  → CallbackComponent calls checkAuth()
  → library exchanges code for token
  → isAuthenticated$ emits true
  → dashboard
```

Two days. One `urlHandler` callback. Happy Life.

Happy Coding!
