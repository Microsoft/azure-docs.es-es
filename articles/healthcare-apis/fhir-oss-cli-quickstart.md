---
title: 'CLI de Azure: Implementación del servidor de FHIR de código abierto para Azure: Azure API for FHIR'
description: En este inicio rápido se explica cómo implementar el servidor de FHIR de código abierto de Microsoft para Azure.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 02/07/2019
ms.author: matjazl
ms.custom: devx-track-azurecli
ms.openlocfilehash: 10af71afd8843e75d5df3be57c909c56a7abca01
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87843596"
---
# <a name="quickstart-deploy-open-source-fhir-server-using-azure-cli"></a>Inicio rápido: Implementación del servidor de FHIR de código abierto mediante la CLI de Azure

En este inicio rápido va a aprender a implementar un servidor de FHIR&reg; de código abierto en Azure mediante la CLI de Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-resource-group"></a>Creación de un grupo de recursos

Elija un nombre para el grupo de recursos que contendrá los recursos aprovisionados y créelo:

```azurecli-interactive
servicename="myfhirservice"
az group create --name $servicename --location westus2
```

## <a name="deploy-template"></a>Implementar plantilla

El [repositorio de GitHub](https://github.com/Microsoft/fhir-server) del servidor de FHIR de Microsoft para Azure contiene una plantilla que implementará todos los recursos necesarios. Impleméntela con:

```azurecli-interactive
az group deployment create -g $servicename --template-uri https://raw.githubusercontent.com/Microsoft/fhir-server/master/samples/templates/default-azuredeploy.json --parameters serviceName=$servicename
```

## <a name="verify-fhir-server-is-running"></a>Comprobación de que el servidor de FHIR está en ejecución

Obtenga una instrucción de funcionalidad del servidor de FHIR con:

```azurecli-interactive
metadataurl="https://${servicename}.azurewebsites.net/metadata"
curl --url $metadataurl
```

El servidor tardará un minuto aproximadamente en responder la primera vez.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine el grupo de recursos mediante los siguientes pasos:

```azurecli-interactive
az group delete --name $servicename
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha implementado el servidor de FHIR de código abierto de Microsoft para Azure en su suscripción. Para obtener información sobre cómo acceder a la API de FHIR con Postman, continúe con el tutorial de Postman.
 
>[!div class="nextstepaction"]
>[Acceso a la API de FHIR con Postman](access-fhir-postman-tutorial.md)
