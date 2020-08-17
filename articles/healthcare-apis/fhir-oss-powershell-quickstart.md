---
title: 'PowerShell: Implementación del servidor de FHIR para Azure: Azure API for FHIR'
description: En este inicio rápido se explica cómo implementar el servidor de FHIR de código abierto de Microsoft mediante PowerShell.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: c133a309cc85ffcfb69be2ae7bbb614cbb540f2e
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87847149"
---
# <a name="quickstart-deploy-open-source-fhir-server-using-powershell"></a>Inicio rápido: Implementación del servidor de FHIR de código abierto mediante PowerShell

En este inicio rápido aprenderá a implementar el servidor de FHIR de código abierto de Microsoft para Azure mediante PowerShell.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Elija un nombre para el grupo de recursos que contendrá los recursos aprovisionados y créelo:

```azurepowershell-interactive
$fhirServiceName = "MyFhirService"
$rg = New-AzResourceGroup -Name $fhirServiceName -Location westus2
```

## <a name="deploy-the-fhir-server-template"></a>Implementación de la plantilla del servidor de FHIR

El [repositorio de GitHub](https://github.com/Microsoft/fhir-server) del servidor de FHIR de Microsoft para Azure contiene una plantilla que implementará todos los recursos necesarios. Impleméntela con:

```azurepowershell-interactive
New-AzResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Microsoft/fhir-server/master/samples/templates/default-azuredeploy.json -ResourceGroupName $rg.ResourceGroupName -serviceName $fhirServiceName
```

## <a name="verify-fhir-server-is-running"></a>Comprobación de que el servidor de FHIR está en ejecución

```azurepowershell-interactive
$metadataUrl = "https://" + $fhirServiceName + ".azurewebsites.net/metadata" 
$metadata = Invoke-WebRequest -Uri $metadataUrl
$metadata.RawContent
```

El servidor tardará un minuto aproximadamente en responder la primera vez.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine el grupo de recursos mediante los siguientes pasos:

```azurepowershell-interactive
Remove-AzResourceGroup -Name $rg.ResourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha implementado el servidor de FHIR de código abierto de Microsoft para Azure en su suscripción. Para obtener información sobre cómo acceder a la API de FHIR con Postman, continúe con el tutorial de Postman.
 
>[!div class="nextstepaction"]
>[Acceso a la API de FHIR con Postman](access-fhir-postman-tutorial.md)
