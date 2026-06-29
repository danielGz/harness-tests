# spot-unified-test

Minimal Harness Code repo content for testing **unified Spot**:

- `elastigroup.json` → manifest code fetch → `<+service.manifests.spot.config>`
- `scripts/startup.sh` → startup-script code fetch → `<+service.startupScript.content>`

## Expression placeholders in `elastigroup.json`

Account-specific values use v0 `<+ >` syntax so you can push to **GitHub** safely and resolve at runtime via pipeline variables + artifact.

| JSON path | Expression | Was (hardcoded) | Supply at runtime via |
|-----------|------------|-----------------|------------------------|
| `group.name` | `<+pipeline.variables.app_name>` | `placeholder` | Pipeline/stage variable |
| `group.compute.availabilityZones[0].name` | `<+pipeline.variables.availability_zone>` | `us-east-1a` | Pipeline variable (e.g. `us-east-1a`) |
| `group.compute.availabilityZones[0].subnetIds[0]` | `<+pipeline.variables.subnet_id>` | `subnet-4c0a1873` | Pipeline variable |
| `group.compute.launchSpecification.securityGroupIds[0]` | `<+pipeline.variables.security_group_id>` | `sg-0ab7a1dcd33864fbe` | Pipeline variable |
| `group.compute.launchSpecification.imageId` | `<+artifact.image>` | `ami-0e8b27000f99d833c` | Service AMI artifact (no variable needed in CD) |

**Left as literals** (not account-specific): instance types, strategy, capacity shell, `product`, monitoring flags.

Example pipeline variables for a render/deploy test:

```yaml
variables:
  - name: app_name
    type: String
    value: spot-unified-test
  - name: availability_zone
    type: String
    value: us-east-1a
  - name: subnet_id
    type: String
    value: subnet-xxxxxxxx   # your subnet
  - name: security_group_id
    type: String
    value: sg-xxxxxxxx       # your SG
```

## Push to Harness Code (project: dan)

1. Harness UI → **Code** → create repo `spot-unified-test` (or import).
2. From this folder:

```bash
cd ~/develop/spot-unified-test
git init
git add elastigroup.json scripts/startup.sh README.md
git commit -m "Add unified Spot manifest and startup script fixtures"
# Add Harness Code remote from UI, then:
git push -u origin main
```

## Unified service snippet

```yaml
service:
  uses: spot
  with:
    manifests:
      sources:
        - id: spot
          uses: spot
          with:
            store:
              uses: code
              with:
                type: branch
                branch: main
                repo: spot-unified-test
                paths:
                  - elastigroup.json
    artifacts:
      primary:
        uses: ami
        with:
          connector: <aws-connector>
          region: us-east-1
          filters:
            ami-image-id: ami-0e8b27000f99d833c
          regex: "*"
    startup-script:
      store:
        uses: code
        with:
          type: branch
          branch: main
          repo: spot-unified-test
          paths:
            - scripts/startup.sh
```

## Test scope

| Goal | Manifest | Startup script |
|------|----------|----------------|
| Startup-script fetch only | inline JSON in service | code (this repo) |
| Both code fetches | code (this repo) | code (this repo) |
| Full Spot deploy | code + valid subnet/SG | code or inline |
