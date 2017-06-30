---
title: "Creación de una aplicación web HTML estática en Azure en cinco minutos | Microsoft Docs"
description: "Aprenda lo fácil que es ejecutar aplicaciones web en App Service mediante la implementación de una aplicación de ejemplo."
services: app-service\web
documentationcenter: 
author: rick-anderson
manager: wpickett
editor: 
ms.assetid: 60495cc5-6963-4bf0-8174-52786d226c26
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 05/08/2017
ms.author: riande
ms.custom: mvc
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 895906e1ab4bc50093ed3b18f043c3dd515ca054
ms.contentlocale: es-es
ms.lasthandoff: 05/10/2017

---
# <a name="create-a-static-html-web-app-in-azure-in-five-minutes"></a>Creación de una aplicación web HTML estática en Azure en cinco minutos

En esta guía de inicio rápido se explica cómo se implementa un sitio HTML+CSS básico en Azure. Se ejecutará la aplicación mediante un [plan de Azure App Service](https://docs.microsoft.com/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview) y se creará una aplicación web en ella mediante la [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli). Para implementar la aplicación en Azure, se usa Git. Una vez instalados los requisitos previos, tardará aproximadamente cinco minutos en completar el tutorial.

![hello-world-in-browser](media/app-service-web-get-started-html/hello-world-in-browser-az.png)

## <a name="prerequisites"></a>Requisitos previos

Antes de crear este ejemplo, descargue e instale los componentes siguientes:

- [Git](https://git-scm.com/)
- [CLI de Azure 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="download-the-sample"></a>Descarga del ejemplo

En una ventana del terminal, clone el repositorio de la aplicación de ejemplo en la máquina local:

```bash
git clone https://github.com/Azure-Samples/html-docs-hello-world.git
```

## <a name="view-the-html"></a>Ver el código HTML

Vaya al directorio que contiene el código HTML de ejemplo. Abra el archivo *index.html* en el explorador.

![hello-world-in-browser](media/app-service-web-get-started-html/hello-world-in-browser.png)

[!INCLUDE [login-to-azure](../../includes/login-to-azure.md)] 
[!INCLUDE [configure-deployment-user](../../includes/configure-deployment-user.md)] 

[!INCLUDE [app-service-web-quickstart1](../../includes/app-service-web-quickstart1.md)] 

Cree una [aplicación web](app-service-web-overview.md) en el plan de App Service de `quickStartPlan`. La aplicación web ofrece un espacio de hospedaje para el código y proporciona una dirección URL para que pueda ver la aplicación implementada.

[!INCLUDE [app-service-web-quickstart2](../../includes/app-service-web-quickstart2.md)] 

La página se ejecuta como una aplicación web de Azure App Service:

![hello-world-in-browser](media/app-service-web-get-started-html/hello-world-in-browser-az.png)

## <a name="update-and-redeploy-the-app"></a>Actualización de la aplicación y nueva implementación

Abra el archivo *index.html* . Realice un cambio en el marcado. Por ejemplo, cambie `Hello world!` a `Hello Azure!`.

Confirme los cambios en Git y, después, inserte los cambios de código en Azure.

```bash
git commit -am "HTML actualizado"
git push azure master
```

Una vez completada la implementación, actualice el explorador para ver los cambios.

[!INCLUDE [manage-azure-web-app](../../includes/manage-azure-web-app.md)]


[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>Pasos siguientes

- Explore [scripts de ejemplo de la CLI para Web Apps](app-service-cli-samples.md).
- Obtenga información sobre cómo [asignar un nombre de dominio personalizado](app-service-web-tutorial-custom-domain.md), por ejemplo, contoso.com, a una [aplicación de App Service](app-service-web-tutorial-custom-domain.md).
