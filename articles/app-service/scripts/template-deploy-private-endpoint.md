---
title: Uso de una plantilla de Azure Resource Manager para implementar un punto de conexión privado para una aplicación web
description: Aprenda a usar una plantilla de ARM para implementar un punto de conexión privado para la aplicación web.
author: ericgre
ms.assetid: 49e460d0-7759-4ceb-b5a4-f1357e4fde56
ms.topic: sample
ms.date: 07/08/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: acf7198477ce0c55f9321fd3f8e57294a60c0961
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89652027"
---
# <a name="create-an-app-service-app-and-deploy-a-private-endpoint-by-using-an-azure-resource-manager-template"></a>Creación de una aplicación de App Service e implementación de un punto de conexión privado mediante una plantilla de Azure Resource Manager

En esta guía de inicio rápido, usará una plantilla de Azure Resource Manager (ARM) para crear una aplicación web y exponerla con un punto de conexión privado.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisite"></a>Requisito previo

Necesita una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado

Esta plantilla crea un punto de conexión privado para una aplicación web de Azure.

### <a name="review-the-template"></a>Revisión de la plantilla

:::code language="json" source="~/quickstart-templates/101-private-endpoint-webapp/azuredeploy.json" :::

### <a name="deploy-the-template"></a>Implementación de la plantilla

A continuación, se muestra cómo implementar la plantilla de Azure Resource Manager en Azure:

1. Seleccione este vínculo para iniciar sesión en Azure y abrir la plantilla:  [Implementarse en Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-webapp%2Fazuredeploy.json). La plantilla crea la red virtual, la aplicación web, el punto de conexión privado y la zona DNS privada.
2. Elija su grupo de recursos o cree uno.
3. Escriba el nombre de la aplicación web, el plan de Azure App Service y el punto de conexión privado.
5. Lea la declaración de los términos y condiciones. Si está de acuerdo con ellos, seleccione **Acepto los términos y condiciones indicados anteriormente** > **Comprar**. La implementación puede tardar unos minutos en finalizar.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con el punto de conexión privado, elimine el grupo de recursos. Esta acción quita el punto de conexión privado y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

- Puede encontrar más plantillas de Azure Resource Manager para aplicaciones web de Azure App Service en [Ejemplos de plantillas de ARM](../samples-resource-manager-templates.md).
