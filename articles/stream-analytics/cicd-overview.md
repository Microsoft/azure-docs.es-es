---
title: Integración e implementación continuas para Azure Stream Analytics
description: En este artículo se proporciona información general sobre una canalización de integración e implementación continuas (CI/CD) para Azure Stream Analytics.
services: stream-analytics
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 9/22/2020
ms.openlocfilehash: ec8f27d0376f7187fd36b3feba556dbced0946e9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931949"
---
# <a name="continuous-integration-and-deployment-cicd-for-azure-stream-analytics"></a>Integración e implementación continuas (CI/CD) para Azure Stream Analytics

Puede implementar el trabajo de Azure Stream Analytics de manera continua mediante la integración de control de código fuente. La integración del control de código fuente habilita un flujo de trabajo en el que una actualización de código desencadena la implementación de un recurso en Azure. En este artículo se describen los pasos básicos para crear una canalización de integración e implementación continuas (CI/CD).

Si no está familiarizado con Azure Stream Analytics, comience con el [Inicio rápido de Azure Stream Analytics](stream-analytics-quick-create-portal.md).

## <a name="create-a-cicd-pipeline"></a>Creación de una canalización de CI/CD

Siga los pasos de esta guía para crear una canalización de CI/CD para Stream Analytics.

1. Cree una consulta de Azure Stream Analytics.

   Use herramientas de Azure Stream Analytics para [Visual Studio Code](quick-create-vs-code.md) o [Visual Studio](stream-analytics-quick-create-vs.md) para [desarrollar y probar consultas localmente](develop-locally.md). También puede [exportar un trabajo existente](visual-studio-code-explore-jobs.md#export-a-job-to-a-local-project) a un proyecto local.

2. Confirme los proyectos de Azure Stream Analytics en el sistema de control de código fuente, como un repositorio de Git.

3. Use [herramientas de CI/CD de Azure Stream Analytics](cicd-tools.md) para compilar los proyectos y generar plantillas de Azure Resource Manager para la implementación.

4. Ejecute [pruebas automatizadas de scripts](cicd-tools.md#automated-test) para la regresión de la calidad.

5. [Implemente el trabajo](cicd-tools.md#deploy-to-azure) en Azure automáticamente.

## <a name="auto-build-test-and-deploy"></a>Compilación, prueba e implementación automáticas

Puede usar la línea de comandos y las [herramientas de CI/CD de Azure Stream Analytics](cicd-tools.md) para realizar de manera automática la compilación, la prueba y la implementación. También puede configurar una canalización de CI/CD en [Azure Pipelines](set-up-cicd-pipeline.md). Azure Pipelines permite habilitar funcionalidades más avanzadas, como la administración de canalizaciones, la visualización y los desencadenadores.

## <a name="next-steps"></a>Pasos siguientes

* [Automatización de las compilaciones, las pruebas y las implementaciones de un trabajo de Azure Stream Analytics mediante herramientas de CI/CD](cicd-tools.md)
* [Configuración de una canalización de CI/CD para un trabajo de Stream Analytics mediante Azure Pipelines](set-up-cicd-pipeline.md)
