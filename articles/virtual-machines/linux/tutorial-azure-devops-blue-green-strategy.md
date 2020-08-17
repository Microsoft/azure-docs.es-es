---
title: 'Tutorial: Configuración de implementaciones controladas para Azure Linux Virtual Machines'
description: En este tutorial, aprenderá a configurar una canalización de implementación continua (CD). Esta canalización actualiza un grupo de máquinas virtuales Linux en Azure con la estrategia de implementación azul-verde.
author: moala
manager: jpconnock
tags: azure-devops-pipelines
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: azure-pipelines
ms.workload: infrastructure
ms.date: 4/10/2020
ms.author: moala
ms.custom: devops
ms.openlocfilehash: 0d001144f1a2703db118261e5cae5417b1d8c17a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080138"
---
# <a name="tutorial---configure-the-blue-green-deployment-strategy-for-azure-linux-virtual-machines"></a>Tutorial: Configuración de la estrategia de implementación azul-verde para Azure Linux Virtual Machines

## <a name="infrastructure-as-a-service-iaas---configure-cicd"></a>Infraestructura como servicio (IaaS): configuración de CI/CD

Azure Pipelines ofrece un conjunto completo de herramientas de automatización de CI/CD para las implementaciones en máquinas virtuales. En Azure Portal, se puede configurar directamente una canalización de entrega continua para una máquina virtual de Azure.

En este artículo, se muestra cómo configurar una canalización de CI/CD que utiliza la estrategia azul-verde para la implementación de varias máquinas. Azure Portal también admite otras estrategias, como [gradual](https://aka.ms/AA7jlh8) y [controlada](https://aka.ms/AA7jdrz).

### <a name="configure-cicd-on-virtual-machines"></a>Configuración de CI/CD en máquinas virtuales

Las máquinas virtuales se pueden agregar como destinos a un [grupo de implementación](/azure/devops/pipelines/release/deployment-groups). Posteriormente, se pueden establecer como destino para las actualizaciones de varias máquinas. Después de efectuar la implementación en las máquinas, vea el **historial de implementación** para un grupo de implementación. Esta vista le permite hacer un seguimiento de la máquina virtual hasta la canalización y, posteriormente, hasta la confirmación.

### <a name="blue-green-deployments"></a>Implementaciones azul-verde

Las implementaciones azul-verde reducen el tiempo de inactividad, ya que tienen un entorno en espera idéntico. Solo hay un entorno activo en cada momento.

Mientras se prepara para una nueva versión, puede realizar la fase final de las pruebas en el entorno verde. Una vez que el software funcione en el entorno verde, cambie el tráfico para que todas las solicitudes entrantes vayan a este. El entorno azul está inactivo.

Puede configurar implementaciones azul-verde en sus "máquinas virtuales" con la opción de entrega continua de Azure Portal. A continuación, se incluyen los pasos detallados:

1. Inicie sesión en Azure Portal y desplácese hasta una máquina virtual.
1. En el panel de la izquierda de la configuración de la máquina virtual, seleccione **Entrega continua**. A continuación, seleccione **Configurar**.

   ![Panel de entrega continua con el botón Configurar](media/tutorial-devops-azure-pipelines-classic/azure-devops-configure.png)

1. En el panel de configuración, haga clic en **Organización de Azure DevOps** para seleccionar una cuenta o crear una. Después, seleccione el proyecto en el que desea configurar la canalización.  

   ![Panel de entrega continua](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling.png)

1. Un grupo de implementación es un conjunto lógico de máquinas de destino de implementación que representan los entornos físicos. Por ejemplo, Dev, Test, UAT y Production. Puede crear un grupo de implementación o seleccionar uno existente.
1. Seleccione la canalización de compilación que publica el paquete que se va a implementar en la máquina virtual. Tenga en cuenta que el paquete publicado debe tener un script de implementación llamado deploy.ps1 o deploy.sh en la carpeta deployscripts de la carpeta raíz del paquete. La canalización ejecuta este script de implementación.
1. En **Deployment strategy** (Estrategia de implementación), seleccione **Blue-Green** (Azul-Verde).
1. Agregue una etiqueta "blue" (azul) o "green" (verde) a las máquinas virtuales que van a formar parte de las implementaciones azul-verde. Si la máquina virtual es para un rol en espera, etiquétela como "green" (verde). De lo contrario, etiquétela como "blue" (azul).

   ![Panel de entrega continua, con el valor azul-verde elegido como estrategia de implementación](media/tutorial-devops-azure-pipelines-classic/azure-devops-blue-green-configure.png)

1. Seleccione **Aceptar** para configurar la canalización de entrega continua que va a implementar en la máquina virtual.

   ![La canalización azul-verde](media/tutorial-devops-azure-pipelines-classic/azure-devops-blue-green-pipeline.png)

1. Se muestran los detalles de la implementación de la máquina virtual. Puede seleccionar el vínculo para ir a la canalización de versión en Azure DevOps. En la canalización de versión, seleccione **Editar** para ver la configuración de canalización. La canalización tiene estas tres fases:

   1. Esta fase es una fase de grupo de implementación. Las aplicaciones se implementan en máquinas virtuales en espera, las cuales se etiquetan como "verdes".
   1. En esta fase, la canalización se detiene y espera una intervención manual para reanudar la ejecución. Los usuarios pueden reanudar la ejecución de la canalización una vez que hayan asegurado manualmente la estabilidad de la implementación en las máquinas virtuales etiquetadas como "verdes".
   1. Esta fase intercambia las etiquetas "azul" y "verde" en las máquinas virtuales. Esto garantiza que las máquinas virtuales con versiones anteriores de la aplicación se etiqueten ahora como "verdes". Durante la siguiente ejecución de la canalización, las aplicaciones se implementarán en estas máquinas virtuales.

      ![El panel Grupo de implementación de la tarea Implementación azul-verde](media/tutorial-devops-azure-pipelines-classic/azure-devops-blue-green-tasks.png)

1. La tarea Execute Deploy Script (Ejecutar script de implementación) ejecuta de forma predeterminada el script de implementación deploy.ps1 or deploy.sh. El script se encuentra en la carpeta deployscripts de la carpeta raíz del paquete publicado. Asegúrese de que la canalización de compilación seleccionada publica la implementación en la carpeta raíz del paquete.

   ![Panel Artefactos, que muestra el script deploy.sh en la carpeta deployscripts](media/tutorial-deployment-strategy/package.png)

## <a name="other-deployment-strategies"></a>Otras estrategias de implementación

- [Configuración de la estrategia de implementación gradual](https://aka.ms/AA7jlh8)
- [Configuración de la estrategia de implementación controlada](https://aka.ms/AA7jdrz)

## <a name="azure-devops-projects"></a>Azure DevOps Projects

Se puede comenzar a utilizar Azure fácilmente. En Azure DevOps Projects, puede empezar a ejecutar su aplicación en cualquier servicio de Azure con solo tres pasos. Para ello, seleccione:

- Un idioma de la aplicación
- Un entorno de ejecución
- Un servicio de Azure

[Más información](https://azure.microsoft.com/features/devops-projects/).

## <a name="additional-resources"></a>Recursos adicionales

- [Implementación de una aplicación de ASP.NET en máquinas virtuales de Azure mediante Azure DevOps Starter](../../devops-project/azure-devops-project-vms.md)
- [Implementación continua de una aplicación en un conjunto de escalado de máquinas virtuales de Azure](/azure/devops/pipelines/apps/cd/azure/deploy-azure-scaleset)
