---
title: 'Conexión a la API de Azure Media Services v3: Python'
description: En este artículo se muestra cómo conectarse a la API de Media Services v3 con Python.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/15/2019
ms.author: juliako
ms.custom: devx-track-python
ms.openlocfilehash: 834d87f607ca65a9d9e0ebee12d2772f5baf7949
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87849648"
---
# <a name="connect-to-media-services-v3-api---python"></a>Conexión a la API de Media Services v3: Python

En este artículo se muestra cómo conectarse al SDK de Python de Azure Media Services v3 con el método de inicio de sesión de la entidad de servicio.

## <a name="prerequisites"></a>Prerrequisitos

- Descargue Python desde [python.org](https://www.python.org/downloads/).
- Asegúrese de establecer la variable de entorno `PATH`.
- [Cree una cuenta de Media Services](./create-account-howto.md). Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services.
- Siga los pasos descritos en el tema [Acceso a la API](./access-api-howto.md). Registre el identificador de suscripción, el identificador de aplicación (el identificador de cliente), la clave de autenticación (el secreto) y el identificador de inquilino que necesitará en el paso posterior.

> [!IMPORTANT]
> Revise las [convenciones de nomenclatura](media-services-apis-overview.md#naming-conventions).

## <a name="install-the-modules"></a>Instalación de los módulos

Para trabajar con Azure Media Services mediante Python, debe instalar estos módulos.

* El módulo `azure-mgmt-resource`, que incluye los módulos de Azure para Active Directory.
* El módulo `azure-mgmt-media`, que incluye las entidades de Media Services.

Abra una herramienta de línea de comandos y use los comandos siguientes para instalar los módulos.

```
pip3 install azure-mgmt-resource
pip3 install azure-mgmt-media==1.1.1
```

## <a name="connect-to-the-python-client"></a>Conexión con el cliente de Python

1. Cree un archivo con una extensión `.py`.
1. Abra el archivo en el editor que prefiera.
1. Agregue el código siguiente al archivo. El código importa los módulos necesarios y crea el objeto de credenciales de Active Directory que necesita para conectarse a Media Services.

      Establezca los valores de las variables en los valores que obtuvo de [Acceso a la API](./access-api-howto.md).

      ```
      import adal
      from msrestazure.azure_active_directory import AdalAuthentication
      from msrestazure.azure_cloud import AZURE_PUBLIC_CLOUD
      from azure.mgmt.media import AzureMediaServices
      from azure.mgmt.media.models import MediaService

      RESOURCE = 'https://management.core.windows.net/'
      # Tenant ID for your Azure Subscription
      TENANT_ID = '00000000-0000-0000-000000000000'
      # Your Service Principal App ID
      CLIENT = '00000000-0000-0000-000000000000'
      # Your Service Principal Password
      KEY = '00000000-0000-0000-000000000000'
      # Your Azure Subscription ID
      SUBSCRIPTION_ID = '00000000-0000-0000-000000000000'
      # Your Azure Media Service (AMS) Account Name
      ACCOUNT_NAME = 'amsv3account'
      # Your Resource Group Name
      RESOUCE_GROUP_NAME = 'AMSv3ResourceGroup'

      LOGIN_ENDPOINT = AZURE_PUBLIC_CLOUD.endpoints.active_directory
      RESOURCE = AZURE_PUBLIC_CLOUD.endpoints.active_directory_resource_id

      context = adal.AuthenticationContext(LOGIN_ENDPOINT + '/' + TENANT_ID)
      credentials = AdalAuthentication(
          context.acquire_token_with_client_credentials,
          RESOURCE,
          CLIENT,
          KEY
      )

      # The AMS Client
      # You can now use this object to perform different operations to your AMS account.
      client = AzureMediaServices(credentials, SUBSCRIPTION_ID)

      print("signed in")

      # Now that you are authenticated, you can manipulate the entities.
      # For example, list assets in your AMS account
      print (client.assets.list(RESOUCE_GROUP_NAME, ACCOUNT_NAME).get(0))
      ```

1. Ejecute el archivo.

## <a name="next-steps"></a>Pasos siguientes

- Use el [SDK de Python](https://aka.ms/ams-v3-python-sdk).
- Revise la documentación de [referencia de Python](https://aka.ms/ams-v3-python-ref) de Media Services.
