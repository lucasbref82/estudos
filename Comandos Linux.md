
# Comandos Linux (Ubuntu) — Básicos a Avançados
**Aviso:** é impossível listar *todos* os comandos possíveis no Linux (existem milhares de utilitários, pacotes e programas de terceiros).  
Este documento contém uma **lista extensa e prática** dos comandos mais úteis — básicos, avançados, combinações e práticas — para administração e uso cotidiano em Ubuntu. Ao final há formas de listar tudo que existe na sua instalação.

---

## Índice
1. Navegação e manipulação de arquivos
2. Visualização e edição de texto
3. Permissões, usuários e grupos
4. Processos e gerenciamento
5. Disco, arquivos e sistemas de arquivos
6. Rede e diagnóstico
7. Pacotes e atualizações (APT, Snap)
8. Systemd e serviços
9. Ferramentas de desenvolvedor e compilação
10. Containers e virtualização
11. Shell (Bash) — builtins, redirecionamentos e teclas
12. Composições/Combinações úteis (pipes, xargs, find + exec)
13. Segurança, logs e auditoria
14. Agendamento de tarefas
15. Dicas para listar todos os comandos do sistema
16. Recursos / Como aprender mais

---

## 1) Navegação e manipulação de arquivos
- `pwd` — mostra diretório atual.
- `ls` — lista arquivos (`ls -l`, `ls -a`, `ls -lh`, `ls -R`).
- `cd /caminho` — muda diretório. `cd -` retorna ao anterior. `cd ~` ou `cd` vai para home.
- `cp arquivo destino` — copia. `cp -r dir dir2` recursivo.
- `mv origem destino` — move/renomeia.
- `rm arquivo` — remove. `rm -f` forçar, `rm -r` recursivo. **Cuidado**.
- `mkdir pasta` — cria pasta. `mkdir -p a/b/c` cria recursivamente.
- `rmdir pasta` — remove diretórios vazios.
- `ln -s alvo link` — cria link simbólico.
- `stat arquivo` — informações do arquivo (timestamps, tamanho).
- `file arquivo` — identifica tipo de arquivo.
- `tree` — mostra árvore (pode precisar `sudo apt install tree`).

Exemplo: `cp -av ~/docs /backup/docs_$(date +%F)` (`-a` preserva atributos, `-v` verbose).

---

## 2) Visualização e edição de texto
- Visualizar:
  - `cat arquivo`
  - `tac arquivo` (cat invertido)
  - `less arquivo` — pager (navegação com PgUp/PgDn, /busca).
  - `more arquivo`
  - `head -n 20 arquivo`
  - `tail -n 20 arquivo` (`tail -f arquivo` para acompanhar logs)
- Editores:
  - `nano arquivo` — editor simples.
  - `vim arquivo` — editor poderoso.
  - `emacs arquivo` — poderoso (opcional).
- Processamento:
  - `grep 'padrao' arquivo` (`-i` ignore case, `-r` recursivo, `-E` regex estendida)
  - `egrep` (igual a `grep -E`)
  - `sed` — stream editor (ex.: `sed -n '1,10p' file`, `sed 's/old/new/g' file`)
  - `awk '{print $1,$3}' file` — poderosa linguagem de texto/colunas.
  - `cut -d',' -f1,3` — extrai campos.
  - `tr` — translate chars.
  - `paste` — junta colunas.
  - `sort`, `uniq -c` — ordenar e contar duplicados.
  - `wc -l` — conta linhas.

Exemplo de pipeline: `grep -i error /var/log/syslog | awk '{print $1,$2,$3,$6}' | sort | uniq -c | sort -nr | head`

---

## 3) Permissões, usuários e grupos
- `whoami` — usuário atual.
- `id usuario` — mostra uid/gid e grupos.
- `sudo comando` — executa como root (requer estar em sudoers).
- `su -` — troca para root (precisa senha root ou ser configurado).
- `useradd -m -s /bin/bash novo_usuario` — adiciona usuário.
- `adduser novo_usuario` — fluxo interativo (Ubuntu).
- `passwd usuario` — altera senha.
- `usermod -aG grupo usuario` — adiciona a um grupo.
- `groupadd grupo`, `groupdel grupo`
- `chown usuário:grupo arquivo` — altera dono.
- `chmod` — muda permissões (`chmod 644 arquivo`, `chmod u+rwx,g+rx,o-rw arquivo`).
- `getfacl` / `setfacl` — ACLs (se filesystem suportar).
- `umask` — máscara padrão para novos arquivos.

Exemplo: `sudo chown -R www-data:www-data /var/www` para dar propriedade recursiva.

---

## 4) Processos e gerenciamento
- `ps aux` — lista processos.
- `top` — monitor interativo.
- `htop` — versão melhor (instalar `sudo apt install htop`).
- `pgrep nome` — procura PIDs por nome.
- `pkill nome` — mata por nome.
- `kill PID`, `kill -9 PID` (SIGKILL).
- `nice -n 10 comando` — ajusta prioridade ao iniciar.
- `renice -n -5 -p PID` — altera prioridade de processo existente.
- `nohup comando &` — roda em background, ignorando hangups.
- `disown` — remove job do shell para não receber SIGHUP.
- `jobs`, `fg`, `bg` — controle de jobs do shell.
- `strace -p PID` — trace de syscalls (debug).
- `lsof` — lista arquivos abertos (ex.: `lsof -i :80` mostra processo na porta 80).

Exemplo: `nohup ./servidor > server.log 2>&1 & echo $! > servidor.pid` — executa em background, redireciona saídas, salva PID.

---

## 5) Disco, arquivos e sistemas de arquivos
- `df -h` — uso de disco por filesystem.
- `du -sh pasta` — tamanho de pasta (`du -h --max-depth=1`).
- `mount` / `umount` — montar/desmontar (pode usar `udisksctl` para desktop).
- `blkid`, `lsblk` — identificar dispositivos e partições.
- `fdisk -l` — particionamento (ou `parted` / `gdisk`).
- `mkfs.ext4 /dev/sdX1` — criar filesystem (CUIDADO).
- `fsck /dev/sdX1` — checar filesystem (em modo apropriado).
- `resize2fs` — redimensionar ext2/3/4 (com cautela).
- `cryptsetup` — LUKS criptografia de discos.
- `mount -o loop arquivo.iso /mnt` — montar ISO.
- `tune2fs`, `dumpe2fs` — ferramentas ext.

Exemplo: `du -sh /var/* | sort -h` — ver quais subpastas de /var ocupam mais espaço.

---

## 6) Rede e diagnóstico
- `ip addr show` — informações de IP.
- `ip link show` — interfaces.
- `ip route` — tabela de roteamento.
- `ss -tulpn` — sockets abertos e serviços escutando (substitui netstat).
- `netstat -tulpn` — (pode não estar instalado).
- `ping host`
- `traceroute host` (instalar `traceroute`).
- `curl`, `wget` — transferências HTTP.
- `dig dominio` — consulta DNS (instalar `dnsutils`).
- `nslookup`, `host`
- `tcpdump -i eth0` — captura de pacotes (requer sudo).
- `nmap` — scanner de portas (instalar).
- `iptables -L` / `nft list ruleset` — firewall (nftables substitui iptables em várias distros).
- `ufw status` — Uncomplicated Firewall (Ubuntu) — `ufw enable`, `ufw allow 22/tcp`, `ufw deny 80`.
- `nmcli` — NetworkManager CLI (configura conexões).
- `systemd-resolve --status` — resolver/systemd-resolved info.

Exemplo: `ss -tulpn | grep :80` — ver qual processo escuta porta 80.

---

## 7) Pacotes e atualizações (APT, Snap)
- `sudo apt update` — atualiza índices.
- `sudo apt upgrade` — atualiza pacotes instalados.
- `sudo apt full-upgrade` — pode remover dependências obsoletas.
- `sudo apt install pacote`
- `sudo apt remove pacote`
- `sudo apt purge pacote` — remove configs.
- `sudo apt autoremove` — limpa dependências não usadas.
- `apt-cache search termo` / `apt search termo`
- `dpkg -i pacote.deb` — instalar .deb diretamente.
- `dpkg -l | grep pacote`
- `apt-mark hold pacote` — bloquear versão.
- `snap install nome` / `snap list` / `snap remove nome`
- `flatpak` — se estiver usando Flatpak (`flatpak install flathub app`).

Exemplo: `sudo apt install htop curl git` — instalar múltiplos pacotes.

---

## 8) Systemd e serviços
- `systemctl status nome.service`
- `systemctl start nome.service`
- `systemctl stop nome.service`
- `systemctl restart nome.service`
- `systemctl enable nome.service` — inicia na inicialização.
- `systemctl disable nome.service`
- `systemctl daemon-reload` — recarrega units depois de editar.
- `journalctl -u nome.service` — logs por unit.
- `journalctl -f` — acompanhar journal em tempo real.
- `systemctl list-units --type=service --state=failed`
- `systemd-analyze blame` — tempos de boot.
- `loginctl` — sessões/usuários logados via systemd-logind.

Exemplo: `sudo systemctl restart apache2 && sudo journalctl -u apache2 -n 200 --no-pager` — reinicia e mostra últimos logs.

---

## 9) Ferramentas de desenvolvedor e compilação
- `gcc` / `g++` — compiladores.
- `make` — build automation.
- `cmake` — gerador de builds.
- `git` — controle de versão.
- `gdb` — debugger.
- `strace`, `ltrace` — tracing.
- `valgrind` — análise de memória.
- `ldd` — dependências de bibliotecas compartilhadas (`ldd binario`).
- `file` — identifica binários.
- `readelf -h binario` — info ELF.
- `objdump -d binario` — disassembler.

---

## 10) Containers e virtualização (comandos comuns)
- Docker: `docker run`, `docker ps`, `docker images`, `docker build -t nome .`, `docker exec -it container /bin/bash`, `docker logs`.
  - Ex.: `docker run -d -p 8080:80 --name web nginx`
- Podman: similar ao Docker (`podman`).
- LXD/LXC: `lxc launch ubuntu:22.04 myctnr`, `lxc list`, `lxc exec`.
- Multipass (Canonical): `multipass launch --name vm1`
- VirtualBox / `virsh` (libvirt): `virsh list --all`, `virt-manager` GUI.

---

## 11) Shell (Bash) — builtins, redirecionamentos e teclas
- Builtins: `cd`, `echo`, `read`, `export VAR=valor`, `alias ll='ls -la'`, `unalias ll`, `type comando`, `help [builtin]`, `builtin`.
- Redirecionamentos:
  - `>` — sobrescreve stdout para arquivo.
  - `>>` — append.
  - `2>` — stderr para arquivo.
  - `&>` ou `> file 2>&1` — stdout+stderr.
  - `<` — stdin a partir de arquivo.
- Pipes:
  - `comando1 | comando2 | comando3`
- Substituição:
  - Command substitution: `$(comando)` ou `` `comando` ``
  - Process substitution: `<(comando)` e `>(comando)` (útil em diff, e.g., `diff <(ls dir1) <(ls dir2)`)
- Operadores lógicos:
  - `cmd1 && cmd2` — executa cmd2 se cmd1 ok (exit 0).
  - `cmd1 || cmd2` — executa cmd2 se cmd1 falhar.
  - `cmd1 ; cmd2` — executa ambos, independente do status.
- Background:
  - `comando &` — roda em background.
  - `disown %1` — desassocia job.
- Variáveis e arrays: `arr=(a b c)`, `${arr[0]}`, `${#arr[@]}`.
- Expansões: brace expansion `file{1..3}.txt`, filename globbing `*.log`.
- History:
  - `history`
  - `!n` (executa história), `!!` repete último.
  - `!grep` re-executa último comando que começou com grep.
- Atalhos do teclado (Bash/readline):
  - `Ctrl+C` — SIGINT (aborta).
  - `Ctrl+Z` — suspende (SIGTSTP).
  - `fg`, `bg` — foreground/background.
  - `Ctrl+D` — EOF (fecha shell se vazio).
  - `Ctrl+R` — reverse-i-search (busca no histórico).
  - `Ctrl+A` (início da linha), `Ctrl+E` (fim da linha).
  - `Alt+F`/`Alt+B` — mover palavra adiante/atrás.

Exemplo de substitution: `FILES=$(find . -type f -name '*.log' -mtime -7)`.

---

## 12) Composições/Combinações úteis (receitas)
- Encontrar arquivos e executar comando:
  - `find /var/log -type f -name '*.log' -mtime +30 -exec rm {} \;`
  - Mais eficiente: `find ... -print0 | xargs -0 rm -f`
- Buscar e compactar:
  - `find /path -type f -name '*.txt' -print0 | tar --null -czvf txts.tar.gz --files-from -`
- Usar xargs:
  - `cat list.txt | xargs -n1 -P4 command` (paraleliza com -P)
- Backup incremental com rsync:
  - `rsync -av --delete /origem/ /destino/`
- Extração e compressão:
  - `tar -cvzf arquivo.tar.gz pasta/`
  - `tar -xvzf arquivo.tar.gz`
  - `zip`, `unzip`
- Buscar uso de disco e remover maiores:
  - `du -ah / | sort -rh | head -n 30`
- Log rotate manual:
  - `logrotate -f /etc/logrotate.conf`
- Verificação rápida de portas remotas:
  - `curl -I http://example.com` ou `nc -zv host port`

Exemplo complexo: encontrar arquivos modificados nas últimas 24h, compactar e copiar:
```
find /var/www -type f -mtime -1 -print0 | tar --null -T - -czvf /tmp/www_changes_$(date +%F).tar.gz
scp /tmp/www_changes_$(date +%F).tar.gz user@backup:/backups/
```

---

## 13) Segurança, logs e auditoria
- `journalctl -xe` — ver mensagens de erro recentes.
- `sudo ausearch -m avc -ts recent` — se auditd ativado (auditoria SELinux/AppArmor).
- `auditctl` / `ausearch` — auditd.
- `fail2ban-client status` — se fail2ban instalado.
- `ufw` / `iptables` / `nft` — firewall.
- `getenforce` — SELinux (não padrão no Ubuntu).
- `aa-status` — AppArmor status.
- `last`, `lastlog` — logins recentes.
- `who`, `w` — usuários conectados.

---

## 14) Agendamento
- `crontab -e` — editar crontab do usuário.
- `sudo crontab -e` — crontab root.
- Linha cron: `0 3 * * * /path/to/backup.sh`
- `at` / `batch` — agendamentos pontuais (instalar `atd`).
- systemd timers: criar unit `.timer` e `.service` para maior controle.

Exemplo: `echo "0 2 * * * /usr/local/bin/backup" | crontab -`

---

## 15) Como listar tudo que existe no seu sistema (comandos possíveis)
- `compgen -c` — lista comandos conhecidos pelo shell (builtins + PATH).
- `compgen -a` — aliases.
- `compgen -b` — builtins.
- `compgen -k` — keywords.
- `ls /bin /usr/bin /usr/local/bin /sbin /usr/sbin` — listar programas instalados nesses diretórios.
- `apt list --installed` — lista pacotes instalados (de onde vêm muitos binários).
- `whereis comando`, `which comando`, `type comando`, `man -k termo` (`apropos termo`).
- `find / -type f -executable -printf '%p\n'` — procura binários executáveis (pode ser lento).

---

## 16) Dicas, boas práticas e segurança
- Teste comandos perigosos com `echo` antes de rodar (ex.: `echo rm -rf /tmp/test/*`).
- Use `--dry-run` quando disponível (rsync, apt, some tools).
- Tenha backups antes de manipular discos/partições.
- Use `sudo` com parcimônia e registre ações (audit).
- Use `screen` ou `tmux` para sessões longas.
- Ao editar arquivos de configuração, faça backup: `cp file file.bak`.
- Leia `man <comando>` e `--help` sempre antes de rodar.

---

## Exemplos rápidos (resumo de sintaxe)
- Redirecionar stdout+stderr: `comando > out.txt 2>&1`
- Run background and get PID: `comando & echo $!`
- Run command on multiple files: `find . -name '*.log' -print0 | xargs -0 gzip`
- Replace in files recursively: `grep -rl 'foo' . | xargs sed -i 's/foo/bar/g'`

---

## Recursos para aprender e referência rápida
- `man comando`
- `info comando`
- `apropos termo`
- Documentação Ubuntu: https://help.ubuntu.com
- TLDR pages: instalar `tldr` (`sudo apt install tldr`) para resumos.

---

## Observação final
Este documento concentra centenas dos comandos e combinações mais usados. Ele **não** inclui todos os pacotes e utilitários de terceiros que podem existir em repositórios (por exemplo ferramentas exóticas, binários especializados e aplicações customizadas). Para obter a lista completa do que está instalado no seu sistema, use `compgen -c` e `apt list --installed` e combine com `ls` nos diretórios de binários.

---

# Fim do documento
