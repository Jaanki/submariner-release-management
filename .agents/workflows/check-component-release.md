# Check Component Release Build

**Used by:** After Step 10 (stage), After Step 16 (prod)

## Process

Monitor component release pipeline execution and verify successful completion.

## Check Release Status

```bash
# Check component release (replace with actual release name)
RELEASE="submariner-0-X-Y-{stage,prod}-YYYYMMDD-01"

STATUS=$(oc get release "$RELEASE" -n submariner-tenant -o jsonpath='{.status.conditions[?(@.type=="Released")].status}')
REASON=$(oc get release "$RELEASE" -n submariner-tenant -o jsonpath='{.status.conditions[?(@.type=="Released")].reason}')

echo "$RELEASE: $STATUS ($REASON)"
```

Expected: `True (Succeeded)`. If shows `False (Failed)`:

## Debug Failed Release

When a release shows `False (Failed)`, investigate to determine if it's an infra failure (retry) or code issue (fix required).

### 1. Get Full Conditions

```bash
# Replace with actual failed release name
RELEASE="submariner-0-X-Y-{stage,prod}-YYYYMMDD-01"

# View all conditions
oc get release "$RELEASE" -n submariner-tenant -o jsonpath='{.status.conditions}' | jq '.[] | {type: .type, status: .status, reason: .reason, message: .message}'
```

Key conditions:

- `Released`: Overall status
- `ManagedPipelineProcessed`: Managed pipeline execution (most failures here)
- `TenantPipelineProcessed`: Tenant pipeline execution

### 2. Check Pipeline Task Details

```bash
# ManagedPipelineProcessed shows task counts
oc get release "$RELEASE" -n submariner-tenant -o jsonpath='{.status.conditions[?(@.type=="ManagedPipelineProcessed")]}' | jq '.'
```

Example failure: `"message": "Tasks Completed: 6 (Failed: 1, Cancelled 0), Skipped: 11"`

### 3. Get Konflux UI Log Link

```bash
# Extract log URL from annotations (opens in Konflux UI)
oc get release "$RELEASE" -n submariner-tenant -o jsonpath='{.metadata.annotations.pac\.test\.appstudio\.openshift\.io/log-url}'
```

Opens pipeline run details in Konflux UI at `https://konflux-ui.apps.kflux-prd-rh02.0fk9.p1.openshiftapps.com/`.

### 4. Determine Retry vs Fix

**Retry (infra failure):**

- Transient errors (network, cluster capacity, task timeouts)
- Same code succeeded in stage (for prod releases)

**Investigate/Fix (code/config issue):**

- Snapshot doesn't exist or has test failures
- Consistent failures across retries
- Code/config changes needed

## Retry Failed Release

If determined to be infra failure, agent prepares retry, then user applies.

### Agent Prepares Retry

```bash
# Example: Retry stage release (increment -01 to -02)
cd releases/0.X/stage

# Rename file
mv submariner-0-X-Y-stage-YYYYMMDD-01.yaml \
   submariner-0-X-Y-stage-YYYYMMDD-02.yaml

# Update metadata.name inside YAML
sed -i 's/name: submariner-0-X-Y-stage-YYYYMMDD-01/name: submariner-0-X-Y-stage-YYYYMMDD-02/' \
   submariner-0-X-Y-stage-YYYYMMDD-02.yaml

# Commit retry YAML
git add submariner-0-X-Y-stage-YYYYMMDD-02.yaml
git commit -s -m "Retry 0.X.Y stage release (-02)

Previous -01 failed due to infra issue (not released)"
```

### User Applies Retry

```bash
make test-remote FILE=releases/0.X/stage/submariner-0-X-Y-stage-YYYYMMDD-02.yaml
make apply FILE=releases/0.X/stage/submariner-0-X-Y-stage-YYYYMMDD-02.yaml
```

After apply, agent returns to **Check Release Status** section to verify retry succeeded.

## Done When

- Release pipeline completed successfully
- Bundle available in registry (stage or prod)
- All tasks passed
- Ready for next step

**Stage:** Bundle now available in stage registry for Step 11 (catalog update).

**Prod:** Bundle now available in prod registry for Step 17 (FBC prod releases).
