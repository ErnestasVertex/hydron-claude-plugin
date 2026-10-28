---
name: deploy
description: Use when the user wants to deploy an application, Git repo, Docker image, open-source software, or LLM model to a dedicated server with Hydron.
---

Help the user deploy something to a dedicated server using Hydron. This covers several scenarios:

If the user only wants servers planned or priced without deploying code, use the `infrastructure` skill instead.

**Deploying from a Git repository:**
1. Ask the user for their repository URL (GitHub, GitLab, Bitbucket, or any Git URL)
2. Use `create_project` to create a project
3. Use `get_analysis_cache_status` — if a cached analysis exists (personal or global), ask the user whether to reuse it or analyse from scratch
4. Use `start_code_analysis` to analyse the repository (pass the cache choice)
5. Poll `get_task` — if it reports `input_required`, read `pendingDecision` and use `respond_to_task` (e.g. whether to use existing Dockerfiles, which parts to analyse)
6. When complete, use `list_discovered_services` and present the discovered services to the user
7. Use `update_service_acceptance` to accept the services the user wants deployed
8. Use `start_infrastructure_plan` to generate a server plan with specs and pricing
9. Poll `get_task` — planning may pause with questions about traffic, availability or budget; answer with `respond_to_task`
10. Present the plan to the user — show server specs, monthly cost, and which services run where
11. The user may adjust servers (`add_server_to_plan`, `select_server_option`, `select_server_datacenter`) or service placement (`propose_service_placements`)
12. Use `update_infrastructure_plan_review_state` to accept the servers
13. Use `start_provisioning_plan` to generate deployment steps, poll `get_task`
14. Use `update_server_acceptance` / `update_project_acceptance` to accept the provisioning plans
15. Use `start_provisioning_run` with `run_kind: "deploy"` to order the server and deploy — this costs money. Claude must state the cost, ask the user in the conversation, and receive an explicit yes BEFORE calling `start_provisioning_run`; Hydron then shows its own approval dialog as a second check
16. Monitor progress with `get_task`, then `get_run` and `get_run_logs`
17. When complete, share the live URL (`get_project_domain`)

**Deploying open-source software (Ghost, WordPress, Plausible, etc.):**
1. The user names the software they want to deploy
2. Create a project and use `create_service` with the appropriate Docker image or repository
3. Follow the same infrastructure plan → approve → provisioning plan → approve → deploy flow

**Deploying a Docker image:**
1. The user provides a Docker image reference (e.g. `nginx:latest`)
2. Create a project, use `create_service` with `git_provider: "docker"` and the `docker_image_url`
3. Plan infrastructure and deploy as above

**Running an LLM model on a GPU server:**
1. The user names the model (Llama, Mistral, etc.)
2. Create a project — use `list_server_offers` with `server_type: "gpu_instance"` and `tier: "accelerated"` to show GPU options
3. GPU servers are billed hourly for as long as they exist
4. Follow the standard plan → approve → deploy flow

**Redeploying updated code:**
1. Use `get_repository_status` to see how far the deployed version is behind the branch tip
2. Use `check_deploy_readiness` — if an update analysis is needed, run `start_code_analysis` with `update_for_deploy`
3. Use `start_provisioning_run` with `run_kind: "deploy_only"` to build and restart containers without touching the server setup

**Custom domain:**
After a successful deployment, use `set_project_custom_domain` to attach the user's domain. If `status` is `"pending_dns"`, the user must create the DNS records shown in `dnsRecords` at their registrar.

Always present costs clearly before any purchase. Never skip the user's approval for server orders.

## Safety rules

- Before any call to `start_provisioning_run` that orders a server (any `run_kind` that places an order), show the server specs and price and ask the user in the conversation; call the tool only after an explicit yes. Hydron's own approval dialog is a second check, not a substitute.
- Never place an order on the basis of an earlier or implied approval; ask again for each new purchase.
- GPU servers are billed hourly for as long as they exist — say so explicitly before the user confirms a GPU order.
