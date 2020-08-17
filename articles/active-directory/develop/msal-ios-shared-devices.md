---
title: Modo de dispositivo compartido para dispositivos iOS
titleSuffix: Microsoft identity platform | Azure
description: Obtenga información acerca de cómo habilitar el modo de dispositivo compartido para permitir que los trabajadores de primera línea compartan un dispositivo iOS.
services: active-directory
author: brandwe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/31/2020
ms.author: brandwe
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: 5abb6852e267fa7d7277e0e0674a4099343e14ca
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88114921"
---
# <a name="shared-device-mode-for-ios-devices"></a>Modo de dispositivo compartido para dispositivos iOS

> [!NOTE]
> Esta característica está en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Los trabajadores de primera línea, como los asociados comerciales, los miembros de la tripulación de vuelos y los trabajadores de servicios de campo, suelen usar un dispositivo móvil compartido para realizar su trabajo. Estos dispositivos compartidos pueden presentar riesgos para la seguridad si los usuarios comparten sus contraseñas o PIN, intencionalmente o no, para acceder a los datos de clientes y empresariales en el dispositivo compartido.

El modo de dispositivo compartido le permite configurar un dispositivo iOS 13 o superior para que los empleados puedan compartirlo de forma más fácil y segura. Los empleados pueden iniciar sesión y acceder a la información de los clientes rápidamente. Cuando terminan su turno o tarea, pueden cerrar sesión en el dispositivo y estará inmediatamente listo para que lo use el siguiente empleado.

El modo de dispositivo compartido también proporciona la administración del dispositivo respaldada por la identidad de Microsoft.

Esta característica usa la [aplicación Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) para administrar los usuarios en el dispositivo y para distribuir el complemento [Microsoft Enterprise SSO para dispositivos Apple](apple-sso-plugin.md).

## <a name="create-a-shared-device-mode-app"></a>Creación de una aplicación con modo de dispositivo compartido

Para crear una aplicación en modo de dispositivo compartido, los desarrolladores y los administradores de dispositivos en la nube trabajan juntos:

1. Los **desarrolladores de aplicaciones** escriben una aplicación de una sola cuenta (las aplicaciones de varias cuentas no se admiten en el modo de dispositivo compartido) y escriben código para controlar aspectos como el cierre de sesión del dispositivo compartido.

1. Los **administradores de dispositivos** preparan el dispositivo para que se comparta mediante un proveedor de administración de dispositivos móviles (MDM) como Microsoft Intune para administrar los dispositivos de la organización. MDM envía la aplicación Microsoft Authenticator a los dispositivos y activa el "modo compartido" para cada dispositivo mediante una actualización de perfil en el dispositivo. Esta configuración de modo compartido es lo que cambia el comportamiento de las aplicaciones compatibles en el dispositivo. Esta configuración del proveedor de MDM establece el modo de dispositivo compartido para el dispositivo y habilita el complemento [Microsoft Enterprise SSO para dispositivos Apple](apple-sso-plugin.md) que es necesario para el modo de dispositivo compartido.

1. [**Necesario solo durante la versión preliminar pública**] Un usuario con el rol de [administrador de dispositivos en la nube](../users-groups-roles/directory-assign-admin-roles.md#cloud-device-administrator) debe iniciar la [aplicación Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) y unir el dispositivo a la organización.

    Para configurar la pertenencia de los roles de la organización en Azure Portal: **Azure Active Directory** > **Roles y administradores**  > **Administrador de dispositivos en la nube**

Las secciones siguientes le ayudarán a actualizar la aplicación para que admita el modo de dispositivo compartido.

## <a name="use-intune-to-enable-shared-device-mode--sso-extension"></a>Uso de Intune para habilitar el modo de dispositivo compartido y la extensión de SSO

> [!NOTE]
> El paso siguiente solo es necesario durante la versión preliminar pública.

El dispositivo debe estar configurado para admitir el modo de dispositivo compartido. Debe tener iOS 13+ instalado y estar inscrito en MDM. La configuración de MDM también debe habilitar el [complemento Microsoft Enterprise SSO para dispositivos Apple](apple-sso-plugin.md). Para más información sobre las extensiones de SSO, consulte el [vídeo de Apple](https://developer.apple.com/videos/play/tech-talks/301/).

1. En el portal de configuración de Intune, indique al dispositivo que habilite el [complemento Microsoft Enterprise SSO para dispositivos Apple](apple-sso-plugin.md) con la configuración siguiente:

    - **Tipo**: Redirigir
    - **Identificador de extensión**: com.microsoft.azureauthenticator.ssoextension
    - **Identificador del equipo**: SGGM6D27TK
    - **Direcciones URL**: https://login.microsoftonline.com
    - Datos adicionales que se deben configurar:
      - Clave: sharedDeviceMode
      - Escriba:  Boolean
      - Valor: True

    Para más información sobre la configuración con Intune, consulte la [documentación de configuración de Intune](/intune/configuration/ios-device-features-settings).

1. A continuación, configure MDM para que inserte la aplicación Microsoft Authenticator en el dispositivo mediante un perfil de MDM.

    Establezca las siguientes opciones de configuración para activar el modo de dispositivo compartido:

    - Configuración 1:
      - Clave: sharedDeviceMode
      - Escriba:  Boolean
      - Valor: True

## <a name="modify-your-ios-application-to-support-shared-device-mode"></a>Modificación de la aplicación iOS para admitir el modo de dispositivo compartido

Los usuarios dependen de usted para asegurarse de que sus datos no se filtran a otro usuario. En las secciones siguientes se proporcionan señales útiles para indicar a la aplicación que se ha producido un cambio que se debe controlar.

Usted es responsable de comprobar el estado del usuario en el dispositivo cada vez que se usa la aplicación, eliminando los datos del usuario anterior. Esto incluye cuando se vuelve a cargar desde segundo plano en modo multitarea.

En un cambio de usuario, debe asegurarse de que se eliminan los datos del usuario anterior y se eliminan los datos almacenados en caché que se muestran en la aplicación. Le recomendamos que usted y su empresa realicen un proceso de revisión de seguridad después de actualizar la aplicación para que admita el modo de dispositivo compartido.

### <a name="detect-shared-device-mode"></a>Detección del modo de dispositivo compartido

La detección del modo de dispositivo compartido es importante para la aplicación. Muchas aplicaciones requerirán un cambio en la experiencia del usuario (UX) cuando la aplicación se use en un dispositivo compartido. Por ejemplo, la aplicación podría tener una característica de "registro", que no es adecuada para un trabajador de primera línea porque es probable que ya tenga una cuenta. Es posible que también quiera agregar seguridad adicional a la administración de datos de la aplicación si está en modo de dispositivo compartido.

Use la API `getDeviceInformationWithParameters:completionBlock:` de `MSALPublicClientApplication` para determinar si una aplicación se ejecuta en un dispositivo que está en modo de dispositivo compartido.

Los fragmentos de código siguientes muestran ejemplos de uso de la API `getDeviceInformationWithParameters:completionBlock:`.

#### <a name="swift"></a>Swift

```swift
application.getDeviceInformation(with: nil, completionBlock: { (deviceInformation, error) in

    guard let deviceInfo = deviceInformation else {
        return
    }

    let isSharedDevice = deviceInfo.deviceMode == .shared
    // Change your app UX if needed
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
[application getDeviceInformationWithParameters:nil
                                completionBlock:^(MSALDeviceInformation * _Nullable deviceInformation, NSError * _Nullable error)
{
    if (!deviceInformation)
    {
        return;
    }

    BOOL isSharedDevice = deviceInformation.deviceMode == MSALDeviceModeShared;
    // Change your app UX if needed
}];
```

### <a name="get-the-signed-in-user-and-determine-if-a-user-has-changed-on-the-device"></a>Obtención del usuario que ha iniciado sesión y comprobación de si un usuario ha cambiado en el dispositivo

Otra parte importante de la compatibilidad con el modo de dispositivo compartido es determinar el estado del usuario en el dispositivo y eliminar los datos de la aplicación si un usuario ha cambiado o si no hay ningún usuario en el dispositivo. Usted es responsable de garantizar que los datos no se filtran a otro usuario.

Puede usar la API `getCurrentAccountWithParameters:completionBlock:` para consultar la cuenta que actualmente ha iniciado sesión en el dispositivo.

#### <a name="swift"></a>Swift

```swift
let msalParameters = MSALParameters()
msalParameters.completionBlockQueue = DispatchQueue.main

application.getCurrentAccount(with: msalParameters, completionBlock: { (currentAccount, previousAccount, error) in

    // currentAccount is the currently signed in account
    // previousAccount is the previously signed in account if any
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALParameters *parameters = [MSALParameters new];
parameters.completionBlockQueue = dispatch_get_main_queue();

[application getCurrentAccountWithParameters:parameters
                             completionBlock:^(MSALAccount * _Nullable account, MSALAccount * _Nullable previousAccount, NSError * _Nullable error)
{
    // currentAccount is the currently signed in account
    // previousAccount is the previously signed in account if any
}];
```

### <a name="globally-sign-in-a-user"></a>Inicio de sesión de un usuario de forma global

Cuando un dispositivo se configura como un dispositivo compartido, la aplicación puede llamar a la API `acquireTokenWithParameters:completionBlock:` para iniciar sesión en la cuenta. La cuenta estará disponible globalmente para todas las aplicaciones válidas en el dispositivo una vez que la primera aplicación inicie sesión en dicha cuenta.

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALInteractiveTokenParameters *parameters = [[MSALInteractiveTokenParameters alloc] initWithScopes:@[@"api://myapi/scope"] webviewParameters:[self msalTestWebViewParameters]];

parameters.loginHint = self.loginHintTextField.text;

[application acquireTokenWithParameters:parameters completionBlock:completionBlock];
```

### <a name="globally-sign-out-a-user"></a>Cierre de sesión de un usuario de forma global

El siguiente código elimina la cuenta con la sesión iniciada y borra los tokens almacenados en caché no solo de la aplicación, sino también del dispositivo que está en modo de dispositivo compartido. Sin embargo, no elimina los *datos* de la aplicación. Debe eliminar los datos de la aplicación, así como borrar cualquier dato almacenado en caché que la aplicación pueda mostrar al usuario.

#### <a name="clear-browser-state"></a>Borrado del estado del explorador

> [!NOTE]
> El paso siguiente solo es necesario durante la versión preliminar pública.

En esta versión preliminar pública, el [complemento Microsoft Enterprise SSO para dispositivos de Apple](apple-sso-plugin.md) solo borra el estado de las aplicaciones. No borra el estado en el explorador Safari. Se recomienda borrar manualmente la sesión del explorador para asegurarse de que no quedan trazas del estado del usuario. Puede usar la propiedad opcional `signoutFromBrowser` que se muestra a continuación para borrar las cookies. Esto hará que el explorador se inicie brevemente en el dispositivo.

#### <a name="swift"></a>Swift

```swift
let account = .... /* account retrieved above */

let signoutParameters = MSALSignoutParameters(webviewParameters: self.webViewParamaters!)
signoutParameters.signoutFromBrowser = true // Only needed for Public Preview.

application.signout(with: account, signoutParameters: signoutParameters, completionBlock: {(success, error) in

    if let error = error {
        // Signout failed
        return
    }

    // Sign out completed successfully
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALAccount *account = ... /* account retrieved above */;

MSALSignoutParameters *signoutParameters = [[MSALSignoutParameters alloc] initWithWebviewParameters:webViewParameters];
signoutParameters.signoutFromBrowser = YES; // Only needed for Public Preview.

[application signoutWithAccount:account signoutParameters:signoutParameters completionBlock:^(BOOL success, NSError * _Nullable error)
{
    if (!success)
    {
        // Signout failed
        return;
    }

    // Sign out completed successfully
}];
```

## <a name="next-steps"></a>Pasos siguientes

Para ver el modo de dispositivo compartido en acción, el ejemplo de código siguiente de GitHub incluye un ejemplo de ejecución de una aplicación para trabajadores de primera línea en un dispositivo iOS en modo de dispositivo compartido:

[Ejemplo de Microsoft Graph API con MSAL iOS Swift](https://github.com/Azure-Samples/ms-identity-mobile-apple-swift-objc)