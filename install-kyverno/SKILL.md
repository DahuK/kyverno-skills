---
name: install-kyverno
description: "Install Kyverno policy engine using Helm chart. Use when: user wants to install Kyverno for the first time, upgrade existing Kyverno installation, or reinstall Kyverno in a new cluster. NOT for: deploying individual policies (use deploy-policies), checking if Kyverno is installed (use check-kyverno), or getting help with Kyverno (use kyverno-help). This installs the Kyverno controllers, not the policies themselves."
metadata:
  openclaw:
    emoji: "📦"
    requires:
      bins: ["helm", "kubectl"]
---

# install-kyverno

Install Kyverno using the official Helm chart. Automates the entire installation process including repository setup, chart installation, and verification.

## When to Use

✅ **USE this skill when:**
- First-time Kyverno installation in a cluster
- Upgrading existing Kyverno to newer version
- Installing Kyverno in a new cluster
- Reinstalling or repairing Kyverno
- Changing controller configuration (replicas, features)
- Need to enable/disable specific controllers

## When NOT to Use

❌ **DON'T use this skill when:**
- Deploying individual policies (use deploy-policies instead)
- Checking if Kyverno is installed (use check-kyverno instead)
- Getting help with Kyverno (use kyverno-help instead)
- Auditing resources against policies (use kyverno-cli-audit instead)
- Viewing policy violations (use show-violations instead)

## Commands

### Installation commands

```bash
# Add Kyverno Helm repository
helm repo add kyverno https://kyverno.github.io/kyverno/

# Update repository cache
helm repo update kyverno

# Install with defaults
helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace

# Install with wait for pods
helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace --wait
```

### Upgrade commands

```bash
# Upgrade to latest version
helm upgrade kyverno kyverno/kyverno --namespace kyverno

# Upgrade to specific version
helm upgrade kyverno kyverno/kyverno --namespace kyverno --version 3.2.0
```

## Parameters

- **namespace** - Installation namespace (default: `kyverno`)
- **version** - Kyverno chart version (default: latest)
- **createNamespace** - Create namespace if needed (default: `true`)
- **replicaCount** - Admission controller replicas (default: `3`)
- **admissionController** - Enable admission controller (default: `true`)
- **backgroundController** - Enable background controller (default: `true`)
- **cleanupController** - Enable cleanup controller (default: `true`)
- **reportsController** - Enable reports controller (default: `true`)
- **extraArgs** - Extra arguments for Kyverno (default: `[]`)

## Installation Steps

1. Add Kyverno Helm repository
2. Update repository cache
3. Install/upgrade Kyverno chart
4. Wait for pods to be ready
5. Verify installation

## Controllers

### Admission Controller
- Validates and mutates requests during API server admission
- Real-time policy enforcement
- Required for policy validation

### Background Controller
- Scans existing resources for policy violations
- Generates PolicyReports
- Runs on a schedule

### Cleanup Controller
- Cleans up resources based on cleanup policies
- Handles resource lifecycle
- Optional for basic usage

### Reports Controller
- Generates policy reports for auditing
- Creates ClusterPolicyReport resources
- Required for show-violations tool

## Common Workflows

### Standard installation

```
# Install with defaults
kyverno install

# Verify installation
kyverno check
```

### Minimal installation

```
# Install with minimal controllers
kyverno install \
  --cleanupController false \
  --reportsController false
```

### High availability

```
# Install with 5 replicas
kyverno install --replicaCount 5
```

### Custom namespace

```
# Install in policy-system namespace
kyverno install \
  --namespace policy-system \
  --createNamespace true
```

### Specific version

```
# Install version 3.2.0
kyverno install --version 3.2.0
```

## Output

Returns installation status:
- Helm repository setup confirmation
- Installation progress
- Pod status verification
- Next steps and usage instructions

## Verification

After installation, verify with:

```bash
# Check Kyverno status
kyverno check

# Or manually
kubectl get pods -n kyverno
kubectl get validatingwebhookconfigurations
```

## Notes

- Requires Helm CLI and kubectl to be available
- Follows official Kyverno installation documentation
- Uses `--wait` flag to ensure installation completes
- Verifies pod status after installation
- Can upgrade existing installation
- Default: 3 replicas for admission controller
- All 4 controllers enabled by default
- Creates namespace if it doesn't exist
