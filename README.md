# network-policy-enforcer-policy

Centrally-managed `egress-rate-shield-baseline` policy distributed to all
[`network-policy-enforcer`](https://github.com/cloudnativedaysjp/network-policy-enforcer)
DaemonSets in the fleet.

`policy.json` is the single source of truth maintained by the Platform / SecOps
team. Each enforcer Pod fetches it from the raw URL on startup and on every
`REFRESH_INTERVAL` cycle (default 30s), so updates merged here propagate to all
clusters within one cycle without a DaemonSet restart.

## Source of truth

```
https://raw.githubusercontent.com/cloudnativedaysjp/network-policy-enforcer-policy/refs/heads/main/policy.json
```

The default `POLICY_URL` baked into the enforcer image points here. Operators
can override `POLICY_URL` per-cluster, but the canonical baseline lives in
this repository and is what gets reviewed for compliance audits.

## Schema (v2.5.0-rc.0+)

| Field | Type | Description |
|---|---|---|
| `source_cidrs` | `[]string` | rate-limit 適用対象の送信元 CIDR (= cluster 内 Pod CIDR を想定) |
| `destination_cidrs` | `[]string` | rate-limit を適用する宛先 CIDR の列挙 |
| `destination_exclude_cidrs` | `[]string` | `destination_cidrs` から除外する宛先網。rate-limit 対象から外したい網 (例: cluster 内向けトラフィックなど) を指定 |
| `rate_limit_pps` | `int` | source Pod IP ごとの `destination_cidrs` 向けトラフィックの pps しきい値 |
| `peer_syn_rate_pps` | `int` | (source, destination peer) ごとの SYN レート上限 (pps) |
| `on_exceed` | `string` | しきい値超過時の動作 (`drop` / `log` 等) |

最終的なマッチ条件:

```
src ∈ source_cidrs
∧ dst ∈ destination_cidrs
∧ dst ∉ destination_exclude_cidrs
```

いずれにも該当しないトラフィックは shield の対象外として素通しされます。

## Updating the policy

Changes to `policy.json` require Platform / SecOps review.

- **Do not** edit policy files in-place on deployed clusters or attempt to
  override via ConfigMap mount — the next refresh cycle will overwrite
  local changes from `POLICY_URL`.
- Open a PR against this repository with the proposed change.
- After merge to `main`, every enforcer in the fleet picks up the new policy
  on its next refresh cycle.

> **Note (since enforcer v2.5.0-rc.0):** Operators can override individual
> policy fields per-DaemonSet via environment variables
> (`POLICY_SOURCE_CIDRS`, `POLICY_DESTINATION_CIDRS`,
> `POLICY_DESTINATION_EXCLUDE_CIDRS`, `POLICY_RATE_LIMIT_PPS`,
> `POLICY_PEER_SYN_RATE_PPS`).
> Env values take precedence over `policy.json` for the matching fields.
> See enforcer issue
> [#1](https://github.com/cloudnativedaysjp/network-policy-enforcer/issues/1)
> for the discussion that led to this feature.

## Schema reference

See the [enforcer README](https://github.com/cloudnativedaysjp/network-policy-enforcer#policy-fields)
for the full policy schema reference.
