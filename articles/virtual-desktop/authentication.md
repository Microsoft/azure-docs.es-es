---
title: 'Autenticación de Windows Virtual Desktop: Azure'
description: Métodos de autenticación de Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 08/27/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 8328fa9c9e32a22e8e5093aa343f333cd1914ddc
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89026141"
---
# <a name="supported-authentication-methods"></a>Métodos de autenticación admitidos

En este artículo, le mostraremos una breve descripción de los tipos de autenticación que puede usar en Windows Virtual Desktop.

## <a name="session-host-authentication"></a>Autenticación de host de sesión

Windows Virtual Desktop admite NT LAN Manager (NTLM) y Kerberos para la autenticación del host de sesión. Sin embargo, para utilizar Kerberos, el cliente debe obtener vales de seguridad de Kerberos de un servicio de Centro de distribución de claves (KDC) que se ejecute en un controlador de dominio. Para obtener vales, el cliente necesita una línea de visión directa al controlador de dominio. Para ello, puede usar la red corporativa. También puede usar una conexión VPN a la red corporativa.

Estos son los métodos de inicio de sesión admitidos actualmente:

- Cliente de escritorio de Windows
    - Nombre de usuario y contraseña
    - Tarjeta inteligente
    - Windows Hello
- Cliente de Microsoft Store
    - Nombre de usuario y contraseña
- Cliente web
    - Nombre de usuario y contraseña
- Android
    - Nombre de usuario y contraseña
- iOS
    - Nombre de usuario y contraseña
- macOS
    - Nombre de usuario y contraseña

>[!NOTE]
>La tarjeta inteligente y Windows Hello solo pueden usar Kerberos para iniciar sesión. Para iniciar la sesión con Kerberos se necesita una línea de visión al controlador de dominio.

## <a name="single-sign-on-sso"></a>Inicio de sesión único (SSO)

Windows Virtual Desktop no admite actualmente Servicios de federación de Active Directory (AD FS) (ADFS) para la autenticación o el inicio de sesión único.

La única manera de evitar que le pidan sus credenciales del host de sesión es guardarlas en el cliente. Se recomienda hacer esto solo con dispositivos seguros a fin de evitar que otros usuarios tengan acceso a los recursos.

## <a name="next-steps"></a>Pasos siguientes

¿Tiene curiosidad sobre otras formas de proteger la implementación? Consulte [Procedimientos recomendados de seguridad](security-guide.md).
