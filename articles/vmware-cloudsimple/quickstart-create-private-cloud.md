---
title: 'Inicio rápido: Creación de una nube privada'
titleSuffix: Azure VMware Solutions by CloudSimple
description: Obtenga información sobre cómo crear y configurar una nube privada con Azure VMware Solution by CloudSimple
author: sharaths-cs
ms.author: dikamath
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 487308aca3231650aee3fac5ae127006649e19b9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073915"
---
# <a name="quickstart---configure-a-private-cloud-environment"></a>Inicio rápido: configuración del entorno de una nube privada

En este artículo, obtenga información sobre cómo crear una nube privada de CloudSimple y configurar el entorno de nube privada.

## <a name="before-you-begin"></a>Antes de empezar

Consulte [Requisitos previos de redes avanzadas](cloudsimple-network-checklist.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-private-cloud"></a>Creación de una nube privada

Una nube privada es una pila de VMware aislada que admite hosts ESXi, vCenter, vSAN y NSX.

Las nubes privadas se administran desde el portal de CloudSimple. Tienen su propio servidor vCenter en su propio dominio de administración. La pila se ejecuta en nodos dedicados y nodos de hardware sin sistema operativo aislados.

1. Seleccione **Todos los servicios**.
2. Busque **Servicios de CloudSimple**.
3. Seleccione el servicio de CloudSimple en el que quiere crear la nube privada.
4. En **Información general**, haga clic en **Crear nube privada** para abrir una nueva pestaña del explorador del portal de CloudSimple.  Si se le solicita, inicie sesión con las credenciales de inicio de sesión en Azure.  

    ![Creación de una nube privada desde Azure](media/create-private-cloud-from-azure.png)

5. En el portal de CloudSimple, proporcione un nombre para su nube privada.
6. Seleccione la **ubicación** de la nube privada.
7. Seleccione **Tipo de nodo** de acuerdo con lo que haya aprovisionado en Azure.
8. Especifique el **número de nodos**.  Para crear una nube privada, se requiere un mínimo de tres nodos.

    ![Creación de una nube privada: información básica](media/create-private-cloud-basic-info.png)

9. Haga clic en **Siguiente: Opciones avanzadas**.
10. Escriba el intervalo CIDR de subredes de vSphere/vSAN. Asegúrese de que el intervalo CIDR no se superponga con ninguna de las subredes locales u otras subredes de Azure (redes virtuales) ni con la subred de la puerta de enlace.

    **Opciones de intervalo de CIDR:** /24, /23, /22 o /21. Un intervalo de CIDR /24 admite hasta 26 nodos, un intervalo de CIDR /23 admite hasta 58 nodos y un intervalo de CIDR /22 y /21 admite hasta 64 nodos (el número máximo de nodos en una nube privada).  Para obtener más información sobre redes VLAN y subredes, consulte la [información general sobre redes VLAN y subredes](cloudsimple-vlans-subnets.md).

      > [!IMPORTANT]
      > Las direcciones IP en el intervalo CIDR de vSphere/vSAN están reservadas para que las use la infraestructura de nube privada.  No use la dirección IP de este intervalo en ninguna máquina virtual.

11. Haga clic en **Siguiente: Revisar y crear**.
12. Revise la configuración. Si necesita cambiar la configuración, haga clic en **Anterior**.
13. Haga clic en **Crear**.

Se inicia el proceso de aprovisionamiento de la nube privada.  La nube privada puede tardar hasta dos horas en aprovisionarse.

## <a name="launch-cloudsimple-portal"></a>Inicio del portal de CloudSimple

Puede acceder al portal de CloudSimple desde Azure Portal.  El portal de CloudSimple se iniciará con sus credenciales de inicio de sesión de Azure mediante el inicio de sesión único (SSO).  Para acceder al portal de CloudSimple, debe autorizar la aplicación **Autorización del servicio de CloudSimple**.  Para obtener más información sobre la concesión de permisos, consulte [Consent to CloudSimple Service Authorization application](access-cloudsimple-portal.md#consent-to-cloudsimple-service-authorization-application) (Dar su consentimiento a la aplicación de autorización de servicio CloudSimple).

1. Seleccione **Todos los servicios**.
2. Busque **Servicios de CloudSimple**.
3. Seleccione el servicio de CloudSimple en el que quiere crear la nube privada.
4. En la introducción, haga clic en **Go to the CloudSimple portal** (Ir al portal de CloudSimple) para abrir una nueva pestaña del explorador del portal de CloudSimple.  Si se le solicita, inicie sesión con las credenciales de inicio de sesión en Azure.  

    ![Inicio del portal de CloudSimple](media/launch-cloudsimple-portal.png)

## <a name="create-point-to-site-vpn"></a>Creación de una VPN de punto a sitio

Una conexión VPN de punto a sitio es la manera más sencilla de conectarse a la nube privada desde su equipo. Utilice la conexión VPN de punto a sitio si se conecta a la nube privada de forma remota.  Para obtener acceso rápido a la nube privada, siga estos pasos.  Desde la red local, se puede acceder a la región de CloudSimple mediante [VPN de sitio a sitio](vpn-gateway.md) o [Azure ExpressRoute](on-premises-connection.md).

### <a name="create-gateway"></a>Crear puerta de enlace

1. Inicie el portal de CloudSimple y seleccione **Network** (Red).
2. Seleccione **VPN Gateway** (Puerta de enlace de VPN).
3. Haga clic en **New VPN Gateway** (Nueva puerta de enlace de VPN).

    ![Creación de una puerta de enlace de VPN](media/create-vpn-gateway.png)

4. En **Gateway Configuration** (Configuración de puerta de enlace), especifique la configuración que se indica a continuación y haga clic en **Next** (Siguiente).

    * Seleccione **Point-to-Site VPN** (VPN de punto a sitio) como el tipo de puerta de enlace.
    * Escriba un nombre para identificar la puerta de enlace.
    * Seleccione la ubicación de Azure en que se va a implementar el servicio de CloudSimple.
    * Especifique la subred de cliente para la puerta de enlace de punto a sitio.  Cuando se conecte, se proporcionarán direcciones DHCP desde esta subred.

5. En **Connection/User** (Conexión/Usuario), especifique la configuración que se indica a continuación y haga clic en **Next** (Siguiente).

    * Para permitir automáticamente que todos los usuarios actuales y futuros tengan acceso a la nube privada a través de esta puerta de enlace de punto a sitio, seleccione **Automatically add all users** (Agregar automáticamente a todos los usuarios). Al seleccionar esta opción, se seleccionan automáticamente todos los usuarios de la lista de usuarios. Para invalidar la opción automática, anule la selección de usuarios individuales en la lista.
    * Para seleccionar únicamente usuarios individuales, haga clic en las casillas de verificación de la lista de usuarios.

6. En la sección VLANs/Subnets (VLAN/Subredes) se pueden especificar las VLAN y las subredes de administración y de usuario para la puerta de enlace y las conexiones.

    * Las opciones de **Automatically add** (Agregar automáticamente) establecen la directiva global para esta puerta de enlace. La configuración se aplica a la puerta de enlace actual. Se puede invalidar la configuración en el área **Select** (Seleccionar).
    * Seleccione **Add management VLANs/Subnets of Private Clouds** (Agregar VLAN y subredes de administración de nubes privadas).
    * Para agregar todas las VLAN y subredes definidas por el usuario, haga clic en **Add user-defined VLANs/Subnets** (Agregar VLAN y subredes definidas por el usuario).
    * La configuración de **Select** (seleccionar) invalida la configuración global en **Automatically add** (Agregar automáticamente).

7. Haga clic en **Next** (Siguiente) para revisar la configuración. Haga clic en los iconos de edición para realizar cambios.
8. Haga clic en **Crete** (Crear) para crear la puerta de enlace VPN.

### <a name="connect-to-cloudsimple-using-point-to-site-vpn"></a>Conexión a CloudSimple mediante VPN de punto a sitio

Para conectarse a CloudSimple desde su equipo, es necesario el cliente VPN.  Descargue el [cliente OpenVPN](https://openvpn.net/community-downloads/) para Windows o [Viscosity](https://www.sparklabs.com/viscosity/download/) para macOS y OS X.

1. Inicie el portal de CloudSimple y seleccione **Network** (Red).
2. Seleccione **VPN Gateway** (Puerta de enlace de VPN).
3. En la lista de puertas de enlace VPN, haga clic en la puerta de enlace VPN de punto a sitio.
4. Seleccione **Usuarios**.
5. Haga clic en **Download my VPN configuration** (Descargar mi configuración de VPN)

    ![Descarga de la configuración de VPN](media/download-p2s-vpn-configuration.png)

6. Importe la configuración en el cliente VPN.

    * Instrucciones para [importar la configuración en el cliente Windows](https://openvpn.net/vpn-server-resources/connecting-to-access-server-with-windows/#openvpn-open-source-openvpn-gui-program)
    * Instrucciones para [importar la configuración en macOS u OS X](https://www.sparklabs.com/support/kb/article/getting-started-with-viscosity-mac/#creating-your-first-connection)

7. Conéctese a CloudSimple.

## <a name="create-a-vlan-for-your-workload-vms"></a>Creación de una VLAN para las máquinas virtuales de carga de trabajo

Después de crear una nube privada, cree la VLAN en que va a implementar las máquinas virtuales de carga de trabajo y aplicación.

1. En el portal de CloudSimple, seleccione **Network** (Red).
2. Haga clic en **VLANs/Sunets** (VLAN/Subredes).
3. Haga clic en **Create VLAN/Subnet** (Crear VLAN o subred).

    ![Creación de una VLAN o subred](media/create-new-vlan-subnet.png)

4. Seleccione la **Nube privada** para la VLAN o la subred nuevas.
5. Seleccione un identificador de VLAN en la lista.  
6. Escriba un nombre de subred para identificar la subred.
7. Especifique el intervalo de CIDR de subred y la máscara.  Este intervalo no debe superponerse a las subredes existentes.
8. Haga clic en **Enviar**.

    ![Creación de los detalles de una VLAN o subred](media/create-new-vlan-subnet-details.png)

Se creará la VLAN o la subred.  Ahora puede usar este identificador de VLAN para crear un grupo de puertos distribuidos en vCenter de nube privada.

## <a name="connect-your-environment-to-an-azure-virtual-network"></a>Conexión del entorno a una red virtual de Azure

CloudSimple le proporciona un circuito ExpressRoute para la nube privada. Puede conectar la red virtual de Azure al circuito ExpressRoute. Para obtener detalles completos sobre cómo configurar la conexión, siga los pasos de [Azure Virtual Network Connection using ExpressRoute](./cloudsimple-azure-network-connection.md) (Conexión de la red virtual de Azure mediante ExpressRoute).

## <a name="sign-in-to-vcenter"></a>Inicio de sesión en vCenter

Ahora puede iniciar sesión en vCenter para configurar las máquinas virtuales y las directivas.

1. Para acceder a vCenter, empiece desde el portal de CloudSimple. En la página principal, en **Common Tasks** (Tareas comunes), haga clic en **Launch vSphere Client** (Iniciar cliente vSphere).  Seleccione la nube privada y, después, haga clic en **Launch vSphere Client** (Iniciar cliente vSphere) en la nube privada.

    ![Inicio del cliente vSphere](media/launch-vcenter-from-cloudsimple-portal.png)

2. Seleccione el cliente vSphere preferido para acceder a vCenter e inicie sesión con su nombre de usuario y contraseña.  Los valores predeterminados son:
    * Nombre de usuario: `CloudOwner@cloudsimple.local`
    * Contraseña: `CloudSimple123!`

Las pantallas de vCenter en los procedimientos siguientes son del cliente vSphere (HTML5).

## <a name="change-your-vcenter-password"></a>Cambio de la contraseña de vCenter

CloudSimple recomienda cambiar la contraseña la primera vez que inicie sesión en vCenter.  
La contraseña que establezca debe cumplir los requisitos siguientes:

* Duración máxima: debe cambiar la contraseña cada 365 días
* Restricción de reutilización: los usuarios no pueden reutilizar ninguna de las últimas cinco contraseñas
* Longitud: entre 8 y 20 caracteres
* Carácter especial: un carácter especial como mínimo
* Caracteres alfabéticos: un carácter en mayúscula (A-Z) como mínimo y un carácter en minúscula (a-z) como mínimo
* Números: un carácter numérico (de 0 a 9) como mínimo
* Máximo de caracteres adyacentes idénticos: tres

    Ejemplo: CC o CCC es aceptable como parte de la contraseña, pero no CCCC.

Si establece una contraseña que no cumpla los requisitos:

* Si usa el cliente vSphere Flash, notifica un error
* Si usa el cliente HTML5, no notifica ningún error. El cliente no acepta el cambio y la contraseña antigua continúa funcionando.

## <a name="access-nsx-manager"></a>Acceso al administrador de NSX

El administrador de NSX se implementa con una contraseña predeterminada. 

* Nombre de usuario: **admin**
* Contraseña: **CloudSimple123!**

Puede encontrar el nombre de dominio completo (FQDN) y la dirección IP del administrador de NSX en el portal de CloudSimple.

1. Inicie el portal de CloudSimple y seleccione **Resources** (Recursos).
2. Haga clic en la nube privada que quiera usar.
3. Seleccione **vSphere management network** (Red de administración de vSphere).
4. Use la dirección IP o el FQDN de **NSX Manager** y conéctese mediante un explorador web.

    ![Buscar el FQDN de NSX Manager](media/private-cloud-nsx-manager-fqdn.png)

## <a name="create-a-port-group"></a>Creación de un grupo de puertos

Para crear un grupo de puertos distribuidos en vSphere:

1. Siga las instrucciones para agregar un grupo de puertos distribuidos en la [Guía de red de vSphere](https://docs.vmware.com/en/VMware-vSphere/6.5/vsphere-esxi-vcenter-server-65-networking-guide.pdf).
2. Al configurar el grupo de puertos distribuidos, proporcione el identificador de VLAN que ha creado en [Creación de una VLAN para la carga de trabajo de máquinas virtuales](#create-a-vlan-for-your-workload-vms).

## <a name="next-steps"></a>Pasos siguientes

* [Uso de máquinas virtuales de VMware en Azure](quickstart-create-vmware-virtual-machine.md)
* [Conexión a la red local mediante Azure ExpressRoute](on-premises-connection.md)
* [Set up Site-to-Site VPN from on-premises](vpn-gateway.md) (Configurar una VPN de sitio a sitio desde un entorno local)
