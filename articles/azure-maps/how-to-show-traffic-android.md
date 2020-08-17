---
title: Visualización de los datos de tráfico en un mapa de Android | Microsoft Azure Maps
description: En este artículo aprenderá cómo mostrar los datos de tráfico en un mapa mediante el Android SDK de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/27/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 8b07b520dec4e9149c6748777a119b68e56c65e9
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87126404"
---
# <a name="show-traffic-data-on-the-map-using-azure-maps-android-sdk"></a>Visualización de los datos de tráfico en el mapa mediante el Android SDK de Azure Maps

Los datos de flujo y los datos de incidentes son los dos tipos de datos de tráfico que se pueden mostrar en el mapa. En esta guía se muestra cómo visualizar ambos tipos de datos de tráfico. Los datos de incidentes se componen de datos basados en puntos y líneas para aspectos como construcciones, cierres de carreteras y accidentes. Los datos de flujo proporcionan métricas sobre el flujo de tráfico en la ruta.

## <a name="prerequisites"></a>Prerrequisitos

Antes de poder mostrar el tráfico en el mapa, debe [crear una cuenta de Azure](quick-demo-map-app.md#create-an-azure-maps-account) y [obtener una clave de suscripción](quick-demo-map-app.md#get-the-primary-key-for-your-account). A continuación, debe instalar el [Android SDK de Azure Maps](https://docs.microsoft.com/azure/azure-maps/how-to-use-android-map-control-library) y cargar un mapa.

## <a name="incidents-traffic-data"></a>Datos de tráfico de incidentes

Deberá importar las siguientes bibliotecas para llamar a `setTraffic` y `incidents`:

```java
import static com.microsoft.com.azure.maps.mapcontrol.options.TrafficOptions.incidents;
```

 En el fragmento de código siguiente se muestra cómo visualizar los datos de tráfico en el mapa. Pasamos un valor booleano al método `incidents` y pasamos eso al método `setTraffic`. 

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mapControl.getMapAsync(map - > {
        map.setTraffic(incidents(true));
    }
}
```

## <a name="flow-traffic-data"></a>Datos de tráfico de flujo

Primero deberá importar las siguientes bibliotecas para llamar a `setTraffic` y `flow`:

```java
import com.microsoft.azure.maps.mapcontrol.options.TrafficFlow;
import static com.microsoft.azure.maps.mapcontrol.options.TrafficOptions.flow;
```

Use el fragmento de código siguiente para establecer los datos de flujo de tráfico. De forma similar al código de la sección anterior, pasamos el valor devuelto del método `flow` al método `setTraffic`. Hay cuatro valores que se pueden pasar a `flow`, y cada valor desencadenaría `flow` para devolver el valor correspondiente. El valor devuelto de `flow` se pasará entonces como argumento a `setTraffic`. Consulte la tabla siguiente para estos cuatro valores:

|Valor de flujo | Descripción|
| :-- | :-- |
| TrafficFlow.NONE | No muestra los datos de tráfico en el mapa. |
| TrafficFlow.RELATIVE | Muestra los datos de tráfico relativos a la velocidad de flujo libre de la carretera. |
| TrafficFlow.RELATIVE_DELAY | Muestra las áreas que son más lentas que el retraso promedio esperado. |
| TrafficFlow.ABSOLUTE | Muestra la velocidad absoluta de todos los vehículos en la carretera. |

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mapControl.getMapAsync(map -> {
        map.setTraffic(flow(TrafficFlow.RELATIVE)));
    }
}
```

## <a name="show-incident-traffic-data-by-clicking-a-feature"></a>Visualización de datos de tráfico de incidentes al hacer clic en una característica

Para obtener los incidentes de una característica específica, puede usar el código siguiente. Cuando se hace clic en una característica, la lógica del código comprueba los incidentes y genera un mensaje sobre el incidente. En la parte inferior de la pantalla aparece un mensaje con los detalles.

1. En primer lugar, debe editar **res > layout > activity_main.xml** para que su aspecto sea similar al siguiente. Puede reemplazar `mapcontrol_centerLat`, `mapcontrol_centerLng`y `mapcontrol_zoom` con los valores deseados. Recuerde que el nivel de zoom es un valor comprendido entre 0 y 22. En el nivel de zoom 0, todo el mundo cabe en un solo mosaico.

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
           app:mapcontrol_centerLat="47.6050"
           app:mapcontrol_centerLng="-122.3344"
           app:mapcontrol_zoom="12"
           />

   </FrameLayout>
   ```

2. Agregue el código siguiente al archivo **MainActivity.java**. El paquete se incluye de forma predeterminada, así que debe asegurarse de mantener el paquete en la parte superior.

   ```java
   package <yourpackagename>;
   import androidx.appcompat.app.AppCompatActivity;

   import android.os.Bundle;
   import android.widget.Toast;

   import com.microsoft.azure.maps.mapcontrol.AzureMaps;
   import com.microsoft.azure.maps.mapcontrol.MapControl;
   import com.mapbox.geojson.Feature;
   import com.microsoft.azure.maps.mapcontrol.events.OnFeatureClick;

   import com.microsoft.azure.maps.mapcontrol.options.TrafficFlow;
   import static com.microsoft.azure.maps.mapcontrol.options.TrafficOptions.flow;
   import static com.microsoft.azure.maps.mapcontrol.options.TrafficOptions.incidents;

   public class MainActivity extends AppCompatActivity {

       static {
           AzureMaps.setSubscriptionKey("Your Azure Maps Subscription Key");
       }

       MapControl mapControl;

       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);

           mapControl = findViewById(R.id.mapcontrol);

           mapControl.onCreate(savedInstanceState);

           //Wait until the map resources are ready.
           mapControl.getMapAsync(map -> {

               map.setTraffic(flow(TrafficFlow.RELATIVE));
               map.setTraffic(incidents(true));

               map.events.add((OnFeatureClick) (features) -> {

                   if (features != null && features.size() > 0) {
                       Feature incident = features.get(0);
                       if (incident.properties() != null) {


                           StringBuilder sb = new StringBuilder();
                           String incidentType = incident.getStringProperty("incidentType");
                           if (incidentType != null) {
                               sb.append(incidentType);
                           }
                           if (sb.length() > 0) sb.append("\n");
                           if ("Road Closed".equals(incidentType)) {
                               sb.append(incident.getStringProperty("from"));
                           } else {
                               String description = incident.getStringProperty("description");
                               if (description != null) {
                                   for (String word : description.split(" ")) {
                                       if (word.length() > 0) {
                                           sb.append(word.substring(0, 1).toUpperCase());
                                           if (word.length() > 1) {
                                               sb.append(word.substring(1));
                                           }
                                           sb.append(" ");
                                       }
                                   }
                               }
                           }
                           String message = sb.toString();

                           if (message.length() > 0) {
                               Toast.makeText(this,message,Toast.LENGTH_LONG).show();
                           }
                       }
                   }
               });
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

3. Una vez que incorpore el código anterior en la aplicación, podrá hacer clic en una característica y ver los detalles de los incidentes de tráfico. En función de los valores de latitud, longitud y nivel de zoom que usó en el archivo **activity_main.xml**, verá resultados similares a la siguiente imagen:

   <center>

   ![Incidente-de-tráfico-en-el-mapa](./media/how-to-show-traffic-android/android-traffic.png)

   </center>

## <a name="next-steps"></a>Pasos siguientes

Consulte las guías siguientes para obtener información sobre cómo agregar más datos al mapa:

> [!div class="nextstepaction"]
> [Adición de una capa de símbolo](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Adición de una capa de icono](how-to-add-tile-layer-android-map.md)

> [!div class="nextstepaction"]
> [Adición de formas a un mapa Android](how-to-add-shapes-to-android-map.md)

> [!div class="nextstepaction"]
> [Presentación de la información de características](display-feature-information-android.md)
