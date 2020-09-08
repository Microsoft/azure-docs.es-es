---
title: Introducción al control de mapa de Android | Microsoft Azure Maps
description: Familiarícese con el SDK para Android de Azure Maps. Vea cómo crear un proyecto en Android Studio, instalar el SDK y crear un mapa interactivo.
author: anastasia-ms
ms.author: v-stharr
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: fca1b48911ff2af977b8415ad1d9ec1cc2801623
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037769"
---
# <a name="getting-started-with-azure-maps-android-sdk"></a>Introducción a Android SDK para Azure Maps

Android SDK para Azure Maps es una biblioteca de mapas vectoriales para Android. En este artículo se ofrece orientación sobre los procesos para instalar Android SDK para Azure Maps y cargar un mapa.

## <a name="prerequisites"></a>Prerrequisitos

### <a name="create-an-azure-maps-account"></a>Crear una cuenta de Azure Maps

Para completar los procedimientos descritos en este artículo, primero ha de [crear una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account) en el plan de tarifa S1 y [obtener la clave principal](quick-demo-map-app.md#get-the-primary-key-for-your-account) para la cuenta.

Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](./how-to-manage-authentication.md).

### <a name="download-android-studio"></a>Descargar Android Studio

Descargue Android Studio y cree un proyecto con una actividad vacía antes de instalar Android SDK para Azure Maps. Puede [descargar Android Studio](https://developer.android.com/studio/) gratis desde Google. 

## <a name="create-a-project-in-android-studio"></a>Creación de un proyecto en Android Studio

En primero lugar, cree un proyecto con una actividad vacía. Siga estos pasos para crear un proyecto de Android Studio:

1. Under **Choose your project** (Elija el proyecto), seleccione **Phone and Tablet** (Teléfono y tableta). La aplicación se ejecutará en este factor de forma.
2. En la pestaña **Phone and Tablet** (Teléfono y tableta), seleccione **Empty Activity** (Actividad vacía) y luego **Next** (Siguiente).
3. En **Configure your project** (Configurar proyecto), seleccione `API 21: Android 5.0.0 (Lollipop)` como el SDK mínimo. Se trata de la versión más antigua compatible con Android SDK para Azure Maps.
4. Acepte los valores predeterminados `Activity Name` y `Layout Name` y haga clic en **Finish** (Finalizar).

Consulte la [documentación de Android Studio](https://developer.android.com/studio/intro/) a fin de obtener más ayuda para la instalación de Android Studio y la creación de un proyecto.

![Creación de un proyecto en Android Studio ](./media/how-to-use-android-map-control-library/form-factor-android.png)

## <a name="set-up-a-virtual-device"></a>Configuración de un dispositivo virtual

Android Studio permite configurar un dispositivo virtual Android en el equipo. Esto puede ayudarle a probar la aplicación durante el desarrollo. Para configurar un dispositivo virtual, seleccione el icono del dispositivo Android Virtual Device (AVD) Manager en la esquina superior derecha de la pantalla de proyecto y, a continuación, seleccione **Create Virtual Device** (Crear dispositivo virtual). También puede obtener AVD Manager seleccionando **Tools** (Herramientas) > **Android** > **AVD Manager** en la barra de herramientas. En la categoría **Phones** (Teléfonos), seleccione **Nexus 5X** y haga clic en **Next** (Siguiente).

Puede obtener más información sobre cómo configurar un AVD en la [documentación de Android Studio](https://developer.android.com/studio/run/managing-avds).

![Android Emulator](./media/how-to-use-android-map-control-library/android-emulator.png)

## <a name="install-the-azure-maps-android-sdk"></a>Instalación de Android SDK para Azure Maps

El siguiente paso en la creación de la aplicación es instalar el Android SDK para Azure Maps. Para instalar el SDK, siga estos pasos:

1. Abra el archivo**build.gradle** de nivel superior y agregue el siguiente código a la sección de bloques **all projects**, **repositories** (todos los proyectos, repositorios):

    ```
    maven {
            url "https://atlas.microsoft.com/sdk/android"
    }
    ```

2. Actualice **app/build.gradle** y agréguele el siguiente código:
    
    1. Asegúrese de que el valor de **minSdkVersion** del proyecto es API 21 o superior.

    2. Agregue el siguiente código a la sección Android:

        ```
        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }
        ```
    3. Actualice el bloque de dependencias y agregue una nueva línea de dependencia de implementación para el Android SDK de Azure Maps más reciente:

        ```
        implementation "com.microsoft.azure.maps:mapcontrol:0.4"
        ```
    
    4. Vaya a **Archivo** en la barra de herramientas y haga clic en **Sincronizar proyecto con archivos de Gradle**.
3. Agregue un fragmento de mapa a la actividad principal (recurso \> diseño \> activity\_main.xml):
    
    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <FrameLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        >

        <com.microsoft.azure.maps.mapcontrol.MapControl
            android:id="@+id/mapcontrol"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            />
    </FrameLayout>
    ```

4. En el archivo **MainActivity.java**, tendrá que hacer lo siguiente:
    
    * agregar las importaciones del SDK de Azure Maps
    * establecer la información de autenticación de Azure Maps
    * obtener la instancia del control de mapa en el método **onCreate**

    La configuración de la información de autenticación en la clase `AzureMaps` de forma global mediante los métodos `setSubscriptionKey` o `setAadProperties` hace que no tenga que agregar la información de autenticación en cada vista. 

    El control de mapa contiene sus propios métodos de ciclo de vida para administrar el ciclo de vida de OpenGL de Android. Estos métodos de ciclo de vida deben llamarse directamente desde la actividad contenedora. Para que la aplicación llame correctamente a los métodos de ciclo de vida del control de mapa, debe invalidar los siguientes métodos de ciclo de vida en la actividad que contiene el control de mapa. Además, debe llamar al método de control de mapa correspondiente. 

    * onCreate(Bundle) 
    * onStart() 
    * onResume() 
    * onPause() 
    * onStop() 
    * onDestroy() 
    * onSaveInstanceState(Bundle) 
    * onLowMemory() 

    Edite el **MainActivity.java** como sigue:
    
    ```java
    package com.example.myapplication;

    //For older versions use: import android.support.v7.app.AppCompatActivity; 
    import androidx.appcompat.app.AppCompatActivity;
    import com.microsoft.azure.maps.mapcontrol.AzureMaps;
    import com.microsoft.azure.maps.mapcontrol.MapControl;
    import com.microsoft.azure.maps.mapcontrol.layer.SymbolLayer;
    import com.microsoft.azure.maps.mapcontrol.options.MapStyle;
    import com.microsoft.azure.maps.mapcontrol.source.DataSource;

    public class MainActivity extends AppCompatActivity {
        
        static {
            AzureMaps.setSubscriptionKey("<Your Azure Maps subscription key>");
        }

        MapControl mapControl;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            mapControl = findViewById(R.id.mapcontrol);

            mapControl.onCreate(savedInstanceState);
    
            //Wait until the map resources are ready.
            mapControl.onReady(map -> {
                //Add your post map load code here.
    
            });
        }

        @Override
        public void onResume() {
            super.onResume();
            mapControl.onResume();
        }

        @Override
        protected void onStart(){
            super.onStart();
            mapControl.onStart();
        }

        @Override
        public void onPause() {
            super.onPause();
            mapControl.onPause();
        }

        @Override
        public void onStop() {
            super.onStop();
            mapControl.onStop();
        }

        @Override
        public void onLowMemory() {
            super.onLowMemory();
            mapControl.onLowMemory();
        }

        @Override
        protected void onDestroy() {
            super.onDestroy();
            mapControl.onDestroy();
        }

        @Override
        protected void onSaveInstanceState(Bundle outState) {
            super.onSaveInstanceState(outState);
            mapControl.onSaveInstanceState(outState);
        }
    }
    ```

## <a name="import-classes"></a>Importación de clases

Después de completar los pasos anteriores, probablemente recibirá advertencias de Android Studio acerca de parte del código. Para solucionar estas advertencias, importe las clases a las que se hace referencia en `MainActivity.java`.

Puede importar automáticamente estas clases seleccionando Alt+Entrar (Option + Return en un equipo Mac).

Seleccione el botón de ejecución, como se muestra en el siguiente gráfico (o presione Control+R en un equipo Mac), para compilar la aplicación.

![Haga clic en Ejecutar.](./media/how-to-use-android-map-control-library/run-app.png)

Android Studio tardará unos segundos en crear la aplicación. Una vez finalizada la compilación, puede probar la aplicación en el dispositivo Android emulado. Debería ver un mapa como el siguiente:

<center>

![Azure Maps en la aplicación de Android](./media/how-to-use-android-map-control-library/android-map.png)</center>

## <a name="localizing-the-map"></a>Localización del mapa

El Android SDK Azure Maps proporciona tres formas diferentes de establecer el idioma y la vista regional del mapa. El código siguiente muestra cómo establecer el idioma en francés ("fr-FR") y la vista regional en "auto". 

La primera opción es pasar el lenguaje y ver la información regional en la clase `AzureMaps` mediante los métodos `setLanguage` y `setView` estáticos globalmente. Así se establecerá el idioma y la vista regional predeterminados en todos los controles de Azure Maps cargados en la aplicación.

```Java
static {
    //Set your Azure Maps Key.
    AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");

    //Set the language to be used by Azure Maps.
    AzureMaps.setLanguage("fr-FR");

    //Set the regional view to be used by Azure Maps.
    AzureMaps.setView("auto");
}
```

La segunda opción es usar la información de idioma y de la vista en el XML del control de mapa.

```XML
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_language="fr-FR"
    app:mapcontrol_view="auto"
    />
```

La tercera opción consiste en establecer mediante programación el idioma y la vista regional del mapa mediante el método `setStyle` de Maps. Esta opción puede realizarse en cualquier momento para cambiar el idioma y la vista regional del mapa.

```Java
mapControl.onReady(map -> {
    map.setStyle(StyleOptions.language("fr-FR"));
    map.setStyle(StyleOptions.view("auto"));
});
```

Este es un ejemplo de Azure Maps con el idioma establecido en "fr-FR" y la vista regional establecida en "auto".

<center>

![Imagen del mapa de Azure Maps que muestra las etiquetas en francés](./media/how-to-use-android-map-control-library/android-localization.png)
</center>

[Aquí](supported-languages.md) encontrará una lista completa de los idiomas y vistas regionales admitidos.

## <a name="navigating-the-map"></a>Navegación por el mapa

Hay varias maneras de ampliar, desplazar lateralmente, girar e inclinar el mapa. A continuación se detallan todas las formas de navegar por el mapa.

**Acercamiento/alejamiento del mapa**

- Toque el mapa con dos dedos y acérquelos para alejarse o aléjelos para acercarse.
- Pulse dos veces el mapa para acercarse un nivel.
- Pulse dos veces con dos dedos para alejar el mapa un nivel.
- Pulse dos veces; en la segunda pulsación, mantenga presionado el dedo en el mapa y arrástrelo hacia arriba o hacia abajo para acercar o alejar la imagen, respectivamente.

**Desplazamiento lateral del mapa**

- Toque el mapa y arrastre en cualquier dirección.

**Giro del mapa**

- Toque el mapa con dos dedos y gírelos.

**Inclinación del mapa**

- Toque el mapa con dos dedos y arrástrelos hacia arriba o hacia abajo.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo agregar datos de superposición en el mapa:

> [!div class="nextstepaction"]
> [Add a symbol layer to an Android map](how-to-add-symbol-to-android-map.md)(Adición de una capa de símbolo a un mapa Android)

> [!div class="nextstepaction"]
> [Adición de formas a un mapa Android](https://docs.microsoft.com/azure/azure-maps/how-to-add-shapes-to-android-map)

> [!div class="nextstepaction"]
> [Cambio de estilos de mapa en mapas de Android](https://docs.microsoft.com/azure/azure-maps/set-android-map-styles)
