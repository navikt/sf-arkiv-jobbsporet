# sf-arkiv-jobbsporet [<img align="right" src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" width="18" alt="GitHub repository" />](https://github.com/navikt/sf-arkiv-jobbsporet)
Lokalarkivinstans for Jobbsporet

Appen tilbyr et API for lagring og henting av dokumentasjon fra Salesforce i PostgreSQL (Cloud SQL) på NAIS GCP.
Denne instansen tilhører team Jobbsporet og bruker namespace `jobbsporet`.

Se swagger for API:

[![Swagger UI (dev)](https://img.shields.io/badge/DEV-Swagger_UI-2ea043)](https://sf-arkiv-jobbsporet.intern.dev.nav.no/internal/swagger-ui/)
[![Swagger UI (prod)](https://img.shields.io/badge/PROD-Swagger_UI-blue)](https://sf-arkiv-jobbsporet.intern.nav.no/internal/swagger-ui/)

## Lagring og henting

**`POST /arkiv`** lagrer en ikke-tom JSON-liste i én transaksjon. Bare `dokumentdato` er påkrevd (`yyyy-MM-dd`); øvrige felt er valgfrie.
Se Swagger for modellene. Kvitteringen inneholder ikke dokumentinnhold eller identifikasjonsfeltene `aktoerid`, `fnr` og `orgnr`.

**`POST /hente`** er ment for å verifisere integrasjonen og bør normalt slås av etter testing.
Send bare feltene du vil filtrere på, for eksempel `{"kilde":"test"}`. Minst ett felt må være utfylt;
alle utfylte felt må matche (AND). Responsen inneholder dokumentinnholdet.

**`konfidentiellt: true`** gir et ekstra vern: dataene lagres, men kan ikke hentes tilbake via `/hente`, selv når henting er aktivert.
Flagget er valgfritt, standardverdien er `false`, og verdien må være en boolsk JSON-verdi.
Responsheaderen `x-konfidentiellt-skipped` fra `/hente` angir antall utelatte treff og røper dermed at de finnes.

**`GET /authping`** kan brukes til å kontrollere om et kall har et gyldig token. `/authping` viser resultatet av tokenvalideringen;
`/arkiv` og `/hente` krever et gyldig token for Salesforce-integrasjonen, med unntak av testbypass beskrevet nedenfor.

## Konfigurasjon

| Variabel | Betydning                                                                                                                                                                                                                                                                                                                                                                                                     |
|---|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DEV_TEST_BYPASS_ALLOWED` | `"true"` tillater testkall uten token i `dev-gcp` når `kilde` er `"test"` (første oppføring i `/arkiv`, søkefeltet i `/hente`). Manglende eller annen verdi slår av bypass. Bruk for testing direkte i swagger.                                                                                                                                                                                               |
| `HENTE_ALLOWED` | `"true"` godkjenner kall til `/hente`. `"false"`, manglende eller annen verdi gir 403, også med gyldig token eller testbypass. Påvirker ikke `/arkiv`.                                                                                                                                                                                                                                                        |
| `DB_INIT_ALLOWED` | `"true"` godkjenner kall til `/initDb`. Brukes for å opprette databasen hvis den ikke allerede finnes i clusteret. Åpne [![init (dev)](https://img.shields.io/badge/DEV-init-2ea043)](https://sf-arkiv-jobbsporet.intern.dev.nav.no/internal/initDb) eller [![init (prod)](https://img.shields.io/badge/PROD-init-blue)](https://sf-arkiv-jobbsporet.intern.nav.no/internal/initDb) for å opprette databasen. |
| `DUMMY_READ_TO_TEST_READINESS` | Virker kun i dev. `"true"` gjør at appen kjører et hente-kall for data med `kilde` `"test"` før podden markeres som ready for Kubernetes. Sikrer at databasetilkoblingen er etablert.                                                                                                                                                                                                                         |

Endepunkter som viser hvilken konfigurasjon som er aktiv:

[![activeConfig (dev)](https://img.shields.io/badge/DEV-activeConfig-2ea043)](https://sf-arkiv-jobbsporet.intern.dev.nav.no/internal/activeConfig)
[![activeConfig (prod)](https://img.shields.io/badge/PROD-activeConfig-blue)](https://sf-arkiv-jobbsporet.intern.nav.no/internal/activeConfig)

## Diagnostics
En del statistikk kan ses her:

[![dbSize (dev)](https://img.shields.io/badge/DEV-dbSize-2ea043)](https://sf-arkiv-jobbsporet.intern.dev.nav.no/internal/dbSize) [![dbGrowth (dev)](https://img.shields.io/badge/DEV-dbGrowth-2ea043)](https://sf-arkiv-jobbsporet.intern.dev.nav.no/internal/dbGrowth)

[![dbSize (prod)](https://img.shields.io/badge/PROD-dbSize-blue)](https://sf-arkiv-jobbsporet.intern.nav.no/internal/dbSize) [![dbGrowth (prod)](https://img.shields.io/badge/PROD-dbGrowth-blue)](https://sf-arkiv-jobbsporet.intern.nav.no/internal/dbGrowth)

## Metrikker

Appen eksponerer disse tellerne til Prometheus:

[![Prometheus (dev)](https://img.shields.io/badge/DEV-Prometheus-2ea043?logo=prometheus&logoColor=white)](https://sf-arkiv-jobbsporet.intern.dev.nav.no/internal/prometheus)
[![Prometheus (prod)](https://img.shields.io/badge/PROD-Prometheus-blue?logo=prometheus&logoColor=white)](https://sf-arkiv-jobbsporet.intern.nav.no/internal/prometheus)

| Metrikk | Betydning |
|---|---|
| `request_arkiv` | Antall kall til `POST /arkiv`, uavhengig av om de godkjennes. |
| `request_hente` | Antall kall til `POST /hente`, også når henting er slått av. |
| `inserted_entries` | Antall arkivposter som er lagret i databasen. |
| `issues` | Antall feil: ugyldig JSON til `/arkiv`, midlertidig databasefeil og uventede feil. Ugyldig JSON til `/hente` teller også. |
| `arkiv_item` | Antall lagrede arkivposter, fordelt på labels: `opprettet_av`, `kilde`, `tema` og `konfidentiellt`. |
