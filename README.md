# Ansible Docker Provisioning Lab

**Personal Docker Automation Project**

This lab is the next step after the Dockerized Ansible Linux setup lab and the Ansible web stack deployment lab. Here, Ansible stops just configuring Linux and starts managing Docker itself: images, containers, networks, and multi-container deployment workflows.

---

## 1. Project Goal

This project teaches you how to use Ansible as the automation layer on top of Docker. You will learn how to provision Docker environments, pull images, create and manage containers, and orchestrate service composition using the modern `community.docker` collection.

Why this is the next step:
- Earlier work taught me how to configure Linux servers and deploy a web stack with Ansible.
- This project shows how Ansible can manage Docker itself, not just the VM or container host.

In short, you move from operating system automation into container automation.

---

## 2. Why This Project Matters

Docker can create and run containers, but Ansible can automate the entire lifecycle around Docker.

This lab shows how to use Ansible to turn container deployment into repeatable, idempotent infrastructure:

- Provision Docker hosts and networks
- Pull and manage container images reliably
- Start, stop, update, and redeploy containers with automation
- Build deployable service stacks with Ansible instead of manually typing `docker` commands

That matters in real DevOps workflows because it makes deployments predictable, version-controlled, and easier to maintain.

---

## 3. Architecture

### What you build
- **Control Node** — runs Ansible and drives Docker automation
- **Target Docker Hosts** — one or more hosts where Docker is managed
- **Docker Engine** — the runtime that actually pulls images and runs containers
- **Ansible  community.docker** collection — the module set used to manage Docker resources

### How it fits together

Ansible connects to the target host(s) and uses Docker modules to manage Docker resources. The host still runs the Docker engine, but the desired state is defined in playbooks.

### ASCII Diagram

```
Your Host
  |
  | docker-compose / SSH
  v
Control Node (Ansible)
  | runs playbooks using community.docker
  v
Target Docker Host(s)
  | creates images, containers, networks
  v
Docker Engine
```

This project is mostly about the relationship between Ansible and Docker. Ansible does not replace Docker; it automates Docker.

---

## 4. Prerequisites

Required:
- Docker installed locally
- Docker Compose available
- Basic Ansible playbook knowledge
- Familiarity with Project 1 and Project 2 concepts
- Comfortable using command line tools

Optional but helpful:
- Basic Docker concepts: image, container, network, volume
- Understanding of containerized application architecture
- Experience with Ansible inventories, variables, and roles

Verify your setup:

```bash
docker --version
docker-compose --version
ansible --version
```

---

## 5. Project Structure

```
Ansible-Docker-Provisioning-Lab/
├── README.md
├── docker-compose.yml             # lab network and service composition
├── control/                       # Ansible control container configuration
│   ├── Dockerfile                 # build Ansible + Docker tooling
│   └── entrypoint.sh              # control node initialization
├── target/                        # target Docker host definition (optional)
│   ├── Dockerfile                 # Docker engine host image
│   └── entrypoint.sh              # target host startup logic
├── playbooks/                     # Ansible automation assets
│   ├── site.yml                   # main playbook
│   ├── docker_provision.yml       # Docker host and network provisioning
│   ├── docker_services.yml        # image and container orchestration
│   ├── inventory.ini              # target host inventory
│   ├── group_vars/                # shared variables
│   │   └── all.yml
│   └── roles/                     # reusable roles
│       └── docker/                # Docker automation role example
└── .gitkeep
```

Major files and folders:
- `docker-compose.yml` — starts the lab containers and shared network
- `control/` — builds the Ansible control container
- `target/` — defines the Docker host environment
- `playbooks/site.yml` — entrypoint playbook that includes provisioning and service orchestration
- `playbooks/docker_provision.yml` — creates Docker networks, installs Docker, configures Docker resources
- `playbooks/docker_services.yml` — pulls images and manages container lifecycle
- `playbooks/inventory.ini` — tells Ansible which host(s) to manage

---

## 6. Core Concepts

### Modern Docker automation with `community.docker`

This project uses the `community.docker` collection rather than the older `docker_*` modules. That means you learn the actively maintained Ansible approach for Docker automation.

### Idempotent container management

Ansible modules like `community.docker.docker_container` and `community.docker.docker_image` are designed to be idempotent. You declare the state you want, and Ansible converges toward it safely.

### Image pulling and container lifecycle

Key modules:
- `community.docker.docker_image` — pull images, manage image tags, control image state
- `community.docker.docker_container` — create, start, stop, update, and remove containers
- `community.docker.docker_network` — create and manage Docker networks
- `community.docker.docker_compose_v2` — optionally deploy docker-compose-style stacks through Ansible

### Service orchestration

Ansible becomes the orchestration layer that manages service dependencies, restart behavior, networks, and container configuration. The focus shifts from manual Docker CLI commands to automated, repeatable deployments.

---

## 7. Getting Started

### Step 1: Clone the lab

```bash
cd ~/Desktop
git clone <repo-url> Ansible-Docker-Provisioning-Lab
cd Ansible-Docker-Provisioning-Lab
```

### Step 2: Build and start the lab

```bash
docker-compose up -d --build
```

### Step 3: Install the required Ansible collection

Inside the control container, install `community.docker`:

```bash
docker-compose exec ansible bash
ansible-galaxy collection install community.docker
```

### Step 4: Verify Docker automation modules are available

```bash
ansible localhost -m ansible.builtin.debug -a 'msg=community.docker available'
```

### Step 5: Run the first playbook

```bash
ansible-playbook -i playbooks/inventory.ini playbooks/site.yml
```

The first run will provision Docker resources, pull container images, create networks, and start your service stack.

### Step 6: Check the result

```bash
docker-compose exec ansible docker ps
```

You should see the containers that Ansible created and started on the target host.

---

## 8. Useful Commands

### Docker & Compose commands

| Command | Purpose |
|---------|---------|
| `docker-compose up -d --build` | Start lab services and build images |
| `docker-compose down` | Tear down the lab |
| `docker-compose logs -f ansible` | Watch control container logs |
| `docker-compose exec ansible bash` | Enter the Ansible control node |
| `docker ps` | List running containers on the host |
| `docker network ls` | List Docker networks |

### Ansible commands

| Command | Purpose |
|---------|---------|
| `ansible-playbook -i playbooks/inventory.ini playbooks/site.yml` | Run the main Docker provisioning lab playbook |
| `ansible-playbook -i playbooks/inventory.ini playbooks/docker_provision.yml` | Provision Docker host and network |
| `ansible-playbook -i playbooks/inventory.ini playbooks/docker_services.yml` | Pull images and manage containers |
| `ansible-playbook -i playbooks/inventory.ini playbooks/site.yml --check` | Dry-run without making changes |
| `ansible-playbook -i playbooks/inventory.ini playbooks/site.yml -vvv` | Debug with verbose output |
| `ansible-inventory -i playbooks/inventory.ini --list` | Show inventory structure |

---

## 9. What You Will Learn

- How to use `community.docker.docker_image` to manage images and tags
- How to use `community.docker.docker_container` to create and control container lifecycles
- How to create Docker networks with `community.docker.docker_network`
- How to model Docker deployment as infrastructure-as-code
- How to make container provisioning repeatable and idempotent
- How to move from host configuration into application orchestration
- How to optionally use `community.docker.docker_compose_v2` for Compose-style deployments

---

## 10. What’s Next

After this project, you are ready for Project 4: taking your automation beyond local Docker and into cloud infrastructure.

Project 4 will show how to use Ansible to provision cloud platforms such as AWS or GCP, deploy container workloads in the cloud, and manage infrastructure alongside application deployment.

That is the bridge from local Docker automation to real cloud-native DevOps workflows.

---

## Notes

This project is deliberately focused on the modern `community.docker` collection rather than the older deprecated Docker modules. That keeps your skills aligned with actively maintained Ansible Docker automation.
