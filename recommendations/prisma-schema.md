Recommended illustration, untested. Not a copy of Cynbit's code. The retention period is a policy decision (see F5).



DOB and gender should only be collected if the company can justify them for the hiring decision; otherwise do not collect them (see F6).



```prisma

generator client {

&#x20; provider = "prisma-client-js"

}



datasource db {

&#x20; provider = "postgresql"

&#x20; url      = env("DATABASE\_URL")

}



enum ApplicationStatus {

&#x20; SUBMITTED

&#x20; UNDER\_REVIEW

&#x20; SHORTLISTED

&#x20; REJECTED

&#x20; WITHDRAWN

&#x20; HIRED

}



// Only fields needed to assess and contact a candidate.

model Applicant {

&#x20; id                     String            @id @default(cuid())

&#x20; fullName               String            @db.VarChar(100)

&#x20; email                  String            @db.VarChar(254)

&#x20; // HMAC-SHA256 of the normalised email, keyed with a server-side secret, used for duplicate detection.

&#x20; emailHash              String            @unique @db.VarChar(64)

&#x20; phone                  String?           @db.VarChar(20)

&#x20; currentCity            String?           @db.VarChar(100)

&#x20; skills                 String?           @db.Text

&#x20; portfolioUrl           String?           @db.VarChar(500)

&#x20; linkedinUrl            String?           @db.VarChar(500)

&#x20; professionalProfileUrl String?           @db.VarChar(500)

&#x20; message                String?           @db.Text

&#x20; status                 ApplicationStatus @default(SUBMITTED)

&#x20; consentVersion         String            @db.VarChar(20)

&#x20; consentedAt            DateTime

&#x20; createdAt              DateTime          @default(now())

&#x20; // Set in application code at creation: createdAt plus the retention period in the policy.

&#x20; retentionExpiresAt     DateTime

&#x20; sensitive              SensitiveProfile?



&#x20; @@index(\[retentionExpiresAt])

&#x20; @@index(\[status])

}



// Collected only after shortlisting, with its own consent record.

model SensitiveProfile {

&#x20; id               String    @id @default(cuid())

&#x20; applicantId      String    @unique

&#x20; applicant        Applicant @relation(fields: \[applicantId], references: \[id], onDelete: Cascade)

&#x20; dateOfBirth      DateTime? @db.Date

&#x20; gender           String?   @db.VarChar(40)

&#x20; permanentAddress String?   @db.Text

&#x20; whatsapp         String?   @db.VarChar(20)

&#x20; consentVersion   String    @db.VarChar(20)

&#x20; consentedAt      DateTime

&#x20; collectedAt      DateTime  @default(now())

}

```



\## Scheduled purge (Vercel cron)



`vercel.json`:



```json

{ "crons": \[{ "path": "/api/cron/purge", "schedule": "0 3 \* \* \*" }] }

```



`app/api/cron/purge/route.ts` (`@/lib/prisma` is a placeholder import):



```ts

import { NextRequest, NextResponse } from 'next/server';

import { prisma } from '@/lib/prisma';



export async function GET(req: NextRequest) {

&#x20; if (req.headers.get('authorization') !== `Bearer ${process.env.CRON\_SECRET}`) {

&#x20;   return new NextResponse('Unauthorized', { status: 401 });

&#x20; }

&#x20; const { count } = await prisma.applicant.deleteMany({

&#x20;   where: { retentionExpiresAt: { lt: new Date() } },

&#x20; });

&#x20; return NextResponse.json({ deleted: count }); // log counts only, never personal data

}

```



Notes: confirm the cron authorization header format in staging. Consider keeping anonymised counts for an audit trail before hard-deleting. Deleting an applicant frees the emailHash, so the same person can apply again.

