---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: d669874040b04af089b4d57333869fe34066720e
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87841846"
---
La manera más sencilla de autenticar una aplicación basada en la nube es con una identidad administrada. Consulte [Uso de identidades administradas de App Service para acceder a Azure Key Vault](/azure/key-vault/general/managed-identity) para más información. 

Sin embargo, en aras de la simplicidad, en este inicio rápido se crea una aplicación de escritorio, que requiere el uso de una entidad de servicio y una directiva de control de acceso. La entidad de servicio requiere un nombre único con el formato "http://&lt;my-unique-service-principal-name&gt;".

Cree una entidad de servicio mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la CLI de Azure:

```azurecli
az ad sp create-for-rbac -n "http://<my-unique-service-principal-name>" --sdk-auth
```

Esta operación devolverá una serie de pares clave-valor.

```console
{
  "clientId": "7da18cae-779c-41fc-992e-0527854c6583",
  "clientSecret": "b421b443-1669-4cd7-b5b1-394d5c945002",
  "subscriptionId": "443e30da-feca-47c4-b68f-1636b75e16b3",
  "tenantId": "35ad10f1-7799-4766-9acf-f2d946161b77",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```
