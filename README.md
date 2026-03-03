# pve-automation

A lightweight Proxmox VE (PVE) automation toolkit.

This repository currently provides a reusable Python API client and CLI for common PVE operations, including node discovery, VM/container lifecycle control, and task polling.

## Repository Structure

- `scripts/pve_client.py`: PVE API client with CLI commands
- `evals/evals.json`: evaluation metadata
- `SKILL.md`: skill documentation

## Requirements

- Python 3.9+
- `requests`
- `urllib3`

Install dependencies:

```bash
pip install requests urllib3
```

## Environment Variables

Set these before running commands:

- `PVE_HOST`: Proxmox host or IP
- `PVE_USER`: PVE user (default: `root@pam`)
- `PVE_TOKEN_ID`: API token ID
- `PVE_SECRET`: API token secret

Example:

```bash
export PVE_HOST=192.168.1.10
export PVE_USER=root@pam
export PVE_TOKEN_ID=automation
export PVE_SECRET=your-token-secret
```

## Quick Start

List nodes:

```bash
python scripts/pve_client.py list-nodes
```

List VMs on a node:

```bash
python scripts/pve_client.py list-vms <node>
```

Start a VM:

```bash
python scripts/pve_client.py start-vm <node> <vmid>
```

Stop a VM:

```bash
python scripts/pve_client.py stop-vm <node> <vmid>
```

## Notes

- API calls default to `https://<PVE_HOST>:8006/api2/json`
- The script currently disables SSL verification for convenience in private networks. Use trusted certificates in production.
