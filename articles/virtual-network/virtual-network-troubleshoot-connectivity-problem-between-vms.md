---
title: Solución de problemas de conectividad entre máquinas virtuales de Azure | Microsoft Docs
description: Obtenga información sobre cómo solucionar y resolver los problemas de conectividad que puede experimentar entre VM de Azure.
services: virtual-network
documentationcenter: na
author: chadmath
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/30/2018
ms.author: genli
ms.openlocfilehash: eb94b67b026ed108f31f6cd802010577665ec0d8
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286094"
---
# <a name="troubleshooting-connectivity-problems-between-azure-vms"></a>Solución de problemas de conectividad entre máquinas virtuales de Azure

Podría experimentar problemas de conectividad entre máquinas virtuales de Azure. Este artículo proporciona pasos de solución de problemas para ayudarle a resolver este problema. 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="symptom"></a>Síntoma

No se puede conectar una máquina virtual de Azure a otra máquina virtual de Azure.

## <a name="troubleshooting-guidance"></a>Guía de solución de problemas 

1. [Compruebe si la NIC está mal configurada](#step-1-check-whether-nic-is-misconfigured)
2. [Compruebe si un NSG o una UDR bloquea el tráfico de red](#step-2-check-whether-network-traffic-is-blocked-by-nsg-or-udr)
3. [Compruebe si el firewall de la máquina virtual bloquea el tráfico de red](#step-3-check-whether-network-traffic-is-blocked-by-vm-firewall)
4. [Compruebe si la aplicación o servicio de la máquina virtual está escuchando en el puerto](#step-4-check-whether-vm-app-or-service-is-listening-on-the-port)
5. [Compruebe si el problema lo provoca SNAT](#step-5-check-whether-the-problem-is-caused-by-snat)
6. [Compruebe si las ACL bloquean el tráfico de la máquina virtual clásica](#step-6-check-whether-traffic-is-blocked-by-acls-for-the-classic-vm)
7. [Compruebe si se creó el punto de conexión de la máquina virtual clásica](#step-7-check-whether-the-endpoint-is-created-for-the-classic-vm)
8. [Intente conectarse a un recurso compartido de red de una máquina virtual](#step-8-try-to-connect-to-a-vm-network-share)
9. [Compruebe la conectividad entre redes virtuales](#step-9-check-inter-vnet-connectivity)

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas

Siga estos pasos para solucionar el problema. Después de completar cada paso, compruebe si el problema se ha resuelto. 

### <a name="step-1-check-whether-nic-is-misconfigured"></a>Paso 1: Compruebe si la NIC está mal configurada

Siga los pasos descritos en [Restablecimiento de la interfaz de red para VM de Microsoft Azure](../virtual-machines/windows/reset-network-interface.md). 

Si el problema se produce después de modificar la interfaz de red (NIC), siga estos pasos:

**Máquinas virtuales con varias NIC**

1. Agregue una NIC.
2. Corrija los problemas de la NIC incorrecta o quítela.  A continuación, vuelva a agregar la NIC.

Para más información, consulte [Incorporación de interfaces de red a máquinas virtuales o eliminación de estas](virtual-network-network-interface-vm.md).

**Máquina virtual con una única NIC** 

- [Nueva implementación de la máquina virtual en Windows](../virtual-machines/windows/redeploy-to-new-node.md)
- [Nueva implementación de la máquina virtual de Linux](../virtual-machines/linux/redeploy-to-new-node.md)

### <a name="step-2-check-whether-network-traffic-is-blocked-by-nsg-or-udr"></a>Paso 2: Compruebe si un NSG o una UDR bloquea el tráfico de red

Use [Comprobación del flujo de IP de Network Watcher](../network-watcher/network-watcher-ip-flow-verify-overview.md) y [Registro de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-overview.md) para determinar si hay un grupo de seguridad de red (NSG) o ruta definida por el usuario (UDR) que esté interfiriendo con el flujo de tráfico.

### <a name="step-3-check-whether-network-traffic-is-blocked-by-vm-firewall"></a>Paso 3: Compruebe si el firewall de la máquina virtual bloquea el tráfico de red

Deshabilite el firewall y, a continuación, pruebe el resultado. Si se resuelve el problema, compruebe la configuración de firewall y, a continuación, vuelva a habilitarlo.

### <a name="step-4-check-whether-vm-app-or-service-is-listening-on-the-port"></a>Paso 4: Compruebe si la aplicación o servicio de la máquina virtual está escuchando en el puerto

Puede usar alguno de los métodos siguientes para comprobar si la aplicación o servicio de la máquina virtual está escuchando en el puerto.

- Ejecute los comandos siguientes para comprobar si el servidor está escuchando en ese puerto.

**Máquina virtual Windows**

```console
netstat –ano
```

**Máquina virtual Linux**

```console
netstat -l
```

- Ejecute el comando **telnet** en la propia máquina virtual para probar el puerto. Si se produce un error en la prueba, la aplicación o servicio no está configurado para escuchar en ese puerto.

### <a name="step-5-check-whether-the-problem-is-caused-by-snat"></a>Paso 5: Compruebe si el problema lo provoca SNAT

En algunos escenarios, la máquina virtual se coloca detrás de una solución de equilibrio de carga que tiene una dependencia de recursos fuera de Azure. En estos casos, si tiene problemas de conexión intermitentes, el problema puede deberse al [agotamiento de los puertos SNAT](../load-balancer/load-balancer-outbound-connections.md). Para resolver el problema, cree una dirección VIP (o ILPIP para el modelo de implementación clásico) para cada máquina virtual que esté detrás del equilibrador de carga y protéjala con un NSG o una ACL. 

### <a name="step-6-check-whether-traffic-is-blocked-by-acls-for-the-classic-vm"></a>Paso 6: Compruebe si las ACL bloquean el tráfico de la máquina virtual clásica

Una lista de control de acceso (ACL) proporciona la capacidad de permitir o denegar tráfico a un punto de conexión de la máquina virtual de forma selectiva. Para más información, consulte [Administración de la ACL en un punto de conexión](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints#manage-the-acl-on-an-endpoint).

### <a name="step-7-check-whether-the-endpoint-is-created-for-the-classic-vm"></a>Paso 7: Compruebe si se creó el punto de conexión de la máquina virtual clásica

Todas las máquinas virtuales que se crean en Azure con el modelo de implementación clásico pueden comunicarse automáticamente a través de un canal de red privada con otras máquinas virtuales del mismo servicio en la nube o de la misma red virtual. Sin embargo, los equipos en otras redes virtuales necesitan puntos de conexión para dirigir el tráfico de red entrante a una máquina virtual. Para más información, consulte [Cómo configurar puntos de conexión](../virtual-machines/windows/classic/setup-endpoints.md).

### <a name="step-8-try-to-connect-to-a-vm-network-share"></a>Paso 8: Intente conectarse a un recurso compartido de red de una máquina virtual

Si no se puede conectar a un recurso compartido de red de máquina virtual, el problema puede deberse a que la NIC no está disponible en la máquina virtual. Para eliminar las NIC no disponibles, consulte [Eliminación de las NIC no disponibles](../virtual-machines/troubleshooting/reset-network-interface.md#delete-the-unavailable-nics)

### <a name="step-9-check-inter-vnet-connectivity"></a>Paso 9: Compruebe la conectividad entre redes virtuales

Use [Comprobación del flujo de IP de Network Watcher](../network-watcher/network-watcher-ip-flow-verify-overview.md) y [Registro de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-overview.md) para determinar si hay un grupo de seguridad de red (NSG) o ruta definida por el usuario (UDR) que esté interfiriendo con el flujo de tráfico. También puede comprobar la configuración entre redes virtuales [aquí](https://support.microsoft.com/en-us/help/4032151/configuring-and-validating-vnet-or-vpn-connections).

### <a name="need-help-contact-support"></a>¿Necesita ayuda? Póngase en contacto con el servicio de soporte técnico.
Si sigue necesitando ayuda, [póngase en contacto con el servicio de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.