# Copy Fail Lab — CVE-2026-31431 (v2)

Devcontainer reproducible para experimentar con la vulnerabilidad **Copy Fail**
(CVE-2026-31431) en un kernel Linux 6.12 controlado dentro de QEMU.

Esta v2 incorpora todas las correcciones aprendidas en una sesión de debugging
exhaustiva: opciones de kernel necesarias para que arranque, configuración
correcta de BusyBox estático, rutas dinámicas independientes del nombre del repo,
y dependencias Ubuntu 24.04 corregidas.

---

## Inicio rápido para el estudiante

1. Abre un Codespace desde este repo.
   ```bash
   #CONFIGURACION DE EJEMPLO!!!!!!!!!!!
   apt update
   apt install gh
   
   gh api user --jq '"\(.name) → \(.email // .login)"'
   
   git config --global user.name "Jonathan E. Tito O."
   git config --global user.email "jonathantito@users.noreply.github.com"
   git config --global --add safe.directory /workspaces/copy-fail-challenge-1
   make setup
   ```
3. Configura tu identidad git:
   ```bash
   git config --global user.name "Tu Nombre"
   git config --global user.email "tu@correo.com"
   ```
4. Ejecuta:
   ```bash
   make setup    # descarga kernel + arma rootfs (~5 min)
   make qemu     # arranca la VM vulnerable
   ```

Para salir de QEMU: `Ctrl+A` luego `X`.

---

## Configuración inicial del docente (una sola vez)

### 1. Subir este repo a GitHub

```bash
cd copyfail-v2
git init && git add -A && git commit -m "initial"
git branch -M main
gh repo create TU-ORG/copy-fail-lab --public --source=. --push
```

### 2. Marcarlo como Template

GitHub → tu repo → Settings → marcar `Template repository`.

### 3. Editar `.devcontainer/devcontainer.json`

Cambia el valor `KERNEL_REPO`:
```json
"KERNEL_REPO": "TU-ORG/copy-fail-lab"
```

Commit y push.

### 4. Disparar el workflow del kernel

GitHub → Actions → `Build Vulnerable Kernel` → Run workflow.
Tarda ~25 min en los servidores de GitHub (no en tu Codespace).
Al terminar crea un Release con el `bzImage_vuln` listo para descarga.

### 5. Verificar

Tu repo → Releases → debe aparecer `kernel-v6.12-vuln` con tres archivos
adjuntos. Los estudiantes ahora pueden hacer `make setup` y descarga en 2 min.

---

## Estructura del repo

```
.
├── .devcontainer/
│   ├── Dockerfile             ← Ubuntu 24.04 + deps verificadas
│   └── devcontainer.json      ← sin rutas hardcodeadas
├── .github/workflows/
│   └── build-kernel.yml       ← compila kernel y crea Release
├── scripts/
│   ├── 00_welcome.sh
│   ├── 01_fetch_kernel.sh     ← descarga del Release
│   ├── 02_build_kernel.sh     ← fallback: compila desde fuente
│   ├── 03_build_rootfs.sh     ← BusyBox estático + initramfs
│   └── 04_run_qemu.sh
├── Makefile
└── README.md
```

---

## Comandos disponibles

| Comando | Acción |
|---|---|
| `make setup` | Descarga kernel + arma rootfs (~5 min) |
| `make qemu` | Arranca la VM vulnerable |
| `make info` | Muestra el estado del ambiente |
| `make rootfs` | Reconstruye solo el initramfs |
| `make fetch-kernel` | Solo descarga el bzImage del Release |
| `make build-kernel` | Compila kernel desde fuente (~25 min) |
| `make clean` | Borra builds (mantiene fuentes) |
| `make clean-all` | Borra todo |

---

## Recursos del CVE

- Write-up técnico: https://xint.io/blog/copy-fail-linux-distributions
- Sitio del CVE: https://copy.fail
- PoC oficial: https://github.com/theori-io/copy-fail-CVE-2026-31431

---

## Lecciones aprendidas (referencia para futuras versiones)

Esta v2 incorpora los siguientes fixes respecto a la v1:

- `hexdump` → `bsdextrautils` en Ubuntu 24.04
- `bzip2` agregado al Dockerfile (lo necesita BusyBox)
- Eliminado el `mounts` con ruta hardcodeada en `devcontainer.json`
- Todos los scripts detectan workspace con `SCRIPT_DIR` dinámico
- Kernel: agregadas opciones críticas `BINFMT_ELF`, `BINFMT_SCRIPT`, `RD_GZIP`
- Kernel: agregada dep `CRYPTO_AEAD` antes de `CRYPTO_AUTHENCESN`
- BusyBox: reemplazado `scripts/config` (no existe) por `sed`
- BusyBox: eliminado `olddefconfig` (no existe en BusyBox)
- BusyBox: deshabilitado `CONFIG_TC` (rompe compilación con kernels nuevos)
- BusyBox: forzado `CONFIG_STATIC=y` y verificado con `file`
- Workflow Actions: greps de verificación con `|| echo`, tolerantes

# ¿Qué kernel corre?
uname -r
6.12.0

# ¿El módulo vulnerable está cargado?
lsmod | grep alg

# ¿Cuál es tu identidad actual? (debe ser student, NO root)
id
whoami
~ $ id
uid=1001(student) gid=1001(student) groups=1001(student)
~ $ whoami
student
~ $ lsmod 
# ¿AF_ALG está disponible?
cat /proc/modules | grep algif

//History
    1  apt update
    2  apt upgrade
    3  gh api user --jq '"\(.name) → \(.email // .login)"'
    4  apt install gh"
    5  apt install gh
    6  apt update
    7  apt upgrade
    8  gh api user --jq '"\(.name) → \(.email // .login)"'
    9  git config --global user.name "Snto666"
   10  git config --global user.email "edyambayfa@uide.edu.ec"
   11  git config --global --add safe.directory /workspaces/copy-fail-challenge-1
   12  make setup
   13  y
   14  make qemu
   15  gh api user --jq '"\(.name) → \(.email // .login)"'
   16  apt-get install -y file musl-tools musl-dev
   17  make qemu
   18  apt update
   19  apt-get install -y file musl-tools musl-dev
   20  make rootfs
   21  make qemu
   22  mount -t proc proc /proc
   23  lsmod | grep alg
   24  cat /proc/modules | grep algif
   25  cp /tmp/hito1.txt evidence/hito1_vuln_confirmed.txt
   26  git add evidence/hito1_vuln_confirmed.txt
   27  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
   28  git tag -a hito-1 -m "Kernel vulnerable corriendo, algif_aead confirmado"
   29  git push origin main --tags
   30  {   echo "=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===";   echo "Fecha: $(date)";   echo "Hostname: $(hostname)";   echo "Kernel: 6.12.0";   echo "Identidad: uid=1001(st
   31  git add evidence/hito1_vuln_confirmed.txt
   32  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
   33  git push origin main
   34  mkdir -p evidence
   35  {   echo "=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===";   echo "Fecha: $(date)";   echo "Hostname: $(hostname)";   echo "Kernel: 6.12.0";   echo "Identidad: uid=1001(st
   36  git add evidence/hito1_vuln_confirmed.txt
   37  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
   38  git push origin main
   39  {   echo "=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===";   echo "Fecha: $(date)";   echo "Hostname: $(hostname)";   echo "Kernel: 6.12.0";   echo "Identidad: uid=1001(st
   40  git add evidence/hito1_vuln_confirmed.txt
   41  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
   42  git push origin main
   43  cp /tmp/hito1.txt evidence/hito1_vuln_confirmed.txt
   44  1.5 Commit del hito 1
   45  git add evidence/hito1_vuln_confirmed.txt
   46  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
   47  git tag -a hito-1 -m "Kernel vulnerable corriendo, algif_aead confirmado"
   48  git push origin main --tags
   49  make verify
   50  git log --oneline
   51  git tag
   52  git tag -d hito-1
   53  git push origin :refs/tags/hito-1
   54  git tag -a hito-1 -m "Kernel vulnerable corriendo, algif_aead confirmado"
   55  git push origin main --tags
   56  git log --oneline
   57  git tag
   58  make qemu
   59  curl -s https://copy.fail/exp
   60  curl -s https://copy.fail/exp > rootfs_overlay/home/student/copy_fail_exp.py 2>/dev/null || curl -s https://copy.fail/exp > /tmp/copy_fail_exp.py
   61  cat /tmp/copy_fail_exp.py
   62  curl -s https://copy.fail/exp > rootfs_overlay/home/student/copy_fail_exp.py 2>/dev/null || curl -s https://copy.fail/exp > /tmp/copy_fail_exp.py
   63  cat /tmp/copy_fail_exp.py
   64  ls /workspaces/copy-fail-challenge-B/
   65  ls kernel/
   66  +
   67  cp /tmp/copy_fail_exp.py kernel/initramfs/
   68  cd kernel/initramfs
   69  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
   70  cd /workspaces/copy-fail-challenge-B
   71  make qemu
   72  ~ $ python3 /copy_fail_exp.py
   73  -sh: python3: not found
   74  ~ $ 
   75  which python3
   76  cp /usr/bin/python3 kernel/initramfs/bin/
   77  ldd /usr/bin/python3
   78  mkdir -p kernel/initramfs/lib/x86_64-linux-gnu kernel/initramfs/lib64
   79  cp /lib/x86_64-linux-gnu/libm.so.6 kernel/initramfs/lib/x86_64-linux-gnu/
   80  cp /lib/x86_64-linux-gnu/libz.so.1 kernel/initramfs/lib/x86_64-linux-gnu/
   81  cp /lib/x86_64-linux-gnu/libexpat.so.1 kernel/initramfs/lib/x86_64-linux-gnu/
   82  cp /lib/x86_64-linux-gnu/libc.so.6 kernel/initramfs/lib/x86_64-linux-gnu/
   83  cp /lib64/ld-linux-x86-64.so.2 kernel/initramfs/lib64/
   84  cd kernel/initramfs
   85  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
   86  cd /workspaces/copy-fail-challenge-B
   87  make qemu
   88  mkdir -p kernel/initramfs/usr/lib
   89  cp -r /usr/lib/python3.12 kernel/initramfs/usr/lib/
   90  cd kernel/initramfs
   91  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
   92  cd /workspaces/copy-fail-challenge-B
   93  make qemu
   94  mkdir -p kernel/initramfs/usr/lib
   95  cp -r /usr/lib/python3.12 kernel/initramfs/usr/lib/
   96  cd kernel/initramfs
   97  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
   98  cd /workspaces/copy-fail-challenge-B
   99  make qemu
  100  ls kernel/initramfs/usr/bin/
  101  which su
  102  cp $(which su) kernel/initramfs/usr/bin/su
  103  ldd $(which su)
  104  cp /lib/x86_64-linux-gnu/libpam.so.0 kernel/initramfs/lib/x86_64-linux-gnu/
  105  cp /lib/x86_64-linux-gnu/libpam_misc.so.0 kernel/initramfs/lib/x86_64-linux-gnu/
  106  cp /lib/x86_64-linux-gnu/libaudit.so.1 kernel/initramfs/lib/x86_64-linux-gnu/
  107  cp /lib/x86_64-linux-gnu/libcap-ng.so.0 kernel/initramfs/lib/x86_64-linux-gnu/
  108  cd kernel/initramfs
  109  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  110  cd /workspaces/copy-fail-challenge-B
  111  make qemu
  112  chmod u+s kernel/initramfs/usr/bin/su
  113  cd kernel/initramfs
  114  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  115  cd /workspaces/copy-fail-challenge-B
  116  make qemu
  117  chmod u+s kernel/initramfs/usr/bin/su
  118  cd kernel/initramfs
  119  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  120  cd /workspaces/copy-fail-challenge-B
  121  make qemu
  122  chmod u+s kernel/initramfs/usr/bin/su
  123  ls -la kernel/initramfs/usr/bin/su
  124  cd kernel/initramfs
  125  find . | cpio -o -H newc --owner root:root | gzip > ../build/initramfs.cpio.gz
  126  cd /workspaces/copy-fail-challenge-B
  127  make qemu
  128  ls -la kernel/initramfs/usr/bin/su
  129  stat kernel/initramfs/usr/bin/su | grep Access
  130  cd kernel/initramfs
  131  sudo find . | sudo cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  132  cd /workspaces/copy-fail-challenge-B
  133  make qemu
  134  cd kernel/initramfs
  135  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  136  cd /workspaces/copy-fail-challenge-B
  137  file kernel/build/initramfs.cpio.gz
  138  cat kernel/initramfs/init
  139  ls kernel/initramfs/
  140  ls kernel/initramfs/etc/ 2>/dev/null
  141  sed -i 's|hostname "copy-fail-Snto666"|chmod u+s /usr/bin/su\nhostname "copy-fail-Snto666"|' kernel/initramfs/init
  142  cat kernel/initramfs/init | grep chmod
  143  cd kernel/initramfs
  144  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  145  cd /workspaces/copy-fail-challenge-B
  146  make qemu
  147  wget https://copy.fail/exp -O copy_fail_exp.py
  148  make qemu
  149  cat kernel/initramfs/init | head -20
  150  ls kernel/initramfs/bin/su 2>/dev/null || ls kernel/initramfs/usr/bin/su
  151  file kernel/initramfs/bin/busybox
  152  chmod u+s kernel/initramfs/bin/su
  153  ls -la kernel/initramfs/bin/su
  154  sed -i 's|chmod u+s /usr/bin/su|chmod u+s /usr/bin/su\nchmod u+s /bin/su|' kernel/initramfs/init
  155  cd kernel/initramfs
  156  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  157  cd /workspaces/copy-fail-challenge-B
  158  make qemu
  159  sed -i 's|/usr/bin/su|/bin/su|g' /tmp/copy_fail_exp.py
  160  cp /tmp/copy_fail_exp.py kernel/initramfs/copy_fail_exp.py
  161  cd kernel/initramfs
  162  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  163  cd /workspaces/copy-fail-challenge-B
  164  make qemu
  165  cat /tmp/copy_fail_exp.py | head -5
  166  grep "su" /tmp/copy_fail_exp.py
  167  grep "su" kernel/initramfs/copy_fail_exp.py
  168  ls -la kernel/initramfs/bin/su
  169  chmod u+s kernel/initramfs/bin/busybox
  170  # Actualiza el init para aplicar setuid a busybox también
  171  sed -i 's|chmod u+s /bin/su|chmod u+s /bin/busybox|' kernel/initramfs/init
  172  cd kernel/initramfs
  173  find . | cpio -o -H newc | gzip > ../build/initramfs.cpio.gz
  174  cd /workspaces/copy-fail-challenge-B
  175  make qemu
  176  cat kernel/initramfs/init
  177  ls kernel/build/
  178  find kernel/linux -name "algif_aead*" 2>/dev/null | head -5
  179  grep -i algif kernel/linux/.config | head -10
  180  grep -i "AF_ALG\|CRYPTO_USER_API\|algif" kernel/linux/.config
  181  make qemu
  182  grep -i authencesn kernel/linux/.config
  183  history
      1  nproc
    2  grep -i "CRYPTO_AUTHENC" kernel/linux/.config
    3  grep -i "AUTHENCESN" kernel/linux/.config
    4  echo "CONFIG_CRYPTO_AUTHENCESN=y" >> kernel/linux/.config
    5  cd kernel/linux
    6  make -j4 crypto/authencesn.ko 2>&1 | tail -5
    7  cd /workspaces/copy-fail-challenge-B/kernel/linux
    8  scripts/config --enable CONFIG_CRYPTO_AUTHENCESN
    9  scripts/config --enable CONFIG_CRYPTO_SEQIV
   10  scripts/config --enable CONFIG_CRYPTO_CBC
   11  make olddefconfig
   12  make -j4 bzImage 2>&1 | tail -10
   13  cd /workspaces/copy-fail-challenge-B
   14  cp kernel/linux/arch/x86/boot/bzImage kernel/build/bzImage_vuln
   15  cd /workspaces/copy-fail-challenge-B
   16  cp kernel/linux/arch/x86/boot/bzImage kernel/build/bzImage_vuln
   17  make qemu
   18  ls kernel/linux/crypto/authencesn* 2>/dev/null
   19  grep "CONFIG_CRYPTO_AUTHENCESN" kernel/linux/.config
   20  Sfind kernel/linux -name "authencesn.ko" 2>/dev/null
   21  find kernel/linux -name "authencesn.ko" 2>/dev/null
   22  cd kernel/linux
   23  make -j4 M=crypto modules 2>&1 | tail -15
   24  cd kernel/linux
   25  make -j4 M=crypto modules 2>&1 | tail -15
   26  cd /workspaces/copy-fail-challenge-B/kernel/linux
   27  scripts/config --enable CONFIG_MODULES
   28  scripts/config --enable CONFIG_CRYPTO_AUTHENCESN
   29  make olddefconfig
   30  make -j4 bzImage 2>&1 | tail -5
   31  cat /workspaces/copy-fail-challenge-B/scripts/02_build_kernel.sh
   32  cd /workspaces/copy-fail-challenge-B/kernel/linux
   33  make tinyconfig
   34  ./scripts/config --enable 64BIT
   35  ./scripts/config --enable SERIAL_8250
   36  ./scripts/config --enable SERIAL_8250_CONSOLE
   37  ./scripts/config --enable TTY
   38  ./scripts/config --enable PRINTK
   39  ./scripts/config --enable BINFMT_ELF
   40  ./scripts/config --enable BINFMT_SCRIPT
   41  ./scripts/config --enable BLK_DEV_INITRD
   42  ./scripts/config --enable RD_GZIP
   43  ./scripts/config --enable TMPFS
   44  ./scripts/config --enable PROC_FS
   45  ./scripts/config --enable SYSFS
   46  ./scripts/config --enable DEVTMPFS
   47  ./scripts/config --enable DEVTMPFS_MOUNT
   48  ./scripts/config --enable NET
   49  ./scripts/config --enable UNIX
   50  ./scripts/config --enable INET
   51  ./scripts/config --enable CRYPTO
   52  ./scripts/config --enable CRYPTO_AEAD
   53  ./scripts/config --enable CRYPTO_AUTHENC
   54  ./scripts/config --enable CRYPTO_USER_API
   55  ./scripts/config --enable CRYPTO_USER_API_AEAD
   56  ./scripts/config --enable CRYPTO_USER_API_SKCIPHER
   57  ./scripts/config --enable CRYPTO_AUTHENCESN
   58  ./scripts/config --enable CRYPTO_AES
   59  ./scripts/config --enable CRYPTO_CBC
   60  ./scripts/config --enable CRYPTO_HMAC
   61  ./scripts/config --enable CRYPTO_SHA256
   62  ./scripts/config --enable MULTIUSER
   63  make olddefconfig
   64  make -j4 bzImage 2>&1 | tail -5
   65  cd /workspaces/copy-fail-challenge-B
   66  cp kernel/linux/arch/x86/boot/bzImage kernel/build/bzImage_vuln
   67  make qemu
   68  grep "CRYPTO_AUTHENCESN" /workspaces/copy-fail-challenge-B/kernel/linux/.config
   69  cd /workspaces/copy-fail-challenge-B/kernel/linux
   70  echo "CONFIG_CRYPTO_AUTHENCESN=y" >> .config
   71  echo "CONFIG_CRYPTO_AEAD2=y" >> .config
   72  make olddefconfig
   73  grep "CRYPTO_AUTHENCESN" .config
   74  grep "CRYPTO_AUTHENC\b" .config
   75  grep "CRYPTO_AEAD\b" .config
   76  echo "CONFIG_CRYPTO_AUTHENCESN=y" >> .config
   77  make -j4 bzImage 2>&1 | tail -5
   78  grep "CRYPTO_AUTHENCESN" .config
   79  scripts/config --enable CRYPTO_AUTHENCESN
   80  grep "CRYPTO_AUTHENCESN" .config
   81  make -j4 bzImage 2>&1 | tail -3
   82  cd /workspaces/copy-fail-challenge-B
   83  cp kernel/linux/arch/x86/boot/bzImage kernel/build/bzImage_vuln
   84  make qemu
   85  cd /workspaces/copy-fail-challenge-B/kernel/linux
   86  touch crypto/authencesn.c
   87  make -j4 bzImage 2>&1 | grep -E "authencesn|bzImage|Error"
   88  cd /workspaces/copy-fail-challenge-B
   89  cp kernel/linux/arch/x86/boot/bzImage kernel/build/bzImage_vuln
   90  make qemu
   91  cd /workspaces/copy-fail-challenge-B
   92  cp kernel/linux/arch/x86/boot/bzImage kernel/build/bzImage_vuln
   93  make qemu
   94  history