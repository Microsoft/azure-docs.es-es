---
title: Uso de identidades administradas de Azure para crear entornos en DevTest Labs | Microsoft Docs
description: Obtenga más información sobre el uso de identidades administradas en Azure para implementar entornos en un laboratorio de Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: e5bac4210afee6db1c7617dac1cd6d2ff9149439
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718990"
---
# <a name="use-azure-managed-identities-to-deploy-environments-in-a-lab"></a>Uso de identidades administradas de Azure para implementar entornos en un laboratorio 

Como propietario de un laboratorio, puede usar una identidad administrada para implementar entornos en un laboratorio. Esta característica es útil en escenarios en los que el entorno contiene o incluye referencias a recursos de Azure, como almacenes de claves, galerías de imágenes compartidas y redes que son externas al grupo de recursos del entorno. Permite la creación de entornos de espacio aislado que no se limitan al grupo de recursos de dicho entorno.

> [!NOTE]
> Actualmente, se admite una única identidad asignada por el usuario por laboratorio. 

## <a name="prerequisites"></a>Prerequisites

- [Creación, enumeración, eliminación o asignación de un rol a una identidad administrada asignada por el usuario mediante Azure Portal](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md). 
    
    Asegúrese de que la identidad administrada se haya creado en la misma región y suscripción que el laboratorio. No es necesario que la identidad administrada esté en el mismo grupo de recursos.

## <a name="use-azure-portal"></a>Usar Azure Portal

En esta sección, como propietario del laboratorio, use Azure Portal para agregar una identidad administrada por el usuario al laboratorio. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **DevTest Labs**.
1. En la lista de laboratorios, seleccione el laboratorio que quiera.
1. Seleccione **Configuración y directivas** -> **Identidad (versión preliminar)** . 
1. Para agregar una identidad asignada por el usuario, seleccione la pestaña **User Assigned** (Asignada por el usuario).
1. Presione **Agregar**.
1. Seleccione en la lista desplegable un usuario existente que haya creado o al que tenga acceso.
 
    ![Adición de una identidad administrada por el usuario](./media/use-managed-identities-environments/add-user-managed-identity.png)
1. Presione **Guardar** en la parte superior de la página.

    Una vez guardado, el laboratorio usará esta identidad al implementar todos los entornos de laboratorio. También puede acceder al recurso de la identidad en Azure mediante la selección de la identidad en la lista. 

El propietario del laboratorio no necesita hacer nada especial al implementar un entorno siempre que la identidad agregada al laboratorio tenga permisos para los recursos externos a los que el laboratorio necesita tener acceso. 

Para cambiar la identidad administrada por el usuario asignada al laboratorio, primero debe quitar la identidad que se ha adjuntado al laboratorio y, a continuación, agregar otra al laboratorio. Para quitar una identidad adjuntada al laboratorio, seleccione **... (puntos suspensivos)** y haga clic en **Quitar**. 

## <a name="use-api"></a>Uso de la API

1. Tras crear una identidad, anote su id. de recurso. Debería ser similar al ejemplo siguiente: 

    `/subscriptions/0000000000-0000-0000-0000-00000000000000/resourceGroups/<RESOURCE GROUP NAME> /providers/Microsoft.ManagedIdentity/userAssignedIdentities/<NAME of USER IDENTITY>`.
1. Ejecute un método HTTPS PUT para agregar un nuevo recurso `ServiceRunner` al laboratorio similar al ejemplo siguiente. El recurso del ejecutor del servicio es un recurso de proxy para administrar y controlar identidades administradas en DevTest Labs. El nombre del ejecutor del servicio puede ser cualquier nombre válido, pero se recomienda usar el nombre del recurso de identidad administrada. 
 
    ```json
    PUT https://management.azure.com/subscriptions/{subId}/resourceGroups/{rg}/providers/Microsoft.Devtestlab/labs/{yourlabname}/serviceRunners/{serviceRunnerName}

    {
        "location": "{location}",
        "identity":{
            "type": "userAssigned",
            "userAssignedIdentities":{
                "[userAssignedIdentityResourceId]":{}
            }
        }
        "properties":{
            "identityUsageType":"Environment"
                     }
          
    }
    ```
 
    Este es un ejemplo: 

    ```json
    PUT https://management.azure.com/subscriptions/0000000000-0000-0000-0000-000000000000000/resourceGroups/exampleRG/providers/Microsoft.Devtestlab/labs/mylab/serviceRunners/sampleuseridentity

    {
        "location": "eastus",
        "identity":{
            "type": "userAssigned",
            "userAssignedIdentities":{
                "/subscriptions/0000000000-0000-0000-0000-000000000000000/resourceGroups/exampleRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/sampleuseridentity":{}
            }
        }
        "properties":{
            "identityUsageType":"Environment"
                     }
    }
    ```
 
Una vez que la identidad asignada por el usuario se agrega al laboratorio, el servicio de Azure DevTest Labs la usará al implementar entornos de Azure Resource Manager. Por ejemplo, si necesita la plantilla de Resource Manager para tener acceso a una imagen externa de la galería de imágenes compartidas, asegúrese de que la identidad que ha agregado al laboratorio tiene los permisos mínimos necesarios para el recurso de la galería de imágenes compartidas. 
