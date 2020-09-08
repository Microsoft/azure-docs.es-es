---
title: Requisitos de paquetes de dibujos en Azure Maps Creator
description: Obtenga información sobre los requisitos de los paquetes de dibujos para convertir los archivos de diseño de las instalaciones a datos de mapa mediante el servicio de conversión de Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 6/12/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philMea
ms.openlocfilehash: af7238ca4229bac678061c742f13953299a96ba4
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89290028"
---
# <a name="drawing-package-requirements"></a>Requisitos de paquetes de dibujos

El servicio de conversión de [Azure Maps](https://docs.microsoft.com/rest/api/maps/conversion) permite convertir paquetes de dibujo cargados en datos de mapa. En este artículo se describen los requisitos de los paquetes de dibujos para la API de conversión. Para ver un paquete de ejemplo, puede descargar el [paquete de dibujos](https://github.com/Azure-Samples/am-creator-indoor-data-examples) de ejemplo.

## <a name="prerequisites"></a>Requisitos previos

El paquete de dibujos incluye dibujos guardados en formato DWG, que es el formato de archivo nativo del software AutoCAD® de Autodesk, [marca comercial de Autodesk, Inc](https://www.autodesk.com/company/legal-notices-trademarks/trademarks/guidelines-for-use#section12).

Puede elegir cualquier software de CAD para generar los dibujos en el paquete de dibujos.  

El [servicio de conversión de Azure Maps](https://docs.microsoft.com/rest/api/maps/conversion) convierte un paquete de dibujos en datos de mapa.  El servicio de conversión se ha desarrollado y probado con el formato de archivo DWG de AutoCAD. `AC1032` es la versión de formato interno para los archivos DWG. Se recomienda seleccionar `AC1032` para la versión del formato de archivo DWG interno.  

Glosario de términos usados en este documento.

| Término  | Definición |
|:-------|:------------|
| Nivel | Una capa de DWG de AutoCAD.|
| Nivel | Área de un edificio en una elevación determinada. Por ejemplo, la planta de un edificio. |
| Xref  |Archivo en formato de archivo DWG de AutoCAD (.dwg) adjunto al dibujo principal como referencia externa.  |
| Característica | Objeto que combina una geometría con información adicional sobre metadatos. |
| Clases de características | Proyecto común para las características. Por ejemplo, una unidad es una clase de característica y una oficina es una característica. |

## <a name="drawing-package-structure"></a>Estructura de un paquete de dibujos

Un paquete de dibujos es un archivo .zip que contiene los archivos siguientes:

* Archivos DWG en formato de archivo DWG de AutoCAD.
* Un archivo _manifest.json_ para una única instalación.

Los archivos DWG se pueden organizar de cualquier manera dentro de la carpeta, pero el archivo de manifiesto debe residir en el directorio raíz de la carpeta. La carpeta debe comprimirse en un solo archivo de almacenamiento, con una extensión .zip. En las secciones siguientes se detallan los requisitos para los archivos DWG, el archivo de manifiesto y el contenido de estos archivos.  

## <a name="dwg-files-requirements"></a>Requisitos de los archivos DWG

Se requiere un único archivo DWG para cada nivel de la instalación. Los datos del nivel deben formar parte de un único archivo DWG. Cualquier referencia externa (_xref_) debe estar enlazada al dibujo principal. Además, cada archivo DWG:

* Debe definir las capas de _exterior_ y de _unidades_. Opcionalmente, puede definir las siguientes capas opcionales: _Pared_, _Puerta_, _UnitLabel_, _Zona_y _ZoneLabel_.
* No debe contener características de varios niveles.
* No debe contener características de varias instalaciones.

El [servicio de conversión de Azure Maps](https://docs.microsoft.com/rest/api/maps/conversion) puede extraer las siguientes clases de características de un archivo DWG:

* Niveles
* Unidades
* Zones
* Aperturas
* Paredes
* Penetraciones verticales

Todos los trabajos de conversión dan como resultado un conjunto mínimo de categorías predeterminadas: room, structure.wall, opening.door, zone y facility. Las categorías adicionales son para cada nombre de categoría al que hacen referencia los objetos.  

Una capa de DWG debe contener características de una sola clase. Las clases no deben compartir una capa. Por ejemplo, las unidades y las paredes no pueden compartir una capa.

Las capas de DWG también deben seguir los siguientes criterios:

* Los orígenes de los dibujos de todos los archivos DWG deben alinearse con la misma latitud y longitud.
* Cada nivel debe tener la misma orientación que los demás niveles.
* Los polígonos que formen intersección con ellos mismos se repararán automáticamente y el [servicio de conversión de Azure Maps](https://docs.microsoft.com/rest/api/maps/conversion) generará una advertencia. Se recomienda inspeccionar manualmente los resultados reparados, ya que es posible que no coincidan con los resultados esperados.

Todas las entidades de las capas deben ser de uno de los siguientes tipos: Línea, polilínea, polígono, arco circular, círculo, texto (una sola línea). Se omitirán todos los demás tipos de entidad.

En la tabla siguiente se describen los tipos de entidad admitidos y las características admitidas para cada capa. Si una capa contiene tipos de entidad no admitidos, el [servicio de conversión de Azure Maps](https://docs.microsoft.com/rest/api/maps/conversion) omitirá las entidades.  

| Nivel | Tipos de entidades | Características |
| :----- | :-------------------| :-------
| [Exterior](#exterior-layer) | Polígono, polilínea (cerrada), círculo | Niveles
| [Unidad](#unit-layer) |  Polígono, polilínea (cerrada), círculo | Penetraciones verticales, unidades
| [Pared](#wall-layer)  | Polígono, polilínea (cerrada), círculo | No aplicable. Para más información, consulte la [capa de paredes](#wall-layer).
| [Puerta](#door-layer) | Polígono, polilínea, línea, arco circular, círculo | Aperturas
| [Zona](#zone-layer) | Polígono, polilínea (cerrada), círculo | Zona
| [UnitLabel](#unitlabel-layer) | Texto (una línea) | No aplicable. Esta capa solo puede agregar propiedades a las características de las unidades de la capa de unidades. Para más información, consulte la [capa de unitLabel](#unitlabel-layer).
| [ZoneLabel](#zonelabel-layer) | Texto (una línea) | No aplicable. Esta capa solo puede agregar propiedades a las características de zona de la capa de unidades. Para más información, consulte la [capa de ZoneLabel](#zonelabel-layer)

En las secciones siguientes se detallan los requisitos de cada capa.

### <a name="exterior-layer"></a>Capa exterior

El archivo DWG de cada nivel debe contener una capa para definir el perímetro de ese nivel. Esta capa se conoce como capa exterior. Por ejemplo, si una instalación contiene dos niveles, debe tener dos archivos DWG, con una capa exterior para cada archivo.

Independientemente de cuántos dibujos de entidad hay en la capa exterior, el [conjunto de la instalación resultante](tutorial-creator-indoor-maps.md#create-a-feature-stateset) contendrá solo **una** característica de nivel para cada archivo DWG. Además:

* Los exteriores deben dibujarse como polígonos, polilíneas (cerradas) o círculos.

* Los exteriores pueden superponerse, pero se disolverán en una geometría.

Si la capa contiene varias polilíneas superpuestas, las polilíneas se disolverán en una única característica de nivel. Como alternativa, si la capa contiene varias polilíneas que no se solapan, la característica de nivel resultante tendrá una representación multipoligonal.

Se puede ver un ejemplo de la capa exterior en la capa OUTLINE del [paquete de dibujos de ejemplo](https://github.com/Azure-Samples/am-creator-indoor-data-examples).

### <a name="unit-layer"></a>Capa de unidades

El archivo DWG para cada nivel debe definir una capa que contenga unidades.  Las unidades son espacios transitables en el edificio, como oficinas, vestíbulos, escaleras y ascensores. La capa de unidades debe cumplir los siguientes requisitos:

* Las unidades deben dibujarse como polígonos, polilíneas (cerradas) o círculos.
* Las unidades deben estar dentro de los límites del perímetro exterior de la instalación.
* Las unidades no pueden superponerse parcialmente.
* Las unidades no pueden contener ninguna geometría que forme intersección con ella misma.

 Asigne un nombre a una unidad creando un objeto de texto en la capa de _unitLabel_ y, a continuación, coloque el objeto dentro de los límites de la unidad. Para más información, consulte la [capa de unitLabel](#unitlabel-layer).

Se puede ver un ejemplo de la capa de unidades en la capa UNITS del [paquete de dibujos de ejemplo](https://github.com/Azure-Samples/am-creator-indoor-data-examples).

### <a name="wall-layer"></a>Capa de paredes

El archivo DWG de cada nivel puede contener una capa que defina las extensiones físicas de las paredes, las columnas y otras estructuras del edificio.

* Las paredes deben dibujarse como polígonos, polilíneas (cerradas) o círculos.
* Las capas de paredes solo deben contener geometría que se interprete como estructura del edificio.

Se puede ver un ejemplo de la capa de paredes en la capa WALLS del [paquete de dibujos de ejemplo](https://github.com/Azure-Samples/am-creator-indoor-data-examples).

### <a name="door-layer"></a>Capa de puertas

Puede incluir una capa de DWG que contenga puertas. Cada puerta debe superponerse con el borde de una unidad de la capa de unidad.

Las aberturas de las puertas de un conjunto de datos de Azure Maps se representan como un segmento de una sola línea que se superpone con varios límites de unidades. Se siguen los pasos siguientes para convertir la geometría en la capa de puertas a características de aperturas en un conjunto de datos.

![Pasos para generar las aperturas](./media/drawing-requirements/opening-steps.png)

### <a name="zone-layer"></a>Capa de zonas

El archivo DWG de cada nivel puede contener una capa de zonas que defina las extensiones físicas de las zonas. Una zona puede ser un espacio vacío interior o un jardín.

* Las zonas deben dibujarse como polígonos, polilíneas (cerradas) o círculos.
* Las zonas pueden superponerse.
* Las zonas pueden estar dentro o fuera del perímetro exterior de la instalación.

Asigne un nombre a una zona creando de un objeto de texto en la capa de _zoneLabel_ y colocando el objeto de texto dentro de los límites de la zona. Para más información, consulte la [capa de ZoneLabel](#zonelabel-layer).

Se puede ver un ejemplo de la capa de zonas en la capa ZONES del [paquete de dibujos de ejemplo](https://github.com/Azure-Samples/am-creator-indoor-data-examples).

### <a name="unitlabel-layer"></a>Capa de unitLabel

El archivo DWG de cada nivel puede contener una capa de etiquetas de unidades. La capa de etiquetas de unidades agrega una propiedad de nombre a las unidades extraídas de la capa de unidades. Las unidades con una propiedad de nombre pueden tener detalles adicionales especificados en el archivo de manifiesto.

* Las etiquetas de las unidades deben ser entidades de texto de una sola línea.
* Las etiquetas de las unidades deben estar dentro de los límites de su unidad.
* Las unidades no deben contener varias entidades de texto en la capa de etiquetas de la unidad.

Se puede ver un ejemplo de la capa de unitLabel en la capa UNITLABELS del [paquete de dibujos de ejemplo](https://github.com/Azure-Samples/am-creator-indoor-data-examples).

### <a name="zonelabel-layer"></a>Capa de ZoneLabel

El archivo DWG de cada nivel puede contener una capa de etiquetas de zonas. Esta capa agrega una propiedad de nombre a las zonas extraídas de la capa de zonas. Las zonas con una propiedad de nombre pueden tener detalles adicionales especificados en el archivo de manifiesto.

* Las etiquetas de las zonas deben ser entidades de texto de una sola línea.
* Las etiquetas de las zonas deben estar dentro de los límites de su zona.
* Las zonas no deben contener varias entidades de texto en la capa de etiquetas de la zona.

Se puede ver un ejemplo de la capa de ZoneLabel en la capa ZONELABELS del [paquete de dibujos de ejemplo](https://github.com/Azure-Samples/am-creator-indoor-data-examples).

## <a name="manifest-file-requirements"></a>Requisitos del archivo de manifiesto

La carpeta ZIP debe contener un archivo de manifiesto en el nivel raíz del directorio y el archivo debe tener el nombre **manifest.json**. Describe los archivos DWG para permitir que el [servicio de conversión de Azure Maps](https://docs.microsoft.com/rest/api/maps/conversion) analice su contenido. Solo se ingerirán los archivos identificados por el manifiesto. Los archivos que se encuentren en la carpeta ZIP pero que no se enumeren correctamente en el manifiesto se omitirán.

Las rutas de acceso de los archivos en el objeto **buildingLevels** del archivo de manifiesto deben ser relativas a la raíz de la carpeta ZIP. El nombre del archivo DWG debe coincidir exactamente con el nombre del nivel de la instalación. Por ejemplo, un archivo DWG para el nivel "Sótano" sería "Sótano.dwg". Un archivo DWG para el nivel 2 tendría el nombre "nivel_2.dwg". Use un carácter de subrayado si el nombre del nivel tiene un espacio.

Aunque hay requisitos al usar los objetos del manifiesto, no se requieren todos los objetos. En la tabla siguiente se muestran los objetos obligatorios y opcionales para la versión 1.1 del [servicio de conversión de Azure Maps](https://docs.microsoft.com/rest/api/maps/conversion).

| Object | Obligatorio | Descripción |
| :----- | :------- | :------- |
| version | true |Versión del esquema de manifiesto. Actualmente, solo se admite la versión 1.1.|
| directoryInfo | true | Describe la información geográfica y de contacto de la instalación. También se puede usar para describir información geográfica y de contacto de un ocupante. |
| buildingLevels | true | Especifica los niveles de los edificios y los archivos que contienen el diseño de los niveles. |
| georeference | true | Contiene información geográfica numérica para el dibujo de la instalación. |
| dwgLayers | true | Enumera los nombres de las capas y cada capa enumera los nombres de sus propias características. |
| unitProperties | false | Se puede usar para insertar metadatos adicionales para las características de la unidad. |
| zoneProperties | false | Se puede usar para insertar metadatos adicionales para las características de la zona. |

En las secciones siguientes se detallan los requisitos de cada objeto.

### <a name="directoryinfo"></a>directoryInfo

| Propiedad  | type | Obligatorio | Descripción |
|-----------|------|----------|-------------|
| name      | string | true   |  Nombre del edificio. |
| streetAddress|    string |    false    | Dirección del edificio. |
|unit     | string    |  false    |  Unidad en el edificio. |
| situación |    string |    false |    Nombre de una zona, un barrio o una región. Por ejemplo, "Overlake" o "Distrito central". La localidad no forma parte de la dirección postal. |
| adminDivisions |    Matriz JSON de cadenas |    false     | Matriz que contiene las designaciones de las direcciones (país, estado, ciudad) o (país, región, ciudad, población). Use códigos de país ISO 3166 y códigos de estado o territorio ISO 3166-2. |
| postalCode |    string    | false    | Código postal. |
| hoursOfOperation |    string |     false | Sigue el formato de [horas de apertura de OSM](https://wiki.openstreetmap.org/wiki/Key:opening_hours/specification). |
| phone    | string |    false |    Número de teléfono asociado con el edificio. Debe incluir el código de país. |
| sitio web    | string |    false    | Sitio web asociado con el edificio. Debe empezar por http o https. |
| nonPublic |    bool    | false | Marca que especifica si el edificio está abierto al público. |
| anchorLatitude | NUMERIC |    false | Latitud del marcador (chincheta) de la instalación. |
| anchorLongitude | NUMERIC |    false | Longitud del marcador (chincheta) de la instalación. |
| anchorHeightAboveSeaLevel  | NUMERIC | false | Altura de la planta baja de la instalación sobre el nivel del mar, en metros. |
| defaultLevelVerticalExtent | NUMERIC | false | Altura (grosor) predeterminada de un nivel de esta instalación que se usará cuando el valor `verticalExtent` de un nivel no esté definido. |

### <a name="buildinglevels"></a>buildingLevels

El objeto `buildingLevels` contiene una matriz JSON de los niveles del edificio.

| Propiedad  | Tipo | Obligatorio | Descripción |
|-----------|------|----------|-------------|
|levelName    |string    |true |    Nombre descriptivo del nivel. Por ejemplo: Piso 1, vestíbulo, aparcamiento, sótano, etc.|
|ordinal | integer |    true | Ordinal se usa para determinar el orden vertical de los niveles. Cada instalación debe tener un nivel con el ordinal 0. |
|heightAboveFacilityAnchor | NUMERIC | false |    Altura del nivel sobre el marcador en metros. |
| verticalExtent | NUMERIC | false | Altura del suelo al techo (grosor) del nivel en metros. |
|filename |    string |    true |    Ruta de acceso del sistema de archivos del dibujo de CAD para un nivel del edificio. Debe ser relativa a la raíz del archivo ZIP del edificio. |

### <a name="georeference"></a>georeference

| Propiedad  | Tipo | Obligatorio | Descripción |
|-----------|------|----------|-------------|
|lat    | NUMERIC |    true |    Representación decimal de los grados de latitud en el origen del dibujo de la instalación. Las coordenadas de origen deben estar en el formato WGS84 Web Mercator (`EPSG:3857`).|
|lon    |NUMERIC|    true|    Representación decimal de los grados de longitud en el origen del dibujo de la instalación. Las coordenadas de origen deben estar en el formato WGS84 Web Mercator (`EPSG:3857`). |
|angle|    NUMERIC|    true|   Ángulo en el sentido de las agujas del reloj, en grados, entre el norte verdadero y el eje vertical (Y) del dibujo.   |

### <a name="dwglayers"></a>dwgLayers

| Propiedad  | Tipo | Obligatorio | Descripción |
|-----------|------|----------|-------------|
|exterior    |Matriz de cadenas|    true|    Nombres de las capas que definen el perfil exterior del edificio.|
|unit|    Matriz de cadenas|    true|    Nombres de las capas que definen las unidades.|
|wall|    Matriz de cadenas    |false|    Nombres de las capas que definen las paredes.|
|door    |Matriz de cadenas|    false   | Nombres de las capas que definen las puertas.|
|unitLabel    |Matriz de cadenas|    false    |Nombres de las capas que definen los nombres de las unidades.|
|zona | Matriz de cadenas    | false    | Nombres de las capas que definen las zonas.|
|zoneLabel | Matriz de cadenas |     false |    Nombres de las capas que definen los nombres de las zonas.|

### <a name="unitproperties"></a>unitProperties

El objeto `unitProperties` contiene una matriz JSON de las propiedades de la unidad.

| Propiedad  | Tipo | Obligatorio | Descripción |
|-----------|------|----------|-------------|
|unitName    |string    |true    |Nombre de la unidad que se asociará a este registro `unitProperty`. Este registro solo es válido cuando se encuentra una etiqueta que coincida con `unitName` en las capas de `unitLabel`. |
|categoryName|    string|    false    |Nombre de la categoría. Para obtener una lista completa de las categorías, consulte las [categorías](https://aka.ms/pa-indoor-spacecategories). |
|navigableBy| Matriz de cadenas |    false    |Indica los tipos de agentes de navegación que pueden atravesar la unidad. For example, "pedestrian". Esta propiedad informará a las funcionalidades de enrutamiento.  Los valores permitidos son `pedestrian`, `wheelchair`, `machine`, `bicycle`, `automobile`, `hiredAuto`, `bus`, `railcar`, `emergency`, `ferry`, `boat` y `disallowed`.|
|routeThroughBehavior|    string|    false    |El comportamiento para crear una ruta a través de la unidad. Los valores admitidos son `disallowed`, `allowed` y `preferred`. El valor predeterminado es `allowed`.|
|occupants    |Matriz de objetos directoryInfo |false    |Lista de ocupantes de la unidad. |
|nameAlt|    string|    false|    Nombre alternativo de la unidad. |
|nameSubtitle|    string    |false|    Subtítulo de la unidad. |
|addressRoomNumber|    string|    false|    Número de habitación/unidad/apartamento/suite de la unidad.|
|verticalPenetrationCategory|    string|    false| Cuando se define esta propiedad, la característica resultante será una penetración vertical (VRT) en lugar de una unidad. Las VRT puede utilizarse para navegar a otras características de VRT en los niveles situados por encima o por debajo. La penetración vertical es un nombre de [categoría](https://aka.ms/pa-indoor-spacecategories). Si se define esta propiedad, la propiedad categoryName se invalida con verticalPenetrationCategory. |
|verticalPenetrationDirection|    string|    false    |Si se define `verticalPenetrationCategory`, defina opcionalmente la dirección válida de desplazamiento. Los valores admitidos son `lowToHigh`, `highToLow`, `both` y `closed`. El valor predeterminado es `both`.|
| nonPublic | bool | false | Indica si la unidad está abierta al público. |
| isRoutable | bool | false | Cuando se establece en `false`, la unidad no es accesible ni transitable. El valor predeterminado es `true`. |
| isOpenArea | bool | false | Permite que el agente de navegación entre en la unidad sin necesidad de una apertura asociada a la unidad. De forma predeterminada, este valor se establece en `true` para las unidades sin aperturas; `false` para las unidades con aperturas.  El establecimiento manual de `isOpenArea` en `false` en una unidad sin aperturas produce una advertencia. Esto se debe a que un agente de navegación no podrá alcanzar la unidad resultante.|

### <a name="the-zoneproperties-object"></a>Objeto zoneProperties

El objeto `zoneProperties` contiene una matriz JSON de las propiedades de la zona.

| Propiedad  | Tipo | Obligatorio | Descripción |
|-----------|------|----------|-------------|
|zoneName        |string    |true    |Nombre de la zona que se asociará al registro `zoneProperty`. Este registro solo es válido cuando se encuentra una etiqueta que coincida con `zoneName` en la capa de `zoneLabel` de la zona.  |
|categoryName|    string|    false    |Nombre de la categoría. Para obtener una lista completa de las categorías, consulte las [categorías](https://aka.ms/pa-indoor-spacecategories). |
|zoneNameAlt|    string|    false    |Nombre alternativo de la zona.  |
|zoneNameSubtitle|    string |    false    |Subtítulo de la zona. |
|zoneSetId|    string |    false    | Establezca el identificador para fijar una relación entre varias zonas para que se puedan consultar o seleccionar como grupo. Por ejemplo, las zonas que abarcan varios niveles. |

### <a name="sample-drawing-package-manifest"></a>Manifiesto de paquete de dibujo de ejemplo

A continuación se muestra un archivo de manifiesto de ejemplo para el paquete de dibujos de ejemplo. Para descargar todo el paquete, haga clic en [paquete de dibujos de ejemplo](https://github.com/Azure-Samples/am-creator-indoor-data-examples).

#### <a name="manifest-file"></a>Archivo de manifiesto

```JSON
{
    "version": "1.1", 
    "directoryInfo": { 
        "name": "Contoso Building", 
        "streetAddress": "Contoso Way", 
        "unit": "1", 
        "locality": "Contoso eastside", 
        "postalCode": "98052", 
        "adminDivisions": [ 
            "Contoso city", 
            "Contoso state", 
            "Contoso country" 
        ], 
        "hoursOfOperation": "Mo-Fr 08:00-17:00 open", 
        "phone": "1 (425) 555-1234", 
        "website": "www.contoso.com", 
        "nonPublic": false, 
        "anchorLatitude": 47.636152, 
        "anchorLongitude": -122.132600, 
        "anchorHeightAboveSeaLevel": 1000, 
        "defaultLevelVerticalExtent": 3  
    }, 
    "buildingLevels": { 
        "levels": [ 
            { 
                "levelName": "Basement", 
                "ordinal": -1, 
                "filename": "./Basement.dwg" 
            }, { 
                "levelName": "Ground", 
                "ordinal": 0, 
                "verticalExtent": 5, 
                "filename": "./Ground.dwg" 
            }, { 
                "levelName": "Level 2", 
                "ordinal": 1, 
                "heightAboveFacilityAnchor": 3.5, 
                "filename": "./Level_2.dwg" 
            } 
        ] 
    }, 
    "georeference": { 
        "lat": 47.636152, 
        "lon": -122.132600, 
        "angle": 0 
    }, 
    "dwgLayers": { 
        "exterior": [ 
            "OUTLINE", "WINDOWS" 
        ], 
        "unit": [ 
            "UNITS" 
        ], 
        "wall": [ 
            "WALLS" 
        ], 
        "door": [ 
            "DOORS" 
        ], 
        "unitLabel": [ 
            "UNITLABELS" 
        ], 
        "zone": [ 
            "ZONES" 
        ], 
        "zoneLabel": [ 
            "ZONELABELS" 
        ] 
    }, 
    "unitProperties": [ 
        { 
            "unitName": "B01", 
            "categoryName": "room.office", 
            "navigableBy": ["pedestrian", "wheelchair", "machine"], 
            "routeThroughBehavior": "disallowed", 
            "occupants": [ 
                { 
                    "name": "Joe's Office", 
                    "phone": "1 (425) 555-1234" 
                } 
            ], 
            "nameAlt": "Basement01", 
            "nameSubtitle": "01", 
            "addressRoomNumber": "B01", 
            "nonPublic": true, 
            "isRoutable": true, 
            "isOpenArea": true 
        }, 
        { 
            "unitName": "B02" 
        }, 
        { 
            "unitName": "B05", 
            "categoryName": "room.office" 
        }, 
        { 
            "unitName": "STRB01", 
            "verticalPenetrationCategory": "verticalPenetration.stairs", 
            "verticalPenetrationDirection": "both" 
        }, 
        { 
            "unitName": "ELVB01", 
            "verticalPenetrationCategory": "verticalPenetration.elevator", 
            "verticalPenetrationDirection": "high_to_low" 
        } 
    ], 
    "zoneProperties": 
    [ 
        { 
            "zoneName": "WifiB01", 
            "categoryName": "Zone", 
            "zoneNameAlt": "MyZone", 
            "zoneNameSubtitle": "Wifi", 
            "zoneSetId": "1234" 
        }, 
        { 
            "zoneName": "Wifi101",
            "categoryName": "Zone",
            "zoneNameAlt": "MyZone",
            "zoneNameSubtitle": "Wifi",
            "zoneSetId": "1234"
        }
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Una vez que el paquete de dibujos cumpla los requisitos, puede usar el [servicio de conversión de Azure Maps](https://docs.microsoft.com/rest/api/maps/conversion) para convertir el paquete en un conjunto de datos de mapa. Después, puede usar el conjunto de datos para generar un plano interior mediante el módulo de planos interiores. Para más información sobre el uso del módulo de mapas interiores, lea los artículos siguientes:

> [!div class="nextstepaction"]
>[Uso de Creator para planos interiores](creator-indoor-maps.md)

> [!div class="nextstepaction"]
> [Tutorial: Creación de un plano interior de Creator](tutorial-creator-indoor-maps.md)

> [!div class="nextstepaction"]
> [Estilo dinámico de planos interiores](indoor-map-dynamic-styling.md)
