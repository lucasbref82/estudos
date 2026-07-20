
# Linux (Ubuntu) — Comandos adicionais e combinações avançadas
**Objetivo:** complementar o documento anterior com comandos e utilitários que frequentemente faltam em listas gerais: administração de kernel, tracing, I/O/performance, compressão avançada, backup, containers/VMs avançados, redes (ferramentas low-level), segurança aprofundada, e *muitas combinações práticas* (recipes).  
Salve como `linux_commands_ubuntu_additional.md`.

---

## Sumário rápido
1. Informações do sistema e kernel
2. Módulos, kernel messages e tunables
3. I/O e performance (iostat, vmstat, sar, perf)
4. Compressão, transferência e pipelines eficientes
5. Backup e snapshot (rsync avançado, borg, restic, LVM snapshots)
6. Virtualização/VMs/Containers (qemu, virt-install, kubectl, docker-compose)
7. Rede (iperf, ethtool, tc, socat, ssh advanced)
8. Segurança avançada (gpg, openssl, chattr, auditd, AppArmor/SELinux)
9. Troubleshooting & recovery (dd, chroot, initramfs, grub)
10. Shell scripting avançado e boas práticas
11. Combinações prontas (recipes)
12. Como descobrir ferramentas instaladas e documentação

---

## 1) Informações do sistema e kernel
- `uname -a` — informações do kernel.
- `uname -r` — versão do kernel.
- `lsb_release -a` — versão/distribuição do Ubuntu.
- `hostnamectl` — informações do host e mudança de hostname.
- `uptime` — tempo de atividade e load average.
- `who -b` — última vez de boot.
- `cat /proc/cpuinfo` — detalhes da CPU.
- `cat /proc/meminfo` — memória.
- `free -h` — resumo de memória.
- `dmidecode` — info de hardware (BIOS, memória, fabricante).
- `inxi -Fxz` — resumo completo (instalar `inxi`).

---

## 2) Módulos, kernel messages e tunables
- `lsmod` — módulos carregados.
- `modprobe nome` — carregar módulo.
- `modinfo nome` — informações sobre um módulo.
- `rmmod nome` — remover módulo.
- `dmesg` — ring buffer do kernel (use `dmesg --ctime` para timestamps).
- `dmesg | less`
- `sysctl -a` — listar parâmetros do kernel.
- `sysctl -w net.ipv4.ip_forward=1` — alterar tunable runtime.
- Persistir em `/etc/sysctl.conf` ou `/etc/sysctl.d/99-custom.conf`.
- `echo 1 > /proc/sys/net/ipv4/ip_forward` — outra forma de escrever diretamente.
- `kmod` — ferramentas de módulos.

---

## 3) I/O e performance
Instalar pacote `sysstat` (iostat, sar, mpstat) e `procps` para ferramentas úteis.

- `iostat -xz 1` — estatísticas de I/O por dispositivo (utilização, await).
- `vmstat 1` — estatísticas de memória/CPU/I/O.
- `mpstat -P ALL 1` — estatísticas de CPU por core (vindo do sysstat).
- `sar -u 1 3` — histórico de uso (se sar estiver coletando).
- `iotop` — monitora I/O por processo (`sudo apt install iotop`).
- `perf top` / `perf record` / `perf report` — profiling de CPU/perf events (kernel perf).
- `blktrace` / `btt` — tracing de bloco.
- `fio` — benchmarking I/O (muito usado para testes de discos e storage).
- `numastat` — se sistema com NUMA.
- `powertop` — diagnóstico de consumo de energia.

---

## 4) Compressão, transferência e pipelines eficientes
- compressores:
  - `gzip`, `gunzip`
  - `bzip2`, `bunzip2`
  - `xz` (`xz -T0` para multi-thread)
  - `zstd` (`zstd -T0` multithread, ótima compressão/velocidade)
  - `pigz` — gzip paralelo (`pigz` e `unpigz`)
- `pv` — progress/throughput monitor (útil em pipes).
- Exemplos:
  - `tar -I 'zstd -T0' -cvf archive.tar.zst pasta/` — tar + zstd multithread.
  - `tar -czf - pasta/ | pv | ssh user@host 'cat > /tmp/pasta.tgz'`
  - `rsync -a --info=progress2 --compress-level=3` — mostra progresso mais informativo.
- `scp -C` (compress) e `rsync -z` (compress), `scp -o CompressionLevel=9` (config).
- `mbuffer` — buffer para transferência entre processos (útil p/ redes instáveis).
- `split` / `cat` — dividir e recompor arquivos.

---

## 5) Backup e snapshot
### Rsync avançado
- `rsync -aAXv --delete --link-dest=/backups/daily.2025-10-10 /source/ /backups/daily.2025-10-11/`
  - `--link-dest` para backups incrementais usando hardlinks (economiza espaço).
- `rsync --bwlimit=5000` — limitar banda.

### Borg
- `borg init --encryption=repokey /mnt/backups/borgrepo`
- `borg create --progress /mnt/backups/borgrepo::$(date +%F) /home/user`
- `borg prune --keep-daily=7 --keep-weekly=4 --keep-monthly=6 /mnt/backups/borgrepo`

### Restic
- `restic init -r sftp:user@host:/path`
- `restic -r sftp:user@host:/path backup /home/user`
- `restic forget --keep-daily 7 --keep-weekly 4 --prune -r sftp:...`

### LVM snapshots
- `lvcreate -L1G -s -n snap_home /dev/vg0/home`
- Montar snapshot: `mount /dev/vg0/snap_home /mnt/snap`
- Após backup, remover: `lvremove /dev/vg0/snap_home`

### Btrfs/ZFS
- Com sistemas de arquivos que suportam snapshots (btrfs, zfs), use `btrfs subvolume snapshot` ou `zfs snapshot` para backups rápidos.

---

## 6) Virtualização / Containers (avançado)
### QEMU/KVM
- Criar VM:
  - `virt-install --name vm1 --ram 4096 --vcpus 2 --disk path=/var/lib/libvirt/images/vm1.qcow2,size=20 --os-variant ubuntu22.04 --cdrom /tmp/ubuntu.iso --network network=default`
- `virsh list --all`, `virsh console vm1`, `virsh shutdown vm1`, `virsh start vm1`.
- `qemu-img create -f qcow2 vm.qcow2 20G`
- `qemu-system-x86_64 -m 2048 -drive file=vm.qcow2,format=qcow2 -cdrom ubuntu.iso -boot d`

### Docker / Podman avançado
- `docker-compose up -d --build` — levantar stack com docker-compose.yml.
- `docker run --cap-add=NET_ADMIN --network host --name test --privileged -v /dev:/dev image`
- `docker exec -it container bash` / `nsenter` para entrar namespaces.
- `docker save image | gzip > image.tar.gz` / `docker load`

### Kubernetes
- `kubectl get pods -A`, `kubectl logs -f pod -c container`
- `kubectl port-forward svc/myservice 8080:80`
- `helm repo add`, `helm install release chart/ -f values.yaml`

---

## 7) Rede — ferramentas low-level e QoS
- `iperf3 -s` / `iperf3 -c host` — medir throughput.
- `ethtool eth0` — ver/alterar parâmetros NIC (speed, duplex, offloads).
- `iw dev wlan0 scan` / `iwlist wlan0 scan` — wireless scan.
- `rfkill list` — ver/ativar/desativar radio devices.
- `tc` (traffic control):
  - `tc qdisc add dev eth0 root tbf rate 1mbit burst 10kb latency 70ms` — token bucket filter.
  - `tc qdisc show dev eth0`
- `socat` — tool "Swiss-army" para encaminhar ports e criar proxys:
  - `socat TCP-LISTEN:8080,reuseaddr,fork TCP:backend:80`
- `ssh` avançado:
  - Multiplex: `~/.ssh/config` with `ControlMaster auto`, `ControlPath ~/.ssh/cm-%r@%h:%p`, `ControlPersist 10m`
  - `ssh -L 8080:localhost:80 user@remote` — local port forwarding
  - `ssh -R 2222:localhost:22 user@remote` — remote port forwarding
  - `ssh -N -f -L ...` — background tunnel
  - `ProxyJump/ProxyCommand` for jump hosts.
- `ncat` (`nmap` package) — similar ao netcat, com TLS.

---

## 8) Segurança avançada
- `gpg --full-generate-key` — criar chave GPG.
- `gpg --encrypt --sign -r recipient file`
- `openssl`:
  - `openssl genrsa -out key.pem 4096`
  - `openssl req -new -x509 -days 365 -key key.pem -out cert.pem`
  - `openssl s_client -connect host:443` — testar TLS
- `chattr +i arquivo` — make immutable (não pode ser modificado, mesmo por root sem remover atributo).
- `lsattr` — listar atributos.
- `auditd`:
  - `auditctl -w /etc/shadow -p wa -k shadow_changes`
  - `ausearch -k shadow_changes`
- `apparmor_status`, `aa-enforce`, `aa-complain` — gerenciar AppArmor.
- `setfacl` / `getfacl` — ACLs.
- `getcap` / `setcap` — capabilities de binários (ex.: `setcap 'cap_net_bind_service=+ep' /usr/bin/nginx`).
- `openssl passwd -6 'senha'` — gerar hash SHA512 para /etc/shadow.

---

## 9) Troubleshooting & recovery
- `dd if=/dev/sda of=/tmp/disk.img bs=4M status=progress` — clonar disco (cuidado).
- `ddrescue` — recuperar discos com bad sectors (`gddrescue`).
- `fsck -fy /dev/sdXn` — checar e reparar FS (montagem off).
- Regenerar initramfs: `sudo update-initramfs -u -k all`
- Reinstalar GRUB: `sudo grub-install /dev/sda && sudo update-grub`
- `chroot /mnt/sysimage /bin/bash` — recuperação após montar root FS a partir de live-cd.
- `rescue mode` systemd: `systemctl --force --force reboot --firmware-setup` (para acessar firmware) — cuidado.
- `badblocks -v /dev/sdX` — checar blocos defeituosos (demorado).

---

## 10) Shell scripting avançado & boas práticas
- No topo do script:
  ```bash
  #!/usr/bin/env bash
  set -Eeuo pipefail
  trap 'echo "Erro no script em linha $LINENO"; exit 1' ERR
  ```
- Teste var: `${VAR:-default}` e `${VAR:?mensagem de erro}`.
- Arrays e loops robustos:
  ```bash
  mapfile -t files < <(find . -type f -name '*.log' -print0 | xargs -0 -n1)
  for f in "${files[@]}"; do
    printf 'Processando %s\n' "$f"
  done
  ```
- Lockfile (prevenir múltiplas execuções):
  ```bash
  exec 9>/var/lock/myscript.lock
  flock -n 9 || { echo "Outra instância rodando"; exit 1; }
  ```
- Uso de `mktemp` para arquivos temporários.
- Use `printf` em vez de `echo` para previsibilidade.
- Documente e valide entradas; use `getopts` para flags.

---

## 11) Combinações prontas (recipes) — copie e adapte
### 1) Backup incremental com rsync + hardlinks (exemplo)
```bash
# folders: /backups/daily.YYYY-MM-DD/
today="/backups/daily.$(date +%F)"
yesterday="/backups/daily.$(date -d "yesterday" +%F)"

mkdir -p "$today"
rsync -aHAX --delete --link-dest="$yesterday" /home/ "$today"/
```

### 2) Compactar logs recentes e copiar via SSH com progress
```bash
find /var/log -type f -mtime -7 -print0 | tar --null -T - -czf - | pv -s $(du -sc $(find /var/log -type f -mtime -7) | tail -n1 | awk '{print $1}') | ssh user@backup 'cat > /backups/logs_$(date +%F).tar.gz'
```

### 3) Testar servidor HTTP e salvar resposta com tempo
```bash
curl -w "@curl-format.txt" -o /tmp/body.html -sS http://example.com
# curl-format.txt pode conter: 
# time_namelookup:  %{time_namelookup}\n
# time_connect:  %{time_connect}\n
# time_starttransfer: %{time_starttransfer}\n
# time_total:  %{time_total}\n
```

### 4) Encontrar processos que usam mais de X MB de memória
```bash
ps aux --sort=-rss | awk 'NR<=20{printf "%s %sMB %s\n",$1,$6/1024,$11}'
```

### 5) Acompanhar logs remotos via SSH com reconexão automática
```bash
while true; do ssh -o ServerAliveInterval=30 user@host 'tail -n 200 -f /var/log/syslog'; sleep 5; done
```

### 6) Criar snapshot LVM, montar e usar rsync
```bash
lvcreate -L5G -s -n snap_data /dev/vg/data
mount /dev/vg/snap_data /mnt/snap
rsync -a /mnt/snap/ user@backup:/path
umount /mnt/snap
lvremove /dev/vg/snap_data
```

### 7) Perfil de CPU por processo com perf
```bash
sudo perf record -F 99 -p $(pgrep -n myprocess) -g -- sleep 60
sudo perf report --stdio > perf-report.txt
```

### 8) Redirecionar porta local para pod Kubernetes
```bash
kubectl port-forward svc/my-service 8080:80 -n mynamespace
# Agora acesse localhost:8080
```

### 9) Usar socat para criar proxy TLS simples (terminação TLS local)
```bash
socat TCP-LISTEN:443,reuseaddr,fork OPENSSL:backend.example.com:443,verify=0
```

### 10) Deploy rápido de site estático com docker + Caddy (HTTPS automático)
```bash
docker run -d --name caddy -p 80:80 -p 443:443 -v /site:/usr/share/caddy -v caddy_data:/data -v caddy_config:/config caddy
```

---

## 12) Descobrir o que está instalado e documentação
- `compgen -c | sort` — listar comandos disponíveis no shell.
- `apt list --installed` — pacotes instalados.
- `dpkg -L pacote` — arquivos instalados por um pacote.
- `man -k termo` / `apropos termo` — procurar manpages.
- `whatis comando` — breve descrição.
- `info comando` — documentação info.
- `/usr/share/doc/pacote/` — documentação de pacotes.

---

## Notas finais
- Muitos comandos avançados dependem de pacotes que podem não estar instalados por padrão (`sysstat`, `iotop`, `fio`, `iperf3`, `pv`, `zstd`, `borg`, `restic`, `socat`, `perf`, etc.). Use `sudo apt install` para obtê-los.
- Sempre teste comandos perigosos em ambientes controlados (VM, snapshot, backup).
- Se quiser, posso:
  - juntar este arquivo com o anterior em um só único Markdown completo,  
  - exportar para PDF/DOCX,  
  - gerar uma versão "cheat-sheet" de 1 página (top 50+ combos), ou  
  - personalizar exemplos para seu ambiente (ex.: paths, nomes de serviços).

# Fim
