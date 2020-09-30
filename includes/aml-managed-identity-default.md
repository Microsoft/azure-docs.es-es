---
title: archivo de inclusión
description: archivo de inclusión
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 08/24/2020
ms.openlocfilehash: aa8aea66c5481454f0d7d4d118934f5fbf34a911
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89147285"
---
La **identidad administrada predeterminada** es la identidad administrada asignada por el sistema o la primera identidad administrada asignada por el usuario.

Durante una ejecución, hay dos aplicaciones de una identidad:

1. El sistema usa una identidad para configurar los montajes de almacenamiento, el registro de contenedor y los almacenes de datos del usuario.

    * En este caso, el sistema usará la identidad administrada predeterminada.

1. El usuario aplica una identidad para acceder a los recursos desde dentro del código para una ejecución enviada.

    * En este caso, proporcione el elemento *client_id* correspondiente a la identidad administrada que quiere usar para recuperar una credencial.
    * También puede obtener el id. de cliente de la identidad asignada por el usuario a través de la variable de entorno *DEFAULT_IDENTITY_CLIENT_ID*.

    Por ejemplo, para recuperar un token para un almacén de datos con la identidad administrada predeterminada:

    ```python
    client_id = os.environ.get('DEFAULT_IDENTITY_CLIENT_ID')
    credential = ManagedIdentityCredential(client_id=client_id)
    token = credential.get_token('https://storage.azure.com/')
    ```