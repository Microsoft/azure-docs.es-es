---
title: Importación de datos en un índice de búsqueda con Azure Portal
titleSuffix: Azure Cognitive Search
description: Aprenda a usar el Asistente para la importación de datos en Azure Portal para rastrear datos de Azure desde Cosmos DB, Blob Storage, Table Storage, SQL Database, SQL Managed Instance y SQL Server en máquinas virtuales de Azure.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 7cff009d5d1e187e8d0330fadca530b57b3e3d21
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935218"
---
# <a name="import-data-wizard-for-azure-cognitive-search"></a>Asistente para la importación de datos de Azure Cognitive Search

Azure Portal incluye un asistente para la **importación de datos** en el panel de Azure Cognitive Search para la creación de prototipos y la carga de datos en un índice. En este artículo se tratan las ventajas y limitaciones del uso del asistente, las entradas y salidas, y la información de uso. Para obtener instrucciones prácticas sobre cómo recorrer paso a paso el asistente con datos de ejemplo integrados, consulte el inicio rápido [Creación de un índice de Azure Cognitive Search en Azure Portal](search-get-started-portal.md).

Entre las operaciones que realiza este asistente se incluyen:

1 - Conexión a un origen de datos de Azure compatible.

2 - Creación de un esquema de índice que se deduce mediante el muestreo de los datos de origen.

3 - Opcionalmente, incorporación de características de enriquecimiento de inteligencia artificial para extraer o generar contenido y estructuras.

4 - Ejecución del asistente para crear objetos, importar datos, establecer una programación y otras opciones de configuración.

El asistente genera una serie de objetos, que se guardan en el servicio de búsqueda, a los que se puede acceder mediante programación o con otras herramientas.

## <a name="advantages-and-limitations"></a>Ventajas y limitaciones

Antes de escribir cualquier código, puede usar el asistente para crear prototipos y pruebas de concepto. El asistente se conecta a orígenes de datos externos, muestrea los datos para crear un índice inicial y, a continuación, los importa como documentos JSON en un índice de Azure Cognitive Search. 

El muestreo es el proceso mediante el cual se infiere un esquema de índice y tiene algunas limitaciones. Al crear el origen de datos, el asistente elige una muestra de documentos para decidir qué columnas forman parte del origen de datos. No se leen todos los archivos, ya que esto podría tardar horas en el caso de orígenes de datos muy grandes. Dada una selección de documentos, los metadatos de origen como, por ejemplo, el nombre o el tipo de campo, se usan para crear una colección de campos en un esquema de índice. En función de la complejidad de los datos de origen, es posible que tenga que modificar el esquema inicial para que sea más preciso, o bien ampliarlo para que sea más exhaustivo. Puede insertar los cambios en la página de definición del índice.

En general, las ventajas del uso del asistente son claras: siempre que se cumplan los requisitos, puede crear un prototipo de un índice consultable en cuestión de minutos. Algunas de las dificultades de la indexación como, por ejemplo, proporcionar datos como documentos JSON, se controlan mediante el asistente.

Las limitaciones conocidas se resumen de la siguiente forma:

+ El asistente no admite la iteración ni la reutilización. Cada paso a través del asistente crea una nueva configuración del índice, del conjunto de aptitudes y del indexador. Solo los orígenes de datos se pueden conservar y reutilizar en el asistente. Para editar o restringir otros objetos, tiene que usar las API REST o el SDK de .NET para recuperar y modificar las estructuras.

+ El contenido de origen debe residir en un origen de datos de Azure compatible.

+ El muestreo se realiza sobre un subconjunto de los datos de origen. En el caso de orígenes de datos de gran tamaño, es posible que el asistente omita algunos campos. Es posible que tenga que ampliar el esquema o corregir los tipos de datos inferidos si el muestreo es insuficiente.

+ El enriquecimiento de inteligencia artificial, tal como se expone en el portal, se limita a algunas aptitudes integradas. 

+ El [almacén de conocimiento](knowledge-store-concept-intro.md), que se puede crear mediante el asistente, se limita a algunas proyecciones predeterminadas. Si desea guardar documentos enriquecidos creados por el asistente, el contenedor de blobs y las tablas incluyen nombres y estructuras predeterminados.

<a name="data-source-inputs"></a>

## <a name="data-source-input"></a>Entrada de origen de datos

El asistente para la **importación de datos** se conecta a un origen de datos externo mediante la lógica interna que proporcionan los indizadores de Azure Cognitive Search, que están equipados para muestrear el origen, leer los metadatos, descifrar documentos para leer el contenido y la estructura, y serializar el contenido como JSON para la importación posterior en Azure Cognitive Search.

Solo se puede importar desde una única tabla, vista de base de datos o estructura de datos equivalente; sin embargo, la estructura puede incluir subestructuras jerárquicas o anidadas. Para más información, consulte el artículo sobre la [Cómo modelar tipos complejos](search-howto-complex-data-types.md).

Debe crear esta tabla o vista única antes de ejecutar el asistente y debe incluir contenido. Por motivos obvios, no tiene sentido ejecutar el asistente para la **importación de datos** en un origen de datos vacío.

|  Número de selección | Descripción |
| ---------- | ----------- |
| **Origen de datos existente** |Si ya tiene indexadores definidos en el servicio de búsqueda, es posible que tenga una definición de origen de datos ya existente que puede volver a utilizar. En Azure Cognitive Search, solo los indizadores usan objetos de origen de datos. Puede crear un objeto de origen de datos mediante programación o con el asistente para la **importación de datos** y reutilizarlo cuando sea necesario.|
| **Muestras**| Azure Cognitive Search proporciona dos orígenes de datos de ejemplo integrados que se usan en tutoriales e inicios rápidos: una base de datos SQL con información inmobiliaria y una base de datos de hoteles hospedada en Cosmos DB. Para ver un tutorial basado en el ejemplo de los hoteles, consulte el inicio rápido [Creación de un índice de Azure Search en Azure Portal](search-get-started-portal.md). |
| [**Azure SQL Database o SQL Managed Instance**](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md) |Se puede especificar el nombre del servicio, las credenciales de un usuario de base de datos con permiso de lectura y un nombre de base de datos en la página o a través de una cadena de conexión de ADO.NET. Elija la opción de cadena de conexión para ver o personalizar las propiedades. <br/><br/>En la página debe especificarse la tabla o vista que proporciona el conjunto de filas. Esta opción aparece una vez realizada correctamente la conexión, lo que proporciona una lista desplegable para que pueda realizar una selección.|
| **SQL Server en máquinas virtuales de Azure** |Especifique un nombre de servicio completo, el identificador de usuario y la contraseña, y la base de datos como una cadena de conexión. Para utilizar este origen de datos, debe haber instalado un certificado en el almacén local que cifra la conexión. Para obtener instrucciones, consulte [Conexión de máquina virtual de SQL a Azure Cognitive Search](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md). <br/><br/>En la página debe especificarse la tabla o vista que proporciona el conjunto de filas. Esta opción aparece una vez realizada correctamente la conexión, lo que proporciona una lista desplegable para que pueda realizar una selección. |
| [**Azure Cosmos DB**](search-howto-index-cosmosdb.md)|Los requisitos incluyen la cuenta, la base de datos y la colección. Todos los documentos de la colección se incluirán en el índice. Puede definir una consulta para aplanar o filtrar el conjunto de filas o dejar en blanco la consulta. No se requiere una consulta en este asistente.|
| [**Azure Blob Storage**](search-howto-indexing-azure-blob-storage.md) |Los requisitos incluyen la cuenta de almacenamiento y un contenedor. Opcionalmente, si los nombres de blobs siguen una convención de nomenclatura virtual con fines de agrupación, puede especificar la parte del directorio virtual del nombre como una carpeta en el contenedor. Consulte [Indexación de Blob Storage](search-howto-indexing-azure-blob-storage.md) para más información. |
| [**Azure Table Storage**](search-howto-indexing-azure-tables.md) |Los requisitos incluyen la cuenta de almacenamiento y un nombre de tabla. Opcionalmente, puede especificar una consulta para recuperar un subconjunto de las tablas. Consulte [Indexación de Table Storage](search-howto-indexing-azure-tables.md) para más información. |

## <a name="wizard-output"></a>Salida del asistente

En segundo plano, el asistente crea, configura e invoca los siguientes objetos. Después de ejecutar el asistente, puede encontrar su salida en las páginas del portal. La página de información general de su servicio tiene listas de índices, indexadores, orígenes de datos y conjuntos de aptitudes. Las definiciones de índice se pueden ver en JSON completo en el portal. Para otras definiciones, puede usar la [API REST](/rest/api/searchservice/) para OBTENER objetos específicos.

| Object | Descripción | 
|--------|-------------|
| [Data Source](/rest/api/searchservice/create-data-source) (Origen de datos)  | Conserva la información de conexión en los datos de origen, incluidas las credenciales. Un objeto de origen de datos se utiliza exclusivamente con indexadores. | 
| [Index](/rest/api/searchservice/create-index) | Estructura de datos física que se usa para la búsqueda de texto completo y otras consultas. | 
| [Conjunto de aptitudes](/rest/api/searchservice/create-skillset) | Un conjunto completo de instrucciones para manipular, transformar y dar forma al contenido que incluye el análisis y la extracción de información de archivos de imagen. Excepto en el caso de estructuras muy simples y limitadas, incluye una referencia a un recurso de Cognitive Services que proporciona enriquecimiento. Opcionalmente, también puede contener una definición de almacén de conocimiento.  | 
| [Indexador](/rest/api/searchservice/create-indexer)  | Un objeto de configuración que especifica un origen de datos, un índice de destino, un conjunto de aptitudes opcional, una programación opcional y valores de configuración opcionales para el control de errores y la codificación en base 64. |


## <a name="how-to-start-the-wizard"></a>Inicio del asistente

El asistente para la importación de datos se inicia desde la barra de comandos de la página Información general del servicio.

1. En [Azure Portal](https://portal.azure.com), abra la página del servicio de búsqueda desde el panel o [busque el servicio](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en la lista.

2. En la página de información general del servicio en la parte superior, haga clic en **Importar datos**.

   ![Comando de importación de datos en el portal](./media/search-import-data-portal/import-data-cmd2.png "Inicio del Asistente para la importación de datos")

Puede iniciar la **importación de datos** desde otros servicios de Azure, como Azure Cosmos DB, Azure SQL Database, SQL Managed Instance y Azure Blob Storage. Busque la opción para **agregar Azure Cognitive Search** en el panel de navegación izquierdo en la página de información general del servicio.

<a name="index-definition"></a>

## <a name="how-to-edit-or-finish-an-index-schema-in-the-wizard"></a>Edición o finalización de un esquema de índice en el asistente

El asistente genera un índice incompleto que se rellena con los documentos que se obtienen del origen de datos de entrada. Para que el índice sea funcional, asegúrese de que tiene definidos los elementos siguientes.

1. ¿Se ha completado la lista de campos? Agregue los nuevos campos que omitió el muestreo y quite aquellos que no aporten valor a una experiencia de búsqueda o que no se utilizarán en una [expresión de filtro](search-query-odata-filter.md) o [perfil de puntuación](index-add-scoring-profiles.md).

1. ¿Es el tipo de datos adecuado para los datos entrantes? Azure Cognitive Search admite los [tipos de datos de Entity Data Model (EDM)](/rest/api/searchservice/supported-data-types). En el caso de datos SQL de Azure, hay un [gráfico de asignaciones](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md#TypeMapping) que muestra valores equivalentes. Para más información, consulte [Transformaciones y asignaciones de campos](search-indexer-field-mappings.md).

1. ¿Tiene un campo que pueda actuar como *clave*? Este campo debe ser Edm.string y debe identificar de forma exclusiva un único documento. En el caso de los datos relacionales, se podría asignar a una clave principal. En el caso de los blobs, esta podría ser `metadata-storage-path`. Si los valores de campo incluyen espacios o guiones, debe establecer la opción **Claves de codificación Base 64** del paso **Crear un indexador** en **Opciones avanzadas**, a fin de suprimir la comprobación de validación para estos caracteres.

1. Establezca los atributos para determinar cómo se utiliza ese campo en un índice. 

   Realice con cuidado este paso porque los atributos determinarán la expresión física de los campos del índice. Si desea cambiar los atributos más adelante, incluso mediante programación, casi siempre tendrá que quitar y recompilar el índice. Los atributos principales como **Searchable** y **Retrievable** tienen un [impacto insignificante en el almacenamiento](search-what-is-an-index.md#index-size). La habilitación de filtros y el uso de proveedores de sugerencias aumentan los requisitos de almacenamiento. 
   
   + **Searchable** permite la búsqueda de texto completo. Todos los campos utilizados en consultas de formato libre o en expresiones de consulta deben tener este atributo. Para cada campo que marque como **Searchable**, se crearán índices invertidos.

   + **Retrievable** devuelve el campo en los resultados de búsqueda. Todos los campos que proporcionan contenido a los resultados de búsqueda deben tener este atributo. Al establecer este campo, el tamaño del índice no se verá afectado de manera apreciable.

   + **Filterable** permite que se haga referencia al campo en las expresiones de filtro. Cada campo utilizado en una expresión **$filter** debe tener este atributo. Las expresiones de filtro son para coincidencias exactas. Dado que las cadenas de texto permanecen intactas, se necesita almacenamiento adicional para dar cabida al contenido textual.

   + **Facetable** permite que el campo se use en la navegación con facetas. Solo los campos marcados también como **Filterable** pueden marcarse como **Facetable**.

   + **Sortable** permite que el campo se use en una ordenación. Cada campo utilizado en una expresión **$Orderby** debe tener este atributo.

1. ¿Necesita [análisis léxico](search-lucene-query-architecture.md#stage-2-lexical-analysis)? En el caso de los campos Edm.string que son **Searchable**, puede establecer un **analizador** si quiere indexación y consulta con un lenguaje mejorado. 

   La opción predeterminada es *Estándar - Lucene*, pero puede elegir *Inglés - Microsoft* si desea usar el analizador de Microsoft para procesamiento léxico avanzado, como para la resolución de formas verbales y sustantivos irregulares. Solo los analizadores de idiomas se pueden especificar en el portal. Si se usa un analizador personalizado o un analizador que no sea de idiomas, como palabra clave o patrón, se deberá hacer mediante programación. Para más información acerca de los analizadores, consulte [Adición de analizadores de idiomas](search-language-support.md).

1. ¿Necesita la funcionalidad de escritura anticipada en forma de autocompletar o de resultados sugeridos? Active la casilla **Proveedor de sugerencias** para habilitar las [sugerencias de consultas de escritura anticipada y la función autocompletar](index-add-suggesters.md) en los campos seleccionados. Los proveedores de sugerencias se suman al número de términos con tokens en el índice y, por tanto, consumen más almacenamiento.


## <a name="next-steps"></a>Pasos siguientes

La mejor manera de comprender las ventajas y limitaciones del asistente es recorrer sus pasos. El siguiente inicio rápido le guiará a través de cada paso.

> [!div class="nextstepaction"]
> [Creación de un índice de Azure Cognitive Search en Azure Portal](search-get-started-portal.md)