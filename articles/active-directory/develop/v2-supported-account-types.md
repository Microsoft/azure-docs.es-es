---
title: 'Cuentas admitidas en aplicaciones (audiencia): plataforma de identidad de Microsoft'
description: Documentación conceptual sobre audiencias y tipos de cuenta admitidos en aplicaciones
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: d751e28859a3ae1104b28c76d0edfedb2a859eb4
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406520"
---
# <a name="supported-account-types"></a>Tipos de cuenta admitidos

En este artículo se explica qué tipos de cuentas (a veces denominados audiencias) se admiten en aplicaciones

<!-- This section can be in an include for many of the scenarios (SPA, Web App signing-in users, protecting a Web API, Desktop (depending on the flows), Mobile -->

## <a name="supported-accounts-types-in-microsoft-identity-platform-applications"></a>Tipos de cuentas admitidos en aplicaciones de la plataforma de identidad de Microsoft

En la nube pública de Microsoft Azure, la mayoría de los tipos de aplicaciones pueden iniciar la sesión de los usuarios con cualquier audiencia:

- Si está escribiendo una aplicación de línea de negocio, puede iniciar la sesión de los usuarios en su propia organización. Este tipo de aplicación a veces se denomina **inquilino único**.
- Si es usted un ISV, puede escribir una aplicación que inicia la sesión de los usuarios.

  - En cualquier organización. Este tipo de aplicación se denomina aplicación web **multiinquilino**. A veces leerá que inicia la sesión de los usuarios con sus cuentas profesionales o educativas.
  - Con su cuenta Microsoft personal o profesional o educativa.
  - Con solo una cuenta personal de Microsoft.
    > [!NOTE]
    > Actualmente, la plataforma de identidad de Microsoft admite cuentas personales de Microsoft solo al registrar una aplicación para **cuentas personales de Microsoft o profesionales o educativas**, y luego limita el inicio de sesión en el código de la aplicación especificando una autoridad de Azure AD, al compilar la aplicación, como `https://login.onmicrosoftonline.com/consumers`.

- Si está escribiendo una aplicación de empresa a consumidores, también puede iniciar la sesión de los usuarios con sus identidades sociales, por medio de Azure AD B2C.

## <a name="certain-authentication-flows-dont-support-all-the-account-types"></a>Ciertos flujos de autenticación no son compatibles con todos los tipos de cuenta

No es posible usar algunos tipos de cuenta con determinados flujos de autenticación. Por ejemplo, en aplicaciones de escritorio, UWP o demonio:

- Las aplicaciones de demonio solo pueden usarse con organizaciones de Azure Active Directory. No tiene sentido intentar utilizar aplicaciones de demonio para manipular las cuentas personales de Microsoft (nunca se concederá el consentimiento del administrador).  
- Solo puede usar el flujo de autenticación integrada con cuentas profesionales o educativas (en su organización o en cualquier organización). De hecho, la autenticación integrada funciona con cuentas de dominio, y requiere que las máquinas estén unidas a dominio o a Azure AD. Este flujo no tiene sentido para cuentas personales de Microsoft.
- La [concesión de contraseña del propietario del recursos](./v2-oauth-ropc.md) (nombre de usuario/contraseña) no puede utilizarse con cuentas personales de Microsoft. De hecho, las cuentas personales de Microsoft requieren que el usuario dé su consentimiento para obtener acceso a los recursos de personales en cada inicio de sesión. Por este motivo, este comportamiento no es compatible con los flujos no interactivos.
- El flujo de código de dispositivo todavía no funciona con cuentas personales de Microsoft.

## <a name="supported-account-types-in-national-clouds"></a>Tipos de cuenta admitidos en las nubes nacionales

 Las aplicaciones también pueden iniciar la sesión de los usuarios en [nubes nacionales](authentication-national-cloud.md). Sin embargo, no se admiten cuentas personales de Microsoft en estas nubes (por definición de estas nubes). Es por eso que los tipos de cuenta admitidos se reducen, para estas nubes, a su organización (inquilino único) o a cualquier organización (aplicaciones multiinquilino).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el [inquilinato en Azure Active Directory](./single-and-multi-tenant-apps.md).
- Obtenga más información sobre las [nubes nacionales](./authentication-national-cloud.md)