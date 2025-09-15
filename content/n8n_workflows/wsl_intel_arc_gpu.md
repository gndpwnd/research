https://intel.github.io/intel-extension-for-tensorflow/latest/docs/install/experimental/install_for_arc_gpu.html

sudo apt update

sudo apt-get install -y gpg-agent wget
wget -qO - https://repositories.intel.com/gpu/intel-graphics.key | 
sudo gpg --dearmor --output /usr/share/keyrings/intel-graphics.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/intel-graphics.gpg] https://repositories.intel.com/gpu/ubuntu jammy/lts/2350 unified" | sudo tee /etc/apt/sources.list.d/intel-gpu-jammy.list
sudo apt-get update


sudo apt-get update 



sudo apt-get install \
    intel-igc-cm \
    intel-level-zero-gpu \
    intel-opencl-icd \
    level-zero \
    libigc1 \
    libigdfcl1 \
    libigdgmm12


wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | sudo gpg --dearmor --output /usr/share/keyrings/oneapi-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list
sudo apt-get update


sudo apt-get install intel-oneapi-runtime-dpcpp-cpp intel-oneapi-runtime-mkl



sudo apt-get install -y clinfo python3-pip intel-oneapi-gpu intel-opencl-runtime


# Check OpenCL devices
echo "=== OpenCL Devices ==="
clinfo | grep -E "Platform|Device"