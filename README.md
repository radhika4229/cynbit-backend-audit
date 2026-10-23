### cynbit-backend-audit

\# Cynbit Backend Audit



Passive backend audit of https://www.cynbit.in for the Backend Developer Intern assessment.

Author: Radhika Sishodiya. Date: 22-23 Oct 2026.



\## Scope and method

Only passive testing: public pages, public JavaScript files, HTTP response headers of public URLs

(browser DevTools and curl, one request per URL) and public checker tools (securityheaders.com).

No forms were submitted. No endpoint, no server action, no tracking URL or ID was called or tested.

Every finding is tagged as OBSERVED, INFERRED or HYPOTHESIS.



\## Summary

| ID | Finding | Category | Severity | Status |

|---|---|---|---|---|

| F1 | Missing security headers (CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy); securityheaders.com grade D | Security | Medium | OBSERVED |

| F5 | Privacy policy gaps: no retention, storage location, correction right, grievance contact, minors | Privacy | Medium | OBSERVED |

| F6 | Careers form collects DOB, gender, address, WhatsApp, but policy claims "minimum information" | Privacy | Medium | OBSERVED |

| F8 | Google Analytics cookies set on first load; GA and Google Fonts not named in privacy policy | Privacy | Medium | OBSERVED |

| F2 | HSTS without includeSubDomains/preload | Security | Low | OBSERVED |

| F3 | Canonical host inconsistency (307 apex to www vs apex in sitemap/metadata) | SEO | Low | OBSERVED |

| F4 | sitemap lastmod changes between requests | SEO | Low | OBSERVED / INFERRED cause |

| F7 | Privacy requests go to the careers mailbox | Privacy | Low | OBSERVED |

| F14 | Newsletter form on every page, not in privacy policy | Privacy | Low | OBSERVED |

| F9 | Client-side-only validation visible | Input validation | Low | HYPOTHESIS |

| F10 | No visible CAPTCHA/honeypot/rate-limit in client code | Security | Low | HYPOTHESIS |

| F11 | Duplicate-application response / tracking links | Authentication | Low | HYPOTHESIS |

| F12 | robots.txt lists /api/, /admin/, /portal/ | Security | Info | OBSERVED |

| F13 | Custom 404, no stack trace (positive) | Error handling | Info | OBSERVED |

| F15 | Access-Control-Allow-Origin: \* on public HTML | Security | Info | OBSERVED |



\## Top 5 priorities

1\. F6  2. F5  3. F1  4. F8  5. F3



\## Positive observations

HSTS (2 years), HTTP/2, Brotli/gzip compression, CDN cache hit, no X-Powered-By, custom 404 without leaks,

No Set-Cookie on the homepage response.



\## Limitations

Passive only. Server side validation, edge/firewall rules, rate limits, authorisation on tracking/portal.

Paths and database behaviour were not tested. HYPOTHESIS items require the company's authorisation.



\## Repo map

\- report/audit.md: architecture and observed behaviour

\- findings/findings.md: all the findings with the evidence and fixes

\- recommendations/: code snippets (illustrations, untested)

\- evidence/: header dumps and screenshots (personal data redacted)

