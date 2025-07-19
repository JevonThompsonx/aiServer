## Ubuntu AI Server Checklist (Docker Compose Edition)

This guide walks you through setting up a powerful, locally-hosted AI chatbot on an Ubuntu server with an NVIDIA GPU.

### Phase 1: System Preparation

  - [ ] **1. Update System:**
    Ensure all your system's packages are up-to-date.

    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

  - [ ] **2. Install Essential Tools:**
    Install `git` for managing code, `curl` for downloading files, and `build-essential` for any necessary compilations.

    ```bash
    sudo apt install -y git curl build-essential
    ```

-----

### Phase 2: NVIDIA Driver & CUDA Installation

  - [ ] **1. Install NVIDIA Drivers:**
    The `ubuntu-drivers` command automatically finds and installs the best proprietary driver for your RTX 2060.

    ```bash
    sudo ubuntu-drivers autoinstall
    ```

  - [ ] **2. Install CUDA Toolkit:**
    This provides the necessary libraries for GPU acceleration in AI tasks.

    ```bash
    sudo apt install -y nvidia-cuda-toolkit
    ```

  - [ ] **3. Reboot System:**
    A reboot is essential for the new kernel modules and drivers to load correctly.

    ```bash
    sudo reboot
    ```

  - [ ] **4. Verify Driver Installation:**
    After rebooting, this command should show your GPU's stats, confirming the driver is working.

    ```bash
    nvidia-smi
    ```

-----

### Phase 3: Docker & Container Toolkit Setup

  - [ ] **1. Install Docker:**
    Follow the official steps to add Docker's repository and install the engine.

    ```bash
    # Add Docker's official GPG key:
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Add the repository to Apt sources:
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update

    # Install Docker Engine, CLI, Containerd, and Compose plugin
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

  - [ ] **2. Install NVIDIA Container Toolkit:**
    This allows Docker containers to access your GPU.

    ```bash
    curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
        sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
        sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
    sudo apt update
    sudo apt install -y nvidia-container-toolkit
    sudo nvidia-ctk runtime configure --runtime=docker
    sudo systemctl restart docker
    ```

  - [ ] **3. Add User to Docker Group:**
    This allows you to run Docker commands without `sudo`. **You must log out and log back in for this to take effect.**

    ```bash
    sudo usermod -aG docker $USER
    ```

-----

### Phase 4: Ollama Installation & Configuration

  - [ ] **1. Install Ollama:**
    The official script makes installation simple.

    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    ```

  - [ ] **2. Configure Ollama for Remote Connections:**
    This is the critical step to allow Open WebUI to connect to Ollama.

    ```bash
    sudo systemctl edit ollama.service
    ```

    This will open a blank text editor. Paste the following content, then save and close the file (in `nano`, press `Ctrl+O`, `Enter`, `Ctrl+X`):

    ```ini
    [Service]
    Environment="OLLAMA_HOST=0.0.0.0"
    Environment="OLLAMA_ORIGINS=*"
    ```

  - [ ] **3. Apply Ollama Configuration:**
    Reload the systemd manager and restart the Ollama service.

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart ollama
    ```

  - [ ] **4. Verify Ollama Configuration:**
    Check the status and ensure the logs show it's listening on all interfaces.

    ```bash
    sudo systemctl status ollama
    ```

    Look for a line that says `msg="Listening on [::]:11434"`. This confirms it's working correctly.

-----

### Phase 5: Download AI Models

  - [ ] **1. Pull Your Desired Models:**
    Download a few models for different purposes.

    ```bash
    # --- General Purpose & Chat ---
    ollama pull llama3:8b
    ollama pull phi3:mini

    # --- Reasoning & Instruction Following ---
    ollama pull nous-hermes2:latest
    ollama pull openhermes:latest

    # --- Coding & Development ---
    ollama pull deepseek-coder:6.7b
    ollama pull starcoder2:3b
    ```

  - [ ] **2. Verify Model Installation:**
    List all downloaded models to confirm they are ready.

    ```bash
    ollama list
    ```

-----

### Phase 6: Install and Configure Open WebUI

  - [ ] **1. Create Project Directory:**
    Keep your configuration organized in a dedicated folder.

    ```bash
    mkdir ~/open-webui && cd ~/open-webui
    ```

  - [ ] **2. Create `docker-compose.yml` File:**
    Create the file that defines your WebUI service.

    ```bash
    nano docker-compose.yml
    ```

    Paste the **exact** content below into the file:

    ```yaml
    services:
      open-webui:
        image: ghcr.io/open-webui/open-webui:main
        container_name: open-webui
        ports:
          - "8080:8080"
        extra_hosts:
          - "host.docker.internal:host-gateway"
        volumes:
          - open-webui:/app/backend/data
        restart: unless-stopped

    volumes:
      open-webui: {}
    ```

  - [ ] **3. Launch Open WebUI:**
    From inside the `~/open-webui` directory, start the service in the background.

    ```bash
    docker compose up -d
    ```

  - [ ] **4. Configure WebUI Connection:**

      * Open a web browser and navigate to `http://<your_server_ip>:8080`.
      * Create your admin account when prompted.
      * Go to **Settings** ⚙️ \> **Connections**.
      * Under the **Ollama API** section, enter the following URL:
        `http://host.docker.internal:11434`
      * Save the connection.

  - [ ] **5. Start Chatting:**
    Navigate back to the main chat interface. Your downloaded models should now appear in the **"Select a model"** dropdown. Enjoy\! 🚀
