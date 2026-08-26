---
name: madaket-credentialing-verification
description: Assemble a credentialing picture for one provider from Madaket — licences,
  education and training, board certifications, specialties, and the sanctions and liability
  record — given a Madaket provider GUID.
api: Madaket Provider API
version: v2.0
base_url: https://provider.madakethealth.com/provider-services
operations:
  - professionalLicenseList
  - deaLicenseList
  - cdsLicenseList
  - educationList
  - graduateEducationList
  - internshipList
  - residencyList
  - fellowshipList
  - providerCertificationList
  - providerSpecialtyList
  - adverseActionList
  - criminalActionList
  - liabilityActionList
  - malpracticeClaimList
  - oigExclusionCodeListAll
  - oigExclusionCodeDescribe
generated: '2026-08-25'
method: generated
source: openapi/madaket-provider-api.yml
---

# Verify a provider's credentials with Madaket

Requires a **Madaket provider GUID**. Get one with `skills/madaket-provider-lookup.md` first.
Authenticate exactly as that skill describes — `api_key` and a ten-minute `auth_token`, both
in the query string.

Every operation below follows the same shape:
`GET /api/v2.0/provider/{providerGuid}/<resource>`.

## 1. Licensure

| What | Operation | Path |
|---|---|---|
| State professional licences | `professionalLicenseList` | `/api/v2.0/provider/{providerGuid}/professional-license` |
| DEA registrations | `deaLicenseList` | `/api/v2.0/provider/{providerGuid}/dea-license` |
| State controlled-substance (CDS) | `cdsLicenseList` | `/api/v2.0/provider/{providerGuid}/cds-license` |

DEA and CDS are **separate registrations** and a provider can hold one without the other.
Check both before concluding a provider may prescribe controlled substances.

## 2. Education and training

`educationList`, `graduateEducationList`, `internshipList`, `residencyList`,
`fellowshipList` — same `/api/v2.0/provider/{providerGuid}/<resource>` shape.

`Education`, `GraduateEducation` and `Fellowship` each reference a `University` or
`Institution` and a `ProviderFileStub` (the diploma or training certificate). Follow those
references only if you need the supporting document.

## 3. Certifications and specialties

- `providerCertificationList` — `/api/v2.0/provider/{providerGuid}/provider-certification`
- `providerSpecialtyList` — `/api/v2.0/provider/{providerGuid}/provider-specialty`

Specialties carry NUCC taxonomy codes. At least one taxonomy is expected and exactly one is
primary — that is the NPPES rule Madaket follows.

## 4. Sanctions, actions and claims — do not skip any of these

- `adverseActionList` — `/api/v2.0/provider/{providerGuid}/adverse-action`
- `criminalActionList` — `/api/v2.0/provider/{providerGuid}/criminal-action`
- `liabilityActionList` — `/api/v2.0/provider/{providerGuid}/liability-action`
- `malpracticeClaimList` — `/api/v2.0/provider/{providerGuid}/malpractice-claims`

These are four **distinct** record types. A clean adverse-action list says nothing about
malpractice claims. Query all four.

To resolve an OIG exclusion reason code to its meaning:
`GET /api/v2.0/oig-exclusion-code` (`oigExclusionCodeListAll`) for the whole set, or
`GET /api/v2.0/oig-exclusion-code/{code}` (`oigExclusionCodeDescribe`) for one.

## Rules

- **No pagination anywhere.** A short list may be a truncated list. Do not report "no
  sanctions found" as a negative finding on the strength of one unpaginated response.
- **This is decision-support, not a primary-source verification.** Madaket aggregates from
  primary sources; it is not the licensing board. Anything that will drive a credentialing
  decision should be confirmed against the issuing authority.
- Handle 401 by recomputing the `auth_token`; handle 503 as a host outage.
- The data here is regulated PII — SSN, DOB, DEA numbers. Do not echo it into logs, prompts
  or third-party tools.
