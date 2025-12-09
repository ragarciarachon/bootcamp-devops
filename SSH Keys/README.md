# Configurar SSH con claves RSA para VM y GitHub en Windows

Este tutorial explica cómo generar claves RSA, subirlas a un servidor (VM) y a GitHub, configurar `ssh-agent` para gestionarlas, y usar un archivo `config` para simplificar las conexiones.


> [!NOTE] 
> Si solo necesitas un **único par de claves SSH** (por ejemplo, solo para GitHub o solo para tu VM), **no es necesario usar el archivo `config` ni `ssh-agent`**.  
>
> Este documento está orientado a un escenario más avanzado donde se crean **dos claves diferentes** (por ejemplo, una para GitHub y otra para una VM) y se necesita gestionarlas correctamente en Windows.

Este tutorial explica cómo generar claves RSA, subirlas a una VM y a GitHub, configurar `ssh-agent` para gestionarlas y usar un archivo `config` para simplificar las conexiones SSH.


# Índice

- [0️⃣ Requisitos previos](#0️⃣-requisitos-previos)
- [1️⃣ Conceptos básicos](#1️⃣-conceptos-básicos)
- [2️⃣ Generar claves RSA](#2️⃣-generar-claves-rsa)
  - [🟦 Generar clave RSA en Git Bash (Linux)](#-generar-clave-rsa-en-git-bash-linux)
  - [🟪 Generar clave RSA en Windows PowerShell](#-generar-clave-rsa-en-windows-powershell)
- [3️⃣ Subir la clave pública al servidor o servicio](#3️⃣-subir-la-clave-pública-al-servidor-o-servicio)
  - [Para la VM](#para-la-vm-2)
  - [Para GitHub](#para-github-2)
- [4️⃣ Configurar ssh-agent](#4️⃣-configurar-ssh-agent)
  - [Iniciar el ssh-agent como servicio de Windows](#iniciar-el-ssh-agent-como-servicio-de-windows)
  - [Añadir las claves al agente](#añadir-las-claves-al-agente)
  - [Verificar las claves cargadas](#verificar-las-claves-cargadas)
- [5️⃣ Crear el archivo de configuración config](#5️⃣-crear-el-archivo-de-configuración-config)
  - [Contenido de ejemplo](#contenido-de-ejemplo)
  - [Explicación](#explicación)
- [6️⃣ Probar las conexiones](#6️⃣-probar-las-conexiones)
  - [Con la VM](#con-la-vm)
  - [Con GitHub](#con-github)
- [7️⃣ Integración con Visual Studio](#7️⃣-integración-con-visual-studio)

<br>

## 0️⃣ Requisitos previos

- Tener instalado OpenSSH
  - **Linux/macOS**: viene instalado por defecto.
  - **Windows 10/11**: instalar OpenSSH Client si fuera necesario o usar **Git Bash**.
- Tener acceso:
  - A tu cuenta de **GitHub**
  - Una **VM** con usuario e IP pública

> [!IMPORTANT]
> En Windows 10 y 11, OpenSSH **Client** viene incluido pero no siempre habilitado.  
> - Solo necesitas **OpenSSH Client** para este tutorial (para conectarte a GitHub o a tu VM).  
> - **OpenSSH Server** no es necesario a menos que quieras que tu Windows reciba conexiones SSH.
>
> Para comprobar si el Client está instalado, abre PowerShell como administrador y ejecuta:
>
> ```powershell
> Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
> ```
>
> Si no está instalado, habilítalo con:
>
> ```powershell
> Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
> ```

## 1️⃣ Conceptos básicos

Antes de comenzar, conviene entender algunos conceptos:

- **Clave RSA**: par de claves criptográficas (privada y pública) usadas para autenticar conexiones SSH sin usar contraseñas.  
  - La **clave privada** debe permanecer secreta en tu máquina.  
  - La **clave pública** se comparte con el servidor o servicio (VM, GitHub).  

- **SSH (Secure Shell)**: protocolo seguro para conectarse a servidores remotos.

- **ssh-agent**: programa que guarda tus claves en memoria para no tener que introducir la passphrase cada vez. Permite **gestionar varias claves** y decidir cuál usar en cada host.

- **Archivo `config` de SSH**: permite definir alias y asociar claves específicas a hosts, simplificando la conexión.

- **Alias de host**: un nombre corto que usamos para referirnos a un host remoto (por ejemplo `mi-vm` en lugar de `usuario@ip`).

## 2️⃣ Generar claves RSA

> [!IMPORTANT]  
> Puedes generar claves desde Git Bash o PowerShell.  
> Elige un método y síguelo completo.

### 🟦 Generar clave RSA en Git Bash (Linux)

Git Bash usa sintaxis tipo Linux.

#### **Para la VM**

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_vm
```

- `-t rsa` → tipo de clave RSA

- `-b 4096` → tamaño de la clave

- `-f ~/.ssh/id_rsa_vm` → ruta de la clave privada

- Se puede añadir una **passphrase** para mayor seguridad

Se generan:

- `~/.ssh/id_rsa_vm` → clave privada

- `~/.ssh/id_rsa_vm.pub` → clave pública

#### **Para GitHub**

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_github
```

<br>

### 🟪 Generar clave RSA en Windows PowerShell

#### Para la VM

```powershell
ssh-keygen -t rsa -b 4096 -f "$env:USERPROFILE\.ssh\id_rsa_vm"
```

#### Para GitHub

```powershell
ssh-keygen -t rsa -b 4096 -f "$env:USERPROFILE\.ssh\id_rsa_github"
```

<br>

## 3️⃣ Subir la clave pública al servidor o servicio

### Para la VM

> [!NOTE]
> `ssh-copy-id` no existe de forma nativa en PowerShell, por lo que se recomienda usar **Git Bash** o copiar manualmente.

#### 1. Asegurar la instalación de SSH dentro de la VM

```bash
# se ejecutan dentro de la VM

sudo apt update
sudo apt install ssh
```

#### 2. Subir clave desde **Git Bash**

```bash
ssh-copy-id -i ~/.ssh/id_rsa_vm.pub usuario@IP_DE_LA_VM
```

<br>

#### Alternativa desde PowerShell (manual)

1. Copiar la clave pública:

    ```powershell
    Get-Content "$env:USERPROFILE\.ssh\id_rsa_vm.pub" | Set-Clipboard
    ```
2. Pegarla en la VM dentro de `~/.ssh/authorized_keys`. Si la carpeta no existe, créala.

> [!TIP]
> Para poder copiar y pegar entre local y VM, habilita el portapapeles compartido (`Configuración > Interfaz de usuario > Dispositivos > Portapapeles compartido`).

### Para GitHub

1. Copiar la clave pública:
   
   **Powershell**

    ```powershell
    cat $env:USERPROFILE\.ssh\id_rsa_github.pub | Set-Clipboard
    ```

    **Git Bash**

    ```bash
    cat ~/.ssh/id_rsa_github.pub
    ```

2. En GitHub, ir a `Settings → SSH and GPG keys → New SSH key`

3. Pegar la clave y guardar.

## 4️⃣ Configurar ssh-agent

> [!IMPORTANT]
> Utilizar Powershell como administrador.

### Iniciar el ssh-agent como servicio de Windows

```powershell
Start-Service ssh-agent
Set-Service -Name ssh-agent -StartupType Automatic
```

### Añadir las claves al agente

```powershell
ssh-add C:\Users\TU_USUARIO\.ssh\id_rsa_vm
ssh-add C:\Users\TU_USUARIO\.ssh\id_rsa_github
```

### Verificar las claves cargadas

```powershell
ssh-add -l
```

Salida esperada:

```powershell
4096 SHA256:xxxx id_rsa_vm (RSA)
4096 SHA256:xxxx id_rsa_github (RSA)
```

Con `ssh-agent` puedes gestionar varias claves y SSH seleccionará la correcta según el host.

## 5️⃣ Crear el archivo de configuración config

> [!NOTE]
> No hacemos alias para GitHub porque rompería las URLs SSH de los repos

📌 Ubicación del archivo: `C:\Users\TU_USUARIO\.ssh\config`

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

### Explicación

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

Salida esperada en GitHub:

```bash
Hi USERNAME! You've successfully authenticated...
```

## 7️⃣ Integración con Visual Studio

- Visual Studio usa el SSH integrado de Windows.

- Mientras las claves estén en el `ssh-agent` y el `config` esté bien configurado:

  - ✓ No pedirá passphrase
  - ✓ Funcionará Git con SSH
  - ✓ No importa si cierras PowerShell o Git Bash