---
title: Uso de puntos de conexión privados para crear una canalización de Azure Data Factory
description: En este tutorial se proporcionan instrucciones paso a paso para usar Azure Portal para crear una factoría de datos con una canalización. La canalización usa la actividad de copia para copiar los datos desde Azure Blob Storage a una base de datos de Azure SQL.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 05/15/2020
ms.author: jingwang
ms.openlocfilehash: 9458c89922fab8450b7cb8e202491d4c47e250e8
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86540524"
---
# <a name="copy-data-securely-from-azure-blob-storage-to-a-sql-database-by-using-private-endpoints"></a>Copia de datos de forma segura desde Azure Blob Storage a SQL Database mediante puntos de conexión privados

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial, creará una factoría de datos mediante la interfaz de usuario (UI) de Azure Data Factory. **La canalización de esta factoría de datos copia los datos de forma segura desde Azure Blob Storage a una base de datos de Azure SQL (en ambas se permite el acceso solo a las redes seleccionadas) mediante puntos de conexión privados en una [red virtual administrada por Azure Data Factory](managed-virtual-network-private-endpoint.md).** El patrón de configuración de este tutorial se aplica a la copia de un almacén de datos basado en archivos a un almacén de datos relacional. Para obtener una lista de los almacenes de datos que se admiten como orígenes y receptores, consulte la tabla de [almacenes de datos admitidos](https://docs.microsoft.com/azure/data-factory/copy-activity-overview).

> [!NOTE]
>
> - Si no está familiarizado con Data Factory, consulte [Introducción a Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction).

En este tutorial, realizará los siguientes pasos:

> * Crear una factoría de datos
> * Creación de una canalización con una actividad de copia


## <a name="prerequisites"></a>Requisitos previos
* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**. Usará Blob Storage como almacén de datos de *origen*. Si no tiene una cuenta de almacenamiento, consulte [Crear una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-portal) para crear una. **Asegúrese de que la cuenta de almacenamiento solo permita el acceso desde las "Redes seleccionadas".** 
* **Azure SQL Database**. Usará la base de datos como un almacén de datos *receptor*. Si no tiene una base de datos de Azure SQL, consulte [Creación de una base de datos de Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal) para ver los pasos para crear una. **Asegúrese de que la cuenta de Azure SQL Database solo permite el acceso desde las "Redes seleccionadas".** 

### <a name="create-a-blob-and-a-sql-table"></a>Creación de un blob y una tabla SQL

Ahora, prepare su almacenamiento de blobs y su base de datos SQL para el tutorial mediante los pasos siguientes:

#### <a name="create-a-source-blob"></a>Creación de un blob de origen

1. Inicie el Bloc de notas. Copie el texto siguiente y guárdelo como un archivo **emp.txt** en el disco:

    ```
    FirstName,LastName
    John,Doe
    Jane,Doe
    ```

1. Cree un contenedor denominado **adftutorial** en su instancia de Blob Storage. Cree una carpeta denominada **input** en este contenedor. A continuación, cargue el archivo **emp.txt** en la carpeta **input**. Use Azure Portal o herramientas como [Explorador de Azure Storage](https://storageexplorer.com/) para realizar estas tareas.

#### <a name="create-a-sink-sql-table"></a>Creación de una tabla SQL receptora

1. Use el siguiente script de SQL para crear la tabla **dbo.emp** en la base de datos SQL:

    ```sql
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50)
    )
    GO

    CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID);
    ```

1. Permita que los servicios de Azure accedan a SQL Server. Asegúrese de que **Permitir el acceso a servicios de Azure** esté **Activado** para SQL Server de forma que Data Factory pueda escribir datos en su instancia de SQL Server. Para verificar y activar esta configuración, vaya a Azure SQL Server > Información general > Establecer el firewall del servidor > establezca la opción **Permitir el acceso a servicios de Azure** en **Activada**.

## <a name="create-a-data-factory"></a>Crear una factoría de datos
En este paso, creará una factoría de datos e iniciará la interfaz de usuario de Data Factory para crear una canalización en la factoría de datos.

1. Abra **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.


2. En el menú de la izquierda, seleccione **Crear un recurso** > **Analytics** > **Data Factory**.

3. En la página **Nueva factoría de datos**, en **Nombre**, escriba **ADFTutorialDataFactory**.

   El nombre de la instancia de Azure Data Factory debe ser *único de forma global*. Si recibe un mensaje de error sobre el valor de nombre, escriba un nombre diferente para la factoría de datos. (Por ejemplo, utilice SuNombreADFTutorialDataFactory). Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Azure Data Factory: reglas de nomenclatura](https://docs.microsoft.com/azure/data-factory/naming-rules).

4. Seleccione la **suscripción** de Azure en la que quiere crear la factoría de datos.

5. Para **Grupo de recursos**, realice uno de los siguientes pasos:

    a. Seleccione en primer lugar **Usar existente**y después un grupo de recursos de la lista desplegable.

    b. Seleccione **Crear nuevo**y escriba el nombre de un grupo de recursos. 
         
    Para más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/overview). 

6. En **Versión**, seleccione **V2**.

7. En **Ubicación**, seleccione la ubicación de la factoría de datos. En la lista desplegable solo se muestran las ubicaciones que se admiten. Los almacenes de datos (por ejemplo, Azure Storage y SQL Database) y los procesos (por ejemplo, Azure HDInsight) que usa la factoría de datos pueden estar en otras regiones.


8. Seleccione **Crear**.


9. Una vez finalizada la creación, verá el aviso en el centro de notificaciones. Seleccione **Ir al recurso** para ir a la página de Data Factory.

10. Haga clic en **Author & Monitor** (Creación y supervisión) para iniciar la interfaz de usuario de Data Factory en una pestaña independiente.

## <a name="create-an-azure-integration-runtime-in-adf-managed-virtual-network"></a>Creación de una instancia de Azure Integration Runtime en una red virtual administrada por ADF
En este paso se crea una instancia de Azure Integration Runtime y se habilita una red virtual administrada.

1. En el portal de ADF, vaya a **Manage Hub** (Administrar centro) y haga clic en **New** (Nuevo) para crear una nueva instancia de Azure Integration Runtime.
   ![Creación de una nueva instancia de Azure Integration Runtime](./media/tutorial-copy-data-portal-private/create-new-azure-ir.png)
2. Elija crear una instancia de Azure** Integration Runtime.
   ![Nueva instancia de Azure Integration Runtime](./media/tutorial-copy-data-portal-private/azure-ir.png)
3. Habilite **Virtual Network** (Red virtual).
   ![Nueva instancia de Azure Integration Runtime](./media/tutorial-copy-data-portal-private/enable-managed-vnet.png)
4. Seleccione **Crear**.

## <a name="create-a-pipeline"></a>Crear una canalización
En este paso, creará una canalización con una actividad de copia en la factoría de datos. La actividad de copia realiza la copia de los datos de Blob Storage a SQL Database. En el [tutorial de inicio rápido](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-portal),creó una canalización mediante estos pasos:

1. Creación del servicio vinculado.
1. Creación del conjunto de datos de entrada y salida.
1. Creación de una canalización

En este tutorial, comenzará a crear la canalización. A continuación, creará servicios vinculados y conjuntos de datos cuando los necesite para configurar la canalización.

1. En la página **Let's get started** (Introducción) seleccione **Create pipeline** (Crear canalización).

   ![Creación de una canalización](./media/doc-common-process/get-started-page.png)
1. En el panel **Properties** (Propiedades) de la canalización, escriba **CopyPipeline** como valor de **Name** (Nombre) de la canalización.

1. En el cuadro de herramientas **Activities** (Actividades), expanda la categoría **Move and Transform** (Mover y transformar) y arrastre y suelte la actividad **Copy Data** (Copiar datos) desde el cuadro de herramientas hasta la superficie de diseño de la canalización. Especifique **CopyFromBlobToSql** en **Name** (Nombre).

    ![Actividad de copia](./media/tutorial-copy-data-portal-private/drag-drop-copy-activity.png)

### <a name="configure-source"></a>Configuración del origen

>[!TIP]
>En este tutorial, usará *Clave de cuenta* como el tipo de autenticación para el almacén de datos de origen, pero puede elegir otros métodos de autenticación compatibles: *identificador URI de SAS*, *entidad de servicio* e *identidad administrada*, si es necesario. Consulte las secciones correspondientes en [este artículo](https://docs.microsoft.com/azure/data-factory/connector-azure-blob-storage#linked-service-properties) para más información.
>Para almacenar los secretos de los almacenes de datos de forma segura, también se recomienda usar Azure Key Vault. Consulte [este artículo](https://docs.microsoft.com/azure/data-factory/store-credentials-in-key-vault) para obtener instrucciones detalladas.

#### <a name="create-source-dataset-and-linked-service"></a>Creación de un conjunto de datos de origen y un servicio vinculado

1. Vaya a la pestaña **Source** (Origen). Haga clic en **+ New** (+ Nuevo) para crear un conjunto de datos de origen.

1. En el cuadro de diálogo **New Dataset** (Nuevo conjunto de datos), seleccione **Azure Blob Storage** y, después, seleccione **Continue** (Continuar). Los datos de origen están en Blob Storage, así que seleccionará **Azure Blob Storage** como conjunto de datos de origen.

1. En el cuadro de diálogo **Select Format** (Seleccionar formato), elija el tipo de formato de los datos y, después, seleccione **Continue** (Continuar).

1. En el cuadro de diálogo **Set Properties** (Establecer propiedades), escriba **SourceBlobDataset** como nombre. Active la casilla **First row as header** (Primera fila como encabezado). En el cuadro de texto **Linked service** (Servicio vinculado), seleccione **+ New** (+ Nuevo).

1. En la ventana **New Linked Service (Azure Blob Storage)** [Nuevo servicio vinculado (Azure Blob Storage)], escriba **AzureStorageLinkedService** como nombre y seleccione la cuenta de almacenamiento en la lista **Nombre de la cuenta de almacenamiento**. 

1. Asegúrese de habilitar **Interactive Authoring** (Creación interactiva). Puede tardar aproximadamente 1 minuto en habilitarse.

    ![Creación interactiva](./media/tutorial-copy-data-portal-private/interactive-authoring.png)

1. Seleccione **Test connection** (Probar conexión); se debería producir un error cuando la cuenta de almacenamiento solo permite el acceso desde la "Red seleccionada" y requiera que Azure Data Factory cree un punto de conexión privado que debe aprobarse antes de su uso. En el mensaje de error, verá un vínculo para crear un **punto de conexión privado** administrado. *Una alternativa es ir directamente a la pestaña Manage (Administrar) y seguir las instrucciones de la [siguiente sección](#create-a-managed-private-endpoint) para crear un punto de conexión privado administrado.*
> [!NOTE]
> Es posible que la pestaña Manage (Administrar) no esté disponible para todas las instancias de Data Factory. Si no la ve, también puede acceder a los puntos de conexión privados mediante la pestaña "**Author** > "**Connections** > "**Private Endpoint**" (Crear > Conexiones > Punto de conexión privado).
1. Mantenga abierto el cuadro de diálogo y, a continuación, vaya a la cuenta de almacenamiento seleccionada anteriormente.

1. Siga las instrucciones de [esta sección](#approval-of-a-private-link-in-storage-account) para aprobar el vínculo privado.

1. Vuelva al cuadro de diálogo. Seleccione de nuevo **Test connection** (Probar conexión) y seleccione **Create** (Crear) para implementar el servicio vinculado.

1. Una vez creado el servicio vinculado, se vuelve a ir a la página **Set Properties** (Establecer propiedades). Junto a **File path** (Ruta de acceso del archivo), seleccione **Browse** (Examinar).

1. Vaya a la carpeta **adftutorial/input**, seleccione el archivo **emp.txt** y, luego, **OK** (Aceptar).

1. Seleccione **Aceptar**. Va automáticamente a la página de canalización. En la pestaña **Source** (Origen), confirme que se selecciona **SourceBlobDataset**. Para obtener una vista previa de los datos de esta página, seleccione **Preview data** (Vista previa de los datos).

    ![Conjunto de datos de origen](./media/tutorial-copy-data-portal-private/source-dataset-selected.png)

#### <a name="create-a-managed-private-endpoint"></a>Creación de un punto de conexión privado administrado

En caso de no haber hecho clic en el hipervínculo al probar la conexión anterior, siga la ruta de acceso siguiente. Ahora debe crear un punto de conexión privado administrado que conectará al servicio vinculado creado anteriormente.

1. Vaya a la pestaña Manage (Administrar).
> [!NOTE]
> Es posible que la pestaña Manage (Administrar) no esté disponible para todas las instancias de Data Factory. Si no la ve, también puede acceder a los puntos de conexión privados mediante la pestaña "**Author** > "**Connections** > "**Private Endpoint**" (Crear > Conexiones > Punto de conexión privado).

1. Vaya a la sección Managed private endpoints (Puntos de conexión privados administrados).

1. Seleccione **+ New** (+ Nuevo) en Managed private endpoints (Puntos de conexión privados administrados).

    ![Nuevo punto de conexión privado administrado](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png) 

1. Seleccione el icono de Azure Blob Storage de la lista y seleccione **Continue** (Continuar).

1. Escriba el nombre de la cuenta de almacenamiento que creó anteriormente.

1. Seleccione **Crear**.

1. Después de esperar algunos segundos, debería ver que el vínculo privado creado necesita una aprobación.

1. Seleccione el punto de conexión privado que creó anteriormente. Puede ver un hipervínculo que le llevará a la página para aprobar el punto de conexión privado en el nivel de cuenta de almacenamiento.

    ![Administrar punto de conexión privado](./media/tutorial-copy-data-portal-private/manage-private-endpoint.png) 

#### <a name="approval-of-a-private-link-in-storage-account"></a>Aprobación de un vínculo privado en una cuenta de almacenamiento
1. En la cuenta de almacenamiento, vaya a **Conexiones de punto de conexión privado** en la sección Configuración.

1. Marque el punto de conexión privado que creó anteriormente y seleccione **Aprobar**.

    ![Aprobación de un punto de conexión privado](./media/tutorial-copy-data-portal-private/approve-private-endpoint.png)

1. Agregue una descripción y haga clic en **Sí**.
1. Vuelva a la sección **Managed private endpoints** (Puntos de conexión privados administrados) de la pestaña **Manage** (Administrar) de Azure Data Factory.
1. La aprobación del punto de conexión privado tardará aproximadamente de 1 a 2 minutos en verse reflejada en la interfaz de usuario de Azure Data Factory.


### <a name="configure-sink"></a>Configuración del receptor
>[!TIP]
>En este tutorial, usará *Autenticación de SQL* como el tipo de autenticación para el almacén de datos receptor, pero puede elegir otros métodos de autenticación compatibles: *entidad de servicio* e *identidad administrada*, si es necesario. Consulte las secciones correspondientes en [este artículo](https://docs.microsoft.com/azure/data-factory/connector-azure-sql-database#linked-service-properties) para más información.
>Para almacenar los secretos de los almacenes de datos de forma segura, también se recomienda usar Azure Key Vault. Consulte [este artículo](https://docs.microsoft.com/azure/data-factory/store-credentials-in-key-vault) para obtener instrucciones detalladas.

#### <a name="create-sink-dataset-and-linked-service"></a>Creación de un conjunto de datos receptor y un servicio vinculado
1. Vaya a la pestaña **Sink** (Receptor) y seleccione **+ New** (+Nuevo) para crear un conjunto de datos del receptor.

1. En el cuadro de diálogo **New Datase** (Nuevo conjunto de datos), escriba "SQL" en el cuadro de búsqueda para filtrar los conectores, seleccione **Azure SQL Database** y, después, seleccione **Continue** (Continuar). En este tutorial, copiará los datos en una base de datos SQL.

1. En el cuadro de diálogo **Set Properties** (Establecer propiedades), escriba **OutputSqlDataset** como nombre. En la lista desplegable **Linked service** (Servicio vinculado), seleccione **+ New** (+ Nuevo). Un conjunto de datos debe estar asociado con un servicio vinculado. El servicio vinculado tiene la cadena de conexión que usa Data Factory para conectarse a la base de datos SQL en tiempo de ejecución. El conjunto de datos especifica el contenedor, la carpeta y el archivo (opcional) donde se copian los datos.

1. En el cuadro de diálogo **New Linked Service (Azure SQL Database)** [Nuevo servicio vinculado (Azure SQL Database)], realice los siguientes pasos:

    1. En **Name** (Nombre), escriba **AzureSqlDatabaseLinkedService**.
    1. En **Server name** (Nombre del servidor), seleccione su instancia de SQL Server.
    1. Asegúrese de habilitar **Interactive Authoring** (Creación interactiva).
    1. En **Database name** (Nombre de la base de datos), seleccione la base de datos SQL.
    1. En **User name** (Nombre de usuario), escriba el nombre del usuario.
    1. En **Password** (Contraseña), escriba la contraseña del usuario.
    1. Seleccione **Test connection** (Probar conexión). Debería producirse un error porque SQL Server solo permite el acceso desde las "redes seleccionadas" y requiere que Azure Data Factory cree un punto de conexión privado, que debe aprobarse antes de su uso. En el mensaje de error, verá un vínculo para crear un **punto de conexión privado** administrado. *Una alternativa es ir directamente a la pestaña Manage (Administrar) y seguir las instrucciones de la siguiente sección para crear un punto de conexión privado administrado.*
    1. Mantenga abierto el cuadro de diálogo y, a continuación, vaya a la instancia de SQL Server seleccionada anteriormente.    
    1. Siga las instrucciones de [esta sección](#approval-of-a-private-link-in-sql-server) para aprobar el vínculo privado.
    1. Vuelva al cuadro de diálogo. Seleccione de nuevo **Test connection** (Probar conexión) y seleccione **Create** (Crear) para implementar el servicio vinculado.

1. Va automáticamente al cuadro de diálogo **Set Properties** (Establecer propiedades). En **Table** (Tabla), seleccione **[dbo].[emp]** . Después, seleccione **Aceptar**.

1. Vaya a la pestaña con la canalización y, en **Sink Dataset** (Conjunto de datos del receptor), confirme que se ha seleccionado **OutputSqlDataset**.

    ![Pestaña Pipeline (Canalización)](./media/tutorial-copy-data-portal-private/pipeline-tab-2.png)       

Opcionalmente, puede asignar el esquema del origen al correspondiente esquema de destino. Para ello, siga las instrucciones de [Asignación de esquemas en la actividad de copia](https://docs.microsoft.com/azure/data-factory/copy-activity-schema-and-type-mapping).

#### <a name="create-a-managed-private-endpoint"></a>Creación de un punto de conexión privado administrado

En caso de no haber hecho clic en el hipervínculo al probar la conexión anterior, siga la ruta de acceso siguiente. Ahora debe crear un punto de conexión privado administrado que conectará al servicio vinculado creado anteriormente.

1. Vaya a la pestaña Manage (Administrar).
1. Vaya a la sección Managed private endpoints (Puntos de conexión privados administrados).
1. Seleccione **+ New** (+ Nuevo) en Managed private endpoints (Puntos de conexión privados administrados).

    ![Nuevo punto de conexión privado administrado](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png) 

1. Seleccione el icono de Azure SQL Database de la lista y seleccione **Continue** (Continuar).
1. Escriba el nombre de la instancia de SQL Server que seleccionó anteriormente.
1. Seleccione **Crear**.
1. Después de esperar algunos segundos, debería ver que el vínculo privado creado necesita una aprobación.
1. Seleccione el punto de conexión privado que creó anteriormente. Puede ver un hipervínculo que le llevará a la página para aprobar el punto de conexión privado en el nivel de instancia de SQL Server.


#### <a name="approval-of-a-private-link-in-sql-server"></a>Aprobación de un vínculo privado en SQL Server
1. En la instancia de SQL Server, vaya a **Conexiones de punto de conexión privado** en la sección Configuración.
1. Marque el punto de conexión privado que creó anteriormente y seleccione **Aprobar**.
1. Agregue una descripción y haga clic en **Sí**.
1. Vuelva a la sección **Managed private endpoints** (Puntos de conexión privados administrados) de la pestaña **Manage** (Administrar) de Azure Data Factory.
1. La aprobación tardará aproximadamente de 1 a 2 minutos en verse reflejada para el punto de conexión privado.

#### <a name="debug-and-publish-the-pipeline"></a>Depuración y publicación de la canalización

Puede depurar una canalización antes de publicar artefactos (servicios vinculados, conjuntos de datos y canalizaciones) en Data Factory o en su propio repositorio Git de Azure Repos.

1. Para depurar la canalización, seleccione **Depurar** en la barra de herramientas. Verá el estado de ejecución de la canalización en la pestaña **Output** (Salida) en la parte inferior de la ventana.
2. Una vez que la canalización se puede ejecutar correctamente, en la barra de herramientas superior, seleccione **Publish all** (Publicar todo). Esta acción publica las entidades (conjuntos de datos y canalizaciones) que creó para Data Factory.
3. Espere a que aparezca el mensaje **Successfully published** (Publicado correctamente). Para ver los mensajes de notificación, haga clic en **Show Notifications** (Mostrar notificaciones) en la parte superior derecha (botón de campana).


#### <a name="summary"></a>Resumen
La canalización de este ejemplo copia datos de Blob Storage a Azure SQL DB mediante un punto de conexión privado en una red virtual administrada. Ha aprendido a:

> * Crear una factoría de datos
> * Creación de una canalización con una actividad de copia

