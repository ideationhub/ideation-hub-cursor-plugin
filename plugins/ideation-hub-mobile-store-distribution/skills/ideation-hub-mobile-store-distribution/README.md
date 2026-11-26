# Mobile store distribution (0.1.0 beta)

A portable Agent Skill for shipping a mobile app on the Apple App Store and Google Play. It covers download size, startup time, offline use, on-device text recognition, consumable in-app purchases, and the permission and privacy disclosures those stores require.

It is generic. A price-tag scanner that sells packs of trip credits is the worked example, not a product this repository ships.

This skill does not call Ideation Hub. It has no MCP server and no secrets. Install it on its own, or next to the other Ideation Hub plugins. The root compatibility bundle does not load it.

## What is in the package

```text
ideation-hub-mobile-store-distribution/
├── SKILL.md
├── references/
│   ├── size-limits.md
│   ├── budgets.md
│   ├── offline-and-scanning.md
│   ├── purchases.md
│   └── privacy-and-permissions.md
├── CHANGELOG.md
└── README.md
```

There are no scripts, so the skill needs no runtime.

## Install

Install the **Ideation Hub Mobile Store Distribution** plugin from this repository's marketplace. Hosts that only load skill folders can copy `plugins/ideation-hub-mobile-store-distribution/skills/ideation-hub-mobile-store-distribution`. A single generated file is `docs/instructions/ideation-hub-mobile-store-distribution.md`.

## Status

0.1.0 is a beta. Store numbers were checked against the linked Apple and Google pages on 26 November 2026. Re-read the page before you treat a number as current.

## Licence

MIT, as the rest of this repository.
