---
name: check-kyverno
description: "Check if Kyverno is installed and running in the cluster. Use when: user wants to verify Kyverno installation status, check if controllers are healthy, or confirm Kyverno is ready before deploying policies. NOT for: installing Kyverno (use install-kyverno), deploying policies (use deploy-policies), or getting help (use kyverno-help). Performs two-tier detection: Helm chart check and controller deployment verification."
metadata:
  openclaw:
    emoji: "✅"
    requires:
      bins: ["helm", "kubectl"]
---

# check-kyverno

Check if Kyverno is installed in the cluster. Performs two-tier detection: Helm chart check and controller deployment verification.

## When to Use

✅ **USE this skill when:**
- Verifying Kyverno installation before deploying policies
- Pre-installation check (to avoid conflicts)
- Health monitoring and troubleshooting
- Multi-cluster installation audit
- Confirming all controllers are ready
- Checking after Kyverno upgrade

## When NOT to Use

❌ **DON'T use this skill when:**
- Installing Kyverno (use install-kyverno instead)
- Deploying policies (use deploy-policies instead)
- Getting help with Kyverno (use kyverno-help instead)
- Auditing resources (use kyverno-cli-audit instead)
- Viewing violations (use show-violations instead)
- Need detailed controller logs (use kubectl logs instead)

## Commands

### Check commands

```bash
# Check using Helm
helm list -n kyverno

# Check deployments
kubectl get deployments -n kyverno

# Check pods
kubectl get pods -n kyverno

# Check specific deployment
kubectl get deployment kyverno-admission-controller -n kyverno
```

### Detailed status

```bash
# Check all controllers
for deploy in kyverno-admission-controller kyverno-background-controller kyverno-cleanup-controller kyverno-reports-controller; do
  kubectl get deployment $deploy -n kyverno
done
```

## Parameters

- **namespace** - Namespace to check (default: `kyverno`)
- **context** - Kubernetes context (optional, uses current if not specified)

## Detection Methods

### 1. Helm Chart Check
Checks if Kyverno is installed via Helm:
```bash
helm list -n kyverno
```

### 2. Deployment Check
Verifies presence and readiness of 4 Kyverno controllers:
- kyverno-admission-controller
- kyverno-background-controller
- kyverno-cleanup-controller
- kyverno-reports-controller

## Status Types

### ✅ FULLY INSTALLED
- Helm release found
- All controllers ready
- System is healthy

### ⚠️ PARTIALLY INSTALLED
- Helm release found
- Some controllers not ready
- Check pod logs and events

### ⚠️ RUNNING (non-Helm)
- Controllers ready
- No Helm release
- Manual installation detected

### ❌ NOT INSTALLED
- No Helm release
- Controllers missing
- Run install-kyverno

## Common Workflows

### Pre-installation check

```
# Check before installing
kyverno check

# If not installed, proceed with installation
kyverno install

# Verify after installation
kyverno check
```

### Multi-cluster audit

```
# Check all clusters
kyverno check --context dev-cluster
kyverno check --context staging-cluster
kyverno check --context prod-cluster
```

### Health monitoring

```
# Regular health check
kyverno check --namespace kyverno

# If partially installed, investigate
kubectl logs -n kyverno -l app.kubernetes.io/name=kyverno
kubectl get events -n kyverno
```

### Troubleshooting

```
# Check shows issues
kyverno check

# Get detailed pod status
kubectl describe pods -n kyverno

# Check controller logs
kubectl logs -n kyverno deployment/kyverno-admission-controller
```

## Output

Returns detailed status:
- Helm Chart Check results
- Deployment Check results (4 controllers)
- Summary with installation status
- Action recommendations if not installed

## Notes

- Requires both Helm and kubectl to be available
- Checks both Helm-managed and manually installed Kyverno
- Provides actionable feedback for each scenario
- Can be used in CI/CD pipelines for verification
- Non-zero exit code if not fully installed
- Fast check - usually completes in seconds
