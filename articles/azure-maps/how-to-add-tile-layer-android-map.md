---
title: Adición de una capa de mosaico a mapas de Android | Microsoft Azure Maps
description: Aprenda a agregar una capa de mosaico a un mapa. Vea un ejemplo en el que se usa Android SDK de Azure Maps para agregar una superposición de radar meteorológico a un mapa.
author: anastasia-ms
ms.author: v-stharr
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: bae1d06f1b5a96ad99d970613d957ce75a0c1393
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037361"
---
# <a name="add-a-tile-layer-to-a-map-using-the-azure-maps-android-sdk"></a>Incorporación de una capa de mosaico a un mapa mediante Android SDK de Azure Maps

En este artículo se muestra cómo representar una capa de mosaico en un mapa mediante Android SDK de Azure Maps. Las capas de mosaico permiten superponer imágenes encima de los mosaicos de mapa base de Azure Maps. Puede encontrar más información en el sistema de mosaico de Azure Maps en la documentación [Niveles de zoom y cuadrícula de mosaico](zoom-levels-and-tile-grid.md).

Una capa de mosaico carga los mosaicos desde un servidor. Estas imágenes pueden representarse previamente y almacenarse como cualquier otra imagen en un servidor, mediante una convención de nomenclatura que la capa de mosaico entienda. O bien, estas imágenes se pueden representar con un servicio dinámico que genere las imágenes casi en tiempo real. Hay tres convenciones diferentes de nomenclatura de servicio de mosaico compatibles con la clase TileLayer de Azure Maps:

* X, Y notación de zoom: según el nivel de zoom, x es la columna e y es la posición de fila del mosaico en la cuadrícula de mosaico.
* Notación Quadkey: combinación de la información de x, y y zoom en un solo valor de cadena que es un identificador único de un mosaico.
* Rectángulo delimitador: las coordenadas del rectángulo delimitador pueden usarse para especificar una imagen con el formato `{west},{south},{east},{north}`, que suele utilizarse en [Web Mapping Services (WMS)](https://www.opengeospatial.org/standards/wms).

> [!TIP]
> Un elemento TileLayer es una excelente manera de visualizar grandes conjuntos de datos en el mapa. No solo puede generarse una capa de mosaico a partir de una imagen, sino que también se pueden representar datos de vector como una capa de mosaico. Con la representación de datos de vectores como una capa de mosaico, el control de mapa solo necesita cargar los mosaicos que pueden tener un tamaño de archivo bastante más reducido que los datos de vector que representan. Esta técnica la usan muchos usuarios que necesitan representar millones de filas de datos en el mapa.

La dirección URL del mosaico pasada a una capa de mosaico debe ser una dirección URL HTTP/HTTPS que apunte a un recurso TileJSON o a una plantilla de URL de mosaico que usa los siguientes parámetros: 

* `{x}`: posición del mosaico en X. También necesita `{y}` y `{z}`.
* `{y}`: posición del mosaico en Y. También necesita `{x}` y `{z}`.
* `{z}`: nivel de zoom del mosaico. También necesita `{x}` y `{y}`.
* `{quadkey}`: identificador quadkey de mosaico basado en la convención de nomenclatura del sistema de mosaico de Bing Maps.
* `{bbox-epsg-3857}`: una cadena de un cuadro delimitador con el formato `{west},{south},{east},{north}` en el sistema de referencia espacial EPSG 3857.
* `{subdomain}`: Marcador de posición para los valores del subdominio, si se especifica el valor de subdominio.

## <a name="prerequisites"></a>Prerrequisitos

Para completar el proceso en este artículo, debe instalar [Android SDK de Azure Maps](https://docs.microsoft.com/azure/azure-maps/how-to-use-android-map-control-library) para cargar un mapa.


## <a name="add-a-tile-layer-to-the-map"></a>Incorporación de una capa de mosaico a un mapa

 En este ejemplo se muestra cómo crear una capa de mosaico que señale a un conjunto de mosaicos. Estos mosaicos usan el sistema de mosaicos "x, y, zoom". La fuente de esta capa de mosaico es una superposición de radar meteorológico de [Iowa Environmental Mesonet of Iowa State University](https://mesonet.agron.iastate.edu/ogc/). 

Puede agregar una capa de mosaico al mapa siguiendo los pasos que se indican a continuación.

1. Edite **res > layout > activity_main.xml** para que su aspecto sea similar al siguiente:

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
            app:mapcontrol_centerLat="40.75"
            app:mapcontrol_centerLng="-99.47"
            app:mapcontrol_zoom="3"
            />
    
    </FrameLayout>
    ```

2. Copie el fragmento de código siguiente en el método **onCreate()** de la clase `MainActivity.java`.

    ```Java
    mapControl.onReady(map -> {
        //Add a tile layer to the map, below the map labels.
        map.layers.add(new TileLayer(
            tileUrl("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png"),
            opacity(0.8f),
            tileSize(256)
        ), "labels");
    });
    ```
    
    En primer lugar, el fragmento de código anterior obtiene una instancia de control de mapas de Azure Maps mediante el método de devolución de llamada **onReady()** . Después crea un objeto `TileLayer` y pasa una dirección URL de mosaico con formato **xyz** a la opción `tileUrl`. La opacidad de la capa se establece en `0.8` y, puesto que los mosaicos del servicio de mosaicos que se usan son de 256 píxeles, esta información se pasa a la opción `tileSize`. A continuación, la capa de mosaico se pasa al administrador de capas de mapas.

    Después de agregar el fragmento de código anterior, su `MainActivity.java` debería tener un aspecto similar al siguiente:
    
    ```Java
    package com.example.myapplication;

    import android.app.Activity;
    import android.os.Bundle;
    import android.support.v7.app.AppCompatActivity;
    import com.microsoft.azure.maps.mapcontrol.layer.TileLayer;
    import java.util.Arrays;
    import java.util.List;
    import com.microsoft.azure.maps.mapcontrol.AzureMaps;
    import com.microsoft.azure.maps.mapcontrol.MapControl;
    import static com.microsoft.azure.maps.mapcontrol.options.TileLayerOptions.tileSize;
    import static com.microsoft.azure.maps.mapcontrol.options.TileLayerOptions.tileUrl;
        
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

                //Add a tile layer to the map, below the map labels.
                map.layers.add(new TileLayer(
                    tileUrl("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png"),
                    opacity(0.8f),
                    tileSize(256)
                ), "labels");
            });    
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

Si ejecuta la aplicación ahora, debe ver una línea en el mapa tal como se muestra a continuación:

<center>

![Línea en un mapa de Android](./media/how-to-add-tile-layer-android-map/xyz-tile-layer-android.png)</center>

## <a name="next-steps"></a>Pasos siguientes

Consulte el siguiente artículo para obtener más información sobre las formas de establecer estilos de mapa

> [!div class="nextstepaction"]
> [Cambio de estilos de mapa en mapas de Android](https://docs.microsoft.com/azure/azure-maps/set-android-map-styles)