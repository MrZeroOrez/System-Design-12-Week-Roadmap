# Week 2 · Day 5 — Authentication, Authorization, Cookies, Sessions and JWT

> Detailed backend engineering revision chapter | Networking and Web Architecture

## Objectives
By the end, explain the foundational concepts, apply them to a Java/Spring Boot system, discuss alternatives and partial failures, and defend the choice in a system design interview.

## Detailed study notes
### 1. Authentication versus authorization

**Authentication (AuthN)** establishes who the caller is; **authorization (AuthZ)** decides what the caller may access or do. Login success does not authorize viewing every resource. Apply object-level authorization checks for user-owned orders; do not rely on hidden UI elements. Model permissions with roles, scopes and ownership policies.

### 2. Sessions and cookies

A session-based system creates a random, high-entropy session identifier after login and stores state on the server or an appropriate shared store. The browser sends the identifier in a cookie. Cookie attributes 'HttpOnly', 'Secure' and 'SameSite' mitigate different threats but do not replace server-side validation. Logout typically invalidates the session. Regenerate session IDs after authentication to help resist session fixation.

### 3. What JWT is and is not

JSON Web Token is a token format with header, claims and signature (for signed JWTs). A JWT is not automatically encrypted; payload data is ordinarily readable. Verify signature algorithm policy, issuer, audience, expiry, not-before and key rotation. Never accept 'alg=none' blindly. Tokens can be revoked through short expiry, denylisting, versioning or introspection-dependent architectures, each with trade-offs.

### 4. Access tokens, refresh tokens and OAuth2/OIDC

**Access tokens** authorize API access and should be short-lived as appropriate. **Refresh tokens** help obtain new access tokens under a defined rotation/revocation policy. **OAuth2** delegates authorization; **OpenID Connect (OIDC)** adds an authentication identity layer. A JWT is one possible token representation; it is not synonymous with OAuth2. Authorization Code + PKCE is standard for browser/mobile public clients.

### 5. Security threats and mitigations

**CSRF** exploits browser-managed credentials such as cookies; mitigate using appropriate SameSite policy and anti-CSRF tokens where required. **XSS** can run malicious scripts; HttpOnly limits JavaScript access to cookies but does not prevent all XSS impacts. **CORS** is browser-enforced cross-origin read/control policy, not API authentication. Enforce TLS, password hashing with suitable modern schemes, rate limits and account lockout/abuse checks.

### 6. Auth architecture in microservices

A gateway can validate access-token authenticity for edge routing, but services must still enforce domain-level permissions. If tokens are forwarded, validate trusted issuer and audience, avoid passing secrets into logs, and use mutual TLS or service identities where needed for machine-to-machine calls. Authentication context propagation should not make internal networks a blanket trust zone.

### 7. HTTP 401 versus 403 and errors

Return 401 when authentication is missing/invalid (commonly with 'WWW-Authenticate'); 403 when authenticated but forbidden. 404 may intentionally conceal resource existence depending on policy. Never return password hashes, tokens, secret keys or sensitive PII in error messages. Log correlation IDs rather than credentials.

### 8. Choosing session versus stateless token

Traditional server-side sessions ease immediate revocation and centralized policy but require shared state when horizontally scaled. Signed tokens enable local verification, but handling theft/revocation and audience restrictions is harder. Choose based on client type, threat model, infrastructure and lifecycle needs, not because one is always 'modern'.

## Diagram: system flow
```mermaid
sequenceDiagram
    participant B as Browser
    participant A as Auth Service
    participant API as Resource API
    participant S as Shared Session Store
    B->>A: POST /login (over HTTPS)
    A->>A: Verify password hash
    A->>S: Store random session ID
    A-->>B: Set-Cookie: sid=...; HttpOnly; Secure
    B->>API: GET /orders/42 + Cookie
    API->>S: Validate session
    API->>API: Check order ownership
    API-->>B: 200 or 403
```

Explain every arrow and trust boundary, and identify at least two failure points.

## Implementation example
```java
// Conceptual Spring Security resource-server protection
@Bean
SecurityFilterChain security(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
            .requestMatchers("/actuator/health").permitAll()
            .requestMatchers("/api/orders/**").authenticated()
            .anyRequest().denyAll())
        .oauth2ResourceServer(oauth -> oauth.jwt(Customizer.withDefaults()));
    return http.build();
}
// Configure trusted issuer + audience verification, and enforce
// per-order ownership in service/domain logic.
```

*Examples are educational excerpts, not a claim of a runnable or tested repository application. Add compatible dependencies, imports, exception handling and tests when implementing.*

## Practical assignment
Create a security design for a Product Catalog and Order API. Mark anonymous routes, authenticated routes and owner-only routes. Demonstrate 401 vs 403, configure a supported Spring Security auth mode, and test 'User A may not read User B's order'. Explain CSRF considerations when cookies are used.

### Verification checklist
- [ ] Requirements and assumptions clearly documented
- [ ] Diagram explains all components and data flow
- [ ] At least two failure scenarios and safe responses described
- [ ] Relevant Java/config or API contract included
- [ ] Tests or reproducible verification procedure specified

## Interview questions
1. Authentication vs authorization?
2. Is JWT data encrypted?
3. Why use HttpOnly + Secure cookies?
4. What is CSRF, and when does it matter?
5. Why must a service check ownership even after gateway auth?

## Five-question knowledge check (answer without notes)
1. What problem does today's primary technique solve?
2. What are its most important trade-offs?
3. What happens when a dependent component fails?
4. How would you keep the design secure?
5. How would you validate behavior and performance?

## Progress
- [ ] Read and summarize chapter
- [ ] Recreate diagram
- [ ] Complete practical assignment
- [ ] Answer interview questions aloud
- [ ] Document one improvement
