# Guia de Instalación de Qemu/KVM + GUI:

## Actualizar el sistema
`sudo pacman -Syu`

## Verificar si los modulos de virtualización estan cargados
Buscar , según corresponda kvm_amd o kvm_intel
`lsmod | grep kvm`

## Verificar que el procesador tenga la Virtualización activada desde el BIOS 
Utilizaremos `grep+virtua` solamente, porque si el S.O. esta en ingles o español cambia la palabra virtualización/tion y sino no podremos encontrarla con el comando `grep` 
`lscpu | grep virtua`     

## Instalar qemu +KVM
`sudo pacman -S qemu-full virt-manager virt-install virt-viewer libvirt edk2-ovmf dnsmasq ebtables edk2-ovmf swtpm libosinfo`
  - qemu: El software de virtualización.
  - virt-manager: Interfaz gráfica para gestionar máquinas virtuales.
  - virt-install - herramienta de consola para crear VMs.
  - virt-viewer - GUI de consola para conectar VMs
  - libvirt: Biblioteca para interactuar con QEMU/KVM.
  - dnsmasq y ebtables: Para soporte de redes en las VMs.
  - edk2-ovmf: Habilita soporte UEFI en las VMs
  - swtpm: TPM Es un emulador de (Trusted Platform Module) para VMs

## Habilitar el backend de Virt-manager.
`sudo systemctl enable libvirtd`
`sudo systemctl start libvirtd`

## Verificar que dispositivos estan virtualizados
`sudo virt-host-validate qemu`

## Habilitar Nested Virtualization
Permite ejecutar una VM dentro de otra contando con acelearación de Hardware del Host(Por ejemplo: Wine dentro de la VM)

### Habilitarlo para una sesion
AMD:
```
sudo modprobe -r kvm_amd
sudo modprobe kvm_amd nested=1
```
Intel:
```
sudo modprobe -r kvm_intel
sudo modprobe kvm_intel nested=1
```

### Habilitarlo permanentemente
AMD:
```
echo "options kvm_amd nested=1" | sudo tee /etc/modprobe.d/kvm-amd.conf
```
Intel:
```
echo "options kvm_intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf
```

## Habilitar AMD SEV (Opcional Recomendado)
Secure Encryptation Virtualization; Esta técnologia usa claves encriptadas diferentes separando el uso de memoria entre el host y las VM para impedir el acceso a información no autorizada.

Vía modrpobe:
```
echo "options kvm_amd sev=1" | sudo tee /etc/modprobe.d/amd-sev.conf
sudo reboot
```
Vía Grub:
1. Abrir con nano, vi, o vim sudo
`/etc/default/grub`
2. Agregar
`GRUB_CMDLINE_LINUX="... mem_encrypt=on kvm_amd.sev=1"`
3. Regenerar `grub.vfg`
```  
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

## Habilitar Intel IOMMU (Opcional Recomendado)
Intel Virtualization Technology for Direct I/O; Gestiona la memoria de I/O asignando direcciones virtuales visibles a dispositivos con direcciones fisicas. Permite accesos DMA eficientes y seguros.

1. Abrir con nano, vi, o vim sudo
`/etc/default/grub`
2. Agregar
`GRUB_CMDLINE_LINUX="... intel_iommu=on iommu=pt"`
3. Regenerar `grub.vfg`
```  
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

## Optimizar el host con TuneD (Opcional)

1. Remplazar si existe power-profiles-daemon con TuneD
```
sudo pacman -Rs power-profiles-daemon
sudo pacman -S tuned
```
2. Verificar y detener power-profiles-daemon
```
systemctl stop power-profiles-daemon
systemctl status power-profiles-daemon
```
3. Habilitar TuneD
```
sudo systemctl start tuned
sudo systemctl enable tuned
```
4. Setear el profile de optimización del host
`
tuned-adm profile virtual-host`
5.Verificar el profile
`sudo tuned-adm verify`

## Seteamos los permisos ACL(acces control list) de KVM para los directorios de las imagenes

1. Borramos permisos actuales, usaremos la ubicación default de las imagenes
`sudo setfacl -R -b /var/lib/libvirt/images/`
2. Le damos permisos al usuario actual
`
sudo setfacl -R -m "u:${USER}:rwX" /var/lib/libvirt/images/`
3. Establecemos permisos default para nuestro usaurio, para poder utilizar los archivos y carpeteas que se creen posteriormente
`sudo setfacl -m "d:u:${USER}:rwx" /var/lib/libvirt/images/ q hace este comando`
4. Verificamos los permisos
`sudo getfacl /var/lib/libvirt/images/`
```
getfacl: Removing leading '/' from absolute path names
# file : var/lib/libvirt/images/
# owner: root
# group: root
user::rwx
user:tatum:rwx
group::--x
mask::rwx
other::--x
default:user::rwx
default:user:MyUser:rwx
default:group::--x
default:mask::rwx
default:other::--x
```

## Ejecutamos el Frontend de qemu. 
`virt-manager`

## Notas: 

- Para activar y desactivar el daemon y servicios asociados, utilizar:
 `https://github.com/e-conte/scripts/blob/main/toggle-virt.sh`
- Para bindear maquinas en I3wm:
 `bindsym "key" exec  virt-manager --connect qemu:///system --show-domain-console  
  "NombreDeLaVM"`

## Info adicional:

- KVM, Kernel based virtualization machine, es decir.
- Qemu es un hypervisor de KVM ó aplicación que permite virtualizar.
- virt-manager es una GUI para KVM, el frontend.
- libvirt es una libreria escrita en C que proporciona una API quee permite usar 
diferentes hypervisores. 
- libvirt también es el daemon de backend de la misma y tiene la herramientas CLI 
virsh.


