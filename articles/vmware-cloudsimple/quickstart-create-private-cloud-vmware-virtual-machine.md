---
title: 'Inicio rápido: Creación de una máquina virtual de VMware de Azure en una nube privada: Azure VMware Solution by CloudSimple'
description: Se describe cómo crear una máquina virtual de VMware de Azure en una nube privada de CloudSimple.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 000e30f2656b88c068654fd8099cdff0ce0a4b37
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091273"
---
# <a name="create-vmware-virtual-machines-on-your-private-cloud"></a>Creación de máquinas virtuales de VMware en la nube privada

Para crear máquinas virtuales en una nube privada propia, primero debe acceder al portal de CloudSimple desde Azure Portal.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="access-the-cloudsimple-portal"></a>Acceso al portal de CloudSimple

1. Seleccione **Todos los servicios**.
2. Busque **Servicios de CloudSimple**.
3. Seleccione el servicio de CloudSimple en el que quiere crear la nube privada.
4. En la página **Información general**, haga clic en **Go to the CloudSimple portal** (Ir al portal de CloudSimple) para abrir una nueva pestaña del explorador para el portal de CloudSimple.  Si se le solicita, inicie sesión con las credenciales de inicio de sesión en Azure.  

    ![Inicio del portal de CloudSimple](media/launch-cloudsimple-portal.png)

## <a name="launch-vcenter-web-ui"></a>Inicio de la interfaz de usuario web de vCenter

Ahora puede iniciar vCenter para configurar máquinas virtuales y directivas.

Para acceder a vCenter, empiece desde el portal de CloudSimple. En la página principal, en **Common Tasks** (Tareas comunes), haga clic en **Launch vSphere Client** (Iniciar cliente vSphere).  Seleccione la nube privada y, después, haga clic en **Launch vSphere Client** (Iniciar cliente vSphere) en la nube privada.

   ![Inicio del cliente vSphere](media/launch-vcenter-from-cloudsimple-portal.png)

## <a name="upload-an-iso-or-vsphere-template"></a>Carga de una plantilla de vSphere o ISO

  > [!WARNING]
  > Para las cargas de ISO, use el cliente HTML5 de vSphere.  El uso de un cliente Flash puede producir un error.

1. Obtenga la plantilla de vSphere o ISO que quiera cargar en vCenter para crear una máquina virtual y póngala a disposición del sistema local.
2. En vCenter, haga clic en el icono **Disk** (Disco) y seleccione **vsanDatastore**. Haga clic en **Files** (Archivos) y después en **New Folder** (nueva carpeta).
    ![ISO de vCenter](media/vciso00.png)

3. Cree una carpeta denominada "Archivos ISO y plantillas".

4. Vaya a la carpeta de archivos ISO en "Archivos ISO y plantillas" y haga clic en **Upload Files** (Cargar archivos). Siga las instrucciones en pantalla para cargar el archivo ISO.

## <a name="create-a-virtual-machine-in-vcenter"></a>Creación de una máquina virtual en vCenter

1. En vCenter, haga clic en el icono **Hosts and Clusters** (Hosts y clústeres).

2. Haga clic con el botón derecho en **Workload** (Carga de trabajo) y seleccione **New Virtual Machine** (Nueva máquina virtual).
    ![Nueva máquina virtual](media/vcvm01.png)

3. Seleccione **Create new virtual machine** (Crear nueva máquina virtual) y haga clic en **Next** (Siguiente).
    ![Nueva máquina virtual](media/vcvm02.png)

4. Asigne un nombre a la máquina virtual, seleccione la ubicación de la **máquina virtual de la carga de trabajo** y haga clic en **Next** (Siguiente).
    ![Nueva máquina virtual](media/vcvm03.png)

5. Seleccione el recurso de proceso **Workload** (Carga de trabajo) y haga clic en **Next** (Siguiente).
    ![Nueva máquina virtual](media/vcvm04.png)

6. Seleccione **vsanDatastore** y haga clic en **Next** (Siguiente).
    ![Nueva máquina virtual](media/vcvm05.png)

7. Mantenga la selección de compatibilidad de ESXi 6.5 predeterminada y haga clic en **Next** (Siguiente).
    ![Nueva máquina virtual](media/vcvm06.png)

8. Seleccione el SO invitado del archivo ISO para la máquina virtual que va a crear y haga clic en **Next** (Siguiente).
    ![Nueva máquina virtual](media/vcvm07.png)

9. Seleccione las opciones de disco duro y de red. Para la nueva unidad de CD/DVD, seleccione **Datastore ISO file** (Archivo ISO de almacén de datos).  Si quiere permitir el tráfico entre la dirección IP pública y esta máquina virtual, seleccione la red como **vm-1**.
    ![Nueva máquina virtual](media/vcvm08.png)

10. Se abrirá una ventana de selección. Seleccione el archivo que ha cargado previamente en la carpeta de archivos ISO y plantillas, y haga clic en **Aceptar**.
    ![Nueva máquina virtual](media/vcvm10.png)

11. Revise la configuración y haga clic en **OK** (Aceptar) para crear la nueva máquina virtual.
    ![Nueva máquina virtual](media/vcvm11.png)

La máquina virtual se agregado ahora a los recursos de proceso de carga de trabajo y está lista para su uso. 
![Nueva máquina virtual](media/vcvm12.png)

La configuración básica ya se ha completado. Puede empezar a usar su nube privada de la misma forma que usaría la infraestructura de máquina virtual local.

Las secciones siguientes contienen información opcional sobre cómo configurar servidores DNS y DHCP para cargas de trabajo de la nube privada y modificar la configuración de red predeterminada.

## <a name="add-users-and-identity-sources-to-vcenter-optional"></a>Adición de usuarios y orígenes de identidades a vCenter (opcional)

CloudSimple asigna una cuenta de usuario de vCenter predeterminada con el nombre de usuario `cloudowner@cloudsimple.local`. Para empezar a trabajar no es necesario configurar cuentas adicionales.  CloudSimple normalmente asigna a los administradores los privilegios que necesitan para realizar las operaciones normales.  Configure la instancia local de Active Directory o Azure AD como un [origen de identidades adicional](set-vcenter-identity.md) en la nube privada.

## <a name="create-a-dns-and-dhcp-server-optional"></a>Creación de un servidor DNS y DHCP (opcional)

Las aplicaciones y cargas de trabajo que se ejecutan en un entorno de nube privada requieren servicios de resolución de nombres y DHCP para la búsqueda y asignación de direcciones IP. Para proporcionar estos servicios se requiere una infraestructura de DHCP y DNS adecuada. Puede configurar una máquina virtual en vCenter para proporcionar estos servicios en el entorno de nube privada.

Prerrequisitos

* Un grupo de puertos distribuidos con VLAN configurada

* Ruta configurada a servidores DNS basados en Internet o locales

* Plantilla de máquina virtual o archivo ISO para crear una máquina virtual

Los vínculos siguientes proporcionan instrucciones sobre cómo configurar los servidores DHCP y DNS en Windows y Linux.

#### <a name="linux-based-dns-server-setup"></a>Configuración del servidor DNS basado en Linux

Linux ofrece diversos paquetes para la configuración de servidores DNS.  Este es un vínculo a instrucciones sobre cómo configurar un servidor DNS BIND de código abierto.

[Configuración de ejemplo](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-centos-7)

#### <a name="windows-based-setup"></a>Configuración basada en Windows

En estos temas de Microsoft se describe cómo configurar un servidor de Windows como un servidor DNS y como un servidor DHCP.

[Windows Server como servidor DNS](/windows-server/networking/dns/dns-top)

[Windows Server como servidor DHCP](/windows-server/networking/technologies/dhcp/dhcp-top)

## <a name="customize-networking-configuration-optional"></a>Personalización de la configuración de red (opcional)

Las páginas de red del portal de CloudSimple permiten especificar la configuración de las tablas de firewall y las direcciones IP públicas para las máquinas virtuales.

### <a name="allocate-public-ips"></a>Asignación de IP públicas

1. Vaya a **Network > Public IP** (Red > Dirección IP pública) en el portal de CloudSimple.
2. Haga clic en **Allocate Public IP** (Asignar IP pública).
3. Escriba un nombre para identificar la entrada de la dirección IP.
4. Mantenga la ubicación predeterminada.
5. Use el control deslizante para cambiar el tiempo de espera de inactividad si lo desea.
6. Escriba la dirección IP local a la que quiere asignar una dirección IP pública.
7. Si lo desea, escriba un nombre DNS asociado.
8. Haga clic en **Done**(Listo).

    ![Dirección IP pública](media/quick-create-pc-public-ip.png)

Comienza la tarea de asignar la dirección IP pública. Puede comprobar el estado de la tarea en la página **Activity > Tasks** (Actividad > Tareas). Cuando se complete la asignación, se muestra la nueva entrada en la página de direcciones IP públicas.

La máquina virtual a la que se debe asignar esta dirección IP se tiene que configurar con la dirección local especificada antes. El procedimiento para configurar una dirección IP es específico del sistema operativo de la máquina virtual. Consulte la documentación del sistema operativo de la máquina virtual para conocer el procedimiento correcto.

#### <a name="example"></a>Ejemplo

Por ejemplo, estos son los detalles para Ubuntu 16.04.

Agregue el método estático a la configuración de la familia de direcciones inet en el archivo /etc/network/interfaces. Cambie los valores de dirección, máscara de red y puerta de enlace. En este ejemplo, se usa la interfaz eth0, dirección IP interna 192.168.24.10, la dirección de puerta de enlace 192.168.24.1 y la máscara de red 255.255.255.0. En su entorno, se proporciona la información de la subred disponible se proporciona en el correo electrónico de bienvenida.

```
sudo vi /etc/network/interfaces
```

```
auto eth0
Iface eth0 inet static
iface eth0 inet static
address 192.168.24.10
netmask 255.255.255.0
gateway 192.168.24.1
dns-nameservers 8.8.8.8
dns-domain acme.com
dns-search acme.com
```
Deshabilite manualmente la interfaz.

```
sudo ifdown eth0
```
Vuelva a habilitar manualmente la interfaz.

```
sudo ifup eth0
```

De forma predeterminada, se **deniega** todo el tráfico entrante desde Internet. Si quiere abrir cualquier otro puerto, cree una [tabla de firewall](firewall.md).

Después de configurar una dirección IP interna como dirección IP estática, compruebe que puede acceder a Internet desde la máquina virtual.

```
ping 8.8.8.8
```
Compruebe también que puede acceder a la máquina virtual desde Internet mediante la dirección IP pública.

Asegúrese de que las reglas de iptable en la máquina virtual no bloqueen el puerto 80 entrante.
        
```
netstat -an | grep 80
```

Inicie un servidor HTTP que escuche en el puerto 80.
       
```
python2.7 -m SimpleHTTPServer 80
```

or

```
python3 -m http.server 80
```
Inicie un explorador en el escritorio y haga que apunte al puerto 80 para que la dirección IP pública examine los archivos de la máquina virtual.

### <a name="default-cloudsimple-firewall-rules-for-public-ip"></a>Reglas de firewall de CloudSimple predeterminadas para la dirección IP pública

* Tráfico VPN: se permite todo el tráfico entre (desde y hacia) la VPN y todas las redes de carga de trabajo y la red de administración.
* Tráfico interno de la nube privada: se permite todo el tráfico este-oeste entre (desde y hacia) las redes de carga de trabajo y la red de administración (mostrada arriba).
* Tráfico de Internet:
  * se deniega todo el tráfico entrante desde Internet a las redes de carga de trabajo y la red de administración.
  * Se permite todo el tráfico saliente a Internet desde las redes de carga de trabajo o la red de administración.

También puede modificar la forma en que se protege el tráfico, mediante la característica Reglas de firewall. Para más información, vea [Configuración de tablas y reglas de firewall](firewall.md).

## <a name="install-solutions-optional"></a>Instalación de soluciones (opcional)

Puede instalar soluciones en la nube privada de CloudSimple para aprovechar al máximo el entorno de vCenter de la nube privada. Puede configurar funciones de copia de seguridad, recuperación ante desastres, replicación y otras para proteger las máquinas virtuales. Algunos ejemplos son el Administrador de recuperación de sitios de VMware (VMware SRM) y Veeam Backup & Replication.

Para instalar una solución, debe solicitar privilegios adicionales durante un período limitado. Vea [Escalado de privilegios](escalate-private-cloud-privileges.md).

## <a name="next-steps"></a>Pasos siguientes

* [Uso de máquinas virtuales de VMware en Azure](quickstart-create-vmware-virtual-machine.md)
* [Conexión a la red local mediante Azure ExpressRoute](on-premises-connection.md)
* [Configuración de puertas de enlace de VPN en la red de CloudSimple](vpn-gateway.md)
