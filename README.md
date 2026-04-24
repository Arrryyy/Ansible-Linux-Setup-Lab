# Ansible Linux Setup Lab 🚀

A fully Dockerized Ansible learning environment where you'll master Infrastructure-as-Code by automating real Linux server configurations—all locally, for free, with zero cloud dependencies.

---

## 1. Project Goal

**What This Teaches:**
This project is your hands-on introduction to Ansible and Infrastructure-as-Code (IaC). Instead of manually SSH-ing into servers and running commands, you'll write playbooks that automate everything from system updates to security hardening.

**Why It Matters:**
- DevOps and SRE roles heavily depend on automation tools like Ansible
- You'll learn declarative infrastructure (define the desired state, Ansible handles the rest)
- You're building real skills: Linux administration + automation tooling
- Once you master this lab, deploying multi-server setups becomes repeatable and bulletproof

**What You'll Build:**
A complete pipeline to harden and configure new Linux servers automatically—user creation, SSH security, firewall rules, fail2ban DDoS protection, and system updates—all with a few playbook runs.

---

## 2. Architecture

### The Setup
Your lab consists of:
- **1 Control Node** — Ansible installed here; you run playbooks from this container
- **3 Managed Nodes** — Ubuntu servers running SSH; Ansible connects here via SSH to execute commands
- **Shared Docker Network** — All containers talk to each other internally; you manage from your host machine

### ASCII Diagram
```
┌─────────────────────────────────────────────────────────────┐
│                     Your Host Machine                        │
│                                                               │
│  docker-compose up → Starts all 4 containers                │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────────┐
         │    Docker Bridge Network (ansible-net) │
         ├────────────────────────────────────────┤
         │                                        │
         │  ┌──────────────────────────────────┐  │
         │  │   Control Node (ansible)         │  │
         │  │   • Ansible core installed       │  │
         │  │   • /ansible/playbooks mounted   │  │
         │  │   • SSH key for auth             │  │
         │  └──────────────────────────────────┘  │
         │           ↓ SSH to ↙  ↙  ↙             │
         │  ┌──────────────────────────────────┐  │
         │  │  Managed Node 1 (node1)          │  │
         │  │  • Ubuntu + SSH server           │  │
         │  │  • Hostname: node1               │  │
         │  └──────────────────────────────────┘  │
         │  ┌──────────────────────────────────┐  │
         │  │  Managed Node 2 (node2)          │  │
         │  │  • Ubuntu + SSH server           │  │
         │  │  • Hostname: node2               │  │
         │  └──────────────────────────────────┘  │
         │  ┌──────────────────────────────────┐  │
         │  │  Managed Node 3 (node3)          │  │
         │  │  • Ubuntu + SSH server           │  │
         │  │  • Hostname: node3               │  │
         │  └──────────────────────────────────┘  │
         │                                        │
         └────────────────────────────────────────┘

How It Works:
1. You edit playbooks on your host machine
2. docker-compose mounts playbooks/ into the control container
3. You run: docker-compose exec ansible ansible-playbook playbooks/site.yml
4. Ansible connects to node1, node2, node3 via SSH (passwords or keys)
5. Your playbook commands execute on all nodes simultaneously
```

### Flow Example
```
Your machine:
$ docker-compose exec ansible ansible-playbook playbooks/site.yml
    ↓
Control node runs the playbook
    ↓
Ansible SSH's into node1, node2, node3
    ↓
Tasks execute in parallel on all nodes
    ↓
Results printed to your terminal
```

---

## 3. Prerequisites

**Required:**
- **Docker** (v20.10+) — [Install here](https://docs.docker.com/get-docker/)
- **Docker Compose** (v1.29+) — Usually included with Docker Desktop
- **Basic terminal skills** — Comfortable with `cd`, `ls`, `ssh`, `vim`/`nano`

**Optional but helpful:**
- Basic Linux knowledge (users, permissions, firewall concepts)
- Curiosity about how servers get configured

**Verify your setup:**
```bash
docker --version
docker-compose --version
```

Both should show version numbers. If not, install Docker first.

---

## 4. Project Structure

```
Ansible/
├── README.md                           # This file — project overview & guide
├── docker-compose.yml                  # Docker setup: control + 3 nodes, networking
├── control/                            # Control node Dockerfile & scripts
│   ├── Dockerfile                      # Ubuntu + Ansible + SSH keys
│   └── entrypoint.sh                   # Startup script (SSH setup)
├── node1/, node2/, node3/              # Managed nodes (identical setup)
│   ├── Dockerfile                      # Ubuntu + SSH server
│   └── entrypoint.sh                   # SSH setup & user creation
├── playbooks/                          # YOUR PLAYBOOKS GO HERE ← Edit these locally
│   ├── site.yml                        # Main playbook (includes all stages)
│   ├── stage1_base_setup.yml           # OS updates, packages, system config
│   ├── stage2_user_management.yml      # Create users, set permissions
│   ├── stage3_security_hardening.yml   # SSH hardening, UFW firewall, fail2ban
│   ├── stage4_multi_env_variables.yml  # Environment-specific configs
│   ├── inventory.ini                   # Host inventory (node1, node2, node3)
│   ├── group_vars/                     # Group-level variables
│   │   └── all.yml                     # Variables applied to all nodes
│   └── roles/                          # Reusable Ansible roles (optional)
│       └── common/                     # Common role example
└── .gitkeep                            # Ensures empty folders are tracked in Git

Key Insight: Everything in playbooks/ is mounted into the container.
            Edit locally → Changes instantly available in the control container.
```

---

## 5. Getting Started

### Step 1: Clone or Extract the Project
```bash
cd ~/Desktop
# If already extracted, just navigate to it
cd Ansible
```

### Step 2: Verify the Project Structure
```bash
ls -la
# You should see: control/, node1/, node2/, node3/, docker-compose.yml, playbooks/, README.md
```

### Step 3: Build and Start the Lab
```bash
docker-compose up -d --build
```

This:
- Builds the control node image (with Ansible installed)
- Builds the 3 managed node images (with SSH servers)
- Starts all 4 containers on a shared network
- Mounts `playbooks/` into the control container

**Wait 10-15 seconds for SSH servers to be ready.**

### Step 4: Verify All Containers Are Running
```bash
docker-compose ps
```

Should show:
```
CONTAINER ID   IMAGE                STATUS              PORTS
abc123...      ansible-control      Up 2 minutes        
def456...      ansible-node         Up 2 minutes        
ghi789...      ansible-node         Up 2 minutes        
jkl012...      ansible-node         Up 2 minutes        
```

### Step 5: Enter the Control Container
```bash
docker-compose exec ansible bash
```

You're now inside the control node. Your prompt changes to: `root@control:/ansible#`

### Step 6: Verify Connectivity (Your First Test!)
```bash
# Inside the control container
ansible all -i playbooks/inventory.ini -m ping
```

Expected output:
```
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

**Congrats!** Ansible can talk to all nodes. You're ready to automate. 🎉

### Step 7: Run Your First Playbook
```bash
# Inside the control container
ansible-playbook -i playbooks/inventory.ini playbooks/site.yml
```

This executes the main playbook across all nodes. Check the output—tasks should execute successfully.

### Step 8: Exit and Explore
```bash
exit
```

You're back on your host machine. Now try:
```bash
# View logs from a specific node
docker-compose logs node1

# Stop the lab (containers still exist)
docker-compose stop

# Start it again
docker-compose up -d

# Completely remove everything (careful!)
docker-compose down
```

---

## 6. What You Will Automate

Your playbooks will automate the entire Linux server setup pipeline in 4 progressive stages:

### Stage 1: Base Setup (`stage1_base_setup.yml`)
✅ System package updates  
✅ Install essential tools (vim, curl, wget, git, etc.)  
✅ Set timezone and NTP sync  
✅ Configure system limits and kernel parameters  

**Why:** Every new server starts here. Updates patch security holes; tools are needed for later work.

### Stage 2: User Management (`stage2_user_management.yml`)
✅ Create application users  
✅ Set up sudo access without passwords (for automation)  
✅ Create home directories with proper permissions  
✅ Disable root SSH login  

**Why:** Best practice: apps run as non-root users. Automation needs passwordless sudo.

### Stage 3: Security Hardening (`stage3_security_hardening.yml`)
✅ SSH hardening (disable password auth, use keys only)  
✅ UFW firewall setup (only allow ports 22, 80, 443)  
✅ fail2ban DDoS protection (auto-ban repeat attackers)  
✅ Disable unnecessary services  
✅ Set up automatic security updates  

**Why:** Once servers are live, attacks happen immediately. These protect against common threats.

### Stage 4: Multi-Environment Variables (`stage4_multi_env_variables.yml`)
✅ Environment-specific configs (dev, staging, production)  
✅ Application deployment readiness  
✅ Monitoring agent installation  

**Why:** Real-world playbooks adapt based on environment. Same playbook, different outcomes.

---

## 7. Useful Commands

### Quick Reference Table

#### Docker & Compose Commands
| Command | Purpose |
|---------|---------|
| `docker-compose up -d --build` | Start all containers, build if needed |
| `docker-compose down` | Stop and remove all containers |
| `docker-compose logs -f ansible` | Stream control node logs |
| `docker-compose logs node1` | View specific node logs |
| `docker-compose exec ansible bash` | Enter control node shell |
| `docker-compose exec node1 bash` | Enter managed node shell |
| `docker-compose ps` | List running containers |

#### Ansible Commands (run inside control container)

| Command | Purpose |
|---------|---------|
| `ansible all -i playbooks/inventory.ini -m ping` | Test connectivity to all nodes |
| `ansible all -i playbooks/inventory.ini -m setup` | Gather system facts from all nodes |
| `ansible-playbook -i playbooks/inventory.ini playbooks/site.yml` | Run main playbook |
| `ansible-playbook -i playbooks/inventory.ini playbooks/stage1_base_setup.yml` | Run only stage 1 |
| `ansible all -i playbooks/inventory.ini -a "uptime"` | Run ad-hoc command: uptime on all |
| `ansible all -i playbooks/inventory.ini -a "whoami"` | Ad-hoc: current user |
| `ansible-playbook -i playbooks/inventory.ini playbooks/site.yml -vvv` | Verbose output (debug) |
| `ansible-playbook -i playbooks/inventory.ini playbooks/site.yml --syntax-check` | Validate playbook syntax |

#### Common Workflows

**Reset everything and start fresh:**
```bash
docker-compose down && docker-compose up -d --build
# Wait 15 seconds, then:
docker-compose exec ansible bash
```

**Edit a playbook, then test immediately:**
```bash
# On your host machine:
# Edit playbooks/site.yml in your text editor
# Then:
docker-compose exec ansible ansible-playbook -i playbooks/inventory.ini playbooks/site.yml
```

**Debug a failed task:**
```bash
# Run with extra verbosity
ansible-playbook -i playbooks/inventory.ini playbooks/site.yml -vvv

# Or run just one task with debugging
ansible-playbook -i playbooks/inventory.ini playbooks/site.yml --start-at-task="Task Name"
```

---

## 8. Learning Roadmap

### After This Project – Your Next Steps

Once you've mastered the Ansible Linux setup lab, you're ready for:

#### Phase 2: Web Stack Deployment
- Deploy a multi-tier web app (Nginx + Python + PostgreSQL + Redis)
- Use Ansible roles for each component
- Practice playbooks for common operations (deploy, rollback, update)

#### Phase 3: Docker Container Provisioning
- Use Ansible to deploy Docker containers across multiple hosts
- Learn docker-compose provisioning via Ansible
- Understand orchestration basics (swarm mode, container networking)

#### Phase 4: Cloud Infrastructure Automation
- Deploy to AWS/Azure/GCP via Ansible modules
- Provision EC2 instances, load balancers, databases
- Combine Ansible with Terraform for full IaC pipelines

#### Phase 5: Configuration Management at Scale
- Galera MySQL clusters (multi-node replication)
- Kubernetes cluster setup
- Monitoring stacks (Prometheus + Grafana automated setup)

#### Career Path
- **Junior DevOps/SRE** — Operational automation is your bread and butter
- **Platform Engineer** — Build self-service deployment platforms using Ansible
- **Cloud Architect** — Use Ansible as part of large-scale infrastructure management

---

## Troubleshooting

**Q: "Connection refused" when running playbooks?**  
A: Wait 15-20 seconds after `docker-compose up` for SSH servers to fully boot.

**Q: "Host key verification failed"?**  
A: SSH keys aren't yet trusted. Run:
```bash
docker-compose exec ansible ssh-keyscan node1 node2 node3 >> ~/.ssh/known_hosts
```

**Q: Changes to playbooks aren't showing up?**  
A: They should be instant (volume-mounted). Restart the container if stuck:
```bash
docker-compose restart ansible
```

**Q: Can't ssh into nodes from control container?**  
A: Verify SSH keys exist:
```bash
docker-compose exec ansible ls -la ~/.ssh/
```

Should show `id_rsa` and `id_rsa.pub`. If missing, the Dockerfile setup failed—rebuild:
```bash
docker-compose down && docker-compose up -d --build
```

---

## Resources

- **Ansible Official Docs:** https://docs.ansible.com/
- **Linux User Management:** https://www.linux.com/training-tutorials/
- **UFW Firewall Guide:** https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04
- **fail2ban Setup:** https://www.fail2ban.org/wiki/index.php/Main_Page

---

## Contributing & Next Steps

This is your learning lab! Suggestions:
- Add more playbooks for advanced tasks
- Experiment with roles and templates
- Add a 4th stage: web application deployment
- Set up a monitoring playbook (Prometheus + Grafana)

Happy automating! 🚀
