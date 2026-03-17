---
name: kyverno-help
description: "Get Kyverno documentation and help information. Use when: user needs help with Kyverno installation, troubleshooting guidance, or wants to learn about Kyverno features. NOT for: actually installing Kyverno (use install-kyverno), checking installation status (use check-kyverno), deploying policies (use deploy-policies), or viewing violations (use show-violations). This is a documentation-only skill that provides static help content."
metadata:
  openclaw:
    emoji: "📚"
    requires:
      bins: []
---

# kyverno-help

Get Kyverno documentation and help information. Provides installation guides, troubleshooting tips, and best practices.

## When to Use

✅ **USE this skill when:**
- User asks for help with Kyverno installation
- Need troubleshooting guidance
- Want to learn about Kyverno features
- Looking for best practices
- First-time Kyverno users
- Need step-by-step guides

## When NOT to Use

❌ **DON'T use this skill when:**
- Actually installing Kyverno (use install-kyverno instead)
- Checking if Kyverno is installed (use check-kyverno instead)
- Deploying policies (use deploy-policies instead)
- Auditing resources (use kyverno-cli-audit instead)
- Viewing violations (use show-violations instead)
- Need real-time cluster information (use other tools)

## Parameters

- **topic** - Help topic: `installation` or `troubleshooting` (default: `installation`)

## Topics

### installation

Includes:
- Prerequisites
- Helm installation steps
- kubectl installation alternative
- Verification steps
- Post-installation configuration
- Quick start guide

### troubleshooting

Includes:
- Common issues and solutions
- Debug commands
- Log locations
- Performance tuning
- Support resources
- FAQ

## Common Workflows

### First-time setup

```
# 1. Get installation guide
kyverno help --topic installation

# 2. Install Kyverno
kyverno install

# 3. Verify installation
kyverno check

# 4. Deploy policies
kyverno deploy --policySets pod-security

# 5. Monitor violations
kyverno violations --namespace all
```

### Debug issues

```
# 1. Get troubleshooting guide
kyverno help --topic troubleshooting

# 2. Check Kyverno status
kyverno check

# 3. View violations
kyverno violations --namespace all

# 4. Check logs if needed
kubectl logs -n kyverno -l app.kubernetes.io/name=kyverno
```

### Learning Kyverno

```
# Get installation help
kyverno help --topic installation

# Read about best practices
# Review policy examples
# Check troubleshooting guide
```

## Output

Returns documentation content:
- Step-by-step guides
- Command examples
- Best practices
- Troubleshooting tips
- Links to official documentation

## Official Resources

- [Kyverno Documentation](https://kyverno.io/docs/)
- [Kyverno Policies](https://kyverno.io/policies/)
- [GitHub Repository](https://github.com/kyverno/kyverno)
- [Slack Community](https://slack.k8s.io/#kyverno)

## Notes

- No cluster access required
- Provides static documentation
- Useful for learning Kyverno
- Can be used offline
- Links to official Kyverno documentation for latest updates
- No binaries required
- Always available
