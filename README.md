# Vagrant + Ansible cluster with an nginx load balancer

Basic high-availability lab: three Ubuntu virtual machines managed with Vagrant and provisioned
with Ansible, where two nginx web servers serve a static page and a third nginx acts as a load
balancer.

## Architecture

```
                      host:8080
                          │
                          ▼
              ┌───────────────────────┐
              │  balanceador (nginx)  │   192.168.56.100
              └───────────┬───────────┘
                 round-robin│
             ┌────────────┴────────────┐
             ▼                         ▼
      ┌─────────────┐           ┌─────────────┐
      │ nodo1       │           │ nodo2       │
      │ nginx :80   │           │ nginx :80   │
      └─────────────┘           └─────────────┘
       192.168.56.101            192.168.56.102
```

| Machine       | IP              | Memory  | Role                                 |
|---------------|-----------------|---------|--------------------------------------|
| `balanceador` | 192.168.56.100  | 512 MB  | nginx as reverse proxy / load balancer |
| `nodo1`       | 192.168.56.101  | 1024 MB | nginx serving `index.html`           |
| `nodo2`       | 192.168.56.102  | 1024 MB | nginx serving `index.html`           |

The base box is `ubuntu/jammy64` (Ubuntu 22.04 LTS).

## Requirements

- [Vagrant](https://developer.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/) installed on the host
  (Vagrant runs it as a *provisioner*)

## Usage

```bash
# Bring up the three machines and run provisioning
vagrant up

# Check status
vagrant status

# SSH into a machine
vagrant ssh nodo1

# Re-run Ansible against the existing machines
vagrant provision

# Shut down / destroy the environment
vagrant halt
vagrant destroy -f
```

Once provisioned, open <http://localhost:8080> in your browser: each reload should show
`Respondiendo desde nodo1` or `Respondiendo desde nodo2`, alternating between the two.

## What provisioning does

`ansible/playbook.yml` runs on all three machines and contains three *plays*:

1. **`hosts: all`** — adds the cluster entries (`*.olivia.local`) to `/etc/hosts`, so nodes can be
   resolved by name as well as by IP.
2. **`hosts: web`** (`nodo1`, `nodo2`) — installs nginx, publishes the page that identifies the node
   (generated from the `index.html.j2` template using `inventory_hostname`) and makes sure the
   service is started and enabled.
3. **`hosts: balanceador`** — installs nginx, removes the default site, deploys
   `ansible/lb.conf` as `/etc/nginx/conf.d/balanceador.conf` and reloads the service.
   Load balancing is *round-robin* by default across `nodo1` and `nodo2`.

## Repository structure

```
.
├── Vagrantfile            # Defines the 3 VMs, private network and forwarded port
└── ansible/
    ├── playbook.yml       # Provisioning: hosts, web servers and load balancer
    ├── lb.conf            # nginx config for the load balancer (upstream + proxy)
    └── index.html.j2      # Template for the page served by each node
```

## Notes

- Port 80 on the `balanceador` is forwarded to port **8080** on the host to avoid conflicts.
- The `.vagrant/` directory is generated automatically by Vagrant and should not be committed.
