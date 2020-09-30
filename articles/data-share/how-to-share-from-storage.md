---
title: Uso compartido y recepción de datos de Azure Blob Storage y Azure Data Lake Storage
description: Obtenga información sobre cómo compartir y recibir datos de Azure Blob Storage y Azure Data Lake Storage
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 08/28/2020
ms.openlocfilehash: a54a9d4c50852fe78fd245723dd29f487d58f4b1
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269909"
---
# <a name="share-and-receive-data-from-azure-blob-storage-and-azure-data-lake-storage"></a>Uso compartido y recepción de datos de Azure Blob Storage y Azure Data Lake Storage

[!INCLUDE[appliesto-storage](includes/appliesto-storage.md)]

Azure Data Share admite el uso compartido basado en instantáneas de la cuenta de almacenamiento. En este artículo se explica cómo compartir y recibir datos de los orígenes siguientes: Azure Blob Storage, Azure Data Lake Storage Gen1 y Azure Data Lake Storage Gen2.

Azure Data Share admite el uso compartido de archivos, carpetas y sistemas de archivos desde Azure Data Lake Gen1 y Azure Data Lake Gen2. También admite el uso compartido de blobs, carpetas y contenedores desde Azure Blob Storage. Actualmente, solo se admiten blobs en bloques. Los datos compartidos de estos orígenes pueden recibirse en Azure Data Lake Gen2 o Azure Blob Storage.

Cuando los sistemas de archivos, contenedores o carpetas se comparten mediante el uso compartido basado en instantáneas, el consumidor de datos puede optar por hacer una copia completa de los datos de los recursos compartidos o aprovechar la capacidad de realizar instantáneas incrementales para copiar solo los archivos nuevos o actualizados. La instantánea incremental se basa en la hora de la última modificación de los archivos. Los archivos existentes con el mismo nombre se sobrescribirán.

## <a name="share-data"></a>Compartir datos

### <a name="prerequisites-to-share-data"></a>Requisitos previos para compartir datos

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* La dirección de correo electrónico de inicio de sesión de Azure del destinatario (con el alias de correo electrónico no funcionará).
* Si el almacén de datos de Azure de origen está en una suscripción de Azure diferente a la que va a usar para crear el recurso de Data Share, registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción en la que se encuentra el almacén de datos de Azure. 

### <a name="prerequisites-for-source-storage-account"></a>Requisitos previos de la cuenta de almacenamiento de origen

* Una cuenta de Azure Storage: si no la tiene, puede crear una cuenta de [Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account)
* Permisos para escribir en la cuenta de almacenamiento, que se encuentra en *Microsoft.Storage/storageAccounts/write*. Este permiso existe en el rol de colaborador.
* Permisos para agregar la asignación de roles a la cuenta de almacenamiento, que se encuentra en *Microsoft.Authorization/role assignments/write*. Este permiso existe en el rol de propietario. 

### <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

### <a name="create-a-data-share-account"></a>Creación de una cuenta de Azure Data Share

Cree un recurso de Azure Data Share en un grupo de recursos de Azure.

1. Seleccione el botón de menú de la esquina superior izquierda del portal y, a continuación, seleccione **Crear un recurso** (+).

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

1. Vaya hasta el objeto que desea compartir y seleccione "Add Datasets" (Agregar conjuntos de datos). 

    ![SelectDatasets](./media/select-datasets.png "Selección de conjuntos de datos")    

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

* Una cuenta de Azure Storage: si no la tiene, puede crear una cuenta de [Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account). 
* Permisos para escribir en la cuenta de almacenamiento, que se encuentra en *Microsoft.Storage/storageAccounts/write*. Este permiso existe en el rol de colaborador. 
* Permisos para agregar la asignación de roles a la cuenta de almacenamiento, que se encuentra en *Microsoft.Authorization/role assignments/write*. Este permiso existe en el rol de propietario.  

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

1. Seleccione un almacén de datos de destino en el que quiera colocar los datos. Los archivos de datos del almacén de datos de destino se sobrescribirán con la misma ruta de acceso y nombre. 

   ![Cuenta de almacenamiento de destino](./media/map-target.png "Almacenamiento de destino") 

1. En el caso del uso compartido basado en instantáneas, si el proveedor de datos ha creado una programación de instantáneas para proporcionar una actualización periódica de los datos, también puede habilitarla seleccionando la pestaña **Programación de instantáneas**. Active la casilla situada junto a la programación de instantáneas y seleccione **+ Habilitar**.

   ![Habilitación de la programación de instantáneas](./media/enable-snapshot-schedule.png "Habilitación de la programación de instantáneas")

### <a name="trigger-a-snapshot"></a>Desencadenamiento de una instantánea
Estos pasos solo se aplican al uso compartido basado en instantáneas.

1. Para desencadenar una instantánea, seleccione la pestaña **Detalles** y, luego, **Desencadenar instantánea**. En este caso, puede desencadenar una instantánea completa o incremental de los datos. Si es la primera vez que recibe datos del proveedor de datos, seleccione una copia completa. 

   ![Desencadenamiento de instantánea](./media/trigger-snapshot.png "Desencadenamiento de instantánea") 

1. Cuando el estado de la última ejecución sea *correcto*, vaya al almacén de datos de destino para ver los datos recibidos. Seleccione **Conjuntos de datos**y haga clic en el vínculo de la ruta de acceso de destino. 

   ![Conjuntos de datos de consumidor](./media/consumer-datasets.png "Asignación de conjunto de datos de consumidor") 

### <a name="view-history"></a>Visualización del historial
Este paso solo se aplica al uso compartido basado en instantáneas. Para ver el historial de las instantáneas, seleccione la pestaña **Historial**. Aquí encontrará un historial de todas las instantáneas que se generaron en los últimos 30 días. 

## <a name="next-steps"></a>Pasos siguientes
Ha obtenido información sobre cómo compartir y recibir datos de la cuenta de almacenamiento mediante el servicio Azure Data Share. Para obtener más información sobre el uso compartido desde otros orígenes de datos, vaya a los [almacenes de datos compatibles](supported-data-stores.md).

