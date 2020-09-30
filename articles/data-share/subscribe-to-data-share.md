---
title: 'Tutorial: Aceptación y recepción de datos mediante Azure Data Share'
description: 'Tutorial: Aceptación y recepción de datos mediante Azure Data Share'
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: tutorial
ms.date: 08/14/2020
ms.openlocfilehash: ce47bc5e880f15eaa1bbf07477673d2475e5a10a
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89489925"
---
# <a name="tutorial-accept-and-receive-data-using-azure-data-share"></a>Tutorial: Aceptación y recepción de datos con Azure Data Share  

En este tutorial aprenderá a aceptar una invitación a un recurso compartido de datos mediante Azure Data Share. En él aprenderá a recibir los datos que se hayan compartido contigo, así como a habilitar un intervalo de actualización periódico para asegurarse de que siempre tiene la instantánea más reciente de los datos que se comparten con usted. 

> [!div class="checklist"]
> * Aceptación de una invitación de Azure Data Share
> * Creación de una cuenta de Azure Data Share
> * Especificación de un destino para los datos
> * Creación de una suscripción en su recurso compartido de datos para la actualización programada

## <a name="prerequisites"></a>Prerrequisitos
Para poder aceptar una invitación a compartir datos, previamente debe aprovisionar un número de recursos de Azure, que se enumeran a continuación. 

Asegúrese de que se cumplen todos los requisitos previos antes de aceptar una invitación para compartir datos. 

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una invitación de Azure Data Share: Una invitación de Microsoft Azure con el asunto "Azure Data Share invitation from **<yourdataprovider@domain.com>** " (Invitación para Azure Data Share de yourdataprovider@domain.com).
* Registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción de Azure en la que va a crear un recurso de Data Share y la suscripción de Azure en la que se encuentran los almacenes de datos de Azure de destino.

### <a name="receive-data-into-a-storage-account"></a>Recepción de los datos en una cuenta de almacenamiento: 

* Una cuenta de Azure Storage: si no la tiene, puede crear una cuenta de [Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account). 
* Permisos para escribir en la cuenta de almacenamiento, que se encuentra en *Microsoft.Storage/storageAccounts/write*. Este permiso existe en el rol de colaborador. 
* Permisos para agregar la asignación de roles a la cuenta de almacenamiento, que se encuentra en *Microsoft.Authorization/role assignments/write*. Este permiso existe en el rol de propietario.  

### <a name="receive-data-into-a-sql-based-target"></a>Recepción de datos en un destino basado en SQL:

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


### <a name="receive-data-into-an-azure-data-explorer-cluster"></a>Recibir datos en un clúster de Azure Data Explorer: 

* Un clúster de Azure Data Explorer en el mismo centro de datos de Azure que el clúster de Data Explorer del proveedor de datos: Si aún no tiene uno, puede crear un [clúster de Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/create-cluster-database-portal). Si no conoce el centro de datos de Azure del clúster del proveedor de datos, puede crear el clúster más adelante en el proceso.
* Permisos para escribir en el clúster de Azure Data Explorer, que se encuentra en *Microsoft.Kusto/clusters/write*. Este permiso existe en el rol de colaborador. 
* Permisos para agregar la asignación de roles al clúster de Azure Data Explorer, que se encuentra en *Microsoft.Authorization/role assignments/write*. Este permiso existe en el rol de propietario. 

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="open-invitation"></a>Apertura de una invitación

1. Puede abrir la invitación desde el correo electrónico o directamente en Azure Portal. 

   Para abrir la invitación desde el correo electrónico, compruebe la bandeja de entrada para ver si hay una invitación de su proveedor de datos. La invitación es de Microsoft Azure y su título es **Azure Data Share invitation from <yourdataprovider@domain.com>** (Invitación para Azure Data Share de yourdataprovider@domain.com). Haga clic en **Ver invitación** para ver su invitación en Azure. 

   Para abrir la invitación desde Azure Portal directamente, busque **Invitaciones de recurso compartido de datos** en Azure Portal. Este paso le lleva a la lista de invitaciones de Data Share.

   ![Lista de invitaciones](./media/invitations.png "Lista de invitaciones") 

1. Seleccione el recurso compartido que desea ver. 

## <a name="accept-invitation"></a>Aceptación de la invitación
1. Asegúrese de que se han revisado todos los campos, incluido el de **condiciones de uso**. Si acepta las condiciones de uso, se le solicitará que seleccione la casilla para indicarlo. 

   ![Condiciones de uso](./media/terms-of-use.png "Términos de uso") 

1. En *Target Data Share Account* (Cuenta de Azure Data Share de destino), seleccione la suscripción y el grupo de recursos en los que va a implementar su instancia de Azure Data Share. 

   En el campo **Data Share Account** (Cuenta de Azure Data Share), seleccione **Crear nuevo** si no tiene una cuenta de Azure Data Share existente. En caso contrario, seleccione la cuenta de Azure Data Share existente en la que desea aceptar los datos compartidos. 

   En el campo **Received Share Name** (Nombre del recurso compartido recibido), puede dejar el valor predeterminado que ha especificado el proveedor de datos o especificar un nombre nuevo para el recurso compartido recibido. 

   Cuando haya aceptado los términos de uso y especificado una cuenta de Data Share para administrar el recurso compartido recibido, seleccione **Accept and configure** (Aceptar y configurar). Se creará una suscripción al recurso compartido. 

   ![Opciones de aceptación](./media/accept-options.png "Opciones de aceptación") 

   Esta opción le lleva al recurso compartido recibido en la cuenta de Data Share. 

   Si no desea aceptar la invitación, seleccione *Reject* (Rechazar). 

## <a name="configure-received-share"></a>Configuración del recurso compartido recibido
Siga los pasos que se indican a continuación para configurar dónde quiere recibir los datos.

1. Seleccione la pestaña **Conjuntos de datos**. Active la casilla situada junto al conjunto de datos al que desearía asignar un destino. Seleccione **+ Asignar a destino** para elegir un almacén de datos de destino. 

   ![Asignación a destino](./media/dataset-map-target.png "Asignar a destino") 

1. Seleccione un tipo de datos de destino en el que quiera colocar los datos. Los archivos de datos o las tablas del almacén de datos de destino se sobrescribirán con la misma ruta de acceso y nombre. 

   Para el uso compartido en contexto, seleccione un almacén de datos en la ubicación especificada. La ubicación es el centro de datos de Azure donde se encuentra el almacén de datos de origen del proveedor de datos. Una vez que se asigna un conjunto de datos, puede seguir el vínculo de la ruta de acceso de destino para acceder a los datos.

   ![Cuenta de almacenamiento de destino](./media/dataset-map-target-sql.png "Almacenamiento de destino") 

1. En el caso del uso compartido basado en instantáneas, si el proveedor de datos ha creado una programación de instantáneas para proporcionar una actualización periódica de los datos, también puede habilitarla seleccionando la pestaña **Programación de instantáneas**. Active la casilla situada junto a la programación de instantáneas y seleccione **+ Habilitar**.

   ![Habilitación de la programación de instantáneas](./media/enable-snapshot-schedule.png "Habilitación de la programación de instantáneas")

## <a name="trigger-a-snapshot"></a>Desencadenamiento de una instantánea
Estos pasos solo se aplican al uso compartido basado en instantáneas.

1. Para desencadenar una instantánea, seleccione la pestaña **Detalles** y, luego, **Desencadenar instantánea**. En este caso, puede desencadenar una instantánea completa o incremental de los datos. Si es la primera vez que recibe datos del proveedor de datos, seleccione una copia completa. 

   ![Desencadenamiento de instantánea](./media/trigger-snapshot.png "Desencadenamiento de instantánea") 

1. Cuando el estado de la última ejecución sea *correcto*, vaya al almacén de datos de destino para ver los datos recibidos. Seleccione **Conjuntos de datos**y haga clic en el vínculo de la ruta de acceso de destino. 

   ![Conjuntos de datos de consumidor](./media/consumer-datasets.png "Asignación de conjunto de datos de consumidor") 

## <a name="view-history"></a>Visualización del historial
Este paso solo se aplica al uso compartido basado en instantáneas. Para ver el historial de las instantáneas, seleccione la pestaña **Historial**. Aquí encontrará un historial de todas las instantáneas que se generaron en los últimos 30 días. 

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido a aceptar y recibir un recurso de Azure Data Share. Para más información acerca de los conceptos de Azure Data Share, diríjase a [Conceptos: terminología de Azure Data Share](terminology.md).