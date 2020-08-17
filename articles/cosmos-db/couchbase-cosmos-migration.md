---
title: Migración de CouchBase a API de SQL de Azure Cosmos DB
description: Guía paso a paso para migrar desde CouchBase a API de SQL de Azure Cosmos DB
ms.service: cosmos-db
ms.topic: how-to
ms.date: 02/11/2020
ms.author: mansha
author: manishmsfte
ms.custom: devx-track-java
ms.openlocfilehash: b0c9ef99e4cbb0683273d613d3a85e7f6455a40d
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87366728"
---
# <a name="migrate-from-couchbase-to-azure-cosmos-db-sql-api"></a>Migración de CouchBase a API de SQL de Azure Cosmos DB

Azure Cosmos DB es una base de datos totalmente administrada, escalable y distribuida globalmente. Proporciona acceso de baja latencia garantizado a los datos. Para obtener más información sobre Azure Cosmos DB, vea el artículo de [información general](introduction.md). En este artículo se proporcionan instrucciones para migrar aplicaciones de Java conectadas a Couchbase a una cuenta de API de SQL de Azure Cosmos DB.

## <a name="differences-in-nomenclature"></a>Diferencias de nomenclatura

A continuación se indican las principales características que funcionan de manera diferente en Azure Cosmos DB en comparación con Couchbase:

|   Couchbase     |   Azure Cosmos DB   |
| ---------------|-------------------|
|Servidor de Couchbase| Cuenta       |
|Cubo           | Base de datos      |
|Cubo           | Contenedor o colección |
|Documento JSON    | Elemento o documento |

## <a name="key-differences"></a>Principales diferencias

* Azure Cosmos DB tiene un campo "id." en el documento, mientras que Couchbase tiene el identificador como parte del cubo. El campo "id." es único en la partición.

* Azure Cosmos DB escala mediante la técnica de creación de particiones o particionamiento. Eso significa que divide los datos en varias particiones. Estas particiones se crean en función de la propiedad de clave de partición que se proporciona. Se puede seleccionar la clave de partición para optimizar tanto operaciones de lectura como de escritura o también optimizadas para lectura/escritura. Para obtener más información, vea el articulo [Creación de particiones](./partition-data.md).

* En Azure Cosmos DB no es necesario que la jerarquía de nivel superior indique la colección, porque ya existe el nombre de la colección. Esta característica simplifica la estructura JSON. A continuación se proporciona un ejemplo en el que se muestran las diferencias en cuanto al modelo de datos entre Couchbase y Azure Cosmos DB:

   **Couchbase**: Id. de documento = "99FF4444"

    ```json
    {
      "TravelDocument":
      {
       "Country":"India",
      "Validity" : "2022-09-01",
        "Person":
        {
          "Name": "Manish",
          "Address": "AB Road, City-z"
        },
        "Visas":
        [
          {
          "Country":"India",
          "Type":"Multi-Entry",
          "Validity":"2022-09-01"
          },
          {
          "Country":"US",
          "Type":"Single-Entry",
          "Validity":"2022-08-01"
          }
        ]
      }
    }
   ```

   **Azure Cosmos DB**: Vea "id." en el documento, como se muestra a continuación

    ```json
    {
      "id" : "99FF4444",
    
      "Country":"India",
       "Validity" : "2022-09-01",
        "Person":
        {
          "Name": "Manish",
          "Address": "AB Road, City-z"
        },
        "Visas":
        [
          {
          "Country":"India",
          "Type":"Multi-Entry",
          "Validity":"2022-09-01"
          },
          {
          "Country":"US",
          "Type":"Single-Entry",
          "Validity":"2022-08-01"
          }
        ]
      }
    
    ```
         
## <a name="java-sdk-support"></a>Compatibilidad con SDK de Java

Azure Cosmos DB cuenta con los siguientes SDK para admitir distintos marcos de Java:

* SDK asincrónico
* SDK de Spring Boot

En las siguientes secciones se explica cuándo usar cada uno de estos SDK. Considere un ejemplo en el que hay tres tipos de cargas de trabajo:

## <a name="couchbase-as-document-repository--spring-data-based-custom-queries"></a>Couchbase como repositorio de documentos y consultas personalizadas basadas en datos de Spring

Si la carga de trabajo que va a migrar se basa en el SDK basado en Spring Boot, puede seguir estos pasos:

1. Agregue el elemento primario al archivo POM.xml:

   ```java
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.1.5.RELEASE</version>
      <relativePath/>
   </parent>
   ```

1. Agregue las propiedades al archivo POM.xml:

   ```java
   <azure.version>2.1.6</azure.version>
   ```

1. Agregue las dependencias al archivo POM.xml:

   ```java
   <dependency>
       <groupId>com.microsoft.azure</groupId>
       <artifactId>azure-cosmosdb-spring-boot-starter</artifactId>
       <version>2.1.6</version>
   </dependency>
   ```

1. Agregue las propiedades de la aplicación bajo los recursos y especifique lo siguiente. Asegúrese de reemplazar los parámetros de dirección URL, clave y nombre de base de datos:

   ```java
      azure.cosmosdb.uri=<your-cosmosDB-URL>
      azure.cosmosdb.key=<your-cosmosDB-key>
      azure.cosmosdb.database=<your-cosmosDB-dbName>
   ```

1. Defina el nombre de la colección en el modelo. Además puede especificar más anotaciones. Por ejemplo, id. o clave de partición para indicarlas de forma explícita:

   ```java
   @Document(collection = "mycollection")
       public class User {
           @id
           private String id;
           private String firstName;
           @PartitionKey
           private String lastName;
       }
   ```

A continuación se muestran los fragmentos de código de las operaciones CRUD:

### <a name="insert-and-update-operations"></a>Operaciones de inserción y actualización

Donde *_repo* es el objeto de repositorio y *doc* es el objeto de la clase POJO. Puede usar `.save` para insertar o upsert (si se ha encontrado un documento con el id. especificado). En el fragmento de código siguiente se muestra cómo insertar o actualizar un objeto doc:

```_repo.save(doc);```

### <a name="delete-operation"></a>Operación de eliminación

Considere el siguiente fragmento de código, donde el objeto doc tiene el identificador y la clave de partición obligatorios para buscar y eliminar el objeto:

```_repo.delete(doc);```

### <a name="read-operation"></a>Lee una operación

Puede leer el documento con o sin clave de partición. Si no se especifica la clave de partición, se trata como una consulta entre particiones. Considere los siguientes ejemplos de código; en el primero se realiza la operación mediante los campos de id. y clave de partición. En el segundo ejemplo se usa un campo normal y no se especifica el campo de clave de partición.

* ```_repo.findByIdAndName(objDoc.getId(),objDoc.getName());```
* ```_repo.findAllByStatus(objDoc.getStatus());```

Eso es todo, ya puede usar la aplicación con Azure Cosmos DB. El ejemplo de código completo del ejemplo que se describe en este documento está disponible en el repositorio de GitHub [CouchbaseToCosmosDB-SpringCosmos](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/master/SpringCosmos).

## <a name="couchbase-as-a-document-repository--using-n1ql-queries"></a>Couchbase como repositorio de documentos y uso de consultas N1QL

Las consultas N1QL son la manera de definir consultas en Couchbase.

|Consulta N1QL | Consulta de Azure Cosmos DB|
|-------------------|-------------------|
|SELECT META(`TravelDocument`).id AS id, `TravelDocument`.* FROM `TravelDocument` WHERE `_type` = "com.xx.xx.xx.xxx.xxx.xxxx " and country = 'India’ and ANY m in Visas SATISFIES m.type == 'Multi-Entry' and m.Country IN ['India', Bhutan’] ORDER BY ` Validity` DESC LIMIT 25 OFFSET 0   | SELECT c.id,c FROM c JOIN m in c.country=’India’ WHERE c._type = " com.xx.xx.xx.xxx.xxx.xxxx" and c.country = 'India' and m.type = 'Multi-Entry' and m.Country IN ('India', 'Bhutan') ORDER BY c.Validity DESC OFFSET 0 LIMIT 25 |

Puede observar los cambios siguientes en las consultas N1QL:

* No es necesario usar la palabra clave META ni hacer referencia al documento de primer nivel, sino que se puede crear una referencia propia al contenedor. En este ejemplo, se considera como "c" (puede ser cualquier cosa). Esta referencia se usa como prefijo para todos los campos de primer nivel. Por ejemplo, c.id, c.country, etc.

* En lugar de "ANY", ahora se puede realizar una combinación en el subdocumento y hacer referencia a él con un alias dedicado como "m". Una vez creado el alias para un subdocumento, debe usarlo. Por ejemplo, m.Country.

* La secuencia de OFFSET es diferente en la consulta de Azure Cosmos DB, ya que en primer lugar se debe especificar OFFSET y luego LIMIT. Se recomienda no usar el SDK de Spring Data si se emplea un número máximo de consultas definidas personalizadas, ya que eso puede plantear una sobrecarga innecesaria en el cliente mientras se pasa la consulta a Azure Cosmos DB. En su lugar se cuenta con un SDK de Java asincrónico directo que se puede usar de manera muy eficaz en este caso.

### <a name="read-operation"></a>Operación de lectura

Use el SDK de Java asincrónico con los pasos siguientes:

1. Configure la siguiente dependencia en el archivo POM.xml:

   ```java
   <!-- https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb -->
   <dependency>
       <groupId>com.microsoft.azure</groupId>
       <artifactId>azure-cosmos</artifactId>
       <version>3.0.0</version>
   </dependency>
   ```

1. Cree un objeto de conexión para Azure Cosmos DB mediante el método `ConnectionBuilder`, como se muestra en el ejemplo siguiente. Asegúrese de colocar esta declaración en el bean, de modo que el código siguiente solo se ejecute una vez:

   ```java
   ConnectionPolicy cp=new ConnectionPolicy();
   cp.connectionMode(ConnectionMode.DIRECT);
    
   if(client==null)
    client= CosmosClient.builder()
        .endpoint(Host)//(Host, MasterKey, dbName, collName).Builder()
        .connectionPolicy(cp)
        .key(MasterKey)
        .consistencyLevel(ConsistencyLevel.EVENTUAL)
        .build();   
   
   container = client.getDatabase(_dbName).getContainer(_collName);
   ```

1. Para ejecutar la consulta, debe ejecutar el siguiente fragmento de código:

   ```java
   Flux<FeedResponse<CosmosItemProperties>> objFlux= container.queryItems(query, fo);
   ```

Ahora, con la ayuda del método anterior, puede pasar varias consultas y ejecutar sin complicaciones. En caso de que tenga el requisito de ejecutar una consulta grande que se puede dividir en varias consultas, pruebe el fragmento de código siguiente en lugar del anterior:

```java
for(SqlQuerySpec query:queries)
{
    objFlux= container.queryItems(query, fo);
    objFlux .publishOn(Schedulers.elastic())
            .subscribe(feedResponse->
                {
                    if(feedResponse.results().size()>0)
                    {
                        _docs.addAll(feedResponse.results());
                    }
                
                },
                Throwable::printStackTrace,latch::countDown);
    lstFlux.add(objFlux);
}
                        
        Flux.merge(lstFlux);
        latch.await();
}
```

Con el código anterior, puede ejecutar consultas en paralelo y aumentar las ejecuciones distribuidas para optimizar. También puede ejecutar las operaciones de inserción y actualización:

### <a name="insert-operation"></a>Operación de inserción

Para insertar el documento, ejecute el siguiente código:

```java 
Mono<CosmosItemResponse> objMono= container.createItem(doc,ro);
```

Luego suscríbase a Mono como:

```java
CountDownLatch latch=new CountDownLatch(1);
objMono .subscribeOn(Schedulers.elastic())
        .subscribe(resourceResponse->
        {
            if(resourceResponse.statusCode()!=successStatus)
                {
                    throw new RuntimeException(resourceResponse.toString());
                }
            },
        Throwable::printStackTrace,latch::countDown);
latch.await();              
```

### <a name="upsert-operation"></a>Operación de upsert

La operación de upsert exige que se especifique el documento que se debe actualizar. Para capturar el documento completo, puede usar el fragmento de código mencionado en el encabezado Operación de lectura y modificar los campos obligatorios. El siguiente fragmento de código realiza la operación de upsert en el documento:

```java
Mono<CosmosItemResponse> obs= container.upsertItem(doc, ro);
```
Luego suscríbase a Mono. Vea el fragmento de código de suscripción a Mono de la operación de inserción.

### <a name="delete-operation"></a>Operación de eliminación

El siguiente fragmento de código realiza la operación de eliminación:

```java     
CosmosItem objItem= container.getItem(doc.Id, doc.Tenant);
Mono<CosmosItemResponse> objMono = objItem.delete(ro);
```

Luego, para suscribirse a Mono, vea el fragmento de código de suscripción a Mono de la operación de inserción. El ejemplo de código completo está disponible en el repositorio de GitHub [CouchbaseToCosmosDB-AsyncInSpring](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/master/AsyncInSpring).

## <a name="couchbase-as-a-keyvalue-pair"></a>Couchbase como par clave-valor

Se trata de un tipo simple de carga de trabajo en el que se pueden realizar búsquedas en lugar de consultas. Use los pasos siguientes para los pares clave-valor:

1. Considere convertir "/ID" en la clave principal, con lo que se aseguraría de poder realizar operaciones de búsqueda directamente en la partición específica. Cree una colección y especifique "/ID" como clave de partición.

1. Desactive la indexación por completo. Dado que va a ejecutar operaciones de búsqueda, no hay motivo para agregar sobrecarga de indexación. Para desactivar la indexación, inicie sesión en Azure Portal y vaya a Cuenta de Azure Cosmos DB. Abra el **Explorador de datos**, seleccione la **Base de datos** y el **Contenedor**. Abra la pestaña **Escala y configuración** y seleccione la **Directiva de indexación**. En este momento, la directiva de indexación tiene esta aspecto:
    
   ```json
   {
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/\"_etag\"/?"
        }
    ]
    }
   ````

   Reemplace la directiva de indexación anterior por la siguiente:

   ```json
   {
    "indexingMode": "none",
    "automatic": false,
    "includedPaths": [],
    "excludedPaths": []
    }
   ```

1. Use el siguiente fragmento de código para crear el objeto de conexión. Objeto de conexión (para colocar en @Bean o convertirlo en estático):

   ```java
   ConnectionPolicy cp=new ConnectionPolicy();
   cp.connectionMode(ConnectionMode.DIRECT);
   
   if(client==null)
    client= CosmosClient.builder()
        .endpoint(Host)//(Host, MasterKey, dbName, collName).Builder()
        .connectionPolicy(cp)
        .key(MasterKey)
        .consistencyLevel(ConsistencyLevel.EVENTUAL)
        .build();
    
   container = client.getDatabase(_dbName).getContainer(_collName);
   ```

Ahora puede ejecutar las operaciones CRUD de la siguiente manera:

### <a name="read-operation"></a>Operación de lectura

Para leer el elemento, use el siguiente fragmento de código:

```java        
CosmosItemRequestOptions ro=new CosmosItemRequestOptions();
ro.partitionKey(new PartitionKey(documentId));
CountDownLatch latch=new CountDownLatch(1);
        
var objCosmosItem= container.getItem(documentId, documentId);
Mono<CosmosItemResponse> objMono = objCosmosItem.read(ro);
objMono .subscribeOn(Schedulers.elastic())
        .subscribe(resourceResponse->
        {
            if(resourceResponse.item()!=null)
            {
                doc= resourceResponse.properties().toObject(UserModel.class);
            }
        },
        Throwable::printStackTrace,latch::countDown);
latch.await();
```

### <a name="insert-operation"></a>Operación de inserción

Para insertar un elemento, puede ejecutar el siguiente código:

```java 
Mono<CosmosItemResponse> objMono= container.createItem(doc,ro);
```

Luego suscríbase a Mono como:

```java
CountDownLatch latch=new CountDownLatch(1);
objMono.subscribeOn(Schedulers.elastic())
        .subscribe(resourceResponse->
        {
            if(resourceResponse.statusCode()!=successStatus)
                {
                    throw new RuntimeException(resourceResponse.toString());
                }
            },
        Throwable::printStackTrace,latch::countDown);
latch.await();
```

### <a name="upsert-operation"></a>Operación de upsert

Para actualizar el valor de un elemento, vea el siguiente fragmento de código:

```java
Mono<CosmosItemResponse> obs= container.upsertItem(doc, ro);
```
Luego, para suscribirse a Mono, vea el fragmento de código de suscripción a Mono de la operación de inserción.

### <a name="delete-operation"></a>Operación de eliminación

Use el siguiente fragmento de código para ejecutar la operación de eliminación:

```java     
CosmosItem objItem= container.getItem(id, id);
Mono<CosmosItemResponse> objMono = objItem.delete(ro);
```

Luego, para suscribirse a Mono, vea el fragmento de código de suscripción a Mono de la operación de inserción. El ejemplo de código completo está disponible en el repositorio de GitHub [CouchbaseToCosmosDB-AsyncKeyValue](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/master/AsyncKeyValue).

## <a name="data-migration"></a>Migración de datos

Existen dos formas de migrar datos.

* **Usar Azure Data Factory:** este es el método más recomendado para migrar los datos. Configure el origen como Couchbase y sincronice como API de SQL de Azure Cosmos DB, vea el artículo de Azure [Conector Data Factory de Cosmos DB](../data-factory/connector-azure-cosmos-db.md) para obtener pasos detallados.

* **Usar la herramienta de importación de datos de Azure Cosmos DB:** se recomienda esta opción para migrar mediante máquinas virtuales con menos cantidad de datos. Para obtener pasos detallados, vea el artículo [Importador de datos](./import-data.md).

## <a name="next-steps"></a>Pasos siguientes

* Para realizar pruebas de rendimiento, vea el artículo [Pruebas de escala y rendimiento con Azure Cosmos DB](./performance-testing.md).
* Para optimizar el código, vea el artículo [Sugerencias de rendimiento para Azure Cosmos DB](./performance-tips-async-java.md).
* Para examinar el SDK de Java asincrónico versión 3, vea el repositorio de GitHub [Referencia de SDK](https://github.com/Azure/azure-cosmosdb-java/tree/v3).
