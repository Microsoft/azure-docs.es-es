---
title: Uso compartido y recepción de datos de Azure SQL Database y Azure Synapse Analytics
description: Aprenda a compartir y recibir datos de Azure SQL Database y Azure Synapse Analytics
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 08/28/2020
ms.openlocfilehash: e813921727ee08bf9a76c0a2dbfe15f45fe4db79
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89490078"
---
# <a name="share-and-receive-data-from-azure-sql-database-and-azure-synapse-analytics"></a>Uso compartido y recepción de datos de Azure SQL Database y Azure Synapse Analytics

[!INCLUDE[appliesto-sql](includes/appliesto-sql.md)]

Azure Data Share admite el uso compartido basado en instantáneas de Azure SQL Database y Azure Synapse Analytics (anteriormente Azure SQL DW). En este artículo se explica cómo compartir y recibir datos desde estos orígenes.

Azure Data Share admite el uso compartido de tablas o vistas desde Azure SQL Database y Azure Synapse Analytics (anteriormente Azure SQL DW). Los consumidores de datos pueden elegir aceptar los datos en Azure Data Lake Storage Gen2 o Azure Blob Storage como archivo CSV o Parquet, así como en Azure SQL Database y Azure Synapse Analytics como tablas.

Al aceptar los datos en Azure Data Lake Store Gen2 o Azure Blob Storage, las instantáneas completas sobrescriben el contenido del archivo de destino si este ya existe.
Cuando se reciben los datos en la tabla y si la tabla de destino no existe todavía, Azure Data Share crea la tabla SQL con el esquema de origen. Si ya existe una tabla de destino con el mismo nombre, se quitará y se sobrescribirá con la instantánea completa más reciente. Actualmente no se admiten instantáneas incrementales.

## <a name="share-data"></a>Compartir datos

### <a name="prerequisites-to-share-data"></a>Requisitos previos para compartir datos

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* La dirección de correo electrónico de inicio de sesión de Azure del destinatario (con el alias de correo electrónico no funcionará).
* Si el almacén de datos de Azure de origen está en una suscripción de Azure diferente a la que va a usar para crear el recurso de Data Share, registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción en la que se encuentra el almacén de datos de Azure. 

### <a name="prerequisites-for-sql-source"></a>Requisitos previos para un origen de SQL

* Una instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente SQL Data Warehouse) con las tablas y vistas que desea compartir.
* Permisos para escribir en las bases de datos de SQL Server, que se encuentra en *Microsoft.Sql/servers/databases/write*. Este permiso existe en el rol de colaborador.
* Permiso para que el recurso compartido acceda al almacenamiento de datos. Esto se puede hacer mediante los siguientes pasos: 
    1. Establézcase como administrador de Azure Active Directory para SQL Server.
    1. Conéctese a Azure SQL Database o Data Warehouse mediante Azure Active Directory.
    1. Use el editor de consultas (versión preliminar) para ejecutar el siguiente script para agregar la identidad administrada del recurso de Data Share como db_datareader. Debe conectarse mediante Active Directory y no con la autenticación de SQL Server. 
    
        ```sql
        create user "<share_acct_name>" from external provider;     
        exec sp_addrolemember db_datareader, "<share_acct_name>"; 
        ```                   
       Tenga en cuenta que *<share_acc_name>* es el nombre del recurso de Data Share. Si aún no ha creado un recurso de Data Share, puede volver a este requisito previo más adelante.  

* Un usuario de Azure SQL Database con acceso "db_datareader" para explorar y seleccionar las tablas o vistas que desea compartir. 

* Acceso al firewall de SQL Server desde la dirección IP del cliente. Esto se puede hacer mediante los siguientes pasos: 
    1. En SQL Server en Azure Portal, vaya a *Firewalls y redes virtuales*.
    1. Haga clic en la alternancia **activado** para permitir el acceso a los servicios de Azure.
    1. Haga clic en **+Agregar dirección IP de cliente** y haga clic en **Guardar**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que comparta datos de SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP. 

### <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

### <a name="create-a-data-share-account"></a>Creación de una cuenta de Azure Data Share

Cree un recurso de Azure Data Share en un grupo de recursos de Azure.

1. Seleccione el botón de menú de la esquina superior izquierda del portal y, a continuación, seleccione **Crear un recurso**.

1. Busque *Data Share*.

1. Seleccione Data Share y, después, **Crear**.

1. Rellene los detalles básicos del recurso de Azure Data Share con la siguiente información. 

     **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | Subscription | Su suscripción | Seleccione la suscripción de Azure que desea usar para la cuenta del recurso compartido de datos.|
    | Grupo de recursos | *test-resource-group* | Use un grupo de recursos existente o cree uno. |
    | Location | *Este de EE. UU. 2* | Seleccione una región para la cuenta de Azure Data Share.
    | Nombre | *datashareaccount* | Especifique un nombre para la cuenta del recurso compartido de datos. |
    | | |

1. Seleccione **Revisar + Crear** y, después, **Crear** para aprovisionar la cuenta del recurso compartido de datos. El aprovisionamiento de una nueva cuenta de Azure Data Share normalmente tarda aproximadamente 2 minutos, o menos. 

1. Cuando la implementación se complete, seleccione **Ir al grupo de recursos**.

### <a name="create-a-share"></a>Creación de un recurso compartido

1. Vaya a la página de información general de Azure Data Share.

    ![Uso compartido de los datos](./media/share-receive-data.png "Uso compartido de los datos") 

1. Seleccione **Start sharing your data** (Empezar a compartir datos).

1. Seleccione **Crear**.   

1. Rellene los detalles del recurso compartido. Especifique un nombre, un tipo de recurso compartido, una descripción del contenido de los recursos compartidos y las condiciones de uso (opcional). 

    ![EnterShareDetails](./media/enter-share-details.png "Introducción de detalles del recurso compartido") 

1. Seleccione **Continuar**.

1. Para agregar conjuntos de datos al recurso compartido, seleccione **Agregar conjuntos de datos**. 

    ![Adición de conjuntos de datos al recurso compartido](./media/datasets.png "Conjuntos de datos")

1. Seleccione el tipo de conjunto de datos que desea agregar. Verá una lista diferente de tipos de conjuntos de datos en función del tipo de recurso compartido (instantánea o en contexto) que haya seleccionado en el paso anterior. 

    ![AddDatasets](./media/add-datasets.png "Incorporación de conjuntos de datos")    

1. Seleccione el servidor SQL Server, proporcione credenciales y seleccione **Siguiente** para ir al objeto que desea compartir y seleccione "Agregar conjuntos de datos". 

    ![SelectDatasets](./media/select-datasets-sql.png "Selección de conjuntos de datos")    

1. En la pestaña Destinatarios, escriba las direcciones de correo electrónico del consumidor de datos, para lo que debe seleccionar "+ Add Recipient" (+ Agregar a destinatario). 

    ![AddRecipients](./media/add-recipient.png "Adición de destinatarios") 

1. Seleccione **Continuar**.

1. Si ha seleccionado el tipo de recurso compartido de instantáneas, puede configurar la programación de instantáneas para proporcionar actualizaciones de los datos al consumidor de datos. 

    ![EnableSnapshots](./media/enable-snapshots.png "Habilitación de instantáneas") 

1. Seleccione una hora de inicio y un intervalo de periodicidad. 

1. Seleccione **Continuar**.

1. En la pestaña Revisar y crear, examine el contenido del paquete, la configuración, los destinatarios y la configuración de la sincronización. Seleccione **Crear**.

Ya se ha creado Azure Data Share y el destinatario de Data Share ya está listo para aceptar la invitación. 

## <a name="receive-data"></a>Recibir datos

### <a name="prerequisites-to-receive-data"></a>Requisitos previos para recibir datos
Para poder aceptar una invitación a compartir datos, previamente debe aprovisionar un número de recursos de Azure, que se enumeran a continuación. 

Asegúrese de que se cumplen todos los requisitos previos antes de aceptar una invitación para compartir datos. 

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una invitación de Azure Data Share: Una invitación de Microsoft Azure con el asunto "Azure Data Share invitation from **<yourdataprovider@domain.com>** " (Invitación para Azure Data Share de yourdataprovider@domain.com).
* Registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción de Azure en la que va a crear un recurso de Data Share y la suscripción de Azure en la que se encuentran los almacenes de datos de Azure de destino.

### <a name="prerequisites-for-target-storage-account"></a>Requisitos previos de la cuenta de almacenamiento de destino
Si elige recibir datos en Azure Storage, a continuación se muestra la lista de requisitos previos.

* Una cuenta de Azure Storage: si no la tiene, puede crear una cuenta de [Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account). 
* Permisos para escribir en la cuenta de almacenamiento, que se encuentra en *Microsoft.Storage/storageAccounts/write*. Este permiso existe en el rol de colaborador. 
* Permisos para agregar la asignación de roles a la cuenta de almacenamiento, que se encuentra en *Microsoft.Authorization/role assignments/write*. Este permiso existe en el rol de propietario.  

### <a name="prerequisites-for-sql-target"></a>Requisitos previos para un destino de SQL
Si opta por recibir datos en Azure SQL Database, Azure Synapse Analytics, esta es la lista de requisitos previos.

* Permisos para escribir en las bases de datos de SQL Server, que se encuentra en *Microsoft.Sql/servers/databases/write*. Este permiso existe en el rol de colaborador. 
* Permisos para que la identidad administrada del recurso compartido de datos acceda a Azure SQL Database o Azure Synapse Analytics. Esto se puede hacer mediante los siguientes pasos: 
    1. Establézcase como administrador de Azure Active Directory para SQL Server.
    1. Conéctese a Azure SQL Database o Data Warehouse mediante Azure Active Directory.
    1. Use el editor de consultas (versión preliminar) para ejecutar el siguiente script para agregar la identidad administrada de Data Share como "db_datareader, db_datawriter, db_ddladmin". Debe conectarse mediante Active Directory y no con la autenticación de SQL Server. 

        ```sql
        create user "<share_acc_name>" from external provider; 
        exec sp_addrolemember db_datareader, "<share_acc_name>"; 
        exec sp_addrolemember db_datawriter, "<share_acc_name>"; 
        exec sp_addrolemember db_ddladmin, "<share_acc_name>";
        ```      
        Tenga en cuenta que *<share_acc_name>* es el nombre del recurso de Data Share. Si aún no ha creado un recurso de Data Share, puede volver a este requisito previo más adelante.         

* Acceso al firewall de SQL Server desde la dirección IP del cliente. Esto se puede hacer mediante los siguientes pasos: 
    1. En SQL Server en Azure Portal, vaya a *Firewalls y redes virtuales*.
    1. Haga clic en la alternancia **activado** para permitir el acceso a los servicios de Azure.
    1. Haga clic en **+Agregar dirección IP de cliente** y haga clic en **Guardar**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que reciba datos en un destino SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP. 

### <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

### <a name="open-invitation"></a>Apertura de una invitación

1. Puede abrir la invitación desde el correo electrónico o directamente en Azure Portal. 

   Para abrir la invitación desde el correo electrónico, compruebe la bandeja de entrada para ver si hay una invitación de su proveedor de datos. La invitación es de Microsoft Azure y su título es **Azure Data Share invitation from <yourdataprovider@domain.com>** (Invitación para Azure Data Share de yourdataprovider@domain.com). Haga clic en **Ver invitación** para ver su invitación en Azure. 

   Para abrir la invitación desde Azure Portal directamente, busque **Invitaciones de recurso compartido de datos** en Azure Portal. Este paso le lleva a la lista de invitaciones de Data Share.

   ![Lista de invitaciones](./media/invitations.png "Lista de invitaciones") 

1. Seleccione el recurso compartido que desea ver. 

### <a name="accept-invitation"></a>Aceptación de la invitación
1. Asegúrese de que se han revisado todos los campos, incluido el de **condiciones de uso**. Si acepta las condiciones de uso, se le solicitará que seleccione la casilla para indicarlo. 

   ![Condiciones de uso](./media/terms-of-use.png "Términos de uso") 

1. En *Target Data Share Account* (Cuenta de Azure Data Share de destino), seleccione la suscripción y el grupo de recursos en los que va a implementar su instancia de Azure Data Share. 

   En el campo **Data Share Account** (Cuenta de Azure Data Share), seleccione **Crear nuevo** si no tiene una cuenta de Azure Data Share existente. En caso contrario, seleccione la cuenta de Azure Data Share existente en la que desea aceptar los datos compartidos. 

   En el campo **Received Share Name** (Nombre del recurso compartido recibido), puede dejar el valor predeterminado que ha especificado el proveedor de datos o especificar un nombre nuevo para el recurso compartido recibido. 

   Cuando haya aceptado los términos de uso y especificado una cuenta de Data Share para administrar el recurso compartido recibido, seleccione **Accept and configure** (Aceptar y configurar). Se creará una suscripción al recurso compartido. 

   ![Opciones de aceptación](./media/accept-options.png "Opciones de aceptación") 

   Esta opción le lleva al recurso compartido recibido en la cuenta de Data Share. 

   Si no desea aceptar la invitación, seleccione *Reject* (Rechazar). 

### <a name="configure-received-share"></a>Configuración del recurso compartido recibido
Siga los pasos que se indican a continuación para configurar dónde quiere recibir los datos.

1. Seleccione la pestaña **Conjuntos de datos**. Active la casilla situada junto al conjunto de datos al que desearía asignar un destino. Seleccione **+ Asignar a destino** para elegir un almacén de datos de destino. 

   ![Asignación a destino](./media/dataset-map-target.png "Asignar a destino") 

1. Seleccione un almacén de datos de destino en el que quiera colocar los datos. Los archivos de datos o las tablas del almacén de datos de destino se sobrescribirán con la misma ruta de acceso y nombre. 

   ![Cuenta de almacenamiento de destino](./media/dataset-map-target-sql.png "Almacén de datos de destino") 

1. En el caso del uso compartido basado en instantáneas, si el proveedor de datos ha creado una programación de instantáneas para proporcionar una actualización periódica de los datos, también puede habilitarla seleccionando la pestaña **Programación de instantáneas**. Active la casilla situada junto a la programación de instantáneas y seleccione **+ Habilitar**.

   ![Habilitación de la programación de instantáneas](./media/enable-snapshot-schedule.png "Habilitación de la programación de instantáneas")

### <a name="trigger-a-snapshot"></a>Desencadenamiento de una instantánea
Estos pasos solo se aplican al uso compartido basado en instantáneas.

1. Para desencadenar una instantánea, seleccione la pestaña **Detalles** y, luego, **Desencadenar instantánea**. En este caso, puede desencadenar una instantánea completa o incremental de los datos. Si es la primera vez que recibe datos del proveedor de datos, seleccione una copia completa. En el caso de los orígenes de SQL, solo se admite la instantánea completa.

   ![Desencadenamiento de instantánea](./media/trigger-snapshot.png "Desencadenamiento de instantánea") 

1. Cuando el estado de la última ejecución sea *correcto*, vaya al almacén de datos de destino para ver los datos recibidos. Seleccione **Conjuntos de datos**y haga clic en el vínculo de la ruta de acceso de destino. 

   ![Conjuntos de datos de consumidor](./media/consumer-datasets.png "Asignación de conjunto de datos de consumidor") 

### <a name="view-history"></a>Visualización del historial
Este paso solo se aplica al uso compartido basado en instantáneas. Para ver el historial de las instantáneas, seleccione la pestaña **Historial**. Aquí encontrará un historial de todas las instantáneas que se generaron en los últimos 30 días. 

## <a name="next-steps"></a>Pasos siguientes
Ha obtenido información sobre cómo compartir y recibir datos de la cuenta de almacenamiento mediante el servicio Azure Data Share. Para más información sobre el uso compartido desde otros orígenes de datos, vaya a los [almacenes de datos compatibles](supported-data-stores.md).

