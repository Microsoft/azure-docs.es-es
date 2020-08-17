---
title: Incorporación de una capa de mapa de Open Geospatial Consortium (OGC) | Microsoft Azure Maps
description: Obtenga información sobre cómo superponer una capa de mapa OGC en el mapa y cómo usar las diferentes opciones de la clase OgcMapLayer.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/02/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 6efc6b27090ecc7171bb66deb303a4764d9b6f04
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87128563"
---
# <a name="add-a-map-layer-from-the-open-geospatial-consortium-ogc"></a>Incorporación de una capa de mapa desde Open Geospatial Consortium (OGC)

La clase `atlas.layer.OgcMapLayer` puede superponer imágenes de Web Map Services (WMS) y de Web Map Tile Services (WMTS) en el mapa. WMS es un protocolo estándar desarrollado por OGC para servir imágenes de mapa georreferenciadas a través de Internet. La georreferenciación de imágenes es el proceso de asociar una imagen a una ubicación geográfica. WMTS es también un protocolo estándar desarrollado por OGC. Está diseñado para servir iconos de mapa representados previamente y con iconos de mapas con georreferencia.

En las secciones siguientes se describen las características del servicio de mapas web que son compatibles con la clase `OgcMapLayer`.

**Web Map Service (WMS)**

- Versiones compatibles: `1.0.0`, `1.1.0`, `1.1.1` y `1.3.0`
- El servicio debe ser compatible con el sistema de proyección de `EPSG:3857` o controlar las reproyecciones.
- GetFeatureInfo requiere que el servicio admita `EPSG:4326` o controle las reproyecciones. 
- Operaciones compatibles:

    | Operación | Descripción |
    | :-- | :-- |
    | GetCapabilities | Recupera los metadatos sobre el servicio con las funciones admitidas. |
    | GetMap | Recupera una imagen de mapa para una región especificada. |
    | GetFeatureInfo | Recupera `feature_info`, que contiene los datos subyacentes de la característica. |

**Web Map Tile Service (WMTS)**

- Versiones admitidas: `1.0.0`
- Los mosaicos deben ser cuadrados, de modo que `TileWidth == TileHeight`.
- CRS compatible: `EPSG:3857` o `GoogleMapsCompatible` 
- El identificador TileMatrix debe ser un valor entero que se corresponda con un nivel de zoom en el mapa. En un mapa de Azure, el nivel de zoom es un valor entre `"0"` y `"22"`. Por lo tanto, `"0"` es compatible, pero `"00"` no.
- Operaciones compatibles:

    | Operación | Descripción |
    | :-- | :-- |
    | GetCapabilities | Recupera las operaciones y características admitidas |
    | GetTile | Recupera imágenes para un mosaico determinado |

## <a name="overlay-an-ogc-map-layer"></a>Superposición de una capa de mapa de OGC

La `url` puede ser la dirección URL base para el servicio o una dirección URL completa con la consulta para obtener las capacidades del servicio. Según los detalles proporcionados, el cliente de WFS puede probar varios formatos de dirección URL estándar para determinar cómo acceder al servicio inicialmente.

En el código siguiente se muestra cómo superponer una capa de mapa de OGC en el mapa.

<br/>

<iframe height='700' scrolling='no' title='Ejemplo de capa de mapa de OGC' src='//codepen.io/azuremaps/embed/xxGLZWB/?height=700&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz <a href='https://codepen.io/azuremaps/pen/xxGLZWB/'>Ejemplo de capa de mapa de OGC</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="ogc-map-layer-options"></a>Opciones de la capa de mapa de OGC

En el ejemplo siguiente se muestran las diferentes opciones de la capa de mapa de OGC. Puede hacer clic en el botón de lápiz de código situado en la esquina superior derecha para editar el lápiz de código.

<br/>

<iframe height='700' scrolling='no' title='Opciones de la capa de mapa de OGC' src='//codepen.io/azuremaps/embed/abOyEVQ/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el lápiz <a href='https://codepen.io/azuremaps/pen/abOyEVQ/'>Opciones de la capa de mapa de OGC</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="ogc-web-map-service-explorer"></a>Explorador de Web Map Service de OGC

La siguiente herramienta superpone las imágenes de Web Map Services (WMS) y de Web Map Tile Services (WMTS) como capas. Puede seleccionar qué capas del servicio se representan en el mapa. También puede ver las leyendas asociadas para estas capas.

<br/>

<iframe height='750' style='width: 100%;' scrolling='no' title='Explorador de Web Map Service de OGC' src='//codepen.io/azuremaps/embed/YzXxYdX/?height=750&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz <a href='https://codepen.io/azuremaps/pen/YzXxYdX/'>Explorador de Web Map Service de OGC</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

También puede especificar la configuración de mapa para usar un servicio de proxy. El servicio de proxy le permite cargar recursos que se hospedan en dominios que no tienen CORS habilitado.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [OgcMapLayer](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.layer.ogcmaplayer)

> [!div class="nextstepaction"]
> [OgcMapLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.ogcmaplayeroptions)

Vea los siguientes artículos, que contienen ejemplos de código que puede agregar a sus mapas:

> [!div class="nextstepaction"]
> [Conexión a un servicio WFS](spatial-io-connect-wfs-service.md)

> [!div class="nextstepaction"]
> [Aprovechamiento de las operaciones básicas](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Detalles de formatos de datos admitidos](spatial-io-supported-data-format-details.md)
