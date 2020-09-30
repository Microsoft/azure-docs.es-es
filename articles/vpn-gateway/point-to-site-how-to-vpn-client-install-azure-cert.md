---
title: 'Azure VPN Gateway: instalación de un certificado de cliente de punto a sitio'
description: 'Instale un certificado de cliente para la autenticación de certificados de conexiones de punto a sitio: Windows, Mac y Linux.'
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 8c4d37f67d89ad1b7973a85fb6c2a9f7e818a028
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421669"
---
# <a name="install-client-certificates-for-p2s-certificate-authentication-connections"></a>Instalación de un certificado de cliente para conexiones de punto a sitio con autenticación de certificados

Todos los clientes que se conectan a una red virtual mediante la autenticación de certificados de Azure de conexiones de punto a sitio requieren un certificado de cliente. Este artículo le ayuda a instalar un certificado de cliente que se usa para la autenticación cuando se conecta a una red virtual mediante una conexión de punto a sitio.

## <a name="acquire-a-client-certificate"></a><a name="generate"></a>Adquisición de un certificado de cliente

Independientemente del sistema operativo cliente al que quiera conectarse, debe tener siempre un certificado de cliente. Puede generar un certificado de cliente desde un certificado raíz que se ha generado mediante una solución de una entidad de certificación empresarial o desde un certificado raíz autofirmado. Vea las instrucciones de [PowerShell](vpn-gateway-certificates-point-to-site.md), [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md) o [Linux](vpn-gateway-certificates-point-to-site-linux.md) para saber los pasos para generar un certificado de cliente. 

## <a name="windows"></a><a name="installwin"></a>Windows

[!INCLUDE [Install on Windows](../../includes/vpn-gateway-certificates-install-client-cert-include.md)]

## <a name="mac"></a><a name="installmac"></a>Mac

>[!NOTE]
>Los clientes VPN de Mac son compatibles únicamente con el modelo de implementación de Resource Manager. No son compatibles con el modelo de implementación clásica.
>
>

[!INCLUDE [Install on Mac](../../includes/vpn-gateway-certificates-install-mac-client-cert-include.md)]

## <a name="linux"></a><a name="installlinux"></a>Linux

El certificado de cliente de Linux se instala en el cliente como parte de la configuración del cliente. Vea [Client configuration - Linux](point-to-site-vpn-client-configuration-azure-cert.md#linuxinstallcli) (Configuración de cliente: Linux) para obtener instrucciones.

## <a name="next-steps"></a>Pasos siguientes

Continúe con los pasos de configuración de punto a sitio en [Creación e instalación de los archivos de configuración de cliente de VPN](point-to-site-vpn-client-configuration-azure-cert.md).
