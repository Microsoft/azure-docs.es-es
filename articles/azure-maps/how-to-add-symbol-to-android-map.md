---
title: Adición de una capa de símbolo a mapas de Android | Microsoft Azure Maps
description: En este artículo, obtendrá información sobre cómo representar datos de punto en un mapa agregando una capa de símbolo a este mediante Android SDK de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 528c5a5ba4314e44334548334e312d1111768cf5
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87127084"
---
# <a name="add-a-symbol-layer-to-a-map-using-azure-maps-android-sdk"></a>Adición de una capa de símbolo a un mapa mediante Android SDK de Azure Maps

En este artículo se explica cómo representar datos de punto de un origen de datos como una capa de símbolos en un mapa mediante el Android SDK de Azure Maps.

## <a name="prerequisites"></a>Requisitos previos

Para seguir todos los pasos en este artículo, debe instalar [Android SDK de Azure Maps](https://docs.microsoft.com/azure/azure-maps/how-to-use-android-map-control-library) para cargar un mapa.

## <a name="add-a-symbol-layer"></a>Adición de una capa de símbolo

Para agregar un marcador en un mapa mediante la capa de símbolos, siga estos pasos:

1. Edite **res** > **layout** > **activity_main.xml** para que su aspecto sea similar al del siguiente XML:
    
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
            app:mapcontrol_centerLat="47.64"
            app:mapcontrol_centerLng="-122.33"
            app:mapcontrol_zoom="12"
            />

    </FrameLayout>
    ```

2. Copie el siguiente fragmento de código en el método **onCreate()** de la clase `MainActivity.java`.

    ```Java
    mapControl.onReady(map -> {
    
        //Create a data source and add it to the map.
        DataSource dataSource = new DataSource();
        map.sources.add(dataSource);
    
        //Create a point feature and add it to the data source.
        dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64)));
    
        //Add a custom image icon to the map resources.
        map.images.add("my-icon", R.drawable.mapcontrol_marker_red);
    
        //Create a symbol layer and add it to the map.
        map.layers.add(new SymbolLayer(dataSource,
            iconImage("my-icon")));
        });
    
    ```
    
    En primer lugar, el fragmento de código anterior obtiene una instancia de control de mapas de Azure Maps mediante el método de devolución de llamada **onReady()** . A continuación, se crea un objeto de origen de datos mediante la clase **DataSource** y se agrega al mapa. Después, le agrega un elemento **Feature** que contiene una geometría de Point. A continuación, se coloca una imagen de marcador rojo como icono para el símbolo. Una **capa de símbolos** usa texto o iconos para representar los datos basados en puntos encapsulados en el origen de datos como símbolos en el mapa. A continuación, se crea una capa de símbolos y recibe el origen de datos para representarlo. Después, dicha capa se agrega a las capas del mapa.
    
    Después de agregar el fragmento de código anterior, su `MainActivity.java` debería tener un aspecto similar al siguiente:
    
    ```Java
    package com.example.myapplication;
    
    import android.app.Activity;
    import android.os.Bundle;
    import com.mapbox.geojson.Feature;
    import com.mapbox.geojson.Point;
    import com.microsoft.azure.maps.mapcontrol.AzureMaps;
    import com.microsoft.azure.maps.mapcontrol.MapControl;
    import com.microsoft.azure.maps.mapcontrol.layer.SymbolLayer;
    import com.microsoft.azure.maps.mapcontrol.source.DataSource;
    import static com.microsoft.azure.maps.mapcontrol.options.SymbolLayerOptions.iconImage;
    public class MainActivity extends AppCompatActivity {
        
        static{
                AzureMaps.setSubscriptionKey("<Your Azure Maps subscription key>");
            }
    
        MapControl mapControl;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            mapControl = findViewById(R.id.mapcontrol);
    
            mapControl.onCreate(savedInstanceState);
    
            mapControl.onReady(map -> {
    
                //Create a data source and add it to the map.
                DataSource dataSource = new DataSource();
                map.sources.add(dataSource);
            
                //Create a point feature and add it to the data source.
                dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64)));
            
                //Add a custom image icon to the map resources.
                map.images.add("my-icon", R.drawable.mapcontrol_marker_red);
            
                //Create a symbol layer and add it to the map.
                map.layers.add(new SymbolLayer(dataSource,
                    iconImage("my-icon")));
            });
        }
    
        @Override
        public void onStart() {
            super.onStart();
            mapControl.onStart();
        }
    
        @Override
        public void onResume() {
            super.onResume();
            mapControl.onResume();
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
    
En este momento, si ejecuta la aplicación debería ver un marcador en el mapa como se muestra a continuación:

<center>

![Marcador en el mapa Android](./media/how-to-add-symbol-to-android-map/android-map-pin.png)</center>

> [!TIP]
> De forma predeterminada, las capas de símbolos optimizan la representación de los símbolos, ya que ocultan los símbolos que se superponen. A medida que se acerca el zoom, los símbolos ocultos se hacen visibles. Para deshabilitar esta característica y representar todos los símbolos en todo momento, establezca la opción `iconAllowOverlap` en `true`.

## <a name="next-steps"></a>Pasos siguientes

Para agregar más material al mapa, consulte:

> [!div class="nextstepaction"]
> [Adición de formas a un mapa Android](https://docs.microsoft.com/azure/azure-maps/how-to-add-shapes-to-android-map)

> [!div class="nextstepaction"]
> [Presentación de la información de características](display-feature-information-android.md)