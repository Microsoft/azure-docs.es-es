---
title: 'Azure Portal: Implementación del servidor de FHIR de código abierto para Azure: Azure API for FHIR'
description: En este inicio rápido se explica cómo implementar el servidor de FHIR de código abierto de Microsoft mediante Azure Portal.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 7fa119db0c974c93aff667060d153b21b8de16bb
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87843494"
---
# <a name="quickstart-deploy-open-source-fhir-server-using-azure-portal"></a>Inicio rápido: Implementación del servidor de FHIR de código abierto mediante Azure Portal

En este inicio rápido va a aprender a implementar un servidor de FHIR de código abierto en Azure mediante Azure Portal. Usaremos vínculos de implementación sencillos del [repositorio de código abierto](https://github.com/Microsoft/fhir-server).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="github-open-source-repository"></a>Repositorio de código abierto de GitHub

Vaya a la [página de implementación de GitHub](https://github.com/Microsoft/fhir-server/blob/master/docs/DefaultDeployment.md) y busque los botones "Implementar en Azure":

![Página Implementación de código abierto](media/quickstart-oss-portal/deployment-page-oss.png)

Haga clic en el botón de implementación y se abrirá Azure Portal.

## <a name="fill-in-deployment-parameters"></a>Rellenar los parámetros de implementación

Cree un grupo de recursos y asígnele un nombre. El otro único parámetro necesario es un nombre para el servicio.

![Parámetros de implementación personalizada](media/quickstart-oss-portal/deployment-custom-parameters.png)

Tenga en cuenta que la implementación extraerá el código fuente directamente del repositorio de código abierto de GitHub. Si ha bifurcado el repositorio, puede apuntar al suyo propio y a una rama específica.

Después de rellenar los detalles, puede iniciar la implementación.

## <a name="validate-fhir-server-is-running"></a>Comprobación de que el servidor de FHIR está en ejecución

Una vez finalizada la implementación, puede ir mediante el explorador a `https://SERVICENAME.azurewebsites.net/metadata` para obtener una instrucción de funcionalidad. El servidor tardará un minuto aproximadamente en responder la primera vez.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar el grupo de recursos y todos los recursos relacionados. Para hacerlo, seleccione el grupo de recursos que contiene los recursos aprovisionados, seleccione **Eliminar grupo de recursos** y confirme el nombre del grupo de recursos que se va a eliminar.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha implementado el servidor de FHIR de código abierto de Microsoft para Azure en su suscripción. Para obtener información sobre cómo acceder a la API de FHIR con Postman, continúe con el tutorial de Postman.
 
>[!div class="nextstepaction"]
>[Acceso a la API de FHIR con Postman](access-fhir-postman-tutorial.md)