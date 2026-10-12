---
name: ttl64
description: Create and use short-lived Kubernetes (K3s) clusters on ttl64 to test manifests, Helm charts, operators or reproduce bugs, then let them expire. Use when the user wants a real cluster for a while, mentions ttl64, or needs to try something on Kubernetes without touching their own clusters.
---

# ttl64: clusters that end on their own

ttl64 gives a real multi-node K3s cluster in seconds. Every cluster has a TTL (30m, 1h, 4h, 8h or 24h) and is deleted when it runs out. It is billed per started hour for each computer (control plane and every worker), so time and size cost money.

## Two ways in

- **MCP tools** (`mcp.ttl64.com`): `create_cluster`, `run_kubectl`, `extend_cluster`, `delete_cluster`, `get_usage`, and the rest. Prefer them when they are connected.
- **CLI** (`ttl64`): when the tools are not connected but the shell has `ttl64` (`ttl64 --version`). Install: `curl -fsSL https://ttl64.com/install | sh`, then `ttl64 login`.

Use one path for a task; don't mix them.

## Workflow

1. **Check before spending.** Call `get_usage` (or `ttl64 usage`) when the cluster is bigger than the default or will run for hours. Plans limit sizes, workers, TTL, clusters at a time and regions.
2. **Create small and short.** Default to `size: small`, 1 worker, the shortest TTL that fits the task (`30m` for a quick check, `1h` for most work). Name it after the task (`helm-redis-test`), or let ttl64 pick one.
   - CLI: `ttl64 create helm-redis-test --size small --workers 1 --ttl 1h`
   - The call returns when the cluster is ready (usually under a minute).
3. **Work inside the cluster.**
   - MCP: `run_kubectl` with `args` as a list, e.g. `["get", "pods", "-A"]`, `["apply", "-f", "https://…"]`, `["logs", "deploy/api", "-n", "app", "--tail", "200"]`. It runs inside the cluster; credentials never come back.
   - CLI: `ttl64 ssh NAME -- kubectl get pods -A` runs the same way, or `ttl64 kubeconfig NAME > kubeconfig.yaml` when local tools need it.
4. **Extend only when needed.** `extend_cluster` / `ttl64 extend NAME 1h`. Tell the user the new end time.
5. **Delete when done.** `delete_cluster` / `ttl64 delete NAME` as soon as the result is in. Don't leave clusters running "just in case": the TTL is a safety net, not the plan.

## Rules

- Never print or store secrets: kubeconfigs, API keys (`ttl64_sk_…`), tokens from Secrets. Mask them if they show up in output.
- Ask before creating anything larger than small with 1 worker, any TTL above 1h, or more than one cluster at a time; say the hourly price (from `get_usage` / the plan) when you ask.
- Ask before `delete_cluster` on a cluster you did not create in this conversation.
- Region: leave it out to use the user's default. A region outside the plan fails with a message saying which plan includes it; relay it, don't retry elsewhere silently.
- If creation fails, `get_cluster` shows the reason; `retry_cluster` (or `ttl64 retry NAME`) retries a failed start once. If it fails again, offer `open_support_ticket` with the cluster attached.

## Recipes

- **Try a Helm chart:** create (1h) → `run_kubectl ["create", "namespace", "x"]` → install through `ttl64 ssh NAME -- helm install …` (the shell has helm) → watch pods → report → delete.
- **Reproduce a bug:** create with the same Kubernetes features the user describes → apply their manifests → collect `describe`, `logs`, `events` → summarize → delete.
- **Templates:** `list_templates` shows ready-made stacks (for example Argo CD); `create_cluster` with `template` starts with it installed.
