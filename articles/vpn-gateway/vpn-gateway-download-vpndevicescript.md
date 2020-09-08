---
title: Descarga de scripts de configuración de dispositivos VPN para conexiones VPN S2S
description: Este artículo lo guiará a la hora de descargar scripts de configuración de dispositivos VPN para conexiones VPN S2S con Azure VPN Gateway usando Azure Resource Manager.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 70d9a8069979a299ad3031de37c525438ab0159d
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89394608"
---
# <a name="download-vpn-device-configuration-scripts-for-s2s-vpn-connections"></a>Descarga de scripts de configuración de dispositivos VPN para conexiones VPN S2S

Este artículo lo guiará a la hora de descargar scripts de configuración de dispositivos VPN para conexiones VPN S2S con Azure VPN Gateway usando Azure Resource Manager. En el siguiente diagrama se muestra el proceso de forma resumida.

![download-script](./media/vpn-gateway-download-vpndevicescript/downloaddevicescript.png)

Los siguientes dispositivos tienen scripts disponibles:

[!INCLUDE [scripts](../../includes/vpn-gateway-device-configuration-scripts.md)]

## <a name="about-vpn-device-configuration-scripts"></a><a name="about"></a>Acerca de los scripts de configuración de dispositivos VPN

Una conexión VPN entre locales consta de una puerta de enlace de VPN de Azure, un dispositivo VPN local y un túnel VPN S2S de IPsec que conecta los dos. El flujo de trabajo típico incluye los siguientes pasos:

1. Crear y configurar una puerta de enlace VPN de Azure (puerta de enlace de red virtual)
2. Creación y configuración de una puerta de enlace de red local de Azure que representa la red local y el dispositivo VPN
3. Crear y configurar una conexión VPN de Azure entre la puerta de enlace de VPN de Azure y la puerta de enlace de red local
4. Configurar el dispositivo VPN local representado por la puerta de enlace de red local para establecer el túnel VPN S2S real con la puerta de enlace de VPN de Azure

Puede completar los pasos del 1 al 3 con Azure [Portal](vpn-gateway-howto-site-to-site-resource-manager-portal.md), [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md) o la [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md). El último paso implica configurar los dispositivos VPN locales fuera de Azure. Esta característica permite descargar un script de configuración para el dispositivo VPN con los valores correspondientes de la puerta de enlace VPN de Azure, la red virtual, los prefijos de direcciones de red local, ñas propiedades de conexión VPN, etc., ya rellenos. Puede usar el script como punto de partida, o bien aplicar el script directamente en los dispositivos VPN local a través de la consola de configuración.

> [!IMPORTANT]
> * La sintaxis de cada script de configuración de dispositivos VPN es diferente y depende en gran medida de los modelos y las versiones de firmware. Preste especial atención a la información de la versión y el modelo del dispositivo con las plantillas disponibles.
> * Algunos valores de parámetro deben ser únicos en el dispositivo y no se pueden determinar sin tener acceso al dispositivo. Los scripts de configuración generados por Azure rellenan previamente estos valores, pero debe asegurarse de que los valores proporcionados sean válidos en el dispositivo. Por ejemplo:
>    * Números de interfaces
>    * Números de listas de control de acceso
>    * Números o nombres de directivas, etc.
> * Busque la palabra clave "**REPLACE**" incrustada en el script para buscar los parámetros que debe comprobar antes de aplicar el script.
> * Algunas plantillas incluyen una sección "**CLEANUP**" que puede aplicar para eliminar las configuraciones. Las secciones de limpieza están comentadas de forma predeterminada.

## <a name="download-the-configuration-script-from-azure-portal"></a>Descarga del script de configuración de Azure Portal

Cree una puerta de enlace de VPN de Azure, una puerta de enlace de red local y un recurso de conexión que conecte los dos. Este asistente lo guiará a través de los pasos siguientes:

* [Creación de una conexión de sitio a sitio mediante Azure Portal](vpn-gateway-howto-site-to-site-resource-manager-portal.md)

Una vez creado el recurso de conexión, siga estas instrucciones para descargar los scripts de configuración de dispositivos VPN:

1. Desde un explorador, vaya a [Azure Portal](https://portal.azure.com) y, si fuera necesario, inicie sesión con su cuenta de Azure.
2. Vaya al recurso de conexión que creó. Puede ver la lista de todos los recursos de conexión haciendo clic en "Todos los servicios" y, luego en "REDES" y "Conexiones".

    ![connection-list](./media/vpn-gateway-download-vpndevicescript/connectionlist.png)

3. Haga clic en la conexión que quiere configurar.

    ![connection-overview](./media/vpn-gateway-download-vpndevicescript/connectionoverview.png)

4. Haga clic en el vínculo "Descargar configuración" resaltado en rojo en la página Información general sobre la conexión; se abrirá la página "Descargar configuración".

    ![download-script-1](./media/vpn-gateway-download-vpndevicescript/downloadscript-1.png)

5. Seleccione la versión del firmware y la familia de modelos del dispositivo VPN; luego, haga clic en el botón "Descargar configuración".

    ![download66-script-2](./media/vpn-gateway-download-vpndevicescript/downloadscript-2.PNG)

6. Se le pedirá que guarde el script descargado (un archivo de texto) desde el explorador.
7. Una vez descargado el script de configuración, ábralo con un editor de texto y busque la palabra clave "REPLACE" para identificar y examinar los parámetros que puede que necesiten reemplazarse.

    ![edit-script](./media/vpn-gateway-download-vpndevicescript/editscript.png)

## <a name="download-the-configuration-script-using-azure-powershell"></a>Descarga del script de configuración con Azure PowerShell



También puede descargar el script de configuración con Azure PowerShell, tal como se muestra en el ejemplo siguiente:

```azurepowershell-interactive
$RG          = "TestRG1"
$GWName      = "VNet1GW"
$Connection  = "VNet1toSite1"

# List the available VPN device models and versions
Get-AzVirtualNetworkGatewaySupportedVpnDevice -Name $GWName -ResourceGroupName $RG

# Download the configuration script for the connection
Get-AzVirtualNetworkGatewayConnectionVpnDeviceConfigScript -Name $Connection -ResourceGroupName $RG -DeviceVendor Juniper -DeviceFamily Juniper_SRX_GA -FirmwareVersion Juniper_SRX_12.x_GA
```

## <a name="apply-the-configuration-script-to-your-vpn-device"></a>Aplicación del script de configuración al dispositivo VPN

Después de haber descargado y validado el script de configuración, el siguiente paso es aplicar el script en el dispositivo VPN. El procedimiento real varía en función de las marcas y los modelos de dispositivos VPN. Consulte los manuales de uso o las páginas de instrucciones de sus dispositivos VPN.

## <a name="next-steps"></a>Pasos siguientes

Continúe configurado la [conexión de sitio a sitio](vpn-gateway-howto-site-to-site-resource-manager-portal.md).
