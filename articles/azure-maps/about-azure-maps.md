---
title: Información general de Microsoft Azure Maps
description: Aprenda sobre los servicios y las funcionalidades de Microsoft Azure Maps y cómo usarlos en las aplicaciones.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/31/2020
ms.topic: overview
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc, references_regions
ms.openlocfilehash: 6422874f57b490773f70b6fb27d5b65c654ded07
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212736"
---
# <a name="what-is-azure-maps"></a>¿Qué es Azure Maps?

Azure Maps es una colección de servicios geoespaciales y SDK que emplea datos de mapas recientes para proporcionar contexto geográfico a las aplicaciones web y móviles. Azure Maps ofrece:

* API REST para representar vectores y mapas de trama en varios estilos y en imágenes de satélite.
* Servicios de creador para crear y representar mapas basados en los datos de mapas de interiores privados.
* Servicio Search para buscar direcciones, lugares y puntos de interés en todo el mundo.
* Varias opciones de enrutamiento: punto a punto, multipunto, optimización multipunto, mapas isócronos, vehículo eléctrico, vehículo comercial, con capa de tráfico y enrutamiento de matrices.
* Vista de flujo de tráfico y vista de incidentes para las aplicaciones que requieren información sobre el tráfico en tiempo real.
* Servicio Mobility para solicitar información sobre el transporte público, planear rutas mediante la combinación de diferentes modos de viaje y llegadas en tiempo real.
* Servicios de zona horaria y ubicación geográfica.
* Servicios de geovalla y almacenamiento de datos de mapas, con información de ubicación hospedada en Azure.
* Inteligencia de ubicación mediante análisis geoespacial.

Además, los servicios de Azure Maps están disponibles mediante el SDK web o Android SDK. Estas herramientas ayudan a los desarrolladores a crear y escalar rápidamente soluciones que integran la información de ubicación en las soluciones de Azure.

Puede registrarse para obtener una [cuenta de Azure Maps](https://azure.microsoft.com/services/azure-maps/) gratuita y empezar a desarrollar.

En el siguiente vídeo se explica Azure Maps en profundidad:

<br/>

<iframe src="https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-Maps/player?format=ny" width="960" height="540" allowFullScreen frameBorder="0"></iframe>

## <a name="map-controls"></a>Controles de mapa

### <a name="web-sdk"></a>SDK web

El SDK web de Azure Maps permite personalizar mapas interactivos con contenido propio e imágenes. Puede usar este mapa interactivo para las aplicaciones web o móviles. El control de mapa usa WebGL, que permite representar grandes conjuntos de datos con alto rendimiento. Puede desarrollar con el SDK con JavaScript o TypeScript.

:::image type="content" source="./media/about-azure-maps/intro_web_map_control.png" alt-text="Mapa de ejemplo de cambio de población creado mediante el SDK web de Azure Maps":::

### <a name="android-sdk"></a>SDK de Android

Use Android SDK de Azure Maps para crear aplicaciones de mapas móviles.

:::image type="content" source="./media/about-azure-maps/android_sdk.png" border="false" alt-text="Ejemplos de mapas en un dispositivo móvil":::

## <a name="services-in-azure-maps"></a>Servicios de Azure Maps

Azure Maps consta de los siguientes servicios que pueden proporcionar contexto geográfico a las aplicaciones de Azure.

### <a name="data-service"></a>Servicio de datos

Los datos son imprescindibles en los mapas. Use el servicio Data para cargar y almacenar datos geoespaciales y usarlos con operaciones espaciales o composición de imágenes.  Acercar los datos de los clientes al servicio Azure Maps reducirá la latencia, aumentará la productividad y creará escenarios en sus aplicaciones. Para más información sobre este servicio, consulte la [documentación del servicio Data](https://docs.microsoft.com/rest/api/maps/data).

### <a name="geolocation-service"></a>Servicio de geolocalización

Use el servicio Geolocation para obtener una vista previa del código de país o región de dos letras recuperado de una dirección IP. Este servicio puede ayudarle a mejorar la experiencia del usuario al proporcionar contenido de la aplicación personalizado según la ubicación geográfica.

Para obtener más información, consulte la [documentación del servicio Geolocation](https://docs.microsoft.com/rest/api/maps/geolocation).

### <a name="mobility-service"></a>Mobility Service

El servicio Mobility de Azure Maps mejora el tiempo de desarrollo de las aplicaciones con características de transporte público, como las rutas del transporte y la búsqueda de paradas de transporte público cercanas. Los usuarios pueden recuperar información detallada sobre las paradas del transporte, las líneas y los horarios. El servicio Mobility también permite a los usuarios recuperar las geometrías de las paradas y las líneas, las alertas de las paradas, las líneas y las áreas de servicio, así como las llegadas de transporte público en tiempo real y las alertas del servicio. Además, Mobility Service proporciona funcionalidades de cálculo de ruta con opciones de planeamiento de trayectos mixtos. El planeamiento de trayectos mixtos incluye caminar, bicicleta y transporte público en el mismo viaje. Los usuarios pueden acceder a itinerarios paso a paso detallados y mixtos.

Para más información sobre el servicio, consulte la [documentación del servicio Mobility](https://docs.microsoft.com/rest/api/maps/mobility).

### <a name="render-service"></a>Render Service

El [servicio Render v2](https://docs.microsoft.com/rest/api/maps/renderv2) que se encuentra ahora en versión preliminar presenta una nueva versión de la [API Get Map Tile V2](https://docs.microsoft.com/rest/api/maps/renderv2/getmaptilepreview). Ahora, la API Get Map Tile V2 permite a los clientes solicitar mosaicos de carretera de Azure Maps, mosaicos meteorológicos o los mosaicos de mapa creados con Azure Maps Creator. Se recomienda usar la nueva API Get Map Tile V2.  

:::image type="content" source="./media/about-azure-maps/intro_map.png" border="false" alt-text="Ejemplo de un mapa del servicio Render V2":::

Para más información, lea la [documentación del servicio Render V2](https://docs.microsoft.com/rest/api/maps/renderv2).

Para obtener más información sobre el servicio Render V1 que se encuentra en GA (disponibilidad general), consulte la [documentación del servicio Render V1](https://docs.microsoft.com/rest/api/maps/render).  

### <a name="route-service"></a>Route Service

Los servicios de ruta se pueden usar para calcular las horas de llegada estimadas (ETA) de cada ruta solicitada. Las API Route tienen en cuenta factores como la información del tráfico en tiempo real y los datos de tráfico históricos como son las velocidades de conducción típicas en el día de la semana y la hora del día solicitados. Las API devuelven las rutas más cortas o más rápidas disponibles para varios destinos a la vez de forma secuencial o en orden optimizado, en función de la hora o la distancia. El servicio permite a los desarrolladores calcular las indicaciones entre los distintos modos de viaje, por ejemplo, en automóvil, camión, bicicleta, a pie o en vehículo eléctrico. El servicio también tiene en cuenta datos de entrada como la hora de salida, las restricciones de peso o el transporte de materiales peligrosos.

:::image type="content" source="./media/about-azure-maps/intro_route.png" border="false" alt-text="Ejemplo de un mapa de Route Service":::

El servicio Route ofrece un conjunto de características avanzadas, como:

* Procesamiento por lotes de varias solicitudes de ruta.
* Matrices de tiempo y distancia de viaje entre un conjunto de orígenes y destinos.
* Búsqueda de rutas o distancias que los usuarios pueden recorrer en función de los requisitos de tiempo o combustible.

Para más información sobre las funcionalidades de ruta, lea la [documentación del servicio Route](https://docs.microsoft.com/rest/api/maps/route).

### <a name="search-service"></a>Servicio de búsqueda

El servicio de búsqueda ayuda a los desarrolladores a buscar direcciones, lugares, listados de empresas por nombre o categoría y otra información geográfica. Además, los servicios pueden realizar la [codificación geográfica inversa](https://en.wikipedia.org/wiki/Reverse_geocoding) de direcciones y cruces de calles en función de la longitud y la latitud.

:::image type="content" source="./media/about-azure-maps/intro_search.png" border="false" alt-text="Ejemplo de una búsqueda en un mapa":::

El servicio de búsqueda también proporciona características avanzadas, como:

* Búsqueda a lo largo de una ruta
* Búsqueda dentro de un área más amplia
* Procesamiento por lotes de un grupo de solicitudes de búsqueda
* Búsqueda de estaciones de carga de vehículos eléctricos y datos de punto de interés (POI) por marca.

Para más información sobre las funcionalidades de búsqueda, lea la [documentación del servicio Search](https://docs.microsoft.com/rest/api/maps/search).

### <a name="spatial-service"></a>Servicio espacial

El servicio Spatial analiza rápidamente la información de ubicación para informar a los clientes de eventos en curso que sucedan en un tiempo y un espacio. Permite análisis en tiempo real y el modelado predictivo de los eventos.

El servicio permite a los clientes mejorar su inteligencia de ubicación gracias a una biblioteca de cálculos matemáticos geoespaciales comunes. Los cálculos comunes incluyen el punto más cercano, la distancia ortodrómica y los búferes. Para más información sobre el servicio y las diversas características, lea la [documentación del servicio Spatial](https://docs.microsoft.com/rest/api/maps/spatial).

### <a name="timezone-service"></a>Timezone Service

El servicio Timezone permite consultar la información de zona horaria actual, histórica y futura. Puede usar pares de latitud y longitud o un [identificador de IANA](https://www.iana.org/) como entrada. El servicio Timezone además permite:

* Convertir los identificadores de zona horaria de Microsoft Windows en zonas horarias de IANA
* Recuperar un desplazamiento de zona horaria a UTC
* Obtener la hora actual de una zona horaria seleccionada

Una respuesta JSON típica de una consulta al servicio Time Zone se parece al ejemplo siguiente:

```JSON
{
  "Version": "2020a",
  "ReferenceUtcTimestamp": "2020-07-31T19:15:14.4570053Z",
  "TimeZones": [
    {
      "Id": "America/Los_Angeles",
      "Names": {
        "ISO6391LanguageCode": "en",
        "Generic": "Pacific Time",
        "Standard": "Pacific Standard Time",
        "Daylight": "Pacific Daylight Time"
      },
      "ReferenceTime": {
        "Tag": "PDT",
        "StandardOffset": "-08:00:00",
        "DaylightSavings": "01:00:00",
        "WallTime": "2020-07-31T12:15:14.4570053-07:00",
        "PosixTzValidYear": 2020,
        "PosixTz": "PST+8PDT,M3.2.0,M11.1.0"
      }
    }
  ]
}
```

Para más información sobre este servicio, lea la [documentación del servicio Time Zone](https://docs.microsoft.com/rest/api/maps/timezone).

### <a name="traffic-service"></a>Traffic Service

El servicio Traffic es un conjunto de servicios web que los desarrolladores pueden usar para crear aplicaciones web o móviles que requieren información del tráfico. El servicio proporciona dos tipos de datos:

* Flujo de tráfico: velocidades observadas en tiempo real y duración del viaje de todas las principales carreteras de la red.
* Incidentes de tráfico: una vista actualizada de atascos e incidentes de tráfico en torno a la red de carreteras.

![Ejemplo de un mapa con información de tráfico](media/about-azure-maps/intro_traffic.png)

Para más información, consulte la [documentación del servicio Traffic](https://docs.microsoft.com/rest/api/maps/traffic).

### <a name="weather-service"></a>Servicio meteorológico

El servicio Weather ofrece API que los desarrolladores pueden usar para recuperar información meteorológica de una ubicación determinada. La información contiene detalles como la fecha y la hora de observación, una breve descripción de las condiciones meteorológicas, el icono meteorológico, las marcas de los indicadores de precipitaciones, la temperatura y la velocidad del viento. También se devuelven detalles adicionales como la sensación térmica de RealFeel™ y el índice UV.

Los desarrolladores pueden usar la [API Get Weather Along Route](https://docs.microsoft.com/rest/api/maps/weather/getweatheralongroutepreview) para recuperar información meteorológica a lo largo de una ruta determinada. Además, el servicio admite la generación de notificaciones meteorológicas para puntos de trayecto afectados por las inclemencias del tiempo, como inundaciones o lluvia intensa.

La [API Get Map Tile V2](https://docs.microsoft.com/rest/api/maps/renderv2/getmaptilepreview) permite solicitar los mosaicos de radar y satélite anteriores, actuales y futuros.

![Ejemplo de mapa con mosaicos de radar meteorológicos en tiempo real](media/about-azure-maps/intro_weather.png)

### <a name="maps-creator-service"></a>Servicio Maps Creator

El servicio Maps Creator es un conjunto de servicios web que los desarrolladores pueden usar para crear aplicaciones con características de mapa basadas en datos de mapas de interiores.

Maps Creator proporciona tres servicios básicos:

* [Servicio de conjunto de datos](https://docs.microsoft.com/rest/api/maps/dataset). Use el servicio de conjunto de datos para crear un conjunto de datos a partir de los datos de un paquete de dibujo convertido. Para obtener información sobre los requisitos de los paquetes de dibujo, consulte Requisitos de los paquetes de dibujo.

* [Servicio Conversion](https://docs.microsoft.com/rest/api/maps/dataset). Use el servicio Conversion para convertir un archivo de diseño DWG en datos del paquete de dibujo para mapas de interiores.

* [Servicio de conjunto de mosaicos](https://docs.microsoft.com/rest/api/maps/tileset). Use el servicio de conjunto de mosaicos para crear una representación basada en vectores de un conjunto de datos. Las aplicaciones pueden utilizar un conjunto de mosaicos para presentar una vista visual basada en mosaicos del conjunto de datos.

* [Servicio Feature State](https://docs.microsoft.com/rest/api/maps/featurestate). Use el servicio Feature State para admitir el estilo de mapa dinámico. El estilo de mapa dinámico permite a las aplicaciones reflejar eventos en tiempo real en espacios que los sistemas IoT hayan proporcionado.

* [Servicio WFS](https://docs.microsoft.com/rest/api/maps/featurestate). Use el servicio WFS para consultar los datos de mapas de interiores. El servicio WFS sigue los estándares de la [API Open Geospatial Consortium](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) para consultar un único conjunto de datos.

## <a name="programming-model"></a>Modelo de programación

Azure Maps se ha diseñado pensando en la movilidad y puede ayudarlo a desarrollar aplicaciones multiplataforma. Usa un modelo de programación independiente del lenguaje y admite salidas JSON mediante las [API REST](https://docs.microsoft.com/rest/api/maps/).

Además, Azure Maps ofrece un cómodo [control de mapa de JavaScript](https://docs.microsoft.com/javascript/api/azure-maps-control) con un modelo de programación simple. El desarrollo es rápido y sencillo para aplicaciones web y móviles.

## <a name="power-bi-visual"></a>Objeto visual de Power BI

El objeto visual Azure Maps para Power BI proporciona un amplio conjunto de visualizaciones de datos para los datos espaciales a partir de un mapa. Se calcula que el 80 % de los datos empresariales tienen un contexto de ubicación. El objeto visual Azure Maps ofrece una solución sin código para entender cómo este contexto de ubicación se relaciona e influye en los datos empresariales.

:::image type="content" source="./media/about-azure-maps/intro-power-bi.png" border="false" alt-text="El escritorio de Power BI con el objeto visual Azure Maps que muestra datos empresariales":::

Para más información, consulte la documentación de introducción del [objeto visual Azure Maps Power BI](power-bi-visual-getting-started.md).

## <a name="usage"></a>Uso

Para acceder a los servicios de Azure Maps, vaya a [Azure Portal](https://portal.azure.com) y cree una cuenta de Azure Maps.

Azure Maps usa un esquema de autenticación basado en claves. Al crear la cuenta, se generan dos claves. Para autenticarse en los servicios de Azure Maps, puede usar cualquiera de ellas.

Nota: Azure Maps comparte las consultas de dirección/ubicación proporcionadas por el cliente ("consultas") con el programa de terceros TomTom para la funcionalidad de los mapas. Las consultas no se vinculan a ningún cliente ni usuario final cuando se comparten con TomTom y no se pueden usar para identificar a los usuarios. Los servicios Mobility y Weather, que incluyen la integración con Moovit y AccuWeather, se encuentran actualmente en [VERSIÓN PRELIMINAR](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Actualmente, Microsoft está en proceso de agregar TomTom, Moovit y AccuWeather a la lista de subcontratistas de servicios en línea.

## <a name="supported-regions"></a>Regiones admitidas

Los servicios de Azure Maps están disponibles actualmente, excepto en los siguientes países o regiones:

* China
* Corea del Sur

Verifique que la ubicación de la dirección IP actual está en un país o región admitidos.

## <a name="next-steps"></a>Pasos siguientes

Pruebe una aplicación de ejemplo que presente Azure Maps:

> [!div class="nextstepaction"]
> [Inicio rápido: Creación de una aplicación web](quick-demo-map-app.md)

Manténgase al día con Azure Maps:

> [!div class="nextstepaction"]
> [Blog de Azure Maps](https://azure.microsoft.com/blog/topics/azure-maps/)
