---
title: 'Inicio de sesión con clave de seguridad sin contraseña (versión preliminar): Azure Active Directory'
description: Habilitar el inicio de sesión con clave de seguridad sin contraseña para Azure AD mediante llaves de seguridad FIDO2 (versión preliminar)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 02/12/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: librown, aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9f17a634a9bc3b410eec8ed6e868d454dc9ecee6
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88116604"
---
# <a name="enable-passwordless-security-key-sign-in-preview"></a>Habilitar el inicio de sesión con clave de seguridad sin contraseña (versión preliminar)

En el caso de las empresas que usan las contraseñas hoy en día y tienen un entorno de PC compartido, las claves de seguridad proporcionan una manera perfecta para que los trabajadores se autentiquen sin escribir un nombre de usuario o una contraseña. Las claves de seguridad proporcionan productividad mejorada para los trabajadores y tienen una mejor seguridad.

Este documento se centra en la habilitación de la autenticación sin contraseña basada en claves de seguridad. Al final de este artículo, será capaz de iniciar sesión en aplicaciones basadas en Web con su cuenta de Azure AD mediante una llave de seguridad FIDO2.

> [!NOTE]
> Las claves de seguridad FIDO2 son una característica en versión preliminar pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="requirements"></a>Requisitos

- [Azure Multi-Factor Authentication](howto-mfa-getstarted.md)
- Habilitar el [registro de información de seguridad combinado (versión preliminar)](concept-registration-mfa-sspr-combined.md)
- [Claves de seguridad FIDO2](concept-authentication-passwordless.md#fido2-security-keys) compatibles
- WebAuthN requiere Windows 10 versión 1809 o superior**

Para usar claves de seguridad para iniciar sesión en servicios y aplicaciones web, debe tener un explorador que admita el protocolo WebAuthN. Entre ellas se incluyen Microsoft Edge, Chrome, Firefox y Safari.

## <a name="prepare-devices-for-preview"></a>Preparar dispositivos para la versión preliminar

Los dispositivos unidos a Azure AD que se usarán para la prueba piloto deben ejecutar Windows 10 versión 1809 o superior. La mejor experiencia se logra con Windows 10 versión 1903 o superior.

Los dispositivos unidos a Azure AD híbrido deben ejecutar Windows 10 Insider Build 18945 o una versión más reciente.

## <a name="enable-passwordless-authentication-method"></a>Habilitar métodos de autenticación sin contraseña

### <a name="enable-the-combined-registration-experience"></a>Habilitar la experiencia de registro combinado

Las características de registro de los métodos de autenticación sin contraseña se basan en la característica de registro combinado. Siga los pasos del artículo [Habilitar el registro de información de seguridad combinado](howto-registration-mfa-sspr-combined.md) para habilitar el registro combinado.

### <a name="enable-fido2-security-key-method"></a>Habilitar el método de llaves de seguridad FIDO2

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a **Azure Active Directory** > **Seguridad** > **Métodos de autenticación** > **Directiva de métodos de autenticación (versión preliminar)** .
1. En el método **Llave de seguridad FIDO2**, elija las opciones siguientes:
   1. **Habilitar**: Sí o No
   1. **Destino**: Todos los usuarios o Seleccionar usuarios
1. **Guarde** la configuración.

## <a name="user-registration-and-management-of-fido2-security-keys"></a>Registro de usuario y administración de claves de seguridad FIDO2

1. Vaya a [https://myprofile.microsoft.com](https://myprofile.microsoft.com).
1. Inicie sesión si aún no lo ha hecho.
1. Haga clic en **Información de seguridad**.
   1. Si el usuario ya tiene al menos un método de Azure Multi-Factor Authentication registrado, puede registrar de inmediato una clave de seguridad FIDO2.
   1. Si no tiene registrado al menos un método de Azure Multi-Factor Authentication, debe agregar uno.
1. Agregue una llave de seguridad FIDO2 al hacer clic en **Agregar método** y seleccionar **Clave de seguridad**.
1. Elija **Dispositivo USB** o **Dispositivo NFC**.
1. Tenga preparada la llave y seleccione **Siguiente**.
1. Aparece un cuadro donde se le pide al usuario que cree o escriba un PIN para la clave de seguridad y luego que realice el gesto necesario para la clave, ya sea biométrico o toque.
1. Se le devuelve al usuario la experiencia de registro combinado y se le pide que proporcione un nombre descriptivo para la llave, de modo que pueda identificar cuál es si tiene varios. Haga clic en **Next**.
1. Haga clic en **Listo** para finalizar el proceso.

## <a name="sign-in-with-passwordless-credential"></a>Iniciar sesión con credenciales sin contraseña

En el ejemplo siguiente, un usuario ya ha aprovisionado su clave de seguridad FIDO2. El usuario puede optar por iniciar sesión en Internet con su llave de seguridad FIDO2 dentro del explorador compatible en Windows 10 versión 1809 o posterior.

![Inicio de sesión de clave de seguridad en Microsoft Edge](./media/howto-authentication-passwordless-security-key/fido2-windows-10-1903-edge-sign-in.png)

## <a name="troubleshooting-and-feedback"></a>Solución de problemas y comentarios

Si quiere compartir comentarios o detectar problemas mientras usa la versión preliminar de esta característica, compártalos mediante la aplicación Centro de opiniones sobre Windows. Para ello, realice los pasos siguientes:

1. Abra el **Centro de opiniones** y asegúrese de que ha iniciado sesión.
1. Envíe los comentarios bajo la categorización siguiente:
   - Categoría: Seguridad y privacidad
   - Subcategoría: FIDO
1. Para capturar registros, use la opción **Recreate my Problem** (Recrear mi problema).

## <a name="known-issues"></a>Problemas conocidos

### <a name="security-key-provisioning"></a>Aprovisionamiento de claves de seguridad

El aprovisionamiento y desaprovisionamiento de administrador de claves de seguridad no está disponible en la versión preliminar pública.

### <a name="upn-changes"></a>Cambios de UPN

Estamos trabajando en la admisión de una característica que permite el cambio de UPN en dispositivos unidos a Azure AD híbridos y dispositivos unidos a Azure AD. Si cambia el UPN de un usuario, ya no puede modificar las llaves de seguridad FIDO2 en consecuencia. La solución consiste en restablecer el dispositivo y que el usuario vuelva a registrarse.

## <a name="next-steps"></a>Pasos siguientes

[Inicio de sesión de Windows 10 con llave de seguridad FIDO2](howto-authentication-passwordless-security-key-windows.md)

[Habilitación de la autenticación de FIDO2 en recursos locales](howto-authentication-passwordless-security-key-on-premises.md)

[Más información sobre el registro de dispositivos](../devices/overview.md)

[Más información sobre Azure Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md)
