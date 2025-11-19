# Update FBC with Stage Release

**When:** After stage release completes (Step 8)

## Process

**TODO:** Update catalog in FBC repo (`~/konflux/submariner-operator-fbc`) with bundle from Step 8.

See FBC repo for detailed catalog editing workflow (to be documented).

## Done When

For each OCP version (4-16 through 4-20):

```bash
# 1. Check latest snapshot created after catalog update
oc get snapshots -n submariner-tenant --sort-by=.metadata.creationTimestamp | grep "^submariner-fbc-4-XX" | tail -1

# 2. Verify both tests pass
SNAPSHOT="submariner-fbc-4-XX-xxxxx"
oc get snapshot $SNAPSHOT -n submariner-tenant -o jsonpath='{.metadata.annotations.test\.appstudio\.openshift\.io/status}' | jq -r '.[] | "\(.scenario): \(.status)"'
# Both must show: TestPassed
```

Record snapshot names for all OCP versions - needed for Step 10.
