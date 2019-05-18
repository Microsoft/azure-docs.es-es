---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 05/06/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 56545b02c35d68455497c93582073c73d6c75da5
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65815477"
---
Para crear una cuenta de almacenamiento de uso general v2 en Azure Portal, siga estos pasos:

1. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba **Cuentas de almacenamiento**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Cuentas de almacenamiento**.
1. En la ventana **Cuentas de almacenamiento** que aparece, elija **Agregar**.
1. Seleccione la suscripción en la que se va a crear la cuenta de almacenamiento.
1. En el campo **Grupo de recursos**, haga clic en **Crear nuevo**. Escriba un nombre para el nuevo grupo de recursos, como se muestra en la siguiente imagen.

    ![Captura de pantalla que muestra cómo crear un grupo de recursos en el portal](./media/storage-create-account-portal-include/create-resource-group.png)

1. Después, escriba un nombre para la cuenta de almacenamiento. El nombre que elija debe ser único en Azure. El nombre debe tener también una longitud de entre 3 y 24 caracteres y solo puede contener números y letras minúsculas.
1. Seleccione una ubicación para la cuenta de almacenamiento o utilice la ubicación predeterminada.
1. Deje estos campos con sus valores predeterminados:

   |Campo  |Valor  |
   |---------|---------|
   |Modelo de implementación     |Resource Manager         |
   |Rendimiento     |Estándar         |
   |Tipo de cuenta     |StorageV2 (uso general v2)         |
   |Replicación     |Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS).         |
   |Nivel de acceso     |Acceso frecuente         |

1. Seleccione **Revisar y crear** para revisar la configuración de la cuenta de almacenamiento y crear la cuenta.
1. Seleccione **Crear**.

Para obtener más información sobre los tipos de cuentas de almacenamiento y otros parámetros de la cuenta de almacenamiento, consulte [Introducción a las cuentas de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-account-overview). Para más información sobre los grupos de recursos, consulte [Información general de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview). 
