---
title: 'Inicio y cierre de sesión de usuarios y llamada a Microsoft Graph (Android): plataforma de identidad de Microsoft | Azure'
description: Obtención de un token de acceso y llamada a Microsoft Graph o a las API que necesitan tokens de acceso desde la plataforma de identidad de Microsoft (Android)
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 11/26/2019
ms.author: hahamil
ms.reviewer: brandwe
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: b4de8a5e96466ea324475030df1f00eb6bb5cf1a
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118295"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-from-an-android-application"></a>Tutorial: Inicio de sesión de usuarios y llamada a Microsoft Graph desde una aplicación Android 

>[!NOTE]
>En este tutorial se muestran ejemplos simplificados de cómo trabajar con MSAL para Android. Para simplificar, en este tutorial solo se usa el modo de cuenta única. También puede ver el repositorio y clonar [la aplicación de ejemplo preconfigurada](https://github.com/Azure-Samples/ms-identity-android-java/) para explorar escenarios más complejos. Consulte el [Inicio rápido](./quickstart-v2-android.md) para más información sobre la aplicación de ejemplo, la configuración y el registro. 

En este tutorial, aprenderá a integrar su aplicación Android con la Plataforma de identidad de Microsoft mediante la biblioteca de autenticación de Microsoft para Android. Aprenderá a iniciar y cerrar la sesión de un usuario, obtener un token de acceso para llamar a la API de Microsoft Graph y realizar una solicitud a Graph API. 

> [!div class="checklist"]
> * Integración de una aplicación Android con la Plataforma de identidad de Microsoft 
> * Inicio de sesión de un usuario 
> * Obtención de un token de acceso para llamar a Microsoft Graph API 
> * Llamar a Microsoft Graph API 
> * Cierre de la sesión de un usuario 

Cuando haya completado este tutorial, la aplicación aceptará inicios de sesión de cuentas Microsoft personales (como outlook.com, live.com y otras), así como de cuentas profesionales y educativas de cualquier empresa u organización que utilice Azure Active Directory.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="how-this-tutorial-works"></a>Funcionamiento de este tutorial

![Muestra cómo funciona la aplicación de ejemplo generada por este tutorial](../../../includes/media/active-directory-develop-guidedsetup-android-intro/android-intro.svg)

La aplicación de este tutorial iniciará la sesión de los usuarios y obtendrá datos en su nombre. A estos datos se accederá a través de una API protegida (Microsoft Graph API) que requiere autorización y que está protegida por la plataforma de identidad de Microsoft.

Más concretamente:

* La aplicación iniciará la sesión del usuario mediante un explorador o Microsoft Authenticator y el Portal de empresa de Intune.
* El usuario final aceptará los permisos que la aplicación ha solicitado.
* Se emitirá un token de acceso a la aplicación para Microsoft Graph API.
* El token de acceso se incluirán en la solicitud HTTP a la API web.
* Se procesa la respuesta de Microsoft Graph.

En este ejemplo se usa la Biblioteca de Microsoft Authentication para Android (MSAL) para implementar la autenticación: [com.microsoft.identity.client](https://javadoc.io/doc/com.microsoft.identity.client/msal).

 MSAL renovará automáticamente los tokens, proporcionará el inicio de sesión único entre las otras aplicaciones del dispositivo y administrará las cuentas.

### <a name="prerequisites"></a>Prerrequisitos

* En este tutorial se requiere la versión 3.5+ de Android Studio.

## <a name="create-a-project"></a>Crear un proyecto
Si aún no tiene una aplicación Android, siga estos pasos para configurar un nuevo proyecto. 

1. Abra Android Studio y seleccione **Start a new Android Studio project** (Iniciar un nuevo proyecto de Android Studio).
2. Seleccione **Basic Activity** (Actividad básica) y, después, seleccione **Siguiente**.
3. Asigne un nombre a la aplicación.
4. Guarde el nombre del paquete. Deberá especificarlo más tarde en Azure Portal.
5. Cambie el lenguaje de **Kotlin** a **Java**.
6. Establezca el **nivel de API mínimo** en **API 19** o superior y haga clic en **Finalizar**.
7. En la vista de proyecto, seleccione **Proyecto** en la lista desplegable para mostrar los archivos de proyecto de origen y no de origen, abra **app/build.gradle** y establezca `targetSdkVersion` en `28`.

## <a name="integrate-with-microsoft-authentication-library"></a>Integración con la biblioteca de autenticación de Microsoft 

### <a name="register-your-application"></a>Registrar su aplicación

1. Vaya a [Azure Portal](https://aka.ms/MobileAppReg).
2. Abra la [hoja de Registros de aplicaciones](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) y haga clic en **+Nuevo registro**.
3. Escriba un **Nombre** para la aplicación y, después, **sin** configurar un identificador URI de redirección, haga clic en **Registrar**.
4. En la sección **Administrar** del panel que aparece, seleccione **Autenticación** >  **+ Agregar una plataforma** > **Android**. (Es posible que tenga que seleccionar "Switch to the new experience" [Probar la nueva experiencia] cerca de la parte superior de la hoja para ver esta sección)
5. Escriba el nombre del paquete del proyecto. Si descargó el código, este valor es `com.azuresamples.msalandroidapp`.
6. En la sección **Hash de firma** de la página **Configurar su aplicación para Android**, haga clic en **Generating a development Signature Hash** (Generación de un hash de firma de desarrollo) y copie el comando KeyTool para usarlo en la plataforma.

   > [!Note]
   > KeyTool. exe se instala como parte de Java Development Kit (JDK). También debe instalar la herramienta OpenSSL para ejecutar el comando KeyTool. Consulte la [documentación de Android sobre cómo generar una clave](https://developer.android.com/studio/publish/app-signing#generate-key) para obtener más información. 

7. Escriba el **hash de firma** generado por KeyTool.
8. Haga clic en `Configure` y guarde la **configuración de MSAL** que aparece en la página **Configuración de Android** para que pueda escribirla cuando configure la aplicación más tarde.  Haga clic en **Done**(Listo).

### <a name="configure-your-application"></a>Configuración de la aplicación 

1. En el panel del proyecto de Android Studio, vaya a **app\src\main\res**.
2. Haga clic con el botón derecho en **res** y elija **New** > **Directory** (Nuevo > Directorio). Escriba `raw` como el nombre del nuevo directorio y haga clic en **OK** (Aceptar).
3. En **app** > **src** > **main** > **res** > **raw**, cree un nuevo archivo JSON llamado `auth_config_single_account.json` y pegue la configuración de MSAL que guardó anteriormente. 

    Debajo del identificador URI de redirección, pegue: 
    ```json
      "account_mode" : "SINGLE",
    ```
    El archivo de configuración debe ser similar a este ejemplo: 
    ```json   
    {
      "client_id" : "0984a7b6-bc13-4141-8b0d-8f767e136bb7",
      "authorization_user_agent" : "DEFAULT",
      "redirect_uri" : "msauth://com.azuresamples.msalandroidapp/1wIqXSqBj7w%2Bh11ZifsnqwgyKrY%3D",
      "account_mode" : "SINGLE",
      "authorities" : [
        {
          "type": "AAD",
          "audience": {
            "type": "AzureADandPersonalMicrosoftAccount",
            "tenant_id": "common"
          }
        }
      ]
    }
   ```
    
   >[!NOTE]
   >En este tutorial solo se muestra cómo configurar una aplicación en el modo de cuenta única. Consulte la documentación para más información sobre el [modo de cuenta única frente a varias cuentas](./single-multi-account.md) y sobre cómo [configurar la aplicación](./msal-configuration.md).
   
4. En **app** > **src** > **main** > **AndroidManifest.xml**, agregue la actividad `BrowserTabActivity` siguiente al cuerpo de la aplicación. Esta entrada permite a Microsoft volver a llamar a la aplicación una vez completada la autenticación:

    ```xml
    <!--Intent filter to capture System Browser or Authenticator calling back to our app after sign-in-->
    <activity
        android:name="com.microsoft.identity.client.BrowserTabActivity">
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="msauth"
                android:host="Enter_the_Package_Name"
                android:path="/Enter_the_Signature_Hash" />
        </intent-filter>
    </activity>
    ```

    Sustituya el nombre del paquete que registró en Azure Portal por el valor `android:host=`.
    Sustituya el hash de la clave que registró en Azure Portal por el valor `android:path=`. El hash de firma **no** debe estar codificado como dirección URL. Asegúrese de que haya un carácter de `/` inicial al principio del hash de firma. 
    >[!NOTE]
    >El "Nombre del paquete" por el que reemplazará el valor de `android:host` debe ser similar a: "com.azuresamples.msalandroidapp". El "Hash de firma" por el que reemplazará el valor de `android:path` debe ser similar a: "/1wIqXSqBj7w+h11ZifsnqwgyKrY=". También podrá encontrar estos valores en la hoja Autenticación del registro de la aplicación. Tenga en cuenta que el identificador URI de redirección tendrá un aspecto similar al siguiente: "msauth://com.azuresamples.msalandroidapp/1wIqXSqBj7w%2Bh11ZifsnqwgyKrY%3D". Aunque el hash de firma está codificado como dirección URL al final de este valor, el hash de firma **no** debe estar codificado como dirección URL en el valor de `android:path`. 

## <a name="use-msal"></a>Uso de MSAL 

### <a name="add-msal-to-your-project"></a>Adición de MSAL al proyecto

1. En la ventana del proyecto de Android Studio, vaya a **app** > **src** > **build.gradle** y agregue lo siguiente: 

    ```gradle
    repositories{
        jcenter()
    }  
    dependencies{
        implementation 'com.microsoft.identity.client:msal:1.+'
        implementation 'com.microsoft.graph:microsoft-graph:1.5.+'
    }
    packagingOptions{
        exclude("META-INF/jersey-module-version") 
    }
    ```
    [Más información sobre el SDK de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-java/)

### <a name="required-imports"></a>Importaciones necesarias 

Agregue lo siguiente en la parte superior de **app** > **src** > **main**> **java** > **com.example(la aplicación)**  > **MainActivity.java**. 

```java
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import com.google.gson.JsonObject;
import com.microsoft.graph.authentication.IAuthenticationProvider; //Imports the Graph sdk Auth interface
import com.microsoft.graph.concurrency.ICallback;
import com.microsoft.graph.core.ClientException;
import com.microsoft.graph.http.IHttpRequest;
import com.microsoft.graph.models.extensions.*;
import com.microsoft.graph.requests.extensions.GraphServiceClient;
import com.microsoft.identity.client.AuthenticationCallback; // Imports MSAL auth methods
import com.microsoft.identity.client.*;
import com.microsoft.identity.client.exception.*;
```

## <a name="instantiate-publicclientapplication"></a>Creación de instancia de PublicClientApplication
#### <a name="initialize-variables"></a>Inicialización de variables 
```java
private final static String[] SCOPES = {"Files.Read"};
/* Azure AD v2 Configs */
final static String AUTHORITY = "https://login.microsoftonline.com/common";
private ISingleAccountPublicClientApplication mSingleAccountApp;

private static final String TAG = MainActivity.class.getSimpleName();

/* UI & Debugging Variables */
Button signInButton;
Button signOutButton;
Button callGraphApiInteractiveButton;
Button callGraphApiSilentButton;
TextView logTextView;
TextView currentUserTextView;
```

### <a name="oncreate"></a>onCreate
Dentro de la clase `MainActivity`, haga referencia al siguiente método onCreate() para crear una instancia de MSAL mediante `SingleAccountPublicClientApplication`.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initializeUI();

    PublicClientApplication.createSingleAccountPublicClientApplication(getApplicationContext(),
            R.raw.auth_config_single_account, new IPublicClientApplication.ISingleAccountApplicationCreatedListener() {
                @Override
                public void onCreated(ISingleAccountPublicClientApplication application) {
                    mSingleAccountApp = application;
                    loadAccount();
                }
                @Override
                public void onError(MsalException exception) {
                    displayError(exception);
                }
            });
}
```

### <a name="loadaccount"></a>loadAccount 

```java
//When app comes to the foreground, load existing account to determine if user is signed in 
private void loadAccount() {
    if (mSingleAccountApp == null) {
        return;
    }

    mSingleAccountApp.getCurrentAccountAsync(new ISingleAccountPublicClientApplication.CurrentAccountCallback() {
        @Override
        public void onAccountLoaded(@Nullable IAccount activeAccount) {
            // You can use the account data to update your UI or your app database.
            updateUI(activeAccount);
        }
        
        @Override
        public void onAccountChanged(@Nullable IAccount priorAccount, @Nullable IAccount currentAccount) {
            if (currentAccount == null) {
                // Perform a cleanup task as the signed-in account changed.
                performOperationOnSignOut();
            }
        }

        @Override
        public void onError(@NonNull MsalException exception) {
            displayError(exception);
        }
    });
}
```

### <a name="initializeui"></a>initializeUI
Puede realizar escuchas en los botones y llamar a métodos o registrar los errores según corresponda. 
```java
private void initializeUI(){
        signInButton = findViewById(R.id.signIn);
        callGraphApiSilentButton = findViewById(R.id.callGraphSilent);
        callGraphApiInteractiveButton = findViewById(R.id.callGraphInteractive);
        signOutButton = findViewById(R.id.clearCache);
        logTextView = findViewById(R.id.txt_log);
        currentUserTextView = findViewById(R.id.current_user);
        
        //Sign in user 
        signInButton.setOnClickListener(new View.OnClickListener(){
            public void onClick(View v) {
                if (mSingleAccountApp == null) {
                    return;
                }
                mSingleAccountApp.signIn(MainActivity.this, null, SCOPES, getAuthInteractiveCallback());
            }
        });
        
        //Sign out user
        signOutButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSingleAccountApp == null){
                    return;
                }
                mSingleAccountApp.signOut(new ISingleAccountPublicClientApplication.SignOutCallback() {
                    @Override
                    public void onSignOut() {
                        updateUI(null);
                        performOperationOnSignOut();
                    }
                    @Override
                    public void onError(@NonNull MsalException exception){
                        displayError(exception);
                    }
                });
            }
        });
        
        //Interactive 
        callGraphApiInteractiveButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSingleAccountApp == null) {
                    return;
                }
                mSingleAccountApp.acquireToken(MainActivity.this, SCOPES, getAuthInteractiveCallback());
            }
        });

        //Silent
        callGraphApiSilentButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSingleAccountApp == null){
                    return;
                }
                mSingleAccountApp.acquireTokenSilentAsync(SCOPES, AUTHORITY, getAuthSilentCallback());
            }
        });
    }
```

> [!Important]
> Si cierra sesión con MSAL, se elimina de esta aplicación toda la información conocida sobre un usuario, pero el usuario seguirá teniendo una sesión activa en su dispositivo. Si el usuario intenta iniciar sesión nuevamente, puede ver la interfaz de usuario de inicio de sesión, pero es posible que no tenga que volver a escribir las credenciales debido a que la sesión del dispositivo sigue activa. 

### <a name="getauthinteractivecallback"></a>getAuthInteractiveCallback
Devolución de llamada usada para las solicitudes interactivas.

```java 
private AuthenticationCallback getAuthInteractiveCallback() {
    return new AuthenticationCallback() {
        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            /* Successfully got a token, use it to call a protected resource - MSGraph */
            Log.d(TAG, "Successfully authenticated");
            /* Update UI */
            updateUI(authenticationResult.getAccount());
            /* call graph */
            callGraphAPI(authenticationResult);
        }

        @Override
        public void onError(MsalException exception) {
            /* Failed to acquireToken */
            Log.d(TAG, "Authentication failed: " + exception.toString());
            displayError(exception);
        }
        @Override
        public void onCancel() {
            /* User canceled the authentication */
            Log.d(TAG, "User cancelled login.");
        }
    };
}
```

### <a name="getauthsilentcallback"></a>getAuthSilentCallback
Devolución de llamada usada para las solicitudes silenciosas. 
```java 
private SilentAuthenticationCallback getAuthSilentCallback() {
    return new SilentAuthenticationCallback() {
        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            Log.d(TAG, "Successfully authenticated");
            callGraphAPI(authenticationResult);
        }
        @Override
        public void onError(MsalException exception) {
            Log.d(TAG, "Authentication failed: " + exception.toString());
            displayError(exception);
        }
    };
}
```

## <a name="call-microsoft-graph-api"></a>Llamada a Microsoft Graph API 

En el código siguiente se muestra cómo llamar a Graph API mediante el SDK de Graph. 

### <a name="callgraphapi"></a>callGraphAPI 

```java
private void callGraphAPI(IAuthenticationResult authenticationResult) {

    final String accessToken = authenticationResult.getAccessToken();

    IGraphServiceClient graphClient =
            GraphServiceClient
                    .builder()
                    .authenticationProvider(new IAuthenticationProvider() {
                        @Override
                        public void authenticateRequest(IHttpRequest request) {
                            Log.d(TAG, "Authenticating request," + request.getRequestUrl());
                            request.addHeader("Authorization", "Bearer " + accessToken);
                        }
                    })
                    .buildClient();
    graphClient
            .me()
            .drive()
            .buildRequest()
            .get(new ICallback<Drive>() {
                @Override
                public void success(final Drive drive) {
                    Log.d(TAG, "Found Drive " + drive.id);
                    displayGraphResult(drive.getRawObject());
                }

                @Override
                public void failure(ClientException ex) {
                    displayError(ex);
                }
            });
}
```

## <a name="add-ui"></a>Incorporación de la interfaz de usuario
### <a name="activity"></a>Actividad 
Si desea modelar la interfaz de usuario de este tutorial, los siguientes métodos proporcionan una guía para actualizar texto y escuchar botones.

#### <a name="updateui"></a>updateUI
Habilitar o deshabilitar botones en función del estado de inicio de sesión y establecer texto.  
```java 
private void updateUI(@Nullable final IAccount account) {
    if (account != null) {
        signInButton.setEnabled(false);
        signOutButton.setEnabled(true);
        callGraphApiInteractiveButton.setEnabled(true);
        callGraphApiSilentButton.setEnabled(true);
        currentUserTextView.setText(account.getUsername());
    } else {
        signInButton.setEnabled(true);
        signOutButton.setEnabled(false);
        callGraphApiInteractiveButton.setEnabled(false);
        callGraphApiSilentButton.setEnabled(false);
        currentUserTextView.setText("");
        logTextView.setText("");
    }
}
```
#### <a name="displayerror"></a>displayError
```java 
private void displayError(@NonNull final Exception exception) {
       logTextView.setText(exception.toString());
   }
```

#### <a name="displaygraphresult"></a>displayGraphResult

```java
private void displayGraphResult(@NonNull final JsonObject graphResponse) {
      logTextView.setText(graphResponse.toString());
  }
```
#### <a name="performoperationonsignout"></a>performOperationOnSignOut
Método para actualizar el texto de la interfaz de usuario para que refleje el cierre de sesión. 

```java
private void performOperationOnSignOut() {
    final String signOutText = "Signed Out.";
    currentUserTextView.setText("");
    Toast.makeText(getApplicationContext(), signOutText, Toast.LENGTH_SHORT)
            .show();
}
```
### <a name="layout"></a>Diseño 

Archivo `activity_main.xml` de ejemplo para mostrar botones y cuadros de texto. 

```xml 
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFFFFF"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:paddingTop="5dp"
        android:paddingBottom="5dp"
        android:weightSum="10">

        <Button
            android:id="@+id/signIn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="5"
            android:gravity="center"
            android:text="Sign In"/>

        <Button
            android:id="@+id/clearCache"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="5"
            android:gravity="center"
            android:text="Sign Out"
            android:enabled="false"/>

    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:orientation="horizontal">

        <Button
            android:id="@+id/callGraphInteractive"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="5"
            android:text="Get Graph Data Interactively"
            android:enabled="false"/>

        <Button
            android:id="@+id/callGraphSilent"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="5"
            android:text="Get Graph Data Silently"
            android:enabled="false"/>
    </LinearLayout>

    <TextView
        android:text="Getting Graph Data..."
        android:textColor="#3f3f3f"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="5dp"
        android:id="@+id/graphData"
        android:visibility="invisible"/>

    <TextView
        android:id="@+id/current_user"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="20dp"
        android:layout_weight="0.8"
        android:text="Account info goes here..." />

    <TextView
        android:id="@+id/txt_log"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="20dp"
        android:layout_weight="0.8"
        android:text="Output goes here..." />
</LinearLayout>
```

## <a name="test-your-app"></a>Prueba de la aplicación

### <a name="run-locally"></a>Ejecución en modo local

Cree e implemente la aplicación en un emulador o dispositivo de prueba. Debe ser capaz de iniciar sesión y obtener tokens para Azure AD o cuentas personales de Microsoft.

Después de iniciar sesión, esta aplicación mostrará los datos devueltos desde el punto de conexión `/me` de Microsoft Graph.

### <a name="consent"></a>Consentimiento

La primera vez que un usuario inicie sesión en su aplicación, Microsoft Identity le pedirá que acepte los permisos solicitados. Algunos inquilinos de Azure AD tienen el consentimiento del usuario deshabilitado, lo que requiere que los administradores den su consentimiento en nombre de todos los usuarios. Para admitir este escenario, tendrá que crear su propio inquilino o recibir el consentimiento del administrador. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el objeto de aplicación que creó en el paso [Registrar su aplicación](#register-your-application).

## <a name="get-help"></a>Obtener ayuda

Visite [Ayuda y soporte técnico](./developer-support-help-options.md) si tienes algún problema con este tutorial o con la plataforma de identidad de Microsoft.