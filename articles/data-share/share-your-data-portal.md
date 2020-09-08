---
title: 'Uso compartido fuera de la organización (Azure Portal): Inicio rápido de Azure Data Share'
description: Aprenda a compartir datos con clientes y asociados mediante Azure Data Share en este inicio rápido.
author: joannapea
ms.author: joanpo
ms.service: data-share
ms.topic: quickstart
ms.date: 08/19/2020
ms.openlocfilehash: 5ceaf949df88468b2239bd901f639ba6096b0d5f
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269648"
---
# <a name="quickstart-share-data-using-azure-data-share-in-the-azure-portal"></a>Inicio rápido: Uso compartido de los datos con Azure Data Share en Azure Portal

En este inicio rápido, obtendrá información sobre cómo configurar un nuevo Azure Data Share mediante Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.


## <a name="create-a-data-share-account"></a>Creación de una cuenta de Azure Data Share

Cree un recurso de Azure Data Share en un grupo de recursos de Azure.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Seleccione el botón **Crear un recurso** (+) de la esquina superior izquierda del portal.

1. Busque *Data Share*.

1. Seleccione **Data Share** y, después, **Crear**.

1. Rellene los detalles básicos del recurso de Azure Data Share con la siguiente información. 

   **Configuración** | **Valor sugerido** | **Descripción del campo**
   |---|---|---|
   | Subscription | Su suscripción | Seleccione la suscripción de Azure que desea usar para la cuenta del recurso compartido de datos.|
   | Grupo de recursos | *test-resource-group* | Use un grupo de recursos existente o cree uno. |
   | Location | *Este de EE. UU. 2* | Seleccione una región para la cuenta de Azure Data Share.
   | Nombre | *datashareaccount* | Especifique un nombre para la cuenta del recurso compartido de datos. |

1. Seleccione **Revisar + Crear** y, después, **Crear** para aprovisionar la cuenta del recurso compartido de datos. El aprovisionamiento de una nueva cuenta de Azure Data Share normalmente tarda aproximadamente 2 minutos, o menos.

1. Cuando la implementación se complete, seleccione **Ir al grupo de recursos**.

## <a name="create-a-share"></a>Creación de un recurso compartido

1. Vaya a la página de información general de Azure Data Share.

   ![Uso compartido de los datos](./media/share-receive-data.png "Uso compartido de los datos") 

1. Seleccione **Start sharing your data** (Empezar a compartir datos).

1. Seleccione **Crear**.

1. Rellene los detalles del recurso compartido. Especifique un nombre, un tipo de recurso compartido, una descripción del contenido de los recursos compartidos y las condiciones de uso (opcional). 

   ![EnterShareDetails](./media/enter-share-details.png "Introducción de detalles del recurso compartido") 

1. Seleccione **Continuar**.

1. Para agregar conjuntos de datos al recurso compartido, seleccione **Agregar conjuntos de datos**. 

   ![Adición de conjuntos de datos al recurso compartido](./media/datasets.png "Conjuntos de datos")

1. Seleccione el tipo de conjunto de datos que desea agregar. Verá una lista diferente de tipos de conjuntos de datos en función del tipo de recurso compartido (instantánea o en contexto) que haya seleccionado en el paso anterior. Si comparte desde Azure SQL Database o Azure SQL Data Warehouse, se le pedirán credenciales de SQL. Realice la autenticación con el usuario que ha creado como parte de los requisitos previos.

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

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando el recurso ya no sea necesario, vaya a la página **Información general de Azure Data Share** y seleccione **Eliminar** para quitarlo.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear un recurso de Azure Data Share. Para obtener información acerca de la forma en que in consumidor de datos puede aceptar y recibir un recurso compartido de datos, vaya al tutorial para [aceptar y recibir datos](subscribe-to-data-share.md). 
