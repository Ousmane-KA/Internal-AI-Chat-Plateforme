# Local LLM Lab – Docker Deployment

## 1. Overview

This project provides a lightweight, self-hosted Large Language Model (LLM) environment running on a Linux virtual machine using Docker.

The objective is to deploy a local AI assistant with a web-based conversational interface, similar to ChatGPT, without relying on external AI APIs or cloud-based LLM services.

The solution is designed for laboratory, testing, and learning purposes, with limited hardware resources.


### Main Components

- **Ollama:** Runs and manages local Large Language Models.
- **Open WebUI:** Provides a web-based conversational interface.
- **Docker Compose:** Manages and orchestrates the containers.

## 2. Architecture


<img width="1408" height="768" alt="llm-chat-ai" src="https://github.com/user-attachments/assets/2c09e754-4466-4442-9a82-a90adb12a368" />


All components run locally on the Linux virtual machine.

## 3. Technical Requirements

### Recommended VM Configuration

| Resource | Configuration |
|---|---|
| Operating System | Ubuntu Server 22.04 / 24.04 |
| CPU | 2 vCPUs |
| RAM | 4 GB minimum |
| Storage | 25 GB |
| GPU | Not required |
| Container Runtime | Docker |
| Orchestration | Docker Compose |

**Note:** Performance depends on available CPU resources, RAM, model size, and the number of simultaneous users. A small quantized model is recommended for low-resource environments.

## 4. Software Stack

| Component | Description |
|---|---|
| Docker Engine | Container runtime |
| Docker Compose | Container orchestration |
| Ollama | Local LLM inference engine |
| Open WebUI | Web-based AI interface |
| Qwen 2.5 1.5B | Lightweight language model |

## 5. Installation

### Step 1 – Update the Linux System

```bash
sudo apt update && sudo apt upgrade -y
```

Install the required packages:

```bash
sudo apt install -y ca-certificates curl openssl
```

### Step 2 – Install Docker

Install Docker using the official installation script:

```bash
curl -fsSL https://get.docker.com | sudo sh
```

Enable and start Docker:

```bash
sudo systemctl enable --now docker
```

Install Docker Compose:

```bash
sudo apt install -y docker-compose-plugin
```

Verify the installation:

```bash
docker --version
docker compose version
```
---
<img width="1058" height="216" alt="image" src="https://github.com/user-attachments/assets/53608727-d0eb-4d2f-8a64-d9999693730b" />

---
### Step 3 – Create the Project Directory

```bash
sudo mkdir -p /opt/llm-lab
sudo chown -R $USER:$USER /opt/llm-lab

cd /opt/llm-lab
```

### Step 4 – Generate the WebUI Secret Key

Generate a secure application secret:

```bash
openssl rand -hex 32
```

Keep this value for the Docker Compose configuration.

### Step 5 – Create the Docker Compose File

Create the configuration file:

```bash
nano docker-compose.yml
```

Add the following configuration:

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    volumes:
      - ollama_data:/root/.ollama
    networks:
      - llm

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    environment:
      OLLAMA_BASE_URL: http://ollama:11434
      WEBUI_SECRET_KEY: "REPLACE_WITH_GENERATED_SECRET"
    volumes:
      - webui_data:/app/backend/data
    depends_on:
      - ollama
    networks:
      - llm

volumes:
  ollama_data:
  webui_data:

networks:
  llm:
    driver: bridge
```

Replace `REPLACE_WITH_GENERATED_SECRET` with the secret generated previously.

### Step 6 – Deploy the Containers

Start the environment:

```bash
docker compose up -d
```

Check container status:

```bash
docker compose ps
```

Expected containers:

---
<img width="1480" height="481" alt="image" src="https://github.com/user-attachments/assets/923d520f-a89c-4e32-8fff-37053d76e87c" />

---
- ollama
- open-webui

Check application logs:

```bash
docker compose logs -f
```

### Step 7 – Download the LLM Model

Download a lightweight language model:

```bash
docker exec -it ollama ollama pull qwen2.5:1.5b
```

---
<img width="1481" height="172" alt="image" src="https://github.com/user-attachments/assets/18be6009-fa6e-4c3a-a472-fcb79094300b" />

---
Verify the installed model:

```bash
docker exec -it ollama ollama list
```
---
<img width="1012" height="166" alt="image" src="https://github.com/user-attachments/assets/4fc1b2f2-033f-4d37-b5a9-eca260537ae3" />

---

Optional: Test the model directly from the terminal:

```bash
docker exec -it ollama ollama run qwen2.5:1.5b
```

## 6. Access the Web Interface

Once the containers are running, access Open WebUI from a web browser:

```text
http://<LINUX_VM_IP>:3000
```

Example:

```text
http://192.168.1.100:3000
```

On the first connection:

1. Open the WebUI URL.
2. Create the initial administrator account.
3. Log in to the application.
4. Select the `qwen2.5:1.5b` model.
5. Start a conversation with your local AI assistant.


---
<img width="1047" height="443" alt="image" src="https://github.com/user-attachments/assets/ecf3ec7c-e827-43f7-9c61-7d004cfc60a0" />

---
---
<img width="1707" height="867" alt="image" src="https://github.com/user-attachments/assets/16ea0528-3e74-4cd8-adbe-39d9d31d61e4" />

---

## 7. Model Management

Ollama supports multiple models.

Some lightweight models suitable for laboratory environments:

| Model | Description |
|---|---|
| qwen2.5:0.5b | Very lightweight model |
| qwen2.5:1.5b | Lightweight conversational model |
| qwen2.5:3b | More capable model |
| llama3.2:1b | Lightweight Meta model |

Download another model:

```bash
docker exec -it ollama ollama pull llama3.2:1b
```

List installed models:

```bash
docker exec -it ollama ollama list
```

Remove a model:

```bash
docker exec -it ollama ollama rm llama3.2:1b
```

## 8. Container Management

### Start the environment

```bash
cd /opt/llm-lab
docker compose up -d
```

### Stop the environment

```bash
docker compose down
```

### Restart the environment

```bash
docker compose restart
```

### Display container status

```bash
docker compose ps
```

### View logs

```bash
docker compose logs -f ollama
docker compose logs -f open-webui
```

### Update container images

```bash
docker compose pull
docker compose up -d
```

**Important:** The `main` image tag is mutable. For a more stable laboratory deployment, consider pinning tested image versions.

## 9. Data Persistence

Docker volumes are used to preserve application data.

| Volume | Purpose |
|---|---|
| ollama_data | Stores downloaded LLM models |
| webui_data | Stores Open WebUI configuration and conversation data |

List Docker volumes:

```bash
docker volume ls
```

Inspect a volume:

```bash
docker volume inspect llm-lab_ollama_data
```

Data is preserved when containers are stopped or recreated.

**Warning:** Running `docker compose down -v` deletes the associated volumes and their data. Avoid this command unless you intend to remove all persistent data.

## 10. Network and Security Considerations

For a laboratory environment:

- Restrict access to TCP port 3000 to trusted network clients.
- Do not expose Ollama port 11434 directly to the network.
- Use strong administrator credentials.
- Keep Docker images and the Linux operating system updated.
- Back up Docker volumes regularly.
- Use HTTPS through a reverse proxy if the interface needs to be accessed beyond a trusted local network.

The application is intended for internal laboratory usage and should not be exposed directly to the public Internet without additional security controls.

## 11. Troubleshooting

### Open WebUI is unavailable

Check the container:

```bash
docker ps
docker logs open-webui
```

Restart the service:

```bash
docker compose restart open-webui
```

### Ollama is unavailable

Check Ollama logs:

```bash
docker logs ollama
```

Check installed models:

```bash
docker exec -it ollama ollama list
```

### Model download issues

Retry the download:

```bash
docker exec -it ollama ollama pull qwen2.5:1.5b
```

### Insufficient memory

If the VM runs out of memory:

- Stop unnecessary services.
- Use a smaller model such as `qwen2.5:0.5b`.
- Reduce the model context length if needed.
- Monitor system resources:

```bash
free -h
docker stats
```

## 12. Future Improvements

Possible extensions for this laboratory environment:

- Deploy additional LLM models.
- Integrate document-based question answering (RAG).
- Enable PDF and Word document analysis.
- Configure HTTPS with Nginx or Traefik.
- Add authentication and multi-user access.
- Integrate an internal knowledge base.
- Experiment with different model sizes and inference performance.

## 13. References

- Ollama: https://ollama.com/
- Ollama Documentation: https://docs.ollama.com/
- Open WebUI: https://openwebui.com/
- Open WebUI Documentation: https://docs.openwebui.com/
- Docker Documentation: https://docs.docker.com/
- Docker Compose: https://docs.docker.com/compose/

---

**Project:** Local LLM Lab  
**Deployment:** Docker Compose  
**Environment:** Linux Virtual Machine  
**Purpose:** Local AI experimentation and learning  
**LLM Runtime:** Ollama  
**Web Interface:** Open WebUI
