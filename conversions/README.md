# Conversions (`cardano-client-conversions`)

Deterministic conversions between Cardano time, absolute slot, epoch, and era bounds.

This module is a clean port of [cf-cardano-conversions-java](https://github.com/cardano-foundation/cf-cardano-conversions-java) release **v2.0.0**. The standalone library continues to be maintained in parallel. Sources, tests, and the `org.cardanofoundation.conversions` packages are copied so a later sync is a diff against that repository rather than a rewrite.

Copyright (c) 2023 Cardano Foundation. Licensed under the MIT License. See [LICENSE](LICENSE).

The public types are unchanged from v2.0.0. See [DESIGN.md](DESIGN.md) for why this is its own module and why it is not part of the aggregate `cardano-client-lib` jar.

**Group Id:** `com.bloxbean.cardano`
**Artifact Id:** `cardano-client-conversions`

Bundled genesis files (Byron and Shelley) cover mainnet, preprod, preview, and sanchonet. Era boundaries after genesis are the static tables from v2.0.0.

## Dependency

```xml
<dependency>
    <groupId>com.bloxbean.cardano</groupId>
    <artifactId>cardano-client-conversions</artifactId>
    <version>0.8.0-pre5-SNAPSHOT</version>
</dependency>
```

```groovy
implementation 'com.bloxbean.cardano:cardano-client-conversions:0.8.0-pre5-SNAPSHOT'
```

Runtime dependencies already used by this repository: SLF4J (provided to every module by the root build) and Jackson Databind (version catalog). Lombok is compile-only.

## Example

```java
var converters = ClasspathConversionsFactory.createConverters(NetworkType.MAINNET);

var epochStartTime = converters.epoch().beginningOfEpochToUTCTime(445);
var utcTime = converters.slot().slotToTime(109090938L);
var epochNo = converters.time().utcTimeToEpochNo(LocalDateTime.of(2023, 11, 22, 9, 48, 58));
var lastAlonzoAbsoluteSlot = converters.epoch().endingOfEpochToAbsoluteSlot(364);
var firstRealAbsoluteSlotBabbage = converters.era().firstRealSlot(EraType.Babbage);
var firstRealTimeBabbage = converters.era().firstRealEraTime(EraType.Babbage);
```

`ConversionsExample` prints the preprod era history and can be run from this module.
