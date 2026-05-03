# Set Up ROCm in WSL on Strix Halo Systems

## Step 1: Install WSL

```powershell
wsl --install Ubuntu-24.04
```

## Step 2: ROCm Setup

```bash
cd ~/
sudo apt update

sudo mkdir --parents --mode=0755 /etc/apt/keyrings
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
    gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null

sudo tee /etc/apt/sources.list.d/rocm.list << EOF
deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/latest noble main
deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/graphics/latest/ubuntu noble main
EOF

sudo tee /etc/apt/preferences.d/rocm-pin-600 << EOF
Package: *
Pin: release o=repo.radeon.com
Pin-Priority: 600
EOF

sudo tee /etc/apt/sources.list.d/amdgpu.list << EOF
deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/amdgpu/6.4.2/ubuntu noble main
EOF

sudo apt update

sudo apt install -y hsa-runtime-rocr4wsl-amdgpu

curl -L -H "Accept: application/octet-stream" -o rocdxg-roct_latest_amd64.deb $(curl -s https://api.github.com/repos/ROCm/librocdxg/releases/latest | grep browser_download_url | grep "rocdxg-roct.*amd64.deb" | cut -d'"' -f4)
sudo apt install -y ./rocdxg-roct_latest_amd64.deb

rm ./rocdxg-roct_latest_amd64.deb

sudo apt install -y rocm

echo -e 'export HSA_ENABLE_DXG_DETECTION=1\nexport PATH=/opt/rocm/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

rocminfo
```

## Step 3: PyTorch (with uv)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
echo 'source $HOME/.local/bin/env' >> ~/.bashrc
source ~/.bashrc
uv venv
source .venv/bin/activate
uv pip install --index-url https://repo.amd.com/rocm/whl/gfx1151/ torch torchvision torchaudio
uv run python -c "
import torch
print(f'PyTorch: {torch.__version__}')
print(f'CUDA available: {torch.cuda.is_available()}')
if torch.cuda.is_available():
    print(f'Device: {torch.cuda.get_device_name(0)}')
"
```



