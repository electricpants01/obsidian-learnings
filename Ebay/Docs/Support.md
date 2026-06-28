# Support & Reference

## Channels

| Channel | Purpose |
|---|---|
| [#digital-collections-android](https://slack.com/app_redirect?channel=digital-collections-android) | Android engineering discussions |
| [#dc-engineering](https://slack.com/app_redirect?channel=dc-engineering) | Cross-platform DC engineering |

## Email

- `digital-collections-eng@ebay.com` — Engineering team

---

## Related Documentation

| Doc | Location |
|---|---|
| Architecture & Best Practices | [../docs/CollectiblesArchitectureAndBestPractices.md](../docs/CollectiblesArchitectureAndBestPractices.md) |
| Decor → ScaffoldDataBuilder Migration | [../digitalCollectionsImpl/docs/DecorToScaffoldDataBuilderMigration.md](../digitalCollectionsImpl/docs/DecorToScaffoldDataBuilderMigration.md) |
| CITA Full Technical Doc | [../digitalCollectionsImpl/src/main/java/.../cashInTheAttic/README.md](../digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/cashInTheAttic/README.md) |
| Internal Test Helper | [../digitalCollectionsInternalTestHelper/README.md](../digitalCollectionsInternalTestHelper/README.md) |

---

## Tools

- **GraphQL Playground** — Link to Apollo Studio / GraphQL playground for DC queries
- **Feature Toggle Dashboard** — Link to internal toggle management UI
- **Monitoring** — Datadog dashboards for DC performance and errors

---

## PR Process

- All PRs require at least one approval from the DC Android team
- Architecture changes should be reviewed against [Architecture.md](Architecture.md) patterns
- New features should include tracking (see [Tracking.md](Tracking.md)) and feature toggles (see [FeatureToggles.md](FeatureToggles.md))
- UI changes should include Compose UI tests
