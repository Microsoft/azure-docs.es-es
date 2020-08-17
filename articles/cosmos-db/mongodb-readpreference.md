---
title: Uso de las preferencias de lectura con la API de Azure Cosmos DB para MongoDB
description: Aprenda a usar las preferencias de lectura de MongoDB con la API de Azure Cosmos DB para MongoDB.
author: sivethe
ms.author: sivethe
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: how-to
ms.date: 02/26/2019
ms.custom: devx-track-javascript
ms.openlocfilehash: 22530e79a2bcf34a88ea857bda56b6f424894461
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422001"
---
# <a name="how-to-globally-distribute-reads-using-azure-cosmos-dbs-api-for-mongodb"></a>Cómo distribuir las lecturas globalmente con la API de Azure Cosmos DB para MongoDB

En este artículo se muestra cómo distribuir globalmente las operaciones de lectura usando la configuración de las [preferencias de lectura de MongoDB](https://docs.mongodb.com/manual/core/read-preference/) con la API de Azure Cosmos DB para MongoDB.

## <a name="prerequisites"></a>Prerrequisitos 
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 
[!INCLUDE [cosmos-db-emulator-mongodb](../../includes/cosmos-db-emulator-mongodb.md)]

Consulte este artículo de [inicio rápido](tutorial-global-distribution-mongodb.md) para obtener instrucciones sobre el uso de Azure Portal para configurar la cuenta de Cosmos con distribución global y, a continuación, conectarse.

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Abra una ventana de terminal de Git, como Git Bash, y `cd` en un directorio de trabajo.  

Ejecute los comandos siguientes para clonar el repositorio de ejemplo. En función de la plataforma de su interés, utilice uno de los repositorios de ejemplo siguientes:

1. [Aplicación de ejemplo de .NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference)
2. [Aplicación de ejemplo de NodeJS]( https://github.com/Azure-Samples/azure-cosmos-db-mongodb-node-geo-readpreference)
3. [Aplicación de ejemplo de Mongoose](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-mongoose-geo-readpreference)
4. [Aplicación de ejemplo de Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference)
5. [Aplicación de ejemplo de SpringBoot](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-spring)


```bash
git clone <sample repo url>
```

## <a name="run-the-application"></a>Ejecución de la aplicación

En función de la plataforma utilizada, instale los paquetes necesarios e inicie la aplicación. Para instalar las dependencias, siga el archivo Léame incluido en el repositorio de la aplicación de ejemplo. Por ejemplo, en la aplicación de ejemplo de NodeJS, utilice los comandos siguientes para instalar los paquetes necesarios e iniciar la aplicación.

```bash
cd mean
npm install
node index.js
```
La aplicación intenta conectarse a un origen de MongoDB y se produce un error porque la cadena de conexión no es válida. Siga los pasos descritos en el archivo Léame para actualizar la dirección `url` de la cadena de conexión. Asimismo, actualice el valor de `readFromRegion` a una región de lectura en la cuenta de Cosmos. Las instrucciones siguientes provienen del ejemplo de NodeJS:

```
* Next, substitute the `url`, `readFromRegion` in App.Config with your Cosmos account's values. 
```

Después de seguir estos pasos, la aplicación de ejemplo se ejecuta y produce la siguiente salida:

```
connected!
readDefaultfunc query completed!
readFromNearestfunc query completed!
readFromRegionfunc query completed!
readDefaultfunc query completed!
readFromNearestfunc query completed!
readFromRegionfunc query completed!
readDefaultfunc query completed!
readFromSecondaryfunc query completed!
```

## <a name="read-using-read-preference-mode"></a>Leer utilizando el modo preferente de lectura

El protocolo de MongoDB proporciona los siguientes modos de preferencias de lectura para su uso por parte de los clientes:

1. PRIMARY
2. PRIMARY_PREFERRED
3. SECONDARY
4. SECONDARY_PREFERRED
5. NEAREST

Consulte la documentación detallada sobre el [comportamiento de las preferencias de lectura de MongoDB](https://docs.mongodb.com/manual/core/read-preference-mechanics/#replica-set-read-preference-behavior) para más información sobre el comportamiento en cada uno de estos modos de preferencias de lectura. En Cosmos DB, primary se asigna a la región de escritura y secondary se asigna a la región de lectura.

Basándose en los escenarios más comunes, se recomienda usar las siguientes opciones:

1. Si son necesarias **lecturas de baja latencia**, use el modo de preferencias de lectura **NEAREST**. Esta opción dirige las operaciones de lectura a la región más cercana disponible. Tenga en cuenta que si la región más cercana es la región de escritura, estas operaciones se dirigen a dicha región.
2. Si necesita **alta disponibilidad y distribución geográfica de las lecturas** (la latencia no es una restricción), use el modo de preferencias de lectura **PRIMARY PREFERRED** o **SECONDARY PREFERRED**. Esta opción dirige las operaciones de lectura a una región de escritura o lectura disponible, respectivamente. Si la región no está disponible, las solicitudes se dirigen a la siguiente región disponible según el comportamiento de las preferencias de lectura.

El siguiente fragmento de código de la aplicación de ejemplo muestra cómo configurar el modo de preferencias de lectura NEAREST en NodeJS:

```javascript
  var query = {};
  var readcoll = client.db('regionDB').collection('regionTest', {readPreference: ReadPreference.NEAREST});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromNearestfunc query completed!");
  });
```

De forma similar, el siguiente fragmento muestra cómo configurar el modo de preferencias de lectura SECONDARY_PREFERRED en NodeJS:

```javascript
  var query = {};
  var readcoll = client.db('regionDB').collection('regionTest', {readPreference: ReadPreference.SECONDARY_PREFERRED});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromSecondaryPreferredfunc query completed!");
  });
```

También se pueden establecer las preferencias de lectura si se pasa `readPreference` como un parámetro en las opciones del identificador URI de la cadena de conexión:

```javascript
const MongoClient = require('mongodb').MongoClient;
const assert = require('assert');

// Connection URL
const url = 'mongodb://localhost:27017?ssl=true&replicaSet=globaldb&readPreference=nearest';

// Database Name
const dbName = 'myproject';

// Use connect method to connect to the Server
MongoClient.connect(url, function(err, client) {
  console.log("Connected correctly to server");

  const db = client.db(dbName);

  client.close();
});
```

Consulte los repositorios de la aplicación de ejemplo correspondientes a otras plataformas, como [.NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference) y [Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference).

## <a name="read-using-tags"></a>Lecturas mediante etiquetas

Además del modo de preferencias de lectura, el protocolo de MongoDB permite el uso de etiquetas para dirigir las operaciones de lectura. En la API de Cosmos DB para MongoDB, la etiqueta `region` se incluye de forma predeterminada como parte de la respuesta `isMaster`:

```json
"tags": {
         "region": "West US"
      }
```

Por lo tanto, el cliente de Mongo puede usar la etiqueta `region` junto con el nombre de la región para dirigir las operaciones de lectura a regiones específicas. Para las cuentas de Cosmos, los nombres de región se pueden encontrar Azure Portal a la izquierda en **Settings->Replica data globally** (Configuración -> Replicar datos globalmente). Esta opción es útil para lograr el **aislamiento de lectura**: casos en los que la aplicación cliente desea dirigir las operaciones de lectura solamente a una región específica. Esta opción es ideal para escenarios de análisis y no de producción, que se ejecutan en segundo plano y no son servicios críticos para la producción.

El siguiente fragmento de código de la aplicación de ejemplo muestra cómo configurar las preferencias de lectura con etiquetas en NodeJS:

```javascript
 var query = {};
  var readcoll = client.db('regionDB').collection('regionTest',{readPreference: new ReadPreference(ReadPreference.SECONDARY_PREFERRED, {"region": "West US"})});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromRegionfunc query completed!");
  });
```

Consulte los repositorios de la aplicación de ejemplo correspondientes a otras plataformas, como [.NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference) y [Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference).

En este artículo ha aprendido cómo distribuir globalmente las operaciones de lectura usando las preferencias de lectura con la API de Azure Cosmos DB para MongoDB.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, siga estos pasos para eliminar todos los recursos creados en este artículo en Azure Portal:

1. En el menú de la izquierda de Azure Portal, haga clic en **Grupos de recursos** y en el nombre del recurso que creó. 
2. En la página del grupo de recursos, haga clic en **Eliminar**, escriba en el cuadro de texto el nombre del recurso que quiere eliminar y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

* [Importación de datos de MongoDB a Azure Cosmos DB](mongodb-migrate.md)
* [Setup a globally distributed database with Azure Cosmos DB's API for MongoDB](tutorial-global-distribution-mongodb.md) (Configurar una base de datos distribuida globalmente con la API de Azure Cosmos DB para MongoDB)
* [Desarrollo local con el Emulador de Azure Cosmos DB](local-emulator.md)
