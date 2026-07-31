# SynBioHub Docker topologies

Clone this repository and run one of the Compose combinations below. The search-backend comparison deliberately enables strict Explorer mode: `SBH_EXPLORER_FALLBACK=false` makes an unavailable or failing Explorer endpoint visible instead of silently retrying the same query against the triplestore.

| Topology | SynBioHub store | SynBioHub Explorer endpoint | Compose files |
| --- | --- | --- | --- |
| `virtuoso-explorer` | Virtuoso | SBOLExplorer | `docker-compose.yml` + `docker-compose.explorer.yml` |
| `sboldb-explorer` | sbol-db (store only) | SBOLExplorer | `docker-compose.sboldb.yml` + `docker-compose.sboldb-store-only.yml` + `docker-compose.explorer.yml` |
| `sboldb-native` | sbol-db | sbol-db compatibility listener | `docker-compose.sboldb.yml` + `docker-compose.sboldb-search.yml` |

## Virtuoso and SBOLExplorer

```sh
docker compose \
  -f docker-compose.yml \
  -f docker-compose.explorer.yml \
  up -d
```

## sbol-db and SBOLExplorer

```sh
docker compose \
  -f docker-compose.sboldb.yml \
  -f docker-compose.sboldb-store-only.yml \
  -f docker-compose.explorer.yml \
  up -d
```

sbol-db answers on the `virtuoso` network alias at port 8890 and exposes the HTTP SPARQL endpoints expected by SynBioHub, so the SynBioHub triplestore configuration is unchanged. The store-only overlay disables sbol-db's embedded worker and native search-maintenance deployment because the separate SBOLExplorer container owns that role; this prevents duplicate, unused index rebuilds during store writes.

## sbol-db as both store and Explorer

```sh
docker compose \
  -f docker-compose.sboldb.yml \
  -f docker-compose.sboldb-search.yml \
  up -d
```

The search overlay starts a second, network-internal sbol-db listener at `http://explorer:13162/`. Both listeners share the same process, database, search index, maintenance workers, and shutdown lifecycle. Port 13162 is not published to the host in this topology. The default sbol-db image includes both the compatibility listener and a bundled Sequence Ontology snapshot. Before the listener starts, a one-shot service loads that snapshot so native text indexing can expand SBOL role IRIs to SO labels and synonyms without a network download. The health check requires that load to be present.

Worker-enabled sbol-db comparison rows default to one maintenance worker so full lexical and vector reconciliations cannot compete for memory. Set `SBOLDB_WORKER_CONCURRENCY` explicitly when evaluating a larger production allocation; the external-SBOLExplorer row uses the store-only overlay and starts no sbol-db worker.

## Reproducible image overrides and isolated runs

Every image and published port can be overridden without editing YAML. For example:

```sh
COMPOSE_PROJECT_NAME=sbh-search-eval \
SYNBIOHUB_IMAGE=synbiohub/synbiohub:sbol-db-and-explorer \
SBOLDB_IMAGE=your-sbol-db-image \
SYNBIOHUB_PORT=17777 \
TRIPLESTORE_PORT=18890 \
docker compose \
  -f docker-compose.sboldb.yml \
  -f docker-compose.sboldb-search.yml \
  up -d
```

Use a distinct `COMPOSE_PROJECT_NAME` for every comparison row so volumes and networks cannot leak state between backends. Record immutable image digests in test reports even when convenient local tags are supplied on the command line. SBOLExplorer is intentionally run as `linux/amd64` because its checked-in vsearch executable is x86-64-only; override `SBOLEXPLORER_PLATFORM` only after replacing that binary with a native or multi-architecture build.

On initial SynBioHub setup, use `https://synbiohub.org/` as the URI prefix when testing production-style URL spoofing.
