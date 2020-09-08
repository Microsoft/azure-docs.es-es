---
title: Configuración de una canalización de CI/CD con la tarea de compilación del emulador de Azure Cosmos DB
description: Tutorial acerca de cómo configurar el flujo de trabajo de compilación y publicación en Azure DevOps mediante la tarea de compilación del emulador de Cosmos DB
author: deborahc
ms.service: cosmos-db
ms.topic: how-to
ms.date: 01/28/2020
ms.author: dech
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 605fba03e65d4200d0f1e18219e892ec6d207bc4
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019324"
---
# <a name="set-up-a-cicd-pipeline-with-the-azure-cosmos-db-emulator-build-task-in-azure-devops"></a>Configuración de una canalización de CI/CD con la tarea de compilación del emulador de Azure Cosmos DB en Azure DevOps

El emulador de Azure Cosmos DB proporciona un entorno local que emula el servicio Azure Cosmos DB con fines de desarrollo. Dicho emulador permite desarrollar y probar aplicaciones localmente sin necesidad de crear una suscripción a Azure ni incurrir en gastos. 

La tarea de compilación del emulador de Azure Cosmos DB para Azure DevOps permite hacer lo mismo en un entorno de integración continua. Con la tarea de compilación, se pueden realizar pruebas en el emulador como parte de los flujos de trabajo de compilación y publicación. La tarea pone en marcha un contenedor de Docker con el emulador ya en ejecución y proporciona un punto de conexión que puede utilizar el resto de la definición de compilación. Puede crear e inicie tantas instancias del emulador como sea necesario, y cada una de ellas se ejecuta en un contenedor independiente. 

En este artículo se muestra cómo configurar una canalización de integración continua en Azure DevOps para una aplicación de ASP.NET que usa la tarea de compilación del emulador de Cosmos DB para realizar las pruebas. Puede usar un enfoque similar para configurar una canalización de CI para una aplicación Node.js o Python. 

## <a name="install-the-emulator-build-task"></a>Instalación de la tarea de compilación del emulador

Para usar la tarea de compilación, en primer lugar es preciso instalarla en la organización de Azure DevOps. Busque la extensión **emulador de Azure Cosmos DB** en el [Marketplace](https://marketplace.visualstudio.com/items?itemName=azure-cosmosdb.emulator-public-preview) y haga clic en **Consígalo gratis.**

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_1.png" alt-text="Busque e instale la tarea de compilación del emulador de Azure Cosmos DB en el Marketplace de Azure DevOps":::

A continuación, elija la organización en la que desea instalar la extensión. 

> [!NOTE]
> Para instalar una extensión en una organización de Azure DevOps, es preciso ser propietario de una cuenta o administrador de una colección de proyectos. Si no tiene permisos, pero es miembro de una cuenta, puede solicitar extensiones en su lugar. [Más información.](https://docs.microsoft.com/azure/devops/marketplace/faq-extensions?view=vsts)

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_2.png" alt-text="Elija la organización de Azure DevOps en la que instalar una extensión":::

## <a name="create-a-build-definition"></a>Creación de una definición de compilación

Ahora que la extensión está instalada, inicie sesión en la organización de Azure DevOps y busque el proyecto en el panel de proyectos. Puede agregar una [canalización de compilación](https://docs.microsoft.com/azure/devops/pipelines/get-started-designer?view=vsts&tabs=new-nav) al proyecto o modificar una existente. Si ya dispone de una canalización de compilación, puede pasar directamente al apartado de [incorporación de una tarea de compilación del emulador a una definición de compilación](#addEmulatorBuildTaskToBuildDefinition).

1. Para crear una definición de compilación, vaya a la pestaña **Compilaciones** de Azure DevOps. Seleccione **+Nuevo.** \> **Nueva canalización de compilación**

   :::image type="content" source="./media/tutorial-setup-ci-cd/CreateNewBuildDef_1.png" alt-text="Crear una canalización de compilación":::

2. Seleccione el elemento **origen** que desee, el **Proyecto de equipo**, el **Repositorio** y la **Rama predeterminada para compilaciones manuales y programadas**. Después de elegir las opciones necesarias, seleccione **Continuar**.

   :::image type="content" source="./media/tutorial-setup-ci-cd/CreateNewBuildDef_2.png" alt-text="Seleccionar el proyecto de equipo, el repositorio y la rama de la canalización de compilación":::

3. Por último, seleccione la plantilla que desee para la canalización de compilación. En este tutorial se seleccionará la plantilla **ASP.NET**. Ya hay una canalización de compilación que se puede configurar para usar la tarea de compilación del emulador de Azure Cosmos DB. 

> [!NOTE]
> El grupo de agentes que se va a seleccionar para la CI debe tener instalado Docker para Windows, a menos que la instalación se haga manualmente en una tarea anterior como parte de la CI. Consulte el artículo sobre los [agentes hospedados de Microsoft](https://docs.microsoft.com/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml) para ver una selección de los grupos de agentes. Se recomienda empezar con `Hosted VS2017`.

El emulador de Azure Cosmos DB no es compatible actualmente con el grupo de agentes VS2019 hospedado. Sin embargo, el emulador ya incluye VS2019, que se inicia con los siguientes cmdlets de PowerShell. Si tiene problemas para usar VS2019, póngase en contacto con el equipo de [Azure DevOps](https://developercommunity.visualstudio.com/spaces/21/index.html) para obtener ayuda:

```powershell
Import-Module "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules\Microsoft.Azure.CosmosDB.Emulator"
Start-CosmosDbEmulator
```

## <a name="add-the-task-to-a-build-pipeline"></a><a name="addEmulatorBuildTaskToBuildDefinition"></a>Incorporación de la tarea a una canalización de compilación

1. Antes de agregar una tarea a la canalización de compilación, debe agregar un trabajo de agente. Vaya a la canalización de compilación, seleccione **...** y elija **Agregar un trabajo de agente**.

1. A continuación, seleccione el símbolo **+** junto el trabajo de agente para agregar la tarea de compilación del emulador. Busque **cosmos** en el cuadro de búsqueda, seleccione el **Emulador de Azure Cosmos DB** y agréguelo al trabajo de agente. La tarea de compilación iniciará un contenedor con una instancia del emulador de Cosmos DB que ya se ejecuta en él. La tarea de emulador de Azure Cosmos DB debe colocarse antes de cualquier otra tarea que espere que el emulador esté en estado de ejecución.

   :::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_3.png" alt-text="Incorporar la tarea de compilación del emulador a la definición de compilación":::

En este tutorial se agregará la tarea al principio para garantizar que el emulador está disponible antes de que se ejecuten las pruebas.

### <a name="add-the-task-using-yaml"></a>Incorporación de la tarea mediante YAML

Este paso es opcional y solo es necesario si está configurando la canalización de CI/CD mediante una tarea YAML. En estos casos, puede definir la tarea YAML tal como se muestra en el código siguiente:

```yml
- task: azure-cosmosdb.emulator-public-preview.run-cosmosdbemulatorcontainer.CosmosDbEmulator@2
  displayName: 'Run Azure Cosmos DB Emulator'

- script: yarn test
  displayName: 'Run API tests (Cosmos DB)'
  env:
    HOST: $(CosmosDbEmulator.Endpoint)
    # Hardcoded key for emulator, not a secret
    AUTH_KEY: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
    # The emulator uses a self-signed cert, disable TLS auth errors
    NODE_TLS_REJECT_UNAUTHORIZED: '0'
```

## <a name="configure-tests-to-use-the-emulator"></a>Configuración de pruebas para usar el emulador

Ahora, se van a configurar las pruebas para usar el emulador. La tarea de compilación del emulador exporta una variable de entorno, 'CosmosDbEmulator.Endpoint', a la que las tareas posteriores de la canalización de compilación pueden realizar solicitudes. 

En este tutorial, se va a usar la tarea [Visual Studio Test](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md) para ejecutar pruebas unitarias configuradas mediante un archivo **.runsettings**. Para más información acerca de la configuración de pruebas unitarias, visite la [documentación](https://docs.microsoft.com/visualstudio/test/configure-unit-tests-by-using-a-dot-runsettings-file?view=vs-2017). El ejemplo de código de aplicación de lista de tareas completo que usa en este documento está disponible en [GitHub](https://github.com/Azure-Samples/documentdb-dotnet-todo-app).

A continuación encontrará un ejemplo de un archivo **.runsettings** que define los parámetros que se van a pasan a las pruebas unitarias de una aplicación. Tenga en cuenta que la variable `authKey` que se usa es la [clave conocida](https://docs.microsoft.com/azure/cosmos-db/local-emulator#authenticating-requests) del emulador. `authKey` es la clave que espera la tarea de compilación del emulador y debe estar definida en el archivo **.runsettings**.

```csharp
<RunSettings>
    <TestRunParameters>
    <Parameter name="endpoint" value="https://localhost:8081" />
    <Parameter name="authKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==" />
    <Parameter name="database" value="ToDoListTest" />
    <Parameter name="collection" value="ItemsTest" />
  </TestRunParameters>
</RunSettings>
```

Si va a configurar una canalización de CI/CD para una aplicación que utiliza la API de Azure Cosmos DB para MongoDB, la cadena de conexión incluye de forma predeterminada el número de puerto 10255. Sin embargo, este puerto no está abierto actualmente. Como alternativa, debe usar el puerto 10250 para establecer la conexión. La cadena de conexión de la API de Azure Cosmos DB para MongoDB permanece igual, salvo que el número de puerto admitido es 10250 en lugar de 10255.

A los parámetros `TestRunParameters` se les hace referencia mediante una propiedad `TestContext` en el proyecto de prueba de la aplicación. Este es un ejemplo de una prueba que se ejecuta en Cosmos DB.

```csharp
namespace todo.Tests
{
    [TestClass]
    public class TodoUnitTests
    {
        public TestContext TestContext { get; set; }

        [TestInitialize()]
        public void Initialize()
        {
            string endpoint = TestContext.Properties["endpoint"].ToString();
            string authKey = TestContext.Properties["authKey"].ToString();
            Console.WriteLine("Using endpoint: ", endpoint);
            DocumentDBRepository<Item>.Initialize(endpoint, authKey);
        }
        [TestMethod]
        public async Task TestCreateItemsAsync()
        {
            var item = new Item
            {
                Id = "1",
                Name = "testName",
                Description = "testDescription",
                Completed = false,
                Category = "testCategory"
            };

            // Create the item
            await DocumentDBRepository<Item>.CreateItemAsync(item);
            // Query for the item
            var returnedItem = await DocumentDBRepository<Item>.GetItemAsync(item.Id, item.Category);
            // Verify the item returned is correct.
            Assert.AreEqual(item.Id, returnedItem.Id);
            Assert.AreEqual(item.Category, returnedItem.Category);
        }

        [TestCleanup()]
        public void Cleanup()
        {
            DocumentDBRepository<Item>.Teardown();
        }
    }
}
```

Vaya a las opciones de ejecución de la tarea Visual Studio Test. En la opción **Archivo de configuración**, especifique que las pruebas se configuran mediante el archivo **.runsettings**. En la opción **Reemplazar parámetros de serie de pruebas**, agregue `-endpoint $(CosmosDbEmulator.Endpoint)`. Al hacerlo, configurará la tarea para hacer referencia al punto de conexión de la tarea de compilación del emulador, en lugar de al definido en el archivo **.runsettings**.  

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_5.png" alt-text="Reemplazar la variable del punto de conexión por el punto de conexión de la tarea de compilación del emulador":::

## <a name="run-the-build"></a>Ejecute la compilación

Ahora, seleccione **Guardar y poner en cola** la compilación. 

:::image type="content" source="./media/tutorial-setup-ci-cd/runBuild_1.png" alt-text="Guardar y ejecutar la compilación":::

Una vez que se ha iniciado la compilación, observe que la tarea del emulador de Cosmos DB ha comenzado a extraer la imagen de Docker con el emulador instalado. 

:::image type="content" source="./media/tutorial-setup-ci-cd/runBuild_4.png" alt-text="Guardar y ejecutar la compilación":::

Cuando se complete la compilación, observe que pasa las pruebas; todas ellas se ejecutan en el emulador de Cosmos DB desde la tarea de compilación.

:::image type="content" source="./media/tutorial-setup-ci-cd/buildComplete_1.png" alt-text="Guardar y ejecutar la compilación":::

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca del uso del emulador para el desarrollo y las pruebas locales, consulte [Uso del Emulador de Azure Cosmos DB para desarrollo y pruebas de forma local](https://docs.microsoft.com/azure/cosmos-db/local-emulator).

Para exportar los certificados TLS/SSL del emulador, consulte [Exportación de los certificados del emulador de Azure Cosmos DB para su uso con Java, Python y Node.js](https://docs.microsoft.com/azure/cosmos-db/local-emulator-export-ssl-certificates)
