---
title: Normalización de datos en Azure Sentinel | Microsoft Docs
description: En este artículo se explica el modo en que Azure Sentinel normaliza los datos de muchos orígenes diferentes y se detalla el esquema de normalización.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: yelevin
ms.openlocfilehash: 0c6129a24e6ed083114971df5f254eca54924400
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932358"
---
# <a name="normalization-in-azure-sentinel"></a>Normalización en Azure Sentinel

En este artículo se explica la normalización de esquemas de datos en Azure Sentinel y, en concreto, el esquema de sesiones y conexiones de red, y se incluye un vínculo a él.

## <a name="what-is-normalization"></a>Qué es la normalización

Trabajar al mismo tiempo con varios tipos de datos y tablas presenta dificultades. Debe familiarizarse con muchos esquemas y tipos de datos diferentes, y tiene que escribir y usar un único conjunto de reglas de análisis, libros y consultas de búsqueda para cada uno de ellos, incluso para los que comparten aspectos comunes (por ejemplo, los que pertenecen a dispositivos de firewall). También es difícil la correlación entre los distintos tipos de datos, pero necesaria para la investigación y la búsqueda. Azure Sentinel proporciona una experiencia perfecta para el tratamiento de datos de orígenes diversos en vistas uniformes y normalizadas.

El **modelo de información común** de Azure Sentinel consta de tres aspectos:

- **Esquemas normalizados**: abarcan conjuntos comunes de tipos de eventos predecibles (tablas) con los que es fácil trabajar y en los que se pueden crear funcionalidades unificadas de forma sencilla (por ejemplo, la tabla de red). El esquema también incluye una convención de columnas normalizada, junto con definiciones para la normalización de valores y formatos (representación coherente y estándar de los datos, como las direcciones IP).

- **Analizadores**: asignan datos existentes de distintos tipos al esquema normalizado. Según el modelo, los datos se pueden analizar en el esquema normalizado en tiempo de consulta (mediante funciones) o en tiempo de ingesta. En este momento, solo se admite el análisis en tiempo de consulta.

- **Contenido de cada esquema normalizado**: incluye reglas de análisis, libros interactivos, consultas de búsqueda y contenido adicional.

### <a name="current-release"></a>Versión actual

Con esta versión, el [esquema normalizado de sesiones y conexiones de red](./normalization-schema.md) (v 1.0.0) está disponible en versión preliminar pública. El esquema describe las sesiones o conexiones de red, como las registradas por firewalls, datos de conexión, grupos de seguridad de red, Netflow, sistemas proxy y puertas de enlace de seguridad web.  Las reglas de análisis y los analizadores seleccionados en tiempo de consulta están disponibles junto con el esquema y hacen uso de él.

El esquema está actualmente disponible únicamente a través de los analizadores en tiempo de consulta (consulte la sección sobre analizadores).

Puede analizar datos en representaciones adicionales y usar el [modelo de nomenclatura de entidades de OSSEM](https://ossemproject.com/cdm/entities/intro.html#) para crear columnas que sean coherentes con las tablas normalizadas existentes y futuras.

## <a name="normalized-schema-and-parsing"></a>Esquema y análisis normalizados

### <a name="how-our-normalized-schemas-are-defined"></a>Definición de los esquemas normalizados

Azure Sentinel se alinea con el modelo de información común [Open Source Security Events Metadata (OSSEM)](https://ossemproject.com/intro.html), lo que permite la correlación predecible de entidades en las tablas normalizadas. OSSEM es un proyecto dirigido por la comunidad que se centra principalmente en la documentación y la normalización de registros de eventos de seguridad procedentes de diversos orígenes de datos y sistemas operativos. Además, el proyecto proporciona un modelo de información común (CIM) que pueden usar los ingenieros de datos durante los procedimientos de normalización de los datos para que los analistas de seguridad puedan consultar y analizar los datos de diversos orígenes.

El [CIM de OSSEM](https://ossemproject.com/cdm/intro.html) define un conjunto de entidades (por ejemplo, archivo, host, IP, proceso) y un conjunto de atributos para cada una de ellas. Además, el CIM define un conjunto de tablas (por ejemplo, [sesión de red](https://ossemproject.com/cdm/tables/network_session.html)) que usan atributos pertinentes de estas entidades, lo que permite una correlación perfecta y predecible. Tenga en cuenta que las entidades se pueden anidar (por ejemplo, la entidad de origen puede contener una entidad de archivo que tendrá un atributo de nombre).

Para más información sobre la estructura de la entidad OSSEM, visite la [referencia oficial de OSSEM](https://ossemproject.com/cdm/guidelines/entity_structure.html).

### <a name="how-the-normalized-schemas-are-implemented-in-azure-sentinel"></a>Implementación de los esquemas normalizados en Azure Sentinel

En la implementación del CIM de OSSEM en Azure Sentinel, se proyecta la representación de OSSEM en una representación tabular Log Analytics, tanto si esta representación es una tabla integrada como si se creó mediante analizadores en tiempo de consulta o funciones que asignan datos existentes a esta representación. En la representación tabular, se concatenan los nombres de entidades y atributos de OSSEM y se asignan de forma colectiva a un único nombre de columna. Por ejemplo, una entidad de origen que contenga una entidad de archivo que contenga una entidad hash que contenga un atributo md5 se implementará como la siguiente columna de Log Analytics: SrcFileHashMd5. (OSSEM usa el formato *snake_case* de forma predeterminada, mientras que Azure Sentinel y Log Analytics usan *PascalCase*. En OSSEM, este tipo de columna sería src_file_hash_md5).

Pueden existir campos personalizados adicionales en la implementación de Azure Sentinel, debido a los requisitos de la plataforma de Log Analytics y a casos de uso específicos de los clientes de Azure Sentinel.

### <a name="schema-reference"></a>Referencia de esquemas

Para más información, consulte el [documento de referencia del esquema](./normalization-schema.md), así como la [documentación oficial del proyecto de OSSEM](https://ossemproject.com/cdm/intro.html).

La referencia del esquema también incluye la normalización de valores y formatos. Los campos de origen, originales o analizados, podrían no tener un formato estándar o usar la lista de valores estándar del esquema y, por tanto, sería necesario convertirlos en la representación estándar del esquema para que se normalicen por completo.

## <a name="parsers"></a>Analizadores

### <a name="what-is-parsing"></a>Qué es el análisis

Con un conjunto básico de tablas normalizadas definidas, deberá transformar (analizar o asignar) los datos en esas tablas. Es decir, extraer datos específicos de su formato sin procesar en columnas conocidas del esquema normalizado. El análisis en Azure Sentinel se produce en **tiempo de consulta**; los analizadores se compilan como funciones de usuario de Log Analytics (mediante el lenguaje de consulta de Kusto, KQL) que transforman los datos de las tablas existentes (como CommonSecurityLog, tablas de registros personalizadas, syslog) en el esquema de las tablas normalizadas.

El otro tipo de análisis, que todavía no se admite en Azure Sentinel, se realiza en **tiempo de ingesta**, lo que permite recopilar datos directamente en las tablas normalizadas a medida que se ingieren desde sus orígenes de datos. El análisis en tiempo de ingesta proporciona un mejor rendimiento, ya que el modelo de datos se consulta directamente sin necesidad de usar funciones.

### <a name="using-query-time-parsers"></a>Uso de analizadores en tiempo de consulta

#### <a name="installing-a-parser"></a>Instalación de un analizador

Los analizadores en tiempo de consulta disponibles se pueden encontrar en el [repositorio de GitHub oficial](https://github.com/Azure/Azure-Sentinel/tree/master/Parsers/Normalized%20Schema%20-%20Networking%20(v1.0.0)) de Azure Sentinel. Cada analizador tiene control de versiones para permitir a los clientes usar y supervisar fácilmente las actualizaciones futuras. Para instalar un analizador:

1. Copie el contenido del analizador pertinente de cada archivo KQL en cuestión del vínculo de GitHub anterior en el portapapeles.

1. En el portal de Azure Sentinel, abra la página de registros y pegue el contenido del archivo KQL en la pantalla de registros; haga clic en **Guardar**.

    :::image type="content" source="./media/normalization/install-new-parser.png" alt-text="Instalación de un nuevo analizador":::

1. En el cuadro de diálogo Guardar, rellene los campos como se indica a continuación:
    1. **Name**: se recomienda utilizar el mismo valor usado en el campo **Alias de función** (en el ejemplo anterior, *CheckPoint_Network_NormalizedParser*).
    
    1. **Guardar como**: seleccione **Función**.

    1. **Alias de función**: se debe nombrar con la convención de nomenclatura siguiente: *PRODUCT_Network_NormalizedParser* (en el ejemplo anterior, *CheckPoint_Network_NormalizedParser*).

    1. **Categoría**: puede seleccionar una categoría existente o crear otra (como *NormalizedNetworkSessionsParsers*)
    
        :::image type="content" source="./media/normalization/save-new-parser.png" alt-text="Guardar el analizador":::

Para usar correctamente los analizadores, también debe instalar el analizador de esquema de red vacío (que crea una vista tabular vacía de todos los campos del esquema de sesiones de red) y el metaanalizador de red (que une todos los analizadores habilitados para crear una vista única de los datos de varios orígenes en el esquema de red). La instalación de estos dos analizadores se realiza de manera similar a los pasos mencionados anteriormente.

Al guardar una función de consulta, puede ser necesario cerrar el explorador de consultas y volver a abrirlo para que se refleje la nueva función.

#### <a name="using-the-parsers"></a>Uso de los analizadores

Una vez habilitado, puede usar el metaanalizador para consultar una vista unificada de todos los analizadores habilitados actualmente. Para ello, vaya a la página de registros de Sentinel y realice consultas con el metaanalizador:

:::image type="content" source="./media/normalization/query-parser.png" alt-text="Realización de consultas con el analizador":::
 
También puede acceder al metaanalizador o a los analizadores individuales mediante el explorador de consultas de la página de registros de Sentinel; para ello, haga clic en "Explorador de consultas":

:::image type="content" source="./media/normalization/query-explorer.png" alt-text="Explorador de consultas":::

En el panel de la derecha, expanda la sección "Consultas guardadas" y busque la carpeta "NormalizedNetworkParsers" (o el nombre de categoría que haya elegido al crear los analizadores):

:::image type="content" source="./media/normalization/find-parser.png" alt-text="Búsqueda del analizador":::

Puede hacer clic en cada analizador individual y ver la función subyacente que utiliza y ejecutarlo (o acceder directamente a él por su alias, como se describió anteriormente). Tenga en cuenta que, por motivos prácticos, algunos analizadores pueden conservar los campos originales a la vez que los campos normalizados. Esta opción puede cambiarse fácilmente en la consulta del analizador.

#### <a name="customizing-parsers"></a>Personalización de los analizadores

Puede repetir los pasos anteriores (buscar el analizador en el explorador de consultas), hacer clic en el analizador pertinente y ver su implementación de función.
Por ejemplo, puede decidir editar el metaanalizador para agregar o quitar analizadores individuales.

:::image type="content" source="./media/normalization/customize-parser.png" alt-text="Personalización del analizador":::
 
Una vez modificada la función, vuelva a hacer clic en "Guardar" y use el mismo nombre, alias y categoría. Se abrirá un cuadro de diálogo de invalidación; presione "Aceptar":

:::image type="content" source="./media/normalization/are-you-sure.png" alt-text="¿Está seguro de que desea continuar?":::

#### <a name="additional-information"></a>Información adicional

Más información sobre las [consultas guardadas](../azure-monitor/log-query/saved-queries.md) (implementación de analizadores en tiempo de consulta) en Log Analytics.


## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido sobre el esquema de normalización de Azure Sentinel. Para información sobre el propio esquema de referencia, consulte [Referencia de esquema de normalización de datos de Azure Sentinel](./normalization-schema.md).

* [Blog de Azure Sentinel](https://aka.ms/azuresentinelblog). Encuentre artículos de blog sobre el cumplimiento y la seguridad de Azure.
