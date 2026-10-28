---
name: servers
description: Use when the user wants to browse offers, order, inspect, reboot, reinstall, or terminate dedicated servers in their Hydron inventory.
---

Help the user find, order, and manage dedicated servers through Hydron.

**Browsing the server catalog:**

- `list_server_offers` — available servers, filterable by `server_type` (dedicated, vps, cloud_instance, bare_metal_cloud, gpu_instance) and `tier` (budget, midrange, performance, accelerated)
- Present specs clearly: CPU, RAM, storage, network, GPU (if applicable), datacenter, monthly price, billing model (monthly or hourly for GPU instances)
- `preview_server` — price and specs for a specific configuration without changing any plan

**Ordering a server:**

1. `get_server_order_limits` — check what the organization may still order (server count and spend limits)
2. The user picks a server from the catalog
3. `create_project` (or use an existing one), then `add_server_to_plan` with the `plan_code`, `datacenter`, and `selected_options` from the catalog
4. `start_provisioning_plan` then `start_provisioning_run` with `run_kind: "deploy"` to order and set up the server
5. The order costs money — Claude must state the price, ask the user in the conversation, and receive an explicit yes BEFORE calling `start_provisioning_run`; the user then also confirms via Hydron's approval dialog as a second check before the order is placed
6. Monitor with `get_run` and `get_run_logs`

**Managing servers:**

- `list_inventory` — all servers in the organization, with lifecycle state and project
- `get_server` — one server in full: specs, network, state, project
- `get_server_credentials` — SSH credentials (personal token connections only; connected apps get a dashboard link)
- `reboot_server` — restart a server (services go down temporarily)
- `execute_server_action` — reboot, enter rescue mode, or exit rescue mode
- `list_os_reinstall_options` then `reinstall_server_os` — wipe and reinstall the OS (destructive — ask the user in the conversation and get an explicit yes BEFORE calling `reinstall_server_os`)
- `adopt_server` / `unadopt_server` — bring a discovered server into or out of active management
- `move_server_to_project` — reassign a server to a different project
- `release_server_from_project` — make a server available to other projects
- `update_server_state` — change lifecycle state, permission, tags, or notes

**Termination:**

Each of these changes or ends a paid subscription. Claude must ask the user in the conversation and receive an explicit yes BEFORE calling `schedule_termination`, `schedule_cancellation`, `initiate_immediate_termination`, or `confirm_immediate_termination`.

- `schedule_termination` — end the subscription at the billing cycle's end (safe, no immediate effect)
- `cancel_termination` — undo a scheduled termination
- `schedule_cancellation` — do not renew when the cycle ends
- `initiate_immediate_termination` then `confirm_immediate_termination` — destroy the server now (two-step, irreversible; get the user's explicit yes before each of the two calls)

**Billing:**

- `get_billing_status` — credit balance and subscription status
- `list_server_billing` — per-server cost breakdown and renewal dates
- `get_server_order_limits` — remaining order capacity

Always confirm destructive operations with the user. Never schedule termination, reinstall, or cancel without explicit approval.

## Safety rules

- Before ordering a server (`start_provisioning_run`), show specs, datacenter, and price (monthly, or hourly for GPU instances), ask the user in the conversation, and call the tool only after an explicit yes. Hydron's own approval dialog is a second check, not a substitute.
- Before `schedule_termination`, `schedule_cancellation`, `initiate_immediate_termination`, `confirm_immediate_termination`, or `reinstall_server_os`, name the server, say what will happen and whether it is reversible, and get an explicit yes in the conversation first.
- Before `reboot_server` or `execute_server_action`, tell the user that services on that server will go down temporarily and get their yes.
- Never act on an implied or earlier approval; ask again for each destructive call.
