---
title: Ayuda para el portal Mis aplicaciones - Azure Active Directory| Microsoft Docs
description: Obtenga ayuda para iniciar sesión en el portal Mis aplicaciones y realizar tareas frecuentes.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 03/21/2019
ms.author: curtand
ms.reviewer: kasimpso
ms.custom: user-help, seo-update-azuread-jan
ms.openlocfilehash: c5d1b08944417493d175de23c7738db58f1c7bd3
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798013"
---
# <a name="troubleshoot-problems-with-the-my-apps-portal"></a>Solución de problemas relacionados con el portal Mis aplicaciones

Si tiene problemas de inicio de sesión o de uso en el portal **Mis aplicaciones**, pruebe estas sugerencias de solución de problemas antes de pedir ayuda al soporte técnico o al administrador.

## <a name="im-having-trouble-installing-the-my-apps-secure-sign-in-extension"></a>Tengo problemas para instalar la Extensión de inicio de sesión seguro de mis aplicaciones

Si tiene problemas para instalar la Extensión de inicio de sesión seguro de mis aplicaciones:

- Asegúrese de que usa un explorador compatible, incluido:

    - **Microsoft Edge.** Con Windows 10 Anniversary Edition o posterior.

    - **Google Chrome.** Con Windows 7 o posterior, y con Mac OS X o posterior.

    - **Mozilla Firefox 26.0 o posterior.** Con Windows XP SP2 o posterior, y con Mac OS X 10.6 posterior.

    - **Internet Explorer 11.** Con Windows 7 o posterior (compatibilidad limitada).

- Asegúrese de que la configuración de la extensión del explorador esté activada.

- Intente reiniciar el explorador e inicie sesión de nuevo en el portal **Mis aplicaciones**.

- Intente borrar las cookies del explorador y, a continuación, reinicie e inicie sesión de nuevo en el portal **Mis aplicaciones**.

## <a name="i-cant-sign-in-to-the-my-apps-portal"></a>No puedo iniciar sesión en el portal **Mis aplicaciones**

Si tiene problemas para iniciar sesión en el portal **Mis aplicaciones**, puede intentar el siguiente procedimiento:

- Asegúrese de que está utilizando la dirección URL correcta. Debe ser https://myapps.microsoft.com o una página personalizada para su organización, como https://myapps.microsoft.com/contoso.com.

- Asegúrese de que la contraseña es correcta y de que no ha expirado. Para obtener más información, consulte [Restablecimiento de la contraseña profesional o educativa](active-directory-passwords-update-your-own-password.md).

- Asegúrese de que la información de comprobación esté actualizada y sea precisa. Para obtener más información, consulte [¿Qué significa Azure Multi-factor Authentication para mí?](./multi-factor-authentication-end-user-first-time.md) o [Cambio de métodos de información de seguridad e información](./security-info-setup-auth-app.md).

- Agregue la dirección URL del portal **Mis aplicaciones** a la configuración **Propiedades de Internet > Seguridad > Sitios de confianza**.

- Borre la caché del explorador e intente iniciar sesión de nuevo.

## <a name="my-password-isnt-working"></a>Mi contraseña no funciona

Si ha olvidado la contraseña, no ha recibido una de su organización, desea cambiarla o no puede acceder a su cuenta, consulte el artículo de [ayuda con la contraseña de Azure AD](active-directory-passwords-update-your-own-password.md).

## <a name="i-want-to-be-able-to-reset-my-own-password"></a>Quiero poder restablecer mi contraseña

Para poder restablecer su contraseña, el administrador debe activar primero la característica para su organización y, a continuación, debe actualizar y comprobar los métodos de verificación necesarios. Para obtener más información sobre cómo actualizar los métodos de comprobación, consulte [Registro para el autoservicio de restablecimiento de contraseña](active-directory-passwords-reset-register.md).

## <a name="im-getting-an-access-denied-message-when-i-start-an-app"></a>Obtengo un mensaje de acceso denegado al iniciar una aplicación

Si recibe un mensaje de **acceso denegado** mensaje después de iniciar una aplicación desde el portal **Mis aplicaciones**, puede intentar realizar el siguiente procedimiento:

- Asegúrese de que ha instalado la [Extensión de inicio de sesión seguro de Mis aplicaciones](my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension) y que esté usando un [explorador compatible](my-apps-portal-end-user-access.md#supported-browsers).

- Asegúrese de que está usando la dirección URL correcta para la aplicación y que la dirección URL se encuentra en su lista **Propiedades de Internet > Seguridad > Sitios de confianza**.

- Asegúrese de que la contraseña es correcta y de que no ha expirado. Para obtener más información, consulte [Restablecimiento de la contraseña profesional o educativa](active-directory-passwords-update-your-own-password.md).

- Asegúrese de que la información de comprobación esté actualizada y sea precisa. Para obtener más información, consulte [¿Qué significa Azure Multi-factor Authentication para mí?](./multi-factor-authentication-end-user-first-time.md) o [Cambio de métodos de información de seguridad e información](./security-info-setup-auth-app.md).

- Borre la caché del explorador e intente iniciar sesión de nuevo.

Si después de probar todo esto no puede acceder a la aplicación, debe ponerse en contacto con el departamento de soporte técnico de su organización para obtener ayuda.

## <a name="next-steps"></a>Pasos siguientes

Después de iniciar sesión en el portal **Mis aplicaciones**, también puede actualizar su perfil e información de cuenta, la información del grupo y la información de revisión de acceso (si tiene permiso).

- [Acceder y usar las aplicaciones del portal Aplicaciones](my-apps-portal-end-user-access.md).

- [Modificación de información de perfil](my-apps-portal-end-user-update-profile.md).

- [Ver y actualizar la información relacionada con grupos](my-apps-portal-end-user-groups.md).

- [Realizar sus propias revisiones de acceso](my-apps-portal-end-user-access-reviews.md).