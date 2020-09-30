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
ms.openlocfilehash: 953dae3c264e76b1e40f0dc07ccea0c00a7464c8
ms.sourcegitcommit: 70ee014d1706e903b7d1e346ba866f5e08b22761
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90024424"
---
# <a name="copy-data-securely-from-azure-blob-storage-to-a-sql-database-by-using-private-endpoints"></a>Copia de datos de forma segura desde Azure Blob Storage a SQL Database mediante puntos de conexión privados

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial, creará una factoría de datos mediante la interfaz de usuario (UI) de Azure Data Factory. *La canalización de esta factoría de datos copia los datos de forma segura desde Azure Blob Storage a una base de datos de Azure SQL (en ambas se permite el acceso solo a las redes seleccionadas) mediante puntos de conexión privados en una [red virtual administrada por Azure Data Factory](managed-virtual-network-private-endpoint.md).* El patrón de configuración de este tutorial se aplica a la copia de un almacén de datos basado en archivos a un almacén de datos relacional. Para obtener una lista de los almacenes de datos que se admiten como orígenes y receptores, consulte la tabla [Almacenes de datos y formatos que se admiten](https://docs.microsoft.com/azure/data-factory/copy-activity-overview).

> [!NOTE]
> Si no está familiarizado con Data Factory, consulte [Introducción a Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction).

En este tutorial, realizará los siguientes pasos:

* Creación de una factoría de datos.
* Creación de una canalización con una actividad de copia.


## <a name="prerequisites"></a>Requisitos previos
* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**. Usará Blob Storage como almacén de datos de *origen*. Si no tiene una cuenta de almacenamiento, consulte [Crear una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-portal) para crear una. *Asegúrese de que la cuenta de almacenamiento solo permita el acceso desde las redes seleccionadas.* 
* **Azure SQL Database**. Usará la base de datos como un almacén de datos *receptor*. Si no tiene una base de datos de Azure SQL, consulte [Creación de una base de datos de Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal) para ver los pasos para crear una. *Asegúrese de que la cuenta de SQL Database solo permite el acceso desde las redes seleccionadas.* 

### <a name="create-a-blob-and-a-sql-table"></a>Creación de un blob y una tabla SQL

Ahora, prepare su instancia de Blob Storage y su base de datos SQL para el tutorial mediante los pasos siguientes.

#### <a name="create-a-source-blob"></a>Creación de un blob de origen

1. Abra Bloc de notas. Copie el texto siguiente y guárdelo como un archivo **emp.txt** en el disco:

    ```
    FirstName,LastName
    John,Doe
    Jane,Doe
    ```

1. Cree un contenedor denominado **adftutorial** en su instancia de Blob Storage. Cree una carpeta denominada **input** en este contenedor. A continuación, cargue el archivo **emp.txt** en la carpeta **input**. Use Azure Portal o herramientas como [Explorador de Azure Storage](https://storageexplorer.com/) para realizar estas tareas.

#### <a name="create-a-sink-sql-table"></a>Creación de una tabla SQL receptora

Use el siguiente script de SQL para crear la tabla **dbo.emp** en la base de datos SQL:

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

## <a name="create-a-data-factory"></a>Crear una factoría de datos
En este paso, creará una factoría de datos e iniciará la interfaz de usuario de Data Factory para crear una canalización en la factoría de datos.

1. Abra Microsoft Edge o Google Chrome. Actualmente, solo los exploradores web Microsoft Edge y Google Chrome admiten la interfaz de usuario de Data Factory.

1. En el menú de la izquierda, seleccione **Crear un recurso** > **Analytics** > **Data Factory**.

1. En la página **Nueva factoría de datos**, en **Nombre**, escriba **ADFTutorialDataFactory**.

   El nombre de la instancia de Azure Data Factory debe ser *único de forma global*. Si recibe un mensaje de error sobre el valor de nombre, escriba un nombre diferente para la factoría de datos (por ejemplo, yournameADFTutorialDataFactory). Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Azure Data Factory: reglas de nomenclatura](https://docs.microsoft.com/azure/data-factory/naming-rules).

1. Seleccione la **suscripción** de Azure en la que quiere crear la factoría de datos.

1. Para **Grupo de recursos**, realice uno de los siguientes pasos:

    - Seleccione en primer lugar **Usar existente**y después un grupo de recursos de la lista desplegable.
    - Seleccione **Crear nuevo**y escriba el nombre de un grupo de recursos. 
     
    Para más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/overview). 

1. En **Versión**, seleccione **V2**.

1. En **Ubicación**, seleccione la ubicación de la factoría de datos. En la lista desplegable solo aparecen las ubicaciones que se admiten. Los almacenes de datos (por ejemplo, Azure Storage y SQL Database) y los procesos (por ejemplo, Azure HDInsight) que usa la factoría de datos pueden estar en otras regiones.

1. Seleccione **Crear**.

1. Una vez finalizada la creación, verá el aviso en el centro de notificaciones. Seleccione **Ir al recurso** para ir a la página de **Data Factory**.

1. Haga clic en **Author & Monitor** (Creación y supervisión) para iniciar la interfaz de usuario de Data Factory en una pestaña independiente.

## <a name="create-an-azure-integration-runtime-in-data-factory-managed-virtual-network"></a>Creación de una instancia de Azure Integration Runtime en una red virtual administrada por Data Factory
En este paso se crea una instancia de Azure Integration Runtime y se habilita una red virtual administrada por Data Factory.

1. En el portal de Data Factory, vaya a **Administrar** y seleccione **Nuevo** para crear una instancia nueva de Azure Integration Runtime.

   ![Captura de pantalla que muestra la creación de una nueva instancia de Azure Integration Runtime.](./media/tutorial-copy-data-portal-private/create-new-azure-ir.png)
1. Elija crear un entorno de ejecución de integración de **Azure**.

   ![Captura de pantalla que muestra una nueva instancia de Azure Integration Runtime.](./media/tutorial-copy-data-portal-private/azure-ir.png)
1. En **Configuración de red virtual (versión preliminar)** , seleccione **Habilitar**.

   ![Captura de pantalla que muestra la habilitación de una nueva instancia de Azure Integration Runtime.](./media/tutorial-copy-data-portal-private/enable-managed-vnet.png)
1. Seleccione **Crear**.

## <a name="create-a-pipeline"></a>Crear una canalización
En este paso, creará una canalización con una actividad de copia en la factoría de datos. La actividad de copia realiza la copia de los datos de Blob Storage a SQL Database. En el [tutorial de inicio rápido](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-portal),creó una canalización mediante estos pasos:

1. Creación del servicio vinculado.
1. Creación del conjunto de datos de entrada y salida.
1. Creación de una canalización

En este tutorial, comenzará por crear una canalización. A continuación, creará servicios vinculados y conjuntos de datos cuando los necesite para configurar la canalización.

1. En la página **Let's get started** (Introducción) seleccione **Create pipeline** (Crear canalización).

   ![Captura de pantalla que muestra la creación de una canalización.](./media/doc-common-process/get-started-page.png)
1. En el panel de propiedades de la canalización, escriba **CopyPipeline** como nombre de la canalización.

1. En el cuadro de herramientas **Actividades**, expanda la categoría **Mover y transformar** y arrastre la actividad **Copiar datos** desde el cuadro de herramientas hasta la superficie de diseño de la canalización. Especifique **CopyFromBlobToSql** como nombre.

    ![Captura de pantalla que muestra la actividad de copia.](./media/tutorial-copy-data-portal-private/drag-drop-copy-activity.png)

### <a name="configure-a-source"></a>Configuración de un origen de datos

>[!TIP]
>En este tutorial, usará **Clave de cuenta** como el tipo de autenticación para el almacén de datos de origen. También puede elegir otros métodos de autenticación admitidos, como **URI de SAS**,**Entidad de servicio** e **Identidad administrada**, si es necesario. Para obtener más información, consulte las secciones correspondientes en [Copia y transformación de datos en Azure Blob Storage mediante Azure Data Factory](https://docs.microsoft.com/azure/data-factory/connector-azure-blob-storage#linked-service-properties).
>
>Para almacenar los secretos de los almacenes de datos de forma segura, también se recomienda usar su instancia de Azure Key Vault. Para obtener más información e ilustraciones, consulte [Almacenamiento de credenciales en Azure Key Vault](https://docs.microsoft.com/azure/data-factory/store-credentials-in-key-vault).

#### <a name="create-a-source-dataset-and-linked-service"></a>Creación de un conjunto de datos de origen y un servicio vinculado

1. Vaya a la pestaña **Source** (Origen). Haga clic en **+ New** (+ Nuevo) para crear un conjunto de datos de origen.

1. En el cuadro de diálogo **New Dataset** (Nuevo conjunto de datos), seleccione **Azure Blob Storage** y, después, seleccione **Continue** (Continuar). Los datos de origen están en Blob Storage, así que seleccionará **Azure Blob Storage** como conjunto de datos de origen.

1. En el cuadro de diálogo **Seleccionar formato**, elija el tipo de formato de los datos y, después, seleccione **Continuar**.

1. En el cuadro de diálogo **Establecer propiedades**, escriba **SourceBlobDataset** como **Nombre**. Active la casilla **First row as header** (Primera fila como encabezado). En el cuadro de texto **Linked service** (Servicio vinculado), seleccione **+ New** (+ Nuevo).

1. En el cuadro de diálogo **New linked service (Azure Blob Storage)** [Nuevo servicio vinculado (Azure Blob Storage)], escriba **AzureStorageLinkedService** como **Nombre** y seleccione su cuenta de almacenamiento en la lista **Nombre de la cuenta de almacenamiento**. 

1. Asegúrese de habilitar **Interactive Authoring** (Creación interactiva). Puede tardar aproximadamente un minuto en habilitarse.

    ![Captura de pantalla que muestra la creación interactiva.](./media/tutorial-copy-data-portal-private/interactive-authoring.png)

1. Seleccione **Test connection** (Probar conexión). Se debería producir un error cuando la cuenta de almacenamiento solo permite el acceso desde las **Redes seleccionadas** y requiere que Data Factory cree un punto de conexión privado que debe aprobarse antes de su uso. En el mensaje de error, debería ver un vínculo para crear un punto de conexión privado que puede usar para crear un punto de conexión privado administrado. Una alternativa es ir directamente a la pestaña **Administrar** y seguir las instrucciones de la [siguiente sección](#create-a-managed-private-endpoint) para crear un punto de conexión privado administrado.

   > [!NOTE]
   > Es posible que dicha pestaña **Administrar** no esté disponible para todas las instancias de Data Factory. Si no se muestra, puede tener acceso a los puntos de conexión privados al seleccionar**Autor** > **Conexiones** > **Punto de conexión privado**.
1. Mantenga abierto el cuadro de diálogo y, después, vaya a la cuenta de almacenamiento.

1. Siga las instrucciones de [esta sección](#approval-of-a-private-link-in-a-storage-account) para aprobar el vínculo privado.

1. Vuelva al cuadro de diálogo. Seleccione de nuevo **Probar conexión** y seleccione **Crear** para implementar el servicio vinculado.

1. Una vez creado el servicio vinculado, se dirige a la página **Establecer propiedades**. Junto a **File path** (Ruta de acceso del archivo), seleccione **Browse** (Examinar).

1. Vaya a la carpeta **adftutorial/input**, seleccione el archivo **emp.txt** y, luego, **Finalizar**.

1. Seleccione **Aceptar**. Se dirige automáticamente a la página de canalización. En la pestaña **Origen**, confirme que se selecciona **SourceBlobDataset**. Para obtener una vista previa de los datos de esta página, seleccione **Preview data** (Vista previa de los datos).

    ![Captura de pantalla que muestra el panel Conjunto de datos de origen.](./media/tutorial-copy-data-portal-private/source-dataset-selected.png)

#### <a name="create-a-managed-private-endpoint"></a>Creación de un punto de conexión privado administrado

Si no seleccionó el hipervínculo al probar la conexión, siga la ruta de acceso. Ahora debe crear un punto de conexión privado administrado que conectará al servicio vinculado que creó anteriormente.

1. Vaya a la pestaña **Administrar**.

   > [!NOTE]
   > Es posible que la pestaña **Administrar** no esté disponible para todas las instancias de Data Factory. Si no se muestra, puede tener acceso a los puntos de conexión privados al seleccionar **Autor** > **Conexiones** > **Punto de conexión privado**.

1. Vaya a la sección **Puntos de conexión privados administrados**.

1. Seleccione **+ Nuevo** debajo de **Puntos de conexión privados administrados**.

    ![Captura de pantalla que muestra el botón Nuevo debajo de los Puntos de conexión privados administrados.](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png) 

1. Seleccione el icono de **Azure Blob Storage** de la lista y seleccione **Continuar**.

1. Escriba el nombre de la cuenta de almacenamiento que ha creado.

1. Seleccione **Crear**.

1. Después de unos segundos, debería ver que el vínculo privado creado necesita aprobación.

1. Seleccione el punto de conexión privado que creó. Puede ver un hipervínculo que le llevará a la página para aprobar el punto de conexión privado en el nivel de cuenta de almacenamiento.

    ![Captura de pantalla que muestra el panel Punto de conexión privado administrado.](./media/tutorial-copy-data-portal-private/manage-private-endpoint.png) 

#### <a name="approval-of-a-private-link-in-a-storage-account"></a>Aprobación de un vínculo privado en una cuenta de almacenamiento
1. En la cuenta de almacenamiento, vaya a **Conexiones de punto de conexión privado** en la sección **Configuración**.

1. Seleccione la casilla del punto de conexión privado que creó y seleccione **Aprobar**.

    ![Captura de pantalla que muestra el botón Aprobar para el punto de conexión privado.](./media/tutorial-copy-data-portal-private/approve-private-endpoint.png)

1. Agregue una descripción y seleccione **Sí**.
1. Vuelva a la sección **Puntos de conexión privados administrados** de la pestaña **Administrar** de Data Factory.
1. Tras aproximadamente uno o dos minutos, debería asegurarse de que la aprobación del punto de conexión privado aparezca en la interfaz de usuario de Data Factory.


### <a name="configure-a-sink"></a>Configuración de un receptor
>[!TIP]
>En este tutorial, usará **Autenticación de SQL** como el tipo de autenticación para el almacén de datos del receptor. También puede elegir otros métodos de autenticación admitidos, como **Entidad de servicio** e **Identidad administrada**, si es necesario. Para obtener más información, consulte las secciones correspondientes en [Copia y transformación de datos en Azure SQL Database mediante Azure Data Factory](https://docs.microsoft.com/azure/data-factory/connector-azure-sql-database#linked-service-properties).
>
>Para almacenar los secretos de los almacenes de datos de forma segura, también se recomienda usar su instancia de Azure Key Vault. Para obtener más información e ilustraciones, consulte [Almacenamiento de credenciales en Azure Key Vault](https://docs.microsoft.com/azure/data-factory/store-credentials-in-key-vault).

#### <a name="create-a-sink-dataset-and-linked-service"></a>Creación de un conjunto de datos receptor y un servicio vinculado
1. Vaya a la pestaña **Sink** (Receptor) y seleccione **+ New** (+Nuevo) para crear un conjunto de datos del receptor.

1. En el cuadro de diálogo **Nuevo conjunto de datos**, escriba **SQL** en el cuadro de búsqueda para filtrar los conectores. Seleccione **Azure SQL Database** y luego **Continuar**. En este tutorial, copiará los datos en una base de datos SQL.

1. En el cuadro de diálogo **Establecer propiedades**, escriba **OutputSqlDataset** como **Nombre**. En la lista desplegable **Servicio vinculado**, seleccione **+ Nuevo**. Un conjunto de datos debe estar asociado con un servicio vinculado. El servicio vinculado tiene la cadena de conexión que usa Data Factory para conectarse a la base de datos SQL en tiempo de ejecución. El conjunto de datos especifica el contenedor, la carpeta y el archivo (opcional) donde se copian los datos.

1. En el cuadro de diálogo **New linked service (Azure SQL Database)** [Nuevo servicio vinculado (Azure SQL Database)], realice los siguientes pasos:

    1. En **Name** (Nombre), escriba **AzureSqlDatabaseLinkedService**.
    1. En **Server name** (Nombre del servidor), seleccione su instancia de SQL Server.
    1. Asegúrese de habilitar **Interactive Authoring** (Creación interactiva).
    1. En **Database name** (Nombre de la base de datos), seleccione la base de datos SQL.
    1. En **User name** (Nombre de usuario), escriba el nombre del usuario.
    1. En **Password** (Contraseña), escriba la contraseña del usuario.
    1. Seleccione **Test connection** (Probar conexión). Debería producirse un error porque SQL Server solo permite el acceso desde las **Redes seleccionadas** y requiere que Azure Data Factory cree un punto de conexión privado, que debe aprobarse antes de su uso. En el mensaje de error, debería ver un vínculo para crear un punto de conexión privado que puede usar para crear un punto de conexión privado administrado. Una alternativa es ir directamente a la pestaña **Administrar** y seguir las instrucciones de la siguiente sección para crear un punto de conexión privado administrado.
    1. Mantenga abierto el cuadro de diálogo y, a continuación, vaya a la instancia de SQL Server seleccionada.
    1. Siga las instrucciones de [esta sección](#approval-of-a-private-link-in-sql-server) para aprobar el vínculo privado.
    1. Vuelva al cuadro de diálogo. Seleccione de nuevo **Probar conexión** y seleccione **Crear** para implementar el servicio vinculado.

1. Se dirigirá automáticamente al cuadro de diálogo **Establecer propiedades**. En **Table** (Tabla), seleccione **[dbo].[emp]** . Después, seleccione **Aceptar**.

1. Vaya a la pestaña con la canalización y, en **Sink dataset** (Conjunto de datos del receptor), confirme que se ha seleccionado **OutputSqlDataset**.

    ![Captura de pantalla que muestra la pestaña Canalización.](./media/tutorial-copy-data-portal-private/pipeline-tab-2.png)

Opcionalmente, puede asignar el esquema del origen al esquema de destino correspondiente. Para ello, siga las instrucciones de [Asignación de esquemas en la actividad de copia](https://docs.microsoft.com/azure/data-factory/copy-activity-schema-and-type-mapping).

#### <a name="create-a-managed-private-endpoint"></a>Creación de un punto de conexión privado administrado

Si no seleccionó el hipervínculo al probar la conexión, siga la ruta de acceso. Ahora debe crear un punto de conexión privado administrado que conectará al servicio vinculado que creó anteriormente.

1. Vaya a la pestaña **Administrar**.
1. Vaya a la sección **Puntos de conexión privados administrados**.
1. Seleccione **+ Nuevo** debajo de **Puntos de conexión privados administrados**.

    ![Captura de pantalla que muestra el botón Nuevo debajo de los Puntos de conexión privados administrados.](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png) 

1. Seleccione el icono de **Azure SQL Database** de la lista y seleccione **Continuar**.
1. Escriba el nombre de la instancia de SQL Server que seleccionó.
1. Seleccione **Crear**.
1. Después de unos segundos, debería ver que el vínculo privado creado necesita aprobación.
1. Seleccione el punto de conexión privado que creó. Puede ver un hipervínculo que le llevará a la página para aprobar el punto de conexión privado en el nivel de instancia de SQL Server.


#### <a name="approval-of-a-private-link-in-sql-server"></a>Aprobación de un vínculo privado en SQL Server
1. En la instancia de SQL Server, vaya a **Conexiones de punto de conexión privado** en la sección **Configuración**.
1. Seleccione la casilla del punto de conexión privado que creó y seleccione **Aprobar**.
1. Agregue una descripción y seleccione **Sí**.
1. Vuelva a la sección **Puntos de conexión privados administrados** de la pestaña **Administrar** de Data Factory.
1. La aprobación para el punto de conexión privado debe tardar uno o dos minutos en aparecer.

#### <a name="debug-and-publish-the-pipeline"></a>Depuración y publicación de la canalización

Puede depurar una canalización antes de publicar artefactos (servicios vinculados, conjuntos de datos y canalizaciones) en Data Factory o en su propio repositorio Git de Azure Repos.

1. Para depurar la canalización, seleccione **Depurar** en la barra de herramientas. Verá el estado de ejecución de la canalización en la pestaña **Output** (Salida) en la parte inferior de la ventana.
1. Después de que la canalización se puede ejecutar correctamente, en la barra de herramientas superior, seleccione **Publicar todo**. Esta acción publica las entidades (conjuntos de datos y canalizaciones) que creó para Data Factory.
1. Espere a que aparezca el mensaje **Successfully published** (Publicado correctamente). Para ver los mensajes de notificación, seleccione **Mostrar notificaciones** en la esquina superior derecha (botón de campana).


#### <a name="summary"></a>Resumen
La canalización de este ejemplo copia datos de Blob Storage a SQL Database mediante puntos de conexión privados en una red virtual administrada por Data Factory. Ha aprendido a:

* Creación de una factoría de datos.
* Creación de una canalización con una actividad de copia.

