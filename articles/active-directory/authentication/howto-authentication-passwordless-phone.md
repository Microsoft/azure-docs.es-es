---
title: 'Inicio de sesión sin contraseña con la aplicación Microsoft Authenticator: Azure Active Directory'
description: Habilitar el inicio de sesión sin contraseña en Azure AD con la aplicación Microsoft Authenticator (versión preliminar)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: librown
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5022dfc5811f3db59774a46a7034ff14c4dcf4a8
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87428023"
---
# <a name="enable-passwordless-sign-in-with-the-microsoft-authenticator-app-preview"></a>Habilitar el inicio de sesión sin contraseña en Azure AD con la aplicación Microsoft Authenticator (versión preliminar)

La aplicación Microsoft Authenticator se puede utilizar para iniciar sesión en cualquier cuenta de Azure AD sin utilizar una contraseña. Similar a la tecnología de [Windows Hello para empresas](/windows/security/identity-protection/hello-for-business/hello-identity-verification), Microsoft Authenticator usa la autenticación basada en claves para habilitar una credencial de usuario que está asociada a un dispositivo y usa un sistema biométrico o un PIN. Este método de autenticación se puede usar en cualquier plataforma de dispositivos, incluidos dispositivos móviles, y con cualquier aplicación o sitio web que se integre con las bibliotecas de autenticación de Microsoft. 

![Ejemplo de un inicio de sesión de explorador que solicita al usuario que apruebe el inicio de sesión](./media/howto-authentication-passwordless-phone/phone-sign-in-microsoft-authenticator-app.png)

En lugar de ver la petición de una contraseña después de escribir un nombre de usuario, una persona que ha habilitado el inicio de sesión por teléfono desde la aplicación Microsoft Authenticator verá un mensaje que le solicita pulsar un número en la aplicación. En la aplicación, el usuario debe hacer coincidir el número, elegir Aprobar y, después, proporcionar el PIN o información biométrica para que la autenticación se complete.

> [!NOTE]
> Esta funcionalidad se incluye en la aplicación Microsoft Authenticator desde marzo de 2017, por lo que existe la posibilidad de que, cuando la directiva esté habilitada para un directorio, los usuarios puedan encontrar este flujo inmediatamente y vean un mensaje de error si no se han habilitado mediante la directiva. Tenga esto en cuenta y prepare a los usuarios para este cambio.

## <a name="prerequisites"></a>Requisitos previos

- Azure Multi-Factor Authentication, con notificaciones de inserción permitidas como método de comprobación 
- Versión más reciente de Microsoft Authenticator instalada en dispositivos que ejecutan iOS 8.0 o superior o Android 6.0 o superior.

> [!NOTE]
> Si habilitó la anterior versión preliminar de inicio de sesión sin contraseña de la aplicación Microsoft Authenticator mediante Azure AD PowerShell, se habilitó para todo el directorio. Si habilita con este nuevo método, reemplazará a la directiva de PowerShell. Se recomienda habilitar para todos los usuarios del inquilino mediante los nuevos métodos de autenticación; de lo contrario, los usuarios que no estén en la nueva directiva ya no podrán iniciar sesión sin contraseña. 

## <a name="enable-passwordless-authentication-methods"></a>Habilitar métodos de autenticación sin contraseña

### <a name="enable-the-combined-registration-experience"></a>Habilitar la experiencia de registro combinado

Las características de registro de los métodos de autenticación sin contraseña se basan en la característica de registro combinado. Siga los pasos del artículo [Habilitar el registro de información de seguridad combinado](howto-registration-mfa-sspr-combined.md) para habilitar el registro combinado.

### <a name="enable-passwordless-phone-sign-in-authentication-methods"></a>Habilitar métodos de autenticación de inicio de sesión en el teléfono sin contraseña

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con la cuenta de **administrador global**.
1. Busque y seleccione *Azure Active Directory*. Seleccione **Seguridad** > **Métodos de autenticación** > **Directiva de métodos de autenticación (versión preliminar)** .
1. En **Inicio de sesión telefónico sin contraseña**, elija las opciones siguientes:
   1. **Habilitar**: Sí o No
   1. **Destino**: Todos los usuarios o Seleccionar usuarios
1. **Guarde** para establecer la nueva directiva.

## <a name="user-registration-and-management-of-microsoft-authenticator-app"></a>Registro de usuario y administración de la aplicación Microsoft Authenticator

1. Vaya a [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo).
1. Inicie sesión si aún no lo ha hecho.
1. Agregue una aplicación de autenticación; para ello, haga clic en **Agregar método**, seleccione **Aplicación de autenticación** y haga clic en **Agregar**.
1. Siga las instrucciones para instalar y configurar la aplicación Microsoft Authenticator en el dispositivo.
1. Haga clic en **Listo** para completar el flujo de configuración de la aplicación de autenticación para MFA. 
1. En **Microsoft Authenticator**, seleccione **Habilitar inicio de sesión en el teléfono** en el menú desplegable de la cuenta.
1. Siga las instrucciones de la aplicación para terminar de registrarse para el inicio de sesión en el teléfono sin contraseña. 

Las organizaciones pueden dirigir a sus usuarios al artículo [Inicio de sesión con el teléfono, no con la contraseña](../user-help/microsoft-authenticator-app-phone-signin-faq.md) para obtener más ayuda para la configuración de la aplicación Microsoft Authenticator y habilitar el inicio de sesión en el teléfono. Para aplicar esta configuración, puede que tenga que cerrar sesión y volver a iniciarla en el inquilino. 

## <a name="sign-in-with-passwordless-credential"></a>Iniciar sesión con credenciales sin contraseña

En la versión preliminar pública, no hay ninguna manera para exigir a los usuarios que creen o utilicen esta nueva credencial. Un usuario solo se encontrará con un inicio de sesión sin contraseña una vez que un administrador haya habilitado su inquilino **y** el usuario haya actualizado la aplicación Microsoft Authenticator para habilitar el inicio de sesión en el teléfono.

Después de escribir el nombre de usuario en la web y seleccionar **Siguiente**, los usuarios verán un número y se les pedirá en su aplicación Microsoft Authenticator que seleccionen el número correcto para autenticarse, en lugar de usar su contraseña. 

![Ejemplo de inicio de sesión en el explorador con la aplicación Microsoft Authenticator](./media/howto-authentication-passwordless-phone/web-sign-in-microsoft-authenticator-app.png)

## <a name="known-issues"></a>Problemas conocidos

### <a name="user-is-not-enabled-by-policy-but-still-has-passwordless-phone-sign-in-method-in-microsoft-authenticator"></a>El usuario no está habilitado por la directiva pero todavía tiene el método de inicio de sesión en el teléfono sin contraseña en Microsoft Authenticator

Es posible que un usuario haya creado en algún momento una credencial de inicio de sesión en el teléfono sin contraseña en su aplicación Microsoft Authenticator actual o en un dispositivo anterior. Una vez que un administrador habilita la directiva de método de autenticación para el inicio de sesión en el teléfono sin contraseña, cualquier usuario con una credencial registrada comenzará a experimentar la nueva solicitud de inicio de sesión, independientemente de si se ha habilitado para usar la directiva o no. Si el usuario no tiene permiso de la directiva para usar las credenciales, verá un error después de completar el flujo de autenticación. 

El administrador puede optar por permitir que el usuario use el inicio de sesión en el teléfono sin contraseña o bien el usuario deberá quitar el método. Si el usuario ya no tiene el dispositivo registrado, puede ir a [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo) y quitarlo. Si todavía está usando el autenticador para MFA, puede **deshabilitar el inicio de sesión en el teléfono** en Microsoft Authenticator.  

### <a name="ad-fs-integration"></a>Integración de AD FS

Cuando un usuario ha habilitado la credencial sin contraseña de Microsoft Authenticator, la autenticación predeterminada para ese usuario siempre será enviar una notificación para su aprobación. Esta lógica evita que los usuarios de un inquilino híbrido se dirijan a AD FS para la comprobación de inicios de sesión sin que el usuario realice un paso adicional para hacer clic en "Use su contraseña en su lugar". En este proceso también se omitirán las directivas de acceso condicional del entorno local y los flujos de autenticación de paso a través. 

Si un usuario tiene pendiente una comprobación de inicio de sesión en el teléfono sin contraseña sin contestar e intenta volver a iniciar sesión, es posible que se dirija al usuario a AD FS para que escriba una contraseña en su lugar.  

### <a name="azure-mfa-server"></a>Servidor de Azure MFA

Los usuarios finales que están habilitados para MFA mediante un servidor de Azure MFA local de la organización pueden seguir creando y usando una credencial de inicio de sesión único telefónica sin contraseña. Si el usuario intenta actualizar varias instalaciones (más de 5) de Microsoft Authenticator con la credencial, este cambio puede dar lugar a un error.  

### <a name="device-registration"></a>Registro de dispositivos

Uno de los requisitos previos para crear esta nueva credencial segura es que el dispositivo, donde está instalada la aplicación Microsoft Authenticator, también debe estar registrado en el inquilino de Azure AD para un usuario individual. Debido a las actuales restricciones del registro de dispositivos, solo se puede registrar un dispositivo en un inquilino. Este límite significa que solo se puede habilitar una cuenta profesional o educativa en la aplicación Microsoft Authenticator para el inicio de sesión telefónico.

### <a name="intune-mobile-application-management"></a>Administración de aplicaciones móviles de Intune 

Los usuarios finales que estén sujetos a una directiva que requiera la administración de aplicaciones móviles (MAM) no pueden registrar la credencial sin contraseña en la aplicación Microsoft Authenticator. 

> [!NOTE]
> El registro de dispositivos no es lo mismo que la administración de dispositivos o "MDM". Solo asocia un identificador de dispositivo y un identificador de usuario en el directorio de Azure AD.  

## <a name="next-steps"></a>Pasos siguientes

[Obtener información sobre cómo funciona la autenticación con contraseñas](concept-authentication-passwordless.md)

[Obtenga información sobre el registro de dispositivos](../devices/overview.md#getting-devices-in-azure-ad)

[Más información sobre Azure Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md)
