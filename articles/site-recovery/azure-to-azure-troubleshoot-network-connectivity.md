---
title: Solución de problemas de la recuperación ante desastres de Azure a Azure con Azure Site Recovery
description: Solución de problemas de conectividad en la recuperación ante desastres de una máquina virtual de Azure
author: sideeksh
manager: rochakm
ms.topic: how-to
ms.date: 04/06/2020
ms.openlocfilehash: afa2cbdb7b0703f9fc0b419442570744c6fefae1
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89049696"
---
# <a name="troubleshoot-azure-to-azure-vm-network-connectivity-issues"></a>Solución de problemas de conectividad de red de máquinas virtuales de Azure en Azure

En este artículo se describen los problemas comunes relacionados con la conectividad de red al replicar y recuperar máquinas virtuales (VM) de Azure de una región a otra. Para más información sobre los requisitos de red, consulte los [requisitos de conectividad para la replicación de máquinas virtuales de Azure](azure-to-azure-about-networking.md).

Para que la replicación de Site Recovery funcione, la máquina virtual debe disponer de conectividad saliente a direcciones URL o intervalos IP específicos. Si la máquina virtual está detrás de un firewall o usa reglas de grupo de seguridad de red (NSG) para controlar la conectividad saliente, puede encontrarse alguno de estos problemas.

| **Nombre**                  | **Comercial**                               | **Gobierno**                                 | **Descripción** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Storage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net`              | Se requiere para que los datos se puedan escribir en la cuenta de almacenamiento de la caché en la región de origen de la máquina virtual. Si conoce todas las cuentas de almacenamiento en caché para las máquinas virtuales, puede usar una lista de permitidos para las direcciones URL de la cuenta de almacenamiento específica. Por ejemplo, `cache1.blob.core.windows.net` y `cache2.blob.core.windows.net` en lugar de `*.blob.core.windows.net`. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Se requiere para la autorización y la autenticación de las direcciones URL del servicio Site Recovery. |
| Replicación               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`   | Se requiere para la comunicación del servicio Site Recovery desde la máquina virtual. Puede usar la _dirección IP de Site Recovery_ correspondiente si el proxy del firewall es compatible con las direcciones IP. |
| Azure Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Se requiere para que se puedan escribir datos de supervisión y diagnóstico de Site Recovery desde la máquina virtual. Puede usar la _Dirección IP de supervisión de Site Recovery_ correspondiente si el proxy del firewall es compatible con las direcciones IP. |

## <a name="outbound-connectivity-for-site-recovery-urls-or-ip-ranges-error-code-151037-or-151072"></a>Conectividad saliente para direcciones URL o intervalos IP de Site Recovery (código de error 151037 o 151072)

### <a name="issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151195"></a>Problema 1: no se pudo registrar la máquina virtual de Azure en Site Recovery (151195)

#### <a name="possible-cause"></a>Causa posible

No se puede establecer una conexión con los puntos de conexión de Site Recovery debido a un error de resolución del Sistema de nombres de dominio (DNS). Este problema es más común durante la reprotección cuando se ha conmutado por error la máquina virtual pero el servidor DNS no es accesible desde la región de recuperación ante desastres (DR).

#### <a name="resolution"></a>Solución

Si utiliza un DNS personalizado, asegúrese de que el servidor DNS es accesible desde la región de recuperación ante desastres.

Para comprobar si la máquina virtual usa una configuración de DNS personalizada:

1. Abra **Máquinas virtuales** y seleccione la máquina virtual.
1. Vaya a la **Configuración** de las máquinas virtuales y seleccione **Redes**.
1. En **Red virtual/subred**, seleccione el vínculo para abrir la página de recursos de la red virtual.
1. Vaya a **Configuración** y seleccione **Servidores DNS**.

Intente acceder al servidor DNS desde la máquina virtual. Si no puede acceder al servidor DNS, intente que esté accesible conmutando por error el servidor DNS o creando la línea del sitio entre la red de recuperación ante desastres y DNS.

  :::image type="content" source="./media/azure-to-azure-troubleshoot-errors/custom_dns.png" alt-text="com-error":::

### <a name="issue-2-site-recovery-configuration-failed-151196"></a>Problema 2: error de configuración de Site Recovery (151196)

> [!NOTE]
> Si las máquinas virtuales están detrás de un equilibrador de carga interno **estándar**, de manera predeterminada, no tendría acceso a las direcciones IP de Office 365 como `login.microsoftonline.com`. Cámbielo al tipo de equilibrador de carga interno **básico** o cree un acceso saliente, tal como se menciona en el artículo [Configurar el equilibrio de carga y las reglas de salida en Standard Load Balancer mediante la CLI de Azure](../load-balancer/quickstart-load-balancer-standard-public-cli.md?tabs=option-1-create-load-balancer-standard#create-outbound-rule-configuration).

#### <a name="possible-cause"></a>Causa posible

No se puede establecer una conexión con los puntos de conexión de autenticación e identidad IP4 de Office 365.

#### <a name="resolution"></a>Solución

- Azure Site Recovery debe tener acceso a los intervalos IP de Office 365 para la autenticación.
- Si utiliza un proxy que actúa como firewall o reglas de grupos de seguridad de red (NSG) de Azure para controlar la conectividad de salida de la red en la máquina virtual, no olvide permitir la comunicación con los intervalos IP de Office 365. Cree una regla de grupos de seguridad de red basada en la [etiqueta de servicio de Azure Active Directory (Azure AD)](../virtual-network/security-overview.md#service-tags) que permite el acceso a todas las direcciones IP correspondientes a Azure AD.
- Si se agregan en el futuro nuevas direcciones a Azure AD, deberá crear nuevas reglas de NSG.

### <a name="example-nsg-configuration"></a>Configuración de NSG de ejemplo

En este ejemplo se muestra cómo configurar reglas de NSG para la replicación de una máquina virtual.

- Si usa reglas de NSG para controlar la conectividad de salida, utilice reglas para **permitir el tráfico HTTPS de salida** al puerto 443 para todos los intervalos IP necesarios.
- En el ejemplo se supone que la ubicación de origen de la máquina virtual es **Este de EE. UU.** y la ubicación de destino es **Centro de EE. UU**.

#### <a name="nsg-rules---east-us"></a>Reglas de NSG: este de EE. UU.

1. Cree una regla de seguridad para el tráfico HTTPS de salida para el grupo de seguridad de red, tal como se muestra en la captura de pantalla siguiente. En este ejemplo se usa la **etiqueta de servicio de destino**: _Storage.EastUS_ y los **intervalos de puertos de destino**: _443_.

     :::image type="content" source="./media/azure-to-azure-about-networking/storage-tag.png" alt-text="storage-tag":::

1. Cree una regla de seguridad para el tráfico HTTPS de salida para el grupo de seguridad de red, tal como se muestra en la captura de pantalla siguiente. En este ejemplo se usa la **etiqueta de servicio de destino**: _AzureActiveDirectory_ y los **intervalos de puertos de destino**: _443_.

     :::image type="content" source="./media/azure-to-azure-about-networking/aad-tag.png" alt-text="aad-tag":::

1. De forma similar a las reglas de seguridad anteriores, cree una regla de seguridad HTTPS (443) de salida para ""EventHub.CentralUS" en el NSG que corresponda a la ubicación de destino. Esto permite el acceso a la supervisión de Site Recovery.
1. Cree una regla de seguridad HTTPS (443) de salida para "AzureSiteRecovery" en el NSG. Esto permite el acceso al servicio Site Recovery en cualquier región.

#### <a name="nsg-rules---central-us"></a>Reglas de NSG: centro de EE. UU.

En este ejemplo, estas reglas de NSG son necesarias para que la replicación se pueda habilitar de la región de destino a la región de origen con posterioridad a la conmutación por error:

1. Cree una regla de seguridad para el tráfico HTTPS de salida para _Storage.CentralUS_:

   - **Etiqueta de servicio de destino**: _Storage.CentralUS_
   - **Intervalos de puertos de destino**: _443_

1. Cree una regla de seguridad para el tráfico HTTPS de salida para _AzureActiveDirectory_.

   - **Etiqueta de servicio de destino**: _AzureActiveDirectory_
   - **Intervalos de puertos de destino**: _443_

1. De forma similar a las reglas de seguridad anteriores, cree una regla de seguridad HTTPS (443) de salida para "EventHub.EastUS" en el NSG que corresponda a la ubicación de origen. Esto permite el acceso a la supervisión de Site Recovery.
1. Cree una regla de seguridad HTTPS (443) de salida para "AzureSiteRecovery" en el NSG. Esto permite el acceso al servicio Site Recovery en cualquier región.

### <a name="issue-3-site-recovery-configuration-failed-151197"></a>Problema 3: error de configuración de Site Recovery (151197)

#### <a name="possible-cause"></a>Causa posible

No se puede establecer una conexión con los puntos de conexión del servicio Azure Site Recovery.

#### <a name="resolution"></a>Solución

Azure Site Recovery necesita tener acceso a [intervalos de IP de Site Recovery](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags) en función de la región. Asegúrese de que los intervalos IP están accesibles desde la máquina virtual.

### <a name="issue-4-azure-to-azure-replication-failed-when-the-network-traffic-goes-through-on-premises-proxy-server-151072"></a>Problema 4: Error de replicación de Azure a Azure cuando el tráfico de red pasa por el servidor proxy local (151072)

#### <a name="possible-cause"></a>Causa posible

La configuración de proxy personalizada no es válida y el agente del servicio Mobility de Azure Site Recovery no detectó automáticamente la configuración de proxy de Internet Explorer (IE).

#### <a name="resolution"></a>Solución

1. El agente del servicio Mobility detecta la configuración de proxy de IE en Windows y de `/etc/environment` en Linux.
1. Si prefiere configurar el proxy solo para el servicio Mobility de Azure Site Recovery, puede proporcionar los detalles del proxy en _ProxyInfo.conf_ que se encuentra en:

   - **Linux**: `/usr/local/InMage/config/`
   - **Windows**: `C:\ProgramData\Microsoft Azure Site Recovery\Config`

1. _ProxyInfo.conf_ debe tener la configuración de proxy en el siguiente formato _INI_:

   ```plaintext
   [proxy]
   Address=http://1.2.3.4
   Port=567
   ```

> [!NOTE]
> El agente del servicio Mobility de Azure Site Recovery solo admite **servidores proxy no autenticados**.

### <a name="fix-the-problem"></a>Corrección del problema

Para permitir [las direcciones URL necesarias](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) o los [intervalos IP necesarios](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags), siga los pasos del [documento de instrucciones para redes](./azure-to-azure-about-networking.md).

## <a name="next-steps"></a>Pasos siguientes

[Replicación de máquinas virtuales de Azure en otra región de Azure](azure-to-azure-how-to-enable-replication.md)