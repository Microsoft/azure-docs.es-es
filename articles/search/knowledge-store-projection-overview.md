---
title: Trabajar con proyecciones en un almacén de información (versión preliminar) - Azure Search
description: Guardar y dar forma a los datos enriquecidos de la canalización de indización de inteligencia artificial para su uso en escenarios que no sean de búsqueda
manager: eladz
author: vkurpad
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: vikurpad
ms.custom: seomay2019
ms.openlocfilehash: f1c7278909557dc92f86c5dfc1f190fddf33f607
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540818"
---
# <a name="working-with-projections-in-a-knowledge-store-in-azure-search"></a>Trabajar con proyecciones en un almacén de información en Azure Search

> [!Note]
> Almacén de conocimiento está en versión preliminar y no se ha diseñado para su uso en producción. El [API de REST versión 2019-05-06-Preview](search-api-preview.md) proporciona esta característica. No hay ninguna compatibilidad de SDK de .NET en este momento.
>

Azure Search permite enriquecimiento de contenido a través de conocimientos cognitivos de inteligencia artificial y habilidades personalizadas como parte de la indización. Enriquecimientos de agregan estructura a los documentos y realizar búsquedas más eficaces. En muchos casos, los documentos enriquecidos son útiles para escenarios que no sean de búsqueda, por ejemplo, para la minería de datos de conocimiento.

Proyecciones, un componente de [almacén knowledge](knowledge-store-concept-intro.md), son vistas de documentos enriquecidos que se pueden guardar en el almacenamiento físico para fines de minería de datos de conocimiento. Una proyección permite "proyecto" los datos en una forma que se adapte a sus necesidades, mantener las relaciones para que herramientas como Power BI pueden leer los datos sin ningún esfuerzo adicional. 

Proyecciones pueden ser tabulares, con los datos almacenados en filas y columnas en Azure Table storage o los objetos JSON almacenados en Azure Blob storage. Puede definir varias proyecciones de los datos tal como se enriquecen. Esto es útil cuando desee que los mismos datos en forma de manera diferente para los casos de uso individual. 

El almacén de información admite dos tipos de proyecciones:

+ **Tablas**: Para los datos que mejor se representan como filas y columnas, las proyecciones de la tabla le permiten definir una forma esquematizada o una proyección en Table storage. 

+ **Objetos**: Cuando necesite una representación JSON de los datos y el grado de enriquecimiento del, las proyecciones de objeto se guardan como blobs.

Para ver las proyecciones que se define en el contexto, recorrer paso a paso [cómo empezar a trabajar con el almacén de información](knowledge-store-howto.md)

## <a name="projection-groups"></a>Grupos de proyección

En algunos casos, necesita los datos enriquecidos de formas diferentes para satisfacer los objetivos diferentes del proyecto. El almacén de información le permite definir varios grupos de proyecciones. Grupos de proyección tienen las siguientes características clave de exclusividad mutua y relevante.

### <a name="mutually-exclusivity"></a>Exclusividad mutuamente

Proyectado en un único grupo de todo el contenido es independiente de datos proyectados en otros grupos de proyección. Esto implica que puede tener los mismos datos en forma de manera diferente, aunque se repite en cada grupo de proyección. 

Una restricción que se aplica en grupos de proyección es la exclusividad mutua de los tipos de proyección con un grupo de proyección. Solo puede definir las proyecciones de la tabla o las proyecciones de objeto en un único grupo. Si desea que las tablas y objetos, defina un grupo de proyección para las tablas y un segundo grupo de proyección para los objetos.

### <a name="relatedness"></a>Relaciones de afinidad

Todo el contenido está previsto dentro de un grupo de proyección única conserva las relaciones dentro de los datos. Las relaciones se basan en una clave generada y cada nodo secundario mantiene una referencia al nodo principal. Las relaciones no abarcan grupos de proyección y las tablas u objetos creados en un grupo de proyección no tienen ninguna relación con los datos generados en otros grupos de proyección.

## <a name="input-shaping"></a>Entrada de forma
Obteniendo sus datos en la forma correcta o una estructura es el uso de claves en efectivo, ya sea tablas u objetos. La capacidad de modelar o estructurar los datos en función de cómo piensa tener acceso y su uso es una capacidad clave que se exponen como el **conformador** habilidades en el conjunto de habilidades.  

Las proyecciones son más fáciles de definir cuando se tiene un objeto en el árbol de enriquecimiento que coincida con el esquema de la proyección. La actualización [aptitud conformador](cognitive-search-skill-shaper.md) permite crear un objeto a partir de diferentes nodos del árbol enriquecimiento y ellos primario bajo un nodo nuevo. El **conformador** habilidades le permite definir tipos complejos con objetos anidados.

Cuando haya definido una forma nueva que contiene todos los elementos que necesita proyecten externamente, ahora puede usar esta forma como origen de sus previsiones o como una entrada para otra habilidad.

## <a name="table-projections"></a>Proyecciones de tabla

Porque hace más fácil importar, se recomienda las proyecciones de la tabla para la exploración de datos con Power BI. Además, las proyecciones de la tabla permiten para cambiar la cardinalidad entre la relación de tabla de cambio. 

Puede proyectar un solo documento en el índice en varias tablas, conservando las relaciones. Al proyectar a varias tablas, la forma completa se proyectará en cada tabla, a menos que un nodo secundario es el origen de otra tabla dentro del mismo grupo.

### <a name="defining-a-table-projection"></a>Define una proyección de la tabla

Al definir una proyección de la tabla dentro de la `knowledgeStore` elemento de tus aptitudes, empiece por asignar un nodo en el árbol de enriquecimiento en la tabla de origen. Normalmente, este nodo es el resultado de una **conformador** conocimientos que ha agregado a la lista de habilidades para producir una forma específica que necesita para proyectar en tablas. Puede segmentar el nodo que elija para el proyecto al proyecto en varias tablas. La definición de tablas es una lista de tablas que desea que el proyecto. 

Cada tabla requiere tres propiedades:

+ tableName: El nombre de la tabla en el almacenamiento de Azure.

+ generatedKeyName: El nombre de columna para la clave que identifica de forma única esta fila.

+ source: El nodo del árbol de enriquecimiento de que obtiene sus enriquecimientos de. Esto suele ser el resultado de un conformador, pero podría ser el resultado de cualquiera de los conocimientos.

Este es un ejemplo de proyecciones de la tabla.

```json
{
    "name": "your-skillset",
    "skills": [
      …your skills
      
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [
            { "tableName": "MainTable", "generatedKeyName": "SomeId", "source": "/document/EnrichedShape" },
            { "tableName": "KeyPhrases", "generatedKeyName": "KeyPhraseId", "source": "/document/EnrichedShape/*/KeyPhrases/*" },
            { "tableName": "Entities", "generatedKeyName": "EntityId", "source": "/document/EnrichedShape/*/Entities/*" }
          ]
        },
        {
          "objects": [
            
          ]
        }
      ]
    }
}
```
Como se muestra en este ejemplo, las frases clave y las entidades se modelan en tablas diferentes y contengan una referencia al elemento primario (MainTable) para cada fila. 

La siguiente ilustración es una referencia para el ejercicio Caselaw en [cómo empezar a trabajar con el almacén de conocimiento](knowledge-store-howto.md). En un escenario donde un caso tiene que escribir varias opiniones, y cada opinión enriquecido mediante la identificación de entidades contenidas dentro de él, podríamos modelar las proyecciones como se muestra aquí.

![Entidades y relaciones en tablas](media/knowledge-store-projection-overview/TableRelationships.png "modelar las relaciones en las proyecciones de tabla")

## <a name="object-projections"></a>Proyecciones de objeto

Las proyecciones de objeto son representaciones de JSON del árbol de enriquecimiento pueden proceder de cualquier nodo. En muchos casos, el mismo **conformador** aptitud que crea una proyección de la tabla se puede usar para generar una proyección del objeto. 

```json
{
 
    "name": "your-skillset",
    "skills": [
      …your skills
      
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [ ]
        },
        {
          "objects": [
            {
              "storageContainer": "Reviews", 
              "format": "json", 
              "source": "/document/Review", 
              "key": "/document/Review/Id" 
            }
          ]
        }
      ]
    }
}
```

Generación de proyección de un objeto requiere unos cuantos atributos específicos del objeto:

+ storageContainer: El contenedor donde se guardarán los objetos
+ source: La ruta de acceso al nodo del árbol de enriquecimiento es la raíz de la proyección
+ Clave: Una ruta de acceso que representa una clave única para el objeto que se almacenará. Se utilizará para crear el nombre del blob del contenedor.

## <a name="projection-lifecycle"></a>Ciclo de vida de proyección

Proyecciones tienen un ciclo de vida que está asociado al origen de datos del origen de datos. Como los datos se actualizan y volver a indizar, proyecciones se actualizan con los resultados de la enriquecimientos de lo que garantiza que sus proyecciones son eventualmente coherentes con los datos del origen de datos. Las proyecciones heredan la directiva de eliminación que ha configurado para el índice. 

## <a name="using-projections"></a>Uso de proyecciones

Después de ejecuta el indexador, puede leer los datos proyectados en los contenedores o tablas que especificó a través de las proyecciones. 

Para el análisis, exploración en Power BI es tan sencillo como configurar el almacenamiento de Azure Table como origen de datos. Puede crear muy fácilmente un conjunto de visualizaciones en los datos aprovechando las relaciones dentro.

Como alternativa, si tiene que usar los datos enriquecidos en una canalización de ciencia de datos, podría [cargar los datos desde los blobs en una trama de datos de Pandas](../machine-learning/team-data-science-process/explore-data-blob.md).

Por último, si tiene que exportar los datos desde el almacén de conocimiento, Azure Data Factory tiene conectores para exportar los datos y se colocan en la base de datos de su elección. 

## <a name="next-steps"></a>Pasos siguientes

Como siguiente paso, cree su primer almacén de conocimiento con instrucciones y datos de ejemplo.

> [!div class="nextstepaction"]
> [Cómo crear un almacén de conocimiento](knowledge-store-howto.md).