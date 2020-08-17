---
title: Solución de problemas detallada de SSH para una máquina virtual de Azure | Microsoft Docs
description: Pasos de solución de problemas de SSH más detallados para los problemas de conexión a una máquina virtual de Azure
keywords: conexión ssh rechazada, error de ssh azure ssh, error de conexión ssh
services: virtual-machines-linux
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.assetid: b8e8be5f-e8a6-489d-9922-9df8de32e839
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 0b4890181721d08b741d327adb74bd097be5c9f2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87069171"
---
# <a name="detailed-ssh-troubleshooting-steps-for-issues-connecting-to-a-linux-vm-in-azure"></a>Pasos detallados de solución de problemas de SSH para los problemas de conexión a una máquina virtual Linux en Azure
Hay muchas razones posibles por las que el cliente SSH podría no ser capaz de ponerse en contacto con el servicio SSH en la máquina virtual. Si ha seguido los [pasos más generales de solución de problemas de SSH](troubleshoot-ssh-connection.md), debe solucionar además el problema de conexión. Este artículo le guiará por los pasos de solución de problemas detallados para determinar dónde se producen errores en la conexión SSH y cómo resolverlos.

## <a name="take-preliminary-steps"></a>Pasos previos
El siguiente diagrama muestra los componentes que intervienen.

![Diagrama que muestra los componentes del servicio SSH](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot1.png)

Los siguientes pasos ayudan a aislar la causa del error y a averiguar soluciones o alternativas.

1. Compruebe el estado de la máquina virtual en el portal.
   En [Azure Portal](https://portal.azure.com), seleccione **Máquinas virtuales** > *Nombre de máquina virtual*.

   El panel de estado de la máquina virtual debe mostrar **En ejecución**. Desplácese hacia abajo para ver la actividad reciente de los recursos de proceso, almacenamiento y de red.

2. Seleccione **Configuración** para examinar los puntos de conexión, las direcciones IP, los grupos de seguridad de red y otros valores de configuración.

   La máquina virtual debe tener un punto de conexión definido para el tráfico SSH que puede ver en **Puntos de conexión** o en **[Grupo de seguridad de red](../../virtual-network/security-overview.md)** . Los puntos de conexión en las máquinas virtuales que se crearon con Resource Manager se almacenan en un grupo de seguridad de red. Compruebe que las reglas se hayan aplicado al grupo de seguridad de red y que se haga referencia a ellas en la subred.

Para comprobar la conectividad de red, compruebe los puntos de conexión configurados y vea si puede conectarse con la máquina virtual a través de otro protocolo, como HTTP u otro servicio.

Después de realizar estos pasos, vuelva a intentar la conexión a SSH.

## <a name="find-the-source-of-the-issue"></a>Averiguar el origen del problema
El cliente de SSH de su equipo podría generar un error al conectarse con el servicio de SSH en la máquina virtual de Azure debido a problemas o errores de configuración en las siguientes áreas:

* [Equipo cliente de SSH](#source-1-ssh-client-computer)
* [Dispositivo perimetral de la organización](#source-2-organization-edge-device)
* [Extremo de servicio en la nube y lista de control de acceso (ACL)](#source-3-cloud-service-endpoint-and-acl)
* [Grupos de seguridad de red](#source-4-network-security-groups)
* [Máquina virtual de Azure basada en Linux](#source-5-linux-based-azure-virtual-machine)

## <a name="source-1-ssh-client-computer"></a>Causa 1: Equipo cliente de SSH
Para descartar el equipo como causa del error, compruebe que puede establecer conexiones a SSH con otro equipo local basado en Linux.

![Diagrama que resalta los componentes del equipo cliente de SSH](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot2.png)

Si la conexión falla, compruebe si el equipo tiene los siguientes errores:

* Una configuración del firewall local que está bloqueando el tráfico SSH entrante o saliente (TCP 22)
* Software de proxy de cliente instalado localmente que impide las conexiones a SSH
* Software de supervisión de red instalado localmente que impide las conexiones a SSH
* Otro tipo de software de seguridad que o bien, supervisar el tráfico o bien, permite o impide ciertos tipos de tráfico.

Si alguna de estas condiciones es aplicable, deshabilite el software temporalmente e intente una conexión SSH a un equipo local para averiguar el motivo de que la conexión esté bloqueada en su equipo. Después, colabore con el administrador de red para corregir la configuración del software a fin de permitir conexiones SSH.

Si utiliza la autenticación de certificados, compruebe que dispone de estos permisos a la carpeta .ssh en el directorio principal:

* Chmod 700 ~/.ssh
* Chmod 644 ~/.ssh/\*.pub
* Chmod 600 ~/.ssh/id_rsa (o cualquier otro archivo en el que tenga almacenadas las claves privadas)
* Chmod 644 ~/.ssh/known_hosts (contiene los hosts a los que se ha conectado a través de SSH)

## <a name="source-2-organization-edge-device"></a>Causa 2: Dispositivo perimetral de la organización
Para descartar el dispositivo perimetral de la organización como causa de los errores, compruebe que un equipo conectado directamente a Internet puede establecer conexiones a SSH con la máquina virtual de Azure. Si va a acceder a la máquina virtual mediante una VPN de sitio a sitio o una conexión de Azure ExpressRoute, vaya a [Causa 4: Grupos de seguridad de red](#nsg)

![Diagrama que resalta el dispositivo perimetral de la organización](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot3.png)

Si no tiene un equipo conectado directamente a Internet, cree una nueva máquina virtual de Azure en su propio grupo de recursos o servicio en la nube y úsela. Para más información, consulte [Creación de una máquina virtual que ejecuta Linux en Azure](../linux/quick-create-cli.md). Cuando acabe de realizar las pruebas, elimine el grupo de recursos o la máquina virtual y el servicio en la nube.

Si puede crear una conexión SSH con un equipo que esté conectado directamente a Internet, compruebe si el dispositivo perimetral de la organización tiene lo siguiente:

* Un firewall interno que bloquea el tráfico SSH a Internet
* Un servidor proxy que impide las conexiones SSH
* Software de detección de intrusiones o supervisión de red en ejecución en los dispositivos de la red perimetral que impide las conexiones SSH

Trabaje con el administrador de red para corregir la configuración de los dispositivos perimetrales de la organización para permitir el tráfico SSH a Internet.

## <a name="source-3-cloud-service-endpoint-and-acl"></a>Causa 3: Punto de conexión de servicio en la nube y ACL

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

> [!NOTE]
> Esta causa se aplica solo a las máquinas virtuales creadas con el modelo de implementación clásica. Si la máquina virtual se creó mediante Resource Manager, vaya a [Causa 4: Grupos de seguridad de red](#nsg)

Para descartar el punto de conexión de servicio en la nube y ACL como el origen del error, compruebe si otra máquina virtual de Azure de la misma red virtual puede conectarse mediante SSH.

![Diagrama que resalta el punto de conexión del servicio en la nube y ACL](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot4.png)

Si no tiene otra máquina virtual en la misma red virtual, puede crear una fácilmente. Para más información, consulte [Creación de una máquina virtual Linux en Azure mediante la CLI](../linux/quick-create-cli.md). Cuando acabe de realizar las pruebas, elimine la máquina virtual que creó.

Si puede crear una conexión SSH con una máquina virtual en la misma red virtual, compruebe las siguientes áreas:

* **La configuración del punto de conexión para el tráfico de SSH en la máquina virtual de destino.** El puerto TCP privado del punto de conexión debe coincidir con el puerto TCP en el que escucha el servicio SSH en la máquina virtual. (El puerto predeterminado es 22). Para comprobar que el número de puerto TCP de SSH en Azure Portal, seleccione **Máquinas virtuales** > *Nombre de máquina virtual* > **Configuración** > **Puntos de conexión**.
* **La ACL del punto de conexión para el tráfico de SSH en la máquina virtual de destino.** Una ACL permite especificar el tráfico entrante de Internet que se permite o se deniega en función de la dirección IP de origen. Las ACL mal configuradas pueden impedir el tráfico entrante de SSH al punto de conexión. Compruebe las ACL para asegurarse de que está permitido el tráfico entrante desde las direcciones IP públicas del proxy o de otro servidor perimetral. Para obtener más información, consulte [Acerca de las listas de control de acceso (ACL) de red](/previous-versions/azure/virtual-network/virtual-networks-acl).

Para descartar el punto de conexión como causa del problema, quite el actual, cree uno y especifique el nombre SSH (puerto TCP 22 para el número de puerto público y privado). Para obtener más información, vea [Configuración de puntos de conexión en una máquina virtual en Azure](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints?toc=/azure/virtual-machines/windows/classic/toc.json).

<a id="nsg"></a>

## <a name="source-4-network-security-groups"></a>Causa 4: Grupos de seguridad de red
Los grupos de seguridad de red permiten un control pormenorizado del tráfico entrante y saliente permitido. Puede crear reglas que abarquen subredes y servicios en la nube en una red virtual de Azure. Compruebe las reglas de los grupos de seguridad de red para asegurarse de que se permite el tráfico de SSH tanto a Internet como desde Internet.
Para obtener más información, consulte [Acerca de los grupos de seguridad de red](../../virtual-network/security-overview.md).

También puede usar la comprobación de IP para validar la configuración del grupo de seguridad de red. Para más información, consulte [Información general sobre la supervisión de red de Azure](../../network-watcher/network-watcher-monitoring-overview.md). 

## <a name="source-5-linux-based-azure-virtual-machine"></a>Causa 5: Máquina virtual de Azure basada en Linux
La última causa de los posibles problemas puede residir en la propia máquina virtual de Azure.

![Diagrama que resalta la máquina virtual de Azure basada en Linux](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot5.png)

Si aún no lo ha hecho, siga las instrucciones [para restablecer una contraseña para máquinas virtuales basadas en Linux](./reset-password.md).

Pruebe de nuevo la conexión desde su equipo. Si sigue sin funcionar, estos son algunos de los posibles problemas:

* El servicio de SSH no se está ejecutando en la máquina virtual de destino.
* El servicio SSH no está escuchando en el puerto TCP 22. Para probar esto, instale un cliente telnet en el equipo local y ejecute "telnet *cloudServiceName*.cloudapp.net 22". Este paso determina si la máquina virtual permite la comunicación entrante y saliente con el punto de conexión SSH.
* El firewall local en la máquina virtual de destino tiene reglas que impiden el tráfico entrante o saliente de SSH.
* El software de detección de intrusiones o supervisión de red que se ejecuta en la máquina virtual de Azure impide las conexiones SSH.

## <a name="additional-resources"></a>Recursos adicionales
Para más información sobre cómo solucionar problemas de acceso a las aplicaciones, consulte [Solución de problemas de acceso a una aplicación que se ejecuta en una máquina virtual de Azure](./troubleshoot-app-connection.md).
