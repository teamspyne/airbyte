# Spyne fork of Airbyte

This repository (`teamspyne/airbyte`) is Spyne's fork of [`airbytehq/airbyte`](https://github.com/airbytehq/airbyte).
It carries a few changes to the Facebook Marketing and TikTok Marketing source connectors on top of upstream.

## Branches

| Branch | Content | Rule |
|---|---|---|
| `spyne` (default) | Upstream + Spyne changes | All Spyne work lands here. Upstream is brought in with a **merge**, never a rebase. |
| `master` | Exact copy of `airbytehq/airbyte` `master` | Never commit to it. It only moves by fast-forward to upstream. |
| `backup/master-2026-11-04` | The old `master` (Spyne commits on an outdated upstream), before the 2026-11-04 resync | Keep as a safety net; do not build on it. |

## Spyne changes

Everything that differs from upstream, outside `.github/`:

### source-facebook-marketing
- `ads_insights` stream: adds `video_thruplay_watched_actions` to the built-in schema
  (`source_facebook_marketing/schemas/ads_insights.json`), so it is requested by default.
- Unit tests updated accordingly (`unit_tests/test_base_insight_streams.py`,
  `unit_tests/integration/test_ads_insights_action_product_id.py`).

### source-tiktok-marketing
- New `smart_plus_ads` stream (`GET smart_plus/ad/get/`): metadata of the parent-level Smart+ Ad
  and its `creative_list`, which links each creative (`smart_plus_creative_id`) to the `ad_id`
  of the `ads` stream. Partitioned by `SmartPlusAdGroupIdsPartitionRouter` in `components.py`
  (Smart+ ad groups only, batched by advertiser). It uses the shared requester error handler.
- `ads_reports_daily`: about 130 extra conversion metrics in `report_metrics`
  (custom conversions, parity with `custom-connectors`).
- `Dockerfile` to build the connector image with our `manifest.yaml` and `components.py`
  on top of `source-declarative-manifest`. Its `FROM` must match `baseImage` in `metadata.yaml`.
- `integration_tests/configured_catalog.json`, `acceptance-test-config.yml`, docs and the
  stream count in `unit_tests/test_source.py` updated for the new stream.

### .github
- `.github/workflows/` and `.github/actions/` are deleted: no GitHub Actions run on this fork.
  The rest of `.github/` follows upstream.

To list the exact diff at any time:

```bash
git fetch origin
git diff origin/master origin/spyne --stat -- . ':!.github'
```

## Updating from upstream

Run this regularly (at least monthly) so that the merges stay small.

```bash
# One-time setup
git remote add upstream https://github.com/airbytehq/airbyte.git

# 1. Sync master with upstream (fast-forward, no force needed)
git fetch upstream master && git fetch origin
git push origin upstream/master:master

# 2. Merge master into spyne
git checkout spyne && git pull
git merge origin/master
```

### Conflict rules

| Where | Resolution |
|---|---|
| `.github/workflows/`, `.github/actions/` | Keep them deleted, including new upstream files: `git rm -r .github/workflows .github/actions` |
| `metadata.yaml` of both connectors | Take upstream (`git checkout --theirs <file>`): version, CDK and base image follow upstream |
| TikTok `manifest.yaml`, `report_metrics` lists | Union of upstream's list and ours, no duplicates (`unit_tests/test_conversion_metrics.py` fails on duplicates) |
| TikTok `Dockerfile` | Set `FROM` to the `baseImage` of the merged `metadata.yaml` |
| TikTok docs changelog | Upstream's table; keep our `spyne-fork` row at the top |
| Anything else | Take upstream, unless it is clearly one of the Spyne changes listed above |

If a conflict changes the behavior of a Spyne change, decide explicitly instead of guessing.
Start from upstream and re-apply our intent on top (read the original commit with `git show <sha>`).

```bash
# No conflict markers left
git grep -n '^<<<<<<<\|^>>>>>>>'
```

### Tests

```bash
(cd airbyte-integrations/connectors/source-facebook-marketing && poetry install && poetry run pytest unit_tests)
(cd airbyte-integrations/connectors/source-tiktok-marketing/unit_tests && poetry install && poetry run pytest .)

# The test runs leave files behind; do not commit them
git checkout -- airbyte-integrations/connectors/source-facebook-marketing/unit_tests/test_migrations
rm -rf airbyte-integrations/connectors/source-tiktok-marketing/unit_tests/REQUEST_CACHE_PATH
```

Tests that usually need updating after a merge:
- TikTok `unit_tests/test_source.py::test_source_streams`: upstream's stream count + 1 (`smart_plus_ads`).
- Facebook `unit_tests/integration/test_ads_insights_action_product_id.py`: `_JOB_START_FIELDS`
  must follow the order of `ads_insights.json`, including `video_thruplay_watched_actions`.

### Push

```bash
git push origin spyne
```
