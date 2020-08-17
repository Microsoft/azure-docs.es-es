---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: 6dfc96df3623e44a4ba513c238f21482c2a2989d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87013137"
---
El método DefaultAzureCredential de nuestra aplicación depende de tres variables de entorno: `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` y `AZURE_TENANT_ID`. Establezca estas variables en los valores clientId, clientSecret y tenantId que se devolvieron en el paso "Creación de una entidad de servicio" con el formato `export VARNAME=VALUE`. (Este método solo establece las variables para el shell actual y los procesos creados desde el shell. Para agregar de forma permanente estas variables a su entorno, edite el archivo `/etc/environment `). 

También tendrá que guardar el nombre del almacén de claves como una variable de entorno llamada `KEY_VAULT_NAME`.

```console
export AZURE_CLIENT_ID=<your-clientID>

export AZURE_CLIENT_SECRET=<your-clientSecret>

export AZURE_TENANT_ID=<your-tenantId>

export KEY_VAULT_NAME=<your-key-vault-name>
````
