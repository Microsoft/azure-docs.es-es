---
title: Uso del modo de dispositivo compartido con MSAL para Android | Azure
description: Aprenda a preparar un dispositivo Android para que se ejecute en modo compartido y ejecute una aplicación para trabajadores de primera línea.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 1/15/2020
ms.author: hahamil
ms.reviewer: brandwe
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: f49a5703b19a76095c8eafe358742b442725d3d0
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118253"
---
# <a name="tutorial-use-shared-device-mode-in-your-android-application"></a>Tutorial: Uso del modo de dispositivo compartido en la aplicación Android

> [!NOTE]
> Esta característica está en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="developer-guide"></a>Guía del desarrollador

En esta guía se proporcionan instrucciones para desarrolladores que permiten implementar el modo de dispositivo compartido en una aplicación Android mediante la Biblioteca de autenticación de Microsoft (MSAL). Consulte el [tutorial de MSAL para Android](./tutorial-v2-android.md) para ver cómo integrar MSAL con su aplicación Android, iniciar sesión con un usuario, llamar a Microsoft Graph y cerrar la sesión de un usuario.

### <a name="download-the-sample"></a>Descarga del ejemplo

Clone la [aplicación de ejemplo](https://github.com/Azure-Samples/ms-identity-android-java/) de GitHub. El ejemplo puede funcionar en [modo de una o de varias cuentas](./single-multi-account.md).

### <a name="add-the-msal-sdk-to-your-local-maven-repository"></a>Adición del SDK de MSAL al repositorio local de Maven

Si no está usando la aplicación de ejemplo, agregue la biblioteca MSAL como una dependencia en el archivo build.gradle, como se muestra a continuación:

```gradle
dependencies{
  implementation 'com.microsoft.identity.client.msal:1.0.+'
}
```

### <a name="configure-your-app-to-use-shared-device-mode"></a>Configuración de la aplicación para que use el modo de dispositivo compartido

Consulte la [documentación de configuración](./msal-configuration.md) para más información sobre cómo configurar el archivo de configuración.

Establezca `"shared_device_mode_supported"` como `true` en el archivo de configuración de MSAL.

Es posible que no tenga previsto admitir el modo de varias cuentas. Este podría ser el caso si no va a usar un dispositivo compartido y el usuario puede iniciar sesión en la aplicación con más de una cuenta al mismo tiempo. Si es así, establezca `"account_mode"` en `"SINGLE"`. De esta forma, se garantiza que la aplicación siempre obtendrá `ISingleAccountPublicClientApplication` y simplifica considerablemente la integración de MSAL. El valor predeterminado de `"account_mode"` es `"MULTIPLE"`, por lo que es importante cambiar este valor en el archivo de configuración si utiliza el modo de `"single account"`.

Este es un ejemplo del archivo auth_config.json incluido en el directorio **app**>**main**>**res**>**raw** de la aplicación de ejemplo:

```json
{
 "client_id":"Client ID after app registration at https://aka.ms/MobileAppReg",
 "authorization_user_agent":"DEFAULT",
 "redirect_uri":"Redirect URI after app registration at https://aka.ms/MobileAppReg",
 "account_mode":"SINGLE",
 "broker_redirect_uri_registered": true,
 "shared_device_mode_supported": true,
 "authorities":[
  {
   "type":"AAD",
   "audience":{
     "type": "AzureADandPersonalMicrosoftAccount",
     "tenant_id":"common"
   }
  }
 ]
}
```

### <a name="detect-shared-device-mode"></a>Detección del modo de dispositivo compartido

El modo de dispositivo compartido le permite configurar dispositivos Android para que los compartan varios empleados, a la vez que ofrece la administración respaldada del dispositivo por parte de Microsoft Identity. Los empleados pueden iniciar sesión en sus dispositivos y acceder a la información sobre los clientes rápidamente. Cuando terminan su turno o su tarea, pueden cerrar la sesión de todas las aplicaciones en el dispositivo compartido con un solo clic y el dispositivo estará inmediatamente listo para que lo use el siguiente empleado.

Use `isSharedDevice()` para determinar si una aplicación se está ejecutando en un dispositivo que está en modo de dispositivo compartido. La aplicación podría usar esta marca para determinar si debe modificar la experiencia de usuario según corresponda.

Este es un fragmento de código que muestra cómo puede usar `isSharedDevice()`.  Procede de la clase `SingleAccountModeFragment` de la aplicación de ejemplo:

```Java
deviceModeTextView.setText(mSingleAccountApp.isSharedDevice() ?"Shared" :"Non-Shared");
```

### <a name="initialize-the-publicclientapplication-object"></a>Inicialización del objeto PublicClientApplication

Si establece `"account_mode":"SINGLE"` en el archivo de configuración MSAL, puede convertir de forma segura el objeto de aplicación devuelto como un `ISingleAccountPublicCLientApplication`.

```java
private ISingleAccountPublicClientApplication mSingleAccountApp;

/*Configure your sample app and save state for this activity*/
PublicClientApplication.create(this.getApplicationCOntext(),
  R.raw.auth_config,
  new PublicClientApplication.ApplicationCreatedListener(){
  @Override
  public void onCreated(IPublicClientApplication application){
  mSingleAccountApp = (ISingleAccountPublicClientApplication)application;
  loadAccount();
  }
  @Override
  public void onError(MsalException exception{
  /*Fail to initialize PublicClientApplication */
  }
});
```

### <a name="detect-single-vs-multiple-account-mode"></a>Detección del modo de varias cuentas o de una sola

Si va a escribir una aplicación que solo utilizarán los trabajadores de primera línea en un dispositivo compartido, se recomienda escribirla para que admita únicamente el modo de una sola cuenta. Esto incluye la mayoría de las aplicaciones que se centran en tareas, como las de registros médicos, facturas y la mayor parte de las de línea de negocio. De este modo, se simplificará el desarrollo, ya que no es necesario acomodar muchas características del SDK.

Si la aplicación admite varias cuentas, así como el modo de dispositivo compartido, debe realizar una comprobación de tipo y convertirla en la interfaz adecuada, como se muestra a continuación.

```java
private IPublicClientApplication mApplication;

        if (mApplication instanceOf IMultipleAccountPublicClientApplication) {
          IMultipleAccountPublicClientApplication multipleAccountApplication = (IMultipleAccountPublicClientApplication) mApplication;
          ...
        } else if (mApplication instanceOf    ISingleAccountPublicClientApplication) {
           ISingleAccountPublicClientApplication singleAccountApplication = (ISingleAccountPublicClientApplication) mApplication;
            ...
        }
```

### <a name="get-the-signed-in-user-and-determine-if-a-user-has-changed-on-the-device"></a>Obtención del usuario que ha iniciado sesión y comprobación de si un usuario ha cambiado en el dispositivo

El método `loadAccount` recupera la cuenta del usuario que ha iniciado sesión. El método `onAccountChanged` determina si el usuario con la sesión iniciada ha cambiado y, en tal caso, se limpia:

```java
private void loadAccount()
{
  mSingleAccountApp.getCurrentAccountAsync(new ISingleAccountPublicClientApplication.CurrentAccountCallback()
  {
    @Overide
    public void onAccountLoaded(@Nullable IAccount activeAccount)
    {
      if (activeAccount != null)
      {
        signedInUser = activeAccount;
        mSingleAccountApp.acquireTokenSilentAsync(SCOPES,"http://login.microsoftonline.com/common",getAuthSilentCallback());
      }
    }
    @Override
    public void on AccountChanged(@Nullable IAccount priorAccount, @Nullable Iaccount currentAccount)
    {
      if (currentAccount == null)
      {
        //Perform a cleanup task as the signed-in account changed.
        updateSingedOutUI();
      }
    }
    @Override
    public void onError(@NonNull Exception exception)
    {
    }
  }
}
```

### <a name="globally-sign-in-a-user"></a>Inicio de sesión de un usuario de forma global

El siguiente código permite al usuario del dispositivo iniciar sesión en otras aplicaciones que usan MSAL con la aplicación Authenticator:

```java
private void onSignInClicked()
{
  mSingleAccountApp.signIn(getActivity(), SCOPES, null, getAuthInteractiveCallback());
}
```

### <a name="globally-sign-out-a-user"></a>Cierre de sesión de un usuario de forma global

El siguiente código elimina la cuenta con la sesión iniciada y borra los tokens almacenados en caché no solo de la aplicación, sino también del dispositivo que está en modo de dispositivo compartido:

```java
private void onSignOutClicked()
{
  mSingleAccountApp.signOut(new ISingleAccountPublicClientApplication.SignOutCallback()
  {
    @Override
    public void onSignOut()
    {
      updateSignedOutUI();
    }
    @Override
    public void onError(@NonNull MsalException exception)
    {
      /*failed to remove account with an exception*/
    }
  });
}
```

## <a name="administrator-guide"></a>Guía del administrador

En los pasos siguientes se describe cómo configurar la aplicación en Azure Portal y cómo poner el dispositivo en modo de dispositivo compartido.

### <a name="register-your-application-in-azure-active-directory"></a>Registro de la aplicación en Azure Active Directory

En primer lugar, registre la aplicación en el inquilino de su organización. A continuación, proporcione estos valores en auth_config.json para que la aplicación se ejecute correctamente.

Para más información sobre cómo hacerlo, consulte [Registro de la aplicación](./tutorial-v2-android.md#register-your-application).

> [!NOTE]
> Al registrar la aplicación, use la guía de inicio rápido que se encuentra a la izquierda y, a continuación, seleccione **Android**. De esta forma, llegará a una página en la que se le pedirá que proporcione el valor de **Nombre del paquete** y **Hash de firma** de la aplicación. Son muy importantes para garantizar que la configuración de la aplicación funcione. Después recibirá un objeto de configuración que puede usar para la aplicación que va a cortar y pegar en el archivo auth_config.json.

![Pantalla de registro de la aplicación](media/tutorial-v2-shared-device-mode/register-app.png) Debe seleccionar **Make this change for me** (Realizar este cambio) y, a continuación, proporcionar los valores que el inicio rápido solicita en Azure Portal. Una vez hecho esto, se generarán todos los archivos de configuración necesarios.

![Pantalla de información de configuración de la aplicación](media/tutorial-v2-shared-device-mode/config-info.png)

## <a name="set-up-a-tenant"></a>Configuración de un inquilino

Para las pruebas, configure lo siguiente en el inquilino: al menos dos empleados, un administrador de dispositivos en la nube y un administrador global. En Azure Portal, configure el administrador de dispositivos en la nube; para ello, modifique los roles de la organización. En Azure Portal, para acceder a los roles de la organización, seleccione **Azure Active Directory** > **Roles y administradores** > **Administrador de dispositivos en la nube**. Agregue los usuarios que puedan poner un dispositivo en modo compartido.

## <a name="set-up-an-android-device-in-shared-mode"></a>Configuración de un dispositivo Android en el modo compartido

### <a name="download-the-authenticator-app"></a>Descarga de la aplicación Authenticator

Descargue la aplicación Microsoft Authenticator de Google Play. Si ya ha descargado la aplicación, asegúrese de que es la versión más reciente.

### <a name="authenticator-app-settings--registering-the-device-in-the-cloud"></a>Configuración de la aplicación Authenticator y registro del dispositivo en la nube

Inicie la aplicación Authenticator y vaya a la página principal de la cuenta. Una vez que vea la página **Agregar cuenta**, estará listo para compartir el dispositivo.

![Pantalla Agregar cuenta de Authenticator](media/tutorial-v2-shared-device-mode/authenticator-add-account.png)

 Vaya al panel **Configuración** con la barra de menús del lado derecho. Seleccione **Registro del dispositivo** en **Cuentas profesionales y educativas**.

 ![Pantalla Agregar cuenta de Authenticator](media/tutorial-v2-shared-device-mode/authenticator-settings.png)

 Al hacer clic en este botón, se le pedirá que autorice el acceso a los contactos del dispositivo. Esto se debe a la integración de la cuenta de Android en el dispositivo. Elija **permitir**.

 ![Pantalla Agregar cuenta de Authenticator](media/tutorial-v2-shared-device-mode/authenticator-allow-screen.png)

El administrador de dispositivos en la nube debe escribir su correo electrónico de la organización en **Registro como dispositivo compartido**. A continuación, haga clic en el botón **Registrar como dispositivo compartido** y escriba sus credenciales.

![pantalla de registro del dispositivo](media/tutorial-v2-shared-device-mode/register-device.png)

![inicio de sesión](media/tutorial-v2-shared-device-mode/sign-in.png)

El dispositivo ya está en modo compartido.

![pantalla de registro del dispositivo](media/tutorial-v2-shared-device-mode/shared-device-mode-screen.png)

 Cualquier inicio y cierre de sesión en el dispositivo será global, lo que significa que se aplicará a todas las aplicaciones que estén integradas con MSAL y Microsoft Authenticator en el dispositivo. Ahora puede implementar aplicaciones en el dispositivo que usen las características del modo de dispositivo compartido.

## <a name="view-the-shared-device-in-the-azure-portal"></a>Visualización del dispositivo compartido en Azure Portal

Una vez que haya colocado un dispositivo en modo compartido, será conocido para la organización y se realizará su seguimiento en el inquilino de esta. Puede ver los dispositivos compartidos si examina el **tipo de combinación** en la hoja de Azure Active Directory de Azure Portal.

![Hoja Todos los dispositivos en Azure Portal](media/tutorial-v2-shared-device-mode/registered-device-screen.png)

## <a name="running-the-sample-app"></a>Ejecución de la aplicación de ejemplo

El ejemplo es una aplicación sencilla que llamará a la instancia de Graph API de la organización. En la primera ejecución, se le pedirá su consentimiento ya que la aplicación es nueva en la cuenta de empleado.

![Pantalla de información de configuración de la aplicación](media/tutorial-v2-shared-device-mode/run-app-permissions-requested.png)

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el modo compartido en [Modo de dispositivo compartido para dispositivos Android](msal-android-shared-devices.md)