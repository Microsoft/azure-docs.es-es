---
title: CI/CD para Azure Spring Cloud
description: CI/CD para Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 10/04/2019
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: d31be45f5cf608b7dfd727741de68544574d7f8a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496464"
---
# <a name="cicd-for-azure-spring-cloud"></a>CI/CD para Azure Spring Cloud

Las herramientas de integración continua y entrega continua permiten a los desarrolladores implementar rápidamente las actualizaciones en las aplicaciones existentes con el mínimo esfuerzo y riesgo. Azure DevOps ayuda a organizar y controlar estos trabajos clave. Actualmente, Azure Spring Cloud no ofrece un complemento de Azure DevOps específico.  Sin embargo, puede integrar las aplicaciones Spring Cloud con DevOps mediante una [tarea de la CLI de Azure](https://docs.microsoft.com/azure/devops/pipelines/tasks/deploy/azure-cli?view=azure-devops). En este artículo se muestra cómo usar una tarea de la CLI de Azure con Azure Spring Cloud para la integración con Azure DevOps.

## <a name="create-an-azure-resource-manager-service-connection"></a>Creación de una conexión de servicio de Azure Resource Manager

Lea [este artículo](https://docs.microsoft.com/azure/devops/pipelines/library/connect-to-azure?view=azure-devops) para aprender a crear una conexión de servicio de Azure Resource Manager con su proyecto de Azure DevOps. Asegúrese de seleccionar la misma suscripción que usa para la instancia de servicio de Azure Spring Cloud.

## <a name="azure-cli-task-templates"></a>Plantillas de tareas de la CLI de Azure

### <a name="deploy-artifacts"></a>Implementación de artefactos

Puede compilar e implementar los proyectos mediante una serie de `tasks`. En primer lugar, este fragmento de código define una tarea de Maven para compilar la aplicación, seguida de una segunda tarea que implementa el archivo JAR con la extensión de la CLI de Azure para Azure Spring Cloud.

```yaml
steps:
- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
- task: AzureCLI@1
  inputs:
    azureSubscription: <your service connection name>
    scriptLocation: inlineScript
    inlineScript: |
      az extension add -y --name spring-cloud
      az spring-cloud app deploy --resource-group <your-resource-group> --service <your-spring-cloud-service> --name <app-name> --jar-path ./target/your-result-jar.jar
      # deploy other app
```

### <a name="deploy-from-source"></a>Implementación desde el origen

Se puede implementar directamente en Azure sin un paso de compilación independiente.

```yaml
- task: AzureCLI@1
  inputs:
    azureSubscription: <your service connection name>
    scriptLocation: inlineScript
    inlineScript: |
      az extension add -y --name spring-cloud
      az spring-cloud app deploy --resource-group <your-resource-group> --service <your-spring-cloud-service> --name <app-name>

      # or if it is a multi-module project
      az spring-cloud app deploy --resource-group <your-resource-group> --service <your-spring-cloud-service> --name <app-name> --target-module relative/path/to/module
```
