---
name: deploy-policies
description: "Deploy Kyverno policies to Kubernetes cluster using kubectl apply. Use when: user wants to install ValidatingPolicy or MutatingPolicy resources into the cluster, update existing policies, or change validationFailureAction from Audit to Enforce. NOT for: dry-run validation (use kyverno-cli-audit), installing Kyverno itself (use install-kyverno), or checking if policies are working (use show-violations). This tool MODIFIES the cluster by creating Policy CRDs."
metadata:
  openclaw:
    emoji: "🚀"
    requires:
      bins: ["kubectl"]
---

# deploy-policies

Deploy Kyverno policies to the Kubernetes cluster. Unlike kyverno-cli-audit which only performs dry-run validation, this tool actually installs policies into the cluster.

## When to Use

✅ **USE this skill when:**
- User wants to deploy ValidatingPolicy resources to the cluster
- User wants to deploy MutatingPolicy resources (e.g., run-as-non-root)
- Updating existing policies with new configurations
- Changing validationFailureAction from Audit to Enforce
- First-time policy deployment after testing
- Need to enable background scanning for existing resources

## When NOT to Use

❌ **DON'T use this skill when:**
- User wants dry-run validation only (use kyverno-cli-audit instead)
- Installing Kyverno itself (use install-kyverno instead)
- Checking if Kyverno is installed (use check-kyverno instead)
- Viewing violations from PolicyReports (use show-violations instead)
- User hasn't tested policies in Audit mode first
- Need help with Kyverno (use kyverno-help instead)

## Commands

### Deploy commands

```bash
# Deploy pod-security policies in Audit mode
kubectl apply -f policies/pod-security.yaml

# Deploy MutatingPolicy for run-as-non-root
kubectl apply -f policies/add-pod-run-as-non-root.yaml

# Deploy all policies
kubectl apply -f policies/
```

### Update validation action

```bash
# Deploy in Audit mode (default, safe)
kubectl apply --validate=false -f policies/

# Deploy in Enforce mode (blocks violations)
# First modify policy YAML to change validationActions to [Enforce]
# Then apply
kubectl apply -f policies/
```

## Parameters

- **policySets** - Policy set to deploy:
  - `pod-security` - Pod Security Standards (4 ValidatingPolicies)
  - `rbac-best-practices` - RBAC security (4 ValidatingPolicies)
  - `kubernetes-best-practices` - K8s best practices (5 ValidatingPolicies)
  - `run-as-non-root` - MutatingPolicy for auto-setting runAsNonRoot
  - `all` - All policies (default)
- **validationFailureAction** - Action on failure: `Audit` or `Enforce` (default: `Audit`)
- **background** - Enable background scanning (default: `true`)
- **context** - Kubernetes context (optional)

## Policy Sets

### pod-security (ValidatingPolicy)
Pod Security Standards (Baseline) - 4 policies:

1. **disallow-capabilities** - Disallow adding capabilities beyond allowed list (high)
2. **disallow-host-namespaces** - Disallow sharing host namespaces (high)
3. **disallow-privileged-containers** - Disallow privileged mode (high)
4. **require-run-as-nonroot** - Require non-root execution (medium)

### rbac-best-practices (ValidatingPolicy)
RBAC security best practices - 4 policies:

1. **restrict-wildcard-resources** - Block wildcards in roles (medium)
2. **restrict-escalation-verbs-roles** - Block impersonate/bind/escalate (high)
3. **restrict-automount-sa-token** - Disable SA token auto-mount (medium)
4. **restrict-binding-system-groups** - Block system group bindings (high)

### kubernetes-best-practices (ValidatingPolicy)
Kubernetes operational best practices - 5 policies:

1. **require-labels** - Require app.kubernetes.io/name label (medium)
2. **disallow-latest-tag** - Disallow :latest image tag (medium)
3. **require-pod-requests-limits** - Require resource requests/limits (medium)
4. **require-probes** - Require liveness/readiness probes (medium)
5. **disallow-default-namespace** - Disallow 'default' namespace (medium)

### run-as-non-root (MutatingPolicy)
Automatically add runAsNonRoot security context:

- **add-pod-run-as-non-root** - Auto-sets `runAsNonRoot: true` in pod securityContext
- Type: MutatingPolicy (modifies resources at admission time)

## Validation Failure Actions

### Audit (default, recommended for testing)
- Logs policy violations
- Does not block resource creation
- Good for testing and monitoring
- Recommended for initial deployment

### Enforce (use with caution)
- Blocks non-compliant resources
- Prevents creation of violating resources
- Use in production after thorough testing
- Requires all policies to pass

## Common Workflows

### Progressive deployment (recommended)

```
# Step 1: Deploy in Audit mode
kyverno deploy --policySets pod-security --validationFailureAction Audit

# Step 2: Monitor violations for 1-2 weeks
kyverno violations --namespace all

# Step 3: Fix issues in applications
# Update deployments, fix configurations

# Step 4: Switch to Enforce mode
kyverno deploy --policySets pod-security --validationFailureAction Enforce
```

### Multi-cluster deployment

```
# Deploy to dev (Audit mode)
kyverno deploy --policySets all --validationFailureAction Audit --context dev-cluster

# Deploy to staging (Audit mode)
kyverno deploy --policySets pod-security --validationFailureAction Audit --context staging-cluster

# Deploy to production (Enforce mode)
kyverno deploy --policySets pod-security --validationFailureAction Enforce --context prod-cluster
```

### Security-focused deployment

```
# Deploy all security policies
kyverno deploy --policySets pod-security --validationFailureAction Enforce

# Deploy RBAC policies
kyverno deploy --policySets rbac-best-practices --validationFailureAction Enforce

# Add auto-mutation for non-root
kyverno deploy --policySets run-as-non-root
```

## Policy Files

All policy templates available in `policies/` directory:

```
policies/
├── pod-security.yaml              # Pod Security Standards
├── rbac-best-practices.yaml       # RBAC Best Practices
├── kubernetes-best-practices.yaml # Kubernetes Best Practices
└── add-pod-run-as-non-root.yaml   # MutatingPolicy
```

These can be customized before deployment.

## Output

Returns deployment status:
- Deployment confirmation
- kubectl apply output
- List of deployed policies
- Policy type (ValidatingPolicy/MutatingPolicy)
- Total count of policies deployed

## Notes

- ⚠️ **WARNING**: This tool MODIFIES the cluster by creating Policy CRDs
- Requires kubectl to be available
- Policies become active immediately after deployment
- Always test in Audit mode before Enforce
- Verify with kyverno-cli-audit first for dry-run validation
- Policy templates can be customized before deployment
- All policies default to Audit mode for safety
- Total: 13 ValidatingPolicy + 1 MutatingPolicy = 14 policies
