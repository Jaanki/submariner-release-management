# Create FBC Prod Release

**When:** After QE approval (following Step 12)

## Process

Create production Release CRs by copying stage YAMLs and changing 2 fields.

**Repo:** `~/konflux/submariner-release-management`

## Creating Prod YAMLs

Agent creates prod YAML for each OCP version (4-16 through 4-20):

```bash
# For each version:
cp releases/fbc/4-XX/stage/submariner-fbc-4-XX-stage-YYYYMMDD-01.yaml \
   releases/fbc/4-XX/prod/submariner-fbc-4-XX-prod-YYYYMMDD-01.yaml

# Edit: change only these 2 fields
#   metadata.name: stage-YYYYMMDD-01 → prod-YYYYMMDD-01
#   spec.releasePlan: stage-4-XX → prod-4-XX
# Keep spec.snapshot identical
```

## Commit

```bash
git add releases/fbc/
git commit -s -m "Add FBC prod releases"
```

User reviews commit, then pushes:

```bash
git push
```

## Apply to Cluster

Agent provides user with commands for each created YAML file:

```bash
make test-remote FILE=releases/fbc/4-XX/prod/submariner-fbc-4-XX-prod-YYYYMMDD-01.yaml
make apply FILE=releases/fbc/4-XX/prod/submariner-fbc-4-XX-prod-YYYYMMDD-01.yaml
make watch NAME=submariner-fbc-4-XX-prod-YYYYMMDD-01
```

## Done When

```bash
# Check cluster
oc get releases -n submariner-tenant | grep "fbc.*prod" | sort

# Check files (expect: 4-16 through 4-20)
ls -1 releases/fbc/*/prod/*.yaml
```
