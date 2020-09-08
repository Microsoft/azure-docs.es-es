---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 07/31/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: fc2393cfe87e2639ce40e66e6053d4d430518719
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87515313"
---
## <a name="1-download-the-file"></a>1. Descargue el archivo

Ejecute los comandos siguientes: Copie la dirección URL del resultado en el explorador para descargar el archivo zip del perfil.

```azurepowershell-interactive
$profile = New-AzVpnClientConfiguration -ResourceGroupName AADAuth -Name AADauthGW -AuthenticationMethod "EapTls"
   
$PROFILE.VpnProfileSASUrl
```

## <a name="2-extract-the-zip-file"></a>2. Extraiga el archivo ZIP

Extraiga el archivo ZIP. El archivo contiene las siguientes carpetas:

* AzureVPN
* Genérico
* OpenVPN (si ha habilitado la **configuración de autenticación** de OpenVPN y **certificado de Azure** en la puerta de enlace). En el caso de VPN Gateway, consulte [Creación de un inquilino](../articles/vpn-gateway/openvpn-azure-ad-tenant.md). En el caso de Virtual WAN, consulte [Creación de un inquilino: VWAN](../articles/virtual-wan/openvpn-azure-ad-tenant.md).

## <a name="3-retrieve-information"></a>3. Recuperar información

En la carpeta **AzureVPN**, desplácese hasta el archivo ***azurevpnconfig. XML*** y ábralo con el Bloc de notas. Tome nota del texto entre las etiquetas siguientes.

```
<audience>          </audience>
<issuer>            </issuer>
<tennant>           </tennant>
<fqdn>              </fqdn>
<serversecret>      </serversecret>
```

## <a name="profile-details"></a>Detalles de perfil

Cuando agregue una conexión, use la información que recopiló en el paso anterior para la página de detalles del perfil. Los campos corresponden con la siguiente información:

   * **Audiencia:** Identifica el recurso de destinatario al que está destinado el token
   * **Emisor:** Identifica el Servicio de token de seguridad (STS) que emitió el token, así como el inquilino de Azure AD
   * **Inquilino:** Contiene un identificador único e inmutable del inquilino de directorio que emitió el token
   * **FQDN:** Nombre de dominio completo (FQDN) en la Azure VPN gateway
   * **ServerSecret:** La clave previamente compartida de VPN Gateway

## <a name="folder-contents"></a>Contenido de la carpeta

* La **carpeta genérica** contiene el certificado de servidor público y el archivo VpnSettings.xml. El archivo VpnSettings.xml contiene la información necesaria para configurar un cliente genérico.

* El archivo zip descargado también puede contener carpetas **WindowsAmd64** y **WindowsX86**. Estas carpetas contienen el instalador de SSTP y IKEv2 para clientes de Windows. Necesita derechos de administrador en el cliente para instalarlos.
