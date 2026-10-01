---
name: provider-lookup
description: Look up a U.S. healthcare provider in the NPPES NPI Registry and return all of their taxonomy codes, primary first. Use whenever a user provides an NPI number or asks for a provider's taxonomy code or specialty. If a search matches more than one provider, ask the user to choose the correct one.
---

# Provider Lookup

Look up healthcare providers in the NPPES NPI Registry and return their taxonomy codes.

## Data source

Use the public NPPES NPI Registry API (no key required):
https://npiregistry.cms.hhs.gov/api/?version=2.1

## Accuracy rules (critical)

This information is used for billing and credentialing, so an incorrect code can
cause real problems. Every piece of provider information must come directly from
the NPPES API response.

- Only report what the API returned. Copy codes, names, NPIs, and descriptions
  exactly as they appear in the response.
- Never guess, infer, fill in, or "correct" any value, even if it looks wrong or
  incomplete. Report it as it appears.
- Never supply a taxonomy code, description, or provider detail from your own
  knowledge, including well-known codes.
- If a field is empty or missing, say "Not listed in NPPES." Don't leave it out
  and don't substitute anything.
- If the API can't be reached or returns an error, tell the user the lookup
  failed and stop. Never answer without a successful lookup.
- Don't add commentary about the provider (quality, reputation, whether they
  accept insurance) that isn't in the NPPES record.

## Step 1: Figure out what the user gave you

- **An NPI number:** go to Step 2.
- **A name or other details** (for example, "Dr. Jane Smith, cardiologist, Chicago"):
  go to Step 3.

## Step 2: Look up by NPI

1. Check that the NPI is exactly 10 digits. If it isn't, tell the user and ask them
   to double-check the number. Don't search.
2. Request: `https://npiregistry.cms.hhs.gov/api/?version=2.1&number=<NPI>`
3. If `result_count` is 0, tell the user no provider was found for that NPI and
   suggest they verify it. Stop.
4. Otherwise, go to Step 4.

## Step 3: Look up by name

1. Build a search from what the user gave you. Available parameters:
   `first_name`, `last_name`, `organization_name`, `city`, `state`, `postal_code`,
   `taxonomy_description`. Add `&limit=20`.
2. If there are no results, tell the user and suggest loosening the search
   (for example, dropping the city).
3. If there is exactly one result, go to Step 4.
4. If there is more than one result, show a numbered list with each provider's
   name, credential, NPI, primary specialty, and city/state. Ask the user which
   one they mean. Do not choose for them. After they choose, go to Step 4.

## Step 4: Return the taxonomy codes

List **every** taxonomy code in the record's `taxonomies` array. Put the one where
`primary` is `true` first and label it "Primary." List the rest after it.

Use this format:

**Provider:** [Name, credential] (or organization name)
**NPI:** [number]
**Status:** [Active / Deactivated]

| | Taxonomy code | Description | License state |
|---|---|---|---|
| Primary | [code] | [description] | [state] |
| | [code] | [description] | [state] |

## Notes

- If the record is deactivated, still show its taxonomy codes, but point out
  the deactivation clearly.
- Individual providers (`NPI-1`) have a first and last name. Organizations
  (`NPI-2`) have an organization name. Show whichever applies.
- NPPES data is self-reported by providers. For billing or credentialing decisions,
  the user should confirm with the provider.
