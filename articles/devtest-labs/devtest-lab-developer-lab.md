---
title: Uso de Azure DevTest Labs para desarrolladores | Microsoft Docs
description: Aprenderá acerca de las diversas características de Azure DevTest Labs que se pueden utilizar para cumplir los requisitos del desarrollador y los pasos detallados que puede seguir para configurar un laboratorio.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b4f195f843e30ad1a56bad7d5b9ca9e73888783b
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89296250"
---
# <a name="use-azure-devtest-labs-for-developers"></a>Uso de Azure DevTest Labs para desarrolladores
Se puede utilizar Azure DevTest Labs para implementar numerosos escenarios clave pero uno de los principales escenarios implica el uso de DevTest Labs para máquinas de desarrollo host para desarrolladores. En este escenario, DevTest Labs ofrece estas ventajas:

- Los desarrolladores pueden aprovisionar rápidamente las máquinas de desarrollo a petición.
- Los desarrolladores pueden personalizar fácilmente las máquinas de desarrollo, siempre que sea necesario.
- Los administradores pueden controlar los costos garantizando que:
  - Los desarrolladores no puedan obtener más máquinas virtuales de las que necesitan para el desarrollo.
  - Las máquinas virtuales se apaguen cuando no estén en uso. 

![Uso de DevTest Labs para entrenamiento](./media/devtest-lab-developer-lab/devtest-lab-developer-lab.png)

En este artículo, aprenderá acerca de las diversas características de Azure DevTest Labs que se pueden utilizar para cumplir los requisitos del desarrollador y los pasos detallados que puede seguir para configurar un laboratorio.

## <a name="implementing-developer-environments-with-azure-devtest-labs"></a>Implementación de entornos de desarrollo con Azure DevTest Labs
1. **Creación del laboratorio** 
   
    Los laboratorios son el punto de partida en Azure DevTest Labs. Una vez creado un laboratorio, puede realizar tareas tales como agregar usuarios (desarrolladores) al laboratorio, establecer directivas para controlar los costos, definir imágenes de máquinas virtuales que se pueden crear rápidamente y muchas más.  
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Creación de un laboratorio con Laboratorios de desarrollo y pruebas de Azure](devtest-lab-create-lab.md) |Aprenderá a crear un laboratorio en Azure DevTest Labs en Azure Portal. |
2. **Creación de máquinas virtuales en cuestión de minutos mediante imágenes listas para usar de Marketplace e imágenes personalizadas** 
   
    Puede elegir imágenes listas para usar a partir de una gran variedad de imágenes de Azure Marketplace y hacer que estas estén disponibles en el laboratorio. Si estas imágenes no satisfacen sus necesidades, puede crear una imagen personalizada mediante la creación de una máquina virtual de laboratorio con una imagen lista para usar de Azure Marketplace. Para ello, instale todo el software que necesite y guarde la máquina virtual como imagen personalizada en el laboratorio.

    Si va a usar imágenes personalizadas, considere la posibilidad de usar un generador de imágenes para crear y distribuir imágenes. Un generador de imágenes es una solución de configuración como código que crea y distribuye periódicamente las imágenes configuradas de forma automática. Así se reduce el tiempo necesario para configurar manualmente el sistema después de que se haya creado una máquina virtual con el sistema operativo base.
  
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Configuración de imágenes de Azure Marketplace](devtest-lab-configure-marketplace-images.md) |Aprenda a permitir imágenes de Azure Marketplace y haga que estén disponibles para su selección solo las imágenes que desee para los desarrolladores.|
   | [Creación de una imagen personalizada](devtest-lab-create-template.md) |Cree una imagen personalizada instalando previamente el software que necesita para que los desarrolladores puedan crear rápidamente una máquina virtual con la imagen personalizada.|
   | [Información acerca del generador de imágenes](./devtest-lab-faq.md#blog-post) |Vea un vídeo que describe cómo configurar y usar un generador de imágenes.|

3. **Creación de plantillas reutilizables para las máquinas de desarrollo** 
   
    Una fórmula en Azure DevTest Labs es una lista de valores de propiedad predeterminados que se usan para crear una máquina virtual. Puede crear una fórmula en el laboratorio seleccionando una imagen, un tamaño de máquina virtual (una combinación de CPU y RAM) y una red virtual. Cada desarrollador puede ver la fórmula en el laboratorio y usarla para crear una máquina virtual. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Administración de fórmulas de DevTest Labs para crear máquinas virtuales](devtest-lab-manage-formulas.md) |Aprenda cómo crear una fórmula en el laboratorio seleccionando una imagen, un tamaño de máquina virtual (una combinación de CPU y RAM) y una red virtual.|

4. **Creación de artefactos para habilitar la personalización de máquinas virtuales flexibles**

   Los artefactos se utilizan para implementar y configurar la aplicación después de aprovisionar una máquina virtual. Los artefactos pueden ser:

   - Herramientas que desea instalar en la máquina virtual, como agentes, Fiddler y Visual Studio.
   - Acciones que desea ejecutar en la máquina virtual, como la clonación de un repositorio.
   - Aplicaciones que desea probar.

   Hay numerosos artefactos disponibles de serie. Puede crear sus propios artefactos personalizados si desea personalizar aún más para sus necesidades específicas.

   Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Creación de artefactos personalizados para la máquina virtual de DevTest Labs](devtest-lab-artifact-author.md) |Cree sus propios artefactos personalizados para las máquinas virtuales de su laboratorio.|
   | [Agregar un repositorio de Git para almacenar artefactos personalizados y plantillas de Azure Resource Manager para su uso en Azure DevTest Labs](devtest-lab-add-artifact-repo.md) |Aprenda a almacenar los artefactos personalizados en su propio repositorio privado de Git.|

5. **Control de los costos**
   
    Azure DevTest Labs le permite establecer una directiva en el laboratorio para especificar el número máximo de máquinas virtuales que un desarrollador puede crear en el laboratorio. 
   
    Si el equipo de desarrolladores tiene una programación de trabajo configurada y usted desea detener todas las máquinas virtuales en un momento determinado del día y, posteriormente, reiniciarlas automáticamente al día siguiente, lo puede hacer fácilmente mediante el establecimiento de directivas de apagado e inicio automático en el laboratorio. 
   
    Por último, al finalizar el desarrollo de aplicaciones puede eliminar todas las máquinas virtuales a la vez mediante la ejecución de un sencillo script de PowerShell. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Definición de directivas de laboratorio](devtest-lab-set-lab-policy.md) |Controle los costos mediante el establecimiento de directivas en el laboratorio. |
   | [Eliminación de todas las máquinas virtuales del laboratorio mediante un script de PowerShell](devtest-lab-faq.md#how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab) |Elimine todos los laboratorios en una sola operación al finalizar el desarrollo.|

1. **Incorporación de una red virtual a una máquina virtual** 
   
    DevTest Labs crea una red virtual (VNET) cada vez que se crea un laboratorio. Si ha configurado su propia red virtual (por ejemplo, mediante el uso de VPN de sitio a sitio o ExpressRoute), puede agregar esta red virtual a la configuración de red virtual de su laboratorio para que esté disponible al crear máquinas virtuales.

    Además, hay un artefacto de unión a un dominio de Azure Active Directory disponible que unirá una máquina virtual a un dominio cuando se crea la máquina virtual. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Configuración de una red virtual en Azure DevTest Labs](devtest-lab-configure-vnet.md) |Aprenda a configurar una red virtual en Azure DevTest Labs mediante Azure Portal.|

6. **Uso compartido del laboratorio con cada desarrollador**
   
    Se puede acceder directamente a los laboratorios mediante un vínculo que puede compartir con los desarrolladores. Ni siquiera es necesario que tengan una cuenta de Azure, siempre y cuando dispongan de una [cuenta de Microsoft](devtest-lab-faq.md#what-is-a-microsoft-account). Los desarrolladores no pueden ver las máquinas virtuales creadas por otros desarrolladores.  
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Incorporación de un desarrollador a un laboratorio de Azure DevTest Labs](devtest-lab-add-devtest-user.md) |Utilice Azure Portal para agregar desarrolladores al laboratorio.|
   | [Incorporación de desarrolladores al laboratorio mediante un script de PowerShell](devtest-lab-add-devtest-user.md#add-an-external-user-to-a-lab-using-powershell) |Use PowerShell para automatizar la incorporación de desarrolladores al laboratorio. |
   | [Obtención de un vínculo al laboratorio](devtest-lab-faq.md#how-do-i-share-a-direct-link-to-my-lab) |Aprenda cómo pueden los desarrolladores tener acceso directamente a un laboratorio a través de un hipervínculo.|

7. **Automatización de la creación de un laboratorio para más equipos** 
   
    Puede automatizar la creación de laboratorios, incluso con una configuración personalizada, mediante la creación de una plantilla de Resource Manager que puede utilizar para crear laboratorios idénticos una y otra vez. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Creación de un laboratorio mediante una plantilla de Resource Manager](devtest-lab-faq.md#how-do-i-create-a-lab-from-a-resource-manager-template) |Cree laboratorios en Azure DevTest Labs mediante plantillas de Resource Manager. |

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]
