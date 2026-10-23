I share a sample illustration that has not been fully tested.



This is not a copy of Cynbits code.



Before using it in production run it on a staging copy first.



Add the following code to your `next.config.js` file.



Begin by using the header name `Content-Security-Policy-Report-Only`.



Verify that Google Analytics and Next.js scripts continue to function.



After that replace the header name with `Content-Security-Policy`.



```Js



const csp = \[



"default-src 'self'"



"script-src 'self' 'unsafe-inline' https://www.googletagmanager.com"



"style-src 'self' 'unsafe-inline' https://fonts.googleapis.com"



"font-src 'self' https://fonts.gstatic.com"



"img-src 'self' data: https:"



"connect-src 'self' https://\*.google-analytics.com https://\*.analytics.google.com https://www.googletagmanager.com"



"form-action 'self'; frame-ancestors 'none'; base-uri 'self'"



].join('; ');



securityHeaders = \[



{ key: 'Strict-Transport-Security' value: 'max-age=63072000; includeSubDomains }



{ key: 'X-Frame-Options value: 'DENY' }



{ key: 'X-Content-Type-Options value: 'nosniff' }



{ key: 'Referrer-Policy' value: 'strict-origin-when-cross-origin' }



{ key: 'Permissions-Policy' value: 'camera=() microphone=() geolocation=()' }



{ key: 'Content-Security-Policy' value: csp }



];



module.exports = {



async headers() {



return \[{ source: '/:path\*' headers: securityHeaders }];



}



};



```



\*\*Notes:\*\*



The use of `'unsafe-inline'`, for script sources is a compromise; you can tighten it later by adding nonces.



Only add the `includeSubDomains` directive if every subdomain is served over HTTPS.



Do not add the `preload` directive until you are certain that all subdomains use HTTPS.

