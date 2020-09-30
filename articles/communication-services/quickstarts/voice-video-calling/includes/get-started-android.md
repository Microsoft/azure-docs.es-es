---
title: 'Inicio rápido: adición de llamadas VoIP a una aplicación de Android mediante Azure Communication Services'
description: En este inicio rápido, aprenderá a usar la biblioteca cliente de llamadas de Azure Communication Services para Android.
author: matthewrobertson
ms.author: marobert
ms.date: 08/11/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 658aaf018dd4c231b9bc9fc8c6dda78b2a6f05c0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945986"
---
En este inicio rápido, aprenderá a iniciar una llamada mediante la biblioteca cliente de llamadas de Azure Communication Services para Android.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Android Studio](https://developer.android.com/studio), para crear la aplicación de Android.
- Un [token de acceso de usuario](../../access-tokens.md) para su instancia de Azure Communication Services.
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).


## <a name="setting-up"></a>Instalación


### <a name="create-an-android-app-with-an-empty-activity"></a>Creación de una aplicación de Android con una actividad vacía

En Android Studio, seleccione Start a new Android Studio project (Iniciar un nuevo proyecto de Android Studio).

:::image type="content" source="../media/android/studio-new-project.png" alt-text="Captura de pantalla que muestra el botón Start a new Android Studio Project (Iniciar un nuevo proyecto de Android Studio) seleccionado en Android Studio.":::

Seleccione la plantilla de proyecto "Actividad vacía" en "Teléfono y tableta".

:::image type="content" source="../media/android/studio-blank-activity.png" alt-text="Captura de pantalla que muestra la opción Actividad vacía seleccionada en la pantalla de la plantilla de proyecto.":::

Seleccione la biblioteca cliente mínima de la "API 26: Android 8.0 (Oreo)" o una versión posterior.

:::image type="content" source="../media/android/studio-calling-min-api.png" alt-text="Captura de pantalla que muestra la opción Actividad vacía seleccionada en la pantalla 2 de la plantilla de proyecto.":::


### <a name="install-the-package"></a>Instalar el paquete

<!-- TODO: update with instructions on how to download, install and add package to project -->
Busque el nivel de proyecto build.gradle y asegúrese de agregar `mavenCentral()` a la lista de repositorios en `buildscript` y `allprojects`.
```groovy
buildscript {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```

```groovy
allprojects {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```
A continuación, en el nivel de módulo build.gradle, agregue las siguientes líneas a las secciones de dependencias y Android.

```groovy
android {
    ...
    packagingOptions {
        pickFirst  'META-INF/*'
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    ...
    implementation 'com.azure.android:azure-communication-calling:1.0.0-beta.1'
    ...
}
```

### <a name="add-permissions-to-application-manifest"></a>Adición de permisos al manifiesto de aplicación

Para solicitar los permisos necesarios para realizar una llamada, primero se deben declarar en el manifiesto de aplicación (`app/src/main/AndroidManifest.xml`). Reemplace el contenido del archivo por lo siguiente:

```xml
    <?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.contoso.acsquickstart">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <!--Our calling client library depends on the Apache HTTP client library.
When targeting Android client library 28+, this library needs to be explicitly referenced.
See https://developer.android.com/about/versions/pie/android-9.0-changes-28#apache-p-->
        <uses-library android:name="org.apache.http.legacy" android:required="false"/>
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
    
```

### <a name="setup-the-layout-for-the-app"></a>Configuración del diseño de la aplicación

Se necesitan dos entradas: una entrada de texto para el identificador del destinatario y un botón para realizar la llamada. Se pueden agregar a través del diseñador o editando el XML de diseño. Cree un botón con un identificador de `call_button` y una entrada de texto de `callee_id`. Vaya a (`app/src/main/res/layout/activity_main.xml`) y reemplace el contenido del archivo por lo siguiente:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/call_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        android:text="Call"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <EditText
        android:id="@+id/callee_id"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="Callee Id"
        android:inputType="textPersonName"
        app:layout_constraintBottom_toTopOf="@+id/call_button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

### <a name="create-the-main-activity-scaffolding-and-bindings"></a>Creación de los enlaces y scaffolding de actividades principales

Con el diseño creado, se pueden agregar los enlaces, así como el scaffolding básico de la actividad. La actividad administrará la solicitud de los permisos en tiempo de ejecución, la creación del agente de llamadas y la realización de llamadas cuando se presione el botón. Cada proceso se tratará en su propia sección. El método `onCreate` se reemplazará para invocar `getAllPermissions` y `createAgent`, así como para agregar los enlaces para el botón de llamada. Solo se producirá una vez cuando se cree la actividad. Para obtener más información sobre `onCreate`, consulte la guía [Descripción del ciclo de vida de la actividad](https://developer.android.com/guide/components/activities/activity-lifecycle).

Vaya a **MainActivity.java** y reemplace el contenido por el siguiente código:

```java
package com.contoso.acsquickstart;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import com.azure.android.communication.common.CommunicationUser;
import com.azure.android.communication.common.CommunicationUserCredential;
import com.azure.communication.calling.CallAgent;
import com.azure.communication.calling.CallClient;
import com.azure.communication.calling.StartCallOptions;


import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {
    
    private CallAgent callAgent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        getAllPermissions();
        createAgent();
        
        // Bind call button to call `startCall`
        Button callButton = findViewById(R.id.call_button);
        callButton.setOnClickListener(l -> startCall());
    }

    /**
     * Request each required permission if the app doesn't already have it.
     */
    private void getAllPermissions() {
        // See section on requesting permissions
    }

    /**
      * Create the call agent for placing calls
      */
    private void createAgent() {
        // See section on creating the call agent
    }

    /**
     * Place a call to the callee id provided in `callee_id` text input.
     */
    private void startCall() {
        // See section on starting the call
    }
}

```

### <a name="request-permissions-at-runtime"></a>Solicitud de permisos en tiempo de ejecución

Para Android 6.0 y las versiones posteriores (nivel de API 23) y `targetSdkVersion` 23 o las versiones posteriores, los permisos se conceden en tiempo de ejecución en lugar de cuando se instala la aplicación. A fin de admitirlo, se puede implementar `getAllPermissions` para llamar a `ActivityCompat.checkSelfPermission` y `ActivityCompat.requestPermissions` para cada permiso necesario.

```java
/**
 * Request each required permission if the app doesn't already have it.
 */
private void getAllPermissions() {
    String[] requiredPermissions = new String[]{Manifest.permission.RECORD_AUDIO, Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.READ_PHONE_STATE};
    ArrayList<String> permissionsToAskFor = new ArrayList<>();
    for (String permission : requiredPermissions) {
        if (ActivityCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
            permissionsToAskFor.add(permission);
        }
    }
    if (!permissionsToAskFor.isEmpty()) {
        ActivityCompat.requestPermissions(this, permissionsToAskFor.toArray(new String[0]), 1);
    }
}
```

> [!NOTE]
> Al diseñar la aplicación, tenga en cuenta cuándo deben solicitarse estos permisos. Se deben solicitar a medida que sean necesarios, pero no con anterioridad. Para obtener más información, consulte la [guía de permisos de Android](https://developer.android.com/training/permissions/requesting).

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de llamadas de Azure Communication Services:

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient| CallClient es el punto de entrada principal a la biblioteca cliente de llamadas.|
| CallAgent | CallAgent se usa para iniciar y administrar llamadas. |
| CommunicationUserCredential | CommunicationUserCredential se usa como la credencial de token para crear una instancia de CallAgent.|

## <a name="create-an-agent-from-the-user-access-token"></a>Creación de un agente a partir del token de acceso de usuario

Con el token de usuario, se puede crear una instancia del agente de llamadas autenticado. Por lo general, este token se generará a partir de un servicio con autenticación específica de la aplicación. Para obtener más información sobre los tokens de acceso de usuario, consulte la guía [Tokens de acceso de usuario](../../access-tokens.md). En el inicio rápido, reemplace `<User_Access_Token>` por un token de acceso de usuario generado para el recurso de Azure Communication Service.

```java

/**
 * Create the call agent for placing calls
 */
private void createAgent() {
    String userToken = "<User_Access_Token>";

    try {
        CommunicationUserCredential credential = new CommunicationUserCredential(userToken);
        callAgent = new CallClient().createCallAgent(getApplicationContext(), credential).get();
    } catch (Exception ex) {
        Toast.makeText(getApplicationContext(), "Failed to create call agent.", Toast.LENGTH_SHORT).show();
    }
}

```

## <a name="start-a-call-using-the-call-agent"></a>Inicio de una llamada mediante el agente de llamadas

La llamada se puede realizar a través del agente de llamadas, y solo se debe proporcionar una lista de los identificadores de destinatarios y las opciones de llamada. Para el inicio rápido, se usarán las opciones de llamada predeterminadas sin vídeo y un identificador del destinatario único de la entrada de texto.

```java
/**
 * Place a call to the callee id provided in `callee_id` text input.
 */
private void startCall() {
    EditText calleeIdView = findViewById(R.id.callee_id);
    String calleeId = calleeIdView.getText().toString();
    
    StartCallOptions options = new StartCallOptions();

    callAgent.call(
        getApplicationContext(),
        new CommunicationUser[] {new CommunicationUser(calleeId)},
        options);
}
```


## <a name="launch-the-app-and-call-the-echo-bot"></a>Inicio de la aplicación y llamada al bot de eco

Ahora se puede iniciar la aplicación con el botón "Ejecutar aplicación" de la barra de herramientas (Mayús + F10). Para verificar que puede realizar llamadas, llame a `8:echo123`. Se reproducirá un mensaje grabado previamente y, luego, se repetirá su mensaje.

:::image type="content" source="../media/android/quickstart-android-call-echobot.png" alt-text="Captura de pantalla que muestra la aplicación completada.":::
