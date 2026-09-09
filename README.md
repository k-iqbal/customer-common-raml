# customer-common-raml

**Type:** RAML 1.0 Library fragment (shared, not independently deployable)
**Status:** Complete — single source of truth for all shared contract pieces.

## Purpose

Holds every RAML fragment shared by all three API layers, so there is exactly **one**
copy of the `Customer` model, the error contract, and the reusable traits/resourceTypes
in the whole platform — instead of three drifting copies.

`customer-exp-api`, `customer-process-api`, and `customer-salesforce-sys-api` each pull
this in with:

```raml
uses:
  common: ../../../../../customer-common-raml/common.raml
```

and reference everything with the `common.` prefix, e.g. `type: common.CustomerResponse`,
`is: [common.correlation-id, common.common-error-responses]`, `type: common.collection`.

## Structure

```
customer-common-raml/
├── common.raml              The Library root — declares types/traits/resourceTypes,
│                             each pointing at the fragment files below.
├── dataTypes/
│   ├── Customer.raml
│   ├── CustomerRequest.raml
│   ├── CustomerResponse.raml
│   ├── CustomerListResponse.raml
│   ├── ErrorResponse.raml
│   ├── ImportResponse.raml
│   └── ExportResponse.raml
├── traits/
│   ├── correlation-id.raml
│   ├── client-id-required.raml
│   └── common-error-responses.raml
├── resourceTypes/
│   ├── collection.raml
│   └── collection-item.raml
└── examples/                 JSON examples, referenced directly by each API's root RAML
    ├── customer.json
    ├── customer-create-request.json
    ├── customer-response.json
    ├── customer-list.json
    ├── error-response.json
    ├── import-response.json
    └── export-response.json
```

## Why a Library and not just shared files

RAML has a specific fragment type for this: `#%RAML 1.0 Library`. A Library is what
`uses:` is designed to import — trying to `uses:` a plain `#%RAML 1.0 DataType` fragment
directly (which is what the very first version of this project did) is invalid RAML and
produces exactly the red error markers you saw in Studio. `common.raml` is the properly
typed Library that fixes that.

## Local development vs. Design Center / Exchange

**Right now** (local files, no Exchange dependency yet): the three API projects reference
this library via a relative filesystem path, since all four projects are siblings under
`customer-integration-platform/`. This works as-is in Studio/VS Code with no extra setup.

**Once published to Exchange:** import this project into Design Center first, publish it
as an asset (e.g. `customer-common-raml` v1.0.0), then in each of the three API projects'
Design Center specs, add it as a dependency via Exchange (Design Center's UI inserts the
correct `uses:` reference for you — you won't hand-edit the path). From that point, this
becomes the versioned, governed source of truth, and the relative-path version here is
superseded.

## Versioning going forward

Because all three APIs depend on this library, changing a shared type is a
**breaking-change surface for all three layers at once**. Once this is in Exchange:
bump the version (e.g. `1.0.0` → `1.1.0` for additive changes, `2.0.0` for breaking ones)
on publish, and update each API spec's dependency version deliberately rather than
always pointing at "latest."
