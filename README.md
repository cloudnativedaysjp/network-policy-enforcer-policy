# network-policy-enforcer-policy

Centrally-managed lateral-movement-baseline policy distributed to all
[`network-policy-enforcer`](https://github.com/cloudnativedaysjp/network-policy-enforcer)
DaemonSets in the fleet.

`policy.json` is the single source of truth maintained by the InfoSec team.
Each enforcer Pod fetches it from the raw URL on startup and on every
`REFRESH_INTERVAL` cycle (default 30s), so updates merged here propagate to
all clusters within one cycle without a DaemonSet restart.

## Source of truth

```
https://raw.githubusercontent.com/cloudnativedaysjp/network-policy-enforcer-policy/refs/heads/main/policy.json
```

The default `POLICY_URL` baked into the enforcer image points here. Operators
can override `POLICY_URL` per-cluster, but the canonical baseline lives in
this repository and is what gets reviewed for compliance audits.

## Updating the policy

Changes to `policy.json` require InfoSec review.

- **Do not** edit policy files in-place on deployed clusters or attempt to
  override via ConfigMap mount — the next refresh cycle will overwrite
  local changes from `POLICY_URL`.
- Open a PR against this repository with the proposed change.
- After merge to `main`, every enforcer in the fleet picks up the new policy
  on its next refresh cycle.

## Schema

See the [enforcer README](https://github.com/cloudnativedaysjp/network-policy-enforcer#policy-fields)
for the full policy schema reference.
