---
title: 'Tutorial`:` Uso de identidades administradas para acceder a Azure Cosmos DB (Linux): Azure AD'
description: Este tutorial contiene directrices acerca de cómo utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Cosmos DB.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/09/2018
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2006c44d68d9570af0bfa410cc7fe908502d2ba5
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267996"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-cosmos-db"></a>Tutorial: Uso de identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Azure Cosmos DB 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]


En este tutorial se muestra cómo usar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Cosmos DB. Aprenderá a:

> [!div class="checklist"]
> * Creación de una cuenta de Cosmos DB
> * Creación de una colección en la cuenta de Cosmos DB
> * Conceder a la identidad administrada asignada por el sistema acceso a una instancia de Azure Cosmos DB
> * Recuperación de `principalID` de la identidad administrada asignada por el sistema de la máquina virtual Linux
> * Obtener un token de acceso y su uso para llamar a Azure Resource Manager
> * Obtención de las claves de acceso desde Azure Resource Manager para realizar llamadas a Cosmos DB

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

Para ejecutar los ejemplos de script de la CLI de este tutorial, tiene dos opciones:

- Use [Azure Cloud Shell](~/articles/cloud-shell/overview.md) desde Azure Portal o mediante el botón **Pruébelo**, situado en la esquina superior derecha de cada bloque de código.
- [Instale la versión más reciente de la CLI 2.0](/cli/azure/install-azure-cli) (2.0.23 o posterior), si prefiere usar una consola de la CLI local.

## <a name="create-a-cosmos-db-account"></a>Creación de una cuenta de Cosmos DB 

Si aún no tiene una, cree una cuenta de Cosmos DB. También puede omitir este paso y usar una cuenta de Cosmos DB existente. 

1. Haga clic en el botón **+/Crear nuevo servicio** de la esquina superior izquierda de Azure Portal.
2. Haga clic en **Bases de datos** y, a continuación, en **Azure Cosmos DB** para mostrar el panel "Nueva cuenta".
3. Escriba un **identificador** para la cuenta de Cosmos DB, el cual se utilizará más adelante.  
4. **API** se debe establecer en "SQL". El enfoque descrito en este tutorial se puede utilizar con los otros tipos de API disponibles, pero los pasos de este tutorial son para la API de SQL.
5. Asegúrese de que **Suscripción** y **Grupo de recursos** coinciden con los que especificó cuando creó la máquina virtual en el paso anterior.  Seleccione una **Ubicación** en la que Cosmos DB esté disponible.
6. Haga clic en **Crear**.

## <a name="create-a-collection-in-the-cosmos-db-account"></a>Creación de una colección en la cuenta de Cosmos DB

A continuación, agregue una colección de datos en la cuenta de Cosmos DB que podrá consultar en pasos posteriores.

1. Vaya a la cuenta de Cosmos DB recién creada.
2. En la pestaña **Información general**, haga clic en el botón **+/Agregar colección** y aparecerá un panel "Agregar colección".
3. Proporcione para la colección un identificador de base de datos, el identificador de la colección, seleccione una capacidad de almacenamiento, escriba una clave de partición, escriba un valor de rendimiento y, luego, haga clic en **Aceptar**.  Para este tutorial, es suficiente con utilizar "Test" como identificador de la base de datos e identificador de la colección, seleccionar una capacidad de almacenamiento fijo y el rendimiento más bajo (400 RU/s).  

## <a name="retrieve-the-principalid-of-the-linux-vms-system-assigned-managed-identity"></a>Recuperación de `principalID` de la identidad administrada asignada por el sistema de la máquina virtual Linux

Para obtener acceso a las claves de acceso de la cuenta de Cosmos DB desde Resource Manager en la siguiente sección, es preciso recuperar el `principalID` de la identidad administrada asignada por el sistema de la máquina virtual Linux.  No olvide reemplazar los valores de los parámetros `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` (grupo de recursos en el que reside la máquina virtual) y `<VM NAME>` por sus propios valores.

```azurecli-interactive
az resource show --id /subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAMe> --api-version 2017-12-01
```
La respuesta incluye los detalles de la identidad administrada asignada por el sistema (observe cómo se usa principalID en la sección siguiente):

```output  
{
    "id": "/subscriptions/<SUBSCRIPTION ID>/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAMe>",
  "identity": {
    "principalId": "6891c322-314a-4e85-b129-52cf2daf47bd",
    "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533f8",
    "type": "SystemAssigned"
 }

```
## <a name="grant-your-linux-vms-system-assigned-identity-access-to-the-cosmos-db-account-access-keys"></a>Concesión de acceso a la identidad asignada por el sistema de la máquina virtual Linux a las claves de acceso de la cuenta de Cosmos DB

Cosmos DB no admite la autenticación de Azure AD de forma nativa. No obstante, puede usar una identidad administradas para recuperar una clave de acceso de Cosmos DB desde Resource Manager y usar dicha clave para acceder a Cosmos DB. En este paso, va a conceder a la identidad administrada asignada por el sistema acceso a las claves de la cuenta de Cosmos DB.

Para conceder a la identidad administrada asignada por el sistema acceso a la cuenta de Cosmos DB en Azure Resource Manager mediante la CLI de Azure, actualice los valores de `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` y `<COSMOS DB ACCOUNT NAME>` para su entorno. Reemplace `<MI PRINCIPALID>` por la propiedad `principalId` que devuelve el comando `az resource show` en Tutorial: Uso de identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Azure Cosmos DB.  Cosmos DB admite dos niveles de granularidad en las claves de acceso: acceso de lectura y escritura a la cuenta y acceso de solo lectura a la cuenta.  Asigne el rol `DocumentDB Account Contributor` si desea obtener claves de lectura y escritura para la cuenta o bien asigne el rol `Cosmos DB Account Reader Role` si desea obtener claves de solo lectura para la cuenta:

```azurecli-interactive
az role assignment create --assignee <MI PRINCIPALID> --role '<ROLE NAME>' --scope "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMODS DB ACCOUNT NAME>"
```

La respuesta incluye los detalles de la asignación de roles que se ha creado:

```output
{
  "id": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT>/providers/Microsoft.Authorization/roleAssignments/5b44e628-394e-4e7b-bbc3-d6cd4f28f15b",
  "name": "5b44e628-394e-4e7b-bbc3-d6cd4f28f15b",
  "properties": {
    "principalId": "c0833082-6cc3-4a26-a1b1-c4b5f90a981f",
    "roleDefinitionId": "/subscriptions/<SUBSCRIPTION ID>/providers/Microsoft.Authorization/roleDefinitions/fbdf93bf-df7d-467e-a4d2-9458aa1360c8",
    "scope": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT>"
  },
  "resourceGroup": "<RESOURCE GROUP>",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="get-an-access-token-using-the-linux-vms-system-assigned-managed-identity-and-use-it-to-call-azure-resource-manager"></a>Obtención de un token de acceso mediante una identidad administrada asignada por el sistema de la máquina virtual Linux y su uso para llamar a Azure Resource Manager

En el resto del tutorial, vamos a trabajar desde la máquina virtual que se creó anteriormente.

Para completar estos pasos, necesitará un cliente SSH. Si usa Windows, puede usar el cliente SSH en el [Subsistema de Windows para Linux](/windows/wsl/install-win10). Si necesita ayuda para configurar las claves del cliente de SSH, consulte [Uso de SSH con Windows en Azure](../../virtual-machines/linux/ssh-from-windows.md) o [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

1. En Azure Portal, vaya a **Máquinas virtuales**, vaya a la máquina virtual Linux y, a continuación, desde la página **Información general**, haga clic en **Conectar** en la parte superior. Copie la cadena para conectarse a la máquina virtual. 
2. Conéctese a la máquina virtual mediante un cliente SSH.  
3. A continuación, se le pedirá que escriba la **contraseña** que agregó al crear la **máquina virtual Linux**. Debe haber iniciado sesión correctamente.  
4. Utilice CURL para obtener un token de acceso para Azure Resource Manager: 
     
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true   
    ```
 
    > [!NOTE]
    > En la solicitud anterior, el valor del parámetro "resource" tiene que coincidir exactamente con el que Azure AD espera. Al usar el id. de recurso de Azure Resource Manager, debe incluir la barra diagonal final en el URI.
    > En la siguiente respuesta, el elemento access_token se ha abreviado.
    
    ```bash
    {"access_token":"eyJ0eXAiOi...",
     "expires_in":"3599",
     "expires_on":"1518503375",
     "not_before":"1518499475",
     "resource":"https://management.azure.com/",
     "token_type":"Bearer",
     "client_id":"1ef89848-e14b-465f-8780-bf541d325cd5"}
     ```
    
## <a name="get-access-keys-from-azure-resource-manager-to-make-cosmos-db-calls"></a>Obtención de las claves de acceso desde Azure Resource Manager para realizar llamadas a Cosmos DB  

Ahora, utilice CURL para llamar a Resource Manager mediante el token de acceso que se recuperó en la sección anterior, para recuperar la clave de acceso de la cuenta de Cosmos DB. Una vez que tenemos la clave de acceso, podemos realizar consultas en Cosmos DB. Asegúrese de reemplazar los valores de los parámetros `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` y `<COSMOS DB ACCOUNT NAME>` con sus propios valores. Reemplace el valor de `<ACCESS TOKEN>` por el token de acceso que se recuperó anteriormente.  Si quiere recuperar claves de lectura y escritura, use el tipo de operación de claves `listKeys`.  Si quiere recuperar claves de solo lectura, use el tipo de operación de claves `readonlykeys`:

```bash 
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT NAME>/<KEY OPERATION TYPE>?api-version=2016-03-31' -X POST -d "" -H "Authorization: Bearer <ACCESS TOKEN>" 
```

> [!NOTE]
> El texto de la dirección URL anterior distingue mayúsculas de minúsculas, por lo que tiene que fijarse en si usa mayúsculas o minúsculas para los grupos de recursos para reflejarlo adecuadamente según corresponda. Además, es importante saber que se trata de una solicitud POST, no una solicitud GET, y asegurarse de que pasa un valor para capturar un límite de longitud con -d que puede ser NULL.  

La respuesta de CURL proporciona la lista de claves.  Por ejemplo, si obtiene las claves de solo lectura:  

```bash 
{"primaryReadonlyMasterKey":"bWpDxS...dzQ==",
"secondaryReadonlyMasterKey":"38v5ns...7bA=="}
```

Ahora que tiene la clave de acceso de la cuenta de Cosmos DB, puede pasársela al SDK de Cosmos DB y realizar llamadas para acceder a la cuenta.  Como ejemplo rápido, puede pasar la clave de acceso a la CLI de Azure.  Puede obtener el valor de `<COSMOS DB CONNECTION URL>` en la pestaña **Información general** de la hoja de la cuenta de Cosmos DB en Azure Portal.  Reemplace `<ACCESS KEY>` por el valor obtenido anteriormente:

```azurecli
az cosmosdb collection show -c <COLLECTION ID> -d <DATABASE ID> --url-connection "<COSMOS DB CONNECTION URL>" --key <ACCESS KEY>
```

Este comando de la CLI devuelve detalles acerca de la colección:

```output
{
  "collection": {
    "_conflicts": "conflicts/",
    "_docs": "docs/",
    "_etag": "\"00006700-0000-0000-0000-5a8271e90000\"",
    "_rid": "Es5SAM2FDwA=",
    "_self": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/",
    "_sprocs": "sprocs/",
    "_triggers": "triggers/",
    "_ts": 1518498281,
    "_udfs": "udfs/",
    "id": "Test",
    "indexingPolicy": {
      "automatic": true,
      "excludedPaths": [],
      "includedPaths": [
        {
          "indexes": [
            {
              "dataType": "Number",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "String",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "Point",
              "kind": "Spatial"
            }
          ],
          "path": "/*"
        }
      ],
      "indexingMode": "consistent"
    }
  },
  "offer": {
    "_etag": "\"00006800-0000-0000-0000-5a8271ea0000\"",
    "_rid": "f4V+",
    "_self": "offers/f4V+/",
    "_ts": 1518498282,
    "content": {
      "offerIsRUPerMinuteThroughputEnabled": false,
      "offerThroughput": 400
    },
    "id": "f4V+",
    "offerResourceId": "Es5SAM2FDwA=",
    "offerType": "Invalid",
    "offerVersion": "V2",
    "resource": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a usar una identidad administrada asignada por el sistema en una máquina virtual Linux para acceder a Cosmos DB.  Para obtener más información sobre Cosmos DB, vea:

> [!div class="nextstepaction"]
>[Introducción a Azure Cosmos DB](../../cosmos-db/introduction.md)