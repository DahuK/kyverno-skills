---
name: kyverno-cli-audit
description: "Audit Kubernetes resources for Kyverno policy violations using kubectl-kyverno CLI. Use when: user wants to check compliance, validate resources against policies, or generate violation reports BEFORE deploying policies. NOT for: actually deploying policies to cluster (use deploy-policies), checking if Kyverno is installed (use check-kyverno), or viewing existing violations from PolicyReports (use show-violations). This is a dry-run tool that does NOT modify the cluster."
metadata:
  openclaw:
    emoji: "🔍"
    requires:
      bins: ["kubectl-kyverno"]
---

# kyverno-cli-audit

Audit cluster resources for policy violations using Kyverno CLI. Dry-run only - does not deploy policies or modify the cluster.

## When to Use

✅ **USE this skill when:**
- User wants to validate resources against policies before deployment
- Need to check compliance in CI/CD pipelines
- Testing policy changes without affecting the cluster
- Generating policy violation reports
- Comparing multiple clusters for compliance
- Pre-deployment validation in staging environments

## When NOT to Use

❌ **DON'T use this skill when:**
- User wants to actually deploy policies (use deploy-policies instead)
- Checking if Kyverno is installed (use check-kyverno instead)
- Viewing violations from existing PolicyReports (use show-violations instead)
- Need to install Kyverno itself (use install-kyverno instead)
- User asks for help with Kyverno installation (use kyverno-help instead)

## Commands

### Basic audit commands

```bash
# Audit default namespace with pod-security policies
kubectl kyverno apply --policy-report --cluster --namespace default policies/pod-security.yaml

# Audit all namespaces
kubectl kyverno apply --policy-report --cluster --namespace all policies/

# Audit specific context
kubectl kyverno apply --policy-report --cluster --namespace production --context prod-cluster policies/
```

### Policy set selection

```bash
# Pod Security Standards only
kubectl kyverno apply --policy-report --cluster policies/pod-security.yaml

# RBAC Best Practices
kubectl kyverno apply --policy-report --cluster policies/rbac-best-practices.yaml

# Kubernetes Best Practices
kubectl kyverno apply --policy-report --cluster policies/kubernetes-best-practices.yaml

# All policies combined
kubectl kyverno apply --policy-report --cluster policies/
```

## Parameters

- **policySets** - Policy set to use:
  - `pod-security` - Pod Security Standards (4 policies)
  - `rbac-best-practices` - RBAC security (4 policies)
  - `kubernetes-best-practices` - K8s best practices (5 policies)
  - `all` - All policies combined (default)
- **namespace** - Target namespace (default: `default`, use `all` for all namespaces)
- **gitBranch** - Git branch for policies (default: `main`)
- **namespace_exclude** - Namespaces to exclude (default: `kube-system,kyverno`)
- **context** - Kubernetes context (optional, uses current if not specified)

## Policy Sets Details

### pod-security (4 policies)
- **disallow-capabilities** - Disallow capabilities beyond allowed list (high)
- **disallow-host-namespaces** - Disallow sharing host namespaces (high)
- **disallow-privileged-containers** - Disallow privileged mode (high)
- **require-run-as-nonroot** - Require non-root execution (medium)

### rbac-best-practices (4 policies)
- **restrict-wildcard-resources** - Block wildcards in roles (medium)
- **restrict-escalation-verbs-roles** - Block impersonate/bind/escalate (high)
- **restrict-automount-sa-token** - Disable SA token auto-mount (medium)
- **restrict-binding-system-groups** - Block system group bindings (high)

### kubernetes-best-practices (5 policies)
- **require-labels** - Require app.kubernetes.io/name label (medium)
- **disallow-latest-tag** - Disallow :latest image tag (medium)
- **require-pod-requests-limits** - Require resource requests/limits (medium)
- **require-probes** - Require liveness/readiness probes (medium)
- **disallow-default-namespace** - Disallow 'default' namespace (medium)

## Output

Returns JSON array of violations:

```json
[
  {
    "policy": "require-run-as-nonroot",
    "rule": "run-as-non-root",
    "message": "Running as root is not allowed",
    "category": "Pod Security Standards (Baseline)",
    "severity": "medium",
    "result": "fail",
    "resources": ["Pod/default/nginx-pod"]
  }
]
```

## Common Workflows

### CI/CD validation

```
# 1. Audit before deployment
kyverno audit --policySets pod-security --namespace staging

# 2. If violations found, fix before merging
# 3. If no violations, proceed with deployment
```

### Multi-environment audit

```
# Audit development
kyverno audit --policySets all --namespace all --context dev-cluster

# Audit staging
kyverno audit --policySets all --namespace all --context staging-cluster

# Audit production
kyverno audit --policySets all --namespace all --context prod-cluster
```

### Compliance reporting

```
# Generate compliance report
kyverno audit --policySets all --namespace all > compliance-report.json

# Focus on high severity
kyverno audit --policySets pod-security --namespace all | jq '.[] | select(.severity == "high")'
```

## Notes

- Requires kubectl-kyverno CLI to be available
- This is a dry-run tool - does NOT modify the cluster
- Returns empty array `[]` if no violations found
- All 13 policies included in `all` set
- Recommended workflow: audit → fix → deploy → monitor
- Use before deploy-policies to preview violations
