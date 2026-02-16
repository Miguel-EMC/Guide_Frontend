# Security Best Practices

Angular protects against many web vulnerabilities by default, but security still requires careful development practices.

## Key Risks

- Cross-Site Scripting (XSS)
- Cross-Site Request Forgery (CSRF)
- Insecure dependencies
- Leaking secrets in client code

## XSS Protection

Angular sanitizes values bound to the DOM automatically. Avoid bypassing sanitization unless you fully trust the content.

```typescript
constructor(private sanitizer: DomSanitizer) {}

safeHtml = this.sanitizer.bypassSecurityTrustHtml('<b>Trusted</b>');
```

## CSRF Protection

If you use cookies for auth, configure the XSRF token:

```typescript
import { provideHttpClient, withXsrfConfiguration } from '@angular/common/http';

provideHttpClient(withXsrfConfiguration({
  cookieName: 'XSRF-TOKEN',
  headerName: 'X-XSRF-TOKEN'
}));
```

## Content Security Policy

Use a strict CSP header to reduce XSS risk. Avoid inline scripts and unsafe eval.

## Best Practices

- Never store secrets in the frontend
- Use HTTPS everywhere
- Pin and audit dependencies
- Validate and encode user input

## Next Steps

- [PWA](./27-pwa.md) - Secure offline behavior
- [Deployment](./29-deployment.md) - Production hardening

---

[Previous: Performance Optimization](./25-performance.md) | [Back to Index](./README.md) | [Next: PWA & Service Workers](./27-pwa.md)
