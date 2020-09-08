---
title: ¿Qué es Azure Dev Spaces?
services: azure-dev-spaces
ms.date: 05/07/2019
ms.topic: overview
description: Aprenda la forma en que Azure Dev Spaces proporciona una experiencia de desarrollo de Kubernetes iterativa y rápida para los equipos de los clústeres de Azure Kubernetes Service
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, containers, kubectl, k8s
manager: gwallace
ms.openlocfilehash: ad884d5e054f23269b238b791e250c732d70a9b8
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88210648"
---
# <a name="what-is-azure-dev-spaces"></a>¿Qué es Azure Dev Spaces?

Azure Dev Spaces proporciona una experiencia de desarrollo de Kubernetes iterativa y rápida para los equipos de los clústeres de Azure Kubernetes Service (AKS). Azure Dev Spaces también permite depurar y probar todos los componentes de la aplicación en AKS con la configuración mínima del equipo de desarrollo, sin replicar ni simular las dependencias.

![En el diagrama se muestran dos versiones de una aplicación desarrolladas de manera independiente. A continuación, se combinan en una en un entorno de desarrollo de Azure Dev Spaces.](media/azure-dev-spaces/collaborate-graphic.gif)

## <a name="how-azure-dev-spaces-simplifies-kubernetes-development"></a>Cómo simplifica Azure Dev Spaces el desarrollo de Kubernetes

Azure Dev Spaces ayuda a los equipos a centrarse en el desarrollo y la iteración rápida de su aplicación de microservicio, ya que les permite trabajar directamente con su arquitectura de microservicios completa o con la aplicación entera que se ejecuta en AKS. Azure Dev Spaces también proporciona una manera de actualizar de forma independiente partes de la arquitectura de microservicios de forma aislada sin afectar al resto del clúster de AKS o a otros desarrolladores. Azure Dev Spaces se usa para desarrollo y pruebas en entornos de desarrollo y pruebas de bajo nivel y no va dirigido a la ejecución en clústeres de AKS de producción.

Puesto que los equipos pueden trabajar con la aplicación entera y colaborar directamente en AKS, Azure Dev Spaces permite:

* Reducir la instalación en la máquina local
* Disminuir el tiempo de instalación para los nuevos desarrolladores del equipo
* Aumentar la velocidad de un equipo gracias a una iteración más rápida
* Reducir el número de entornos de desarrollo e integración redundantes puesto que los miembros del equipo pueden compartir un clúster
* Eliminar la necesidad de replicar o simular las dependencias
* Mejorar la colaboración entre los equipos de desarrollo y los equipos con los que trabajan, como pueden ser los equipos de DevOps

Azure Dev Spaces proporciona herramientas para generar recursos de Docker y Kubernetes para sus proyectos. Estas herramientas le permiten agregar fácilmente las aplicaciones nuevas y existentes a un espacio de desarrollo y a otros clústeres de AKS.

Para más información sobre cómo funciona Azure Dev Spaces, consulte [How Azure Dev Spaces works and is configured][how-dev-spaces-works] (Funcionamiento y configuración de Azure Dev Spaces).

## <a name="supported-regions-and-configurations"></a>Configuraciones y regiones admitidas

Azure Dev Spaces solo es compatible con los clústeres de AKS en [algunas regiones][supported-regions]. Azure Dev Spaces admite el uso de la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) o [Visual Studio Code](https://code.visualstudio.com/download) con la [extensión de Azure Dev Spaces](https://marketplace.visualstudio.com/items?itemName=azuredevspaces.azds) instalada en Linux, macOS o Windows 8 (o alguna versión posterior) para compilar y ejecutar las aplicaciones en AKS. También admite el uso de [Visual Studio 2019](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs) instalado en Windows con la carga de trabajo de desarrollo de Azure.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo funciona Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Funcionamiento de Azure Dev Spaces](how-dev-spaces-works.md)

[how-dev-spaces-works]: how-dev-spaces-works.md
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
