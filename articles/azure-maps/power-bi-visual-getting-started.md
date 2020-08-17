---
title: Introducción al objeto visual de Azure Maps en Power BI | Microsoft Azure Maps
description: En este artículo, aprenderá a usar el objeto visual de Microsoft Azure Maps para Power BI.
author: rbrundritt
ms.author: richbrun
ms.date: 06/26/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 9f7f44e053cfb26cbf7d897b8a1df1ad2861fd07
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87552006"
---
# <a name="getting-started-with-the-azure-maps-power-bi-visual"></a>Introducción al objeto visual de Azure Maps en Power BI

<Token>**Se aplica a:** ![yes](media/power-bi-visual/yes.png) Servicio de Power BI para ***consumidores*** ![yes](media/power-bi-visual/yes.png) Servicio de Power BI para diseñadores y desarrolladores ![yes](media/power-bi-visual/yes.png) Power BI Desktop ![no](media/power-bi-visual/no.png) Se requiere una licencia Pro o Premium </Token>

En este artículo se muestra cómo usar el objeto visual de Microsoft Azure Maps para Power BI.

> [!NOTE]
> Este objeto visual se puede crear y ver tanto en Power BI Desktop como en el servicio de Power BI. Los pasos e ilustraciones de este artículo corresponden a Power BI Desktop.

El objeto visual Azure Maps para Power BI proporciona un amplio conjunto de visualizaciones de datos para los datos espaciales a partir de un mapa. Se calcula que el 80 % de los datos empresariales tienen un contexto de ubicación. El objeto visual Azure Maps puede usarse para obtener conclusiones sobre cómo este contexto de ubicación se relaciona con los datos empresariales e influye en estos.

![Power BI Desktop con el objeto visual Azure Maps que muestra datos empresariales](media/power-bi-visual/azure-maps-visual-hero.png)

## <a name="what-is-sent-to-azure"></a>¿Qué se envía a Azure?

El objeto visual de Azure Maps se conecta al servicio en la nube hospedado en Azure para recuperar los datos de ubicación, como las imágenes y coordenadas de un mapa, que se usan para crear la visualización del mapa. 

-   Los detalles sobre el área en la que se enfoca el mapa se envían a Azure para recuperar las imágenes necesarias para representar el lienzo del mapa (también conocido como mosaicos de mapa). 
-   Los datos de los cubos Ubicación, Latitud y Longitud se pueden enviar a Azure para recuperar las coordenadas del mapa (proceso denominado geocodificación). 
-   Se pueden recopilar datos de telemetría sobre el mantenimiento del elemento visual (p. ej., informes de bloqueo) si la opción de telemetría de Power BI está habilitada.

Aparte de los escenarios descritos anteriormente, no se envía ningún otro dato superpuesto en el mapa a los servidores de Azure Maps. Toda la representación de los datos se produce localmente en el cliente.

Usted o su administrador pueden tener que actualizar el firewall para permitir el acceso a la plataforma de Azure Maps que usa la siguiente dirección URL.

> `https://atlas.microsoft.com`

Para obtener más información sobre la privacidad y las condiciones de uso en relación con el objeto visual de Azure Maps, consulte [Información legal de Microsoft Azure](https://azure.microsoft.com/support/legal/).

## <a name="preview-behavior-and-requirements"></a>Comportamiento y requisitos de la versión preliminar

Existen algunas consideraciones y requisitos para esta versión preliminar de **Azure Maps**:

-   El objeto visual de **Azure Maps** se encuentra en versión preliminar y debe habilitarse en Power BI Desktop. Para habilitar **Azure Maps**, seleccione **Archivo** &gt; **Opciones y configuración** &gt; **Opciones** &gt; **Características de versión preliminar** y, a continuación, seleccione la casilla **Azure Maps Visual** (Objeto visual de Azure Maps). Si el objeto visual de Azure Maps no está disponible después de hacer esto, es probable que sea necesario habilitar un conmutador de administrador de inquilinos en el portal de administración.
-   El conjunto de datos debe tener campos que contengan información de **latitud** y **longitud**. La geocodificación de los campos de ubicación se agregará en una actualización futura.
-   Actualmente, el control de leyendas integrado para Power BI no aparece en esta versión preliminar. Se agregará en una actualización futura.

## <a name="use-the-azure-maps-visual"></a>Uso del objeto visual de Azure Maps

Una vez que el objeto visual de **Azure Maps** esté habilitado, seleccione el icono **Azure Maps** en el panel **Visualizaciones**.

![Botón de objeto visual de Azure Maps en el panel Visualizaciones](media/power-bi-visual/azure-maps-in-visualizations-pane.png)

Power BI crea un lienzo de diseño de objeto visual de Azure Maps vacío. En versión preliminar, se muestra una declinación de responsabilidades adicional.

![Power BI Desktop con el objeto visual de Azure Maps cargado en estado inicial](media/power-bi-visual/visual-initial-load.png)

Siga los pasos a continuación para cargar el objeto visual de Azure Maps:

1.  En el panel **Campos**, arrastre los campos de datos que contengan la información de las coordenadas de latitud y longitud a los cubos **Latitud** o **Longitud**. Estos son los datos mínimos necesarios para cargar el objeto visual de Azure Maps.
    
    > [!div class="mx-imgBorder"]
    > ![Objeto visual de Azure Maps que muestra puntos como burbujas en el mapa después de indicar los campos de latitud y longitud](media/power-bi-visual/bubble-layer.png)

2.  Para colorear los datos en función de la categorización, arrastre un campo de categorías al cubo **Leyenda** del panel **Campos**. En este ejemplo, vamos a usar la columna **AdminDistrict** (también conocida como estado o provincia).  
    
    > [!div class="mx-imgBorder"]
    > ![Objeto visual de Azure Maps que muestra puntos como burbujas con color en el mapa después de indicar el campo de leyenda](media/power-bi-visual/bubble-layer-with-legend-color.png)

    > [!NOTE]
    > Actualmente, el control de leyendas integrado para Power BI no aparece en esta versión preliminar. Se agregará en una actualización futura.

3.  Para escalar los datos relativamente, arrastre una medida hasta el cubo **Tamaño** del panel **Campos**. En este ejemplo, se usa la columna **Sales** (Ventas).  
    
    > [!div class="mx-imgBorder"]
    > ![Objeto visual de Azure Maps que muestra puntos como burbujas con color y con escalas en el mapa después de indicar el campo de tamaño.](media/power-bi-visual/bubble-layer-with-legend-color-and-size.png)

4.  Utilice las opciones del panel **Formato** para personalizar cómo se representan los datos. La siguiente imagen es el mismo mapa que el anterior, pero con la opción de transparencia de relleno de las capas de burbuja establecida en el 50 %, y la opción de contorno de contraste alto habilitada.  
    
    > [!div class="mx-imgBorder"]
    > ![Objeto visual de Azure Maps que muestran puntos como burbujas en el mapa con un estilo personalizado](media/power-bi-visual/bubble-layer-styled.png)

## <a name="fields-pane-buckets"></a>Cubos del panel Campos

Los cubos de datos siguientes están disponibles en el panel **Campos** del objeto visual de Azure Maps.

| Campo     | Descripción  |
|-----------|--------------|
| Latitud  | Campo que se usa para especificar el valor de latitud de los puntos de datos. Los valores de latitud deben estar entre -90 y 90 en formato de grados decimales.  |
| Longitud | Campo que se usa para especificar el valor de longitud de los puntos de datos. Los valores de longitud deben estar entre -180 y 180 en formato de grados decimales.  |
| Leyenda    | Campo que se usa para clasificar los datos y asignar un color único a los puntos de datos de cada categoría. Cuando se rellene este cubo, aparecerá una sección **Colores de datos** en el panel **Formato** que permite realizar ajustes en los colores. |
| Size      | Medida usada para el tamaño relativo de los puntos de datos en el mapa.   |
| Información sobre herramientas  | Campos de datos adicionales que se muestran en la información sobre herramientas cuando se pasa el mouse sobre las formas. |

## <a name="map-settings"></a>Configuración del mapa

La sección **Configuración del mapa** del panel Formato proporciona opciones para personalizar cómo se muestra la asignación y cómo reacciona a las actualizaciones.

| Configuración             | Descripción  |
|---------------------|--------------|
| Ampliar automáticamente           | Amplía automáticamente en el mapa los datos cargados a través del panel **Campos** del objeto visual. A medida que cambian los datos, el mapa actualiza su posición en consecuencia. Cuando el control deslizante está en la posición **Desactivado**, se muestra la configuración de la vista de mapa adicional para la vista de mapa predeterminada. |
| World wrap (Envolvente mundial)          | Permite al usuario desplazar el mapa horizontalmente de forma infinita. |
| Selector de estilos        | Agrega un botón al mapa para permitir a los lectores de informes cambiar el estilo del mapa. |
| Controles de navegación | Agrega botones al mapa como otro método para permitir que los lectores de informes acerquen, alejen, roten y cambien la inclinación del mapa. Para obtener más información, consulte este documento sobre la [Navegación por el mapa](map-accessibility.md#navigating-the-map) para obtener detalles sobre las distintas formas en que los usuarios pueden navegar por el mapa. |
| Estilo del mapa           | Estilo del mapa. Para obtener más información, consulte este documento sobre los [estilos de mapa admitidos](supported-map-styles.md). |

### <a name="map-view-settings"></a>Configuración de la vista de mapa

Si el control deslizante **Ampliar automáticamente** está en la posición **Desactivado**, se muestra la siguiente configuración, que permite al usuario especificar la información de la vista de mapa predeterminada.

| Configuración          | Descripción   |
|------------------|---------------|
| Zoom             | Nivel de zoom predeterminado del mapa. Puede ser un número entre 0 y 22. |
| Latitud del centro  | Latitud predeterminada en el centro del mapa. |
| Longitud del centro | Longitud predeterminada en el centro del mapa. |
| Dirección          | Orientación predeterminada del mapa en grados, donde 0 indica el norte; 90, el este; 180, el sur; y 270, el oeste. Puede ser cualquier número entre 0 y 360. |
| Inclinación            | Inclinación predeterminada del mapa en grados, de 0 a 60; 0 indica que el mapa se muestra desde arriba. |

## <a name="considerations-and-limitations"></a>Consideraciones y limitaciones

El objeto visual de Azure Maps está disponible en los siguientes servicios y aplicaciones:

| Servicio/aplicación                              | Disponibilidad |
|------------------------------------------|--------------|
| Power BI Desktop                         | Sí          |
| Servicio Power BI (app.powerbi.com)       | Sí          |
| Aplicaciones móviles de Power BI             | Sí          |
| Publicar en la web desde Power BI                  | No           |
| Power BI Embedded                        | No           |
| Inserción del servicio Power BI (PowerBI.com) | Sí          |

La compatibilidad con aplicaciones o servicios de Power BI adicionales se agregará en futuras actualizaciones.

**¿Dónde está disponible Azure Maps?**

En este momento, Azure Maps está disponibles en todos los países y regiones, excepto los siguientes:

- China
- Corea del Sur

Para obtener detalles de cobertura para los diferentes servicios de Azure Maps que permiten este objeto visual, consulte el documento [Información sobre cobertura geográfica](geographic-coverage.md).

**¿Qué exploradores web son compatibles con el objeto visual de Azure Maps?**

Consulte esta documentación para obtener información sobre los [Exploradores admitidos por el SDK web de Azure Maps](supported-browsers.md).

**¿Cuántos puntos de datos se pueden visualizar?**

Este objeto visual admite hasta 30 000 puntos de datos.

**¿Pueden usarse direcciones u otras cadenas de ubicación en este objeto visual?**

La versión preliminar inicial de este objeto visual solo admite valores de latitud y longitud en grados decimales. En una futura actualización se agregará compatibilidad para las direcciones y otras cadenas de ubicación.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el objeto visual de Azure Maps en Power BI:

> [!div class="nextstepaction"]
> [Información sobre las capas en el objeto visual de Azure Maps para Power BI](power-bi-visual-understanding-layers.md)

> [!div class="nextstepaction"]
> [Administración del objeto visual de Azure Maps dentro de la organización](power-bi-visual-manage-access.md)

Personalización del objeto visual:

> [!div class="nextstepaction"]
> [Sugerencias y trucos para el formato de color en Power BI](https://docs.microsoft.com/power-bi/visuals/service-tips-and-tricks-for-color-formatting)

> [!div class="nextstepaction"]
> [Personalización de los títulos, los fondos y las leyendas de las visualizaciones](https://docs.microsoft.com/power-bi/visuals/power-bi-visualization-customize-title-background-and-legend)
