# dsh-plugins

A [DeepSeek Harness](https://deepseek-harness.github.io/deepseek-harness/en/guide/quickstart) **plugin market index** (format 1) for the bundles maintained by [ogidimitrov](https://github.com/ogidimitrov).

The index is the single file [`index.json`](index.json):

```text
https://raw.githubusercontent.com/ogidimitrov/dsh-plugins/main/index.json
```

## What it lists

| Bundle | Version | What it does |
|---|---|---|
| [`dsh-peak-cost`](https://github.com/ogidimitrov/dsh-peak-cost) | 1.0.0 | Peak-hours indicator and estimated USD spend for the current session, below the composer |
| [`dsh-laravel-skills`](https://github.com/ogidimitrov/dsh-laravel) | 0.1.0 | Version-aware Laravel engineering skills: stack detection, per-version guidance, SOLID design, stack playbooks |

## Install from this index

Harness reads markets only when the optional market bundle is enabled, so the first time:

1. In the app, open **Plugins** and add/enable the **custom plugin markets** bundle (`@deepseek-ai/dsh-experimental-plugin-market-bundle`). Without it there is no **Discover** tab and no index source field.
2. Open the **Discover** tab and add the index URL above as a source.
3. Search for the plugin and install it. The install goes through the normal bundle install, so the plugin lands in your profile like any other.

Without the market, each plugin installs directly from its repository — either is fine:

```text
github:ogidimitrov/dsh-peak-cost
github:ogidimitrov/dsh-laravel
```

## Format

Format 1 is a plain JSON document validated by the Harness package `@deepseek-ai/dsh-plugin-market-index`. One document lists any number of plugins; each plugin lists one or more exact versions, and each version names where that exact version comes from:

```json
{
  "format": 1,
  "title": { "en": "…" },
  "plugins": [
    {
      "name": "dsh-peak-cost",
      "title": { "en": "Peak Hours & Session Spend" },
      "metadata": { "keywords": ["…"] },
      "versions": [
        {
          "version": "1.0.0",
          "publishTimestamp": 1792785510,
          "source": {
            "type": "git",
            "url": "https://github.com/ogidimitrov/dsh-peak-cost",
            "commit": "659b813b50abc1753758e941b48d8f983a5e13c2"
          }
        }
      ]
    }
  ]
}
```

Rules that matter when editing:

- `version` is exact SemVer; `publishTimestamp` is Unix **seconds**, not milliseconds.
- `source.type` is `git` (requires a full 40-hex commit, `https` URL without `.git` suffix or query) or `npm` (optionally with `registry`).
- A `git` source without a `path` means the package is at the repository root.
- Optional members: `icon`, `author`, `homepage`, `repository`, `description`, `metadata.category`, `metadata.labels`, `metadata.keywords`, `engines.dsh`.
- Category and label ids must be declared in the document's `metadata` first.
- Icons are either a `data:` URL (≤ 32 KiB decoded) or an `https` URL, or a path relative to the index URL.

## Validate before publishing

The validator is the same code the app runs, so a document that passes here is accepted by Discover:

```bash
npm install semver --no-save
curl -sO https://raw.githubusercontent.com/deepseek-ai/deepseek-harness/master/packages/util/plugin-market-index/lib/index.js
node --input-type=module -e "
import { readFile } from 'node:fs/promises';
import { parseMarketIndex, marketInstallTarget } from './index.js';
const doc = JSON.parse(await readFile('index.json', 'utf8'));
const parsed = parseMarketIndex(doc, 'https://raw.githubusercontent.com/ogidimitrov/dsh-plugins/main/index.json');
for (const p of parsed.plugins) for (const v of p.versions) {
  console.log(p.name, v.version, '->', marketInstallTarget(p.name, v).spec);
}
"
```

A published index is cached by the app for ten minutes (`cacheTtlMs`); the Discover tab can force a refresh.

## License

MIT
