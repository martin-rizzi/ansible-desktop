# CLAUDE.md

Guía para Claude Code al trabajar en este repo. Léela antes de tocar el playbook.

## Qué es esto

Playbook de Ansible para provisionar una Fedora Workstation (KDE Plasma)
nueva con el stack DevOps + CLI diario del dueño del repo. Corre siempre
contra `localhost` (`ansible_connection=local`), no gestiona servidores
remotos — es exclusivamente para setup de escritorio personal.

## Estructura

```
playbook.yml    # todo el provisioning, en un solo archivo, por secciones numeradas
inventory.ini   # localhost únicamente
README.md       # uso, tags disponibles, notas por sección
```

No dividir en roles ni en múltiples playbooks a menos que se pida
explícitamente — el archivo único es intencional, es más fácil de
leer de arriba a abajo para un solo host.

## Convenciones del playbook

- **Todo tarea tiene `tags`.** Cada sección numerada en los comentarios
  (`# N. Nombre`) corresponde a un tag (`base`, `cli`, `zsh-plugins`,
  `docker`, `ansible`, `tailscale`, `starship`, `node`, `claude-code`,
  `vscode`, `suspend-fix`). Al agregar una sección nueva, sumale su tag y
  actualizá la lista de tags en el README.
- **Numeración de comentarios en orden.** Si se inserta una sección en
  el medio, renumerar los comentarios `# N. ...` de las secciones
  siguientes para que quede correlativo.
- **Idempotencia real, no solo la que da el módulo `dnf`.** Para
  instaladores por `curl | sh` (starship, oh-my-zsh) siempre se
  antepone un `stat`/`register`/`when: not X.stat.exists`, o se usa
  `args: creates: <path>`. No agregar un `shell`/`command` sin guardia
  de idempotencia.
- **`become: false` en tareas que tocan el home del usuario** (`.zshrc`,
  clones de plugins, dotfiles). El playbook global corre con
  `become: true`, así que cada tarea de usuario tiene que desactivarlo
  explícitamente o los archivos quedan root-owned.
- **Repos vía `get_url`/`yum_repository`, nunca `dnf config-manager
  --add-repo`.** Esta Fedora usa dnf5, que cambió esa sintaxis
  (`addrepo --from-repofile=`). Bajar el `.repo` directo evita
  problemas de compatibilidad entre dnf4/dnf5.
- **Fixes de hardware/bugs puntuales van con el tag `never` además del
  suyo propio** (ver sección `suspend-fix`), para que nunca corran en
  un `ansible-playbook playbook.yml -K` sin `--tags` explícito. Son
  cambios a nivel kernel/sistema, no parte del provisioning base.

## Comandos

```bash
# Provisioning completo
ansible-playbook -i inventory.ini playbook.yml -K

# Solo una sección
ansible-playbook -i inventory.ini playbook.yml -K --tags docker

# Todo menos una sección
ansible-playbook -i inventory.ini playbook.yml -K --skip-tags vscode

# Fixes con tag `never` (no corren solos)
ansible-playbook -i inventory.ini playbook.yml -K --tags suspend-fix

# Validar sintaxis sin ejecutar
ansible-playbook -i inventory.ini playbook.yml --syntax-check

# Ver qué tareas correrían sin aplicarlas
ansible-playbook -i inventory.ini playbook.yml -K --check --diff
```

## Al agregar una herramienta nueva

1. Sección nueva al final (o donde corresponda temáticamente), numerada
   y taggeada.
2. Si agrega un repo externo: `get_url` a `/etc/yum.repos.d/`, nunca
   `command` con `config-manager --add-repo`.
3. Si es un instalador `curl | sh`: guardia de idempotencia (`stat` +
   `when`, o `creates`).
4. Si toca `.zshrc` u otro dotfile: `become: false` y usar
   `lineinfile`/`blockinfile` con `regexp`/`marker` único — no
   appendear a ciegas, para que el playbook siga siendo re-ejecutable
   sin duplicar líneas.
5. Actualizar el README: sección de qué instala, tag nuevo en la lista
   de tags disponibles, y nota si tiene alguna particularidad (como
   `suspend-fix` con `never`).

## Qué NO hacer

- No usar `sudo` a mano dentro de comandos `shell`/`command` — el
  `become: true`/`become: false` de la tarea ya controla los
  privilegios.
- No asumir Ubuntu/Debian: es Fedora, dnf5. Nada de `apt`.
- No instalar kubectl/terraform salvo pedido explícito — se sacaron
  del playbook a propósito porque no se usan todavía.
- No cambiar `hosts: fedora_local` ni el `inventory.ini` — este repo
  es de un solo host local, no un fleet.
