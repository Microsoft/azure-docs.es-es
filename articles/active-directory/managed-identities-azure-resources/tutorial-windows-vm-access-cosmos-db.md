---
title: 'Tutorial: Uso de identidades administradas para acceder a Azure Cosmos DB (Windows): Azure AD'
description: Este tutorial le guía por el proceso de usar una identidad administrada asignada por el sistema en una máquina virtual Windows para acceder a Azure Cosmos DB.
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
ms.date: 01/14/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 17cdebb1291f78706178e129a62b932d45f38537
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89263083"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-cosmos-db"></a>Tutorial: Uso de las identidades administradas asignadas por el sistema de una máquina virtual Windows para acceder a Azure Cosmos DB

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo usar una identidad administrada asignada por el sistema en una máquina virtual Windows para acceder a Cosmos DB. Aprenderá a:

> [!div class="checklist"]
> * Creación de una cuenta de Cosmos DB
> * Conceder acceso a la identidad administrada asignada por el sistema en la máquina virtual Windows a las claves de acceso de la cuenta de Cosmos DB
> * Obtener un token de acceso mediante una identidad administrada asignada por el sistema de la máquina virtual Windows para llamar a Azure Resource Manager
> * Obtención de las claves de acceso desde Azure Resource Manager para realizar llamadas a Cosmos DB

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [Instalar la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps)


## <a name="enable"></a>Habilitar

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]



## <a name="grant-access"></a>Conceder acceso


### <a name="create-a-cosmos-db-account"></a>Creación de una cuenta de Cosmos DB 

Si aún no tiene una, cree una cuenta de Cosmos DB. También puede omitir este paso y usar una cuenta de Cosmos DB existente. 

1. Haga clic en el botón **+/Crear nuevo servicio** de la esquina superior izquierda de Azure Portal.
2. Haga clic en **Bases de datos** y, a continuación, en **Azure Cosmos DB** para mostrar el panel "Nueva cuenta".
3. Escriba un **identificador** para la cuenta de Cosmos DB, el cual se utilizará más adelante.  
4. **API** se debe establecer en "SQL". El enfoque descrito en este tutorial se puede utilizar con los otros tipos de API disponibles, pero los pasos de este tutorial son para la API de SQL.
5. Asegúrese de que **Suscripción** y **Grupo de recursos** coinciden con los que especificó cuando creó la máquina virtual en el paso anterior.  Seleccione una **Ubicación** en la que Cosmos DB esté disponible.
6. Haga clic en **Crear**.

### <a name="create-a-collection"></a>Creación de una colección 

A continuación, agregue una colección de datos en la cuenta de Cosmos DB que podrá consultar en pasos posteriores.

1. Vaya a la cuenta de Cosmos DB recién creada.
2. En la pestaña **Información general**, haga clic en el botón **+/Agregar colección** y aparecerá un panel "Agregar colección".
3. Proporcione para la colección un identificador de base de datos, el identificador de la colección, seleccione una capacidad de almacenamiento, escriba una clave de partición, escriba un valor de rendimiento y, luego, haga clic en **Aceptar**.  Para este tutorial, es suficiente con utilizar "Test" como identificador de la base de datos e identificador de la colección, seleccionar una capacidad de almacenamiento fijo y el rendimiento más bajo (400 RU/s).  


### <a name="grant-access-to-the-cosmos-db-account-access-keys"></a>Concesión de acceso a las claves de acceso de la cuenta de Cosmos DB

En esta sección se muestra cómo conceder acceso a la identidad administrada asignada por el sistema en la máquina virtual Windows a las claves de acceso de la cuenta de Cosmos DB. Cosmos DB no admite la autenticación de Azure AD de forma nativa. No obstante, puede usar una identidad administrada asignada por el sistema para recuperar una clave de acceso de Cosmos DB desde Resource Manager y usar dicha clave para acceder a Cosmos DB. En este paso, va a conceder a la identidad administrada asignada por el sistema de la máquina virtual Windows acceso a las claves de la cuenta de Cosmos DB.

Para conceder a la identidad administrada asignada por el sistema de la máquina virtual Windows acceso a la cuenta de Cosmos DB en Azure Resource Manager mediante PowerShell, actualice los valores de `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` y `<COSMOS DB ACCOUNT NAME>` para su entorno. Cosmos DB admite dos niveles de granularidad en las claves de acceso: acceso de lectura y escritura a la cuenta y acceso de solo lectura a la cuenta.  Asigne el rol `DocumentDB Account Contributor` si desea obtener claves de lectura y escritura para la cuenta o bien asigne el rol `Cosmos DB Account Reader Role` si desea obtener claves de solo lectura para la cuenta.  Para este tutorial, asigne `Cosmos DB Account Reader Role`:

```azurepowershell
$spID = (Get-AzVM -ResourceGroupName myRG -Name myVM).identity.principalid
New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Cosmos DB Account Reader Role" -Scope "/subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDb/databaseAccounts/<COSMOS DB ACCOUNT NAME>"
```
## <a name="access-data"></a>Acceso a los datos

En esta sección se muestra cómo llamar a Azure Resource Manager con un token de acceso para la identidad administrada asignada por el sistema de la máquina virtual Windows. En el resto del tutorial, vamos a trabajar desde la máquina virtual que se creó anteriormente. 

Tiene que instalar la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli) en la máquina virtual Windows.



### <a name="get-an-access-token"></a>Obtención de un token de acceso

1. En Azure Portal, vaya a **Máquinas virtuales**, vaya a la máquina virtual Windows y, a continuación, desde la página **Información general**, haga clic en **Conectar** en la parte superior. 
2. Escriba su **nombre de usuario** y **contraseña** que agregó cuando creó la máquina virtual Windows. 
3. Ahora que ha creado una **conexión a Escritorio remoto** con la máquina virtual, abra PowerShell en la sesión remota.
4. Mediante el comando Invoke-WebRequest de Powershell, realice una solicitud al punto de conexión local de Managed Identities for Azure Resources y obtenga un token de acceso para Azure Resource Manager.

   ```powershell
   $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Method GET -Headers @{Metadata="true"}
   ```

   > [!NOTE]
   > El valor del parámetro "resource" debe coincidir exactamente con el que Azure AD espera. Al usar el id. de recurso de Azure Resource Manager, debe incluir la barra diagonal final en el URI.
    
   A continuación, extraiga el elemento "Content", que se almacena como una cadena con formato de notación de objetos JavaScript (JSON) en el objeto $response. 
    
   ```powershell
   $content = $response.Content | ConvertFrom-Json
   ```
   Luego, extraiga el token de acceso de la respuesta.
    
   ```powershell
   $ArmToken = $content.access_token
   ```

### <a name="get-access-keys"></a>Obtención de las claves de acceso 

En esta sección se muestra cómo obtener las claves de acceso desde Azure Resource Manager para realizar llamadas a Cosmos DB. Ahora, utilice PowerShell para llamar a Resource Manager mediante el token de acceso que se recuperó en la sección anterior, para recuperar la clave de acceso de la cuenta de Cosmos DB. Una vez que tenemos la clave de acceso, podemos realizar consultas en Cosmos DB. Asegúrese de reemplazar los valores de los parámetros `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` y `<COSMOS DB ACCOUNT NAME>` con sus propios valores. Reemplace el valor de `<ACCESS TOKEN>` por el token de acceso que se recuperó anteriormente.  Si quiere recuperar claves de lectura y escritura, use el tipo de operación de claves `listKeys`.  Si quiere recuperar claves de solo lectura, use el tipo de operación de claves `readonlykeys`:

```powershell
Invoke-WebRequest -Uri 'https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.DocumentDb/databaseAccounts/<COSMOS DB ACCOUNT NAME>/listKeys/?api-version=2016-03-31' -Method POST -Headers @{Authorization="Bearer $ARMToken"}
```
La respuesta proporciona la lista de claves.  Por ejemplo, si obtiene las claves de solo lectura:

```powershell
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


## <a name="disable"></a>Disable

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad asignada por el sistema de una máquina virtual Windows para acceder a Cosmos DB.  Para obtener más información sobre Cosmos DB, vea:

> [!div class="nextstepaction"]
>[Introducción a Azure Cosmos DB](../../cosmos-db/introduction.md)