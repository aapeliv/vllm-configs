# vLLM configs

Systemd configs for running vLLM. Designed to be cloned to `/opt/vllm` so the
repo, `uv`, the venv, and the model cache all live in one directory.

## Files

- `pyproject.toml` / `uv.lock` / `.python-version` — `uv` project pinning `vllm`.
- `vllm-qwen3.6-27b-fp8.service` — systemd unit for `Qwen/Qwen3.6-27B-FP8`.
- `vllm-gemma4-26b-a4b.service` — systemd unit for `cyankiwi/gemma-4-26B-A4B-it-AWQ-4bit`.
- `vllm-qwen3-embedding-4b.service` — systemd unit for `Qwen/Qwen3-Embedding-4B`.
- `vllm-qwen3.8-27b.service` — systemd unit for `cyankiwi/Qwen3.8-27B-AWQ-INT4`.
- `vllm-muse-glimmer-30b-text.service` — systemd unit for `RedHatAI/Muse-Glimmer-30B-W4A16`, text-only.
- `vllm-muse-glimmer-30b-mm.service` — same, with images enabled.

After the service has run once, `/opt/vllm` will also contain `bin/`, `.venv/`,
`hf-cache/`, and `uv-cache/`.

## Install

```bash
# 1. Clone to /opt/vllm
sudo install -d -o "$USER" /opt/vllm
sudo git clone https://github.com/aapeliv/vllm-configs.git /opt/vllm

# 2. Create the service user and hand it the directory
sudo useradd --system --home-dir /opt/vllm --shell /usr/sbin/nologin vllm
sudo usermod -aG video,render vllm
sudo chown -R vllm:vllm /opt/vllm

# 3. Install uv into /opt/vllm/bin (no system-wide install)
sudo -u vllm sh -c '
    curl -LsSf https://astral.sh/uv/install.sh \
        | env UV_INSTALL_DIR=/opt/vllm/bin INSTALLER_NO_MODIFY_PATH=1 sh
'

# 4. Enable the service. First start runs `uv sync` and downloads the model,
#    so it can take a while — `TimeoutStartSec=1800` in the unit covers it.
sudo ln -s /opt/vllm/vllm-qwen3.6-27b-fp8.service \
    /etc/systemd/system/vllm-qwen3.6-27b-fp8.service
sudo systemctl daemon-reload
sudo systemctl enable --now vllm-qwen3.6-27b-fp8.service

# 5. Tail the logs
journalctl -u vllm-qwen3.6-27b-fp8 -f
```

## Updating

```bash
sudo -u vllm git -C /opt/vllm pull
sudo systemctl restart vllm-qwen3.6-27b-fp8
```

The restart triggers `uv run`, which re-syncs the venv against the updated
lockfile before launching vLLM.
