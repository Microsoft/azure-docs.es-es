---
title: Desarrollo para Azure NetApp Files con la API REST | Microsoft Docs
description: La API REST para el servicio Azure NetApp Files define las operaciones HTTP para los recursos como la cuenta de NetApp, el grupo de capacidades, los volúmenes y las instantáneas.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/02/2020
ms.author: b-juche
ms.openlocfilehash: 90d3ff807275c74ce4495c5631ab913971ebe3e3
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513016"
---
# <a name="develop-for-azure-netapp-files-with-rest-api"></a>Desarrollo para Azure NetApp Files con la API REST 

La API REST para el servicio Azure NetApp Files define las operaciones HTTP en los recursos como la cuenta de NetApp, el grupo de capacidades, los volúmenes y las instantáneas. Este artículo le ayuda a empezar a usar la API REST de Azure NetApp Files.

## <a name="azure-netapp-files-rest-api-specification"></a>Especificación de la API de REST de Azure NetApp Files

La especificación de la API de REST de Azure NetApp Files se publica a través de [GitHub](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager):

`https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager`


## <a name="access-the-azure-netapp-files-rest-api"></a>Acceso a la API REST de Azure NetApp Files  

1. [Instale la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) si no lo ha hecho ya.
2. Cree una entidad de servicio en su instancia de Azure Active Directory (Azure AD):
   1. Compruebe que tiene [permisos suficientes](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#required-permissions).

   2. En la CLI de Azure, escriba el siguiente comando: 
    
        ```azurecli
        az ad sp create-for-rbac --name $YOURSPNAMEGOESHERE
        ```

      La salida del comando es similar al ejemplo siguiente:  

        ```output
        { 
            "appId": "appIDgoeshere", 
            "displayName": "APPNAME", 
            "name": "http://APPNAME", 
            "password": "supersecretpassword", 
            "tenant": "tenantIDgoeshere" 
        } 
        ```

      Conserve la salida del comando.  Necesitará los valores `appId`, `password` y `tenant`. 

3. Solicite un token de acceso de OAuth:

    Los ejemplos de este artículo utilizan cURL. También puede usar diversas herramientas de API como [Postman](https://www.getpostman.com/), [Insomnia](https://insomnia.rest/) y [Paw](https://paw.cloud/).  

    Reemplace las variables en el ejemplo siguiente con la salida del comando del Paso 2. 
    
    ```azurecli
    curl -X POST -d 'grant_type=client_credentials&client_id=[APP_ID]&client_secret=[PASSWORD]&resource=https%3A%2F%2Fmanagement.azure.com%2F' https://login.microsoftonline.com/[TENANT_ID]/oauth2/token
    ```

    La salida proporciona un token de acceso similar al ejemplo siguiente:

    `eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSIsImtpZCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSJ9`

    El token que se muestra es válido durante 3600 segundos. Pasado este tiempo, tendrá que solicitar un nuevo token. 
    Guarde el token en un editor de texto.  Lo necesitará en el siguiente paso.

4. Envíe una llamada de prueba e incluya el token para validar el acceso a la API REST:

    ```azurecli
    curl -X GET -H "Authorization: Bearer [TOKEN]" -H "Content-Type: application/json" https://management.azure.com/subscriptions/[SUBSCRIPTION_ID]/providers/Microsoft.Web/sites?api-version=2019-11-01
    ```

## <a name="examples-using-the-api"></a>Ejemplos de uso de la API  

Este artículo usa la siguiente dirección URL para la base de referencia de las solicitudes. Esta dirección URL apunta a la raíz del espacio de nombres de Azure NetApp Files. 

`https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts?api-version=2019-11-01`

En los ejemplos siguientes, reemplace los valores `SUBIDGOESHERE` y `RESOURCEGROUPGOESHERE` por los suyos propios. 

### <a name="get-request-examples"></a>Ejemplos de solicitud GET

Use una solicitud GET para consultar objetos de Azure NetApp Files en una suscripción, como se muestra en los ejemplos siguientes: 

```azurecli
#get NetApp accounts 
curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts?api-version=2019-11-01
```

```azurecli
#get capacity pools for NetApp account 
curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools?api-version=2019-11-01
```

```azurecli
#get volumes in NetApp account & capacity pool 
curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE/volumes?api-version=2019-11-01
```

```azurecli
#get snapshots for a volume 
curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE/volumes/VOLUMEGOESHERE/snapshots?api-version=2019-11-01
```

### <a name="put-request-examples"></a>Ejemplos de solicitud PUT

Use una solicitud PUT para crear nuevos objetos de Azure NetApp Files, como se muestra en los ejemplos a continuación. El cuerpo de la solicitud PUT puede incluir los datos con formato JSON para los cambios. Debe incluirse en el comando de CURL como texto o como referencias como archivo. Para hacer referencia al cuerpo como un archivo, guarde el ejemplo de JSON en un archivo y agregue `-d @<filename>` al comando de CURL.

```azurecli
#create a NetApp account  
curl -d @<filename> -X PUT -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE?api-version=2019-11-01
```

```azurecli
#create a capacity pool  
curl -d @<filename> -X PUT -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE?api-version=2019-11-01
```

```azurecli
#create a volume  
curl -d @<filename> -X PUT -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE/volumes/MYNEWVOLUME?api-version=2019-11-01
```

```azurecli
 #create a volume snapshot  
curl -d @<filename> -X PUT -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE/volumes/MYNEWVOLUME/Snapshots/SNAPNAME?api-version=2019-11-01
```

### <a name="json-examples"></a>Ejemplos de JSON

En el ejemplo siguiente se muestra cómo crear una cuenta de NetApp:

```json
{ 
    "name": "MYNETAPPACCOUNT", 
    "type": "Microsoft.NetApp/netAppAccounts", 
    "location": "westus2", 
    "properties": { 
        "name": "MYNETAPPACCOUNT" 
    }
} 
```

En el ejemplo siguiente se muestra cómo crear un grupo de capacidad: 

```json
{
    "name": "MYNETAPPACCOUNT/POOLNAME",
    "type": "Microsoft.NetApp/netAppAccounts/capacityPools",
    "location": "westus2",
    "properties": {
        "name": "POOLNAME",
        "size": "4398046511104",
        "serviceLevel": "Premium"
    }
}
```

En el ejemplo siguiente se muestra cómo crear un nuevo volumen. (El protocolo predeterminado para el volumen es NFSV3). 

```json
{
    "name": "MYNEWVOLUME",
    "type": "Microsoft.NetApp/netAppAccounts/capacityPools/volumes",
    "location": "westus2",
    "properties": {
        "serviceLevel": "Premium",
        "usageThreshold": "322122547200",
        "creationToken": "MY-FILEPATH",
        "snapshotId": "",
        "subnetId": "/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.Network/virtualNetworks/VNETGOESHERE/subnets/MYDELEGATEDSUBNET.sn"
         }
}
```

En el ejemplo siguiente se muestra cómo crear una instantánea de un volumen: 

```json
{
    "name": "apitest2/apiPool01/apiVol01/snap02",
    "type": "Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/Snapshots",
    "location": "westus2",
    "properties": {
         "name": "snap02",
        "fileSystemId": "0168704a-bbec-da81-2c29-503825fe7420"
    }
}
```

> [!NOTE] 
> Tiene que especificar `fileSystemId` para crear una instantánea.  Puede obtener el valor `fileSystemId` con una solicitud GET a un volumen. 

## <a name="next-steps"></a>Pasos siguientes

[Consulte la referencia de la API REST de Azure NetApp Files](https://docs.microsoft.com/rest/api/netapp/)
