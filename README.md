# 🚀 Ultimate Guide: Ollama + Open WebUI on DigitalOcean Ubuntu

A complete, production-ready guide to self-hosting **Ollama** with the **Qwen 2.5:7B** model and **Open WebUI** on a fresh DigitalOcean Droplet. 

---

## 🛠️ Step-by-Step Deployment

* ### **STEP 1: Allocate a 4GB Swap File**
    *Prevent system crashes and out-of-memory errors when loading large models by configuring virtual memory.*
    ```bash
    sudo fallocate -l 4G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
    ```

* ### **STEP 2: Install Ollama Engine**
    *Download and deploy the native Ollama service onto your host machine.*
    ```bash
    curl -fsSL [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh
    ```

* ### **STEP 3: Expose Ollama Network Binding**
    *Break down the default local-only barrier (`127.0.0.1`) so internal Docker containers can access the engine.*
    ```bash
    sudo mkdir -p /etc/systemd/system/ollama.service.d
    echo '[Service]' | sudo tee /etc/systemd/system/ollama.service.d/override.conf
    echo 'Environment="OLLAMA_HOST=0.0.0.0"' | sudo tee -a /etc/systemd/system/ollama.service.d/override.conf
    sudo systemctl daemon-reload
    sudo systemctl restart ollama
    ```

* ### **STEP 4: Configure UFW Firewall Rules**
    *Safely grant your internal Docker network bridge permission to cross the Ubuntu host firewall line.*
    ```bash
    sudo ufw allow in on docker0 to any port 11434 proto tcp
    ```

* ### **STEP 5: Launch Open WebUI Container**
    *Deploy the frontend user interface, mapping it directly back to your host-managed Ollama network pipeline.*
    ```bash
    sudo docker run -d -p 80:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
    ```

* ### **STEP 6: Pull the Qwen 2.5:7B Model**
    *Download the intelligence layer straight into your freshly configured engine pipeline.*
    ```bash
    ollama run qwen2.5:7B
    ```

---

## 🔍 Debugging Toolkit

> Use these tailored diagnostic commands whenever connections drop, settings fail to save, or models go missing in the UI dropdown.

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
If you change model properties in `Admin Panel -> Models -> Advanced Params` (like `max_tokens`) and find they won't save, it is an environment bug where Ollama's back-end sync overwrites WebUI. **Fix it by navigating to Workspace ➡️ Models ➡️ Create/Clone a Model**, and adjusting your parameters inside the custom preset layout instead!
