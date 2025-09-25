## 1. Prepare WSL & Intel Arc GPU Support

1. **Install WSL2 (if not already installed)**

```powershell
wsl --install -d Ubuntu-22.04
wsl --update
wsl --shutdown
```

2. **Update GPU drivers for Intel Arc**

* Download and install Intel GPU driver for WSL / OpenCL / OneAPI:
  [Intel Arc WSL driver](https://www.intel.com/content/www/us/en/developer/articles/tool/openvino-toolkit.html)
* Verify GPU visibility in WSL:

```bash
clinfo | grep "Platform Name"
```

You should see `Intel(R) OpenCL Graphics`.


```bash
mkdir ai-interactive-reporting
```

3. **Install Python + pip inside WSL**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-venv python3-pip -y
python3.11 -m venv ~/venv
source ~/venv/bin/activate
pip install --upgrade pip
```


**install python without sudo**

restart install from local if needed

```bash
rm -rf $HOME/local/python3.11
```


```bash
mkdir -p $HOME/local
cd $HOME/local
# https://www.openssl.org/source/
wget https://github.com/openssl/openssl/releases/download/openssl-3.5.2/openssl-3.5.2.tar.gz
tar -xvf openssl-3.5.2.tar.gz
cd openssl-3.5.2
./config --prefix=$HOME/local/openssl --openssldir=$HOME/local/openssl
make -j$(nproc)
make install
```

```bash
cd $HOME/local
# https://sourceware.org/libffi/
wget https://github.com/libffi/libffi/releases/download/v3.4.5/libffi-3.4.5.tar.gz
tar -xvf libffi-3.4.5.tar.gz
cd libffi-3.4.5
./configure --prefix=$HOME/local/libffi
make -j$(nproc)
make install
```

```bash
cd $HOME/local
# https://github.com/madler/zlib/releases
wget https://github.com/madler/zlib/releases/download/v1.3.1/zlib-1.3.1.tar.gz
tar -xvf zlib-1.3.1.tar.gz
cd zlib-1.3.1
./configure --prefix=$HOME/local/zlib
make -j$(nproc)
make install
```

```bash
cd $HOME
wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz
tar -xvf Python-3.11.0.tgz
cd Python-3.11.0

./configure \
    --prefix=$HOME/local/python3.11 \
    --with-openssl=$HOME/local/openssl \
    --enable-optimizations \
    CPPFLAGS="-I$HOME/local/openssl/include -I$HOME/local/libffi/include -I$HOME/local/zlib/include" \
    LDFLAGS="-L$HOME/local/openssl/lib -L$HOME/local/libffi/lib -L$HOME/local/zlib/lib" \
    LD_RUN_PATH="$HOME/local/openssl/lib:$HOME/local/libffi/lib:$HOME/local/zlib/lib"

make -j$(nproc)
make install
echo 'export PATH="$HOME/local/python3.11/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

if you don't need to pre-install openssl, libffi, zlib, you can simplify to:

```bash
wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz; tar -xvf Python-3.11.0.tgz; cd Python-3.11.0
./configure --prefix=$HOME/local/python3.11 --enable-optimizations; make -j$(nproc); make install
echo 'export PATH="$HOME/local/python3.11/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

`--prefix=$HOME/local/python3.11` tells the build system to install everything there. After `make install`, the Python binaries, libraries, and headers are copied into that directory.

You can safely delete the tarball (Python-3.11.0.tgz) and the extracted source folder (Python-3.11.0) after make install completes. The installed Python does not depend on the source folder—it’s fully installed in `$HOME/local/python3.11`.

Your one-liner `echo 'export PATH=…' >> ~/.bashrc && source ~/.bashrc` ensures your shell sees this new Python immediately.



```bash
python3 --version
python3 -m venv venv
source venv/bin/activate
```

```bash
pip install --upgrade pip
pip install torch-directml torchvision torchaudio diffusers transformers safetensors accelerate opencv-python imageio pdfplumber markdown
```

```bash
pip install -r Real-ESRGAN/requirements.txt
pip install -r stable-diffusion-webui/requirements.txt
pip install -r Wan2.1/requirements.txt
pip install -r orpheus/requirements.txt
pip install musicgen
```

---

## 2. Text Models (Planning, Scripts)

* **Model:** LLaMA 3 8B
* **Optimal Platform:** Ollama

**Setup Steps:**

```bash
# Install Ollama CLI
curl -sSL https://ollama.com/install.sh | bash

# Pull LLaMA 3 8B
ollama pull llamamodels/llama-3-8b

# Test a prompt
ollama run llama-3-8b "Write a short story..."
```

> Ollama automatically uses available GPU if supported.

---

## 3. Image Generation & Upscaling

### Image Generation (SDXL)

```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
python3 launch.py --xformers
```

* Select **SDXL** checkpoint in WebUI.
* Set **device** to Intel GPU (OpenCL if supported).

### Image Upscaling (Real-ESRGAN / SwinIR)

```bash
git clone https://github.com/xinntao/Real-ESRGAN.git
cd Real-ESRGAN
pip install -r requirements.txt

python inference_realesrgan.py -i input.png -o output.png -n RealESRGAN_x4plus
```

* For 4K: increase VRAM or tile the image.
* For video frames: use **EDVR** or **Video2X**.

---

## 4. Video Generation (720p → 1080p/4K)

* **Model:** Wan 2.1 (small/medium)

```bash
git clone https://github.com/Wan-Video/Wan2.1.git
cd Wan2.1
pip install -r requirements.txt

# Generate 720p clip
python generate.py --resolution 720p --length 10 --model medium
```

### Upscaling Video Frames

```bash
git clone https://github.com/k4yt3x/video2x.git
cd video2x
python video2x.py -i input.mp4 -o output.mp4 --upscaler realesrgan
```

---

## 5. Audio Generation

### Speech / Narration (Orpheus 3B)

```bash
git clone https://github.com/openai/orpheus.git
cd orpheus
pip install -r requirements.txt

python tts.py --text "Hello, world" --output out.wav
```

### Music / SFX (MusicGen 1.5B)

```bash
pip install musicgen

musicgen-cli --text "Epic cinematic music" --length 30 --output music.wav
```

> GPU acceleration via PyTorch + Intel OpenCL or `torch-directml` if needed.

---

## 6. WSL GPU Configuration Tips

```python
# Test GPU support in Python
import torch
print(torch.backends.mps.is_available()) # For Apple GPUs
print(torch.cuda.is_available())         # For Nvidia GPUs

# Intel Arc via DirectML (if PyTorch cannot detect GPU)
pip install torch-directml

import torch_directml
dml = torch_directml.device()
x = torch.tensor([1.0,2.0]).to(dml)
```

---

## 7. Workflow Summary

| Task             | Model         | Resolution | Notes                                       |
| ---------------- | ------------- | ---------- | ------------------------------------------- |
| Text generation  | LLaMA 3 8B    | N/A        | Ollama environment, Intel GPU optional      |
| Image generation | SDXL          | 512–768 px | Upscale to 1080p/4K with Real-ESRGAN/SwinIR |
| Video generation | Wan 2.1       | 720p       | Upscale with Video2X + Real-ESRGAN/EDVR     |
| Speech           | Orpheus 3B    | N/A        | Multi-lingual TTS                           |
| Music/SFX        | MusicGen 1.5B | N/A        | Short cinematic tracks                      |

---

## Next Steps

1. Test **Ollama LLaMA 3 8B** first.
2. Install **SDXL + Real-ESRGAN** for images.
3. Install **Wan 2.1 + Video2X** for 30fps video generation/upscaling.
