# Run RL Swarm (Testnet) Node

A Simple  guide to setting up an RL Swarm node and the web UI dashboard for GensynAI's Testnet.

---

## Overview

RL Swarm is an open source framework from GensynAI for building reinforcement learning training swarms. This guide helps you set up a Testnet node and monitor it via the web UI. It covers hardware recommendations, environment setup (Windows, Cloud GPU, VPS), dependency installation, running the node (CLI or Docker), login, monitoring, backups, and troubleshooting.

---

## Hardware requirements

- For CPU-only:
  - arm64 or x86 CPU
  - Minimum 32 GB RAM (64 GB recommended; other apps may crash training)
- For GPU:
  - Supported GPUs: RTX 3090, RTX 4090, RTX 5090, A100, H100
  - Recommended ≥24 GB VRAM (Gensyn now supports <24GB as well)
  - CUDA driver ≥ 12.4

Default supported models on Testnet:
- `Gensyn/Qwen2.5-0.5B-Instruct`
- `Qwen/Qwen3-0.6B`
- `nvidia/AceInstruct-1.5B`
- `dnotitia/Smoothie-Qwen3-1.7B`
- `Gensyn/Qwen2.5-1.5B-Instruct`

Choose a smaller model for weaker hardware.

---

## Quick navigation

- Getting started
- Environment setup (Windows / Cloud GPU / VPS)
- Install dependencies
- HuggingFace access token
- Clone repository
- Run the swarm (CLI / Docker)
- Login and web UI
- Join Judge (prediction market)
- Multiple nodes & backups
- Monitor node health and rewards
- Gswarm Telegram bot
- Update & troubleshooting
- Cloud providers notes

---

## 1. Getting started

Pick your environment:
- Windows (use WSL/Ubuntu)
- Cloud GPU (Vast.ai, QuickPod, Hyperbolic)
- VPS (CPU-only option)

---

## 2. Environment setup

### Windows (recommended: WSL + Ubuntu)
- Install Ubuntu on Windows via WSL.
- Ensure NVIDIA drivers & CUDA are installed if using GPU:
  ```bash
  sudo apt-get update
  sudo apt-get install -y nvidia-cuda-toolkit
  nvidia-smi
  nvcc --version
  ```

### Cloud GPU providers (high level)
- Vast.ai: create SSH key, select PyTorch template, choose GPU and disk.
- QuickPod: uses web-based terminal; choose CUDA 12.4 template, configure pod.
- Hyperbolic: add SSH key to dashboard, rent GPU, use SSH with `-L 3000:localhost:3000` to tunnel port 3000.

### VPS (CPU-only)
- Recommended: 16+ cores, 32+ GB RAM (64 GB better).
- Host suggestion: Hostbrr (optional).

---

## 3. Install dependencies (Ubuntu/Debian)

1) Update packages:
```bash
sudo apt update && sudo apt upgrade -y
```

2) General tools:
```bash
sudo apt install -y screen curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip
```

3) Python:
```bash
sudo apt install -y python3 python3-pip python3-venv python3-dev
```

4) Node.js & Yarn:
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt install -y nodejs
node -v
npm install -g yarn
yarn -v

# Install Yarn (alternate)
curl -o- -L https://yarnpkg.com/install.sh | bash
export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"
source ~/.bashrc
```

5) Docker (optional: for Docker method)
- Install Docker & docker-compose via official guide for your platform.

---

## 4. HuggingFace setup

1. Create a HuggingFace account.
2. Create an access token with Write permissions and save it (only needed if you plan to push trained models to HF).

---

## 5. Clone repository

```bash
git clone https://github.com/gensyn-ai/rl-swarm/
cd rl-swarm
```

---

## 6. Run the swarm

> Note: If you already have a `swarm.pem` identity file, put it in the `rl-swarm` directory before starting.

Two main methods: CLI (recommended for GPU) and Docker (CPU / Mac / Docker-compatible GPUs).

### CLI Method (GPU)
1. Start a screen session:
```bash
screen -S swarm
```
2. Create & activate Python venv:
```bash
python3 -m venv .venv
source .venv/bin/activate
# if the above fails:
. .venv/bin/activate
```
3. Run the installer / node:
```bash
./run_rl_swarm.sh
```

### Docker Method (GPU, Mac, CPU)
- Default `swarm.pem` directory inside container: `/rl-swarm/user/keys/`
- GPU Docker may only work on providers that support Ubuntu VM templates (Vast).
1. Start a screen:
```bash
screen -S swarm
cd rl-swarm
```
2. Build/run:
- Mac or CPU-only:
```bash
docker compose run --rm --build -Pit swarm-cpu
```
- GPU:
```bash
docker compose run --rm --build -Pit swarm-gpu
```

---

## 7. Login and web UI

1. Watch logs until you see:
```
Waiting for userData.json to be created...
```

2. Open the login page:
- Local PC: http://localhost:3000/
- Remote VPS / Cloud: tunnel port 3000 to local machine.

Localtunnel example (on VPS):
```bash
sudo npm install -g localtunnel
# Get a password token (your VPS IP is used as password)
curl https://loca.lt/mytunnelpassword
# Start tunnel
lt --port 3000
# Visit the prompted URL and enter the password
```

3. Authenticate via the web UI (choose your preferred method). After login, terminal proceeds to installation and asks prompts.

4. Example prompts:
- "Would you like to push models to Hugging Face? [y/N]" → Press `N` to join Testnet.
- "Enter model name in huggingface repo/name format, or press Enter for default" → Press Enter for default or specify one of the models listed earlier.

---

## 8. Joining the Judge (AI Prediction Market)

During setup you'll be asked:
```
Would you like to participate in the AI Prediction Market? (Y/n)
```
- Press Enter or `Y` to participate (default).
- The Judge is an experiment where swarm models place bets on reasoning problems; early correct bets reward more.

---

## 9. Node identity & screen management

- After boot, find your node name in logs after `Hello`, e.g., `whistling hulking armadillo`.
- Screen shortcuts:
  - Minimize: CTRL + A, then D
  - Reattach: `screen -r swarm`
  - Stop & kill: `screen -XS swarm quit`
  - List screens: `screen -ls`
  - Kill by screen id: `screen -XS <screen-id> quit`

---

## 10. Running multiple nodes

- Start new node by connecting with same email on a new instance — creates new animal name, Peer ID, new `swarm.pem`.
- To reuse an animal name, import the corresponding `swarm.pem`.
- Monitor all nodes via the Gensyn dashboard by logging in with the same email.

---

## 11. Backup & recovery

- Important file to backup: `swarm.pem` (keeps your node identity).
- Typical locations:
  - VPS: `/root/rl-swarm/swarm.pem` or `/home/ubuntu/rl-swarm/swarm.pem`
  - WSL: `\\wsl.localhost\Ubuntu\home\<username>\rl-swarm\swarm.pem`
  - Docker method save path: `/root/rl-swarm/user/keys/`
- Download via SFTP:
```bash
# Example
sftp -P <PORT> ubuntu@host.example.com
cd /home/ubuntu/rl-swarm
get swarm.pem
exit
```
- To upload for recovery:
```bash
sftp -P <PORT> ubuntu@host.example.com
put swarm.pem /home/ubuntu/rl-swarm/swarm.pem
```

---

## 12. Monitoring & rewards

- Official Gensyn Dashboard: login to check node health and leaderboards.
- Contract explorer: view on-chain info via Alchemy / Etherscan-like explorer (example URL format provided in logs / dashboard).
- Telegram bot and Gswarm provide notifications and role rewards.

---

## 13. Gswarm (Telegram) bot setup

1. Install Go & gswarm:
```bash
# Remove old Go if necessary
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> $HOME/.bash_profile
source .bash_profile
go version
go install github.com/Deep-Commit/gswarm/cmd/gswarm@latest
```
2. Create Telegram bot via @BotFather, save bot token.
3. Get your chat ID:
   - Send a message to the bot; fetch updates:
     ```
     https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
     ```
   - Extract `chat.id` from the response.
4. Run `gswarm` and follow prompts (enter bot token, chat ID, EOA address).

Link Discord & Telegram:
- In Discord `#|swarm-link` channel, use `/link-telegram` to get a code.
- In Telegram, send `/verify <code>` to the bot.

---

## 14. Update node

1. Stop node screens:
```bash
screen -ls
screen -XS <screen-id> quit
# or by name
screen -XS swarm quit
```

2. Update repo (choose the method appropriate to how you cloned):
- If no local changes:
```bash
cd rl-swarm
rm -rf .venv
git pull
```
- If you have local changes:
```bash
cd rl-swarm
git reset --hard
rm -rf .venv
git pull
```
- Recommended: re-clone if unsure:
```bash
cp ./swarm.pem ~/swarm.pem
cd ..
rm -rf rl-swarm
git clone https://github.com/gensyn-ai/rl-swarm
cd rl-swarm
cp ~/swarm.pem ./swarm.pem
```

3. Re-run the node as in Section 6.

---

## 15. Troubleshooting (common issues & fixes)

- PS1 unbound variable when using `sed` fix:
```bash
sed -i '1i # ~/.bashrc: executed by bash(1) for non-login shells.\n\n# If not running interactively, don'\''t do anything\ncase $- in\n    *i*) ;;\n    *) return;;\nesac\n' ~/.bashrc
```

- Daemon failed to start in 15 seconds:
  - Increase timeout in `hivemind` daemon:
    ```bash
    cd rl-swarm
    python3 -m venv .venv
    source .venv/bin/activate
    nano $(python3 -c "import hivemind.p2p.p2p_daemon as m; print(m.__file__)")
    # find startup_timeout: float = 15 and change to 120
    ```
- Stuck at loading localhost page (React auth modal fix):
```bash
cd rl-swarm
sed -i '/^  return (/i\  useEffect(() => {\n    if (!user && !signerStatus.isInitializing) {\n      openAuthModal();\n    }\n  }, [user, signerStatus.isInitializing]);\n\n' modal-login/app/page.tsx
```

- CUDA memory error / low VRAM GPUs:
  - Use an optimized config (reduce memory usage), e.g., in `rg-swarm.yaml`:
    ```yaml
    num_generations: 2
    num_train_samples: 1
    num_transplant_trees: 1
    dtype: 'bfloat16'
    enable_gradient_checkpointing: true
    beam_size: 20
    ```
  - Set environment variable before running:
    ```bash
    export PYTORCH_CUDA_ALLOC_CONF="expandable_segments:True,max_split_size_mb:128"
    ```
  - For MPS (mac GPU) or CPU configs, turn off bf16:
    ```bash
    # example: edit config file
    cd rl-swarm
    nano hivemind_exp/configs/mac/grpo-qwen-2.5-0.5b-deepseek-r1.yaml
    # set bf16: false and reduce max_steps to 5
    ```

---

## 16. Low-level tips & notes

- If running on providers that require `sudo` removal (e.g., QuickPod), adjust commands accordingly.
- Use `screen` or `tmux` to keep processes running after disconnect.
- Keep a backup of `swarm.pem` externally (do not lose it if you care about identity & name recovery).
- If pushing to HuggingFace, ensure you have sufficient upload bandwidth (2GB per model training upload).

---

## 17. Cloud provider quick notes

- Vast.ai:
  - Good Ubuntu templates and GPU options.
  - Use SSH key; tunnel `-L 3000:localhost:3000` for web UI.
- QuickPod:
  - No SSH key required often; use web terminal or SSH command provided.
  - Choose CUDA 12.4 templates for compatibility.
- Hyperbolic:
  - Add SSH key in dashboard, choose PyTorch template.

---

## 18. Final notes

- Default Testnet model and prompts are suitable for quick participation.
- If you want production usage or large-scale experiments, follow the official repository docs and tune configs to your hardware.
- Bookmark or save `swarm.pem` and persist your environment variables in `.bashrc` or systemd service for long-term running.

---

Written by Johnbosco
