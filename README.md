# ansible

Playbook de Ansible para provisionar una Fedora Workstation (KDE Plasma)
nueva con el stack DevOps + CLI diario. Corre siempre contra `localhost`
(`ansible_connection=local`) — no gestiona servidores remotos.

## Uso

```bash
# Provisioning completo
ansible-playbook -i inventory.ini playbook.yml -K

# Solo una sección
ansible-playbook -i inventory.ini playbook.yml -K --tags docker

# Todo menos una sección
ansible-playbook -i inventory.ini playbook.yml -K --skip-tags vscode

# Validar sintaxis sin ejecutar
ansible-playbook -i inventory.ini playbook.yml --syntax-check

# Ver qué tareas correrían sin aplicarlas
ansible-playbook -i inventory.ini playbook.yml -K --check --diff
```

`-K` pide la password de sudo (`become`).

## Tags disponibles

| Tag           | Sección                      | Notas                                       |
|---------------|-------------------------------|-----------------------------------------------|
| `base`        | 1. Sistema base y repos        | dnf-plugins-core + paquetes base              |
| `cli`         | 2. Terminal / CLI tools        | zsh, tmux, oh-my-zsh, etc.                    |
| `zsh-plugins` | 3. Plugins de zsh / oh-my-zsh  | autosuggestions, syntax-highlighting, etc.    |
| `docker`      | 4. Docker + Docker Compose     | repo oficial + engine + grupo docker          |
| `ansible`     | 5. Ansible                     | para gestionar la PC desde sí misma           |
| `tailscale`   | 6. Tailscale                   | repo + servicio habilitado                    |
| `starship`    | 7. Starship prompt             | instala e integra en `.zshrc`                 |
| `node`        | 8. Node.js                     | repo NodeSource + Node + PM2 global           |
| `claude-code` | 9. Claude Code + skill superpowers | npm global + `claude plugin install`      |
| `vscode`      | 10. VSCode                     | opcional, comentar si no se usa en la PC      |
| `suspend-fix` | 11. Fix suspend/resume i915 PSR| tag `never` — solo corre con `--tags` explícito |

## Notas por sección

- **cli**: oh-my-zsh se instala solo si no existe ya (`stat`/`when`), y el
  shell por defecto del usuario se cambia a zsh.
- **zsh-plugins**: clona `zsh-autosuggestions` y `zsh-syntax-highlighting`,
  activa los plugins en `.zshrc` y arma el `fpath`/`compinit` para
  completions (compartido con `starship`).
- **starship**: la instalación se guarda con `stat`/`when`; genera su
  archivo de completions con `creates:` y el `eval` en `.zshrc` se agrega
  con `become: false` para no dejar el archivo root-owned.
- **node**: el repo NodeSource se agrega con `creates:` como guardia de
  idempotencia sobre el script `curl | bash`.
- **claude-code**: instala el CLI vía `npm -g` (depende de que `node`
  haya corrido antes) y el plugin/skill `superpowers@claude-plugins-official`
  con `claude plugin install`, guardado con `claude plugin list` +
  `when` para no reinstalar si ya está.
- **vscode**: sección opcional — comentarla si esta PC no la necesita.
- **suspend-fix**: fix puntual para el bug conocido de kwin_wayland + i915
  PSR (pantalla negra en resume, iGPU Intel UHD 630/CometLake y
  similares). Tiene tag `never` además del propio, así que nunca corre en
  un run normal — solo con
  `ansible-playbook -i inventory.ini playbook.yml -K --tags suspend-fix`.
  Requiere reboot para tomar efecto.

Ver [`claude.md`](claude.md) para las convenciones internas del playbook
(idempotencia, `become`, numeración de secciones, etc.).
