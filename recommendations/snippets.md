Recommended illustrations, untested. Not a copy of Cynbit's code. Adjust for your versions (zod v3 uses `z.string().email()`, v4 uses `z.email()`; `headers()` is async in Next.js 15 and sync in 14).



\## (b) Apex to www permanent redirect and canonical tag



In `next.config.js`. `permanent: true` sends 308. If the current 307 comes from the Vercel Domains setting, change it there instead and remove the domain-level redirect first.



```js

async redirects() {

&#x20; return \[{

&#x20;   source: '/:path\*',

&#x20;   has: \[{ type: 'host', value: 'cynbit.in' }],

&#x20;   destination: 'https://www.cynbit.in/:path\*',

&#x20;   permanent: true,

&#x20; }];

},

```



In `app/layout.tsx`:



```tsx

import type { Metadata } from 'next';



export const metadata: Metadata = {

&#x20; metadataBase: new URL('https://www.cynbit.in'),

&#x20; alternates: { canonical: '/' },

&#x20; openGraph: { url: 'https://www.cynbit.in/', siteName: 'Cynbit Technologies' },

};

```



Each page needs its own `alternates.canonical`. Make the sitemap, robots.txt, og:url and JSON-LD use the same host.



\## (c) Server action with validation



```ts

'use server';

import { z } from 'zod';



const ContactSchema = z.object({

&#x20; fullName: z.string().trim().min(2).max(100),

&#x20; email: z.string().trim().email().max(254),

&#x20; company: z.string().trim().max(120).optional(),

&#x20; industry: z.string().trim().max(80).optional(),

&#x20; message: z.string().trim().min(10).max(2000),

&#x20; website: z.string().optional(), // honeypot, must stay empty

}).strict(); // rejects unknown fields



export async function submitContact(formData: FormData) {

&#x20; const raw = Object.fromEntries(

&#x20;   \[...formData.entries()].filter((\[key]) => !key.startsWith('$ACTION')),

&#x20; );

&#x20; const parsed = ContactSchema.safeParse(raw);

&#x20; if (!parsed.success) return { success: false, error: 'Please check the fields and try again.' };

&#x20; if (parsed.data.website) return { success: true }; // honeypot filled: accept silently

&#x20; // Store with Prisma and send with Resend, using parsed.data only.

&#x20; return { success: true };

}

```



\## (d) Rate limiting with Upstash



Add a hidden field to the form: `<input name="website" tabIndex={-1} autoComplete="off" aria-hidden="true" className="hidden" />`



```ts

'use server';

import { headers } from 'next/headers';

import { Ratelimit } from '@upstash/ratelimit';

import { Redis } from '@upstash/redis';



// Needs UPSTASH\_REDIS\_REST\_URL and UPSTASH\_REDIS\_REST\_TOKEN in the environment.

const limiter = new Ratelimit({

&#x20; redis: Redis.fromEnv(),

&#x20; limiter: Ratelimit.slidingWindow(5, '10 m'),

&#x20; prefix: 'rl:contact',

});



export async function submitGuarded(formData: FormData) {

&#x20; if (String(formData.get('website') ?? '') !== '') return { success: true };

&#x20; // Confirm which client-IP header Vercel sets before relying on x-forwarded-for.

&#x20; const ip = (await headers()).get('x-forwarded-for')?.split(',')\[0]?.trim() ?? 'unknown';

&#x20; const { success } = await limiter.limit(ip);

&#x20; if (!success) return { success: false, error: 'Too many attempts. Please try again later.' };

&#x20; // Continue to validation as in (c).

&#x20; return { success: true };

}

```



\## (e) Generic response for duplicate applications



```ts

'use server';



// The same reply for new and duplicate applications, so the response does not reveal who has applied.

const GENERIC\_REPLY = {

&#x20; success: true,

&#x20; message: 'Thanks. If your application is new, you will receive a confirmation email.',

} as const;



export async function submitApplication(formData: FormData) {

&#x20; // Validate the fields as in (c).

&#x20; // Duplicate check happens here, but its result is never returned or displayed.

&#x20; // If the applicant needs a status link, email it to the address on file as a signed, expiring token.

&#x20; return GENERIC\_REPLY;

}

```

