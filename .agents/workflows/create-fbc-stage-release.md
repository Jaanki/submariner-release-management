# Create FBC Stage Release

**When:** After FBC catalog updated (Step 9)

**Prerequisite:** Step 9 complete with snapshot names recorded

## Process

Create Release CRs for each OCP version using snapshots from Step 9.

**Repo:** `~/konflux/submariner-release-management`

**Output:** `releases/fbc/4-X/stage/`

**Note:** FBC releases omit `spec.data.releaseNotes` (inherited from ReleasePlan).

## Creating Release YAMLs

Get snapshots from Step 9, or look up recent passing snapshots:

```bash
# Find recent FBC snapshots (within last ~5 days)
oc get snapshots -n submariner-tenant --sort-by=.metadata.creationTimestamp | grep "^submariner-fbc" | tail -10

# Verify test status for each candidate snapshot
oc get snapshot submariner-fbc-4-XX-xxxxx -n submariner-tenant -o jsonpath='{.metadata.annotations.test\.appstudio\.openshift\.io/status}' | jq -r '.[] | "\(.scenario): \(.status)"'
# Both tests must show: TestPassed
```

Agent creates YAML for each OCP version (4-16 through 4-20) with passing snapshot:

```yaml
---
apiVersion: appstudio.redhat.com/v1alpha1
kind: Release
metadata:
  name: submariner-fbc-4-XX-stage-YYYYMMDD-01
  namespace: submariner-tenant
  labels:
    release.appstudio.openshift.io/author: 'dfarrell07'
spec:
  releasePlan: submariner-fbc-release-plan-stage-4-XX
  snapshot: submariner-fbc-4-XX-xxxxx  # From Step 9
```

Replace `4-XX` with OCP version (4-16, 4-17, 4-18, 4-19, 4-20), `YYYYMMDD` with today's date,
`xxxxx` with passing snapshot for that version. Save to:
`releases/fbc/4-XX/stage/submariner-fbc-4-XX-stage-YYYYMMDD-01.yaml`

## Commit

```bash
git add releases/fbc/
git commit -s -m "Add FBC stage releases"
```

User reviews commit, then pushes:

```bash
git push
```

## Apply to Cluster

Agent provides user with commands for each created YAML file:

```bash
make test-remote FILE=releases/fbc/4-XX/stage/submariner-fbc-4-XX-stage-YYYYMMDD-01.yaml
make apply FILE=releases/fbc/4-XX/stage/submariner-fbc-4-XX-stage-YYYYMMDD-01.yaml
make watch NAME=submariner-fbc-4-XX-stage-YYYYMMDD-01
```

## Done When

```bash
# Check cluster
oc get releases -n submariner-tenant | grep "fbc.*stage" | sort

# Check files (expect: 4-16 through 4-20, skip EOL versions)
ls -1 releases/fbc/*/stage/*.yaml
```
