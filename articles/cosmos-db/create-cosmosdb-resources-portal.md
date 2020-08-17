---
title: 'Inicio rápido: Creación de recursos de Azure Cosmos DB en Azure Portal'
description: En este inicio rápido se muestra cómo crear una base de datos, un contenedor y elementos de Azure Cosmos mediante Azure Portal.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 08/05/2020
ms.openlocfilehash: 31bdcd9994ebfcea77b25422997463cc6a9bfddc
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053531"
---
# <a name="quickstart-create-an-azure-cosmos-account-database-container-and-items-from-the-azure-portal"></a>Inicio rápido: Creación de una base de datos, un contenedor y elementos de Azure Cosmos desde Azure Portal

> [!div class="op_single_selector"]
> * [Azure Portal](create-cosmosdb-resources-portal.md)
> * [.NET](create-sql-api-dotnet.md)
> * [Java](create-sql-api-java.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)
>  

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede usar Azure Cosmos DB para crear rápidamente una clave de consulta y bases de datos de valores, así como bases de datos de documentos y de grafos, y realizar consultas en ellas. Todas estas bases de datos se beneficiarán de las funcionalidades de distribución global y escala horizontal de Azure Cosmos DB. 

En este inicio rápido se muestra cómo usar Azure Portal para crear una cuenta de [SQL API](sql-api-introduction.md) de Azure Cosmos DB, crear una base de datos de documentos y un contenedor, y agregar datos a dicho contenedor. 

## <a name="prerequisites"></a>Requisitos previos

Una suscripción a Azure o una cuenta de evaluación gratuita de Azure Cosmos DB
- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 

- [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]  

<a id="create-account"></a>
## <a name="create-an-azure-cosmos-db-account"></a>Creación de una cuenta de Azure Cosmos DB

Vaya a [Azure Portal](https://portal.azure.com/) para crear una cuenta de Azure Cosmos DB. Busque y seleccione **Azure Cosmos DB**.

   :::image type="content" source="./media/create-cosmosdb-resources-portal/find-nosql-cosmosdb-marketplace.png" alt-text="El panel de las bases de datos de Azure Portal":::

1. Seleccione **Agregar**.
1. En la página **Creación de una cuenta de Azure Cosmos DB** , especifique la configuración básica de la nueva cuenta de Azure Cosmos. 

    |Configuración|Value|Descripción |
    |---|---|---|
    |Suscripción|Nombre de suscripción|Seleccione la suscripción de Azure que desea usar para esta cuenta de Azure Cosmos. |
    |Grupo de recursos|Definición de un nombre de grupo de recursos|Seleccione un grupo de recursos o seleccione **Crear nuevo** y escriba un nombre único para el grupo de recursos nuevo. |
    |Nombre de cuenta|Un nombre único|Escriba un nombre para identificar la cuenta de Azure Cosmos. Dado que *documents.azure.com* se anexa al nombre que se proporciona para crear el identificador URI, debe usar un nombre único.<br><br>El nombre solo puede contener letras minúsculas, números y el carácter de guion (-). Debe tener una longitud de entre 3 y 31 caracteres.|
    |API|El tipo de cuenta que se va a crear|Seleccione **Core(SQL)** para crear una base de datos de documentos y consultarla mediante la sintaxis SQL. <br><br>La API determina el tipo de cuenta que se va a crear. Azure Cosmos DB proporciona cinco API: Core (SQL) y MongoDB para datos de documento, Gremlin para datos de gráfico, Azure Table y Cassandra. Actualmente, debe crear una cuenta independiente para cada API. <br><br>[Más información acerca de SQL API](introduction.md).|
    |Aplicación de descuento por nivel Gratis|Aplicar o no aplicar|Con el nivel Gratis de Azure Cosmos DB, obtendrá los primeros 400 RU/s y 5 GB de almacenamiento gratis en una cuenta. Más información acerca del [nivel Gratis](https://azure.microsoft.com/pricing/details/cosmos-db/).|
    |Location|Región más cercana a los usuarios|Seleccione una ubicación geográfica para hospedar la cuenta de Azure Cosmos DB. Use la ubicación más cercana a los usuarios para que puedan acceder de la forma más rápida posible a los datos.|
    |Tipo de cuenta|Producción o no producción|Seleccione **Producción** si la cuenta se va a usar para una carga de trabajo de producción. Seleccione **No producción** si la cuenta se va a usar para entornos que no son de producción, como desarrollo, pruebas, control de calidad o almacenamiento provisional. Se trata de una configuración de etiquetas de recursos de Azure que ajusta la experiencia del portal, pero no afecta a la cuenta de Azure Cosmos DB subyacente. Este valor se puede cambiar en cualquier momento.|
    |Redundancia geográfica|Habilitar o deshabilitar|Habilite o deshabilite la distribución global en su cuenta. Para ello, debe emparejar su región con una región de par. Puede agregar más regiones a su cuenta más adelante.|
    |Escrituras de varias regiones|Habilitar o deshabilitar|La funcionalidad de escrituras en varias regiones le permite aprovechar el rendimiento aprovisionado para sus bases de datos y contenedores de todo el mundo.|
    |Zonas de disponibilidad|Habilitar o deshabilitar|Las zonas de disponibilidad le ayudan a mejorar aún más la disponibilidad y la resistencia de una aplicación.|


> [!NOTE]
> Puede tener una cuenta de Azure Cosmos DB de nivel Gratis por cada suscripción de Azure y debe optar por tenerla al crear la cuenta. Si no ve la opción para aplicar el descuento por nivel Gratis, significará que en otra cuenta de la suscripción ya se ha habilitado el nivel Gratis.
   
   :::image type="content" source="./media/create-cosmosdb-resources-portal/azure-cosmos-db-create-new-account-detail.png" alt-text="Página de la nueva cuenta de Azure Cosmos DB":::

1. Seleccione **Revisar + crear**. Puede omitir las secciones **Red** y **Etiquetas**.

1. Revise la configuración de la cuenta y seleccione **Crear**. La operación de creación de la cuenta tarda unos minutos. Espere hasta que la página del portal muestre **Se completó la implementación** . 

    :::image type="content" source="./media/create-cosmosdb-resources-portal/azure-cosmos-db-account-deployment-successful.png" alt-text="Panel de notificaciones de Azure Portal":::

1. Seleccione **Ir al recurso** para ir a la página de la cuenta de Azure Cosmos DB. 

    :::image type="content" source="./media/create-cosmosdb-resources-portal/azure-cosmos-db-account-quickstart-pane.png" alt-text="Página de la cuenta de Azure Cosmos DB":::

<a id="create-container-database"></a>
## <a name="add-a-database-and-a-container"></a>Adición de una base de datos y un contenedor 

Para crear una base de datos y un contenedor, puede usar Data Explorer en Azure Portal. 

1.  Seleccione **Data Explorer** en el panel de navegación izquierdo de la página de la cuenta de Azure Cosmos DB y, después, seleccione **Nuevo contenedor**. 
    
    Es posible que deba desplazarse hacia la derecha para ver la ventana **Agregar contenedor**.
    
    :::image type="content" source="./media/create-sql-api-dotnet/azure-cosmosdb-data-explorer-dotnet.png" alt-text="Explorador de datos de Azure Portal, panel Agregar contenedor":::
    
1.  En el panel **Agregar contenedor**, especifique la configuración del nuevo contenedor.
    
    |Configuración|Valor sugerido|Descripción
    |---|---|---|
    |**Identificador de base de datos**|ToDoList|Escriba *ToDoList* como nombre de la nueva base de datos. Los nombres de base de datos tienen que tener entre 1 y 255 caracteres y no pueden contener `/, \\, #, ?` o espacios finales. Seleccione la opción **Provision database throughput** (Aprovisionar rendimiento de la base de datos) que le permite compartir el rendimiento aprovisionado de la base de datos entre todos los contenedores de esta. Esta opción también le ayudará a reducir los costos. |
    |**Rendimiento**|400|Deje el rendimiento en 400 unidades de solicitud por segundo (RU/s). Si quiere reducir la latencia, puede escalar verticalmente el rendimiento más adelante.| 
    |**ID de contenedor**|Elementos|Escriba *Elementos* como nombre del nuevo contenedor. Los identificadores de contenedor tienen los mismos requisitos de caracteres que los nombres de las bases de datos.|
    |**Clave de partición**| /categoría| El ejemplo que se describe en este artículo usa */category* como clave de partición.|

    
    No agregue **claves únicas** en este ejemplo. Las claves únicas le permiten agregar una capa de integridad de datos a la base de datos mediante la garantía de unicidad de uno o varios valores por clave de partición. Para más información, consulte [Claves únicas en Azure Cosmos DB](unique-keys.md).
    
1.  Seleccione **Aceptar**. Data Explorer muestra la nueva base de datos y el contenedor que ha creado.

## <a name="add-data-to-your-database"></a>Adición de datos a la base de datos

Agregue datos a una base de datos nueva mediante Data Explorer.

1. En **Data Explorer**, expanda la base de datos **ToDoList** y el contenedor **Items**. A continuación, seleccione **Elementos** y, después, **Nuevo elemento**. 
   
   :::image type="content" source="./media/create-sql-api-dotnet/azure-cosmosdb-new-document.png" alt-text="Creación de documentos en el Explorador de datos En Azure Portal":::
   
1. Agregue la siguiente estructura al documento de la derecha del panel **Documentos**:

     ```json
     {
         "id": "1",
         "category": "personal",
         "name": "groceries",
         "description": "Pick up apples and strawberries.",
         "isComplete": false
     }
     ```

1. Seleccione **Guardar**.
   
   :::image type="content" source="./media/create-sql-api-dotnet/azure-cosmosdb-save-document.png" alt-text="Copiar los datos JSON y seleccionar Guardar en el Explorador de datos en Azure Portal":::
   
1. Vuelva a seleccionar **Nuevo documento** y cree y guarde otro documento con un valor de `id` único y las restantes propiedades y valores que desee. Los documentos pueden tener cualquier estructura, ya que Azure Cosmos DB no impone ningún esquema en los datos.

## <a name="query-your-data"></a>Consulta de los datos

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)] 

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

Si desea eliminar solo la base de datos y usar la cuenta de Azure Cosmos en el futuro, puede eliminar la base de datos con los pasos siguientes:

* Vaya a la cuenta de Azure Cosmos.
* Abra **Explorador de datos**, haga clic con el botón derecho en la base de datos que desee eliminar y seleccione **Eliminar base de datos**.
* Escriba el nombre de la base de datos o su identificador para confirmar la eliminación. 

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido ha aprendido a crear una cuenta de Azure Cosmos DB, una base de datos y un contenedor con el Explorador de datos. Ahora puede importar datos adicionales en la cuenta de Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos a Azure Cosmos DB](import-data.md)
