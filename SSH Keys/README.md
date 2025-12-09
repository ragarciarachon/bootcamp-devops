# Configurar SSH con claves RSA para VM y GitHub en Windows

Este tutorial explica cómo generar claves RSA, subirlas a un servidor (VM) y a GitHub, configurar `ssh-agent` para gestionarlas, y usar un archivo `config` para simplificar las conexiones.

# Índice

- [1️⃣ Conceptos básicos](#1️⃣-conceptos-básicos)
- [2️⃣ Crear claves RSA](#2️⃣-crear-claves-rsa)
  - [Para la VM](#para-la-vm)
  - [Para GitHub](#para-github)
- [3️⃣ Subir la clave pública al servidor o servicio](#3️⃣-subir-la-clave-pública-al-servidor-o-servicio)
  - [Para la VM](#para-la-vm-1)
    - [En la VM, tenemos que instalar ssh:](#en-la-vm-tenemos-que-instalar-ssh)
    - [En Windows:](#en-windows)
  - [Para GitHub](#para-github-1)
- [4️⃣ Configurar ssh-agent](#4️⃣-configurar-ssh-agent)
  - [Iniciar el ssh-agent como servicio de Windows](#iniciar-el-ssh-agent-como-servicio-de-windows)
  - [Añadir las claves al agente](#añadir-las-claves-al-agente)
  - [Verificar las claves cargadas](#verificar-las-claves-cargadas)
- [5️⃣ Crear el archivo de configuración config](#5️⃣-crear-el-archivo-de-configuración-config)
  - [Contenido de ejemplo](#contenido-de-ejemplo)
  - [Explicación:](#explicación)
- [6️⃣ Probar las conexiones](#6️⃣-probar-las-conexiones)
  - [Con la VM](#con-la-vm)
  - [Con GitHub](#con-github)
- [7️⃣ Integración con Visual Studio](#7️⃣-integración-con-visual-studio)


<br>

## 1️⃣ Conceptos básicos

Antes de comenzar, conviene entender algunos conceptos:

- **Clave RSA**: par de claves criptográficas (privada y pública) usadas para autenticar conexiones SSH sin usar contraseñas.  
  - La **clave privada** debe permanecer secreta en tu máquina.  
  - La **clave pública** se comparte con el servidor o servicio (VM, GitHub).  

- **SSH (Secure Shell)**: protocolo seguro para conectarse a servidores remotos.

- **ssh-agent**: programa que guarda tus claves en memoria para no tener que introducir la passphrase cada vez. Permite **gestionar varias claves** y decidir cuál usar en cada host.

- **Archivo `config` de SSH**: permite definir alias y asociar claves específicas a hosts, simplificando la conexión.

- **Alias de host**: un nombre corto que usamos para referirnos a un host remoto (por ejemplo `mi-vm` en lugar de `usuario@ip`).



## 2️⃣ Crear claves RSA

Abrimos Git Bash y generamos las claves.

### Para la VM

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_vm
```

- -t rsa → tipo de clave RSA

- -b 4096 → tamaño de la clave

- -f ~/.ssh/id_rsa_vm → ruta de la clave privada

- Se puede añadir una passphrase para mayor seguridad

Se generan:

- ~/.ssh/id_rsa_vm → clave privada

- ~/.ssh/id_rsa_vm.pub → clave pública

### Para GitHub

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_github
```

Tener claves separadas para cada host es más seguro y permite gestionarlas de forma independiente.

## 3️⃣ Subir la clave pública al servidor o servicio

### Para la VM

> [!NOTE]
> Utilizaremos **Git Bash** para poder usar el comando de copiado de clave a máquina, ya que este comando no existe en Windows.

#### En la VM, tenemos que instalar ssh:

```bash
sudo apt update
sudo apt install ssh
```

#### En Windows:

Si ssh-copy-id está disponible:

```bash
ssh-copy-id -i ~/.ssh/id_rsa_vm.pub usuario@IP_DE_LA_VM
```

Si no, copiar manualmente el contenido de `id_rsa_vm.pub` al archivo `~/.ssh/authorized_keys` en la VM.

### Para GitHub

1. Copiar la clave pública:

    ```bash
    # en Powershell
    cat $env:USERPROFILE\.ssh\id_rsa_github.pub | Set-Clipboard

    # en Git Bash
    cat ~/.ssh/id_rsa_github.pub
    ```

2. En GitHub, ir a Settings → SSH and GPG keys → New SSH key

3. Pegar la clave y guardar.

## 4️⃣ Configurar ssh-agent

> [!IMPORTANT]
> Utilizar Powershell como administrador.

### Iniciar el ssh-agent como servicio de Windows

```bash
Start-Service ssh-agent
Set-Service -Name ssh-agent -StartupType Automatic
```

### Añadir las claves al agente

```bash
ssh-add C:\Users\TU_USUARIO\.ssh\id_rsa_vm
ssh-add C:\Users\TU_USUARIO\.ssh\id_rsa_github
```

### Verificar las claves cargadas

```bash
ssh-add -l
```

Salida de ejemplo:

```bash
4096 SHA256:xxxx id_rsa_vm (RSA)
4096 SHA256:xxxx id_rsa_github (RSA)
```

Con `ssh-agent` puedes gestionar varias claves y SSH seleccionará la correcta según el host.

## 5️⃣ Crear el archivo de configuración config

> [!NOTE]
> Para GitHub no usaremos alias, ya que habría que modificar el comando de clonación por SSH cuando queramos clonar un repositorio. Nada práctico.

Ubicación: `C:\Users\TU_USUARIO\.ssh\config`

### Contenido de ejemplo

```text
# VM
Host vm-example
    HostName IP_DE_LA_VM
    User usuario
    IdentityFile ~/.ssh/id_rsa_vm
    IdentitiesOnly yes

# GitHub con URL completa
Host github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github
    IdentitiesOnly yes
```

### Explicación:

- `Host vm-example` → alias que usarás en SSH (`ssh vm-example`)

- `HostName` → dirección real del host

- `User` → usuario con el que te conectas

- `IdentityFile` → ruta de la clave privada a usar

- `IdentitiesOnly yes` → fuerza a SSH a usar solo la clave indicada
- 

## 6️⃣ Probar las conexiones

### Con la VM

```bash
ssh vm-example
```

### Con GitHub

```bash
ssh -T git@github.com
```

Ambos deberían responder algo como:

```bash
Hi USERNAME! You've successfully authenticated...
```

## 7️⃣ Integración con Visual Studio

- Visual Studio usa `ssh-agent` y `ssh.exe` de Windows.

- Mientras las claves estén cargadas en el agente y el `config` esté correcto, Visual Studio puede conectarse **sin pedir passphrase**.

- Esto funciona incluso si cierras Git Bash o PowerShell, porque el agente corre como servicio.

<br>

