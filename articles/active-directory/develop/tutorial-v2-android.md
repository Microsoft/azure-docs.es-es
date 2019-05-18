---
title: 'Introducción a Android: plataforma de identidad de Microsoft | Azure'
description: Cómo puede una aplicación de Android obtener un token de acceso y llamar a Microsoft Graph API o a las API que requieran tokens de acceso desde la plataforma de identidad de Microsoft.
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/26/2019
ms.author: dadobali
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2914d6b1f4a6c94d7e3d4456c8255c1563a71b3e
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406653"
---
# <a name="sign-in-users-and-call-the-microsoft-graph-from-an-android-app"></a>Inicie la sesión de los usuarios y llame a Microsoft Graph desde una aplicación Android

En este tutorial, obtendrá información sobre cómo integrar una aplicación Android en la plataforma de identidad de Microsoft. En concreto, su aplicación iniciará la sesión de un usuario, obtendrá un token de acceso para llamar a Microsoft Graph API y realizará una solicitud a Microsoft Graph API.  

Cuando haya completado la guía, la aplicación podrá aceptar inicios de sesión de cuentas Microsoft personales (como outlook.com, live.com y otras), y cuentas profesionales y educativas de cualquier empresa u organización que utilice Azure Active Directory.

## <a name="how-this-tutorial-works"></a>Funcionamiento de este tutorial

![Muestra cómo funciona la aplicación de ejemplo generada por este tutorial](../../../includes/media/active-directory-develop-guidedsetup-android-intro/android-intro.svg)

La aplicación de este ejemplo iniciará la sesión de los usuarios y obtendrá datos en su nombre.  Se accederá a los datos a través de una API protegida (Microsoft Graph API en este caso) que requiere autorización.

Más concretamente:

* La aplicación iniciará la sesión del usuario mediante un explorador o Microsoft Authenticator y el Portal de empresa de Intune.
* El usuario final aceptará los permisos que la aplicación ha solicitado. 
* Se emitirá un token de acceso a la aplicación para Microsoft Graph API.
* El token de acceso se incluirán en la solicitud HTTP a la API web.
* Se procesa la respuesta de Microsoft Graph.

Este ejemplo utiliza la biblioteca de Microsoft Authentication para Android (MSAL) para implementar la autenticación. MSAL renovará automáticamente los tokens, proporcionará inicio de sesión único entre las otras aplicaciones del dispositivo y administrará las cuentas.

## <a name="prerequisites"></a>Requisitos previos

* Esta configuración guiada usa Android Studio.
* Se necesita Android 16 o versiones posteriores (se recomienda la versión 19 o posterior).

## <a name="library"></a>Biblioteca

Esta guía utiliza la siguiente biblioteca de autenticación:

|Biblioteca|DESCRIPCIÓN|
|---|---|
|[com.microsoft.identity.client](https://javadoc.io/doc/com.microsoft.identity.client/msal)|Biblioteca de autenticación de Microsoft (MSAL)|

## <a name="set-up-your-project"></a>Configurar su proyecto

En este tutorial, vamos a crear un nuevo proyecto. Si desea descargar el tutorial completado en su lugar, [descargue el código](https://github.com/Azure-Samples/active-directory-android-native-v2/archive/master.zip).

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

1. Abra Android Studio y seleccione **Start a new Android Studio project** (Iniciar un nuevo proyecto de Android Studio).
    - Si Android Studio ya está abierto, seleccione **File** > **New** > **New Project** (Archivo > Nuevo > Nuevo proyecto).
2. Deje **Empty Activity** (Actividad vacía) tal como está y seleccione **Next** (Siguiente).
3. Ponga un nombre a la aplicación, configure `Minimum API level` en la **API 19 o posterior** y seleccione **Finish** (Finalizar).
5. En `app/build.gradle`, establezca `targetedSdkVersion` en 27. 

## <a name="register-your-application"></a>Registrar su aplicación

Puede registrar su aplicación de dos maneras distintas, como se describe en las dos secciones siguientes.

### <a name="register-your-app"></a>Registro de la aplicación

1. Vaya a [Azure Portal](https://aka.ms/MobileAppReg) > Seleccione `New registration`. 
2. Escriba un **nombre** para la aplicación > `Register`. **No establezca un URI de redireccionamiento en esta fase**. 
3. En la sección `Manage`, vaya a `Authentication` > `Add a platform` > `Android`.
    - Escriba el nombre del paquete del proyecto. Si descargó el código, este valor es `com.azuresamples.msalandroidapp`. 
    - Especifique el hash de firma de depuración y desarrollo. Use el comando KeyTool del portal para generar el hash de firma. 
4. Presione `Configure` y almacene la ***configuración de MSAL*** para su uso posterior. 

## <a name="build-your-app"></a>Compilación de la aplicación

### <a name="configure-your-android-app"></a>Configuración de su aplicación para Android

1. Haga clic en **res**(recurso) > **New**(Nuevo) > **Folder**(Carpeta) > **Raw Resources Folder** (Carpeta de recursos sin formato).
2. En **app**(aplicación) > **res**(recurso) > **raw**(sin formato), cree un nuevo archivo JSON llamado `auth_config.json` y pegue su ***configuración de MSAL***. Consulte [MSAL Configuration](https://github.com/AzureAD/microsoft-authentication-library-for-android/wiki/Configuring-your-app) (Configuración de MSAL) para obtener más información.
   <!-- Workaround for Docs conversion bug -->
3. En **app**(aplicación) > **manifests**(manifiestos) > **AndroidManifest.xml**, agregue la actividad `BrowserTabActivity` siguiente. Esta entrada permite a Microsoft volver a llamar a la aplicación una vez completada la autenticación:

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

    Tenga en cuenta que el hash de firma utilizado no debe ser la dirección URL codificada en **AndroidManifest.xml**. 

4. Dentro de **AndroidManifest.xml** y justo encima de la etiqueta `<application>`, agregue los siguientes permisos:

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    ```

5. En `BrowserTabActivity`, reemplace el ***Package Name*** (Nombre del paquete) y ***Signature Hash*** (Hash de firma) por los valores registrados en Azure Portal.

### <a name="create-the-apps-ui"></a>Creación de la interfaz de usuario de la aplicación

1. Vaya a **res** > **layout** y abra **activity_main.xml**.
2. Cambie el diseño de la actividad de `android.support.constraint.ConstraintLayout` u otro a `LinearLayout`.
3. Agregue la propiedad `android:orientation="vertical"` al nodo `LinearLayout`.
4. Pegue el código siguiente en el nodo `LinearLayout`, reemplazando el contenido actual:

    ```xml
    <TextView
        android:text="Welcome, "
        android:textColor="#3f3f3f"
        android:textSize="50px"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginTop="15dp"
        android:id="@+id/welcome"
        android:visibility="invisible"/>

    <Button
        android:id="@+id/callGraph"
        android:text="Call Microsoft Graph"
        android:textColor="#FFFFFF"
        android:background="#00a1f1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="200dp"
        android:textAllCaps="false" />

    <TextView
        android:text="Getting Graph Data..."
        android:textColor="#3f3f3f"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="5dp"
        android:id="@+id/graphData"
        android:visibility="invisible"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dip"
        android:layout_weight="1"
        android:gravity="center|bottom"
        android:orientation="vertical" >

        <Button
            android:text="Sign Out"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="15dp"
            android:textColor="#FFFFFF"
            android:background="#00a1f1"
            android:textAllCaps="false"
            android:id="@+id/clearCache"
            android:visibility="invisible" />
    </LinearLayout>
    ```

### <a name="add-msal-to-your-project"></a>Adición de MSAL al proyecto

1. En Android Studio, seleccione **Gradle Scripts** > **build.gradle (Module: app)**.
2. En **Dependencies** (Dependencias), pegue el código siguiente:

    ```gradle  
    implementation 'com.android.volley:volley:1.1.1'
    implementation 'com.microsoft.identity.client:msal:0.3.+'
    ```

### <a name="use-msal"></a>Uso de MSAL 

En las secciones siguientes se realizarán cambios dentro de `MainAcitivty.java`. Recorreremos cada uno de los pasos necesarios para agregar y usar MSAL en la aplicación.

#### <a name="required-imports"></a>Importaciones necesarias

Agregue las importaciones siguientes al proyecto: 

```java
import android.app.Activity;
import android.content.Intent;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;
import com.android.volley.*;
import com.android.volley.toolbox.JsonObjectRequest;
import com.android.volley.toolbox.Volley;
import org.json.JSONObject;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import com.microsoft.identity.client.*;
import com.microsoft.identity.client.exception.*;
```

#### <a name="instantiating-msal"></a>Creación de instancias de MSAL 

Dentro de la clase `MainActivity`, será necesario crear una instancia de MSAL junto con algunas configuraciones acerca de lo que hará la aplicación, incluidos los ámbitos y la API web a la que queremos acceder. 

Copie las siguientes variables dentro de `MainActivity`:

```java
final static String SCOPES [] = {"https://graph.microsoft.com/User.Read"};
final static String MSGRAPH_URL = "https://graph.microsoft.com/v1.0/me";

/* UI & Debugging Variables */
private static final String TAG = MainActivity.class.getSimpleName();
Button callGraphButton;
Button signOutButton;

/* Azure AD Variables */
private PublicClientApplication sampleApp;
private IAuthenticationResult authResult;
```

Ahora para crear una instancia de MSAL, copie el código siguiente dentro del método `onCreate(...)`:

```java
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

callGraphButton = (Button) findViewById(R.id.callGraph);
signOutButton = (Button) findViewById(R.id.clearCache);

callGraphButton.setOnClickListener(new View.OnClickListener() {
    public void onClick(View v) {
        onCallGraphClicked();
    }
});

signOutButton.setOnClickListener(new View.OnClickListener() {
    public void onClick(View v) {
        onSignOutClicked();
    }
});

/* Configure your sample app and save state for this activity */
sampleApp = new PublicClientApplication(
        this.getApplicationContext(),
        R.raw.auth_config);

/* Attempt to get a user and acquireTokenSilent */
sampleApp.getAccounts(new PublicClientApplication.AccountsLoadedCallback() {
    @Override
    public void onAccountsLoaded(final List<IAccount> accounts) {
        if (!accounts.isEmpty()) {
            /* This sample doesn't support multi-account scenarios, use the first account */
            sampleApp.acquireTokenSilentAsync(SCOPES, accounts.get(0), getAuthSilentCallback());
        } else {
            /* No accounts */
        }
    }
});
```

El bloque de código anterior trata de iniciar la sesión de los usuarios silenciosamente cuando abren la aplicación a través de `getAccounts(...)` y, si el proceso es correcto, `acquireTokenSilentAsync(...)`.  En las secciones siguientes, implementaremos el controlador de devolución de llamada para el caso en que no haya cuentas con la sesión iniciada. 

#### <a name="use-msal-to-get-tokens"></a>Uso de MSAL para obtener tokens

Ahora, podemos implementar la lógica de procesamiento de la interfaz de usuario de la aplicación y obtener tokens interactivamente a través de MSAL. 

MSAL expone dos métodos principales para obtener los tokens: `acquireTokenSilentAsync` y `acquireToken`.  

`acquireTokenSilentAsync` inicia la sesión de un usuario y obtiene tokens sin interacción del usuario si hay una cuenta. Si se realiza correctamente, MSAL entrega los tokens a su aplicación; si se produce un error, se generará un `MsalUiRequiredException`.  Si se genera esta excepción o si desea que el usuario tenga una experiencia de inicio de sesión interactiva (credenciales, MFA u otras directivas de acceso condicional pueden requerirse o no), puede usar `acquireToken`.  

`acquireToken` siempre mostrará la interfaz de usuario cuando intente iniciar sesión en el usuario y obtener tokens; sin embargo, puede usar cookies de sesión en el explorador o una cuenta en Microsoft Authenticator para brindar una experiencia interactiva de SSO. 

Para comenzar, cree los siguientes tres métodos de interfaz de usuario dentro de la clase `MainActivity`:

```java
/* Set the UI for successful token acquisition data */
private void updateSuccessUI() {
    callGraphButton.setVisibility(View.INVISIBLE);
    signOutButton.setVisibility(View.VISIBLE);
    findViewById(R.id.welcome).setVisibility(View.VISIBLE);
    ((TextView) findViewById(R.id.welcome)).setText("Welcome, " +
            authResult.getAccount().getUsername());
    findViewById(R.id.graphData).setVisibility(View.VISIBLE);
}

/* Set the UI for signed out account */
private void updateSignedOutUI() {
    callGraphButton.setVisibility(View.VISIBLE);
    signOutButton.setVisibility(View.INVISIBLE);
    findViewById(R.id.welcome).setVisibility(View.INVISIBLE);
    findViewById(R.id.graphData).setVisibility(View.INVISIBLE);
    ((TextView) findViewById(R.id.graphData)).setText("No Data");

    Toast.makeText(getBaseContext(), "Signed Out!", Toast.LENGTH_SHORT)
            .show();
}

/* Use MSAL to acquireToken for the end-user
 * Callback will call Graph api w/ access token & update UI
 */
private void onCallGraphClicked() {
    sampleApp.acquireToken(getActivity(), SCOPES, getAuthInteractiveCallback());
}
```

A continuación, agregue un método para obtener la actividad actual y procesar devoluciones de llamadas silenciosas e interactivas:

```java
public Activity getActivity() {
    return this;
}

/* Callback used in for silent acquireToken calls.
 * Looks if tokens are in the cache (refreshes if necessary and if we don't forceRefresh)
 * else errors that we need to do an interactive request.
 */
private AuthenticationCallback getAuthSilentCallback() {
    return new AuthenticationCallback() {

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            /* Successfully got a token, call graph now */
            Log.d(TAG, "Successfully authenticated");

            /* Store the authResult */
            authResult = authenticationResult;

            /* call graph */
            callGraphAPI();

            /* update the UI to post call graph state */
            updateSuccessUI();
        }

        @Override
        public void onError(MsalException exception) {
            /* Failed to acquireToken */
            Log.d(TAG, "Authentication failed: " + exception.toString());

            if (exception instanceof MsalClientException) {
                /* Exception inside MSAL, more info inside the exception */
            } else if (exception instanceof MsalServiceException) {
                /* Exception when communicating with the STS, likely config issue */
            } else if (exception instanceof MsalUiRequiredException) {
                /* Tokens expired or no session, retry with interactive */
            }
        }

        @Override
        public void onCancel() {
            /* User cancelled the authentication */
            Log.d(TAG, "User cancelled login.");
        }
    };
}

/* Callback used for interactive request.  If succeeds we use the access
 * token to call the Microsoft Graph. Does not check cache
 */
private AuthenticationCallback getAuthInteractiveCallback() {
    return new AuthenticationCallback() {

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            /* Successfully got a token, call graph now */
            Log.d(TAG, "Successfully authenticated");
            Log.d(TAG, "ID Token: " + authenticationResult.getIdToken());

            /* Store the auth result */
            authResult = authenticationResult;

            /* call graph */
            callGraphAPI();

            /* update the UI to post call graph state */
            updateSuccessUI();
        }

        @Override
        public void onError(MsalException exception) {
            /* Failed to acquireToken */
            Log.d(TAG, "Authentication failed: " + exception.toString());

            if (exception instanceof MsalClientException) {
                /* Exception inside MSAL, more info inside the exception */
            } else if (exception instanceof MsalServiceException) {
                /* Exception when communicating with the STS, likely config issue */
            }
        }

        @Override
        public void onCancel() {
            /* User cancelled the authentication */
            Log.d(TAG, "User cancelled login.");
        }
    };
}
```

#### <a name="use-msal-for-sign-out"></a>Uso de MSAL para cierre de sesión

A continuación, vamos a agregar compatibilidad para el cierre de sesión a nuestra aplicación. 

Es importante tener en cuenta que el cierre de sesión con MSAL elimina de esta aplicación toda la información conocida acerca de un usuario, pero el usuario seguirá teniendo una sesión activa en su dispositivo. Si el usuario intenta iniciar sesión nuevamente, puede ver la interacción, pero es posible que no tenga que volver a escribir sus credenciales debido a que la sesión del dispositivo está activa. 

Para agregar el cierre de sesión, copie el siguiente método en la aplicación que recorre todas las cuentas y las quita:

```java
/* Clears an account's tokens from the cache.
 * Logically similar to "sign out" but only signs out of this app.
 * User will get interactive SSO if trying to sign back-in.
 */
private void onSignOutClicked() {
    /* Attempt to get a user and acquireTokenSilent
     * If this fails we do an interactive request
     */
    sampleApp.getAccounts(new PublicClientApplication.AccountsLoadedCallback() {
        @Override
        public void onAccountsLoaded(final List<IAccount> accounts) {

            if (accounts.isEmpty()) {
                /* No accounts to remove */

            } else {
                for (final IAccount account : accounts) {
                    sampleApp.removeAccount(
                            account,
                            new PublicClientApplication.AccountsRemovedCallback() {
                        @Override
                        public void onAccountsRemoved(Boolean isSuccess) {
                            if (isSuccess) {
                                /* successfully removed account */
                            } else {
                                /* failed to remove account */
                            }
                        }
                    });
                }
            }

            updateSignedOutUI();
        }
    });
}
```

#### <a name="call-the-microsoft-graph-api"></a>Llamar a Microsoft Graph API

Una vez que hayamos recibido un token, podemos realizar una solicitud a Microsoft Graph API. El token de acceso estará dentro del `AuthenticationResult` dentro del método `onSuccess(...)` de la devolución de llamada de autenticación. Para construir una solicitud autorizada, la aplicación necesitará agregar el token de acceso al encabezado HTTP:

| clave del encabezado    | value                 |
| ------------- | --------------------- |
| Autorización | Portador <token_de_acceso> |

Para hacerlo en el código, agregue los siguientes dos métodos a su aplicación para llamar a Graph y actualizar la interfaz de usuario: 

```java
    /* Use Volley to make an HTTP request to the /me endpoint from MS Graph using an access token */
private void callGraphAPI() {
    Log.d(TAG, "Starting volley request to graph");

    /* Make sure we have a token to send to graph */
    if (authResult.getAccessToken() == null) {return;}

    RequestQueue queue = Volley.newRequestQueue(this);
    JSONObject parameters = new JSONObject();

    try {
        parameters.put("key", "value");
    } catch (Exception e) {
        Log.d(TAG, "Failed to put parameters: " + e.toString());
    }
    JsonObjectRequest request = new JsonObjectRequest(Request.Method.GET, MSGRAPH_URL,
            parameters,new Response.Listener<JSONObject>() {
        @Override
        public void onResponse(JSONObject response) {
            /* Successfully called graph, process data and send to UI */
            Log.d(TAG, "Response: " + response.toString());

            updateGraphUI(response);
        }
    }, new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            Log.d(TAG, "Error: " + error.toString());
        }
    }) {
        @Override
        public Map<String, String> getHeaders() {
            Map<String, String> headers = new HashMap<>();
            headers.put("Authorization", "Bearer " + authResult.getAccessToken());
            return headers;
        }
    };

    Log.d(TAG, "Adding HTTP GET to Queue, Request: " + request.toString());

    request.setRetryPolicy(new DefaultRetryPolicy(
            3000,
            DefaultRetryPolicy.DEFAULT_MAX_RETRIES,
            DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
    queue.add(request);
}

/* Sets the graph response */
private void updateGraphUI(JSONObject graphResponse) {
    TextView graphText = findViewById(R.id.graphData);
    graphText.setText(graphResponse.toString());
}
```

Obtenga más información acerca de [Microsoft Graph API](https://graph.microsoft.com).

#### <a name="multi-account-applications"></a>Aplicaciones de varias cuentas

Esta aplicación se ha creado para un escenario de una única cuenta. MSAL admite también escenarios de varias cuentas, pero requiere algún trabajo adicional por parte de las aplicaciones. Deberá crear una interfaz de usuario para ayudar al usuario a seleccionar qué cuenta desea usar para cada acción que requiera tokens. Como alternativa, la aplicación puede implementar un método heurístico para seleccionar qué cuenta se usará mediante el método `getAccounts(...)`. 

## <a name="test-your-app"></a>Prueba de la aplicación

### <a name="run-locally"></a>Ejecución en modo local

Si ha seguido el código anterior, pruebe a compilar e implementar la aplicación en un emulador o dispositivo de prueba. Debe ser capaz de iniciar sesión y obtener tokens para Azure AD o cuentas personales de Microsoft. Después de que un usuario inicie sesión, esta aplicación mostrará los datos devueltos desde el punto de conexión `/me` de Microsoft Graph. 

Si tiene algún problema, no dude abrir una incidencia en este documento o en la biblioteca MSAL y comunicárnoslo. 

### <a name="consent-to-your-app"></a>Consentimiento para su aplicación

La primera vez que un usuario inicie sesión en su aplicación, Microsoft Identity le pedirá que acepte los permisos solicitados.  Si bien la mayoría de los usuarios pueden dar su consentimiento, algunos inquilinos de Azure AD tienen deshabilitado el consentimiento del usuario, lo que requiere que los administradores den su consentimiento en nombre de todos los usuarios.  Para admitir este escenario, asegúrese de registrar los ámbitos de la aplicación en Azure Portal.

## <a name="help-and-support"></a>Ayuda y soporte técnico

¿Ha tenido algún problema con este tutorial o con la plataforma de identidad de Microsoft? Vea la sección de [ayuda y soporte técnico](https://docs.microsoft.com/azure/active-directory/develop/developer-support-help-options).
