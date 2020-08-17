---
title: Creación de un ejemplo de la aplicación SDK de Azure Cosmos DB para Java v4 de un extremo a otro mediante la fuente de cambios
description: Esta guía le orienta por una sencilla aplicación de SQL API de Java que inserta documentos en un contenedor de Azure Cosmos DB, mientras mantiene una vista materializada del contenedor mediante la fuente de cambios.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: how-to
ms.date: 06/11/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 3f2dcefa8ed2f4b80ec66851cdc67ee2283a6ac7
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322829"
---
# <a name="how-to-create-a-java-application-that-uses-azure-cosmos-db-sql-api-and-change-feed-processor"></a>Cómo crear una aplicación de Java que utiliza SQL API de Azure Cosmos DB y el procesador de fuente de cambios

Esta guía de procedimientos le orienta por una sencilla aplicación de Java que utiliza SQL API de Azure Cosmos DB para insertar documentos en un contenedor de Azure Cosmos DB, mientras mantiene una vista materializada del contenedor mediante la fuente de cambios y el procesador de fuente de cambios. La aplicación de Java se comunica con SQL API de Azure Cosmos DB mediante el SDK de Azure Cosmos DB para Java v4.

> [!IMPORTANT]  
> Este tutorial solo es para el SDK de Azure Cosmos DB para Java v4. Consulte las [notas de la versión](sql-api-sdk-java-v4.md) del SDK de Azure Cosmos DB para Java v4, el [repositorio de Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos), las [sugerencias de rendimiento](performance-tips-java-sdk-v4-sql.md) del SDK de Azure Cosmos DB para Java v4 y la [guía de solución de problemas](troubleshoot-java-sdk-v4-sql.md) del SDK de Azure Cosmos DB para Java v4 para más información. Si en la actualidad usa una versión anterior a v4, vea la guía [Migración al SDK de Azure Cosmos DB para Java v4](migrate-java-v4-sdk.md) a fin de obtener ayuda para actualizar a v4.
>

## <a name="prerequisites"></a>Requisitos previos

* URI y clave de la cuenta de Azure Cosmos DB

* Maven

* Java 8

## <a name="background"></a>Información previa

La fuente de cambios de Azure Cosmos DB proporciona una interfaz orientada a eventos para desencadenar acciones en respuesta a la inserción de documentos. Esto tiene muchas utilidades. Por ejemplo, en aplicaciones en las que se realizan gran cantidad de operaciones de lectura y escritura, la fuente de cambios se usa principalmente para crear una **vista materializada** en tiempo real de un contenedor durante la ingesta de documentos. El contenedor de la vista materializada contendrá los mismos datos pero particionados para facilitar las lecturas, lo que garantizará la eficacia de las operaciones de lectura y escritura de la aplicación.

La biblioteca de procesadores de fuente de cambios integrada en el SDK se ocupa en gran medida del trabajo de administración de eventos de la fuente de cambios. Esta biblioteca es lo suficientemente eficaz como para distribuir eventos de la fuente de cambios entre varios trabajos, si así se desea. Todo lo que tiene que hacer es proporcionar una devolución de llamada a la biblioteca de fuentes de cambios.

En este sencillo ejemplo se muestra cómo la biblioteca de procesadores de fuente de cambios con un solo trabajo crea y elimina documentos de una vista materializada.

## <a name="setup"></a>Configurar

Si todavía no lo ha hecho, clone el repositorio de ejemplo de la aplicación:

```bash
git clone https://github.com/Azure-Samples/azure-cosmos-java-sql-app-example.git
```

Abra un terminal en el directorio del repositorio. Ejecute lo siguiente para compilar la aplicación:

```bash
mvn clean package
```

## <a name="walkthrough"></a>Tutorial

1. Como primer requisito, debe tener una cuenta de Azure Cosmos DB. Abra **Azure Portal** en el explorador, vaya a la cuenta de Azure Cosmos DB y, en el panel izquierdo, vaya a **Explorador de datos**.

   :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_account_empty.JPG" alt-text="Cuenta de Azure Cosmos DB":::

1. Ejecute la aplicación en el terminal con los siguientes comandos:

    ```bash
    mvn exec:java -Dexec.mainClass="com.azure.cosmos.workedappexample.SampleGroceryStore" -DACCOUNT_HOST="your-account-uri" -DACCOUNT_KEY="your-account-key" -Dexec.cleanupDaemonThreads=false
    ```

1. Presione Entrar cuando vea lo siguiente:

    ```bash
    Press enter to create the grocery store inventory system...
    ```

    A continuación, vuelva al Explorador de datos de Azure Portal en el explorador. Verá que se ha agregado la base de datos **GroceryStoreDatabase** con tres contenedores vacíos: 

    * **InventoryContainer**: el registro de inventario de nuestra tienda de comestibles de ejemplo, con particiones en el elemento ```id```, que es un UUID.
    * **InventoryContainer-pktype**: una vista materializada del registro de inventario, optimizada para consultas sobre el elemento ```type```.
    * **InventoryContainer-leases**: un contenedor de concesiones siempre es necesario para la fuente de cambios; las concesiones realizan un seguimiento del progreso de la aplicación en la lectura de la fuente de cambios.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_account_resources_lease_empty.JPG" alt-text="Contenedores vacíos":::

1. En el terminal, ahora debería aparecer un mensaje:

    ```bash
    Press enter to start creating the materialized view...
    ```

    Presione Entrar. Ahora, el siguiente bloque de código ejecutará e inicializará el procesador de fuente de cambios en otro subproceso: 

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>SDK para Java v4 (Maven com.azure::azure-cosmos) API asincrónica

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=InitializeCFP)]

    ```"SampleHost_1"``` es el nombre del trabajo del procesador de fuente de cambios. ```changeFeedProcessorInstance.start()``` es lo que realmente inicia el procesador de fuente de cambios.

    Vuelva al Explorador de datos de Azure Portal en el explorador. En el contenedor **InventoryContainer-leases**, haga clic en **Elementos** para ver su contenido. Verá que el procesador de fuente de cambios ha rellenado el contenedor de concesiones; es decir, el procesador ha asignado una concesión al trabajo ```SampleHost_1``` en algunas particiones del contenedor **InventoryContainer**.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_leases.JPG" alt-text="Concesiones":::

1. Presione Entrar de nuevo en el terminal. Esto desencadenará 10 documentos que se insertarán en **InventoryContainer**. Cada inserción de documento aparece en la fuente de cambios como JSON; el siguiente código de devolución de llamada controla estos eventos mediante el reflejo los documentos JSON en una vista materializada:

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>SDK para Java v4 (Maven com.azure::azure-cosmos) API asincrónica

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=CFPCallback)]

1. Deje que el código se ejecute entre 5 y 10 segundos. A continuación, vuelva al Explorador de datos de Azure Portal y vaya a **InventoryContainer > Elementos**. Debería ver que los elementos se insertan en el contenedor de inventario. Anote la clave de partición (```id```).

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_items.JPG" alt-text="Contenedor de la fuente":::

1. Ahora, en el Explorador de datos, vaya a **InventoryContainer-pktype > Elementos**. Esta es la vista materializada: los elementos de este contenedor reflejan el contenedor **InventoryContainer** porque se han insertado mediante programación con la fuente de cambios. Anote la clave de partición (```type```). Esta vista materializada está optimizada para el filtrado de consultas por ```type```, lo que sería ineficaz en **InventoryContainer** porque está particionado en ```id```.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_materializedview2.JPG" alt-text="Vista materializada":::

1. Vamos a eliminar un documento de **InventoryContainer** y de **InventoryContainer-pktype** usando simplemente una única llamada de ```upsertItem()```. En primer lugar, observe el Explorador de datos de Azure Portal. Eliminaremos el documento para el que ```/type == "plums"``` aparece en un recuadro rojo a continuación

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_materializedview-emph-todelete.JPG" alt-text="Vista materializada":::

    Presione Entrar de nuevo para llamar a la función ```deleteDocument()``` en el código de ejemplo. Esta función, que se muestra a continuación, actualiza o inserta una nueva versión del documento con ```/ttl == 5```, que establece el período de vida (TTL) del documento en 5 segundos. 
    
    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>SDK para Java v4 (Maven com.azure::azure-cosmos) API asincrónica

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=DeleteWithTTL)]

    La fuente de cambios ```feedPollDelay``` está establecida en 100 ms; por lo tanto, la fuente de cambios responde a esta actualización casi al instante y llama a ```updateInventoryTypeMaterializedView()``` mostrado anteriormente. Esa última llamada de función actualizará o insertará el nuevo documento con un período de vida de 5 segundos en **InventoryContainer-pktype**.

    En consecuencia, después de unos 5 segundos, el documento expirará y se eliminará de ambos contenedores.

    Este procedimiento es necesario porque la fuente de cambios solo emite eventos al insertar o actualizar elementos, no al eliminarlos.

1. Presione Entrar una vez más para cerrar el programa y limpiar sus recursos.
