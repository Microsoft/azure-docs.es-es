---
title: Administración de recursos de SQL API de Azure Cosmos DB mediante el SDK de .Net V4
description: Inicio rápido sobre cómo compilar una aplicación de consola mediante el SDK de .NET V4 para administrar recursos de la cuenta de SQL API de Azure Cosmos DB.
author: anfeldma-ms
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 05/11/2020
ms.custom: devx-track-dotnet
ms.openlocfilehash: 6fec96d2fb70c0ccf6dc66690d5a61c76038fd74
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011130"
---
# <a name="quickstart-build-a-console-app-using-the-net-v4-sdk-to-manage-azure-cosmos-db-sql-api-account-resources"></a>Inicio rápido: Compilación de una aplicación de consola mediante el SDK de .NET V4 para administrar recursos de la cuenta de SQL API de Azure Cosmos DB.

> [!div class="op_single_selector"]
> * [.NET V3](create-sql-api-dotnet.md)
> * [.NET V4](create-sql-api-dotnet-V4.md)
> * [SDK para Java v4](create-sql-api-java.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Introducción a la biblioteca cliente de API de SQL de Azure Cosmos DB para .NET. Siga los pasos de este documento para instalar el paquete .NET V4 (Azure.Cosmos), compilar una aplicación y probar el código de ejemplo para las operaciones CRUD básicas en los datos almacenados en Azure Cosmos DB. 

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede usar Azure Cosmos DB para crear y consultar rápidamente las bases de datos de gráficos, documentos, claves y valores. Use la biblioteca cliente de API de SQL de Azure Cosmos DB para .NET para:

* Crear una base de datos y un contenedor de Azure Cosmos
* Agregar datos de ejemplo al contenedor
* Consultar los datos 
* Eliminación de la base de datos

[Código fuente de la biblioteca](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/v4) | [Paquete (NuGet)](https://www.nuget.org/packages/Azure.Cosmos)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción de Azure: [cree una de forma gratuita](https://azure.microsoft.com/free/) o [pruebe Azure Cosmos DB de forma gratuita](https://azure.microsoft.com/try/cosmosdb/) sin necesidad de tener una suscripción de Azure, sin cargos ni compromiso. 
* [SDK de .NET Core 3](https://dotnet.microsoft.com/download/dotnet-core). Puede comprobar qué versión está disponible en su entorno mediante la ejecución de `dotnet --version`.

## <a name="setting-up"></a>Instalación

En esta sección se le guía por el proceso de creación de una cuenta de Azure Cosmos y de configuración de un proyecto que usa una biblioteca cliente de API de SQL de Azure Cosmos DB para .NET para administrar recursos. El código de ejemplo descrito en este artículo crea una base de datos de `FamilyDatabase` y miembros de la familia (cada miembro de la familia es un elemento) en la base de datos. Cada miembro de la familia tiene propiedades como `Id, FamilyName, FirstName, LastName, Parents, Children, Address,`. La propiedad `LastName` se usa como clave de partición para el contenedor. 

### <a name="create-an-azure-cosmos-account"></a><a id="create-account"></a>Crear una cuenta de Azure Cosmos

Si usa la opción [Prueba gratuita del emulador de Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) para crear una cuenta de Azure Cosmos, debe crear una cuenta de Azure Cosmos DB del tipo de **SQL API**. Ya se ha creado una cuenta de prueba de Azure Cosmos DB. No tiene que crear la cuenta explícitamente, por lo que puede omitir esta sección y pasar a la siguiente.

Si tiene su propia suscripción de Azure o creó una suscripción gratuita, debe crear una cuenta de Azure Cosmos explícitamente. El código siguiente creará una cuenta de Azure Cosmos con coherencia de sesión. La cuenta se replica en `South Central US` y `North Central US`.  

Puede usar Azure Cloud Shell para crear la cuenta de Azure Cosmos. Azure Cloud Shell es un shell interactivo, autenticado y al que se puede acceder desde un explorador para administrar recursos de Azure. Ofrece la flexibilidad de poder elegir la experiencia de shell que mejor se adapte a la forma de trabajar de cada uno, Bash o PowerShell. En esta guía de inicio rápido, elija el modo **Bash**. Azure Cloud Shell también requiere una cuenta de almacenamiento; puede crear una cuando se le solicite.

Seleccione el botón **Pruébelo** junto al siguiente código, elija el modo **Bash**, seleccione **Crear una cuenta de almacenamiento** e inicie sesión en Cloud Shell. Después, copie y pegue el código siguiente en Azure Cloud Shell y ejecútelo. El nombre de la cuenta de Azure Cosmos debe ser único globalmente; asegúrese de actualizar el valor de `mysqlapicosmosdb` antes de ejecutar el comando.

```azurecli-interactive

# Set variables for the new SQL API account, database, and container
resourceGroupName='myResourceGroup'
location='southcentralus'

# The Azure Cosmos account name must be globally unique, make sure to update the `mysqlapicosmosdb` value before you run the command
accountName='mysqlapicosmosdb'

# Create a resource group
az group create \
    --name $resourceGroupName \
    --location $location

# Create a SQL API Cosmos DB account with session consistency and multi-master enabled
az cosmosdb create \
    --resource-group $resourceGroupName \
    --name $accountName \
    --kind GlobalDocumentDB \
    --locations regionName="South Central US" failoverPriority=0 --locations regionName="North Central US" failoverPriority=1 \
    --default-consistency-level "Session" \
    --enable-multiple-write-locations true

```

La creación de la cuenta de Azure Cosmos tarda un rato;, una vez que la operación se realiza correctamente, puede ver la salida de confirmación. Una vez que el comando se complete correctamente, inicie sesión en [Azure Portal](https://portal.azure.com/) y compruebe que existe la cuenta de Azure Cosmos con el nombre especificado. Puede cerrar la ventana de Azure Cloud Shell una vez creado el recurso. 

### <a name="create-a-new-net-app"></a><a id="create-dotnet-core-app"></a>Crear una nueva aplicación .NET

Cree una nueva aplicación .NET en el IDE o editor que prefiera. Abra el símbolo del sistema de Windows o una ventana de Terminal desde el equipo local. Ejecutará todos los comandos de las secciones siguientes desde el símbolo del sistema o desde el terminal.  Ejecute el comando dotnet new para crear una nueva aplicación llamada `todo`. El parámetro --langVersion establece la propiedad LangVersion en el archivo de proyecto creado.

   ```bash
   dotnet new console –langVersion:8 -n todo
   ```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

   ```bash
   cd todo
   dotnet build
   ```

El resultado esperado de la compilación debe parecerse a lo siguiente:

```bash
  Restore completed in 100.37 ms for C:\Users\user1\Downloads\CosmosDB_Samples\todo\todo.csproj.
  todo -> C:\Users\user1\Downloads\CosmosDB_Samples\todo\bin\Debug\netcoreapp3.0\todo.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:34.17
```

### <a name="install-the-azure-cosmos-db-package"></a><a id="install-package"></a>Instalar el paquete de Azure Cosmos DB

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de Azure Cosmos DB para .NET Core con el comando dotnet add package.

   ```bash
   dotnet add package Azure.Cosmos --version 4.0.0-preview3
   ```

### <a name="copy-your-azure-cosmos-account-credentials-from-the-azure-portal"></a>Copiar las credenciales de la cuenta de Azure Cosmos desde Azure Portal

La aplicación de ejemplo debe autenticarse para la cuenta de Azure Cosmos. Para realizar la autenticación, debe pasar las credenciales de cuenta de Azure Cosmos a la aplicación. Para obtener las credenciales de cuenta de Azure Cosmos, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Vaya a la cuenta de Azure Cosmos.

1. Abra el panel **Claves** y copia el **URI** y la **CLAVE PRINCIPAL** de la cuenta. En el paso siguiente, agregará los valores de URI y claves a una variable de entorno.

## <a name="object-model"></a><a id="object-model"></a>Modelo de objetos

Antes de empezar a compilar la aplicación, consulte la jerarquía de recursos de Azure Cosmos DB y el modelo de objetos usado para crear estos recursos y obtener acceso a ellos. Azure Cosmos DB crea recursos en el orden siguiente:

* Cuenta de Azure Cosmos 
* Bases de datos 
* Contenedores 
* Elementos

Para obtener más información acerca de la jerarquía de diferentes entidades, consulte el artículo sobre cómo [trabajar con bases de datos, contenedores y elementos en Azure Cosmos DB](databases-containers-items.md). Usará las siguientes clases de .NET para interactuar con estos recursos:

* CosmosClient: esta clase proporciona una representación lógica del lado cliente para el servicio Azure Cosmos DB. El objeto de cliente se usa para configurar y ejecutar solicitudes en el servicio.
* CreateDatabaseIfNotExistsAsync: este método crea (si no existe) u obtiene (si ya existe) un recurso de base de datos como operación asincrónica. 
* CreateContainerIfNotExistsAsync: este método crea (si no existe) u obtiene (si ya existe) un contenedor como operación asincrónica. Puede comprobar el código de estado de la respuesta para determinar si el contenedor se creó recientemente (201) o si se devolvió un contenedor existente (200). 
* CreateItemAsync: este método crea un elemento en el contenedor.
* UpsertItemAsync: este método crea un elemento en el contenedor si aún no existe o lo reemplaza si ya existe. 
* GetItemQueryIterator: este método crea una consulta para los elementos de un contenedor en una base de datos de Azure Cosmos mediante una instrucción SQL con valores con parámetros. 
* DeleteAsync: elimina la base de datos especificada de la cuenta de Azure Cosmos. El método `DeleteAsync` solo elimina la base de datos.

 ## <a name="code-examples"></a><a id="code-examples"></a>Ejemplos de código

En el código de ejemplo que se describe en este artículo, se crea una base de datos de familia en Azure Cosmos DB. La base de datos de la familia contiene detalles de la familia como el nombre, la dirección, la ubicación, los padres, los hijos y las mascotas. Antes de rellenar los datos en su cuenta de Azure Cosmos, defina las propiedades de un elemento de familia. Cree una nueva clase denominada `Family.cs` en el nivel raíz de la aplicación de ejemplo y agréguele el código siguiente:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Family.cs)]

### <a name="add-the-using-directives--define-the-client-object"></a>Agregar las directivas de uso y definir el objeto de cliente

Desde el directorio del proyecto, abra el archivo `Program.cs` en el editor y agregue lo siguiente con directivas en la parte superior de la aplicación:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=Usings)]


En la clase `Program`, agregue las variables globales siguientes. Estas incluirán el punto de conexión y las claves de autorización, el nombre de la base de datos y el contenedor que creará. Asegúrese de reemplazar los valores de punto de conexión y de clave de autorización según su entorno. 

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=Constants)]

Finalmente, reemplace el método `Main`:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=Main)]

### <a name="create-a-database"></a>Crear una base de datos 

Defina el método `CreateDatabaseAsync` dentro de la clase `program.cs`. Este método crea `FamilyDatabase` si todavía no existe.

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=CreateDatabaseAsync)]

### <a name="create-a-container"></a>Crear un contenedor

Defina el método `CreateContainerAsync` dentro de la clase `Program`. Este método crea `FamilyContainer` si todavía no existe. 

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=CreateContainerAsync)]

### <a name="create-an-item"></a>Crear un elemento

Para crear un elemento de familia, agregue el método `AddItemsToContainerAsync` con el código siguiente. Puede usar los métodos `CreateItemAsync` o `UpsertItemAsync` para crear un elemento:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=AddItemsToContainerAsync)]

### <a name="query-the-items"></a>Consultar elementos

Después de insertar un elemento, puede ejecutar una consulta para obtener los detalles de la familia "Andersen". En el código siguiente se muestra cómo ejecutar la consulta directamente con la consulta SQL. La consulta SQL para obtener los detalles de la familia "Anderson" es: `SELECT * FROM c WHERE c.LastName = 'Andersen'`. Defina el método `QueryItemsAsync` en la clase `Program` y agréguele el código siguiente:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=QueryItemsAsync)]

### <a name="replace-an-item"></a>Reemplazo de un elemento 

Lea un elemento de familia y, a continuación, actualícelo mediante la adición del método `ReplaceFamilyItemAsync` con el código siguiente.

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=ReplaceFamilyItemAsync)]

### <a name="delete-an-item"></a>Eliminación de un elemento 

Para eliminar un elemento de familia, agregue el método `DeleteFamilyItemAsync` con el código siguiente.

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=DeleteFamilyItemAsync)]

### <a name="delete-the-database"></a>Eliminación de la base de datos 

Por último, para eliminar la base de datos, agregue el método `DeleteDatabaseAndCleanupAsync` con el código siguiente:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=DeleteDatabaseAndCleanupAsync)]

Después de agregar todos los métodos necesarios, guarde el archivo `Program`. 

## <a name="run-the-code"></a>Ejecución del código

A continuación, compile y ejecute la aplicación para crear los recursos de Azure Cosmos DB.

   ```bash
   dotnet run
   ```

Al ejecutar la aplicación, se genera el siguiente resultado. También puede iniciar sesión en Azure Portal y validar la creación de los recursos:

   ```bash
   Created Database: FamilyDatabase
   
   Created Container: FamilyContainer
   
   Created item in database with id: Andersen.1
   
   Running query: SELECT * FROM c WHERE c.LastName = 'Andersen'
   
           Read {"id":"Andersen.1","LastName":"Andersen","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null   "FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets": [{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":false}
   
   Updated Family [Wakefield,Wakefield.7].
           Body is now: {"id":"Wakefield.7","LastName":"Wakefield","Parents":[{"FamilyName":"Wakefield","FirstName":"Robin"}   {"FamilyName":"Miller","FirstName":"Ben"}],"Children":[{"FamilyName":"Merriam","FirstName":"Jesse","Gender":"female","Grade":6   "Pets":[{"GivenName":"Goofy"},{"GivenName":"Shadow"}]},{"FamilyName":"Miller","FirstName":"Lisa","Gender":"female","Grade":1   "Pets":null}],"Address":{"State":"NY","County":"Manhattan","City":"NY"},"IsRegistered":true}
   
   Deleted Family [Wakefield,Wakefield.7]
   
   Deleted Database: FamilyDatabase
   
   End of demo, press any key to exit.
   ```

Para validar la creación de los datos, inicie sesión en Azure Portal y vea los elementos necesarios en su cuenta de Azure Cosmos. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el CLI de Azure o Azure PowerShell para quitar la cuenta de Azure Cosmos y el grupo de recursos correspondiente. El siguiente comando muestra cómo eliminar el grupo de recursos con el CLI de Azure:

```azurecli
az group delete -g "myResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una base de datos y un contenedor en Azure Portal, y agregado datos de ejemplo mediante el ejemplo de .NET. Ahora puede importar datos adicionales a su cuenta de Azure Cosmos con las instrucciones del siguiente artículo. 

> [!div class="nextstepaction"]
> [Importación de datos a Azure Cosmos DB](import-data.md)
