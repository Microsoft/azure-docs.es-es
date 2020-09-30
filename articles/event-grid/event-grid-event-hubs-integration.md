---
title: 'Tutorial: Envío de datos de Event Hubs al almacenamiento de datos: Event Grid'
description: 'Tutorial: Describe cómo usar Azure Event Grid y Azure Event Hubs para migrar datos a una instancia de Azure Synapse Analytics. Usa una función de Azure para recuperar un archivo de Capture.'
ms.topic: tutorial
ms.date: 07/07/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: d45fcedb570e384b851a7ac815ca175c67cc00a0
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89435038"
---
# <a name="tutorial-stream-big-data-into-a-data-warehouse"></a>Tutorial: Transmitir macrodatos a un almacenamiento de datos
Azure [Event Grid](overview.md) es un servicio inteligente de enrutamiento de eventos que permite reaccionar ante las notificaciones (eventos) procedentes de aplicaciones y servicios. Por ejemplo, puede desencadenar una función de Azure que procese los datos de Event Hubs que se han capturado en una instancia de Azure Blob Storage o Azure Data Lake Store y migrar esos datos a otros repositorios. En este [ejemplo de integración de Event Hubs y Event Grid](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo) se muestra cómo usar Event Hubs con Event Grid para migrar datos capturados de Event Hubs procedentes de Blob Storage a una instancia de Azure Synapse Analytics (anteriormente SQL Data Warehouse) sin ningún tipo de problema.

![Información general de la aplicación](media/event-grid-event-hubs-integration/overview.png)

En este diagrama se muestra el flujo de trabajo de la solución que se compila en este tutorial: 

1. Los datos que se envían a un centro de eventos de Azure se capturan en una instancia de Azure Blob Storage.
2. Una vez que se completa la captura de los datos, se genera un evento y se envía a una cuadrícula de eventos de Azure. 
3. La cuadrícula de eventos desvía estos datos de evento a una aplicación de función de Azure.
4. La aplicación de función usa la dirección URL del blob de los datos del evento para recuperar el blob desde el almacenamiento. 
5. La aplicación de funciones migra los datos del blob a una instancia de Azure Synapse Analytics. 

En este artículo, podrá llevar a cabo estos pasos:

> [!div class="checklist"]
> * Usar una plantilla de Azure Resource Manager para implementar la infraestructura: un centro de eventos, una cuenta de almacenamiento, una aplicación de funciones y una instancia de Synapse Analytics.
> * Crear una tabla en el almacenamiento de datos.
> * Agregar código a la aplicación de función.
> * Suscribirse al evento. 
> * Ejecutar una aplicación que envía los datos al centro de eventos.
> * Ver los datos migrados en el almacenamiento de datos.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para realizar este tutorial, necesitará lo siguiente:

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* [Visual Studio 2019](https://www.visualstudio.com/vs/) con cargas de trabajo para: desarrollo de escritorio de .NET, desarrollo de Azure, desarrollo web y de ASP.NET, desarrollo de Node.js y desarrollo de Python.
* Descargue el [proyecto de ejemplo EventHubsCaptureEventGridDemo](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo) en su equipo.

## <a name="deploy-the-infrastructure"></a>Implementar la infraestructura
En este paso, implementará la infraestructura requerida con una [plantilla de Resource Manager](https://github.com/Azure/azure-docs-json-samples/blob/master/event-grid/EventHubsDataMigration.json). Cuando implemente la plantilla, se crearán estos recursos:

* Un centro de eventos con la característica Captura habilitada.
* Una cuenta de almacenamiento para los archivos capturados. 
* Un plan de App Service para hospedar la aplicación de función
* Una aplicación de función para procesar el evento
* Un servidor SQL Server para hospedar el almacenamiento de datos
* Azure Synapse Analytics para el almacenamiento de los datos migrados

### <a name="launch-azure-cloud-shell-in-azure-portal"></a>Inicio de Azure Cloud Shell en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. Seleccione el botón **Cloud Shell** que se encuentra en la parte superior.

    ![Azure portal](media/event-grid-event-hubs-integration/azure-portal.png)
3. Verá Cloud Shell abierto en la parte inferior del explorador.

    ![Cloud Shell](media/event-grid-event-hubs-integration/launch-cloud-shell.png) 
4. En Cloud Shell, si ve una opción para seleccionar entre **Bash** y **PowerShell**, seleccione **Bash**. 
5. Si usa Cloud Shell por primera vez, seleccione **Crear almacenamiento** para crear una cuenta de almacenamiento. Azure Cloud Shell requiere una cuenta de Azure Storage donde almacenar algunos archivos. 

    ![Creación de almacenamiento para Cloud Shell](media/event-grid-event-hubs-integration/create-storage-cloud-shell.png)
6. Espere hasta que se inicialice Cloud Shell. 

    ![Creación de almacenamiento para Cloud Shell](media/event-grid-event-hubs-integration/cloud-shell-initialized.png)


### <a name="use-azure-cli"></a>Uso de CLI de Azure

1. Para crear un grupo de recursos de Azure, ejecute el comando de la CLI siguiente: 
    1. Copie y pegue este comando en la ventana Cloud Shell. Cambie el nombre del grupo de recursos y su ubicación si lo desea.

        ```azurecli
        az group create -l eastus -n rgDataMigration
        ```
    2. Presione **ENTRAR**. 

        Este es un ejemplo:
    
        ```azurecli
        user@Azure:~$ az group create -l eastus -n ehubegridgrp
        {
          "id": "/subscriptions/00000000-0000-0000-0000-0000000000000/resourceGroups/ehubegridgrp",
          "location": "eastus",
          "managedBy": null,
          "name": "ehubegridgrp",
          "properties": {
            "provisioningState": "Succeeded"
          },
          "tags": null
        }
        ```
2. Para implementar todos los recursos mencionados en la sección anterior (centro de eventos, cuenta de almacenamiento, aplicación de funciones, instancia de Azure Synapse Analytics), ejecute el siguiente comando de la CLI: 
    1. Copie y pegue el comando en la ventana Cloud Shell. También puede que quiera copiar y pegar en el editor que prefiera, establecer valores y luego copiar el comando en Cloud Shell. 

        ```azurecli
        az group deployment create \
            --resource-group rgDataMigration \
            --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json \
            --parameters eventHubNamespaceName=<event-hub-namespace> eventHubName=hubdatamigration sqlServerName=<sql-server-name> sqlServerUserName=<user-name> sqlServerPassword=<password> sqlServerDatabaseName=<database-name> storageName=<unique-storage-name> functionAppName=<app-name>
        ```
    2. Especifique valores para las entidades siguientes:
        1. Nombre del grupo de recursos que creó anteriormente.
        2. Nombre del espacio de nombres del centro de eventos. 
        3. Nombre del centro de eventos. Puede dejar el valor sin modificaciones (hubdatamigration).
        4. Nombre del servidor SQL Server.
        5. Nombre del usuario y la contraseña de SQL. 
        6. Nombre de la instancia de Azure Synapse Analytics.
        7. Nombre de la cuenta de almacenamiento. 
        8. Nombre de la aplicación de función. 
    3.  Presione **ENTRAR** en la ventana Cloud Shell para ejecutar el comando. Este proceso puede tardar un poco debido a que se están creando varios recursos. En el resultado del comando, asegúrese de que no se produjo ningún error. 
    

### <a name="use-azure-powershell"></a>Uso de Azure PowerShell

1. En Azure Cloud Shell, cambie al modo PowerShell. Seleccione la flecha abajo en la esquina superior izquierda de Azure Cloud Shell y seleccione **PowerShell**.

    ![Cambio a PowerShell](media/event-grid-event-hubs-integration/select-powershell-cloud-shell.png)
2. Para crear un grupo de recursos de Azure, ejecute el comando siguiente: 
    1. Copie y pegue este comando en la ventana Cloud Shell.

        ```powershell
        New-AzResourceGroup -Name rgDataMigration -Location eastus
        ```
    2. Especifique un nombre para el **grupo de recursos**.
    3. Presione ENTRAR. 
3. Para implementar todos los recursos mencionados en la sección anterior (centro de eventos, cuenta de almacenamiento, aplicación de funciones, instancia de Azure Synapse Analytics), ejecute el siguiente comando:
    1. Copie y pegue el comando en la ventana Cloud Shell. También puede que quiera copiar y pegar en el editor que prefiera, establecer valores y luego copiar el comando en Cloud Shell. 

        ```powershell
        New-AzResourceGroupDeployment -ResourceGroupName rgDataMigration -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json -eventHubNamespaceName <event-hub-namespace> -eventHubName hubdatamigration -sqlServerName <sql-server-name> -sqlServerUserName <user-name> -sqlServerDatabaseName <database-name> -storageName <unique-storage-name> -functionAppName <app-name>
        ```
    2. Especifique valores para las entidades siguientes:
        1. Nombre del grupo de recursos que creó anteriormente.
        2. Nombre del espacio de nombres del centro de eventos. 
        3. Nombre del centro de eventos. Puede dejar el valor sin modificaciones (hubdatamigration).
        4. Nombre del servidor SQL Server.
        5. Nombre del usuario y la contraseña de SQL. 
        6. Nombre de la instancia de Azure Synapse Analytics.
        7. Nombre de la cuenta de almacenamiento. 
        8. Nombre de la aplicación de función. 
    3.  Presione **ENTRAR** en la ventana Cloud Shell para ejecutar el comando. Este proceso puede tardar un poco debido a que se están creando varios recursos. En el resultado del comando, asegúrese de que no se produjo ningún error. 

### <a name="close-the-cloud-shell"></a>Cierre de Cloud Shell 
Para cerrar Cloud Shell, seleccione el botón **Cloud Shell** en el portal (o) el botón **X** que se encuentra en la esquina superior derecha de la ventana Cloud Shell. 

### <a name="verify-that-the-resources-are-created"></a>Comprobación de la creación de los recursos

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda. 
2. Para filtrar la lista de grupos de recursos, escriba el nombre del grupo de recursos en el cuadro de búsqueda. 
3. Seleccione el grupo de recursos en la lista.

    ![Seleccionar el grupo de recursos](media/event-grid-event-hubs-integration/select-resource-group.png)
4. Confirme que ve los recursos siguientes en el grupo de recursos:

    ![Recursos del grupo de recursos](media/event-grid-event-hubs-integration/resources-in-resource-group.png)

### <a name="create-a-table-in-azure-synapse-analytics"></a>Creación de una tabla en Azure Synapse Analytics
Para crear una tabla en el almacenamiento de datos, ejecute el script [CreateDataWarehouseTable.sql](https://github.com/Azure/azure-event-hubs/blob/master/samples/e2e/EventHubsCaptureEventGridDemo/scripts/CreateDataWarehouseTable.sql). Para ejecutar el script, puede usar Visual Studio o el Editor de consultas en el portal. Estos pasos le muestran cómo usar el Editor de consultas: 

1. En la lista de recursos del grupo de recursos, seleccione el **grupo de Synapse SQL (almacenamiento de datos)** . 
2. En la página de Azure Synapse Analytics, seleccione **Editor de consultas (versión preliminar)** en el menú de la izquierda. 

    ![Página de Azure Synapse Analytics](media/event-grid-event-hubs-integration/sql-data-warehouse-page.png)
2. Escriba el nombre de **usuario** y la **contraseña** del servidor SQL Server y seleccione **Aceptar**. Es posible que necesite agregar la dirección IP del cliente al firewall para iniciar sesión correctamente en SQL Server. 

    ![Autenticación de SQL Server](media/event-grid-event-hubs-integration/sql-server-authentication.png)
4. En la ventana de consulta, copie y ejecute el script SQL siguiente: 

    ```sql
    CREATE TABLE [dbo].[Fact_WindTurbineMetrics] (
        [DeviceId] nvarchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL, 
        [MeasureTime] datetime NULL, 
        [GeneratedPower] float NULL, 
        [WindSpeed] float NULL, 
        [TurbineSpeed] float NULL
    )
    WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
    ```

    ![Ejecución de la consulta SQL](media/event-grid-event-hubs-integration/run-sql-query.png)
5. Mantenga abierta esta pestaña o ventana para poder comprobar que los datos se crean al final del tutorial. 

### <a name="update-the-function-runtime-version"></a>Actualización de la versión del runtime de la función

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Seleccione el grupo de recursos en el que exista la aplicación de funciones. 
3. Seleccione la aplicación de funciones del tipo **App Service** en la lista de recursos del grupo de recursos.
4. Seleccione **Configuración** en **Configuración** en el menú de la izquierda. 
5. Cambie a la pestaña **Configuración del tiempo de ejecución de la función**, en el panel derecho. 
5. Actualice la **versión del runtime** a la **versión ~3**. 

    ![Actualización de la versión del runtime de la función](media/event-grid-event-hubs-integration/function-runtime-version.png)
    

## <a name="publish-the-azure-functions-app"></a>Publicar la aplicación de Azure Functions

1. Inicie Visual Studio.
2. Abra la solución **EventHubsCaptureEventGridDemo.sln** que descargó de [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo) como parte de los requisitos previos.
3. En el Explorador de soluciones, haga clic con el botón derecho en **FunctionEGDWDumper** y seleccione **Publicar**.

   ![Publicar la aplicación de función](media/event-grid-event-hubs-integration/publish-function-app.png)
4. Si ve la pantalla siguiente, seleccione **Iniciar**. 

   ![Botón Iniciar publicación](media/event-grid-event-hubs-integration/start-publish-button.png) 
5. En el cuadro de diálogo **Publicar**, seleccione **Azure** en **Destino**y seleccione **Siguiente**. 

   ![Botón Iniciar publicación](media/event-grid-event-hubs-integration/publish-select-azure.png)
6. Seleccione **Azure Function App (Windows)**  y seleccione **Siguiente**. 

   ![Seleccionar Azure Function App (Windows)](media/event-grid-event-hubs-integration/select-azure-function-windows.png)
7. En la pestaña **Functions instance** (Instancia de Functions), seleccione su suscripción de Azure, expanda el grupo de recursos, seleccione su aplicación de funciones y, después, seleccione **Finalizar**. Si aún no lo ha hecho, es preciso que inicie sesión en su cuenta de Azure. 

   ![Selección de la aplicación de función](media/event-grid-event-hubs-integration/publish-select-function-app.png)
8. En la sección **Dependencias de servicio**, seleccione **Configurar**.
9. En la página **Configure dependency** (Configurar dependencia), seleccione la cuenta de almacenamiento que creó anteriormente y, después, seleccione **Siguiente**. 
10. Mantenga la configuración del nombre y valor de la cadena de conexión y seleccione **Siguiente**.
11. Desactive la opción **Secrets store** (Almacén de secretos) y, después, seleccione **Finish** (Finalizar).  
8. Cuando Visual Studio haya configurado el perfil, seleccione **Publicar**.

   ![Select publish](media/event-grid-event-hubs-integration/select-publish.png)

Después de publicar la función, estará listo para suscribirse al evento.

## <a name="subscribe-to-the-event"></a>Nos suscribiremos al evento.

1. En una pestaña o ventana nueva de un explorador web, vaya a [Azure Portal](https://portal.azure.com).
2. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda. 
3. Para filtrar la lista de grupos de recursos, escriba el nombre del grupo de recursos en el cuadro de búsqueda. 
4. Seleccione el grupo de recursos en la lista.

    ![Seleccionar el grupo de recursos](media/event-grid-event-hubs-integration/select-resource-group.png)
4. Seleccione el plan de App Service (no la instancia de App Service) en la lista de recursos del grupo de recursos. 
5. En la página Plan de App Service, seleccione **Aplicaciones** en el menú de la izquierda y, luego, la aplicación de función. 

    ![Selección de la aplicación de función](media/event-grid-event-hubs-integration/select-function-app-app-service-plan.png)
6. Expanda la aplicación de función, expanda las funciones y seleccione la función. 
7. Seleccione **Agregar suscripción de Event Grid** en la barra de herramientas. 

    ![Seleccionar la función de Azure](media/event-grid-event-hubs-integration/select-function-add-button.png)
8. En la página **Creación de una suscripción de Event Grid**, haga lo siguiente: 
    1. En la página **DETALLES DE SUSCRIPCIONES DE EVENTOS**, escriba un nombre para la suscripción (por ejemplo, captureEventSub) y seleccione **Crear**. 
    2. En la sección **DETALLES DEL TEMA**, haga lo siguiente:
        1. Seleccione **Espacios de nombres de Event Hubs** en **Tipos de temas**. 
        2. Seleccione su suscripción a Azure.
        2. Seleccione el grupo de recursos de Azure.
        3. Seleccione el espacio de nombres de Event Hubs.
    3. En la sección **EVENT TYPES** (TIPOS DE EVENTO), confirme que **Capture File Created** (Capturar archivo creado) está seleccionado en **Filter to Event Types** (Filtro para tipos de evento). 
    4. En la sección **ENDPOINT DETAILS** (Detalles de punto de conexión), confirme que en **Endpoint type** (Tipo de punto de conexión) se ha seleccionado **Azure Function** (Función de Azure) y que en **Endpoint** (Punto de conexión) se haya seleccionado una función de Azure. 
    
        ![Creación de una suscripción de Event Grid](media/event-grid-event-hubs-integration/create-event-subscription.png)

## <a name="run-the-app-to-generate-data"></a>Ejecutar la aplicación para generar datos
Ya ha terminado de configurar el centro de eventos, Azure Synapse Analytics, la aplicación de Azure Functions y la suscripción a eventos. Hay que configurar algunos valores antes de ejecutar una aplicación que genere los datos del centro de eventos.

1. En Azure Portal, vaya al grupo de recursos como lo hizo anteriormente. 
2. Seleccione el espacio de nombres de Event Hubs.
3. En la página **Espacio de nombres de Event Hubs**, seleccione **Directivas de acceso compartido** en el menú de la izquierda.
4. Seleccione **RootManageSharedAccessKey** en la lista de directivas. 
5. Seleccione el botón Copiar que está junto al cuadro de texto **Cadena de conexión: clave principal**. 

    ![Cadena de conexión del espacio de nombres del centro de eventos](media/event-grid-event-hubs-integration/get-connection-string.png)
1. Vuelva a la solución de Visual Studio. 
2. En el proyecto WindTurbineDataGenerator, abra **program.cs**.
5. Reemplace los dos valores constantes. Use el valor copiado en **EventHubConnectionString**. Use **hubdatamigration** como el nombre del concentrador de eventos. Si usó otro nombre para el centro de eventos, especifíquelo. 

   ```cs
   private const string EventHubConnectionString = "Endpoint=sb://demomigrationnamespace.servicebus.windows.net/...";
   private const string EventHubName = "hubdatamigration";
   ```

6. Compile la solución. Ejecute la aplicación **WindTurbineGenerator.exe**. 
7. Transcurridos unos minutos, realice una consulta en la tabla del almacenamiento de datos relativa a los datos migrados.

    ![Resultados de la consulta](media/event-grid-event-hubs-integration/query-results.png)

### <a name="event-data-generated-by-the-event-hub"></a>Datos de eventos generados por el centro de eventos
Event Grid distribuye datos del evento a los suscriptores. En el ejemplo siguiente se muestran los datos de eventos que se generan cuando el flujo de datos a través de un centro de eventos se captura en un blob. En concreto, observe que la propiedad `fileUrl` del objeto `data` apunta al blob en el almacenamiento. La aplicación de función usa esta dirección URL para recuperar el archivo de blob con los datos capturados.

```json
[
    {
        "topic": "/subscriptions/<guid>/resourcegroups/rgDataMigrationSample/providers/Microsoft.EventHub/namespaces/tfdatamigratens",
        "subject": "eventhubs/hubdatamigration",
        "eventType": "Microsoft.EventHub.CaptureFileCreated",
        "eventTime": "2017-08-31T19:12:46.0498024Z",
        "id": "14e87d03-6fbf-4bb2-9a21-92bd1281f247",
        "data": {
            "fileUrl": "https://tf0831datamigrate.blob.core.windows.net/windturbinecapture/tfdatamigratens/hubdatamigration/1/2017/08/31/19/11/45.avro",
            "fileType": "AzureBlockBlob",
            "partitionId": "1",
            "sizeInBytes": 249168,
            "eventCount": 1500,
            "firstSequenceNumber": 2400,
            "lastSequenceNumber": 3899,
            "firstEnqueueTime": "2017-08-31T19:12:14.674Z",
            "lastEnqueueTime": "2017-08-31T19:12:44.309Z"
        }
    }
]
```


## <a name="next-steps"></a>Pasos siguientes

* Para información sobre las diferencias en los servicios de mensajería de Azure, consulte [Elija entre los servicios de Azure de entrega de mensajes](compare-messaging-services.md).
* Para obtener una introducción a Event Grid, vea [Acerca de Event Grid](overview.md).
* Para obtener una introducción a la función de captura de Event Hubs, vea [Habilitación de la funcionalidad de captura de Event Hubs mediante Azure Portal](../event-hubs/event-hubs-capture-enable-through-portal.md).
* Para más información sobre cómo configurar y ejecutar el ejemplo, vea el [ejemplo de la función de captura de Event Hubs y Event Grid](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo).
