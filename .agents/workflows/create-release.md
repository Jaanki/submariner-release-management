## Create Downstream Release

Create ReleasePlanAdmission for stage or prod release.

**Repo:** https://github.com/dfarrell07/submariner-release-management (this repo)
**Local:** `~/konflux/submariner-release-management`

---

**Stage:** After bundle SHAs updated → `releases/0.20/stage/`

**Prod:** After QE approval → `releases/0.20/prod/`

---

## Finding Snapshots

```bash
oc get snapshots -n submariner-tenant --sort-by=.metadata.creationTimestamp
oc get snapshot <name> -n submariner-tenant \
  -o jsonpath='{.metadata.annotations.test\.appstudio\.openshift\.io/status}' \
  | jq
```

Look for snapshots where all tests show `"status": "TestPassed"`.

## Creating Release YAML

1. Copy existing YAML from same environment (stage or prod)
2. Update `metadata.name`, `spec.snapshot`, `spec.data.releaseNotes` (type/cves/issues)
3. `make test-remote` then `make apply FILE=...`
4. `make watch NAME=...`

## Requirements

- Advisory types: RHSA (security, must have ≥1 CVE), RHBA (bug fix), RHEA (enhancement)
- Component names must have version suffix: `lighthouse-coredns-0-20`
- Issue IDs:
  - Jira: `PROJECT-12345` (source: `issues.redhat.com`)
  - Bugzilla: `1234567` (source: `bugzilla.redhat.com`)

## Validation

Releases: `make test` | `make test-remote` (requires cluster)

Markdown: `npx markdownlint-cli2 "**/*.md"`
