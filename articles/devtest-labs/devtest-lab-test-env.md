---
title: Usar Azure DevTest Labs para entornos de prueba de máquina virtual y PaaS | Microsoft Docs
description: Aprenda a usar Azure DevTest Labs para escenarios de entornos de prueba de máquina virtual y PaaS.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 9207c070c13c5e276a6ab95e3bcf1bb8671dc382
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87272904"
---
# <a name="use-azure-devtest-labs-for-vm-and-paas-test-environments"></a>Usar Azure DevTest Labs para entornos de prueba de máquina virtual y PaaS

Puede usar Azure DevTest Labs para implementar numerosos escenarios clave, aunque un escenario principal implica el uso de DevTest Labs para hospedar máquinas para evaluadores. 

En este escenario, DevTest Labs ofrece estas ventajas:

- Los evaluadores pueden probar la versión más reciente de la aplicación aprovisionando rápidamente entornos de Windows y Linux mediante plantillas y artefactos reutilizables.
- Los evaluadores pueden escalar verticalmente la prueba de carga aprovisionando varios agentes de prueba.
- Los administradores pueden controlar los costos garantizando que:
  - Los evaluadores no puedan obtener más máquinas virtuales de las que necesitan.
  - Las máquinas virtuales se apaguen cuando no estén en uso.

![Uso de DevTest Labs para entrenamiento](./media/devtest-lab-developer-lab/devtest-lab-developer-lab.png)

En este artículo obtendrá información sobre varias características de Azure DevTest Labs que se usan para cumplir los requisitos de los evaluadores y los pasos detallados para configurar un laboratorio.

## <a name="implementing-test-environments-with-azure-devtest-labs"></a>Implementación de entornos de prueba con Azure DevTest Labs
1. **Creación del laboratorio** 
   
    Los laboratorios son el punto de partida en Azure DevTest Labs. Una vez creado un laboratorio, puede llevar a cabo tareas tales como agregar usuarios (evaluadores) al laboratorio, establecer directivas para controlar los costos, definir imágenes de máquinas virtuales que se pueden crear rápidamente, etc.  
   
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
   | [Configuración de imágenes de Azure Marketplace](devtest-lab-configure-marketplace-images.md) |Aprenda a incluir imágenes de Azure Marketplace en una lista de permitidos y haga que estén disponibles para su selección solo las imágenes que quiera para los evaluadores.|
   | [Creación de una imagen personalizada](devtest-lab-create-template.md) |Cree una imagen personalizada instalando previamente el software que necesita para que los evaluadores puedan crear rápidamente una máquina virtual con la imagen personalizada.|
   | [Información acerca del generador de imágenes](./devtest-lab-faq.md#blog-post) |Vea un vídeo que describe cómo configurar y usar un generador de imágenes.|

3. **Creación de plantillas reutilizables para las máquinas de prueba** 
   
    Una fórmula en Azure DevTest Labs es una lista de valores de propiedad predeterminados que se usan para crear una máquina virtual. Puede crear una fórmula en el laboratorio seleccionando una imagen, un tamaño de máquina virtual (una combinación de CPU y RAM) y una red virtual. Cada evaluador puede ver la fórmula en el laboratorio y usarla para crear una máquina virtual. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Administración de fórmulas de DevTest Labs para crear máquinas virtuales](devtest-lab-manage-formulas.md) |Aprenda cómo crear una fórmula en el laboratorio seleccionando una imagen, un tamaño de máquina virtual (una combinación de CPU y RAM) y una red virtual.|

3. **Crear entornos de prueba de varias máquinas virtuales** 
   
    Puede usar plantillas de Azure Resource Manager para definir la infraestructura y la configuración de la solución de Azure e implementar repetidamente varias máquinas virtuales de prueba de manera coherente.

    Los recursos de PaaS de Azure se pueden aprovisionar en un entorno a partir de una plantilla de Resource Manager, de modo que aparezcan en el seguimiento de costos, aunque el cierre automático de las máquinas virtuales no se aplica a los recursos de PaaS.

    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Creación de entornos de varias máquinas virtuales y recursos de PaaS con plantillas de Azure Resource Manager](devtest-lab-create-environment-from-arm.md) |Obtenga información sobre cómo puede implementar varias máquinas virtuales en un estado coherente para el entorno de prueba.|

4. **Creación de artefactos para habilitar la personalización de máquinas virtuales flexibles**

   Los artefactos se utilizan para implementar y configurar la aplicación después de aprovisionar una máquina virtual. Los artefactos pueden ser:

   - Herramientas que desea instalar en la máquina virtual, como agentes, Fiddler y Visual Studio.
   - Acciones que desea ejecutar en la máquina virtual, como la clonación de un repositorio.
   - Aplicaciones que desea probar.

   Hay numerosos artefactos disponibles de serie. Puede crear sus propios artefactos personalizados si quiere obtener una mayor personalización para sus necesidades específicas.

   Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Creación de artefactos personalizados para la máquina virtual de DevTest Labs](devtest-lab-artifact-author.md) |Cree sus propios artefactos personalizados para las máquinas virtuales de su laboratorio.|
   | [Agregar un repositorio de Git para almacenar artefactos personalizados y plantillas de Azure Resource Manager para su uso en Azure DevTest Labs](devtest-lab-add-artifact-repo.md) |Aprenda a almacenar los artefactos personalizados en su propio repositorio privado de Git.|

5. **Control de los costos**
   
    Azure DevTest Labs le permite establecer una directiva en el laboratorio para especificar el número máximo de máquinas virtuales que puede crear un evaluador en el laboratorio. 
   
    Si el equipo de pruebas tiene una programación de trabajo configurada y usted quiere detener todas las máquinas virtuales en un momento determinado del día y, posteriormente, reiniciarlas automáticamente al día siguiente, lo puede hacer fácilmente mediante el establecimiento de directivas de apagado e inicio automático en el laboratorio. 
   
    Por último, al finalizar el desarrollo de aplicaciones puede eliminar todas las máquinas virtuales a la vez mediante la ejecución de un sencillo script de PowerShell. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Definición de directivas de laboratorio](devtest-lab-set-lab-policy.md) |Controle los costos mediante el establecimiento de directivas en el laboratorio. |
   | [Eliminación de todas las máquinas virtuales del laboratorio mediante un script de PowerShell](devtest-lab-faq.md#how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab) |Elimine todos los laboratorios en una sola operación al finalizar la prueba.|

1. **Incorporación de una red virtual a un laboratorio** 
   
    DevTest Labs crea una red virtual (VNET) cada vez que se crea un laboratorio. Si ha configurado su propia red virtual (por ejemplo, mediante el uso de VPN de sitio a sitio o ExpressRoute), puede agregar esta red virtual a la configuración de red virtual de su laboratorio para que esté disponible al crear máquinas virtuales.

    Además, hay un artefacto de unión a un dominio de Azure Active Directory disponible que une una máquina virtual a un dominio cuando se crea la máquina virtual. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Configuración de una red virtual en Azure DevTest Labs](devtest-lab-configure-vnet.md) |Aprenda a configurar una red virtual en Azure DevTest Labs mediante Azure Portal.|

6. **Uso compartido del laboratorio con cada evaluador**
   
    Se puede obtener acceso directo a los laboratorios mediante un vínculo que puede compartir con los evaluadores. Ni siquiera es necesario que tengan una cuenta de Azure, siempre y cuando dispongan de una [cuenta de Microsoft](devtest-lab-faq.md#what-is-a-microsoft-account). Los evaluadores no pueden ver las máquinas virtuales creadas por otros evaluadores.  
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Incorporación de un evaluador a un laboratorio de Azure DevTest Labs](devtest-lab-add-devtest-user.md) |Use Azure Portal para agregar evaluadores al laboratorio.|
   | [Incorporación de evaluadores al laboratorio mediante un script de PowerShell](devtest-lab-add-devtest-user.md#add-an-external-user-to-a-lab-using-powershell) |Use PowerShell para automatizar la incorporación de evaluadores al laboratorio. |
   | [Obtención de un vínculo al laboratorio](devtest-lab-faq.md#how-do-i-share-a-direct-link-to-my-lab) |Aprenda cómo pueden los evaluadores obtener acceso directo a un laboratorio a través de un hipervínculo.|

7. **Automatización de la creación de un laboratorio para más equipos** 
   
    Puede automatizar la creación de laboratorios, incluso con una configuración personalizada, mediante la creación de una plantilla de Resource Manager que puede utilizar para crear laboratorios idénticos una y otra vez. 
   
    Para más información, haga clic en los vínculos de la tabla siguiente:
   
   | Tarea | Conocimientos que adquirirá |
   | --- | --- |
   | [Creación de un laboratorio mediante una plantilla de Resource Manager](devtest-lab-faq.md#how-do-i-create-a-lab-from-a-resource-manager-template) |Cree laboratorios en Azure DevTest Labs mediante plantillas de Resource Manager. |

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]
