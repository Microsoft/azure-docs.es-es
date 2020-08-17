---
title: Introducción a la representación
description: Introducción al uso de Azure para la representación e información general de las funcionalidades de representación con Azure Batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: 9fac5d3efabc5d9f796c91d688f35e01aeefdca3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092769"
---
# <a name="rendering-using-azure"></a>Representación mediante Azure

La representación es el proceso de tomar los modelos 3D y convertirlos en imágenes 2D. Los archivos de escena 3D se crean en aplicaciones como Autodesk 3ds Max, Autodesk Maya y Blender.  Las aplicaciones de representación, como Autodesk Maya, Autodesk Arnold, Chaos Group V-Ray y Blender Cycles, generan imágenes 2D.  A veces, las imágenes individuales se crean a partir de los archivos de escena. Sin embargo, es común modelar y representar varias imágenes y luego combinarlas en una animación.

La carga de trabajo de representación se usa mucho como efectos especiales (VFX) en el sector multimedia y de entretenimiento. La representación también se usa en muchos otros sectores como la publicidad, el mercado minorista, petróleo y gas y fabricación.

El proceso de representación es muy intenso desde el punto de vista computacional; puede haber muchos fotogramas e imágenes para generar y cada imagen puede tardar muchas horas en representarse.  Por lo tanto, la representación es una carga de trabajo de procesamiento por lotes perfecta que puede aprovechar Azure y Azure Batch para ejecutar muchas representaciones en paralelo.

## <a name="why-use-azure-for-rendering"></a>¿Por qué usar Azure para la representación?

Por muchas razones, la representación es una carga de trabajo perfectamente adecuada para Azure y Azure Batch:

* Los trabajos de representación se pueden dividir en muchas partes que se pueden ejecutar en paralelo mediante varias máquinas virtuales:
  * Las animaciones constan de muchos fotogramas y cada fotograma se puede representar en paralelo.  Cuantas más máquinas virtuales haya disponibles para procesar cada fotograma, más rápido podrán generarse todos los fotogramas y la animación.
  * Algunos programas de software de representación permiten que los fotogramas individuales se dividan en varias partes, como iconos o segmentos.  Cada parte se puede representar por separado y luego combinarse en la imagen final cuando todas las partes han terminado.  Cuantas más máquinas virtuales haya disponibles, más rápido se podrá representar un fotograma.
* Los proyectos de representación puede requerir una gran escala:
  * Los fotogramas individuales pueden ser complejos y requerir muchas horas para representarse, incluso en hardware de gama alta; las animaciones pueden constar de cientos de miles de fotogramas.  Se requiere una gran inmensa cantidad de proceso para representar las animaciones de alta calidad en un período de tiempo razonable.  En algunos casos, se han utilizado más de 100 000 núcleos para representar miles de fotogramas en paralelo.
* Los proyectos de representación se basan en proyecto y requieren cantidades variables de proceso:
  * Asignar capacidad de proceso y almacenamiento cuando sea necesario, escalarla o reducirla verticalmente según la carga durante un proyecto y quitarla cuando haya finalizado un proyecto.
  * Pagar por capacidad cuando se asigna, pero no pagar por ella cuando no hay ninguna carga, como entre proyectos.
  * Preste atención a las ráfagas debidas a cambios inesperados; escale más alto si hay cambios inesperados tarde en un proyecto y esos cambios se deben procesar en un horario ajustado.
* Elija entre una gran variedad de hardware conforme a la aplicación, la carga de trabajo y el período de tiempo:
  * Hay una gran variedad de hardware disponible en Azure que se puede asignar y administrar con Batch.
  * Según el proyecto, el requisito puede ser la mejor relación precio/rendimiento o el mejor rendimiento global.  Diferentes escenas y/o aplicaciones de representación tendrán requisitos de memoria distintos.  Alguna aplicación de representación puede aprovechar las GPU para el mejor rendimiento o determinadas características. 
* Las máquinas virtuales de prioridad baja reducen los costos:
  * Las máquinas virtuales de baja prioridad están disponibles por un gran descuento en comparación con las máquinas virtuales convencionales a petición y son adecuadas para algunos tipos de trabajo.
  * Azure Batch puede asignar máquinas virtuales de baja prioridad y proporciona flexibilidad sobre cómo se usan para satisfacer un amplio conjunto de requisitos.  Los grupos de Batch pueden constar tanto de máquinas virtuales dedicadas como de baja prioridad, siendo posible cambiar la combinación de tipos de máquina virtual en cualquier momento.

## <a name="options-for-rendering-on-azure"></a>Opciones de representación en Azure

Existe una amplia gama de funcionalidades de Azure que se pueden usar para representar cargas de trabajo.  Cuáles de estas funcionalidades se deben utilizar depende del entorno existente y los requisitos.

### <a name="existing-on-premises-rendering-environment-using-a-render-management-application"></a>Entorno de representación local existente mediante una aplicación de administración de representación

El caso más común es que haya una granja de representación local que se administre mediante una aplicación de administración de representación, como PipelineFX Qube, Royal Render o Thinkbox Deadline.  El requisito es ampliar la capacidad de la granja de representación local mediante máquinas virtuales de Azure.

El software de administración de representación cuenta con soporte técnico de Azure integrado o hacemos que haya complementos disponibles que agregan soporte técnico de Azure. Para más información sobre los administradores de representación admitidos y la funcionalidad habilitada, consulte el artículo sobre el [uso de administradores de representación](./batch-rendering-render-managers.md).

### <a name="custom-rendering-workflow"></a>Flujo de trabajo de representación personalizado

El requisito es para que las máquinas virtuales amplíen una granja de representación existente.  Los grupos de Azure Batch pueden asignar grandes cantidades de máquinas virtuales, permitir el uso de máquinas virtuales de baja prioridad y el escalado automático dinámicamente con máquinas virtuales de precio completo, y proporcionar licencias de pago por uso para aplicaciones de representación populares.

### <a name="no-existing-render-farm"></a>Granja de representación no existente

Las estaciones de trabajo cliente pueden realizar la representación, pero la carga de trabajo de representación aumenta y se tarda demasiado en usar únicamente la capacidad de la estación de trabajo.  Azure Batch se puede usar tanto para asignar proceso de granja de representación a petición como para programar los trabajos de representación en la granja de representación de Azure.

## <a name="azure-batch-rendering-capabilities"></a>Funcionalidades de representación de Azure Batch

Azure Batch permite la ejecución de cargas de trabajo paralelas en Azure.  Permite la creación y administración de un gran número de máquinas virtuales en las que se instalan y ejecutan las aplicaciones.  También ofrece funcionalidades de programación de trabajo completas para ejecutar instancias de esas aplicaciones, lo que proporciona la asignación de tareas a las máquinas virtuales, la puesta en cola, la supervisión de aplicaciones, etc.

Azure Batch se utiliza para muchas cargas de trabajo, pero las siguientes funcionalidades están disponibles para hacer que sea específicamente más fácil y rápido ejecutar cargas de trabajo de representación.

* Imágenes de máquina virtual con aplicaciones de gráficos y representación preinstaladas:
  * Hay imágenes de máquina virtual de Azure Marketplace disponibles que contienen aplicaciones de gráficos y representación populares, lo que evita la necesidad de instalar las aplicaciones por sí mismo o crear sus propias imágenes personalizadas con las aplicaciones instaladas. 
* Licencias de pago por uso para aplicaciones de representación:
  * Puede optar por pagar por las aplicaciones por minuto, además de pagar por las máquinas virtuales de proceso, lo que evita tener que comprar licencias y la posible configuración de un servidor de licencias para las aplicaciones.  Pagar por el uso también significa que es posible atender cargas variables e inesperadas, ya que no hay un número fijo de licencias.
  * También es posible usar las aplicaciones preinstaladas con sus propias licencias y no usar las licencias de pago por uso. Para ello, normalmente se instala un servidor de licencias local o basado en Azure y se usa una red virtual de Azure para conectar el grupo de representación al servidor de licencias.
* Complementos para aplicaciones de modelado y diseño de cliente:
  * Los complementos permiten a los usuarios finales usar Azure Batch directamente desde la aplicación cliente, como Autodesk Maya, lo que les permite crear grupos, enviar trabajos y hacer uso de más capacidad de proceso a realizar representaciones más rápido.
* Integración del administrador de representación:
  * Azure Batch se integra en las aplicaciones de administración de presentación o hay complementos disponibles para proporcionar la integración de Azure Batch.

Hay varias maneras de usar Azure Batch, que también se aplican a la representación con Azure Batch.

* API:
  * Escriba código mediante [REST](/rest/api/batchservice), [.NET](/dotnet/api/overview/azure/batch), [Python](/python/api/overview/azure/batch), [Java](/java/api/overview/azure/batch) u otras API admitidas.  Los desarrolladores pueden integrar funcionalidades de Azure Batch en sus aplicaciones o su flujo de trabajo existentes, independientemente de que se encuentre en la nube o de forma local.  Por ejemplo, el [complemento Autodesk Maya](https://github.com/Azure/azure-batch-maya) utiliza la API de Python de Batch para invocar a Batch, crear y administrar grupos, enviar trabajos y tareas y supervisar el estado.
* Herramientas de línea de comandos:
  * La [línea de comandos de Azure](/cli/azure/) o [Azure PowerShell](/powershell/azure/) se puede usar para crear un script de uso de Batch.
  * En concreto, la compatibilidad con plantillas de la CLI de Batch facilita la creación de grupos y el envío de trabajos.
* Interfaces de usuario:
  * [Batch Explorer](https://github.com/Azure/BatchExplorer) es una herramienta de cliente multiplataforma que también permite administrar y supervisar las cuentas de Batch, pero proporciona algunas funcionalidades más versátiles en comparación con la interfaz de usuario de Azure Portal.  Se proporciona un conjunto de plantillas de grupo y trabajo adaptadas a cada aplicación admitida y se puede usar para crear fácilmente grupos y enviar trabajos.
  * Azure Portal se puede usar para administrar y supervisar Azure Batch.
* Complementos de aplicación cliente:
  * Hay complementos disponibles que permiten que la representación de Batch se use directamente desde las aplicaciones de modelado y diseño del cliente. Los complementos principalmente invocan la aplicación Batch Explorer con información contextual sobre el modelo 3D actual.
  * Están disponibles los siguientes complementos:
    * [Azure Batch para Maya](https://github.com/Azure/azure-batch-maya)
    * [3ds Max](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/3ds-max)
    * [Blender](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/blender)

## <a name="getting-started-with-azure-batch-rendering"></a>Introducción a la representación con Azure Batch

Consulte los siguientes tutoriales de introducción para probar la representación con Azure Batch:

* [Tutorial: Representación de una escena de Blender con Batch Explorer](./tutorial-rendering-batchexplorer-blender.md)
* [Uso de la CLI de Batch para representar una escena de Autodesk 3ds Max](./tutorial-rendering-cli.md)

## <a name="next-steps"></a>Pasos siguientes

Determine la lista de aplicaciones y versiones de representación incluidas en las imágenes de máquina virtual de Azure Marketplace en [este artículo](./batch-rendering-applications.md).
