---
tags: helm, kubernetes, cheatsheet
---

# Helm Cheatsheet

## Repository Management

**Add a new repo:**
```bash
helm repo add [repo-name] [repo-url]
```

**List all added repos:**
```bash
helm repo list
```

**Update list of charts from all repositories**:
```bash
helm repo update
```

**Remove a repo:**
```bash
helm repo remove [repo-name]
```

## Chart Management

**Search for charts in the repo:**
```bash
helm search repo [repo-name]
```

If you pass the option `--version`, it will show all available versions
```bash
helm search repo [repo-name] --versions
```

**Download and untar a chart to your local disk:**
```
helm pull yugabyte/yugabyte --untar
```

**Install a chart:**
```bash
helm install [release-name] [repo-name]/[chart-name]
```

**List all installed releases:**
```bash
helm list
```

**Upgrade an existing release:**
```bash
helm upgrade [release-name] [repo-name]/[chart-name]
```

**Rollback a release to a previous revision:**
```bash
helm rollback [release-name] [revision-number]
```

**Uninstall a release:**
```bash
helm uninstall [release-name]
```

## Templating & Debugging

**View the rendered templates:**
```bash
helm template [release-name] [chart-path] > output.yaml
```

**Debug install (shows computed values and generated templates):**
```bash
helm install [release-name] [chart-path] --debug --dry-run
```

## Chart Development

**Create a new chart:**
```bash
helm create [chart-name]
```

**Lint a chart:**
```bash
helm lint [chart-path]
```

**Package a chart:**

```bash
helm package [chart-path]
```

## How to get the values.yaml file

```
helm show values <repo>/<chart> > values.yaml
```