---
title: Enlace de Azure Cosmos DB con la aplicación Azure Spring Cloud
description: Aprenda a enlazar Azure Cosmos DB con la aplicación Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 10/06/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 881005c2597eadc3b3b0be9a01fbf9d82d35d050
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87070773"
---
# <a name="bind-an-azure-cosmos-db-database-to-your-azure-spring-cloud-application"></a>Enlace de una base de datos de Azure Cosmos DB a una aplicación de Azure Spring Cloud

En lugar de configurar manualmente las aplicaciones de Spring Boot, puede enlazar automáticamente servicios de Azure seleccionados a las aplicaciones mediante Azure Spring Cloud. En este artículo se muestra cómo enlazar la aplicación a una base de datos de Azure Cosmos DB.

Requisitos previos:

* Una instancia de Azure Spring Cloud implementada. Para comenzar, siga nuestro [inicio rápido sobre la implementación mediante la CLI de Azure](spring-cloud-quickstart-launch-app-cli.md).
* Una cuenta de Azure Cosmos DB con un nivel mínimo de permisos de colaborador.

## <a name="bind-azure-cosmos-db"></a>Enlace de Azure Cosmos DB

Azure Cosmos DB tiene cinco tipos de API diferentes que admiten el enlace. En el procedimiento siguiente se muestra cómo usarlas:

1. Crea una base de datos de Azure Cosmos DB. Consulte el inicio rápido sobre la [creación de una base de datos](https://docs.microsoft.com/azure/cosmos-db/create-cosmosdb-resources-portal) para obtener ayuda. 

1. Registre el nombre de la base de datos. En este procedimiento, el nombre de la base de datos es **testdb**.

1. Agregue una de las siguientes dependencias al archivo pom.xml de la aplicación de Azure Spring Cloud. Elija la dependencia que sea adecuada para el tipo de API.

    * Tipo de API: Core (SQL)

      ```xml
      <dependency>
          <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-cosmosdb-spring-boot-starter</artifactId>
          <version>2.1.6</version>
      </dependency>
      ```

    * Tipo de API: MongoDB

      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-mongodb</artifactId>
      </dependency>
      ```

    * Tipo de API: Cassandra

      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-cassandra</artifactId>
      </dependency>
      ```

    * Tipo de API: Gremlin (graph)

      ```xml
      <dependency>
          <groupId>com.microsoft.spring.data.gremlin</groupId>
          <artifactId>spring-data-gremlin</artifactId>
          <version>2.1.7</version>
      </dependency>
      ```

    * Tipo de API: tabla de Azure

      ```xml
      <dependency>
          <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-storage-spring-boot-starter</artifactId>
          <version>2.0.5</version>
      </dependency>
      ```

1. Use `az spring-cloud app update` para actualizar la implementación actual o `az spring-cloud app deployment create` para crear una implementación. Estos comandos actualizarán o crearán la aplicación con la nueva dependencia.

1. Vaya a la página del servicio Azure Spring Cloud en Azure Portal. Vaya a **Application Dashboard** (Panel de la aplicación) y seleccione la aplicación que va a enlazar a Azure Cosmos DB. Esta aplicación es la misma que actualizó o implementó en el paso anterior.

1. Seleccione **Service binding** (Enlace de servicio) y seleccione **Create service binding** (Crear enlace de servicio). Para rellenar el formulario, seleccione:
   * El valor de **Binding type** (Tipo de enlace) de **Azure Cosmos DB**.
   * El tipo de API.
   * El nombre de la base de datos.
   * La cuenta de Azure Cosmos DB.

    > [!NOTE]
    > Si usa Cassandra, utilice un espacio de claves para el nombre de la base de datos.

1. Seleccione **Restart** (Reiniciar) en la página de la aplicación para reiniciar la aplicación.

1. Para tener la seguridad de que el servicio esté enlazado correctamente, seleccione el nombre del enlace y compruebe sus detalles. El campo `property` debe ser parecido a este ejemplo:

    ```
    azure.cosmosdb.uri=https://<some account>.documents.azure.com:443
    azure.cosmosdb.key=abc******
    azure.cosmosdb.database=testdb
    ```

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido a enlazar una aplicación de Azure Spring Cloud a una base de datos de Azure Cosmos DB. Para más información sobre el enlace de servicios a una aplicación, consulte [Enlace a una caché de Azure Cache for Redis](spring-cloud-tutorial-bind-redis.md).
