---
title: Establecimiento de un estilo de mapa mediante Android SDK de Azure Maps | Microsoft Azure Maps
description: En este artículo, se aprenderán las funcionalidades relacionadas con el estilo de Microsoft Azure Maps para Android SDK.
author: anastasia-ms
ms.author: v-stharr
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 81f89a4b5aa758d506ecdb9cf7b4f510e333ef25
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131266"
---
# <a name="set-map-style-using-azure-maps-android-sdk"></a>Establecimiento del estilo de mapa mediante Android SDK de Azure Maps

En este artículo se muestran dos maneras de establecer estilos de mapa mediante Android SDK de Azure Maps. Azure Maps tiene cuatro estilos de mapas entre los que puede elegir. Para obtener más información sobre los estilos de mapa compatibles, consulte [Estilos de mapa admitidos en Azure Maps](./supported-map-styles.md).


## <a name="prerequisites"></a>Prerequisites

Para completar el proceso en este artículo, debe instalar [Android SDK de Azure Maps](https://docs.microsoft.com/azure/azure-maps/how-to-use-android-map-control-library) para cargar un mapa.


## <a name="set-map-style-in-the-layout"></a>Establecimiento del estilo del mapa en el diseño

Puede establecer un estilo de mapa en el archivo de diseño de su clase de actividad. Edite **res > layout > activity_main.xml**, para que su aspecto sea similar al siguiente:

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
        app:mapcontrol_centerLat="47.602806"
        app:mapcontrol_centerLng="-122.329330"
        app:mapcontrol_zoom="12"
        app:mapcontrol_style="grayscale_dark"
        />

</FrameLayout>
```

El atributo `mapcontrol_style` anterior establece el estilo de mapa en **grayscale_dark**. 

<center>

![style-grayscale_dark](./media/set-android-map-styles/grayscale-dark.png)</center>

## <a name="set-map-style-in-the-activity-class"></a>Establecimiento del estilo del mapa en la clase de actividad

El estilo del mapa se puede establecer en la clase de actividad. Copie el siguiente fragmento de código en el método **onCreate()** de la clase `MainActivity.java`. Este código establecerá el estilo del mapa en **satellite_road_labels**.

```Java
mapControl.onReady(map -> {
    //Set the camera of the map.
    map.setCamera(center(47.64, -122.33), zoom(14));

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE));
});
```

<center>

![style-satellite-road-labels](./media/set-android-map-styles/satellite-road-labels.png)</center>