---
title: 'Generación y exportación de certificados para conexiones de punto a sitio: Linux: CLI'
description: Cree un certificado raíz autofirmado, exporte la clave pública y genere los certificados de cliente mediante la CLI de Linux (strongSwan).
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: alzam
ms.openlocfilehash: 0be5bb042649b0fe425077b5b3feb3cea728218c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89394013"
---
# <a name="generate-and-export-certificates"></a>Generación y exportación de certificados

Las conexiones de punto a sitio utilizan certificados para realizar la autenticación. En este artículo, se muestra cómo crear un certificado raíz autofirmado y generar certificados de cliente con la CLI de Linux y strongSwan. Si busca otras instrucciones de certificado, vea los artículos sobre [Powershell](vpn-gateway-certificates-point-to-site.md) o [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Para obtener información sobre cómo instalar strongSwan mediante la GUI en lugar de la CLI, vea los pasos descritos en el artículo [Configuración del cliente](point-to-site-vpn-client-configuration-azure-cert.md#install).

## <a name="install-strongswan"></a>Instalación de strongSwan

[!INCLUDE [strongSwan Install](../../includes/vpn-gateway-strongswan-install-include.md)]

## <a name="generate-and-export-certificates"></a>Generación y exportación de certificados

[!INCLUDE [strongSwan certificates](../../includes/vpn-gateway-strongswan-certificates-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Continúe con la configuración de punto a sitio para [crear e instalar archivos de configuración de cliente de VPN](point-to-site-vpn-client-configuration-azure-cert.md#linuxinstallcli).
