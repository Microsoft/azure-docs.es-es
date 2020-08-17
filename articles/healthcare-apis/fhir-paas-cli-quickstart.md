---
title: 'Inicio rápido: Implementación de Azure API for FHIR mediante la CLI de Azure'
description: En este inicio rápido va a aprender a implementar Azure API for FHIR en Azure mediante la CLI de Azure.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 10/15/2019
ms.author: matjazl
ms.custom: devx-track-azurecli
ms.openlocfilehash: e1aa3362217a4edcfcf547c69b9effde00459f7e
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87850566"
---
# <a name="quickstart-deploy-azure-api-for-fhir-using-azure-cli"></a>Inicio rápido: Implementación de Azure API for FHIR mediante la CLI de Azure

En este inicio rápido va a aprender a implementar Azure API for FHIR en Azure mediante la CLI de Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="add-healthcareapis-extension"></a>Incorporación de la extensión HealthcareAPIs

```azurecli-interactive
az extension add --name healthcareapis
```

Obtenga una lista de comandos para HealthcareAPIs:

```azurecli-interactive
az healthcareapis --help
```

## <a name="create-azure-resource-group"></a>Creación del grupo de recursos de Azure

Elija un nombre para el grupo de recursos que contendrá Azure API for FHIR y créelo:

```azurecli-interactive
az group create --name "myResourceGroup" --location westus2
```

## <a name="deploy-the-azure-api-for-fhir"></a>Implementación de Azure API for FHIR

```azurecli-interactive
az healthcareapis create --resource-group myResourceGroup --name nameoffhiraccount --kind fhir-r4 --location westus2 
```

## <a name="fetch-fhir-api-capability-statement"></a>Captura de la instrucción de funcionalidad de la API de FHIR

Obtenga una instrucción de funcionalidad de la API de FHIR con:

```azurecli-interactive
curl --url "https://nameoffhiraccount.azurehealthcareapis.com/metadata"
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine el grupo de recursos mediante los siguientes pasos:

```azurecli-interactive
az group delete --name "myResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado Azure API for FHIR en su suscripción. Para establecer opciones de configuración adicionales en Azure API for FHIR, continúe con la guía paso a paso de configuración adicional.

>[!div class="nextstepaction"]
>[Configuración adicional en Azure API for FHIR](azure-api-for-fhir-additional-settings.md)
