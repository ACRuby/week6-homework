# week6-homework

Homework for Week 6 of AI Foundations: documenting the **Provider Lookup** Claude skill.

## Demo

**Try it:** https://acruby.github.io/week6-homework/demo/

The demo page ([demo/index.html](demo/index.html)) walks through the skill's
workflow in the browser:
- **Look up by NPI.** It checks for exactly 10 digits and rejects anything else.
- **Look up by name.** When several providers match, it lists them and asks you to
  pick one.
- **Show the taxonomy codes.** It lists every code, primary first, and shows
  "Not listed in NPPES." for empty fields.

Browsers can't call the NPPES API directly, because the API doesn't allow
cross-site requests. So the demo searches a fixed set of 17 real NPPES records
([demo/sample-providers.json](demo/sample-providers.json)), copied unchanged from
the API. Each result links to that provider's live record on the NPPES site, so you
can check it. Claude itself calls the live API when it uses the skill.

The demo is a standalone web page that imitates the skill's steps. It doesn't use
Claude or read the skill file.

Example searches built into the page:
- NPI `1922074434`: one organization with 5 taxonomy codes
- Organization "Mayo Clinic", state MN: 12 matches, so you choose one
- Last name "Nguyen", Houston, TX, specialty "cardiovascular": 5 matches
- NPI `12345`: rejected for not being 10 digits

The full skill definition is in [provider-lookup/SKILL.md](provider-lookup/SKILL.md).

## Install the skill

The skill is packaged as **[provider-lookup.skill](provider-lookup.skill)**
([download](https://github.com/ACRuby/week6-homework/raw/feature/skill/provider-lookup.skill)).
A `.skill` file is a zip archive of the skill folder, `provider-lookup/SKILL.md`. It
was built and validated with the packaging script from Anthropic's skill-creator
skill.

- **Claude app:** go to the Skills section of Settings and upload
  `provider-lookup.skill`.
- **Claude Code:** extract the file into your personal skills folder. Windows'
  built-in `tar` can read the zip format:

  ```bash
  tar -xf provider-lookup.skill -C ~/.claude/skills
  ```

  This creates `~/.claude/skills/provider-lookup/SKILL.md`.

### Repo layout

| Path | What it is |
|---|---|
| `provider-lookup/SKILL.md` | The skill's source |
| `provider-lookup.skill` | The packaged skill, ready to install |
| `demo/` | The browser demo and its sample NPPES records |

To rebuild the package after editing `SKILL.md`, run skill-creator's packaging
script on the `provider-lookup` folder:
`python -m scripts.package_skill <path>/provider-lookup <output-dir>`. Run it from
the skill-creator folder; it needs PyYAML.

## What the skill is

Provider Lookup (`provider-lookup`) is a Claude skill that looks up U.S. healthcare
providers in the **NPPES NPI Registry** and returns their **taxonomy codes**. A
taxonomy code identifies a provider's type and specialty, such as a family medicine
physician or a physical therapist. The skill uses the public NPPES API
(`https://npiregistry.cms.hhs.gov/api/?version=2.1`), which needs no API key.

Claude uses the skill when a user:
- gives an NPI (National Provider Identifier) number, or
- asks for a provider's taxonomy code or specialty.

## What it accomplishes

Given an NPI or a provider's name and details, the skill returns **every** taxonomy
code on that provider's NPPES record. The primary code is listed first and labeled
"Primary", and each code includes its description and license state. The output
also shows the provider's name (or organization name), NPI, and whether the record
is active or deactivated.

People working in billing and credentialing need these codes, and getting one wrong
can cause real problems. So the skill aims to return exactly what the registry says
and nothing else.

## How it works

1. **Identify the input.** The input is either an NPI number or a name with other
   details, such as "Dr. Jane Smith, cardiologist, Chicago".
2. **Look up by NPI.** The skill checks that the NPI is exactly 10 digits before
   searching. If no provider matches, it tells the user and suggests verifying the
   number.
3. **Look up by name.** The skill searches using the details given: first and last
   name, organization name, city, state, postal code, or specialty. It returns up to
   20 results.
   - No results: it suggests loosening the search, for example by dropping the city.
   - One result: it continues to step 4.
   - More than one result: it shows a numbered list (name, credential, NPI, primary
     specialty, city/state) and **asks the user to choose**. It never picks for them.
4. **Return the taxonomy codes.** The results are shown in a table like this:

   **Provider:** Name, credential (or organization name)
   **NPI:** number
   **Status:** Active / Deactivated

   | | Taxonomy code | Description | License state |
   |---|---|---|---|
   | Primary | code | description | state |
   | | code | description | state |

## Accuracy rules

The skill's main safeguard is that every value must come straight from the API
response:

- It copies codes, names, NPIs, and descriptions exactly as returned. It never
  guesses, infers, or "corrects" any value.
- It never supplies a taxonomy code or provider detail from Claude's own knowledge,
  even a well-known code.
- If a field is empty, it shows "Not listed in NPPES." instead of leaving the field
  out or filling it in.
- If the API fails or can't be reached, it says the lookup failed and stops. It
  never answers without a successful lookup.
- It adds no commentary about the provider, such as reputation or insurance
  accepted, that isn't in the NPPES record.

## Other notes

- Deactivated records still show their taxonomy codes, and the deactivation is
  pointed out clearly.
- Individual providers (NPI-1) are shown by first and last name. Organizations
  (NPI-2) are shown by organization name.
- NPPES data is self-reported by providers, so users should confirm with the
  provider before making billing or credentialing decisions.
