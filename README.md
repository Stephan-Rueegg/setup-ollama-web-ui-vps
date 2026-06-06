# 🚀 Ultimate Guide: Ollama + Open WebUI on DigitalOcean Ubuntu

A complete, production-ready guide to self-hosting **Ollama** with the **Qwen 2.5:7B** model and **Open WebUI** on a fresh DigitalOcean Droplet running Ubuntu.

---

## 🛠️ Step-by-Step Deployment

* ### **STEP 1: Allocate a 4GB Swap File**
    *Prevent system crashes and out-of-memory (OOM) errors when loading large models by configuring virtual memory.*
    ```bash
    sudo fallocate -l 4G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
    ```

* ### **STEP 2: Install Docker Engine**
    *Install the official Docker runtime environment and its dependencies to containerize the Open WebUI frontend.*
    ```bash
    # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install -y ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # Add the repository to Apt sources:
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update

    # Install Docker packages:
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

* ### **STEP 3: Install Ollama Engine**
    *Download and deploy the native backend Ollama intelligence service onto your host machine.*
    ```bash
    curl -fsSL [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh
    ```

* ### **STEP 4: Expose Ollama Network Binding**
    *Modify the default local-only barrier (`127.0.0.1`) by setting the host environment variable to `0.0.0.0` so internal Docker networks can reach it.*
    ```bash
    sudo mkdir -p /etc/systemd/system/ollama.service.d
    echo '[Service]' | sudo tee /etc/systemd/system/ollama.service.d/override.conf
    echo 'Environment="OLLAMA_HOST=0.0.0.0"' | sudo tee -a /etc/systemd/system/ollama.service.d/override.conf
    sudo systemctl daemon-reload
    sudo systemctl restart ollama
    ```

* ### **STEP 5: Configure UFW Firewall Rules**
    *Explicitly allow inbound traffic from your internal Docker network bridge (`docker0`) to talk to Ollama without being silently dropped or frozen.*
    ```bash
    sudo ufw allow in on docker0 to any port 11434 proto tcp
    ```

* ### **STEP 6: Launch Open WebUI Container**
    *Deploy the frontend container, routing it to look at the host gateway interface to find the underlying Ollama instance.*
    ```bash
    sudo docker run -d -p 80:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
    ```

* ### **STEP 7: Pull the Qwen Model**
    *Download the intelligence layer directly into the backend engine pipeline.*
    ```bash
    ollama run qwen2.5:7B
    ```

---

## 🔍 Debugging Toolkit

> Use these tailored diagnostic commands whenever connections drop, settings fail to save, or models disappear from the frontend interface.

### 🌐 Network & Firewall Inspection

* **Verify Active Port & Interface Binding**
    ```bash
    sudo ss -tulpn | grep 11434
    ```
* **Check Active UFW Firewall Security Policies**
    ```bash
    sudo ufw status
    ```
* **Test Internal Docker-to-Host Communication Routing**
    ```bash
    sudo docker exec -it open-webui curl [http://host.docker.internal:11434/](http://host.docker.internal:11434/)
    ```

### 📦 Container System Health

* **View Real-Time Web-UI Application Live Logs**
    ```bash
    sudo docker logs --tail 30 open-webui
    ```
* **Verify Active Container Engine Environments**
    ```bash
    sudo docker exec open-webui env
    ```
* **Audit Internal Storage Engine Permissions**
    ```bash
    sudo docker exec -it open-webui ls -la /app/backend/data/
    ```
* **Inspect Internal SQLite Configuration Table Layout**
    ```bash
    sudo docker exec -it open-webui python3 -c "import sqlite3; conn=sqlite3.connect('/app/backend/data/webui.db'); c=conn.cursor(); c.execute(\"SELECT name FROM sqlite_master WHERE type='table';\"); print([t[0] for t in c.fetchall()])"
    ```
* **Dump Saved Database Parameter Variables**
    ```bash
    sudo docker exec -it open-webui python3 -c "import sqlite3; conn=sqlite3.connect('/app/backend/data/webui.db'); c=conn.cursor(); c.execute('SELECT * FROM config;'); print(c.fetchall())"
    ```

### 💾 Host Hardware Environment

* **Inspect Server Partition Disk Space Availability**
    ```bash
    df -h /
    ```
* **Check System Memory and Active Swap Allocations**
    ```bash
    free -h
    ```

---

### 💡 Pro-Tip: Advanced Parameters Resetting?
If you change model properties in `Admin Panel -> Models -> Advanced Params` (like `max_tokens`) and find they won't save, it is an environment bug where Ollama's backend sync overwrites the WebUI schema. **Fix it by navigating to Workspace ➡️ Models ➡️ Create/Clone a Model**, and adjusting your parameters inside the custom preset layout instead!
