---
title: Introducción a Azure DevOps Starter | Microsoft Docs
description: Obtenga información sobre la forma en que Azure DevOps Starter le ayuda a iniciar, implementar y administrar cualquier aplicación desde una vista individual en Azure Portal.
services: devops-project
documentationcenter: ''
author: mlearned
manager: gwallace
editor: mlearned
ms.assetid: ''
ms.service: az-devops-project
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: ''
ms.date: 03/24/2020
ms.author: mlearned
ms.openlocfilehash: 91e62cae242279e1923fc31970c233ec70bf7f11
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461347"
---
# <a name="overview-of-azure-devops-starter"></a>Introducción a Azure DevOps Starter

 Azure DevOps Starter facilita la introducción a Azure. Le ayuda a iniciar su aplicación favorita en el servicio de Azure que elija en unos pocos pasos rápidos desde Azure Portal. 

 DevOps Starter configura todo lo necesario para desarrollar, implementar y supervisar la aplicación. El panel de DevOps Starter se puede usar para supervisar confirmaciones de código, compilaciones e implementaciones, y todo ello, desde una sola vista en Azure Portal.

## <a name="advantages-of-using-devops-starter"></a>Ventajas del uso de DevOps Starter

  DevOps Starter automatiza la configuración de toda una canalización de integración continua (CI) y de entrega continua (CD) en Azure.  Puede empezar con el código existente o usar una de las aplicaciones de ejemplo proporcionadas. Luego puede implementar rápidamente dicha aplicación en varios servicios de Azure como Virtual Machines, App Service, Azure Kubernetes Services (AKS), Azure SQL Database y Azure Service Fabric.  

  DevOps Starter realiza todo el trabajo de la configuración inicial de una canalización de DevOps, lo que incluye configurar el repositorio Git inicial y la canalización de CI/CD, crear un recurso de Application Insights para la supervisión y proporcionar una sola vista de toda la solución con la creación de un panel de DevOps Projects en Azure Portal.

Puede usar DevOps Starter para:

* Implementar rápidamente la aplicación en Azure
* Automatizar la configuración de una canalización de CI/CD
* Ver cómo se configura correctamente una canalización de CI/CD y aprender a hacerlo
* Personalizar aún más las canalizaciones de versión en función de sus escenarios concretos

## <a name="how-to-use-devops-starter"></a>¿Cómo se usa DevOps Starter?

  DevOps Starter está disponible en Azure Portal. Los recursos de DevOps Starter se crean como cualquier otro recurso de Azure desde el portal. DevOps Projects proporciona una experiencia de asistente paso a paso para las distintas opciones de configuración.  

Elija varias opciones de configuración como parte de la instalación inicial. Entre estas opciones se incluyen:

* El uso de la aplicación de ejemplo proporcionada o de su propio código
* Selección del lenguaje de una aplicación
* Elección de un marco de aplicación en función del lenguaje
* Selección de un servicio de Azure (destino de implementación)
* Creación de una organización de Azure DevOps nueva o uso de una existente 
* Elección de una suscripción de Azure
* Elección de la ubicación de los servicios de Azure
* Elección entre varios planes de tarifa de servicios de Azure

Después de usar DevOps Starter también puede eliminar todos los recursos desde un mismo lugar desde el panel de DevOps Starter en Azure Portal.

## <a name="devops-starter-and-azure-devops-integration"></a>Integración de DevOps Starter y Azure DevOps

DevOps Starter cuenta con tecnología de Azure DevOps. DevOps Starter automatiza todo el trabajo que se necesita en Azure Pipelines para configurar una canalización de CI/CD. Crea un repositorio Git en una organización de Azure DevOps nueva o existente y, luego, confirma una aplicación de ejemplo o el código existente en un nuevo repositorio Git.  

La automatización también establece un desencadenador de integración continua para la compilación, con el fin de que cada confirmación de código inicie una compilación. DevOps Starter crea un desencadenador de implementación continua e implementa cada nueva compilación correcta en el servicio de Azure que elija.  

Las canalizaciones de compilación y versión se pueden personalizar para otros escenarios. Además, puede clonar las canalizaciones de compilación y de versión para usarlas en otros proyectos.

Después de crear la instancia de DevOps Starter, también puede:

* Personalizar la canalización de compilación y versión
* Usar solicitudes de incorporación de cambios para administrar el flujo de código y mantener un alto nivel de calidad
* Probar y compilar cada confirmación antes de fusionar el código para elevar el listón de calidad
* Realizar un seguimiento de los problemas y el trabajo pendiente junto con la aplicación

## <a name="getting-started-with-devops-starter"></a>Introducción a DevOps Starter

* [Introducción a DevOps Starter](https://docs.microsoft.com/azure/devops-project/azure-devops-project-github)

##  <a name="devops-starter-videos"></a>Vídeos de DevOps Starter

* [Creación de CI/CD con Azure DevOps Starter](https://www.youtube.com/watch?v=NuYDAs3kNV8)
