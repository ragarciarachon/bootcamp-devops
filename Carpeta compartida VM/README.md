# 📁 Carpeta compartida VirtualBox-Windows

Guía completa para configurar una carpeta compartida entre **Windows (*host*)** y una **VM Ubuntu** usando VirtualBox. Perfecta para trabajar con los mismos archivos en ambos sistemas de manera fluida.

## 📚 Índice <!-- omit from toc -->

1. [Requisitos previos](#requisitos-previos)
2. [Crear carpeta compartida (desde el *host*)](#crear-carpeta-compartida-desde-el-host)
3. [Configurar permisos](#configurar-permisos)
4. [Comprobación](#comprobación)
5. [Problemas comunes](#problemas-comunes)


---

## Requisitos previos

Antes de comenzar, asegúrate de tener:

- VirtualBox instalado en el *host* Windows
- Una VM Ubuntu creada y funcionando
- *Guest Additions* instaladas en la VM para permitir carpetas compartidas

<br>

> [!CAUTION]
> Sin *Guest Additions*, la carpeta compartida no funcionará correctamente.


#### Instalar *Guest Additions*

1. Inicia la VM
2. En el menú de VirtualBox: ``Dispositivos → Insertar imagen de CD de las Guest Additions``
3. Ejecuta el instalador dentro de la VM
4. Reinicia la VM para aplicar los cambios

---

## Crear carpeta compartida (desde el *host*)

1. Apaga la VM
2. En VirtualBox, selecciona la ``VM → Configuración → Carpetas compartidas``
3. Haz clic en ``Añadir nueva carpeta``

#### Opciones recomendadas

- **Ruta de la carpeta**: ruta de la carpeta del *host* Windows (ej. ``C:\Users\Usuario\NombreCarpeta``)
- **Nombre de la carpeta**: se rellena automáticamente cuando añades la ruta
- **Mount point (opcional)**: ruta dentro de la VM (ej. ``/home/usuario/NombreCarpeta``)

    > [!TIP]
    > Si vas a usar una ruta personalizada, la carpeta debe existir previamente en la VM, de lo contrario el montaje puede fallar.
- **Opciones**:
  - ✅ Automontar: monta la carpeta automáticamente cada vez que inicia la VM.
  - ✅ Make Global: hace que la carpeta compartida esté disponible para todas las VMs de VirtualBox.

<br>

> [!NOTE]
> 💡 Si dejas **Mount point** vacío, VirtualBox montará automáticamente la carpeta en ``/media/sf_compartida``.

---

## Configurar permisos

Cuando se crea un *Mount point* personalizado, es necesario ajustar permisos para que no se pida contraseña o se deniegue el acceso cada vez que accedes a la carpeta.

#### Añadir el usuario al grupo ``vboxsf``

```bash
# Añadir el usuario al grupo
sudo usermod -aG vboxsf $USER

# Reiniciar la VM
sudo reboot
```

## Comprobación

```bash
# Entrar en la carpeta compartida
cd /ruta/de/la/carpeta

# Crear un archivo de prueba
touch prueba.txt
```

Si no pide contraseña y el archivo aparece también en Windows, 🎉 todo correcto.

## Problemas comunes

- ❌ No aparece la carpeta
  - ✔️ *Guest Additions* no instaladas
- ❌ Pide contraseña al acceder o modificar archivos
  - ✔️ Usuario no pertenece al grupo ``vboxsf``
  - ✔️ Permisos del *mount point* incorrectos
- ❌ Archivos no se sincronizan
  - ✔️ Verifica que la opción Automontar esté activada

---