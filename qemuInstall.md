##########################################
# Guia de Instalación de Qemu/KVM + GUI: #
##########################################

#   Actualizar el sistema
sudo pacman -Syu

#   Chequear si los modulos de virtualización estan cargados
lsmod | grep kvm

#   Buscar , según corresponda kvm_amd o kvm_intel

#   Chequear que el procesador tenga la Virtualización activada desde el BIOS 
lscpu | grep virtua     #   virtua solamente, porque si esta en ingles o español cambia la plabra virtualización/tion

#   Instalar qemu +KVM
sudo pacman -S qemu virt-manager libvirt dnsmasq ebtables

#   * qemu: El software de virtualización.
#   * virt-manager: Interfaz gráfica para gestionar máquinas virtuales.
#   * libvirt: Biblioteca para interactuar con QEMU/KVM.
#   * dnsmasq y ebtables: Para soporte de redes en las VMs.

#   Habilitar el backend de Virt-manager.
sudo systemctl enable libvirtd
sudo systemctl start libvirtd

#   Agregar el usuario a el grupo libvirtd.
#   Probamos si el grupo existe.
groups 

#   sino existe lo  creamos con:   
newgrp libvirt

#   Agregamos nuestro usuario o el usuario que querramos usar con la qemu.
useradd -aG libvirt $(whomi)

#   Ejecutamos el Frontend de qemu (virt-manager). 
virt-manager
###########
#  Notas: #
###########
#  Para activar y desactivar el daemon y servicios asociados, utilizar: https://github.com/e-conte/scripts/blob/main/toggle-virt.sh

#  para bindear maquinas en I3wm:
bindsym "key" exec  virt-manager --connect qemu:///system --show-domain-console  "NombreDeLaVM"

####################
#  Info adicional: #
####################

#   * KVM, Kernel based virtualization machine, es decir.
#   * Qemu es un hypervisor de KVM ó aplicación que permite virtualizar.
#   * virt-manager es una GUI para KVM, el frontend.
#   * libvirt es una libreria escrita en C que proporciona una API quee permite usar diferentes hypervisores. 
#   * libvirt también es el daemon de backend de la misma y tiene la herramientas CLI virsh.
