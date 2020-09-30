---
title: Configuración de módulos de IoT Edge en Azure SQL Edge
description: En la segunda parte de este tutorial de tres partes de Azure SQL Edge para predecir impurezas de mineral de hierro, configurará módulos y conexiones de IoT Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sourabha, sstein
ms.date: 09/22/2020
ms.openlocfilehash: 7b2432fda70e8f9a5fa8bc64ede846d977672e9e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90886478"
---
# <a name="set-up-iot-edge-modules-and-connections"></a>Configuración de módulos y conexiones de IoT Edge

En la segunda parte de este tutorial de tres partes para predecir impurezas de mineral de hierro en Azure SQL Edge, configurará los siguientes módulos de IoT Edge:

- Azure SQL Edge
- Módulo IoT Edge del generador de datos

## <a name="specify-container-registry-credentials"></a>Especificación de las credenciales de registro de contenedor

Debe especificar las credenciales en los registros de contenedor que hospedan las imágenes de módulo. Se pueden encontrar en el registro de contenedor que se creó en el grupo de recursos. Vaya a la sección **Claves de acceso**. Anote los campos siguientes:

- Nombre de registro
- Servidor de inicio de sesión
- Nombre de usuario
- Contraseña

Ahora, especifique las credenciales del contenedor en el módulo IoT Edge.

1. Vaya al centro de IoT que creó en el grupo de recursos.

2. En la sección **IoT Edge**, en **Administración automática de dispositivos**, haga clic en **Id. de dispositivo**. Para este tutorial, el identificador es `IronOrePredictionDevice`.

3. Seleccione la sección **Establecer módulos**.

4. En **Credenciales de Container Registry**, escriba los valores siguientes:

   _Campo_|_Valor_
   -------|-------
   Nombre|Nombre de registro
   Dirección|Servidor de inicio de sesión
   Nombre de usuario|Nombre de usuario
   Contraseña|Contraseña
  
## <a name="deploy-the-data-generator-module"></a>Implementación del módulo de generador de datos

1. En la sección **IoT Edge**, en **Administración automática de dispositivos**, haga clic en **Id. de dispositivo**. En este tutorial, el identificador es `IronOrePredictionDevice` y, a continuación, haga clic en **Establecer módulos**.

2.  En la sección **Módulos IoT Edge** en la página **Establecer módulos en el dispositivo:** , haga clic en **+ AGREGAR** y seleccione **Módulo IoT Edge**.

3. Proporcione un nombre y un URI de imagen válidos para módulo IoT Edge.
   El URI de la imagen se puede encontrar en el registro de contenedor del grupo de recursos creado en la primera parte de este tutorial. Seleccione la sección **Repositorios** en **Servicios**. En este tutorial, elija el repositorio denominado `silicaprediction`. Seleccione la etiqueta apropiada. El URI de la imagen tendrá el formato siguiente:

   *servidor de inicio de sesión del registro de contenedor*/*nombre del repositorio*:*nombre de la etiqueta*

   Por ejemplo:

   ```
   ASEdemocontregistry.azurecr.io/silicaprediction:amd64
   ```

4. Deje los campos *Directiva de reinicio* y *Estado deseado* tal como están.

5. Haga clic en **Agregar**.


## <a name="deploy-the-azure-sql-edge-module"></a>Implementación del módulo de Azure SQL Edge

1. Implemente el módulo Azure SQL Edge; para ello, haga clic en **+ Agregar** y, después, en **Módulo de Marketplace**. 

2. En la hoja **Marketplace de módulos IoT Edge**, busque *Azure SQL Edge* y seleccione *Desarrollador de Azure SQL Edge*. 

3. Haga clic en el módulo *Azure SQL Edge* recién agregado en **Módulos IoT Edge** para configurar el módulo Azure SQL Edge. Para obtener más información sobre las opciones de configuración, consulte [Implementación de Azure SQL Edge](https://docs.microsoft.com/azure/azure-sql-edge/deploy-portal).

4. Agregue la variable de entorno `MSSQL_PACKAGE` a la implementación del módulo *Azure SQL Edge* y especifique la dirección URL de SAS del archivo dacpac de base de datos creado en el paso 8 de la [primera parte](tutorial-deploy-azure-resources.md) de este tutorial.

5. Haga clic en **Update** (Actualizar).

6. En la página **Establecer módulos en el dispositivo**, haga clic en **Siguiente: Rutas >** .

7. En el panel de rutas de la página **Establecer módulos en el dispositivo**, especifique las rutas del módulo para la comunicación del centro de IoT Edge como se describe a continuación. Asegúrese de actualizar los nombres de los módulos en las definiciones de ruta siguientes.

   ```
   FROM /messages/modules/<your_data_generator_module>/outputs/IronOreMeasures INTO
   BrokeredEndpoint("/modules/<your_azure_sql_edge_module>/inputs/IronOreMeasures")
   ```

   Por ejemplo:

   ```
   FROM /messages/modules/ASEDataGenerator/outputs/IronOreMeasures INTO BrokeredEndpoint("/modules/AzureSQLEdge/inputs/IronOreMeasures")
   ```


7. En la página **Establecer módulos en el dispositivo**, haga clic en **Siguiente: Revisar y crear >**

8. En la página **Establecer módulos en el dispositivo**, haga clic en **Crear**.

## <a name="create-and-start-the-t-sql-streaming-job-in-azure-sql-edge"></a>Cree e inicie el trabajo de streaming de T-SQL en Azure SQL Edge.

1. Abra Azure Data Studio.

2. En la pestaña **Bienvenida**, inicie una nueva conexión con los detalles siguientes:

   |_Campo_|_Valor_|
   |-------|-------|
   |Tipo de conexión| Microsoft SQL Server|
   |Server|Dirección IP pública mencionada en la VM que se creó para esta demostración.|
   |Nombre de usuario|sa|
   |Contraseña|Contraseña segura que se usó al crear la instancia de Azure SQL Edge.|
   |Base de datos|Valor predeterminado|
   |Grupo del servidor|Valor predeterminado|
   |Nombre (opcional)|Puede proporcionar un nombre opcional.|

3. Haga clic en **Conectar**

4. En la pestaña del menú **Archivo**, abra un cuaderno nuevo o use el método abreviado de teclado CTRL+N.

5. En la nueva ventana Consulta, ejecute el siguiente script para crear el trabajo de streaming de T-SQL. Antes de ejecutar el script, asegúrese de cambiar las siguientes variables. 
   - *SQL_SA_Password:* El valor MSSQL_SA_PASSWORD especificado durante la implementación del módulo Azure SQL Edge. 
   
   ```sql
   Use IronOreSilicaPrediction
   Go

   Declare @SQL_SA_Password varchar(200) = '<SQL_SA_Password>'
   declare @query varchar(max) 

   /*
   Create Objects Required for Streaming
   */

   CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'MyStr0ng3stP@ssw0rd';

   If NOT Exists (select name from sys.external_file_formats where name = 'JSONFormat')
   Begin
      CREATE EXTERNAL FILE FORMAT [JSONFormat]  
      WITH ( FORMAT_TYPE = JSON)
   End 


   If NOT Exists (select name from sys.external_data_sources where name = 'EdgeHub')
   Begin
      Create EXTERNAL DATA SOURCE [EdgeHub] 
      With(
         LOCATION = N'edgehub://'
      )
   End 

   If NOT Exists (select name from sys.external_streams where name = 'IronOreInput')
   Begin
      CREATE EXTERNAL STREAM IronOreInput WITH 
      (
         DATA_SOURCE = EdgeHub,
         FILE_FORMAT = JSONFormat,
         LOCATION = N'IronOreMeasures'
       )
   End


   If NOT Exists (select name from sys.database_scoped_credentials where name = 'SQLCredential')
   Begin
       set @query = 'CREATE DATABASE SCOPED CREDENTIAL SQLCredential
                 WITH IDENTITY = ''sa'', SECRET = ''' + @SQL_SA_Password + ''''
       Execute(@query)
   End 

   If NOT Exists (select name from sys.external_data_sources where name = 'LocalSQLOutput')
   Begin
      CREATE EXTERNAL DATA SOURCE LocalSQLOutput WITH (
      LOCATION = 'sqlserver://tcp:.,1433',CREDENTIAL = SQLCredential)
   End

   If NOT Exists (select name from sys.external_streams where name = 'IronOreOutput')
   Begin
      CREATE EXTERNAL STREAM IronOreOutput WITH 
      (
         DATA_SOURCE = LocalSQLOutput,
         LOCATION = N'IronOreSilicaPrediction.dbo.IronOreMeasurements'
      )
   End

   EXEC sys.sp_create_streaming_job @name=N'IronOreData',
   @statement= N'Select * INTO IronOreOutput from IronOreInput'

   exec sys.sp_start_streaming_job @name=N'IronOreData'
   ```

6. Utilice la siguiente consulta para comprobar que los datos del módulo de generación de datos se estén transmitiendo a la base de datos. 

   ```sql
   Select Top 10 * from dbo.IronOreMeasurements
   order by timestamp desc
   ```


En este tutorial, hemos implementado el módulo generador de datos y el módulo SQL Edge. Después, hemos creado un trabajo de streaming para transmitir los datos generados por el módulo de generación de datos a SQL. 

## <a name="next-steps"></a>Pasos siguientes

- [Implementación del modelo de ML en Azure SQL Edge mediante ONNX](tutorial-run-ml-model-on-sql-edge.md)
