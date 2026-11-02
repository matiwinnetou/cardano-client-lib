# Design note: porting cf-cardano-conversions-java

## Decision

Add a new Gradle module `:conversions`, published as `com.bloxbean.cardano:cardano-client-conversions`.

It is a clean copy of [cardano-foundation/cf-cardano-conversions-java](https://github.com/cardano-foundation/cf-cardano-conversions-java) **v2.0.0** (git `87f32f02e3a56369f696ff3993846a364c69a902`). Packages stay `org.cardanofoundation.conversions`. The module compiles as Java 17 (`--release 17`). It is published on its own and is **not** pulled into the aggregate `cardano-client-lib` jar.

The Cardano Foundation MIT notice is kept in this directory as `LICENSE` (Copyright (c) 2023 Cardano Foundation). Upstream Java sources have no per-file license headers; none were added, so a future sync stays a straight diff. The repository root license remains the BloxBean MIT license and covers the rest of the project.

## Why a new module

The library is self-contained: genesis JSON in, slot/epoch/time/era conversions out. It does not need transaction serialization, addresses, or a backend. A sibling module matches `governance` and `hd-wallet`: its own artifact, its own tests, and a small `build.gradle` that only adds what the root `subprojects` block does not already provide.

Dependencies, all already in this build:

| Need | Where it comes from |
| --- | --- |
| `slf4j-api` | `api` on every subproject |
| Lombok | `compileOnly` / `annotationProcessor` on every subproject |
| `jackson-databind` | version catalog (`2.21.0`; upstream declared `2.16.0`) |

Jackson is an `api` dependency because `ObjectMapper` is a parameter of `ClasspathConversionsFactory.createConverters` and `GenesisConfig`. No other third-party library was added. Tests use the JUnit 5, AssertJ, and SLF4J binding already configured for every module (`slf4j-log4j12` instead of upstream's `slf4j-simple`).

## Alternatives considered

### Fold into `common`

`common` already exposes Jackson, and it is where `SlotConfig` / `SlotConfigs` live. Rejected.

- Almost every module depends on `common`. This port bundles genesis files, including a ~1 MB mainnet Byron genesis. That payload would ship with every consumer.
- `SlotConfig` is one linear Shelley-era clock: slot length, zero slot, zero time, for mainnet, preprod, and preview. It has no Byron 20-second slots, no hard-fork boundaries, and no epoch windows. Nothing else in the tree calls it. The conversions library is a different model (era history plus genesis). Merging them would either leave two unrelated APIs in `common` or rewrite `SlotConfig`, which is not a syncable copy.
- `common` is documented as the module with no project dependencies and the small utilities (hex, JSON, CBOR).

### Fold into `common-spec`, `core`, or `backend`

Rejected.

- `com.bloxbean.cardano.client.spec.Era` is a serialization tag (`Babbage = 6`, `Conway = 7`). `org.cardanofoundation.conversions.domain.Era` is a ledger era plus an optional genesis URL. Putting them in one package, or making one extend the other, collides.
- `core` already depends on transaction serialization, Plutus, and coin selection. The conversions code needs none of that.
- `backend.model.Genesis` and `NetworkInfoService.getNetworkInfo()` are a remote network-info DTO (epoch length, slot length, system start as integers). They are not a local multi-era converter, and they require a backend implementation.

### Rename the package to `com.bloxbean.cardano.client.conversions`

This would match the usual package convention. Rejected for this port because the standalone library stays maintained in parallel. Keeping `org.cardanofoundation.conversions` means upstream commits can be reapplied with a diff, and a later replacement does not force callers to change imports. A rename is left as an open question; it is an API change, not a requirement of the build.

### Put the module on the aggregate `cardano-client-lib` classpath

The aggregate jar currently exposes only `:core`, `:function`, `:cip`, and `:quicktx`. `governance`, `hd-wallet`, `txflow`, and the verified-structure modules are opt-in, even though older README wording says the aggregate includes everything except backends. Conversions are not required to build or submit a transaction, and the module carries the bundled genesis files. Publishing it separately matches those siblings. Adding `api project(':conversions')` in the root project is a one-line follow-up if maintainers want it by default.

## Java 17

Upstream targets Java 17. The port is compiled with `options.release = 17` for main and test sources, which rejects both newer language levels and newer JDK APIs.

v2.0.0 already stays within Java 17. The features it uses were all standardized earlier:

- records (Java 16), including the nested record in `MainNetBoundaryAuditTest`
- switch expressions, comma-separated case labels, and `yield` (Java 14)
- `String.formatted` (Java 15)
- `var`, no-arg `Optional.orElseThrow` (Java 10)
- `List.of` / `Optional` factories (Java 9)

Nothing in v2.0.0 uses pattern matching for switch, record patterns, sequenced collections (`getFirst` / `getLast` / `reversed` on JDK collections), virtual threads, string templates, or JDK methods added after 17. The `getFirstEraHistoryItem` / `getLastEraHistoryItem` methods in `EraConversions` are ordinary private methods, not `SequencedCollection` calls. No source rewrite was required.

## What was not reformatted

Upstream formats with google-java-format (two-space indent). This repository uses four spaces. The port keeps the upstream layout so the next sync is not a full-file diff.
