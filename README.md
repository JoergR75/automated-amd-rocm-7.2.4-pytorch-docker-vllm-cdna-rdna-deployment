# ⚙️ Automated AMD AI ROCm 7.2.4 Stack + OpenCL + PyTorch + Transformers + Docker + optional vLLM Setup

[![ROCm](https://img.shields.io/badge/ROCm-7.2.4-ff6b6b?logo=amd)](https://rocm.docs.amd.com/en/docs-7.2.4/about/release-notes.html)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.12.0%20%28Stable%29-ee4c2c?logo=pytorch)](https://pytorch.org/get-started/locally/)
[![Docker](https://img.shields.io/badge/Docker-29.5.x-blue?logo=docker)](https://www.docker.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20%7C%2024.04-e95420?logo=ubuntu)](https://ubuntu.com/download/server)
[![AMD Radeon AI PRO R9000 Series](https://img.shields.io/badge/AMD-RDNA4%20Radeon(TM)%20AI%20PRO%20R9000%20Series-8B0000?logo=amd)](https://www.amd.com/en/products/graphics/workstations.html)
[![AMD Ryzen AI](https://img.shields.io/badge/AMD-Ryzen%20AI%20-8B0000?logo=amd)](https://ryzen-ai.com/en/)
[![AMD CDNA MI300 Series](https://img.shields.io/badge/AMD-CDNA%20Instint(TM)%20Architecture-8B0000?logo=amd)](https://www.amd.com/en/technologies/cdna.html)

## 📌 Overview
The script provisions a fully automated, non-interactive AMD GPU software development environment for AI and HPC software engineering on **Ubuntu 22.04** and **24.04**, centered on **ROCm 7.2.4** and **PyTorch** Stable.

At the platform layer, it installs the AMD GPU kernel driver (**amdgpu-dkms**) and the ROCm 7.2.4 runtime, including **HIP** and **OpenCL 2.x**, ensuring compatibility across **CDNA1**, **CDNA2**, **CDNA3** **CDNA4**, **RDNA3**, **RDNA4** GPUs and **Strix APUs**. The script configures **OpenCL ICD** paths, user group permissions (video, render, sudo), and kernel headers required for compiling GPU-accelerated native extensions.

For the AI framework layer, the script installs **PyTorch 2.12 Stable** (**ROCm 7.2.4 wheels**) directly from the official PyTorch ROCm nightly repository, enabling access to the latest HIP backends, kernel fusion paths, and compiler features. It complements PyTorch with Transformers, Accelerate, Diffusers, Datasets, SentencePiece, and supporting Python build tooling, allowing immediate development, testing, and profiling of modern LLM, diffusion, and data-parallel workloads.

The developer toolchain is rounded out with C/C++ build and system utilities required for low-level GPU software engineering and extension development, including **cmake**, **libstdc++ dev headers**, **git** / **git-lfs**, **libmsgpack**, and **rocm-bandwidth-test** for validating PCIe and HBM bandwidth. Runtime observability and system inspection are supported via htop, ncdu, and ROCm diagnostics (rocminfo, rocm-smi, amd-smi).

A validation script is generated to verify end-to-end GPU availability, confirming ROCm detection, PyTorch HIP enablement, GPU enumeration, and successful on-device tensor execution.

The setup is fully **non-interactive** and optimized for both **desktop** and **server** deployments. In addition it checks whether ROCm or PyTorch (installed via pip) is already present on the system.
If an existing ROCm installation is detected, it removes ROCm and related packages to ensure a clean environment. It also **detects** and **uninstalls** any PyTorch packages (including ROCm-specific builds) to prevent version conflicts before proceeding with a fresh installation.

---

## 🖥️ Supported Platforms

| **Component**      | **Supported Versions**                                |
|---------------------|------------------------------------------------------|
| **OS**            | Ubuntu 22.04.x (Jammy Jellyfish), Ubuntu 24.04.x (Noble Numbat) |
| **Kernels** tested       | 5.15.0-181 (22.04.5) • 6.8.0-124 (24.04.4)                       |
| **GPUs**          | AMD **CDNA1** • **CDNA2** • **CDNA3** • **CDNA4** • **RDNA3** • **RDNA4**              |
| **APUs**        | AMD **Strix** • **Strix Halo**                                       |
| **ROCm**          | 7.2.4                                                |
| **PyTorch**       | torch 2.12.0+rocm7.2, torchvision 0.27.0+rocm7.2       |       |

**⚠️ Note**: **Ubuntu 20.04.x (Focal Fossa)** is **not supported**. The last compatible ROCm version for 20.04 is **6.4.0**.

---

## ⚡ Features
- Automated **ROCm GPU drivers + HIP + OpenCL SDK** installation
- **PyTorch ROCm Stable** with GPU acceleration
- Preinstalled **Transformers**, **Accelerate**, **Diffusers**, and **Datasets**
- Integrated **Docker environment** with ROCm GPU passthrough
- **vLLM Docker images** for **RDNA4** & **CDNA**
- Optimized for **AI workloads**, **LLM inference**, and **model fine-tuning**

---

## 🚀 Installation

### 1️⃣ **System preperation**
Install **Ubuntu 22.04.5 LTS** or **Ubuntu 24.04.4 LTS** (Server or Desktop version).

**Recommendations:**
- Use a fresh Ubuntu installation if possible
- Assign the full storage capacity during installation
- Install **OpenSSH** for remote SSH management
- The script automatically checks the system for installed versions of ROCm, PyTorch, and Docker, and removes them if found
  - On a fresh Ubuntu installation, the script automatically skips the deinstallation routine, as illustrated below
    <img width="1460" height="605" alt="image" src="https://github.com/user-attachments/assets/2bb31cf0-a110-440f-8285-1eacaaa29af3" />
  - If an existing version is detected, it will be deleted, regardless of whether it is the same or an older release.
    <img width="590" height="298" alt="image" src="https://github.com/user-attachments/assets/6323bb70-5f3e-46d3-bf40-e8949d05c5a8" />

- SBIOS settings:
  - When using Linux, you should disable Secure Boot
  - On WRX80 and WRX90 motherboard solutions, make sure SR-IOV is enabled — there are known issues with Ubuntu Linux detecting the network otherwise

- Ubuntu 22.04.5:
  
  During installation, it may be required to add `nomodeset` to the GRUB boot parameters to prevent boot hangs.

  In the GRUB menu (for example, at **"Try or Install Ubuntu Server"**):
  - Highlight the installation entry
  - Press **`e`** to edit the boot parameters
  - Locate the line beginning with:

     ```bash
     linux /casper/vmlinuz
     ```

  - Add `nomodeset` before the final `---`:

     ```bash
     linux /casper/vmlinuz nomodeset ---
     ```

  - Press **Ctrl + X** or **F10** to boot with the updated parameters

### 2️⃣ **Download the Script from the Repository**
```bash
wget https://raw.githubusercontent.com/JoergR75/automated-amd-rocm-7.2.4-pytorch-docker-vllm-cdna-rdna-deployment/refs/heads/main/script_module_ROCm_724_Ubuntu_22.04-24.04_pytorch_server.sh
```

<img width="2495" height="473" alt="image" src="https://github.com/user-attachments/assets/ae0b7041-7926-4878-9f33-b78312209f43" />

### 3️⃣ **Run the Installer**
```bash
bash script_module_ROCm_724_Ubuntu_22.04-24.04_pytorch_server.sh
```
**⚠️ Note**: Entering the user password may be required.

<img width="1516" height="469" alt="image" src="https://github.com/user-attachments/assets/97f52845-9a52-41ab-9782-2e2fe151f8d6" />

The installation takes ~15 minutes depending on internet speed and hardware performance.

### 4️⃣ **Reboot the System**
After the successful installation, press "y" to reboot the system and activate all installed components.

<img width="1960" height="749" alt="image" src="https://github.com/user-attachments/assets/99302837-005c-454e-a471-c9d6de002887" />

## 🧪 Testing ROCm + PyTorch

After rebooting, verify your setup:

This script creates a simple diagnostic python file (test.py) to verify that PyTorch with ROCm support is correctly installed and working.

What it does:

- Shows the CPU and installed memory
- Prints the ROCm, PyTorch and Transformers version.
- Checks if ROCm is available and how many GPUs are detected.
- Displays the name of the first GPU (if available).
- Creates two random 3x3 tensors directly on the GPU (if available).
- Performs a simple tensor addition operation on the GPU.
- Prints confirmation that the operation was successful and shows the result.

Example usage:
```bash
python3 test.py
```
Expected Output Example:
| Ubuntu 24.04.04 LTS | Ubuntu 22.04.5 LTS |
|--------|--------|
| ![](https://github.com/user-attachments/assets/7b4177c2-5af8-4495-b882-40903d201f7b) | ![](https://github.com/user-attachments/assets/6c8e2a61-1d68-4e2c-ae4d-b2c7971e1af3) |

More details about the ROCm driver version can be reviewed:
```bash
apt show rocm-libs -a
```

<img width="2489" height="715" alt="image" src="https://github.com/user-attachments/assets/12e897a9-b00e-46bc-a0e1-6238848972dd" />

or `amd-smi`
```bash
amd-smi
```

<img width="1797" height="689" alt="image" src="https://github.com/user-attachments/assets/50e666fb-24c1-4e46-93f0-e403f7bf777a" />

## 📶 ROCm Bandwidth Test

**AMD’s ROCm Bandwidth Test utility** with the **`tb p2p` (Peer-to-peer device memory bandwidth test)** flag runs a complete set of bandwidth diagnostics.

### What it does

`rocm-bandwidth-test` is a diagnostic tool included in ROCm that measures **memory bandwidth performance** between:

- Host (CPU) ↔ GPU(s)  
- GPU ↔ GPU (if multiple GPUs are installed)  
- GPU internal memory  

### `tb p2p` option

Using the `--run tb p2p` flag runs **Peer-to-peer device memory bandwidth test**, including:

- **Host-to-Device (H2D)** bandwidth  
- **Device-to-Host (D2H)** bandwidth  
- **Device-to-Device (D2D)** bandwidth (for multi-GPU)  
- **Bidirectional / concurrent** bandwidth tests  

Run the P2P test
```bash
cd /opt/rocm/bin && ./rocm_bandwidth_test plugin --run tb p2p
```

### Output

The tool prints results in a **matrix format** showing bandwidth (GB/s) between every device pair.

<img width="1789" height="2116" alt="image" src="https://github.com/user-attachments/assets/61d38362-4efc-462c-b536-ed89f67184ad" />

More details about the setup can be verified by
```bash
cd /opt/rocm/bin && ./rocm_bandwidth_test plugin --run tb
```

<img width="1830" height="507" alt="image" src="https://github.com/user-attachments/assets/26ee19ff-b6cd-433b-b9fc-e33d3fc757ee" />

⚠️ **Caution:**  
Make sure **"Re-Size BAR"** is enabled in the **SBIOS**.  
If it is disabled, **P2P** will be deactivated, as shown below:

<img width="977" height="777" alt="{FD9B95A3-BEFA-4857-8BBB-8D06A90108F2}" src="https://github.com/user-attachments/assets/cc148322-45b3-4164-b215-521276749f9d" />

More details about the setup can be verified by
```bash
cd /opt/rocm/bin && ./rocm_bandwidth_test plugin --run tb
```

<img width="904" height="274" alt="{3F58A790-E952-4BD9-9F0A-B99FD8F0B081}" src="https://github.com/user-attachments/assets/28b1808a-8216-4d7c-b1ea-db599f140056" />

### ⚙️ How to Enable **Re-Size BAR** in SBIOS (example ASRock WRX90 evo)

1. Enter **SBIOS**

<img width="1007" height="760" alt="{F9649127-0F1F-4E14-8008-1F3782FBBDEF}" src="https://github.com/user-attachments/assets/9685c1a4-ecab-4fea-8e91-dd21b9869c7e" />

3. Navigate to **Advanced**

<img width="1018" height="761" alt="{135D3B4C-0732-4652-A3C0-1224D275A515}" src="https://github.com/user-attachments/assets/b1cdc3ce-b526-4cdc-b44f-71d1119cf6d7" />

5. Go to **PCI Subsystem Settings** and change **Re-Size BAR Support** to **Enable** 

<img width="1016" height="761" alt="{3C54C3DA-8B82-483C-AEA5-D0A511508780}" src="https://github.com/user-attachments/assets/60536e2b-e59f-4486-a1fc-ab3ff33a3cd8" />

## 🐋 Docker Integration

The script sets up a Docker environment with GPU passthrough support via ROCm.

Check Docker installation and version
```bash
docker -v
```

<img width="1260" height="99" alt="image" src="https://github.com/user-attachments/assets/b4d21bf7-0283-4fe6-b619-8e0ceb7d5f5a" />

### 🤖 vLLM Docker Images

To use vLLM optimized for RDNA4 and CDNA:

Use the container image you need.

**RDNA4** architecture running on Ubuntu 22.04
```bash
docker pull vllm/vllm-openai-rocm:v0.22.0
```

<img width="1791" height="1120" alt="image" src="https://github.com/user-attachments/assets/1e1b5730-3159-4d5e-abd0-765d9cbbe754" />

Further vLLM Docker versions for RDNA4 can be verified on Docker Hub:  
https://hub.docker.com/r/rocm/vllm-dev/tags?name=navi or https://hub.docker.com/r/vllm/vllm-openai-rocm/tags

or for **CDNA** architecture
```bash
sudo docker pull rocm/vllm:latest
```

Run vLLM with all available AMD GPU access (example for RDNA4 on Ubuntu 24.04)
```bash
sudo docker run -it \
    --device=/dev/kfd \
    --device=/dev/dri \
    --security-opt seccomp=unconfined \
    --group-add video \
    --entrypoint /bin/bash \
    vllm/vllm-openai-rocm:v0.22.0
```

<img width="1386" height="253" alt="image" src="https://github.com/user-attachments/assets/b69cf9c7-3ce0-47fa-b604-50d9a386a51c" />

With `rocm-smi`, you can verify all available GPUs (in this case, 2× Radeon AI PRO R9700 GPUs).

<img width="1843" height="415" alt="image" src="https://github.com/user-attachments/assets/fd5c0747-2a07-4781-86bd-fb597949c148" />

or `amd-smi`

<img width="1650" height="687" alt="image" src="https://github.com/user-attachments/assets/664edd2c-77cd-408f-a35a-048347a0c3bf" />

Download `test_vllm.py`
```bash
wget https://raw.githubusercontent.com/JoergR75/automated-amd-rocm-7.2.4-pytorch-docker-vllm-cdna-rdna-deployment/refs/heads/main/test_vllm.py
```

running `test_vllm.py`
```bash
python3 test_vllm.py
```

<img width="1190" height="1708" alt="image" src="https://github.com/user-attachments/assets/ea43c788-b885-42c1-b70a-43128ab7e6c3" />

If you need to add a specific GPU, you can use the **passthrough** option.  
First, verify the available GPUs in the `/dev/dri` directory (host).
```bash
cd /dev/dri && ls
```

<img width="1227" height="100" alt="image" src="https://github.com/user-attachments/assets/dfb844c0-12d4-4707-a9ab-eb0e301d2e76" />

Let's choose **GPU2**, also referred to as **"card2"** or **"renderD129"**.
```bash
sudo docker run -it \
    --device=/dev/kfd \
    --device=/dev/dri/card2 \
    --device=/dev/dri/renderD129 \
    --security-opt seccomp=unconfined \
    --group-add video \
    --entrypoint /bin/bash \
    vllm/vllm-openai-rocm:v0.22.0
```
GPU2 has been added to the container

<img width="1478" height="842" alt="image" src="https://github.com/user-attachments/assets/62b52e5a-36e0-481a-8647-95e03d755453" />

## How to Save a Modified Docker Container

1️⃣ Open your container and modify it as needed (e.g., install packages, change configurations).

**⚠️ Note: Do not stop or close the container!**

2️⃣ Open another terminal (CLI) window.

3️⃣ Verify the running and stopped containers:
```bash
sudo docker ps -a
```

<img width="844" height="126" alt="image" src="https://github.com/user-attachments/assets/b879c0a2-a071-4307-adba-0da66534fd15" />

4️⃣ In this example, we want to save the running container `loving_wescoff` as a new image named `rocm/vllm-dev:rocm7.2.1_navi_ubuntu24.04_py3.12_pytorch_2.9_vllm_0.16.0_2`:
```bash
docker commit loving_wescoff vllm/vllm-openai-rocm:v0.20.1_2
```

<img width="842" height="46" alt="image" src="https://github.com/user-attachments/assets/968c0c38-20c9-4cac-8928-c4a7797e15a7" />

5️⃣ Verify that the new image was created successfully:
```bash
sudo docker images
```

<img width="855" height="138" alt="image" src="https://github.com/user-attachments/assets/86a03be1-e4e2-4e88-8a28-6d362fb14d7b" />

6️⃣ Start the new container with one GPU (renderD129):
```bash
sudo docker run -it \
    --device=/dev/kfd \
    --device=/dev/dri/card2 \
    --device=/dev/dri/renderD129 \
    --security-opt seccomp=unconfined \
    --group-add video \
    vllm/vllm-openai-rocm:v0.20.1_2
```

<img width="828" height="395" alt="image" src="https://github.com/user-attachments/assets/e7349f84-b08b-4500-988d-19aff77025be" />
