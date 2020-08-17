---
title: Desarrollo y depuración local de trabajos de Azure Stream Analytics
description: Obtenga información acerca de cómo desarrollar y probar trabajos de Azure Stream Analytics en el equipo local antes de ejecutarlos en Azure Portal.
ms.author: mamccrea
author: mamccrea
ms.topic: conceptual
ms.date: 03/31/2020
ms.service: stream-analytics
ms.openlocfilehash: 70097ad02c17f8c23bd9613487a905ccd562635e
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337558"
---
# <a name="develop-and-debug-azure-stream-analytics-jobs-locally"></a>Desarrollo y depuración local de trabajos de Azure Stream Analytics

Aunque puede crear y probar trabajos de Azure Stream Analytics en Azure Portal, muchos desarrolladores prefieren una experiencia de desarrollo local. Stream Analytics le permite usar el editor de código y las herramientas de desarrollo que prefiera para crear y probar trabajos con secuencias de eventos en directo desde Azure Event Hubs, IoT Hub y Blob Storage con un entorno de tiempo de ejecución local de un solo nodo completamente funcional. También puede enviar trabajos a Azure directamente desde el entorno de desarrollo local.

## <a name="local-development-environments"></a>Entornos de desarrollo locales

La forma en que desarrolle trabajos de Stream Analytics en el equipo local depende de sus preferencias de herramientas y de la disponibilidad de características. Consulte [Comparación de características de Azure Stream Analytics](feature-comparison.md) para ver qué características son compatibles con cada entorno de desarrollo.

Los entornos de la siguiente tabla admiten el desarrollo local:

|Entorno                              |Descripción    |
|-----------------------------------------|------------|
|[Visual Studio Code](visual-studio-code-explore-jobs.md)| La [extensión Azure Stream Analytics Tools](https://marketplace.visualstudio.com/items?itemName=ms-bigdatatools.vscode-asa) para Visual Studio Code permite crear, administrar y probar un trabajo de Stream Analytics tanto a nivel local como en la nube con IntelliSense enriquecido y control de código fuente nativo. Es compatible con el desarrollo en Linux, macOS y Windows. Para más información, consulte [Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-vs-code.md). La extensión también admite [Visual Studio Codespaces](https://visualstudio.microsoft.com/services/visual-studio-codespaces/), que es un entorno de desarrollo hospedado en la nube.|
|[Visual Studio 2019](stream-analytics-tools-for-visual-studio-install.md) |Stream Analytics Tools forma parte de las cargas de trabajo de almacenamiento y procesamiento de datos y desarrollo de Azure en Visual Studio. Puede usar Visual Studio para escribir deserializadores y funciones personalizadas definidas por el usuario de C#. Para más información, consulte [Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md).|
|[Símbolo del sistema o terminal](stream-analytics-tools-for-visual-studio-cicd.md)|El paquete NuGet de CI/CD de Azure Stream Analytics proporciona herramientas para la compilación de proyectos de Visual Studio y para la realización de pruebas locales en una máquina arbitraria. El paquete NPM de CI/CD de Azure Stream Analytics proporciona herramientas para compilaciones de proyectos de Visual Studio Code (que generan una plantilla de Azure Resource Manager) en una máquina arbitraria.|

## <a name="next-steps"></a>Pasos siguientes

* [Prueba de consultas de Stream Analytics localmente con datos de ejemplo mediante Visual Studio Code](visual-studio-code-local-run.md)
* [Prueba de las consultas de Stream Analytics localmente en una entrada de streaming en vivo con Visual Studio Code](visual-studio-code-local-run-live-input.md)
* [Prueba de las consultas de Stream Analytics localmente con Visual Studio](stream-analytics-vs-tools-local-run.md)
* [Prueba local de datos activos mediante las herramientas de Azure Stream Analytics para Visual Studio](stream-analytics-live-data-local-testing.md)
