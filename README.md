# Arch Linux AI Server Checklist (Docker Compose Edition)

### Phase 1: System Preparation

  - [ ] **1. Update System:**
    Ensure your system is fully up-to-date.
    ` bash sudo pacman -Syu  `

  - [ ] **2. Install Essential Tools:**
    Install Git for cloning repositories and an AUR helper like `yay` to easily install packages from the Arch User Repository.
    \`\`\`bash
    \# Install git and base-devel for building packages
    sudo pacman -S --needed git base-devel

    ````
    # Clone and install yay (AUR Helper)
    git clone https://aur.archlinux.org/yay.git
    cd yay
    makepkg -si
    cd ..
    ```
    ````

### Phase 2: NVIDIA Driver Installation

  - [ ] **1. Install NVIDIA Drivers & CUDA Toolkit:**
    This installs the proprietary driver for your RTX 2060 and the CUDA toolkit needed by Ollama.
    ` bash sudo pacman -S nvidia nvidia-utils cuda  `

  - [ ] **2. Reboot System:**
    A reboot is essential for the new kernel modules and drivers to load correctly.
    ` bash sudo reboot  `

  - [ ] **3. Verify Driver Installation:**
    After rebooting, run this command. It should show your RTX 2060's stats, confirming the driver is working.
    ` bash nvidia-smi  `

### Phase 3: Core AI Software Installation

  - [ ] **1. Install Docker and Docker Compose:**
    This provides the container engine and the compose tool.
    ` bash sudo pacman -S docker docker-compose  `

  - [ ] **2. Configure Docker Service:**
    Start the Docker service and enable it to launch on boot.
    ` bash sudo systemctl start docker sudo systemctl enable docker  `

  - [ ] **3. Add User to Docker Group:**
    This allows you to run Docker commands without `sudo`. **You must log out and log back in for this change to take effect.**
    ` bash sudo usermod -aG docker $USER  `

  - [ ] **4. Install Ollama:**
    Using the `yay` AUR helper makes this simple.
    ` bash yay -S ollama-cuda  `

  - [ ] **5. Enable Ollama Service:**
    This ensures Ollama starts automatically on boot.
    ` bash sudo systemctl enable --now ollama  `

### Phase 4: Download AI Models

  - [ ] **1. Pull the "Everyday" Model:**
    Meta's Llama 3 8B for general chat and tasks.
    ` bash ollama pull llama3:8b  `

  - [ ] **2. Pull the "Coding" Model:**
    DeepSeek Coder for programming assistance.
    ` bash ollama pull deepseek-coder:6.7b  `

  - [ ] **3. Pull the "Reasoning" Model:**
    Nous Hermes 2 for more complex logic and puzzles.
    ` bash ollama pull nous-hermes2:mistral  `

  - [ ] **4. Verify Model Installation:**
    List all downloaded models to confirm they are ready.
    ` bash ollama list  `

### Phase 5: Install and Configure Open WebUI (with Docker Compose)

  - [ ] **1. Create a Project Directory:**
    It's good practice to keep your configuration file in a dedicated folder.
    ` bash mkdir ~/open-webui cd ~/open-webui  `

  - [ ] **2. Create the `docker-compose.yml` File:**
    Create a new file named `docker-compose.yml` and paste the following content into it. This file declaratively defines the Open WebUI service, making it easy to manage.
    ` bash nano docker-compose.yml  `
    **Paste this content:**
    \`\`\`yaml
    version: '3.8'

    ````
    services:
      open-webui:
        image: ghcr.io/open-webui/open-webui:main
        container_name: open-webui
        ports:
          - "3000:8080"
        volumes:
          - open-webui:/app/backend/data
        extra_hosts:
          - "host.docker.internal:host-gateway"
        restart: unless-stopped

    volumes:
      open-webui: {}
    ```
    ````

  - [ ] **3. Launch Open WebUI:**
    From inside the `~/open-webui` directory, run this command. It will read your `docker-compose.yml` file and start the service in the background (`-d`).
    ` bash docker compose up -d  `

  - [ ] **4. Check Container Status:**
    Ensure the container is running without errors.
    ` bash docker ps  `

  - [ ] **5. Initial Setup:**
    Open a web browser and navigate to `http://<your-server-ip>:3000`. Create your admin account. The UI should automatically detect and display the Ollama models you downloaded.

### Phase 6: Enable Remote Access

  - [ ] **1. Choose Your Method:**
    \* **For private access:** Use [Tailscale](https://tailscale.com/kb/1017/install/).
    \* **For public access:** Use a [Cloudflare Tunnel](https://www.google.com/search?q=https://developers.cloudflare.com/zerotrust/get-started/get-started-os/linux-and-freebsd/).

  - [ ] **2. Install and Configure:**
    Follow the official guide for your chosen method on Arch Linux. When configuring the tunnel, point it to the Open WebUI service at `http://localhost:3000`.

### Phase 7: Final Verification

  - [ ] **1. Reboot the Server:**
    Perform a final reboot to test the full automation.
    ` bash sudo reboot  `

  - [ ] **2. Test Everything:**
    After the server is back online, wait a minute, then:
    \* Try accessing your Open WebUI instance through its Tailscale or Cloudflare address from another device (your phone or laptop).
    \* Select one of the models and start a chat.

