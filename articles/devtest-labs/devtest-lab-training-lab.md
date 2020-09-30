---
title: Uso de Azure DevTest Labs para entrenamiento | Microsoft Docs
description: En este artículo se proporcionan pasos detallados que puede seguir para configurar un laboratorio para entrenamiento en Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b2de9550c62f04286a4f9ad42238bfefb9846477
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294856"
---
# <a name="use-azure-devtest-labs-for-training"></a>Uso de Azure DevTest Labs para entrenamiento
Azure DevTest Labs se puede utilizar para implementar muchos escenarios claves además de los de desarrollo y pruebas. Uno de estos escenarios es la configuración de un laboratorio para entrenamiento. Azure DevTest Labs le permite crear un laboratorio en el que puede proporcionar plantillas personalizadas que cada aprendiz puede usar para crear entornos idénticos y aislados con fines de entrenamiento. Puede aplicar directivas para garantizar que los entornos de entrenamiento estarán disponibles para cada aprendiz solo cuando sea necesario y cuando contengan los suficientes recursos necesarios como, por ejemplo, máquinas virtuales, para el entrenamiento. Por último, puede compartir fácilmente el laboratorio con los aprendices y estos podrán acceder fácilmente a él con un solo clic.

![Uso de DevTest Labs para entrenamiento](./media/devtest-lab-training-lab/devtest-lab-training.png)

Azure DevTest Labs cumple los siguientes requisitos necesarios para llevar a cabo el entrenamiento en cualquier entorno virtual: 

* Los aprendices no pueden ver las máquinas virtuales creadas por otros aprendices
* Cada máquina de entrenamiento debe ser idéntica
* Los aprendices pueden aprovisionar rápidamente sus entornos de entrenamiento
* Se pueden controlar los costos garantizando que los aprendices no obtengan más máquinas virtuales de las necesarias para el entrenamiento y también apagando estas cuando no estén en uso
* Se puede compartir fácilmente el laboratorio de entrenamiento con cada aprendiz
* El laboratorio de entrenamiento se puede volver a usar una y otra vez

En este artículo, aprenderá acerca de las diversas características de Azure DevTest Labs que se pueden utilizar para cumplir los requisitos de entrenamiento descritos anteriormente y los pasos detallados que puede seguir para configurar un laboratorio para el entrenamiento.  

## <a name="implementing-training-with-azure-devtest-labs"></a>Implementación de un entrenamiento con Azure DevTest Labs
1. **Creación del laboratorio** 
   
    Los laboratorios son el punto de partida en Azure DevTest Labs. Una vez creado un laboratorio, puede realizar tareas tales como agregar usuarios (aprendices) al laboratorio, establecer directivas para controlar los costos, definir imágenes de máquinas virtuales que se pueden crear rápidamente y muchas más.   
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Creación de un laboratorio con Laboratorios de desarrollo y pruebas de Azure](devtest-lab-create-lab.md) |Aprenderá a crear un laboratorio en Azure DevTest Labs en Azure Portal. |
2. **Creación de máquinas virtuales de entrenamiento en cuestión de minutos mediante imágenes listas para usar de Marketplace e imágenes personalizadas** 
   
    Puede elegir imágenes listas para usar a partir de una gran variedad de imágenes de Azure Marketplace y hacer que estas estén disponibles para los aprendices del laboratorio. Si estas imágenes no satisfacen sus necesidades, puede crear una imagen personalizada mediante la creación de una máquina virtual de laboratorio con una imagen lista para usar de Azure Marketplace. Para ello, instale todo el software que necesita para el entrenamiento y guarde la máquina virtual como imagen personalizada en el laboratorio. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Configuración de imágenes de Azure Marketplace](devtest-lab-configure-marketplace-images.md) |Aprenda a permitir imágenes de Azure Marketplace y haga que estén disponibles para su selección solo las imágenes que desee para el entrenamiento. |
   | [Creación de una imagen personalizada](devtest-lab-create-template.md) |Cree una imagen personalizada instalando previamente el software que necesita para el entrenamiento para que los aprendices puedan crear rápidamente una máquina virtual con la imagen personalizada. |
3. **Creación de plantillas reutilizables para las máquinas de entrenamiento** 
   
    Una fórmula en Azure DevTest Labs es una lista de valores de propiedad predeterminados que se usan para crear una máquina virtual. Puede crear una fórmula en el laboratorio seleccionando una imagen, un tamaño de máquina virtual (una combinación de CPU y RAM) y una red virtual. Cada aprendiz puede ver la fórmula en el laboratorio y usarla para crear una máquina virtual. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Administración de fórmulas de DevTest Labs para crear máquinas virtuales](devtest-lab-manage-formulas.md) |Aprenda cómo crear una fórmula en el laboratorio seleccionando una imagen, un tamaño de máquina virtual (una combinación de CPU y RAM) y una red virtual. |
4. **Control de los costos**
   
    Azure DevTest Labs le permite establecer una directiva en el laboratorio para especificar el número máximo de máquinas virtuales que un aprendiz puede crear en el laboratorio. 
   
    Si está realizando un entrenamiento de varios días y desea detener todas las máquinas virtuales en un momento determinado del día y, posteriormente, reiniciarlas automáticamente al día siguiente, lo puede hacer fácilmente mediante el establecimiento de directivas de apagado e inicio automático en el laboratorio. 
   
    Por último, al finalizar el entrenamiento puede eliminar todas las máquinas virtuales a la vez mediante la ejecución de un sencillo script de PowerShell. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Definición de directivas de laboratorio](devtest-lab-set-lab-policy.md) |Controle los costos mediante el establecimiento de directivas en el laboratorio. |
   | [Eliminación de todas las máquinas virtuales del laboratorio mediante un script de PowerShell](devtest-lab-faq.md#how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab) |Elimine todos los laboratorios en una sola operación al finalizar el entrenamiento. |
5. **Uso compartido del laboratorio con cada aprendiz**
   
    Se puede acceder directamente a los laboratorios mediante un vínculo que puede compartir con los aprendices. Ni siquiera es necesario que estos tengan una cuenta de Azure, siempre que dispongan de una [cuenta Microsoft](devtest-lab-faq.md#what-is-a-microsoft-account). Los aprendices no pueden ver las máquinas virtuales creadas por otros aprendices.  
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Incorporación de un aprendiz a un laboratorio de Azure DevTest Labs](devtest-lab-add-devtest-user.md) |Utilice Azure Portal para agregar aprendices al laboratorio de entrenamiento. |
   | [Incorporación de aprendices al laboratorio mediante un script de PowerShell](devtest-lab-add-devtest-user.md#add-an-external-user-to-a-lab-using-powershell) |Use PowerShell para automatizar la incorporación de aprendices al laboratorio de entrenamiento. |
   | [Obtención de un vínculo al laboratorio](devtest-lab-faq.md#how-do-i-share-a-direct-link-to-my-lab) |Aprenda cómo se puede acceder directamente a un laboratorio a través de un hipervínculo. |
6. **Reutilización sin límites del laboratorio** 
   
    Puede automatizar la creación de laboratorios, incluso con una configuración personalizada, mediante la creación de una plantilla de Resource Manager que puede utilizar para crear laboratorios idénticos una y otra vez. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Creación de un laboratorio mediante una plantilla de Resource Manager](devtest-lab-faq.md#how-do-i-create-a-lab-from-a-resource-manager-template) |Cree laboratorios en Azure DevTest Labs mediante plantillas de Resource Manager. |

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]
