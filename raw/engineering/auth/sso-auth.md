# SSO Auth Project Draft

Source: Conversation draft
Collected: 2026-06-09
Published: Unknown
Status: Draft

## Purpose

Build a centralized Single Sign-On platform for many applications, especially B2B SaaS applications across web and mobile.

The platform should provide shared authentication, identity federation, application registration, organization-aware access, and secure token/session handling. It should let many apps use one identity system without forcing every app to reimplement enterprise login, OIDC, mobile auth, audit trails, or user identity linking.

## Positioning

This project is a centralized authentication and enterprise SSO layer for multi-tenant B2B SaaS applications.

It supports web, mobile, first-party, and third-party clients using standards-based OIDC/OAuth flows. The SSO system owns identity and authentication. Each application owns its own local session and application-specific authorization.

## Core Decisions

- OIDC is sufficient for the initial version.
- Better Auth is the MVP authentication engine; do not build OAuth/OIDC/session/token primitives from scratch.
- Each application creates its own local session after SSO.
- Users are globally unique by email.
- Each app may have a controlled white-label login page.
- Mobile offline access should use long-lived refresh tokens, not long-lived access tokens.
- The platform should support both first-party and third-party applications.
- SAML, SCIM, advanced MFA, device trust, and complex policy engines are deferred.

## Target Users

- SaaS application teams that need shared authentication across multiple apps.
- B2B customer admins managing employees and application access.
- Enterprise IT teams configuring identity-provider login.
- End users signing in to web and mobile applications.
- Third-party developers or partners integrating with the platform.

## Scope

### In Scope For MVP

- Multi-tenant organizations.
- Globally unique users by email.
- Application registration.
- OIDC authorization code flow with PKCE.
- Web login support.
- Mobile login support with PKCE.
- Short-lived access tokens.
- Refresh tokens with rotation.
- Offline mobile access through refresh tokens and local app caching.
- ID tokens.
- JWKS endpoint.
- Basic organization membership.
- Basic roles.
- Local app session bootstrap after successful SSO.
- Controlled app-level white-label login settings.
- Generic OIDC enterprise identity provider connection.
- Third-party app registration with scopes and approval.
- Audit logs for sensitive identity and app authorization actions.

### Deferred

- SAML.
- SCIM provisioning.
- Advanced MFA.
- Device trust.
- Risk scoring.
- Full identity-provider marketplace.
- Complex fine-grained authorization engine.
- Arbitrary custom HTML/CSS login pages.
- HRIS provisioning marketplace.

## Architecture Principle

SSO owns authentication. Applications own their own sessions and app-specific authorization.

The SSO token is identity proof. It should not become the primary long-lived login state for every application.

Recommended model:

```text
User -> App -> SSO Login -> App Callback -> App validates result -> App creates local session
```

This keeps authentication centralized while allowing each app to control authorization, tenant context, feature access, session duration, app-specific logout, and product UX.

## Auth Library Decision

Use Better Auth as the authentication engine for the MVP.

Better Auth should provide the lower-level authentication primitives:

- OIDC/OAuth provider behavior.
- Authorization code flow with PKCE.
- Sessions.
- Refresh tokens.
- JWT/JWKS.
- Consent support.
- Organization/member primitives.
- Generic OAuth / enterprise IdP connection support.

The SSO platform should still own the product and policy layer:

- Application registry.
- App-level white-label login branding.
- Organization domain verification.
- Organization SSO enforcement rules.
- Break-glass admin policy.
- Third-party app approval workflow.
- Scope descriptions and risk labels.
- Audit logs.
- Admin console UI.
- Application access rules.
- Mobile offline access policy.
- Compliance/security reporting.

Decision rule:

```text
Better Auth owns auth primitives.
The SSO platform owns product policy, admin workflows, branding, auditability, and app governance.
```

Rationale:

Building OAuth/OIDC, refresh-token rotation, JWKS, consent, redirect URI validation, sessions, and account linking from scratch is risky and expensive for MVP. Better Auth gives a strong foundation while still allowing the platform to control the B2B SaaS product experience.

Before production commitment, run a focused Better Auth spike to verify:

- Token shape and claims are customizable enough.
- JWKS behavior works for client applications.
- Refresh token rotation supports mobile offline access.
- Trusted first-party apps can bypass consent.
- Third-party apps can require consent and organization approval.
- Login and consent UI can support app-level white-label branding.
- Global email uniqueness works cleanly with organization memberships.

## Session Ownership

Applications should create their own local session after completing SSO.

For web applications:

- SSO authenticates the user.
- The app receives and validates the OIDC result.
- The app creates an `HttpOnly`, `Secure`, `SameSite` session cookie.
- The app uses its local session for normal product requests.
- The app owns app-specific authorization and selected organization context.

For mobile applications:

- The app uses authorization code flow with PKCE.
- The app receives short-lived access tokens and refresh tokens.
- Refresh tokens are stored in iOS Keychain or Android Keystore.
- The app stores product state locally for offline access.
- The app refreshes access tokens when network connectivity is available.

## OIDC Protocol Scope

OIDC is sufficient for the initial version.

The MVP should focus on OIDC/OAuth rather than implementing SAML or SCIM immediately.

Initial supported flows and endpoints:

- Authorization code flow with PKCE.
- OIDC ID tokens.
- Short-lived access tokens.
- Refresh tokens where needed.
- Refresh token rotation and reuse detection.
- JWKS endpoint for token verification.
- Generic OIDC enterprise identity provider connection.
- OIDC discovery metadata.
- Token revocation endpoint.
- Logout endpoint.

SAML can be added later for enterprise customers that require it.

## User Identity Model

Users are globally unique by email.

A single email address maps to one global user account across the SSO platform. Organization access is represented through memberships, not duplicated user records.

Implications:

- One user can belong to many organizations.
- One user can access many applications.
- Enterprise IdP accounts link to the same global user by verified email.
- Application sessions should reference the global user ID and selected organization.
- Email changes must be verified before replacing the global email.
- Account merge and conflict handling should be designed carefully.
- Suspended organization membership should not necessarily suspend the global user.

## Multi-Tenancy Model

The organization is the primary tenant boundary.

Each organization may have:

- Name.
- Slug.
- Verified domains.
- Users.
- Memberships.
- Groups.
- Roles.
- SSO configuration.
- Security policies.
- Application access rules.
- Audit logs.

Users can belong to multiple organizations. Applications can serve multiple organizations. Identity provider configuration is organization-specific.

## Application Model

Each connected app is registered in the SSO platform.

Application fields:

- Application name.
- Application type: web, mobile, backend service, third-party app.
- Environment: development, staging, production.
- Client ID.
- Client secret for confidential clients.
- Redirect URI allowlist.
- Allowed origins.
- Token audience.
- Allowed scopes.
- Login callback settings.
- Logout callback settings.
- White-label login settings.
- App owner.
- App verification status.

## White-Label Login Pages

Each registered application may have its own branded login experience.

The login page should be selected using the application `client_id`, redirect URL, and request context. The first version should use controlled theme settings rather than arbitrary custom HTML/CSS.

Supported branding fields:

- Application display name.
- Logo.
- Favicon.
- Primary color.
- Background color or image.
- Login page title.
- Login page subtitle.
- Support link.
- Terms link.
- Privacy link.
- Allowed email domains, if applicable.
- Default organization hint, if applicable.

Recommended rule:

```text
App branding controls the login page appearance.
Organization SSO configuration controls how the user authenticates.
```

If both app branding and organization branding are needed later, use this precedence:

1. Application login shell and product identity.
2. Organization-specific logo or label, if enabled.
3. Identity provider selection based on verified domain or explicit organization selection.

## Enterprise OIDC

Enterprise customers should be able to connect their own OIDC identity provider.

Supported configuration:

- OIDC discovery URL.
- Client ID.
- Client secret.
- Issuer validation.
- Redirect URL generation.
- Attribute mapping.
- Domain verification.
- Test connection.
- Enforced SSO by domain.
- Break-glass admin account.

Likely identity providers:

- Okta.
- Microsoft Entra ID.
- Google Workspace.
- OneLogin.
- Auth0.
- Ping Identity.
- Generic OIDC provider.

## Web Flow

Recommended web login flow:

1. User opens an application.
2. Application redirects to the SSO authorization endpoint.
3. SSO identifies the application from `client_id`.
4. SSO renders the app-specific white-label login page.
5. User authenticates through password, magic link, or enterprise OIDC.
6. SSO redirects back with an authorization code.
7. Application exchanges the code for tokens.
8. Application validates the ID token, issuer, audience, expiry, nonce, and organization context.
9. Application creates its own local session.
10. Application uses its local session for product requests.

## Mobile Flow

Recommended mobile login flow:

1. Mobile app opens the system browser or secure auth session.
2. App starts authorization code flow with PKCE.
3. SSO identifies the app from `client_id`.
4. SSO renders the app-specific white-label login page.
5. User authenticates through password, magic link, or enterprise OIDC.
6. SSO redirects back using universal link or app link.
7. Mobile app exchanges the code plus verifier for tokens.
8. Mobile app stores refresh tokens securely in Keychain or Keystore.
9. Mobile app uses short-lived access tokens for API calls.
10. Mobile app caches product data locally for offline use.
11. Mobile app syncs queued offline writes when connectivity returns.

Do not use embedded webviews for authentication.

## Mobile Offline Access

Mobile applications should support offline access, but access tokens should remain short-lived.

Recommended model:

- Mobile uses authorization code flow with PKCE.
- SSO issues short-lived access tokens.
- SSO issues long-lived refresh tokens when `offline_access` is granted.
- Refresh tokens are stored in iOS Keychain or Android Keystore.
- Refresh tokens use rotation and reuse detection.
- Mobile apps cache required data locally for offline use.
- Offline writes are queued locally and synced when the device reconnects.

Rule:

```text
Offline access should be implemented with secure refresh tokens and local app caching, not long-lived access tokens.
```

## Third-Party Application Support

The platform should support both first-party and third-party applications.

First-party apps are owned by the platform operator. Third-party apps are created by external developers, partners, or customer teams.

Third-party app requirements:

- Developer app registration.
- Public and confidential client support.
- Redirect URI allowlist.
- Client ID issuance.
- Client secret issuance for confidential clients.
- PKCE required for public clients.
- Scope-based access.
- User consent screen.
- Organization admin approval.
- App verification or review flow.
- Rate limits.
- Audit logs for app authorization.
- Token revocation.
- Secret rotation.
- Sandbox and production environments.

Recommended rule:

```text
First-party apps can use trusted internal policies.
Third-party apps must use explicit scopes, consent, approval, and stricter token boundaries.
```

## Authorization

The SSO platform should provide shared identity and access primitives, but avoid becoming a full application authorization engine in the MVP.

Initial primitives:

- Global user.
- Organization membership.
- Application access.
- Basic role.
- Basic group.
- Admin/member distinction.
- OAuth scopes for third-party apps.

Applications should own product-specific permissions, feature flags, workflows, and app-level authorization decisions.

## Tokens

Token types:

- Authorization code.
- ID token.
- Short-lived access token.
- Refresh token.

Token requirements:

- JWT signing with rotating keys.
- JWKS endpoint.
- Issuer validation.
- Audience validation.
- Expiry validation.
- Nonce validation for OIDC.
- Organization claims where applicable.
- User claims.
- Scope claims.
- Role/group claims only where appropriate.
- Token revocation support.
- Refresh token rotation.

Avoid putting too much application-specific authorization into SSO tokens. Claims should be stable, minimal, and useful for bootstrapping the app session.

## Sessions

The platform has two session layers:

- Central SSO session.
- Local application session.

The central SSO session controls whether the user needs to reauthenticate at SSO. Local application sessions control whether the user is logged into a specific application.

Session requirements:

- Secure HTTP-only cookies for SSO web sessions.
- Secure HTTP-only cookies for app web sessions.
- Short-lived access tokens.
- Refresh tokens for mobile and approved offline access.
- Session revocation.
- Device/session listing.
- Global logout.
- Per-application logout.
- Session expiry policies.

## Admin Console

The SSO platform should include an admin console.

Core sections:

- Applications.
- Organizations.
- Users.
- Groups.
- Roles.
- Identity providers.
- Domains.
- Sessions.
- Audit logs.
- Security settings.
- Third-party app approvals.
- API keys and service credentials.

## Security Requirements

Baseline requirements:

- OAuth/OIDC authorization code flow with PKCE.
- OIDC-compatible ID tokens.
- CSRF protection.
- Secure cookies.
- Refresh token rotation.
- Refresh token reuse detection.
- Rate limiting.
- Brute-force protection.
- Account lockout or risk throttling.
- MFA-ready architecture.
- Audit logging for sensitive actions.
- Key rotation.
- Secrets encryption at rest.
- Tenant isolation checks.
- Admin action logging.
- Break-glass recovery path.
- Redirect URI allowlist enforcement.
- Strict third-party app scope boundaries.
- Consent and organization approval for third-party apps.

## Audit Events

Audit logs should capture:

- User login.
- User logout.
- Failed login.
- Password reset.
- Magic link requested.
- Enterprise OIDC configuration changed.
- Domain verified.
- User invited.
- User removed from organization.
- Role changed.
- Group changed.
- Application created.
- Application secret rotated.
- Session revoked.
- Refresh token reuse detected.
- IdP connection tested.
- Admin policy changed.
- Third-party app created.
- Third-party app approved.
- Third-party app revoked.
- User consent granted.
- User consent revoked.

## Suggested MVP Build Order

1. Define domain model: users, organizations, memberships, applications, sessions, identity providers, scopes, and audit events.
2. Implement application registration.
3. Implement OIDC authorization endpoint.
4. Implement authorization code flow with PKCE.
5. Implement token exchange.
6. Implement JWT signing and JWKS.
7. Implement web login flow.
8. Implement local app session bootstrap pattern.
9. Implement mobile callback and PKCE support.
10. Implement refresh tokens with rotation.
11. Implement app-level white-label login settings.
12. Implement basic organization membership and roles.
13. Implement audit logging.
14. Implement generic OIDC enterprise IdP connection.
15. Implement third-party app registration, scopes, consent, and org approval.

## Open Questions

1. Which apps are the first launch customers for this SSO service?
2. Should password login exist in MVP, or should the MVP use magic link plus enterprise OIDC?
3. What is the expected default lifetime for SSO sessions, app sessions, access tokens, and refresh tokens?
4. Should organization selection happen before login, after login, or be inferred from email domain?
5. Which third-party app scopes are needed first?
6. Should app white-label settings be self-serve or managed by platform admins only?
7. What is the required audit log retention period?
8. What are the compliance targets, if any, such as SOC 2, ISO 27001, HIPAA, or GDPR?
