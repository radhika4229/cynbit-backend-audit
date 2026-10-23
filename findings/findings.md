\# Findings



Each finding: Severity | Status | Evidence | Impact | Fix. Statuses: OBSERVED (seen directly), INFERRED (reasoned from evidence), HYPOTHESIS (needs authorization to verify).



\## F1 Missing security headers | Medium | OBSERVED

Evidence: curl response headers and securityheaders.com (grade D) show no Content-Security-Policy, X-Frame-Options, X-Content-Type-Options, Referrer-Policy or Permissions-Policy. See evidence/.

Impact: no clickjacking or MIME-sniffing protection on a site that hosts forms collecting personal data.

Fix: add the headers in next.config.js (see recommendations/security-headers.md). Start CSP in Report-Only mode.



\## F2 HSTS without includeSubDomains or preload | Low | OBSERVED

Evidence: `strict-transport-security: max-age=63072000`.

Fix: add includeSubDomains once all subdomains are HTTPS; consider preload later.



\## F3 Canonical host inconsistency | Low | OBSERVED

Evidence: the apex returns 307 (temporary) to www; the robots.txt Sitemap line, sitemap `<loc>` entries, og:url and JSON-LD use the apex; og:image is an SVG.

Fix: pick one host, redirect with 308, align sitemap, robots.txt, og:url and JSON-LD, add canonical tags, use a PNG or JPG for og:image.



\## F4 sitemap lastmod changes between requests | Low | OBSERVED values, INFERRED cause

Evidence: the first entries showed 2026-10-22T09:44:54Z in one fetch and 2026-10-22T11:07:59Z in a later fetch.

Fix: use real content-modified dates.



\## F5 Privacy policy gaps | Medium | OBSERVED

Evidence: /privacy (updated May 20, 2026) has 5 sections and no retention period, storage location, correction right, grievance contact or mention of minors. "Contact Privacy Team" links to /contact.

Impact: applicants and students cannot tell how long data is kept or how to correct it (India DPDP Act 2023 context).

Fix: add retention per data type, hosting locations, a correction route, a named contact and an age statement.



\## F6 Data collected exceeds the "minimum information" claim | Medium | OBSERVED

Evidence: /privacy section 1 says "minimum information necessary, typically your name, email, and professional context". The careers form requires date of birth, gender, phone, WhatsApp and permanent address.

Fix: remove fields that are not needed, or collect them only after shortlisting with a stated purpose (see recommendations/prisma-schema.md).



\## F7 Privacy requests go to the careers mailbox | Low | OBSERVED

Evidence: /privacy section 5 uses opportunities@cynbit.in, the same address used for applications.

Fix: a separate privacy address or form, a named owner and an identity check.



\## F8 Analytics cookies on first load, not described in the privacy policy | Medium | OBSERVED

Evidence: a fresh Incognito load shows `\_ga` and `\_ga\_LK6...` cookies (evidence/cookies-first-load.png). /privacy names only Resend and Prisma; /cookies mentions Google Analytics. Google Fonts loads from fonts.googleapis.com and fonts.gstatic.com.

Fix: name Google Analytics and Google Fonts in the policy, ask for consent before analytics where required, self-host fonts.



\## F9 Client-side-only validation visible | Low | HYPOTHESIS

Evidence: HTML5 attributes only (required, type email/url/date, minLength 10); no zod or yup in the public JS. Server-side validation is NOT VISIBLE.

Fix: server-side schema validation with length caps, and reject unknown fields.

Verify with authorization: review the server code. Do not submit inputs.



\## F10 No visible abuse controls | Low | HYPOTHESIS

Evidence: no CAPTCHA, honeypot or rate-limit message in client code. Edge protection is NOT VISIBLE.

Fix: rate limiting plus a honeypot or CAPTCHA.

Verify with authorization: ask for the firewall and rate-limit configuration.



\## F11 Duplicate-application response and tracking links | Low | HYPOTHESIS

Evidence: the submit handler reads `isDuplicate` and `trackingUrl`. Whether this allows user enumeration or unauthorized tracking-page access is NOT VISIBLE.

Fix: same generic response for new and duplicate submissions; email a signed, expiring link.

Verify with authorization: code review of the duplicate logic and tracking-page access control.



\## F12 robots.txt lists sensitive path names | Info | OBSERVED

Evidence: `Disallow: /api/`, `/admin/`, `/portal/`. robots.txt is not an access control.

Fix: make sure those paths enforce authentication and authorization on the server.



\## F13 Custom 404 without leaks (positive) | Info | OBSERVED

Evidence: the 404 page reuses the site layout and shows no stack trace, file paths or framework version. Only x-matched-path and x-next-error-status headers are exposed.



\## F14 Newsletter form not in the privacy policy | Low | OBSERVED

Evidence: an email form sits in the site-wide layout; /privacy has no newsletter, unsubscribe or retention text.

Fix: document the newsletter, sender, retention and unsubscribe route.



\## F15 Access-Control-Allow-Origin: \* on public HTML | Info | OBSERVED

Evidence: response header on the homepage. Low impact for public content.

Fix: confirm API routes use a restrictive CORS policy.

