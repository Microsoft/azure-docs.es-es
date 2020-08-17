---
title: Solución de problemas de acceso de aplicaciones en una máquina virtual en Azure | Microsoft Docs
description: Use estos pasos detallados de solución de problemas para aislar problemas de conexión a aplicaciones que se ejecutan en máquinas virtuales en Azure.
services: virtual-machines
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
keywords: no se puede iniciar la aplicación, el programa no se abre, puerto de escucha bloqueado, no se puede iniciar el programa, puerto de escucha bloqueado
ms.assetid: b9ff7cd0-0c5d-4c3c-a6be-3ac47abf31ba
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: a2fe1b8bdc80a5265add22bd3602050034471fe2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87036444"
---
# <a name="troubleshoot-application-connectivity-issues-on-virtual-machines-in-azure"></a>Solución de problemas de conectividad de aplicaciones en una máquina virtual en Azure

Hay varios motivos que le pueden impedir iniciar o conectar con una aplicación que se ejecuta en una máquina virtual de Azure. Entre ellos se incluyen situaciones como que la aplicación no se esté ejecutando o no esté a la escucha en los puertos esperados, que el puerto de escucha esté bloqueado o que las reglas de redes no estén pasando el tráfico correctamente a la aplicación. En este artículo se describe un enfoque metódico para buscar y corregir el problema.

Si tiene problemas para conectarse a la máquina virtual con RDP o SSH, consulte en primer lugar uno de los siguientes artículos:

* [Solución de problemas de conexiones del Escritorio remoto a una máquina virtual de Azure con Windows](troubleshoot-rdp-connection.md)
* [Solución de problemas de conexiones de Secure Shell (SSH) en una máquina virtual de Azure basada en Linux](troubleshoot-ssh-connection.md).

Si necesita más ayuda en cualquier momento con este artículo, puede ponerse en contacto con los expertos de Azure en [los foros de MSDN Azure y de desbordamiento de pila](https://azure.microsoft.com/support/forums/). Como alternativa, también puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**.

## <a name="quick-start-troubleshooting-steps"></a>Pasos de inicio rápido para solucionar problemas
Si tiene problemas para conectarse a una aplicación, intente los siguientes pasos de solución de problemas generales. Después de cada paso, vuelva a intentar conectarse a la aplicación:

* Reiniciar la máquina virtual
* Volver a crear el punto de conexión / reglas de firewall / reglas de grupo de seguridad de red (NSG)
  * [Modelo de Resource Manager: administración de grupos de seguridad de red](../../virtual-network/manage-network-security-group.md)
  * [Modelo clásico: administración de puntos de conexión de Cloud Services](../../cloud-services/cloud-services-enable-communication-role-instances.md)
* Conectarse desde otra ubicación, como una red virtual de Azure diferente
* Volver a implementar la máquina virtual
  * [Nueva implementación de la máquina virtual en Windows](redeploy-to-new-node-windows.md)
  * [Nueva implementación de la máquina virtual de Linux](redeploy-to-new-node-linux.md)
* Volver a crear la máquina virtual

Para obtener más información, consulte [Solución de problemas con la conectividad del punto de conexión (RDP, SSH, HTTP u otros errores)](https://social.msdn.microsoft.com/Forums/azure/en-US/538a8f18-7c1f-4d6e-b81c-70c00e25c93d/troubleshooting-endpoint-connectivity-rdpsshhttp-etc-failures?forum=WAVirtualMachinesforWindows).

## <a name="detailed-troubleshooting-overview"></a>Visión detallada de la solución de problemas
Hay cuatro áreas principales para solucionar el acceso de una aplicación que se ejecuta en una máquina virtual de Azure.

![solución de problemas no se puede iniciar la aplicación](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access1.png)

1. La aplicación que se ejecuta en la máquina virtual de Azure.
   * ¿La aplicación se ejecuta correctamente?
2. La máquina virtual de Azure.
   * ¿La VM se ejecuta correctamente y responde a las solicitudes?
3. Puntos de conexión de red de Azure.
   * Puntos de conexión de servicios en la nube para máquinas virtuales en el modelo de implementación clásica.
   * Grupos de seguridad de red y reglas NAT de entrada para las máquinas virtuales en el modelo de implementación de Resource Manager.
   * ¿El tráfico fluye entre los usuarios y la máquina virtual o la aplicación en los puertos esperados?
4. El dispositivo perimetral de Internet.
   * ¿Hay instauradas reglas de firewall que impidan que el tráfico fluya correctamente?

Para los equipos cliente que tienen acceso a la aplicación a través de una conexión ExpressRoute o de VPN de sitio a sitio, las áreas principales que pueden causar problemas son la aplicación y la máquina virtual de Azure.

Para determinar el origen del problema y su corrección, siga estos pasos.

## <a name="step-1-access-application-from-target-vm"></a>Paso 1: Acceso a la aplicación desde la máquina virtual de destino
Intente obtener acceso a la aplicación con el programa cliente adecuado desde la VM en la que se ejecuta. Use el nombre de host local, la dirección IP local o la dirección de bucle invertido (127.0.0.1).

![iniciar la aplicación directamente desde la máquina virtual](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access2.png)

Por ejemplo, si la aplicación es un servidor web, abra un explorador en la VM e intente obtener acceso a una página web hospedada en ella.

Si puede tener acceso a la aplicación, vaya al [paso 2](#step2).

Si no se puede acceder a la aplicación, compruebe la siguiente configuración:

* La aplicación se ejecuta en la máquina virtual de destino.
* La aplicación está escuchando en los puertos TCP y UDP esperados.

En máquinas virtuales basadas en Windows y Linux, use el comando **netstat -a** para mostrar los puertos de escucha activos. Examine la salida para los puertos esperados en el que debe escuchar su aplicación. Reinicie la aplicación o configúrela para que use los puertos esperados según sea necesario e intente de nuevo el acceso a la aplicación de forma local.

## <a name="step-2-access-application-from-another-vm-in-the-same-virtual-network"></a><a id="step2"></a>Paso 2: Acceso a la aplicación desde otra máquina virtual de la misma red virtual
Intente obtener acceso a la aplicación desde otra VM (que esté en la misma red virtual) con el nombre de host de la VM o con su dirección IP de proveedor, privada o pública asignada por Azure. Para las máquinas virtuales creadas con el modelo de implementación clásica, no use la dirección IP pública del servicio en la nube.

![iniciar la aplicación desde una máquina virtual diferente](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access3.png)

Por ejemplo, si la aplicación es un servidor web, intente obtener acceso a una página web desde un explorador en una VM diferente de la misma red virtual.

Si puede tener acceso a la aplicación, vaya al [paso 3](#step3).

Si no se puede acceder a la aplicación, compruebe la siguiente configuración:

* El firewall del host de la VM de destino permite el tráfico de la solicitud entrante y el tráfico de la respuesta saliente.
* El software de detección de intrusiones o de supervisión de red que se ejecuta en la VM de destino permite el tráfico.
* Los puntos de conexión de Cloud Services o los grupos de seguridad de red permiten el tráfico:
  * [Modelo clásico: administración de puntos de conexión de Cloud Services](../../cloud-services/cloud-services-enable-communication-role-instances.md)
  * [Modelo de Resource Manager: administración de grupos de seguridad de red](../../virtual-network/manage-network-security-group.md)
* Un componente independiente que se ejecuta en la VM en la ruta de acceso entre la VM de prueba y su VM (como un equilibrador de carga o un firewall) permite el tráfico.

En una máquina virtual basada en Windows, use el Firewall de Windows con seguridad avanzada para determinar si las reglas de firewall excluyen el tráfico entrante y saliente de la aplicación.

## <a name="step-3-access-application-from-outside-the-virtual-network"></a><a id="step3"></a>Paso 3: Acceso a la aplicación desde fuera de la red virtual
Intente acceder a la aplicación desde una máquina de fuera de la red virtual, como la máquina virtual en la que se está ejecutando la aplicación. Use una red diferente como equipo cliente de origen.

![iniciar la aplicación desde un equipo situado fuera de la red virtual](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access4.png)

Por ejemplo, si la aplicación es un servidor web, intente tener acceso a la página web desde un explorador de un equipo que no esté en la red virtual.

Si no se puede acceder a la aplicación, compruebe la siguiente configuración:

* Para las máquinas virtuales creadas con el modelo de implementación clásica:
  
  * Compruebe que la configuración del punto de conexión de la máquina virtual permita el tráfico entrante, sobre todo el protocolo (TCP o UDP) y los números de puerto público y privado.
  * Compruebe que las listas de control de acceso (ACL) en el punto de conexión no impidan el tráfico entrante desde Internet.
  * Para más información, consulte [Configuración de puntos de conexión en una máquina virtual](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints).
* Para las máquinas virtuales creadas con el modelo de implementación de Resource Manager:
  
  * Compruebe que la configuración de la regla NAT de entrada de la máquina virtual permita el tráfico entrante, sobre todo el protocolo (TCP o UDP) y los números de puerto público y privado.
  * Compruebe que los grupos de seguridad de red permitan la solicitud entrante y el tráfico de respuesta saliente.
  * Para más información, consulte [¿Qué es un grupo de seguridad de red?](../../virtual-network/security-overview.md)

Si la máquina virtual o el extremo es un miembro de un conjunto con equilibrio de carga:

* Compruebe que el protocolo de sondeo (TCP o UDP) y el número de puerto son correctos.
* Si el protocolo de sondeo y el puerto es diferente del protocolo y puerto del conjunto con equilibrio de carga:
  * Compruebe que la aplicación está escuchando el protocolo de sondeo (TCP o UDP) y el número de puerto (use **netstat –a** en la máquina virtual de destino).
  * Compruebe que el firewall del host de la máquina virtual de destino permite la solicitud de sondeo entrante y el tráfico de respuesta de sondeo saliente.

Si puede tener acceso a la aplicación, asegúrese de que el dispositivo perimetral de Internet permita:

* El tráfico de solicitud saliente de aplicaciones desde el equipo cliente hasta la máquina virtual de Azure.
* El tráfico de respuesta de aplicación entrante desde la máquina virtual de Azure.

## <a name="step-4-if-you-cannot-access-the-application-use-ip-verify-to-check-the-settings"></a>Paso 4 Si no puede obtener acceso a la aplicación, use la comprobación de IP para comprobar la configuración. 

Para más información, consulte [Información general sobre la supervisión de red de Azure](../../network-watcher/network-watcher-monitoring-overview.md). 

## <a name="additional-resources"></a>Recursos adicionales
[Solución de problemas de conexiones del Escritorio remoto a una máquina virtual de Azure con Windows](troubleshoot-rdp-connection.md)

[Solución de problemas de conexiones de Secure Shell (SSH) en una máquina virtual de Azure basada en Linux](troubleshoot-ssh-connection.md)
