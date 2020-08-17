---
title: 'Aplicación de TLS 1.2: servicio de registro de Azure Active Directory'
description: Retirada de la compatibilidad con TLS 1.0 y 1.1 para el servicio Registro de dispositivos de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: reference
ms.date: 07/10/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: spunukol
ms.collection: M365-identity-device-management
ms.openlocfilehash: bbfbad53fd2361752824292e0ade82f3ac231916
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095178"
---
# <a name="enforce-tls-12-for-the-azure-ad-registration-service"></a>Aplicación de TLS 1.2 para el servicio de registro de Azure AD

El servicio Registro de dispositivos de Azure Active Directory (Azure AD) se utiliza para conectar dispositivos a la nube con un id. de dispositivo. El servicio Registro de dispositivos de Azure AD admite actualmente el uso de Seguridad de la capa de transporte (TLS) 1.2 para las comunicaciones con Azure. Para garantizar la seguridad y el mejor cifrado, Microsoft recomienda deshabilitar TLS 1.0 y 1.1. En este documento se proporciona información sobre cómo asegurarse de que las máquinas usadas para completar el registro y comunicarse con el servicio Registro de dispositivos de Azure AD utilizan TLS 1.2.

El protocolo TLS, versión 1.2, es un protocolo de criptografía que está diseñado para proporcionar comunicaciones seguras. El protocolo TLS tiene como objetivo principal proporcionar privacidad e integridad de los datos. TLS ha pasado por muchas iteraciones con la versión 1.2; estas se definen en [RFC 5246 (vínculo externo)](https://tools.ietf.org/html/rfc5246).

El análisis actual de conexiones muestra un uso escaso de TLS 1.1 y 1.0, pero proporcionamos esta información para que pueda actualizar cualquier cliente o servidor afectado según sea necesario antes de que finalice la compatibilidad con TLS 1.1 y 1.0. Si usa una infraestructura local para escenarios híbridos o Servicios de federación de Active Directory (AD FS), asegúrese de que la infraestructura puede admitir tanto conexiones entrantes como salientes que usen TLS 1.2.

## <a name="update-windows-servers"></a>Actualización de servidores de Windows

En el caso de los servidores de Windows que usan el servicio Registro de dispositivos de Azure AD o actúan como servidores proxy, realice los pasos siguientes para asegurarse de que TLS 1.2 está habilitado:

> [!IMPORTANT]
> Una vez que haya actualizado el registro, debe reiniciar el servidor de Windows para que los cambios surtan efecto.

### <a name="enable-tls-12"></a>Habilitar TLS 1.2

Asegúrese de que las siguientes cadenas del registro están configuradas de la siguiente manera:

- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client
  - "DisabledByDefault"=dword:00000000
  - "Enabled"=dword:00000001
- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server
  - "DisabledByDefault"=dword:00000000
  - "Enabled"=dword:00000001
- HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319
  - "SchUseStrongCrypto"=dword:00000001

## <a name="update-non-windows-proxies"></a>Actualización de servidores proxy que no son de Windows

Todas las máquinas que actúen como servidores proxy entre dispositivos y el servicio Registro de dispositivos de Azure AD deben asegurarse de que TLS 1.2 está habilitado. Siga las instrucciones del proveedor para garantizar el soporte técnico.

## <a name="update-ad-fs-servers"></a>Actualización de servidores de AD FS

Todos los servidores de AD FS utilizados para comunicarse con el servicio Registro de dispositivos de Azure AD deben asegurarse de que TLS 1.2 está habilitado. Consulte [Administración de conjuntos de cifrado y protocolos SSL/TLS de AD FS](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs) para encontrar información sobre cómo habilitar o comprobar esta configuración.

## <a name="client-updates"></a>Actualizaciones de clientes

Dado que todas las combinaciones cliente-servidor y explorador-servidor deben usar TLS 1.2 para conectarse con el servicio Registro de dispositivos de Azure AD, puede que tenga que actualizar estos dispositivos.

Se sabe que los siguientes clientes no pueden admitir TLS 1.2. Actualice los clientes para garantizar el acceso ininterrumpido.

- Android versión 4.3 y anteriores
- Firefox versión 5.0 y anteriores
- Internet Explorer versiones 8-10 en Windows 7 y anteriores
- Internet Explorer 10 en Windows Phone 8.0
- Safari versión 6.0.4 en OS X 10.8.4 y versiones anteriores

## <a name="next-steps"></a>Pasos siguientes

[Información general de TLS/SSL (Schannel SSP)](https://docs.microsoft.com/windows-server/security/tls/tls-ssl-schannel-ssp-overview)
