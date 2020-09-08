---
title: 'Tutorial: Migración de los datos a una cuenta de Cassandra API en Azure Cosmos DB'
description: En este tutorial aprenderá a usar el comando Copy de CQL y Spark para copiar datos de Apache Cassandra a la cuenta de Cassandra API en Azure Cosmos DB
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 12/03/2018
ms.custom: seodec18
ms.openlocfilehash: b450d8463c09647b8f87547afe9d714c4b8659c4
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "82858558"
---
# <a name="tutorial-migrate-your-data-to-cassandra-api-account-in-azure-cosmos-db"></a>Tutorial: Migración de los datos a una cuenta de Cassandra API en Azure Cosmos DB

Como desarrollador, es posible que tenga cargas de trabajo de Cassandra existentes que se ejecutan de forma local o en la nube y puede que desee migrarlas a Azure. Puede migrar estas cargas a una cuenta de Cassandra API en Azure Cosmos DB. Este tutorial proporciona instrucciones sobre las diferentes opciones disponibles para migrar datos de Apache Cassandra a la cuenta de Cassandra API en Azure Cosmos DB.

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Planear la migración
> * Requisitos previos para la migración
> * Migrar datos mediante el comando COPY de cqlsh
> * Migrar datos con Spark

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites-for-migration"></a>Requisitos previos para la migración

* **Cálculo de las necesidades de rendimiento:** Antes de migrar datos a la cuenta de Cassandra API en Azure Cosmos DB, debe calcular las necesidades de rendimiento de la carga de trabajo. En general, se recomienda comenzar con el rendimiento medio requerido por las operaciones CRUD y después incluir el rendimiento adicional necesario para las operaciones de picos o Extraer carga de transformación (ETL). Necesita los siguientes detalles para planear la migración: 

  * **Tamaño de datos existente o tamaño de datos estimado:** Define el requisito de rendimiento y de tamaño mínimo de la base de datos. Si realiza la estimación de tamaño de los datos para una nueva aplicación, puede asumir que los datos se distribuyen uniformemente entre las filas y calcular el valor multiplicando con el tamaño de los datos. 

  * **Rendimiento necesario:** Tasa de rendimiento de lectura (consultar/obtener) y escritura (actualizar/eliminar/insertar) aproximada. Este valor es necesario para calcular las unidades de solicitud necesarios junto con el tamaño de datos de estado estable.  

  * **El esquema:** Conéctese a su clúster de Cassandra existente mediante cqlsh y exporte el esquema desde Cassandra: 

    ```bash
    cqlsh [IP] "-e DESC SCHEMA" > orig_schema.cql
    ```

    Después de identificar los requisitos de carga de trabajo existentes, debe crear una cuenta de Azure Cosmos DB, una base de datos y varios contenedores según los requisitos de rendimiento recopilados.  

  * **Determinación del cargo de RU para una operación:** Puede determinar las RU mediante el uso de cualquiera de los SDK compatible con Cassandra API. En este ejemplo se muestra la versión de .NET de la obtención de cargos de la unidad de solicitud.

    ```csharp
    var tableInsertStatement = table.Insert(sampleEntity);
    var insertResult = await tableInsertStatement.ExecuteAsync();

    foreach (string key in insertResult.Info.IncomingPayload)
      {
         byte[] valueInBytes = customPayload[key];
         double value = Encoding.UTF8.GetString(valueInBytes);
         Console.WriteLine($"CustomPayload:  {key}: {value}");
      }
    ```

* **Asignación del rendimiento necesario:** Azure Cosmos DB puede escalar automáticamente el almacenamiento y rendimiento a medida que crecen sus necesidades. Puede estimar sus necesidades de rendimiento mediante el uso de la [Calculadora de unidades de solicitud de Azure Cosmos DB](https://www.documentdb.com/capacityplanner). 

* **Creación de tablas en la cuenta de Cassandra API:** Antes de comenzar la migración de datos, cree previamente todas las tablas desde Azure Portal o desde cqlsh. Si va a migrar a una cuenta de Azure Cosmos DB que tiene un rendimiento de nivel de base de datos, asegúrese de proporcionar una clave de partición al crear los contenedores de Azure Cosmos DB.

* **Aumento del rendimiento:** La duración de la migración de datos depende de la cantidad de rendimiento aprovisionado para las tablas de Azure Cosmos DB. Aumente el rendimiento de la duración de la migración. Gracias al mayor rendimiento, puede evitar la limitación de velocidad y realizar la migración en menos tiempo. Después de haber completado la migración, reduzca el rendimiento para ahorrar costos. También se recomienda tener la cuenta de Azure Cosmos DB en la misma región que la base de datos de origen. 

* **Habilitación de TLS:** Azure Cosmos DB tiene estándares y requisitos de seguridad estrictos. No olvide habilitar TLS al interactuar con la cuenta. Al usar CQL con SSH, tiene una opción para proporcionar información de TLS.

## <a name="options-to-migrate-data"></a>Opciones para migrar datos

Puede mover los datos de cargas de trabajo existentes de Cassandra para Azure Cosmos DB mediante el uso de las siguientes opciones:

* [Usar el comando COPY de cqlsh](#migrate-data-using-cqlsh-copy-command)  
* [Usar Spark](#migrate-data-using-spark) 

## <a name="migrate-data-using-cqlsh-copy-command"></a>Migrar datos mediante el comando COPY de cqlsh

[El comando COPY de CQL](https://cassandra.apache.org/doc/latest/tools/cqlsh.html#cqlsh) se usa para copiar datos locales en la cuenta de Cassandra API en Azure Cosmos DB. Use los siguientes pasos para copiar datos:

1. Obtenga la información de cadena de conexión de su cuenta de Cassandra API:

   * Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la cuenta de Azure Cosmos DB.

   * Abra el panel **Cadena de conexión** que contenga toda la información que necesite para conectarse a su cuenta de API de Cassandra desde cqlsh.

2. Inicie sesión en cqlsh mediante la información de conexión desde el portal.

3. Use el comando COPY de CQL para copiar datos locales a la cuenta de API de Cassandra.

   ```bash
   COPY exampleks.tablename FROM filefolderx/*.csv 
   ```

## <a name="migrate-data-using-spark"></a>Migrar datos con Spark 

Siga estos pasos para migrar datos a la cuenta de Cassandra API en Azure Cosmos DB con Spark:

- Aprovisione un [clúster de Azure Databricks](cassandra-spark-databricks.md) o un [clúster de HDInsight](cassandra-spark-hdinsight.md) 

- Mueva los datos al punto de conexión de Cassandra API de destino mediante la [operación de copia de tabla](cassandra-spark-table-copy-ops.md) 

La migración de datos mediante el uso de trabajos de Spark es una opción recomendada si tiene datos que residen en un clúster existente en máquinas virtuales de Azure o cualquier otra nube. Esta opción requiere que Spark se configure como intermediario para la ingesta única o regular. Puede acelerar la migración mediante el uso de conectividad de Azure ExpressRoute entre local y Azure. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar el grupo de recursos, la cuenta de Azure Cosmos DB y todos los recursos relacionados. Para ello, seleccione el grupo de recursos de la máquina virtual, seleccione **Eliminar** y luego confirme el nombre del grupo de recursos que va a eliminar.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo migrar los datos a una cuenta de Cassandra API en Azure Cosmos DB. Ahora puede continuar con el siguiente artículo para obtener información sobre otros conceptos de Azure Cosmos DB:

> [!div class="nextstepaction"]
> [Niveles de coherencia de datos optimizables en Azure Cosmos DB](../cosmos-db/consistency-levels.md)


