---
title: Sincronización de datos desde Azure SQL Edge mediante Azure Data Factory
description: Información sobre la sincronización de datos entre Azure SQL Edge y Azure Blob Storage
keywords: SQL Edge, sincronización de datos desde SQL Edge, factoría de datos de SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 672c9f0d5403ae27a26d58617dca44f0f1121411
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904168"
---
# <a name="tutorial-sync-data-from-sql-edge-to-azure-blob-storage-by-using-azure-data-factory"></a>Tutorial: Sincronización de datos desde SQL Edge a Azure Blob Storage mediante Azure Data Factory

En este tutorial, usará Azure Data Factory para realizar sincronizaciones incrementales de datos en Azure Blob Storage a partir de una tabla de una instancia de Azure SQL Edge.

## <a name="before-you-begin"></a>Antes de empezar

Si no ha creado una base de datos o una tabla en la implementación de Azure SQL Edge, use uno de estos métodos para hacerlo:

* Use [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms/) o [Azure Data Studio](/sql/azure-data-studio/download/) para conectarse a SQL Edge. Ejecute un script SQL para crear la base de datos y la tabla.
* Cree una base de datos y una tabla mediante [SQLCMD](/sql/tools/sqlcmd-utility/), para lo cual debe conectarse directamente al módulo de SQL Edge. Para más información, consulte el artículo en el que se explica cómo [conectarse al motor de base de datos mediante sqlcmd](/sql/ssms/scripting/sqlcmd-connect-to-the-database-engine/).
* Use SQLPackage.exe para implementar un archivo de paquete DAC en el contenedor de SQL Edge. Para automatizar este proceso es preciso especificar el identificador URI del archivo SqlPackage como parte de la configuración de las propiedades deseadas del módulo. También puede usar directamente la herramienta de cliente SqlPackage.exe para implementar un paquete DAC en SQL Edge.

    Para obtener información sobre cómo descargar Sqlpackage.exe, consulte [Descarga e instalación de sqlpackage](/sql/tools/sqlpackage-download/). A continuación, encontrará algunos comandos de ejemplo de SqlPackage.exe. Para más información, consulte la documentación de SqlPackage.exe.

    **Creación de un paquete DAC**

    ```cmd
    sqlpackage /Action:Extract /SourceConnectionString:"Data Source=<Server_Name>,<port>;Initial Catalog=<DB_name>;User ID=<user>;Password=<password>" /TargetFile:<dacpac_file_name>
    ```

    **Aplicación de un paquete DAC**

    ```cmd
    sqlpackage /Action:Publish /Sourcefile:<dacpac_file_name> /TargetServerName:<Server_Name>,<port> /TargetDatabaseName:<DB_Name> /TargetUser:<user> /TargetPassword:<password>
    ```

## <a name="create-a-sql-table-and-procedure-to-store-and-update-the-watermark-levels"></a>Creación de una tabla SQL y un procedimiento para almacenar y actualizar los niveles de marca de agua

Las tablas de marca de agua se usan para almacenar la última marca de tiempo en que los datos se han sincronizado con Azure Storage. Los procedimientos almacenados de Transact-SQL (T-SQL) se utilizan para actualizar la tabla de marca de agua después de cada sincronización.

Ejecute estos comandos en la instancia de SQL Edge:

```sql
    Create table [dbo].[watermarktable]
    (
    TableName varchar(255),
    WatermarkValue datetime,
    )
    GO

    CREATE PROCEDURE usp_write_watermark @timestamp datetime, @TableName varchar(50)  
    AS  
    BEGIN
    UPDATE [dbo].[watermarktable]
    SET [WatermarkValue] = @timestamp WHERE [TableName] = @TableName
    END
    Go
```

## <a name="create-a-data-factory-pipeline"></a>Creación de una canalización de Data Factory

En esta sección, creará una canalización de Azure Data Factory para sincronizar en Azure Blob Storage datos de una tabla de Azure SQL Edge.

### <a name="create-a-data-factory-by-using-the-data-factory-ui"></a>Creación de una factoría de datos mediante la interfaz de usuario de Data Factory

Para crear una factoría de datos, siga las instrucciones que se proporcionan en este [tutorial](../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory).

### <a name="create-a-data-factory-pipeline"></a>Creación de una canalización de Data Factory

1. En la página **Empecemos** de la interfaz de usuario de Data Factory, seleccione **Crear canalización**.

    ![Creación de una canalización de Data Factory](media/tutorial-sync-data-factory/data-factory-get-started.png)

2. En la página **General** de la ventana **Propiedades** de la canalización, escriba el nombre **PeriodicSync**.

3. Agregue la actividad Búsqueda para recuperar el valor de marca de agua anterior. En el panel **Actividades**, expanda **General** y arrastre la actividad **Búsqueda** hasta la superficie del diseñador de canalizaciones. Cambie el nombre de la actividad a **OldWatermark**.

    ![Agregar la búsqueda de la marca de agua antigua](media/tutorial-sync-data-factory/create-old-watermark-lookup.png)

4. Cambie a la pestaña **Configuración** y seleccione **Nuevo** en **Conjunto de datos de origen**. Ahora va a crear un conjunto de datos que represente los datos de la tabla de marca de agua. Esta tabla contiene la marca de agua que se utilizó anteriormente en la operación de copia anterior.

5. En la ventana **Nuevo conjunto de datos**, seleccione **Azure SQL Server** y, después, **Continuar**.  

6. En la ventana **Establecer propiedades** del conjunto de datos, en **Nombre**, escriba **WatermarkDataset**.

7. En **Servicio vinculado**, seleccione **Nuevo** y siga estos pasos:

    1. En **Nombre**, escriba **SQLDBEdgeLinkedService**.

    2. En **Nombre del servidor**, escriba los detalles del servidor de SQL Edge.

    3. Seleccione el **nombre de la base de datos** en la lista.

    4. Escriba su **User name** (Nombre de usuario) y **Password** (Contraseña).

    5. Para probar la conexión a la instancia de SQL Edge, seleccione **Prueba de conexión**.

    6. Seleccione **Crear**.

    ![Creación de un servicio vinculado](media/tutorial-sync-data-factory/create-linked-service.png)

    7. Seleccione **Aceptar**.

8. En la pestaña **Configuración**, seleccione **Edición**.

9. En la pestaña **Conexión**, seleccione **[dbo].[watermarktable]** en **Tabla**. Si desea una vista previa de los datos de la tabla, seleccione **Preview data** (Vista previa de los datos).

10. Cambie al editor de canalización; para ello, seleccione la pestaña de la canalización de la parte superior o el nombre de esta de la vista de árbol de la izquierda. En la ventana de actividades de la actividad Búsqueda, confirme que **WatermarkDataset** está seleccionado en la lista **Conjunto de datos de origen**.

11. En el panel **Actividades**, expanda **General** y arrastre otra actividad **Búsqueda** hasta la superficie del diseñador de canalizaciones. Seleccione el nombre **NewWatermark** en la pestaña **General** de la ventana de propiedades. Esta actividad de búsqueda obtiene el nuevo valor de marca de agua de la tabla que contiene los datos de origen, por lo que se puede copiar en el destino.

12. En la ventana de propiedades de la segunda actividad Búsqueda, cambie a la pestaña **Configuración** y seleccione **Nuevo** para crear un conjunto de datos que apunte a la tabla de origen que contenga el nuevo valor de marca de agua.

13. En la ventana **Nuevo conjunto de datos**, seleccione **Instancia de SQL Edge** y, después, **Continuar**.

    1. En la ventana **Establecer propiedades**, en **Nombre**, escriba **SourceDataset**. En **Servicio vinculado**, seleccione **SQLDBEdgeLinkedService** como servicio vinculado.

    2. En **Tabla**, seleccione la tabla que desea sincronizar. También puede especificar una consulta para este conjunto de datos, como se describe más adelante en este mismo tutorial. La consulta tiene prioridad sobre la tabla que se especifica en este paso.

    3. Seleccione **Aceptar**.

14. Cambie al editor de canalización; para ello, seleccione la pestaña de la canalización de la parte superior o el nombre de esta de la vista de árbol de la izquierda. En la ventana de propiedades de la actividad Búsqueda, confirme que **SourceDataset** está seleccionado en la lista **Conjunto de datos de origen**.

15. Seleccione **Consulta** en **Usar consulta**. Actualice el nombre de la tabla en la siguiente consulta y escriba la consulta. Va a seleccionar solo el valor máximo de `timestamp` de la tabla. Asegúrese de seleccionar **Solo la primera fila**.

    ```sql
    select MAX(timestamp) as NewWatermarkvalue from [TableName]
    ```

    ![seleccionar consulta](media/tutorial-sync-data-factory/select-query-data-factory.png)

16. En el panel **Actividades**, expanda **Mover y transformar**, y arrastre la actividad **Copia** del panel **Actividades** a la superficie del diseñador. Asigne a la actividad el nombre **IncrementalCopy**.

17. Conecte las dos actividades Búsqueda con la actividad Copia; para ello, arrastre el botón verde de las actividades Búsqueda a la actividad Copia. Suelte el botón del mouse cuando vea el color del borde de la actividad de copia cambiar a azul.

18. Seleccione la actividad Copia y confirme que ve sus propiedades en la ventana **Propiedades**.

19. Cambie a la pestaña **Origen** de la ventana **Propiedades** y siga estos pasos:

    1. En el cuadro **Conjunto de datos de origen**, seleccione **SourceDataset**.

    2. En **Usar consulta**, seleccione **Consulta**.

    3. Escriba la consulta SQL en el cuadro **Consulta**. Esta es una consulta de ejemplo:

    ```sql
    select * from TemperatureSensor where timestamp > '@{activity('OldWaterMark').output.firstRow.WatermarkValue}' and timestamp <= '@{activity('NewWaterMark').output.firstRow.NewWatermarkvalue}'
    ```

20. En la pestaña **Receptor**, seleccione **Nuevo** en **Conjunto de datos del receptor**.

21. En este tutorial, el almacén de datos del receptor es de Azure Blob Storage. Seleccione **Azure Blob Storage** y **Continuar** en la ventana **Nuevo conjunto de datos**.

22. En la ventana **Seleccionar formato**, seleccione el formato de los datos y, después, seleccione **Continuar**.

23. En la ventana **Establecer propiedades**, en **Nombre**, escriba **SinkDataset**. En **Servicio vinculado**, seleccione **Nuevo**. Ahora va a crear una conexión (un servicio vinculado) a su instancia de Azure Blob Storage.

24. En la ventana **New Linked Service (Azure Blob Storage)** [Nuevo servicio vinculado (Azure Blob Storage)], siga estos pasos:

    1. En el cuadro **Nombre**, escriba **AzureStorageLinkedService**.

    2. En **Nombre de la cuenta de Storage**, seleccione la cuenta de Azure Storage de su suscripción a Azure.

    3. Pruebe la conexión y seleccione **Finalizar**.

25. En la ventana **Establecer propiedades**, confirme que **AzureStorageLinkedService** está seleccionado en **Servicio vinculado**. Seleccione **Crear** y **Aceptar**.

26. En la pestaña **Receptor**, seleccione **Edición**.

27. Vaya a la pestaña **Conexión** de SinkDataset y siga estos pasos:

    1. En **Ruta de acceso al archivo**, escriba *asdedatasync/incrementalcopy*, donde *asdedatasync* es el nombre del contenedor de blobs e *incrementalcopy* es el nombre de la carpeta. Cree el contenedor, en caso de que no exista, o use el nombre de uno existente. Si no existe, Azure Data Factory crea automáticamente la carpeta de salida *incrementalcopy*. También puede usar el botón **Browse** (Examinar) para ir a una carpeta del contenedor de blobs mediante la **ruta de acceso del archivo**.

    2. En la parte **Archivo** de **Ruta de acceso al archivo**, seleccione **Agregar contenido dinámico [Alt+P]** y, después, escriba **@CONCAT("Incremental-", pipeline().RunId, ".txt")** en la ventana que se abre. Seleccione **Finalizar**. La expresión genera el nombre de archivo dinámicamente. Cada ejecución de canalización tiene un identificador único. La actividad de copia usa el identificador de ejecución para generar el nombre de archivo.

28. Cambie al editor de canalización; para ello, seleccione la pestaña de la canalización de la parte superior o el nombre de esta de la vista de árbol de la izquierda.

29. En el panel **Actividades**, expanda **General**, arrastre la actividad **Procedimiento almacenado** del panel **Actividades** para colocarla en la superficie del diseñador de canalizaciones. Conecte la salida verde (correcto) de la actividad Copia con la actividad Procedimiento almacenado.

30. Seleccione **Actividad Procedimiento almacenado** en el diseñador de canalizaciones y cambie su nombre a **SPtoUpdateWatermarkActivity**.

31. Cambie a la pestaña **Cuenta de SQL** y seleccione ***QLDBEdgeLinkedService** en **Servicio vinculado**.

32. Cambie a la pestaña **Procedimiento almacenado** y siga estos pasos:

    1. En **Nombre de procedimiento almacenado**, seleccione **[dbo].[usp_write_watermark]** .

    2. Para especificar los valores de los parámetros del procedimiento almacenado, seleccione **Parámetro de importación** y escriba los valores siguientes para los parámetros:

    |Nombre|Tipo|Value|
    |-----|----|-----|
    |LastModifiedtime|DateTime|@{activity('NewWaterMark').output.firstRow.NewWatermarkvalue}|
    |TableName|String|@{activity('OldWaterMark').output.firstRow.TableName}|

33. Para comprobar la configuración de la canalización, seleccione **Validate** (Comprobar) en la barra de herramientas. Confirme que no haya errores de comprobación. Para cerrar la ventana **Informe de comprobación de canalización**, seleccione **>>** .

34. Publique las entidades (servicios vinculados, conjuntos de datos y canalizaciones) en el servicio Azure Data Factory, para lo que debe seleccionar el botón **Publicar todo**. Espere hasta que vea un mensaje que confirme que la operación de publicación se ha realizado correctamente.

## <a name="trigger-a-pipeline-based-on-a-schedule"></a>Desencadenamiento de canalizaciones según una programación

1. En la barra de herramientas de la canalización, seleccione **Agregar desencadenador**, después, **Nuevo/Editar** y, finalmente, **+ Nuevo**.

2. Asigne al desencadenador el nombre **HourlySync**. En **Tipo**, seleccione **Programar**. En **Periodicidad**, seleccione cada 1 hora.

3. Seleccione **Aceptar**.

4. Seleccione **Publish All** (Publicar todo).

5. Seleccione **Trigger Now** (Desencadenar ahora).

6. Cambie a la pestaña **Monitor** (Supervisar) de la izquierda. Puede ver el estado de ejecución de la canalización activado por el desencadenador manual. Seleccione **Refresh** (Actualizar) para actualizar la lista.

## <a name="next-steps"></a>Pasos siguientes

La canalización de Azure Data Factory de este tutorial copia los datos de una tabla de una instancia de SQL Edge en una ubicación de Azure Blob Storage cada hora. Para aprender a usar Data Factory en otros escenarios, consulte los siguientes [tutoriales](../data-factory/tutorial-copy-data-portal.md).
