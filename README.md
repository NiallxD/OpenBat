# OpenBat

OpenBat is an iOS bat detector app. This repo also hosts the **Species Field
Guide** data — a single community-editable JSON file the app downloads and
displays in its Species section. This README covers how to contribute to
that guide. 

## Contributing a species or region

The field guide lives entirely in one file: [`SpeciesGuideData.json`](./SpeciesGuideData.json).
No app code changes are needed to add a species, edit an existing entry, or
add a new region — just edit the JSON and open a PR.

1. Fork the repo and edit `SpeciesGuideData.json`.
2. Add your species (or region) following the schema below.
3. **Bump `dataVersion` by 1** and set `updatedAt` to today's date — see
   [Versioning](#versioning). This is required for every content change,
   however small, or the app won't pick it up.
4. Open a PR. Once merged to `main`, every app install picks up the change
   automatically the next time it launches (no app update needed).

### Schema

Top level:

| Field | Type | Notes |
|---|---|---|
| `schemaVersion` | Int | **Don't change this.** See [Versioning](#versioning). |
| `dataVersion` | Int | Bump by 1 on every content edit. |
| `updatedAt` | String (ISO 8601) | e.g. `"2026-07-06T00:00:00Z"`. Set alongside every `dataVersion` bump. |
| `regions` | [Region] | Pins shown on the explorer globe. |
| `species` | [Species] | The field guide entries. |

**Region:**

| Field | Type | Notes |
|---|---|---|
| `id` | String | Stable slug, e.g. `"uk-ireland"`. Referenced by species' `regions` list — don't rename an existing one without updating every species that references it. |
| `name` | String | Display name, e.g. `"UK & Ireland"`. |
| `latitude` / `longitude` | Double | Where the pin sits on the globe. |

**Species:** only `id`, `commonName`, `scientificName`, and `regions` are
required — everything else is optional. Add what you know; leave the rest
out. The app only shows a section if its fields are present, so a sparse
entry just renders a shorter page rather than empty boxes.

| Field | Type | Notes |
|---|---|---|
| `id` | String | Stable slug, e.g. `"pipistrellus-pipistrellus"` (genus-species, lowercase, hyphenated). |
| `commonName` | String | e.g. `"Common Pipistrelle"`. |
| `scientificName` | String | e.g. `"Pipistrellus pipistrellus"`. Genus (for the taxonomy breadcrumb) is parsed from this automatically — don't add a separate genus field. |
| `order` | String? | Taxonomic order, e.g. `"Chiroptera"`. |
| `family` | String? | e.g. `"Vespertilionidae"`. |
| `regions` | [String] | List of region `id`s where this species occurs. |
| `summary` | String? | Short intro paragraph. |
| `measurements` | Object? | `forearmMmRange`, `wingspanCmRange`, `weightGRange` (each `{ "min": Double, "max": Double }`), `color` (free text). |
| `morphology` | Object? | `earType`, `tailType`, `noseType` (free text), `otherFeatures` (array of short strings). |
| `echolocation` | Object? | `callType` (e.g. `"FM"`, `"CF-FM"`), `peakFreqHzRange` (Pf), `characteristicFreqHzRange` (Cf/knee), `freqHighHzRange` (Fhigh), `freqLowHzRange` (Flow), `durationMsRange` — all `{ "min", "max" }` pairs in Hz/ms — plus free-text `notes` and an optional `exemplarImageName` (must match a bundled app image asset; leave unset if you don't have one). |
| `conservation` | Object? | `iucnStatus` (e.g. `"Least Concern"`), `localStatus` (free text, varies by region/authority). |
| `habits` | Object? | `roosting`, `migration`, `feeding`, `reproduction`, `other` — each free text. |
| `references` | [String]? | Citations, rendered verbatim in small type at the foot of the species page. Any format is fine as long as it's readable — e.g. `"Author, A. (Year) Title. Journal, Vol(Issue), pages."` |

### Example entry

```json
{
  "id": "myotis-daubentonii",
  "commonName": "Daubenton's Bat",
  "scientificName": "Myotis daubentonii",
  "order": "Chiroptera",
  "family": "Vespertilionidae",
  "regions": ["uk-ireland", "continental-europe"],
  "summary": "A small, agile bat that trawls for insects low over still or slow-moving water, using its large feet and tail membrane.",
  "echolocation": {
    "callType": "FM",
    "peakFreqHzRange": { "min": 45000, "max": 50000 },
    "durationMsRange": { "min": 2.0, "max": 5.0 },
    "notes": "Steep, broadband FM sweep; often confused with other Myotis species without call context."
  },
  "conservation": {
    "iucnStatus": "Least Concern"
  },
  "references": [
    "Jones, G. & Rayner, J.M.V. (1988) Flight performance, foraging tactics and echolocation in free-living Daubenton's bats. Journal of Zoology, 215(1), 113–132."
  ]
}
```

See existing entries in `SpeciesGuideData.json` for more complete, real
examples.

### Versioning

- **`dataVersion`**: bump this by 1 for *any* content change — new species,
  corrected text, a new reference, anything. The app compares this number
  to decide whether to adopt your update; if you forget to bump it, your
  change won't reach anyone's device even after merging.
- **`updatedAt`**: an ISO 8601 timestamp set alongside every `dataVersion`
  bump, shown in the app's guide footer so users can see how fresh the data
  is.
- **`schemaVersion`**: this is *not* a content-versioning field — leave it
  alone. It only changes when the JSON's *structure* changes in a way old
  app versions can't safely read (e.g. renaming or repurposing an existing
  field), and that requires a coordinated app release. Adding a new
  optional field (as most content contributions will) never needs a
  `schemaVersion` bump.

### What not to touch

Please don't rename or remove an existing region `id` or species `id` in
the same PR as unrelated content changes — other entries and, in the case
of species ids, potentially saved user data reference these by string, so a
rename should be its own deliberate PR.
