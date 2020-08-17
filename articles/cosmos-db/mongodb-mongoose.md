---
title: Conexión de una aplicación Mongoose de Node.js a Azure Cosmos DB
description: Obtenga información sobre cómo usar el marco Mongoose para almacenar y administrar datos en Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: how-to
ms.date: 03/20/2020
author: timsander1
ms.author: tisande
ms.custom: seodec18, devx-track-javascript
ms.openlocfilehash: eb91b6108cedd12299fcb3cfa89bf2cc91bcd69f
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422018"
---
# <a name="connect-a-nodejs-mongoose-application-to-azure-cosmos-db"></a>Conexión de una aplicación Mongoose de Node.js a Azure Cosmos DB

En este tutorial se muestra cómo usar el [marco Mongoose](https://mongoosejs.com/) al almacenar datos en Cosmos DB. En este tutorial usaremos la API de Azure Cosmos DB para MongoDB. Si no le resulta familiar, Mongoose es un marco de modelado de objetos de MongoDB en Node.js que proporciona una solución sencilla y basada en esquemas para dar forma a los datos de la aplicación.

Cosmos DB es el servicio de base de datos de varios modelos de distribución global de Microsoft. Puede crear rápidamente bases de datos de documentos, clave-valor y grafos, y realizar consultas en ellas. Todas las bases de datos se beneficiarán de las funcionalidades de distribución global y escala horizontal en Cosmos DB.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

[Node.js](https://nodejs.org/) versión v0.10.29 o superior

## <a name="create-a-cosmos-account"></a>Crear una cuenta de Cosmos

Cree una cuenta de Cosmos. Si ya tiene una cuenta que desea usar, puede ir directamente a Configuración de la aplicación de Node.js. Si usa el emulador de Azure Cosmos DB, siga los pasos que se indican en [Emulador de Azure Cosmos DB](local-emulator.md) para configurar el emulador y vaya directamente a Configuración de la aplicación de Node.js.

[!INCLUDE [cosmos-db-create-dbaccount-mongodb](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

### <a name="create-a-database"></a>Crear una base de datos 
En esta aplicación se tratarán dos maneras de crear colecciones en Azure Cosmos DB: 
- **Almacenar cada modelo de objetos en una colección independiente**: Se recomienda [crear una base de datos con rendimiento dedicado](set-throughput.md#set-throughput-on-a-database). El uso de este modelo de capacidad le proporcionará una mayor rentabilidad.

    :::image type="content" source="./media/mongodb-mongoose/db-level-throughput.png" alt-text="Tutorial de Node.js: captura de pantalla de Azure Portal en la que se muestra cómo crear una base de datos en el Explorador de datos de una cuenta de Azure Cosmos DB, para usarla con el módulo de nodo de Mongoose":::

- **Almacenar todos los modelos de objetos en una sola colección de Cosmos DB**: Si prefiere almacenar todos los modelos en una sola colección, solo tiene que crear una base de datos sin seleccionar la opción Aprovisionar rendimiento. El uso de este modelo de capacidad creará una colección con su propia capacidad de rendimiento para cada modelo de objetos.

Después de crear la base de datos, deberá usar el nombre en la variable de entorno `COSMOSDB_DBNAME` a continuación.

## <a name="set-up-your-nodejs-application"></a>Configurar la aplicación de Node.js

>[!Note]
> Si quiere usar el código de ejemplo en lugar de configurar la propia aplicación, clone el [ejemplo](https://github.com/Azure-Samples/Mongoose_CosmosDB) que se usa en este tutorial y compile su propia aplicación Node.js de Mongoose en Azure Cosmos DB.

1. Para crear una aplicación Node.js en una carpeta de su elección, ejecute el comando siguiente en un símbolo del sistema de nodo.

    ```npm init```

    Responda a las preguntas y su proyecto estará listo.

2. Agregue un nuevo archivo a la carpeta y asígnele el nombre ```index.js```.
3. Instale los paquetes necesarios mediante una de las opciones ```npm install```:
   * Mongoose: ```npm install mongoose@5 --save```

     > [!Note]
     > La siguiente conexión de ejemplo de Mongoose se basa en Mongoose 5 +, que ha cambiado desde hace varias versiones.
    
   * Dotenv (si quiere cargar los secretos de un archivo .env): ```npm install dotenv --save```

     >[!Note]
     > La marca ```--save``` agrega la dependencia al archivo package.json.

4. Importe las dependencias en el archivo index.js.

    ```JavaScript
   var mongoose = require('mongoose');
   var env = require('dotenv').config();   //Use the .env file to load the variables
    ```

5. Agregue la cadena de conexión y el nombre de Cosmos DB al archivo ```.env```. Reemplace los marcadores de posición {cosmos-account-name} y {dbname} por el nombre de la cuenta y de la base de datos de Cosmos, sin los símbolos de llave.

    ```JavaScript
   # You can get the following connection details from the Azure portal. You can find the details on the Connection string pane of your Azure Cosmos account.

   COSMODDB_USER = "<Azure Cosmos account's user name, usually the database account name>"
   COSMOSDB_PASSWORD = "<Azure Cosmos account password, this is one of the keys specified in your account>"
   COSMOSDB_DBNAME = "<Azure Cosmos database name>"
   COSMOSDB_HOST= "<Azure Cosmos Host name>"
   COSMOSDB_PORT=10255
    ```

6. Conéctese a Cosmos DB mediante el marco Mongoose; para ello, debe agregar el siguiente código al final del archivo index.js.
    ```JavaScript
   mongoose.connect("mongodb://"+process.env.COSMOSDB_HOST+":"+process.env.COSMOSDB_PORT+"/"+process.env.COSMOSDB_DBNAME+"?ssl=true&replicaSet=globaldb", {
      auth: {
        user: process.env.COSMODDB_USER,
        password: process.env.COSMOSDB_PASSWORD
      },
    useNewUrlParser: true,
    useUnifiedTopology: true,
    retryWrites: false
    })
    .then(() => console.log('Connection to CosmosDB successful'))
    .catch((err) => console.error(err));
    ```
    >[!Note]
    > En este caso, las variables de entorno se cargan mediante process.env.{variableName} con el paquete de npm "dotenv".

    Una vez que esté conectado a Azure Cosmos DB, puede empezar a configurar los modelos de objetos en Mongoose.

## <a name="best-practices-for-using-mongoose-with-cosmos-db"></a>Procedimientos recomendados para usar Mongoose con Cosmos DB

Por cada modelo que se crea, Mongoose crea por otro lado una nueva colección. Esta situación se aborda mejor con la [opción Rendimiento en el nivel de base de datos](set-throughput.md#set-throughput-on-a-database), que se analizó anteriormente. Para usar una sola colección, debe utilizar los [discriminadores](https://mongoosejs.com/docs/discriminators.html) de Mongoose. Los discriminadores son un mecanismo de herencia de esquemas. Le permiten tener varios modelos con esquemas superpuestos en la misma colección subyacente de MongoDB.

Puede almacenar los distintos modelos de datos en la misma colección y, a continuación, usar una cláusula de filtro a la hora de realizar la consulta para así descargar solo los datos necesarios. Vamos a ver en detalle cada uno de estos modelos.

### <a name="one-collection-per-object-model"></a>Una colección por modelo de objetos

En esta sección le indicaremos cómo puede lograrlo gracias a la API de Azure Cosmos DB para MongoDB. Este método es la estrategia recomendada, ya que le permite controlar el costo y la capacidad. Como resultado, la cantidad de unidades de solicitud en la base de datos no depende del número de modelos de objetos. Este es el modelo de funcionamiento predeterminado de Mongoose, por lo tanto es posible que ya esté familiarizado con él.

1. Abra el archivo ```index.js``` de nuevo.

1. Cree la definición de esquema para "Family".

    ```JavaScript
    const Family = mongoose.model('Family', new mongoose.Schema({
        lastName: String,
        parents: [{
            familyName: String,
            firstName: String,
            gender: String
        }],
        children: [{
            familyName: String,
            firstName: String,
            gender: String,
            grade: Number
        }],
        pets:[{
            givenName: String
        }],
        address: {
            country: String,
            state: String,
            city: String
        }
    }));
    ```

1. Cree un objeto para "Family".

    ```JavaScript
    const family = new Family({
        lastName: "Volum",
        parents: [
            { firstName: "Thomas" },
            { firstName: "Mary Kay" }
        ],
        children: [
            { firstName: "Ryan", gender: "male", grade: 8 },
            { firstName: "Patrick", gender: "male", grade: 7 }
        ],
        pets: [
            { givenName: "Buddy" }
        ],
        address: { country: "USA", state: "WA", city: "Seattle" }
    });
    ```

1. Por último, guarde el objeto en Cosmos DB. Esta opción crea una colección en segundo plano.

    ```JavaScript
    family.save((err, saveFamily) => {
        console.log(JSON.stringify(saveFamily));
    });
    ```

1. A continuación, cree otro esquema y objeto. Esta vez vamos a crear uno para "Vacation Destinations" que pueden ser destinos de interés para las familias.
   1. Igual que la última vez, cree el esquema.
      ```JavaScript
      const VacationDestinations = mongoose.model('VacationDestinations', new mongoose.Schema({
       name: String,
       country: String
      }));
      ```

   1. Cree un objeto de ejemplo (puede agregar varios objetos a este esquema) y guárdelo.
      ```JavaScript
      const vacaySpot = new VacationDestinations({
       name: "Honolulu",
       country: "USA"
      });

      vacaySpot.save((err, saveVacay) => {
       console.log(JSON.stringify(saveVacay));
      });
      ```

1. A continuación, vaya a Azure Portal y verá dos colecciones que se crearon en Cosmos DB.

   :::image type="content" source="./media/mongodb-mongoose/mongo-mutliple-collections.png" alt-text="Tutorial de Node.js: captura de pantalla de Azure Portal, donde se muestra una cuenta de Azure Cosmos DB con varios nombres de colección resaltados: base de datos de Node":::

1. Por último, lea los datos de Cosmos DB. Puesto que vamos a usar el modelo de funcionamiento predeterminado de Mongoose, las lecturas son las mismas que cualquier otra en Mongoose.

    ```JavaScript
    Family.find({ 'children.gender' : "male"}, function(err, foundFamily){
        foundFamily.forEach(fam => console.log("Found Family: " + JSON.stringify(fam)));
    });
    ```

### <a name="using-mongoose-discriminators-to-store-data-in-a-single-collection"></a>Usar discriminadores de Mongoose para almacenar datos en una sola colección

En este método se usan los [discriminadores de Mongoose](https://mongoosejs.com/docs/discriminators.html) para optimizar los costos de cada colección. Los discriminadores le permiten definir una "Clave" diferenciadora que a su vez le permite almacenar, diferenciar y filtrar los diferentes modelos de objetos.

En este caso, crearemos un modelo de objeto base, definiremos una clave diferenciadora y agregaremos "Family" y "VacationDestinations" a modo de extensión para el modelo base.

1. Realice la configuración base y defina la clave del discriminador.

    ```JavaScript
    const baseConfig = {
        discriminatorKey: "_type", //If you've got a lot of different data types, you could also consider setting up a secondary index here.
        collection: "alldata"   //Name of the Common Collection
    };
    ```

1. A continuación, defina el modelo de objetos comunes.

    ```JavaScript
    const commonModel = mongoose.model('Common', new mongoose.Schema({}, baseConfig));
    ```

1. Defina el modelo "Family". Tenga en cuenta que estamos usando ```commonModel.discriminator``` en lugar de ```mongoose.model```. Además, también agregaremos la configuración base al esquema de Mongoose. Por lo tanto, en este caso, la clave discriminatorKey es ```FamilyType```.

    ```JavaScript
    const Family_common = commonModel.discriminator('FamilyType', new     mongoose.Schema({
        lastName: String,
        parents: [{
            familyName: String,
            firstName: String,
            gender: String
        }],
        children: [{
            familyName: String,
            firstName: String,
           gender: String,
            grade: Number
        }],
        pets:[{
            givenName: String
        }],
        address: {
            country: String,
            state: String,
            city: String
        }
    }, baseConfig));
    ```

1. De forma similar, agregaremos otro esquema; esta vez para "VacationDestinations". En este caso, la clave DiscriminatorKey es ```VacationDestinationsType```.

    ```JavaScript
    const Vacation_common = commonModel.discriminator('VacationDestinationsType', new mongoose.Schema({
        name: String,
        country: String
    }, baseConfig));
    ```

1. Por último, crearemos objetos para el modelo y lo guardaremos.
   1. Agreguemos objetos al modelo "Family".
      ```JavaScript
      const family_common = new Family_common({
       lastName: "Volum",
       parents: [
           { firstName: "Thomas" },
           { firstName: "Mary Kay" }
       ],
       children: [
           { firstName: "Ryan", gender: "male", grade: 8 },
           { firstName: "Patrick", gender: "male", grade: 7 }
       ],
       pets: [
           { givenName: "Buddy" }
       ],
       address: { country: "USA", state: "WA", city: "Seattle" }
      });

      family_common.save((err, saveFamily) => {
       console.log("Saved: " + JSON.stringify(saveFamily));
      });
      ```

   1. A continuación, vamos a agregar objetos al modelo "VacationDestinations" y lo guardaremos.
      ```JavaScript
      const vacay_common = new Vacation_common({
       name: "Honolulu",
       country: "USA"
      });

      vacay_common.save((err, saveVacay) => {
       console.log("Saved: " + JSON.stringify(saveVacay));
      });
      ```

1. Si vuelve a Azure Portal, verá que solo tiene una colección denominada ```alldata``` con los datos de "Family" y "VacationDestinations".

   :::image type="content" source="./media/mongodb-mongoose/mongo-collections-alldata.png" alt-text="Tutorial de Node.js: captura de pantalla de Azure Portal, que muestra una cuenta de Azure Cosmos DB con el nombre de la colección resaltado: base de datos de Node":::

1. Además, tenga en cuenta que cada objeto tiene otro atributo denominado "```__type```", que le ayudará a diferenciar entre los dos modelos de objetos diferentes.

1. Por último, lea los datos que se guardan en Azure Cosmos DB. Mongoose se encarga de filtrar los datos basados en el modelo. Por lo tanto, no tiene que hacer nada al leer los datos. Simplemente especifique el modelo (en este caso, ```Family_common```) y Mongoose se encargará de los filtros en "DiscriminatorKey".

    ```JavaScript
    Family_common.find({ 'children.gender' : "male"}, function(err, foundFamily){
        foundFamily.forEach(fam => console.log("Found Family (using discriminator): " + JSON.stringify(fam)));
    });
    ```

Como puede ver, es muy fácil trabajar con discriminadores de Mongoose. Así pues, si tiene una aplicación que usa el marco Mongoose, este tutorial es una manera de desarrollar la aplicación y ejecutarla mediante la API de Azure Cosmos DB para MongoDB sin tener que hacer demasiados cambios.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [usar Studio 3T](mongodb-mongochef.md) con la API de Azure Cosmos DB para MongoDB.
- Aprenda a [usar Robo 3T](mongodb-robomongo.md) con la API de Azure Cosmos DB para MongoDB.
- Explore [ejemplos](mongodb-samples.md) de MongoDB con la API de Azure Cosmos DB para MongoDB.

[dbleveltp]: ./media/mongodb-mongoose/db-level-throughput.png
