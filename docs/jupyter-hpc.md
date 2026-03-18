# Connecting to a Jupyter Server on Fred Hutch HPC

Your DevContainer runs on your Mac, but your data and compute live on the Fred Hutch cluster. This guide explains how to run a Jupyter kernel on an HPC node and connect VS Code (inside the DevContainer) to it.

## Why this is needed

VS Code runs inside a Docker container on your Mac. The container cannot resolve HPC hostnames like `gizmok114` — it doesn't have access to Fred Hutch DNS. We solve this with an SSH tunnel that makes the remote Jupyter server accessible through Docker's internal networking.

## Step-by-step

### 1. Start Jupyter on an HPC node

SSH into the cluster and grab an interactive node, then start Jupyter:

```bash
# On the HPC node (e.g. gizmok114)
jupyter lab --ip=$(hostname) --port=$(fhfreeport) --no-browser
```

Note the URL from the output — you need the **port** and **token**:

```
http://gizmok114:34563/lab?token=abc123...
```

### 2. Open an SSH tunnel from your Mac

In a **Mac terminal** (not inside the DevContainer), forward the port:

```bash
ssh -N -L <PORT>:<HOSTNAME>:<PORT> <USERNAME>@<HOSTNAME>
```

For example:

```bash
ssh -N -L 34563:gizmok114:34563 <USERNAME>@gizmok114
```

The terminal will appear frozen — this is normal. The tunnel stays open as long as this command runs. Use `Ctrl+C` to close it when done.

### 3. Connect VS Code to the kernel

In VS Code (inside the DevContainer):

1. Open a `.ipynb` notebook
2. Click **Select Kernel** → **Existing Jupyter Server**
3. Enter this URL (replacing the port and token with yours):

```
http://host.docker.internal:<PORT>/lab?token=<YOUR_TOKEN>
```

For example:

```
http://host.docker.internal:34563/lab?token=abc123...
```

**Key detail:** Use `host.docker.internal` — not `localhost` or the HPC hostname. This is a special Docker DNS name that routes to your Mac's network, where the SSH tunnel is listening.

### 4. Select the kernel

After connecting, VS Code will show available kernels from the remote server. Pick the one matching your conda environment.

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `ENOTFOUND gizmok114` | Container can't resolve HPC hostname | Use `host.docker.internal` instead |
| `ECONNREFUSED 127.0.0.1` | Used `localhost` instead of `host.docker.internal` | Container's localhost ≠ Mac's localhost |
| `ECONNREFUSED host.docker.internal` | SSH tunnel not running | Check your Mac terminal — restart the `ssh -N -L` command |
| Tunnel terminal unfroze | SSH session timed out or disconnected | Re-run the `ssh -N -L` command |

## How it works

```
+---------------------------+     SSH tunnel      +------------------+
|  Mac                      |  (localhost:34563   |  HPC node        |
|                           |   → gizmok:34563)  |  (gizmok114)     |
|  +---------------------+ |                     |                  |
|  | DevContainer        | |                     |  Jupyter Server  |
|  |                     | |                     |  :34563          |
|  | VS Code  ──────────────── host.docker.internal:34563           |
|  |                     | |         ↓            |                  |
|  +---------------------+ |   Mac localhost:34563 ──── SSH ──────> |
+---------------------------+                     +------------------+
```
