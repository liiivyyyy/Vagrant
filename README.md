# Clúster Vagrant + Ansible con balanceador nginx

Laboratorio de alta disponibilidad básica: tres máquinas virtuales Ubuntu gestionadas con Vagrant,
aprovisionadas con Ansible, donde dos servidores web nginx sirven una página estática y un tercer
nginx actúa como balanceador de carga.

## Arquitectura

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

| Máquina       | IP              | Memoria | Rol                                  |
|---------------|-----------------|---------|--------------------------------------|
| `balanceador` | 192.168.56.100  | 512 MB  | nginx como proxy inverso / balanceo  |
| `nodo1`       | 192.168.56.101  | 1024 MB | nginx sirviendo `index.html`         |
| `nodo2`       | 192.168.56.102  | 1024 MB | nginx sirviendo `index.html`         |

La caja utilizada es `ubuntu/jammy64` (Ubuntu 22.04 LTS).

## Requisitos

- [Vagrant](https://developer.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/) instalado en el host
  (Vagrant lo ejecuta como *provisioner*)

## Uso

```bash
# Levantar las tres máquinas y ejecutar el aprovisionamiento
vagrant up

# Comprobar el estado
vagrant status

# Acceder a una máquina
vagrant ssh nodo1

# Volver a ejecutar Ansible sobre las máquinas ya creadas
vagrant provision

# Apagar / destruir el entorno
vagrant halt
vagrant destroy -f
```

Una vez aprovisionado, abre <http://localhost:8080> en el navegador: cada recarga debe mostrar
`Respondiendo desde nodo1` o `Respondiendo desde nodo2`, alternando entre ambos.

## Qué hace el aprovisionamiento

`ansible/playbook.yml` se ejecuta sobre las tres máquinas y contiene tres *plays*:

1. **`hosts: all`** — añade las entradas del clúster (`*.olivia.local`) a `/etc/hosts`,
   de modo que los nodos se resuelven por nombre además de por IP.
2. **`hosts: web`** (`nodo1`, `nodo2`) — instala nginx, publica la página que identifica al nodo
   (generada desde la plantilla `index.html.j2` con `inventory_hostname`) y asegura el servicio
   arrancado y habilitado.
3. **`hosts: balanceador`** — instala nginx, elimina el sitio por defecto, despliega
   `ansible/lb.conf` como `/etc/nginx/conf.d/balanceador.conf` y recarga el servicio.
   El balanceo es *round-robin* por defecto hacia `nodo1` y `nodo2`.

## Estructura del repositorio

```
.
├── Vagrantfile            # Definición de las 3 VM, red privada y puerto reenviado
└── ansible/
    ├── playbook.yml       # Aprovisionamiento: hosts, webs y balanceador
    ├── lb.conf            # Configuración nginx del balanceador (upstream + proxy)
    └── index.html.j2      # Plantilla de la página servida por cada nodo
```

## Notas

- El puerto 80 del `balanceador` se reenvía al **8080** del host para evitar conflictos.
- La carpeta `.vagrant/` la genera Vagrant automáticamente y no debe subirse al repositorio.
