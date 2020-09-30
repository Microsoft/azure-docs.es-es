---
title: 'Tutorial: Configuración de implementaciones graduales para máquinas virtuales Linux en Azure'
description: En este tutorial, aprenderá a configurar una canalización de implementación continua (CD). Esta canalización actualiza incrementalmente un grupo de máquinas virtuales Linux en Azure mediante la estrategia de implementación gradual.
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
ms.openlocfilehash: 6cc4fdb82e98c30ca33eec58675f2175dfcc2607
ms.sourcegitcommit: f845ca2f4b626ef9db73b88ca71279ac80538559
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89612788"
---
# <a name="tutorial---configure-the-rolling-deployment-strategy-for-azure-linux-virtual-machines"></a>Tutorial: Configuración de una estrategia de implementación gradual para máquinas virtuales de Linux en Azure

Azure DevOps es un servicio integrado de Azure que automatiza todas las partes del proceso de DevOps para los recursos de Azure. Independientemente de si su aplicación utiliza máquinas virtuales, aplicaciones web, Kubernetes o cualquier otro recurso, puede implementar la infraestructura como código (IaC), además de la integración, prueba, entrega y supervisión continuas con Azure y Azure DevOps.

![Azure Portal con Azure DevOps seleccionado como servicio](media/tutorial-devops-azure-pipelines-classic/azdevops-view.png)

## <a name="infrastructure-as-a-service-iaas---configure-cicd"></a>Infraestructura como servicio (IaaS): configuración de CI/CD

Azure Pipelines ofrece un conjunto completo de herramientas de automatización de CI/CD para las implementaciones en máquinas virtuales. En Azure Portal, se puede configurar directamente una canalización de entrega continua para una máquina virtual de Azure.

En este artículo, se muestra cómo configurar una canalización de CI/CD para implementaciones graduales de varias máquinas desde Azure Portal. Azure Portal también admite otras estrategias, como las implementaciones [controladas](https://aka.ms/AA7jdrz) y [azul-verde](https://aka.ms/AA83fwu).

### <a name="configure-cicd-on-virtual-machines"></a>Configuración de CI/CD en máquinas virtuales

Las máquinas virtuales se pueden agregar como destinos a un [grupo de implementación](/azure/devops/pipelines/release/deployment-groups). Posteriormente, se pueden establecer como destino para actualizaciones de varias máquinas. Después de efectuar la implementación en las máquinas, vea el **historial de implementación** para un grupo de implementación. Esta vista le permite hacer un seguimiento de la máquina virtual hasta la canalización y, posteriormente, hasta la confirmación.

### <a name="rolling-deployments"></a>Implementaciones graduales

En cada iteración, una implementación gradual reemplaza las instancias de la versión anterior de una aplicación. Se reemplazan por instancias de la nueva versión para un conjunto fijo de máquinas (conjunto de implementación gradual). La siguiente guía paso a paso muestra cómo configurar una actualización gradual para las máquinas virtuales.

Mediante la opción de entrega continua, se pueden configurar las actualizaciones graduales para sus máquinas virtuales en Azure Portal. A continuación, se incluyen los pasos detallados:

1. Inicie sesión en Azure Portal y desplácese hasta una máquina virtual.
1. En el panel de la izquierda de la configuración de la máquina virtual, seleccione **Entrega continua**. A continuación, seleccione **Configurar**.

   ![Panel de entrega continua con el botón Configurar](media/tutorial-devops-azure-pipelines-classic/azure-devops-configure.png)

1. En el panel de configuración, haga clic en **Organización de Azure DevOps** para seleccionar una cuenta existente o crear una nueva. Después, seleccione el proyecto en el que desea configurar la canalización.  

   ![El panel de entrega continua](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling.png)

1. Un grupo de implementación es un conjunto lógico de máquinas de destino de implementación que representan los entornos físicos. Por ejemplo, Dev, Test, UAT y Production. Puede crear un grupo de implementación o seleccionar uno existente.
1. Seleccione la canalización de compilación que publica el paquete que se va a implementar en la máquina virtual. Tenga en cuenta que el paquete publicado debe tener un script de implementación llamado deploy.ps1 o deploy.sh en la carpeta deployscripts de la carpeta raíz del paquete. La canalización ejecuta este script de implementación.
1. En **Deployment strategy** (Estrategia de implementación), seleccione **Rolling** (Gradual).
1. Como alternativa, puede etiquetar cada máquina con el rol correspondiente. Las etiquetas "web" y "db" se utilizan como ejemplo. Estas etiquetas le ayudan a dirigirse solo a las máquinas virtuales que tienen un rol específico.
1. Seleccione **Aceptar** para configurar la canalización de entrega continua.
1. Una vez finalizada la configuración, dispondrá de una canalización de entrega continua para implementarla en la máquina virtual.  

   ![El panel de entrega continua que muestra el historial de implementaciones](media/tutorial-devops-azure-pipelines-classic/azure-devops-deployment-history.png)

1. Se muestran los detalles de la implementación de la máquina virtual. Puede seleccionar el vínculo para ir a la canalización, **Release-1** para ver la implementación, o bien **Edit** (Editar) para modificar la definición de la canalización de lanzamiento.

1. Si va a configurar varias máquinas virtuales, repita los pasos 2 a 4 con otras máquinas virtuales que vaya a agregar al grupo de implementación. Si selecciona un grupo de implementación que ya tiene una ejecución de la canalización, las máquinas virtuales se agregarán al grupo de implementación. No se crearán nuevas canalizaciones.
1. Una vez finalizada la configuración, seleccione la definición de la canalización, vaya a la organización de Azure DevOps y haga clic en **Edit** (Editar) para editar la canalización de versión.

   ![Edición de la canalización gradual](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling-pipeline.png)

1. Seleccione **1 job, 1 task** (1 trabajo, 1 tarea) en la fase **dev**. Seleccione la fase **Deploy** (Implementación).

   ![Tareas de canalización gradual con la tarea de implementación seleccionada](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling-pipeline-tasks.png)

1. En el panel de configuración situado más a la derecha, puede especificar el número de máquinas que desea implementar en paralelo en cada iteración. Si desea hacer la implementación en varias máquinas a la vez, puede especificarlo en forma de porcentaje mediante el control deslizante.  

1. La tarea Execute Deploy Script (Ejecutar script de implementación) ejecutará de forma predeterminada el script de implementación deploy.ps1 o deploy.sh. El script se encuentra en la carpeta deployscripts de la carpeta raíz del paquete publicado.

   ![Panel Artefactos, que muestra el script deploy.sh en la carpeta deployscripts](media/tutorial-deployment-strategy/package.png)

## <a name="other-deployment-strategies"></a>Otras estrategias de implementación

- [Configuración de la estrategia de implementación controlada](https://aka.ms/AA7jdrz)
- [Configuración de la estrategia de implementación azul-verde](https://aka.ms/AA83fwu)

## <a name="azure-devops-projects"></a>Azure DevOps Projects

Se puede comenzar a utilizar Azure fácilmente. En Azure DevOps Projects, puede empezar a ejecutar su aplicación en cualquier servicio de Azure con solo tres pasos. Para ello, seleccione:

- Un idioma de la aplicación
- Un entorno de ejecución
- Un servicio de Azure
 
[Más información](https://azure.microsoft.com/features/devops-projects/).
 
## <a name="additional-resources"></a>Recursos adicionales

- [Implementación de una aplicación de ASP.NET en máquinas virtuales de Azure mediante Azure DevOps Starter](../../devops-project/azure-devops-project-vms.md)
- [Implementación continua de una aplicación en un conjunto de escalado de máquinas virtuales de Azure](/azure/devops/pipelines/apps/cd/azure/deploy-azure-scaleset)
