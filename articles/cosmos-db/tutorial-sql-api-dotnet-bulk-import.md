---
title: Importación masiva de datos a la cuenta de SQL API de Azure Cosmos DB mediante el SDK para .NET
description: Obtenga información sobre cómo importar o ingerir datos en Azure Cosmos DB mediante la creación de una aplicación de consola de .NET que optimice el rendimiento aprovisionado (RU/s) necesario para importar datos.
author: ealsur
ms.author: maquaran
ms.service: cosmos-db
ms.topic: tutorial
ms.date: 11/04/2019
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: b16402f809da18588b26995e7129d2f27575b48d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019256"
---
# <a name="bulk-import-data-to-azure-cosmos-db-sql-api-account-by-using-the-net-sdk"></a>Importación masiva de datos a la cuenta de SQL API de Azure Cosmos DB mediante el SDK para .NET

En este tutorial se muestra cómo compilar una aplicación de consola de .NET que optimice la capacidad de proceso aprovisionada (RU/s) necesaria para importar datos en Azure Cosmos DB. En este artículo se leen los datos de un origen de datos de ejemplo y se importan en un contenedor de Azure Cosmos.
Este tutorial usa la [versión 3.0+](https://www.nuget.org/packages/Microsoft.Azure.Cosmos) del SDK de .NET de Azure Cosmos DB, cuyo destino puede ser .NET Framework o .NET Core.

Esta tutorial abarca lo siguiente:

> [!div class="checklist"]
> * Creación de una cuenta de Azure Cosmos
> * Configuración del proyecto
> * Conexión a una cuenta de Azure Cosmos con compatibilidad para la ejecución en bloque habilitada
> * Realización de una importación de datos mediante operaciones simultáneas de creación

## <a name="prerequisites"></a>Prerrequisitos

Antes de seguir las instrucciones del presente artículo, asegúrese de tener los siguientes recursos:

* Una cuenta de Azure activa. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [SDK de .NET Core 3](https://dotnet.microsoft.com/download/dotnet-core). Puede comprobar qué versión está disponible en su entorno mediante la ejecución de `dotnet --version`.

## <a name="step-1-create-an-azure-cosmos-db-account"></a>Paso 1: Creación de una cuenta de Azure Cosmos DB

[Cree una cuenta de SQL API de Azure Cosmos DB](create-cosmosdb-resources-portal.md) desde Azure Portal o bien créela mediante el [emulador de Azure Cosmos DB](local-emulator.md).

## <a name="step-2-set-up-your-net-project"></a>Paso 2: Configuración del proyecto .NET

Abra el símbolo del sistema de Windows o una ventana de Terminal desde el equipo local. Ejecutará todos los comandos de las secciones siguientes desde el símbolo del sistema o desde el terminal. Ejecute el siguiente comando dotnet new para crear una nueva aplicación con el nombre *bulk-import-demo*. El parámetro `--langVersion` establece la propiedad *LangVersion* en el archivo de proyecto creado.

   ```bash
   dotnet new console –langVersion:8 -n bulk-import-demo
   ```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

   ```bash
   cd bulk-import-demo
   dotnet build
   ```

El resultado esperado de la compilación debe parecerse a lo siguiente:

   ```bash
   Restore completed in 100.37 ms for C:\Users\user1\Downloads\CosmosDB_Samples\bulk-import-demo\bulk-import-demo.csproj.
     bulk -> C:\Users\user1\Downloads\CosmosDB_Samples\bulk-import-demo \bin\Debug\netcoreapp2.2\bulk-import-demo.dll

   Build succeeded.
       0 Warning(s)
       0 Error(s)

   Time Elapsed 00:00:34.17
   ```

## <a name="step-3-add-the-azure-cosmos-db-package"></a>Paso 3: Incorporación del paquete de Azure Cosmos DB

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de Azure Cosmos DB para .NET Core con el comando dotnet add package.

   ```bash
   dotnet add package Microsoft.Azure.Cosmos
   ```

## <a name="step-4-get-your-azure-cosmos-account-credentials"></a>Paso 4: Obtención de las credenciales de la cuenta de Azure Cosmos

La aplicación de ejemplo debe autenticarse para la cuenta de Azure Cosmos. Para realizar la autenticación, debe pasar las credenciales de cuenta de Azure Cosmos a la aplicación. Para obtener las credenciales de cuenta de Azure Cosmos, siga estos pasos:

1.  Inicie sesión en [Azure Portal](https://portal.azure.com/).
1.  Vaya a la cuenta de Azure Cosmos.
1.  Abra el panel **Claves** y copia el **URI** y la **CLAVE PRINCIPAL** de la cuenta.

Si usa el emulador de Azure Cosmos DB, obtenga las [credenciales del emulador de este artículo](local-emulator.md#authenticating-requests).

## <a name="step-5-initialize-the-cosmosclient-object-with-bulk-execution-support"></a>Paso 5: Inicialización del objeto CosmosClient con compatibilidad de ejecución en bloque

Abra el archivo `Program.cs` generado en un editor de código. Creará una nueva instancia de CosmosClient con la ejecución en bloque habilitada y la usará para realizar operaciones en Azure Cosmos DB. 

Comencemos por sobrescribir el método `Main` predeterminado y definir las variables globales. Estas variables globales incluirán el punto de conexión y las claves de autorización, el nombre de la base de datos, el contenedor que creará y el número de elementos que se van a insertar en bloque. Asegúrese de reemplazar los valores de URL de punto de conexión y clave de autorización según su entorno. 


   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Diagnostics;
   using System.IO;
   using System.Text.Json;
   using System.Threading.Tasks;
   using Microsoft.Azure.Cosmos;

   public class Program
   {
        private const string EndpointUrl = "https://<your-account>.documents.azure.com:443/";
        private const string AuthorizationKey = "<your-account-key>";
        private const string DatabaseName = "bulk-tutorial";
        private const string ContainerName = "items";
        private const int ItemsToInsert = 300000;

        static async Task Main(string[] args)
        {

        }
   }
   ```

Dentro del método `Main`, agregue el siguiente código para inicializar el objeto CosmosClient:

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=CreateClient)]

Una vez habilitada la ejecución en bloque, CosmosClient agrupa internamente las operaciones simultáneas en llamadas de servicio únicas. De este modo, optimiza el uso de la capacidad de proceso, ya que distribuye las llamadas de servicio entre las particiones y, por último, asigna los resultados individuales a los llamadores originales.

Después, se puede crear un contenedor para almacenar todos los elementos.  Defina `/pk` como la clave de partición, 50 000 RU/s como capacidad de proceso aprovisionada y una directiva de indexación personalizada que excluya todos los campos para optimizar el proceso de escritura. Agregue el siguiente código después de la instrucción de inicialización de CosmosClient:

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=Initialize)]

## <a name="step-6-populate-a-list-of-concurrent-tasks"></a>Paso 6: Rellenado de una lista de tareas simultáneas

Para aprovechar las ventajas de la compatibilidad con la ejecución en bloque, cree una lista de tareas asincrónicas en función del origen de los datos y de las operaciones que desee realizar, y use `Task.WhenAll` para ejecutarlas simultáneamente.
Comencemos usando datos de "Bogus" para generar una lista de elementos de nuestro modelo de datos. En una aplicación real, los elementos procederían del origen de datos deseado.

En primer lugar, agregue el paquete Bogus a la solución mediante el comando dotnet add package.

   ```bash
   dotnet add package Bogus
   ```

Establezca la definición de los elementos que desea guardar. Debe definir la clase `Item` en el archivo `Program.cs`:

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=Model)]

A continuación, cree una función auxiliar dentro de la clase `Program`. Esta función auxiliar obtendrá el número de elementos que definió para insertar y generará datos aleatorios:

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=Bogus)]

Lea los elementos y serialícelos en instancias de flujo mediante el uso de la clase `System.Text.Json`. Debido a la naturaleza de los datos generados automáticamente, los datos se serializan como flujos. También puede usar la instancia del elemento directamente, pero si las convierte en flujos, puede aprovechar el rendimiento de las API de flujo de CosmosClient. Por lo general, los datos se pueden usar directamente siempre y cuando se conozca la clave de partición. 


Para convertir los datos en instancias de flujo, en el método `Main`, agregue el siguiente código justo después de crear el contenedor:

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=Operations)]

A continuación, use los flujos de datos para crear tareas simultáneas y rellenar la lista de tareas de modo que se inserten los elementos en el contenedor. Para realizar esta operación, agregue el siguiente código a la clase `Program`:

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=ConcurrentTasks)]

Todas estas operaciones de punto simultáneas se ejecutarán juntas (es decir, en bloque) tal como se describe en la sección de introducción.

## <a name="step-7-run-the-sample"></a>Paso 7: Ejecución del ejemplo

Puede ejecutar el ejemplo simplemente con el comando `dotnet`:

   ```bash
   dotnet run
   ```

## <a name="get-the-complete-sample"></a>Obtención del ejemplo completo

Si no dispuso de tiempo para completar los pasos de este tutorial, o simplemente desea descargar los ejemplos de código, puede obtenerlos de [GitHub](https://github.com/Azure-Samples/cosmos-dotnet-bulk-import-throughput-optimizer).

Después de clonar el proyecto, asegúrese de actualizar las credenciales que desee en [Program.cs](https://github.com/Azure-Samples/cosmos-dotnet-bulk-import-throughput-optimizer/blob/master/src/Program.cs#L25).

El ejemplo se puede ejecutar cambiando al directorio del repositorio y usando `dotnet`:

   ```bash
   cd cosmos-dotnet-bulk-import-throughput-optimizer
   dotnet run
   ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha llevado a cabo los siguientes pasos:

> [!div class="checklist"]
> * Creación de una cuenta de Azure Cosmos
> * Configuración del proyecto
> * Conexión a una cuenta de Azure Cosmos con compatibilidad para la ejecución en bloque habilitada
> * Realización de una importación de datos mediante operaciones simultáneas de creación

Ahora puede continuar en el siguiente tutorial:

> [!div class="nextstepaction"]
>[Consulta de Azure Cosmos DB mediante SQL API](tutorial-query-sql-api.md)