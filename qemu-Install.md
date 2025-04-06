# Guia de Instalación de Qemu/KVM + GUI:

## Actualizar el sistema
`sudo pacman -Syu`

## Chequear si los modulos de virtualización estan cargados
`lsmod | grep kvm`

## Buscar , según corresponda kvm_amd o kvm_intel

## Chequear que el procesador tenga la Virtualización activada desde el BIOS 
`lscpu | grep virtua`     #   virtua solamente, porque si el S.O. esta en ingles o español cambia la palabra virtualización/tion

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

## Habilitar virtualización dentro de una VM con acelearación de Hardware del Host(Por ejemplo para utilizar el emulador Wine dentro de la VM)

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

##Optimizar el host con TuneD (Opcional)
1. Instalar TuneD
`sudo pacman -S tuned`
2. Habilitar TuneD
```
sudo systemctl start tuned
sudo systemctl enable tuned
```
3. Setear el profile de optimización del host
`tuned-adm profile virtual-host`
4.Verificar el profile
`sudo tuned-adm verify`

## Agregar el usuario a el grupo libvirtd.
### Probamos si el grupo existe.
`groups` 

### Sino existe lo  creamos con:   
`newgrp libvirt`

### Agregamos nuestro usuario o el usuario que querramos usar con la qemu.
`useradd -aG libvirt $(whomi)`

## Ejecutamos el Frontend de qemu (virt-manager). 
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


