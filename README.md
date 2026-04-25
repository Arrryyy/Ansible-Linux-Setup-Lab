# Ansible Web Stack Deployment

## 1. Project Goal

This project takes you from basic Ansible commands to production-ready infrastructure automation. You'll deploy a complete 3-tier web stack (web server, app server, database) across multiple servers, each configured for its specific role.

**Key Features:**
- Real-world Ansible uses roles for reusable, organized code
- Host groups let you target servers by function (webservers, appservers, dbservers)
- Templates make configs dynamic and environment-aware
- Handlers ensure services restart only when needed
- This is how you automate complex, multi-server applications

**What You'll Build:**
A fully functional web stack where:
- Nginx serves static files and proxies to Flask
- Flask app connects to PostgreSQL database
- Everything is configured via Ansible playbooks
- The stack is ready for production traffic

**Step Up from Project 1:**
Project 1 was about connectivity and basic commands. Here, you'll use Ansible's advanced features to manage a distributed application—roles, templates, and service management.

---

## 2. Architecture

### The Web Stack Setup
Your lab deploys a classic 3-tier architecture:
- **Control Node** — Ansible runs here, orchestrates the deployment
- **node1 (Web Server)** — Nginx web server, handles HTTP requests
- **node2 (App Server)** — Python Flask application, business logic
- **node3 (Database Server)** — PostgreSQL database, data persistence

### How It Connects
- Ansible connects via SSH over the Docker bridge network
- Each node gets role-specific configuration based on its host group
- Nginx proxies requests to Flask, Flask queries PostgreSQL

### ASCII Diagram
```
┌─────────────────────────────────────────────────────────────┐
│                     Your Host Machine                        │
│                                                               │
│  docker-compose up → Deploys the full stack                │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────────┐
         │    Docker Bridge Network (ansible-net) │
         ├────────────────────────────────────────┤
         │                                        │
         │  ┌──────────────────────────────────┐  │
         │  │   Control Node (ansible)         │  │
         │  │   • Ansible core + roles         │  │
         │  │   • /ansible/playbooks mounted   │  │
         │  │   • SSH key for auth             │  │
         │  └──────────────────────────────────┘  │
         │           ↓ SSH to ↙  ↙  ↙             │
         │  ┌──────────────────────────────────┐  │
         │  │  Web Server (node1)              │  │
         │  │  • Nginx installed & configured  │  │
         │  │  • Serves static files           │  │
         │  │  • Proxies to Flask on node2     │  │
         │  └──────────────────────────────────┘  │
         │  ┌──────────────────────────────────┐  │
         │  │  App Server (node2)              │  │
         │  │  • Python Flask app deployed     │  │
         │  │  • Connects to PostgreSQL        │  │
         │  │  • Runs on port 5000             │  │
         │  └──────────────────────────────────┘  │
         │  ┌──────────────────────────────────┐  │
         │  │  Database Server (node3)         │  │
         │  │  • PostgreSQL installed          │  │
         │  │  • Database created for app      │  │
         │  │  • User permissions set          │  │
         │  └──────────────────────────────────┘  │
         │                                        │
         └────────────────────────────────────────┘

Web Request Flow:
User → Nginx (node1:80) → Flask (node2:5000) → PostgreSQL (node3:5432)
```

---

## 3. Prerequisites

**Required:**
- **Project 1 Completed** — You know how to start the lab and run basic Ansible commands
- **Docker & Docker Compose** — Same as Project 1
- **Basic Ansible Knowledge** — Playbooks, inventory, modules
- **Terminal Comfort** — Editing files, running commands

**Optional but Helpful:**
- Understanding of web stacks (client-server-database)
- Basic Python/Flask knowledge
- Familiarity with Nginx config basics

**Verify Setup:**
```bash
docker --version
docker-compose --version
# And test Project 1 commands still work
```

---

## 4. Project Structure

```
Ansible-Web-Stack/
├── README.md                           # This file — project guide
├── docker-compose.yml                  # Docker setup: control + 3 nodes with roles
├── control/                            # Control node Dockerfile & setup
│   ├── Dockerfile                      # Ubuntu + Ansible + SSH
│   └── entrypoint.sh                   # SSH key generation
├── node1/, node2/, node3/              # Managed nodes (web, app, db roles)
│   ├── Dockerfile                      # Ubuntu + SSH + role-specific packages
│   └── entrypoint.sh                   # SSH setup + initial user creation
├── playbooks/                          # YOUR PLAYBOOKS — edit locally
│   ├── site.yml                        # Main playbook (includes all roles)
│   ├── inventory.ini                   # Host inventory with groups
│   ├── group_vars/                     # Group-level variables
│   │   ├── webservers.yml              # Nginx config vars
│   │   ├── appservers.yml              # Flask app vars
│   │   └── dbservers.yml               # PostgreSQL vars
│   └── roles/                          # Ansible roles (NEW!)
│       ├── webserver/                  # Nginx role
│       │   ├── tasks/                  # Main tasks (install, config)
│       │   │   └── main.yml
│       │   ├── handlers/               # Service restarts
│       │   │   └── main.yml
│       │   ├── templates/              # Jinja2 templates
│       │   │   ├── nginx.conf.j2
│       │   │   └── default.conf.j2
│       │   ├── defaults/               # Default variables
│       │   │   └── main.yml
│       │   └── files/                  # Static files (optional)
│       ├── appserver/                  # Flask app role
│       │   ├── tasks/
│       │   │   └── main.yml
│       │   ├── templates/
│       │   │   ├── app.py.j2
│       │   │   └── requirements.txt.j2
│       │   ├── handlers/
│       │   │   └── main.yml
│       │   └── defaults/
│       │     └── main.yml
│       └── dbserver/                   # PostgreSQL role
│           ├── tasks/
│           │   └── main.yml
│           ├── handlers/
│           │   └── main.yml
│           ├── templates/
│           │   ├── postgresql.conf.j2
│           │   └── pg_hba.conf.j2
│           └── defaults/
│               └── main.yml
└── .gitkeep                            # Ensures empty folders are tracked

Key Insights:
- roles/ structure: tasks (what to do), handlers (when to restart), templates (dynamic configs), defaults (variables)
- Host groups in inventory.ini: [webservers], [appservers], [dbservers]
- Templates use Jinja2: {{ variables }} for dynamic content
- Handlers: notify "restart nginx" only when config changes
```

---

## 5. Getting Started

### Step 1: Switch to the Project Branch
```bash
cd /path/to/your/ansible/project
git checkout project-2-web-stack
# Or create it if it doesn't exist:
# git checkout -b project-2-web-stack
```

### Step 2: Verify the Project Structure
```bash
ls -la
# You should see: control/, node1/, node2/, node3/, docker-compose.yml, playbooks/, README.md
```

### Step 3: Build and Start the Web Stack Lab
```bash
docker-compose up -d --build
```

This builds role-specific images:
- node1: Ubuntu + Nginx packages
- node2: Ubuntu + Python packages
- node3: Ubuntu + PostgreSQL packages

**Wait 20-30 seconds for services to initialize.**

### Step 4: Verify All Containers Are Running
```bash
docker-compose ps
```

Should show control, node1, node2, node3 as "Up".

### Step 5: Enter the Control Container
```bash
docker-compose exec control bash
```

You're now in the control node at `/ansible#`.

### Step 6: Test Connectivity
```bash
ansible all -i /ansible/inventory.ini -m ping
```

All nodes should respond "pong".

### Step 7: Run the Web Stack Deployment
```bash
ansible-playbook -i /ansible/inventory.ini /ansible/site.yml
```

This applies roles to each host group:
- webservers (node1) → Nginx role
- appservers (node2) → Flask role  
- dbservers (node3) → PostgreSQL role

### Step 8: Verify the Stack Works
From your host machine:
```bash
# Check if Nginx is serving
curl http://localhost  # Should proxy to Flask
```

Or enter a node to inspect:
```bash
docker-compose exec node1 bash
# Check Nginx status: systemctl status nginx
```

### Step 9: Explore and Experiment
- Edit templates in `roles/*/templates/`
- Modify variables in `group_vars/`
- Re-run the playbook to see changes

---

## 6. How It Works

### Host Groups in Inventory
Instead of targeting "all" nodes the same way, you group them by function:
```
[webservers]
node1

[appservers]  
node2

[dbservers]
node3
```

Playbook then applies roles based on groups:
```yaml
- hosts: webservers
  roles:
    - webserver

- hosts: appservers
  roles:
    - appserver
```

### Ansible Roles
Roles organize code into reusable components:
- **tasks/main.yml** — What to do (install packages, copy files)
- **handlers/main.yml** — When to restart services (only if config changed)
- **templates/** — Jinja2 files with `{{ variables }}` for dynamic configs
- **defaults/main.yml** — Default variables (overridable)

### Jinja2 Templates
Templates generate configs dynamically:
```jinja2
# nginx.conf.j2
server {
    listen {{ nginx_port }};
    server_name {{ domain_name }};
    # ...
}
```

Variables come from `defaults/main.yml` or `group_vars/`.

### Handlers
Handlers run only when notified:
```yaml
# In tasks
- name: Copy nginx config
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
  notify: restart nginx

# In handlers  
- name: restart nginx
  service: name=nginx state=restarted
```

### Service Module
Manage services declaratively:
```yaml
- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

---

## 7. Useful Commands

### Docker & Compose
| Command | Purpose |
|---------|---------|
| `docker-compose up -d --build` | Start stack, build images |
| `docker-compose down` | Stop and remove all |
| `docker-compose logs -f node1` | Stream node1 logs |
| `docker-compose exec control bash` | Enter control node |
| `docker-compose exec node1 bash` | Inspect web server |

### Ansible (from control container)
| Command | Purpose |
|---------|---------|
| `ansible all -i /ansible/inventory.ini -m ping` | Test all nodes |
| `ansible webservers -i /ansible/inventory.ini -m ping` | Test web group |
| `ansible-playbook -i /ansible/inventory.ini /ansible/site.yml` | Run full deployment |
| `ansible-playbook -i /ansible/inventory.ini /ansible/site.yml --tags webserver` | Run only web role |
| `ansible-playbook -i /ansible/inventory.ini /ansible/site.yml -vvv` | Verbose output |

### Role Development
| Command | Purpose |
|---------|---------|
| `ansible-playbook -i /ansible/inventory.ini /ansible/site.yml --check` | Dry run (see what would change) |
| `ansible-playbook -i /ansible/inventory.ini /ansible/site.yml --diff` | Show config differences |
| Edit `roles/webserver/defaults/main.yml` | Change default variables |
| Edit `roles/webserver/templates/nginx.conf.j2` | Modify Nginx config template |

### Debugging
| Command | Purpose |
|---------|---------|
| `docker-compose logs control` | Check Ansible output |
| `docker-compose exec node1 systemctl status nginx` | Check service status |
| `docker-compose exec node2 python3 /opt/app/app.py` | Test Flask app manually |
| `docker-compose exec node3 psql -U appuser -d appdb` | Connect to PostgreSQL |

---

## 8. What’s Next

### Project 3: Docker Provisioning with Ansible
Take automation to the next level:
- Use Ansible to deploy Docker containers across your infrastructure
- Manage containerized applications at scale
- Combine Ansible with Docker Compose for full stack orchestration
- Learn container networking, volumes, and service discovery

### Career Progression
- **DevOps Engineer** — You're automating deployments now
- **Site Reliability Engineer** — Roles and templates are your toolkit
- **Platform Engineer** — Build self-service infrastructure platforms
- **Cloud Architect** — Use Ansible as part of large-scale infrastructure management

### Advanced Topics to Explore
- Ansible Tower/AWX for GUI management
- Molecule for role testing
- Ansible Vault for secrets management
- Dynamic inventories from cloud providers

You've got the web stack running—now scale it up!

---

## Troubleshooting

**Q: "Failed to connect to the host via ssh"?**  
A: Wait longer after `docker-compose up`—SSH services take time to start.

**Q: Role tasks not running?**  
A: Check inventory groups match playbook hosts. Use `ansible webservers -m ping`.

**Q: Template variables not working?**  
A: Verify variables in `defaults/main.yml` or `group_vars/`. Use `ansible-playbook --check`.

**Q: Services not starting?**  
A: Check handlers fired: `docker-compose logs node1 | grep restart`.

---

## Resources

- **Ansible Roles Documentation:** https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html
- **Jinja2 Templates:** https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html
- **Nginx Configuration:** https://nginx.org/en/docs/
- **Flask Deployment:** https://flask.palletsprojects.com/en/2.3.x/deploying/

---

Happy deploying! This is where Ansible gets real.
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

Happy automating!
