---
title: 'Generación y exportación de certificados para la conexión de punto a sitio: PowerShell'
titleSuffix: Azure VPN Gateway
description: Cree un certificado raíz autofirmado, exporte la clave pública y genere los certificados de cliente mediante PowerShell en Windows 10 o Windows Server 2016.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: dc9f9ca72247ac54286037475912cf6051d8d2af
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89394195"
---
# <a name="generate-and-export-certificates-for-point-to-site-using-powershell"></a>Generación y exportación de certificados para conexiones de punto a sitio con PowerShell

Las conexiones de punto a sitio utilizan certificados para realizar la autenticación. En este artículo, se muestra cómo crear un certificado raíz autofirmado y generar certificados de cliente usando PowerShell en Windows 10 o Windows Server 2016. Si busca otras instrucciones de certificado, vea los artículos sobre [certificados con Linux](vpn-gateway-certificates-point-to-site-linux.md) o [certificados con MakeCert](vpn-gateway-certificates-point-to-site-makecert.md).

Debe realizar los pasos de este artículo en un equipo que ejecute Windows 10 o Windows Server 2016. Los cmdlets de PowerShell que se usan para generar certificados forman parte del sistema operativo y no funcionan en otras versiones de Windows. El equipo con Windows 10 o Windows Server 2016 solo es necesario para generar los certificados. Una vez que se generan los certificados, puede cargarlos o instalarlos en cualquier sistema operativo cliente compatible.

Si no tiene acceso a un equipo con Windows 10 o Windows Server 2016, puede usar [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md) para generar certificados. Los certificados que genera mediante cualquiera de estos métodos pueden instalarse en cualquier sistema operativo cliente [compatible](vpn-gateway-howto-point-to-site-resource-manager-portal.md#faq).

[!INCLUDE [generate and export certificates](../../includes/vpn-gateway-generate-export-certificates-include.md)]

## <a name="install-an-exported-client-certificate"></a><a name="install"></a>Instalación de un certificado de cliente exportado

Cada cliente que se conecta a la red virtual a través de una conexión de punto a sitio requiere que haya un certificado de cliente instalado localmente.

Para instalar un certificado de cliente, consulte cómo [instalar un certificado de cliente para conexiones de punto a sitio](point-to-site-how-to-vpn-client-install-azure-cert.md).

## <a name="next-steps"></a>Pasos siguientes

Continúe con la configuración de punto a sitio.

* Para conocer los pasos del modelo de implementación de **Resource Manager**, consulte cómo [configurar una conexión de punto a sitio mediante la autenticación de certificados de Azure nativa](vpn-gateway-howto-point-to-site-resource-manager-portal.md).
* Para ver los pasos del modelo de implementación **clásica**, consulte el artículo [Configuración de una conexión VPN de punto a sitio a una red virtual mediante el Portal clásico](vpn-gateway-howto-point-to-site-classic-azure-portal.md).