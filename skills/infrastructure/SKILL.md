---
name: infrastructure
description: Use when the user wants to plan, size, price, or provision dedicated server infrastructure with Hydron without yet deploying code.
---

Help the user build infrastructure on dedicated servers using Hydron. This is for users who want servers planned and set up — with or without an application to deploy.

If the user has a repo or image to deploy, use the `deploy` skill, which includes planning.

**Building infrastructure with Hydron's planner:**

1. **Create a project** — `create_project` with a name
2. **Code analysis (if deploying code)** — check `get_analysis_cache_status`, then `start_code_analysis`. Poll `get_task`; answer any `input_required` with `respond_to_task`. Accept discovered services with `list_discovered_services` and `update_service_acceptance`
3. **Generate an infrastructure plan** — `start_infrastructure_plan` with optional `requirements` (workload, traffic, budget hints). Poll `get_task`. If it pauses for a question, answer with `respond_to_task`. Present the plan: server specs, datacenter, monthly cost, which services go where
4. **User reviews the plan** — `update_infrastructure_plan_review_state` to accept or reject each server. The user can adjust before accepting:
   - `list_server_offers` to browse available servers by type and tier
   - `select_server_option` to change hardware options (RAM, disk, GPU)
   - `select_server_datacenter` to move to a different datacenter
   - `preview_server` to compare configurations and prices without changing the plan
5. **Generate provisioning plans** — `start_provisioning_plan` writes the setup steps for each server. Poll `get_task`
6. **Accept and deploy** — `update_project_acceptance` to approve, then `start_provisioning_run` with `run_kind: "deploy"` to order and set up the servers (costs money — Claude must state the cost, ask the user in the conversation, and receive an explicit yes BEFORE calling `start_provisioning_run`; Hydron then shows its own approval dialog as a second check)
7. **Monitor** — `get_run` and `get_run_logs` for progress. When done, the servers are ready

**Building infrastructure manually (no AI planning):**

1. `create_project`
2. `list_server_offers` to browse the catalog — filter by `server_type` (dedicated, vps, cloud_instance, gpu_instance) and `tier` (budget, midrange, performance, accelerated)
3. `add_server_to_plan` to add servers from the catalog, choosing `plan_code`, `datacenter`, and `selected_options`
4. `add_inventory_server_to_plan` to add servers the organization already owns
5. Place services with `propose_service_placements` then `apply_service_placement`
6. Review, generate provisioning plans, and deploy as above

**Managing existing infrastructure:**

- `list_inventory` — all servers in the organization
- `get_server` — full details on a server
- `get_infrastructure_plan` — the project's current plan (servers, services, proposals)
- `propose_service_placements` — move services between servers
- `release_server_from_plan` — remove a server from the plan
- `move_server_to_project` / `release_server_from_project` — reassign servers between projects

Always show prices before any purchase. The user approves every server order.

## Safety rules

- Before any call to `start_provisioning_run` that orders servers, show the full plan with per-server specs, datacenter, and monthly cost, then ask the user in the conversation; call the tool only after an explicit yes. Hydron's own approval dialog is a second check, not a substitute.
- Planning, previewing (`preview_server`), and browsing offers are free and do not need confirmation; ordering does. Ask again for every new order.
- Do not call `release_server_from_plan`, `release_server_from_project`, or `move_server_to_project` on a server that is running services without telling the user what will be affected and getting their yes first.
