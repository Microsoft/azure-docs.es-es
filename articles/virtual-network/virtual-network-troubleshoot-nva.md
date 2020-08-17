---
title: Solución de problemas de aplicaciones virtuales de red en Azure | Microsoft Docs
description: Solución de problemas de la aplicación virtual de red (NVA) en Azure y validación de los requisitos básicos de la plataforma Azure para las configuraciones de NVA.
services: virtual-network
documentationcenter: na
author: genlin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/26/2018
ms.author: genli
ms.openlocfilehash: 112792d4ccee2be7f85e6a5a6c0caf64df9a019e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286077"
---
# <a name="network-virtual-appliance-issues-in-azure"></a>Problemas de aplicaciones virtuales de red en Azure

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Es posible que experimente problemas y errores de conectividad de máquina virtual o de VPN al utilizar la aplicación virtual de red (NVA) de terceros en Microsoft Azure. En este artículo se proporcionan los pasos básicos para ayudarle a validar los requisitos básicos de la plataforma Azure para las configuraciones de aplicación virtual de red.

El soporte técnico para las aplicaciones virtuales de red de terceros y su integración con la plataforma Azure lo proporciona el proveedor de la aplicación virtual de red.

> [!NOTE]
> Si tiene un problema de conectividad o de enrutamiento que implica una aplicación virtual de red, debería [ponerse en contacto directamente con el proveedor de la aplicación virtual de red](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="checklist-for-troubleshooting-with-nva-vendor"></a>Lista de comprobación para la solución de problemas con el proveedor de la aplicación virtual de red

- Actualizaciones de software para software de máquinas virtuales de aplicaciones virtuales de red
- Funcionalidad y configuración de la cuenta de servicio
- Rutas definidas por el usuario (UDR) en subredes de red virtual que dirigen el tráfico a la aplicación virtual de red
- Rutas definidas por el usuario (UDR) en subredes de red virtual que dirigen el tráfico de la aplicación virtual de red
- Tablas de enrutamiento y las reglas en la aplicación virtual de red (por ejemplo, de NIC1 a NIC2)
- Seguimiento de las NIC de la aplicación virtual de red para comprobar la recepción y el envío del tráfico de red
- Cuando usa una SKU estándar y direcciones IP públicas, debe haber creado un grupo de seguridad de red y una regla explícita para permitir que el tráfico se enrute a la aplicación virtual de red.

## <a name="basic-troubleshooting-steps"></a>Pasos básicos para solucionar problemas

- Comprobación de la configuración básica
- Comprobación del rendimiento de la aplicación virtual de red
- Solución de problemas avanzados de red

## <a name="check-the-minimum-configuration-requirements-for-nvas-on-azure"></a>Comprobación de los requisitos mínimos de la configuración de las aplicaciones virtuales de red en Azure

Cada aplicación virtual de red tiene requisitos básicos de configuración para funcionar correctamente en Azure. En la sección siguiente se proporcionan los pasos para comprobar estas configuraciones básicas. Para más información, [póngase en contacto con el proveedor de la aplicación virtual de red](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).

**Comprobación de si está habilitado el reenvío IP en la aplicación virtual de red**

Usar Azure Portal

1. Busque el recurso de la aplicación virtual de red en [Azure Portal](https://portal.azure.com), seleccione las redes y, después, seleccione la interfaz de red.
2. En la página Interfaz de red, seleccione Configuraciones de IP.
3. Asegúrese de que está habilitado el reenvío IP.

Uso de PowerShell

1. Abra PowerShell e inicie sesión en la cuenta de Azure.
2. Ejecute el siguiente comando (reemplace los valores entre corchetes por su información):

   ```powershell
   Get-AzNetworkInterface -ResourceGroupName <ResourceGroupName> -Name <NicName>
   ```

3. Compruebe la propiedad **EnableIPForwarding**.
4. Si el reenvío IP no está habilitado, ejecute los siguientes comandos para habilitarlo:

   ```powershell
   $nic2 = Get-AzNetworkInterface -ResourceGroupName <ResourceGroupName> -Name <NicName>
   $nic2.EnableIPForwarding = 1
   Set-AzNetworkInterface -NetworkInterface $nic2
   Execute: $nic2 #and check for an expected output:
   EnableIPForwarding   : True
   NetworkSecurityGroup : null
   ```

**Comprobación de NSG al usar la dirección IP pública de una SKU estándar** Cuando use una SKU estándar y una dirección IP pública, debe haber creado un grupo de seguridad de red y una regla explícita para permitir el tráfico a la aplicación virtual de red.

**Comprobación de si se puede enrutar el tráfico a la aplicación virtual de red**

1. En [Azure Portal](https://portal.azure.com), abra **Network Watcher**, seleccione **Próximo salto**.
2. Especifique una máquina virtual que esté configurada para redirigir el tráfico a la aplicación virtual de red y una dirección IP de destino en la cual se va a ver el próximo salto. 
3. Si la aplicación virtual de red no aparece como **próximo salto**, comprueba y actualice las tablas de rutas de Azure.

**Comprobación de si el tráfico puede llegar a la aplicación virtual de red**

1. En [Azure Portal](https://portal.azure.com), abra **Network Watcher** y, después, seleccione **Comprobación del flujo de IP**. 
2. Especifique la máquina virtual y la dirección IP de la aplicación virtual de red y, después, compruebe si el tráfico está bloqueado por algún grupo de seguridad de red (NSG).
3. Si existe una regla del grupo de seguridad de red que bloquee el tráfico, localice dicho grupo en las reglas de **seguridad eficaces** y, a continuación, actualícela para permitir el paso del tráfico. A continuación, ejecute **Comprobación del flujo de IP** de nuevo y use **Solución de problemas de conexión** para probar las comunicaciones TCP de la máquina virtual a la dirección IP interna o externa.

**Comprobación de si la aplicación virtual de red y las máquinas virtuales están escuchando el tráfico esperado**

1. Conéctese a la aplicación virtual de red mediante RDP o SSH y, después, ejecute el siguiente comando:

    Para Windows:

    ```console
   netstat -an
    ```

    Para Linux:

    ```console
   netstat -an | grep -i listen
    ```
2. Si no ve el puerto TCP que utiliza el software de la aplicación virtual de red que aparece en los resultados, debe configurar la aplicación en la aplicación virtual de red y en la máquina virtual para que escuche y responda al tráfico que llega a esos puertos. [Póngase en contacto con el proveedor de la aplicación virtual de red para obtener ayuda, según sea necesario](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).

## <a name="check-nva-performance"></a>Comprobación del rendimiento de la aplicación virtual de red

### <a name="validate-vm-cpu"></a>Validación de la CPU de la máquina virtual

Si el uso de CPU se acerca al 100 por ciento, es posible que se produzca un problema que afecte a las colocaciones de paquetes de red. La máquina virtual notifica la CPU promedio durante un intervalo de tiempo específico en Azure Portal. Durante un pico de CPU, investigue qué proceso de la máquina virtual invitada está causando la elevada CPU y lo solucione, si es posible. También es posible que tenga que cambiar el tamaño de la máquina virtual a un tamaño de SKU más grande o, para el conjunto de escalado de máquinas virtuales, aumentar el número de instancias o establecer el escalado automático en el uso de CPU. Para cada uno de estos problemas, [póngase en contacto con el proveedor de la aplicación virtual de red para obtener ayuda](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines), según sea necesario.

### <a name="validate-vm-network-statistics"></a>Validación de las estadísticas de red de la máquina virtual

Si la red de VM utiliza picos o muestra períodos de uso elevado, es posible que también tenga que aumentar el tamaño de la SKU de la máquina virtual para obtener mayores capacidades de rendimiento. También puede volver a implementar la máquina virtual con las redes aceleradas habilitadas. Para comprobar si el dispositivo virtual de red admite la característica de redes aceleradas, [póngase en contacto con el proveedor de la aplicación virtual de red](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines), según sea necesario.

## <a name="advanced-network-administrator-troubleshooting"></a>Solución de problemas del administrador de red avanzado

### <a name="capture-network-trace"></a>Captura del seguimiento de red
Capture un seguimiento de red simultáneo en la máquina virtual de origen, la aplicación virtual de red y la máquina virtual de destino mientras ejecuta **[PsPing](https://docs.microsoft.com/sysinternals/downloads/psping)** o **Nmap** y, después, detenga el seguimiento.

1. Para capturar un seguimiento de red simultáneo, ejecute el siguiente comando:

   **Para Windows**

   netsh trace start capture=yes tracefile=c:\server_IP.etl scenario=netconnection

   **Para Linux**

   sudo tcpdump -s0 -i eth0 -X -w vmtrace.cap

2. Utilice **PsPing** o **Nmap** desde la máquina virtual de origen a la máquina virtual de destino (por ejemplo: `PsPing 10.0.0.4:80` o `Nmap -p 80 10.0.0.4`).
3. Abra el seguimiento de red desde la máquina virtual de destino mediante el [Monitor de red](https://www.microsoft.com/download/details.aspx?id=4865) o tcpdump. Aplique un filtro de visualización para la IP de la máquina virtual de origen desde la que ejecutó **PsPing** o **Nmap**, como `IPv4.address==10.0.0.4 (Windows netmon)` o `tcpdump -nn -r vmtrace.cap src or dst host 10.0.0.4` (Linux).

### <a name="analyze-traces"></a>Análisis de seguimientos

Si no ve los paquetes de entrada para el seguimiento de la máquina virtual de back-end, es probable que un grupo de seguridad de red o un enrutamiento definido por el usuario esté interfiriendo o que las tablas de enrutamiento de la aplicación virtual de red sean incorrectas.

Si ve que los paquetes entran, pero no hay ninguna respuesta, puede ser necesario resolver un problema de la aplicación de la máquina virtual o del firewall. Para cada uno de estos problemas, [póngase en contacto con el proveedor de la aplicación virtual de red para obtener ayuda, según sea necesario](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).