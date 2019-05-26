## Entrega SO2019
### Integrantes:
- Aparicio, Natalia Elizabeth 12667/7
- Eusebi, Cirano 12469/2

---

### Ambiente utilizado
Para resolver la práctica, nos pusimos como objetivo evitar la instalación de cualquier herramienta.

Para esto hizo falta entonces virtualizar. Elegimos Vagrant como manejador de virtualización, ya que nos permite versionar nuestras configuraciones y aprovisionar con lo necesario la VM.

Al querer probar las imágenes generadas dentro de la VM nos encontramos con la problematica de no tener la virtualización anidada habilitada. Decidimos migrar a un esquema con Hyper-V.

---

### Problemas y soluciones encontradas:
- Faltaban dependencias en la imágen de buildroot
    - Agregar las dependencias faltantes
- Errores en la configuracion de buildroot
    - Leer el manual de buildroot
- No tener habilitada la virtualización
    - Buscar una imagen Hyper-V con virtualización anidada (windows)
    - Correr las imágenes desde la máquina host con qemu-kvm (gnu/linux)
- Kernel panic al montar el initramfs modificado (missing magic initramfs)
    - Agregar el header -H newc al comando cpio
- Kernel panic switch root siempre imprimia el help
    - Usar el comando `exec` para switch_root para que mantenga el PID 1
- Error "cant open /dev/console"
    - Crear y mover `/bin/mount --move /dev /mnt/root/dev` antes del `switch_root`

---
## Tareas realizadas

### Para la compilación:
1. Leer docs buildroot
2. Instalar buildroot en Vagrant VM (provista por buildroot).
3. Configurar compilación segun especificacion de la catedra
    - Target x86_64
    - Config custom /board/qemu/x86/linux.config
    - Init system: BusyBox
    - Version de Kernel 4.19
        - Default config
    - cpio root file system compress xz
    - ext4 root filesystem
    - Bootloader syslinux en MBR
4. Compilar imagenes
5. Agregar las dependencias que iban faltando
    - libssl-dev
    - libelf-dev

### Para generar las imágenes:
1. Usar imagen de centOS (Hyper-V) con virtualizacion anidada
    - Correr el script enable-nested-v.ps1
2. Instalar dependencias que faltan:
    - qemu-kvm 
    - libvirt 
    - libvirt-python 
    - libguestfs-tools
    - virt-install
    - qemu
3. Probar imágenes generadas:
    - `kvm -m 512 -kernel bzImage -initrd rootfs.cpio.xz --append "console=ttyS0" -nographic`
    - `kvm -m 512 -kernel bzImage -append "root=/dev/sda console=ttyS0" rootfs.ext2 -nographic`
4. Modificar cpio:
    - Descomprimir y expandir el .cpio (cpio -idv < ../rootfs.cpio)
    - Modificar script /init
    - Reempaquetar cpio (ls | cpio -ov -H newc > ../rootfs.mod.cpio)
    - Comprimir con xz (xz -k --check=crc32 rootfs.mod.cpio)
5. Modificar ext4:
    - Montar el filesystem en una carpeta (dentro de /mnt)
    - Agregar script que imprime nombres a /etc/init.d
        - No olvidar dar permisos de ejecución.
    - Desmontar filesystem.
6. Probar imágenes modificadas:
    - `kvm -m 512 -kernel bzImage -initrd rootfs.mod.cpio.xz -append "root=/dev/sda console=ttyS0" rootfs.ext2 -nographic`