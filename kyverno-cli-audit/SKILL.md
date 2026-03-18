---
name: kyverno-cli-audit
description: "Audit Kubernetes resources for Kyverno policy violations using Kyverno CLI. Use when: user wants to check compliance, validate resources against policies, or generate violation reports BEFORE deploying policies. NOT for: actually deploying policies to cluster (use deploy-policies), checking if Kyverno is installed (use check-kyverno), or viewing existing violations from PolicyReports (use show-violations). This is a dry-run tool that does NOT modify the cluster."
metadata:
openclaw:
emoji: "🔍"
requires:
bins: ["kyverno"]
---

# kyverno-cli-audit

Audit cluster or local resources for policy violations using **Kyverno CLI**. The CLI validates and tests policy behavior on resources prior to adding them to a cluster; it is purpose-built for CI/CD and local validation. Dry-run only — does not deploy policies or modify the cluster.

**Reference:** [Kyverno CLI — Kyverno Docs](https://kyverno.io/docs/subprojects/kyverno-cli/)

## When to Use

✅ **USE this skill when:**
- User wants to validate resources against policies before deployment
- Need to check compliance in CI/CD pipelines
- Testing policy changes without affecting the cluster
- Generating policy violation reports (Policy Reports / ClusterReport)
- Auditing resources from files, stdin, or live cluster
- Comparing expected vs actual results for policies (apply then interpret)

## When NOT to Use

❌ **DON'T use this skill when:**
- User wants to actually deploy policies (use deploy-policies instead)
- Checking if Kyverno is installed (use check-kyverno instead)
- Viewing violations from existing PolicyReports (use show-violations instead)
- Need to install Kyverno itself (use install-kyverno instead)
- User asks for help with Kyverno installation (use kyverno-help instead)

## Core Command: `kyverno apply`

The main command for auditing is **`kyverno apply`**. It performs a dry run of one or more policies against a set of resources. Use **`--policy-report`** (or **`-p`**) to get a structured policy report.

### Resource sources

- **Local files:** `--resource /path/to/resource.yaml` or `-r`
- **Stdin:** `--resource -` (e.g. `kustomize build . | kyverno apply policy.yaml -r -`)
- **Live cluster:** `--cluster` (uses current `kubectl` context; combine with `-n` for namespace)

### Basic audit commands

```bash
# Audit resources in default namespace (cluster)
kyverno apply policy.yaml --cluster --policy-report -n default

# Audit all resources in cluster
kyverno apply policy.yaml --cluster --policy-report

# Audit local resource files (no cluster)
kyverno apply policy.yaml --resource resource1.yaml --resource resource2.yaml --policy-report

# Multiple policies and resources
kyverno apply policy1.yaml policy2.yaml -r resource1.yaml -r resource2.yaml --policy-report
```

### Policy report + cluster (official combinations)

| Policy       | Resource   | Cluster | Namespace     | Meaning                                              |
|-------------|------------|---------|---------------|------------------------------------------------------|
| policy.yaml | -r res.yaml| false  | —             | Apply policy to resources in res.yaml                |
| policy.yaml | -r name    | true   | —             | Apply to resource named `name` in cluster            |
| policy.yaml | —          | true   | —             | Apply to all matching resources in cluster           |
| policy.yaml | -r name    | true   | -n=ns         | Apply to resource `name` in namespace `ns`            |
| policy.yaml | —          | true   | -n=ns         | Apply to all matching resources in namespace `ns`     |

```bash
# Example: audit all Pods in default namespace
kyverno apply policies/pod-security.yaml --cluster --policy-report -n default

# Example: audit specific resources by name in cluster
kyverno apply policy.yaml -r nginx1 -r nginx2 --cluster --policy-report

# Example: audit from local YAML only (no cluster)
kyverno apply policy.yaml -r deployment.yaml --policy-report
```

### Policy exceptions and variables

```bash
# With policy exception
kyverno apply policy.yaml -r resource.yaml --exception exception.yaml

# With values file (variables, namespaceSelector, etc.)
kyverno apply policy1.yaml policy2.yaml -r r1.yaml -r r2.yaml -f values.yaml --policy-report
```

### Audit as warnings and exit codes

- **`--audit-warn`**: Treat policies with `failureAction: Audit` as warnings; can yield non-zero exit code when only audit failures exist.
- **`--warn-exit-code N`**: When used with `--audit-warn`, exit with code N when there are warnings (e.g. for CI).
- **`--warn-no-pass`**: Exit with warning code when no objects satisfy the policy (e.g. empty resources).

```bash
kyverno apply policy.yaml -r resource.yaml --audit-warn --warn-exit-code 3
```

## Policy types supported (apply)

- **Kyverno ClusterPolicy / Policy** (validate, mutate, generate — reports for validate)
- **ValidatingAdmissionPolicy** (+ binding)
- **MutatingAdmissionPolicy** (+ binding)
- **ValidatingPolicy** (Kyverno CEL; report kind: ClusterReport)

For **ValidatingPolicy** (or policies using cluster lookups like `resource.Get()`), use **`--context-file`** (or **`-p`** in some docs) when not using `--cluster`, so the CLI can resolve context (e.g. ConfigMaps) from a file.

## Parameters (for automation / wrappers)

- **policySets** — Which policy set to use, e.g.:
  - `pod-security` — Pod Security Standards
  - `rbac-best-practices` — RBAC
  - `kubernetes-best-practices` — K8s best practices
  - `all` — All of the above (default when applicable)
- **namespace** — Target namespace (`default` or `all`); with `--cluster` use `-n` or `-n=all`.
- **gitBranch** — Git branch for policy repo (default: `main`)
- **namespace_exclude** — Namespaces to exclude (e.g. `kube-system,kyverno`)
- **context** — Kubernetes context (ensure `kubectl` context is set before `kyverno apply --cluster`)

## Policy sets (example groupings)

### pod-security
- disallow-capabilities, disallow-host-namespaces, disallow-privileged-containers, require-run-as-nonroot

### rbac-best-practices
- restrict-wildcard-resources, restrict-escalation-verbs-roles, restrict-automount-sa-token, restrict-binding-system-groups

### kubernetes-best-practices
- require-labels, disallow-latest-tag, require-pod-requests-limits, require-probes, disallow-default-namespace

## Output

With **`--policy-report`**, output is a report (e.g. ClusterPolicyReport or ClusterReport for ValidatingPolicy), for example:

```yaml
# ClusterPolicyReport (validate policies)
apiVersion: wgpolicyk8s.io/v1alpha1
kind: ClusterPolicyReport
metadata:
  name: clusterpolicyreport
results:
  - message: "Validation rule 'validate-resources' succeeded."
    policy: require-pod-requests-limits
    resources: [{ apiVersion: v1, kind: Pod, name: nginx1, namespace: default }]
    rule: validate-resources
    scored: true
    status: pass
  - message: "Validation error: ..."
    policy: require-pod-requests-limits
    resources: [{ apiVersion: v1, kind: Pod, name: nginx2, namespace: default }]
    rule: validate-resources
    scored: true
    status: fail
summary:
  error: 0
  fail: 1
  pass: 1
  skip: 0
  warn: 0
```

You can parse this (e.g. with `jq`/yq) to produce a JSON array of violations for downstream tools.

## Common workflows

### CI/CD validation

```bash
# 1. Audit before deploy (cluster)
kyverno apply policies/pod-security.yaml --cluster --policy-report -n staging

# 2. Or from built manifests (no cluster)
kustomize build overlays/staging | kyverno apply policies/pod-security.yaml -r - --policy-report

# 3. Fail CI on failures or warnings
kyverno apply policy.yaml -r manifest.yaml --audit-warn --warn-exit-code 3
```

### Multi-environment audit

```bash
# Use kubectl context then apply
kubectl config use-context dev-cluster
kyverno apply policies/ --cluster --policy-report -n all

kubectl config use-context prod-cluster
kyverno apply policies/ --cluster --policy-report -n all
```

### Compliance report from cluster

```bash
kyverno apply policy.yaml --cluster --policy-report > compliance-report.yaml
# Or filter by severity from report (depends on report schema)
```

## Installation

Kyverno CLI is a **standalone binary** (separate from the in-cluster Kyverno controller). Prefer standalone when using with kustomize.

```bash
# Homebrew
brew install kyverno

# Krew (kubectl plugin)
kubectl krew install kyverno
kubectl kyverno version

# Manual (example Linux amd64)
# curl -LO https://github.com/kyverno/kyverno/releases/download/v1.12.0/kyverno-cli_v1.12.0_linux_x86_64.tar.gz
# tar -xvf kyverno-cli_*.tar.gz && sudo cp kyverno /usr/local/bin/

# Verify
kyverno version
```

## Notes

- Requires the **`kyverno`** binary (CLI), not the in-cluster Kyverno image.
- **Dry-run only:** `kyverno apply` does not modify cluster or deploy policies.
- Use **`--policy-report`** to get structured pass/fail/skip/warn and summary counts.
- For policies that need cluster lookups (e.g. `resource.Get()`), either run with **`--cluster`** or supply **`--context-file`** when testing locally.
- Recommended flow: audit (apply + policy-report) → fix → deploy policies → monitor.
- Optional: use **`kyverno test`** with `kyverno-test.yaml` to assert expected pass/fail/skip for policy tests; use this skill for ad-hoc or report-oriented audits.
