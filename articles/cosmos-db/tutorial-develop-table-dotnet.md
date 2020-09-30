---
title: Table API de Azure Cosmos DB con el SDK de .NET Standard
description: Obtenga información acerca de cómo almacenar y consultar los datos estructurados en una cuenta de Table API de Azure Cosmos DB.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 12/03/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 56198392f3c769837d8d672b861baa9b341d284e
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89419357"
---
# <a name="get-started-with-azure-cosmos-db-table-api-and-azure-table-storage-using-the-net-sdk"></a>Introducción a Table API de Azure Cosmos DB y Azure Table Storage mediante el SDK de .NET

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]

[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Table API de Azure Cosmos DB o Azure Table Storage se pueden usar para almacenar datos NoSQL estructurados en la nube y proporcionar un almacén de claves y atributos con un diseño sin esquema. Dado que tanto Table API de Azure Cosmos DB como Table Storage carecen de esquema, es fácil adaptar los datos a medida que evolucionan las necesidades de la aplicación. Tanto Table API de Azure Cosmos DB como Table Storage se pueden usar para almacenar conjuntos de datos flexibles, como datos de usuarios para aplicaciones web, libretas de direcciones, información de dispositivos u otros tipos de metadatos que el servicio requiera. 

Este tutorial describe un ejemplo que muestra cómo usar [Microsoft Azure Cosmos DB Table Library for .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) con escenarios de almacenamiento de Table API de Azure Cosmos DB y Azure Table Storage. Debe usar la conexión específica del servicio de Azure. Estos escenarios se describen mediante ejemplos de C# que ilustran cómo crear tablas, insertar y actualizar datos, consultar los datos y eliminar las tablas.

## <a name="prerequisites"></a>Requisitos previos

Necesitará lo siguiente para completar este ejemplo correctamente:

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)

* [Microsoft Azure Cosmos DB Table Library for .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table): esta biblioteca está disponible actualmente para .NET Standard y .NET Framework. 

* [Cuenta de Table API de Azure Cosmos DB](create-table-dotnet.md#create-a-database-account).

## <a name="create-an-azure-cosmos-db-table-api-account"></a>Creación de una cuenta de Table API de Azure Cosmos DB

[!INCLUDE [cosmos-db-create-dbaccount-table](../../includes/cosmos-db-create-dbaccount-table.md)]

## <a name="create-a-net-console-project"></a>Creación de un proyecto de consola de .NET

En Visual Studio, cree una nueva aplicación de consola de .NET. Los siguientes pasos muestran cómo crear una aplicación de consola en Visual Studio 2019. Azure Cosmos DB Table Library se puede usar en cualquier tipo de aplicación. NET, incluidos cualquier servicio en la nube o aplicación web de Azure, y aplicaciones de escritorio o móviles. En esta guía, usamos una aplicación de consola para hacerlo más sencillo.

1. Seleccione **File (Archivo)**  > **New (Nuevo)**  > **Project (Proyecto)** .

1. Elija **Aplicación de consola (.NET Core)** y, a continuación, seleccione **Siguiente**.

1. En el campo **Nombre del proyecto**, escriba un nombre para la aplicación, como **CosmosTableSamples**. (Puede proporcionar un nombre diferente para adaptarse a sus necesidades).

1. Seleccione **Crear**.

Todos los ejemplos de código de este ejemplo se pueden agregar al método Main() del archivo **Program.cs** de la aplicación de consola.

## <a name="install-the-required-nuget-package"></a>Instalación del paquete NuGet requerido

Para obtener el paquete NuGet, siga estos pasos:

1. Haga clic con el botón derecho en el proyecto, en el **Explorador de soluciones**, y elija **Administrar paquetes NuGet**.

1. Busque en línea [`Microsoft.Azure.Cosmos.Table`](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), [`Microsoft.Extensions.Configuration`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration), [`Microsoft.Extensions.Configuration.Json`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json) y [`Microsoft.Extensions.Configuration.Binder`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Binder) y seleccione **Instalar** para instalar Microsoft Azure Cosmos DB Table Library.

## <a name="configure-your-storage-connection-string"></a>Configuración de la cadena de conexión de almacenamiento.

1. Desde [Azure Portal](https://portal.azure.com/), vaya a su cuenta de Azure Cosmos o la cuenta de Table Storage. 

1. Abra el panel **Cadena de conexión** o **Claves de acceso**. Use el botón de copia en el lado derecho de la ventana para copiar la **CADENA DE CONEXIÓN PRINCIPAL**.

   :::image type="content" source="./media/create-table-dotnet/connection-string.png" alt-text="Visualización y copia de la cadena de conexión principal (PRIMARY CONNECTION STRING) en el panel Cadena de conexión":::
   
1. Para configurar la cadena de conexión, desde Visual Studio haga clic con el botón derecho en el proyecto **CosmosTableSamples**.

1. Seleccione **Agregar** y, a continuación, **Nuevo elemento**. Cree un nuevo archivo **Settings.json** con el tipo de archivo como **Configuración de JSON de TypeScript**. 

1. Reemplace el código en el archivo Settings.json por el código siguiente y asigne la cadena de conexión principal:

   ```csharp
   {
   "StorageConnectionString": <Primary connection string of your Azure Cosmos DB account>
   }
   ```

1. Haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nuevo elemento** y agregue una clase denominada **AppSettings.cs**.

1. Agregue el siguiente código al archivo AppSettings.cs. Este archivo lee la cadena de conexión del archivo Settings.json y la asigna al parámetro de configuración:

  :::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/AppSettings.cs":::

## <a name="parse-and-validate-the-connection-details"></a>Análisis y validación de los detalles de conexión

1. Haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nuevo elemento** y agregue una clase denominada **Common.cs**. Escribirá código para validar los detalles de conexión y crear una tabla dentro de esta clase.

1. Defina un método `CreateStorageAccountFromConnectionString` tal como se muestra a continuación. Este método analizará los detalles de la cadena de conexión y comprobará que el nombre de la cuenta y los detalles de la clave de cuenta proporcionados en el archivo "Settings.json" son válidos.

   :::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Common.cs" id="createStorageAccount":::

## <a name="create-a-table"></a>Creación de una tabla 

La clase [CloudTableClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.cloudtableclient) le permite recuperar las tablas y las entidades almacenadas en Almacenamiento de tablas. Dado que no hay ninguna tabla en la cuenta de Table API de Cosmos DB, vamos a agregar el método `CreateTableAsync` a la clase **Common.cs** para crear una tabla:

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Common.cs" id="CreateTable":::

Si recibe una "excepción 503 de servicio no disponible", es posible que los puertos necesarios para el modo de conexión estén bloqueados por un firewall. Para corregir este problema, abra los puertos necesarios o use la conectividad del modo de puerta de enlace como se indica en el código siguiente:

```csharp
tableClient.TableClientConfiguration.UseRestExecutorForCosmosEndpoint = true;
```

## <a name="define-the-entity"></a>Definición de la entidad 

Las entidades se asignan a objetos C# utilizando una clase personalizada derivada de [TableEntity](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.tableentity). Para agregar una entidad a una tabla, cree una clase que defina las propiedades de la entidad.

Haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nueva carpeta** y asígnele el nombre **Model**. En la carpeta Model, agregue una clase llamada **CustomerEntity.cs** y agréguele el código siguiente.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Model/CustomerEntity.cs":::

El código define una clase de entidad que utiliza el nombre de pila del cliente como clave de fila y el apellido como clave de partición. En conjunto, las claves de partición y de fila de una entidad la identifican inequívocamente en la tabla. Las entidades con la misma clave de partición pueden consultarse más rápidamente que aquellas con diferentes claves de partición, pero el uso de claves de partición diversas permite una mayor escalabilidad de las operaciones paralelas. Las entidades que se almacenan en tablas deben ser de un tipo compatible, por ejemplo, derivadas de la clase [TableEntity](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.tableentity). Las propiedades de entidad que desee almacenar en una tabla deben ser propiedades públicas del tipo, y deben admitir que se obtengan y establezcan valores. Además, el tipo de entidad debe exponer un constructor sin parámetros.

## <a name="insert-or-merge-an-entity"></a>Inserción o combinación de la entidad

En el ejemplo de código siguiente se crea un objeto de entidad y se agrega a la tabla. El método InsertOrMerge dentro de la clase [TableOperation](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.tableoperation) se utiliza para insertar o combinar una entidad. Se llama al método [CloudTable.ExecuteAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.cloudtable.executeasync?view=azure-dotnet) para ejecutar la operación. 

Haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nuevo elemento** y agregue una clase denominada **SamplesUtils.cs**. Esta clase almacena todo el código necesario para realizar operaciones CRUD en las entidades. 

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="InsertItem":::

## <a name="get-an-entity-from-a-partition"></a>Obtención de una entidad de una partición

Puede obtener la entidad de una partición mediante el método Retrieve de la clase [TableOperation](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.tableoperation). En el ejemplo de código siguiente se obtiene la clave de fila de la clave de partición, el correo electrónico y el número de teléfono de una entidad de cliente. Este ejemplo también imprime las unidades de solicitud utilizadas para consultar la entidad. Para consultar una entidad, anexe el código siguiente al archivo **SamplesUtils.cs**:

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="QueryData":::

## <a name="delete-an-entity"></a>Eliminación de una entidad

Puede eliminar fácilmente una entidad una vez recuperada utilizando el mismo patrón mostrado para actualizar una entidad. El código siguiente recupera y elimina una entidad de cliente. Para eliminar una entidad, anexe el código siguiente al archivo **SamplesUtils.cs**: 

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="QueryData":::

## <a name="execute-the-crud-operations-on-sample-data"></a>Ejecución de operaciones CRUD en datos de ejemplo

Después de definir los métodos para crear una tabla e insertar o combinar entidades, ejecute estos métodos en los datos de ejemplo. Para ello, haga clic con el botón derecho en el proyecto **CosmosTableSamples**. Seleccione **Agregar**, **Nuevo elemento**, agregue una clase denominada **BasicSamples.cs** y agréguele el código siguiente. Este código crea una tabla y le agrega las entidades.

Si no desea eliminar la entidad y la tabla al final del proyecto, comente los métodos `await table.DeleteIfExistsAsync()` y `SamplesUtils.DeleteEntityAsync(table, customer)` del código siguiente. Es mejor comentar estos métodos y validar los datos antes de eliminar la tabla.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/BasicSamples.cs":::

El código anterior crea una tabla que empieza por "demo" y el GUID generado se anexa al nombre de la tabla. A continuación, agrega una entidad de cliente con nombre y apellido como "Walter Harp" y más tarde actualiza el número de teléfono de este usuario. 

En este tutorial, ha compilado código para realizar operaciones CRUD básicas en los datos almacenados en la cuenta de Table API. También puede realizar operaciones avanzadas, como insertar datos por lotes, consultar todos los datos de una partición, consultar un intervalo de datos de una partición y enumerar tablas en la cuenta cuyos nombres comiencen por el prefijo especificado. Puede descargar el ejemplo completo del repositorio de GitHub [azure-cosmos-table-dotnet-core-getting-started](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started). La clase [AdvancedSamples.cs](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started/blob/master/CosmosTableSamples/AdvancedSamples.cs) tiene más operaciones que puede realizar en los datos.  

## <a name="run-the-project"></a>Ejecución del proyecto

En el proyecto **CosmosTableSamples**. Abra la clase denominada **Program.CS** y agréguele el siguiente código para llamar a BasicSamples cuando se ejecute el proyecto.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Program.cs":::

Ahora, compile la solución y presione F5 para ejecutar el proyecto. Cuando se ejecute el proyecto, verá la siguiente salida en el símbolo del sistema:

:::image type="content" source="./media/tutorial-develop-table-standard/output-from-sample.png" alt-text="Salida del símbolo del sistema":::

Si recibe un error que indica que no se encuentra el archivo Settings.json al ejecutar el proyecto, puede resolverlo agregando la siguiente entrada XML a la configuración del proyecto. Haga clic con el botón derecho en CosmosTableSamples, seleccione Editar CosmosTableSamples.csproj y agregue el itemGroup siguiente: 

```csharp
  <ItemGroup>
    <None Update="Settings.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
```
Ahora puede iniciar sesión en Azure Portal y comprobar que los datos existen en la tabla. 

:::image type="content" source="./media/tutorial-develop-table-standard/results-in-portal.png" alt-text="Resultados en el portal":::

## <a name="next-steps"></a>Pasos siguientes

Ahora puede pasar al siguiente tutorial y aprender a migrar datos a la cuenta de Table API de Azure Cosmos DB. 

> [!div class="nextstepaction"]
>[Consulta de datos](../cosmos-db/table-import.md)
