---
title: Preguntas más frecuentes sobre las plantillas de Resource Manager
description: Preguntas más frecuentes (P+F) sobre las plantillas de Azure Resource Manager.
ms.topic: conceptual
ms.date: 07/27/2020
ms.author: tomfitz
author: tfitzmac
ms.openlocfilehash: e23334daf0f4f6d5a6917af49c786d47c4557375
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327266"
---
# <a name="frequently-asked-questions-about-arm-templates"></a>Preguntas más frecuentes sobre las plantillas de Resource Manager

En este artículo se responde a preguntas frecuentes sobre las plantillas de Azure Resource Manager (ARM).

## <a name="getting-started"></a>Introducción

* **¿Qué son las plantillas de Resource Manager y por qué deben usarse?**

  Las plantillas de Resource Manager son archivos JSON en los que se define lo que se va a implementar en Azure. Las plantillas le ayudan a implementar una solución de infraestructura como código para Azure. Su organización puede implementar de forma repetida y confiable la infraestructura necesaria en diferentes entornos.
  
  Consulte [¿Qué son las plantillas de Resource Manager?](overview.md) para más información sobre cómo las plantillas de Resource Manager le ayudan a administrar la infraestructura de Azure.

* **¿Cómo empezar con las plantillas?**

  Para simplificar la creación de plantillas de Resource Manager, necesita las herramientas adecuadas. Se recomienda instalar [Visual Studio Code](https://code.visualstudio.com/) y la [extensión de herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools). Para obtener una introducción rápida a estas herramientas, consulte [Inicio rápido: Creación de plantillas de Azure Resource Manager con Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

  Cuando esté a punto para aprender a crear plantillas de Resource Manager, inicie la [serie de tutoriales para principiantes sobre las plantillas de Resource Manager](template-tutorial-create-first-template.md). Estos tutoriales le llevan paso a paso por el proceso de creación de una plantilla de Resource Manager. Obtendrá información sobre las diferentes secciones de la plantilla y cómo funcionarán en conjunto. Este contenido también está disponible como [módulo de Microsoft Learn](/learn/modules/authoring-arm-templates/).

* **¿Debo usar plantillas de Resource Manager o Terraform para realizar implementaciones en Azure?**

  Use la opción que más le guste. Ambos servicios son útiles para automatizar las implementaciones en Azure.
  
  Creemos que existen ventajas al usar plantillas de Resource Manager en otros servicios de infraestructura como código. Consulte [¿Por qué elegir plantillas de Resource Manager?](overview.md#why-choose-arm-templates) para más información sobre estas ventajas.

## <a name="build-2020"></a>Build 2020

* **He perdido la presentación en Microsoft Build 2020. ¿La presentación está disponible para su visualización?**

  Sí, [puede verla cuando quiera](https://mybuild.microsoft.com/sessions/82984db4-37a4-4ed3-bf8b-13298841ed18?source=sessions).

* **¿Dónde puedo obtener más información sobre las nuevas características anunciadas en Build?**

  Para obtener información general sobre las características que estamos desarrollando, únase al [grupo de Yammer para implementaciones de Azure Advisor](https://aka.ms/ARMMeet).

  Para más información sobre el nuevo lenguaje de plantilla, [suscríbase a las notificaciones](https://aka.ms/armLangUpdates).

  Para más información acerca de las especificaciones de plantilla, consulte [Especificaciones de plantilla de Azure Resource Manager (versión preliminar)](template-specs.md).

## <a name="creating-and-testing-templates"></a>Creación y pruebas de plantillas

* **¿Dónde puedo obtener información sobre los procedimientos recomendados para las plantillas de Resource Manager?**

  Para obtener recomendaciones sobre cómo implementar las plantillas, consulte [Procedimientos recomendados para plantilla de Resource Manager](template-best-practices.md). Después de crear una plantilla, ejecute el [kit de herramientas para pruebas de plantillas de Azure Resource Manager](https://github.com/azure/arm-ttk). Permite comprobar si la plantilla coincide con los procedimientos recomendados.

* **He configurado mi entorno a través del portal. ¿Hay alguna manera de obtener la plantilla de un grupo de recursos existente?**

  Sí, puede [exportar la plantilla](export-template-portal.md) desde un grupo de recursos. La plantilla exportada es un buen punto de partida para obtener información sobre las plantillas, pero probablemente querrá revisarla antes de usarla en un entorno de producción.
  
  Al exportar la plantilla, puede seleccionar los recursos que quiera incluir en ella.

* **¿Puedo crear un grupo de recursos en una plantilla de Resource Manager e implementar recursos en él?**

  Sí, puede crear un grupo de recursos en una plantilla al implementar la plantilla en el nivel de la suscripción de Azure. Para obtener un ejemplo de cómo crear un grupo de recursos e implementar recursos, consulte [Grupo de recursos y recursos](deploy-to-subscription.md#resource-groups).

* **¿Puedo crear una suscripción en una plantilla de Resource Manager?**

  Todavía no, pero estamos trabajando en ello.

* **¿Cómo se puede probar la plantilla antes de implementarla?**

  Se recomienda ejecutar el [kit de herramientas para pruebas de plantillas de Resource Manager](https://github.com/azure/arm-ttk) y la [operación hipotética](template-deploy-what-if.md) en las plantillas antes de implementarlas. El kit de herramientas para pruebas comprueba si la plantilla usa los procedimientos recomendados. Proporciona advertencias al identificar los cambios que podrían mejorar la implementación de la plantilla.

  La operación hipotética muestra los cambios que la plantilla realizará en su entorno. Puede ver los cambios no deseados antes de que se implementen. La característica de hipótesis (what-if) también devuelve los errores que puede detectar durante la validación preparatoria. Por ejemplo, si la plantilla contiene un error sintáctico, devolverá el error. También devuelve los errores que puede determinar sobre el estado final de los recursos implementados. Por ejemplo, si la plantilla implementa una cuenta de almacenamiento con un nombre que ya está en uso, la característica de hipótesis devuelve ese error.

* **¿Dónde puedo encontrar información acerca de las propiedades que están disponibles para cada tipo de recurso?**

  VS Code proporciona IntelliSense para trabajar con las propiedades del recurso. También puede ver las propiedades y descripciones en la [referencia de plantilla](/azure/templates/).

* **He de crear varias instancias de un tipo de recurso. ¿Cómo puedo crear un iterador en mi plantilla?**

  Use el elemento de copia para especificar más de una instancia. También puede usar la copia en [recursos](copy-resources.md), [propiedades](copy-properties.md), [variables](copy-variables.md) y [salidas](copy-outputs.md).

## <a name="template-language"></a>Lenguaje de plantilla

* **He oído que se está trabajando en un nuevo lenguaje de plantilla. ¿Dónde puedo encontrar más información al respecto?**

  Para más información sobre el nuevo lenguaje de plantilla, [suscríbase a las notificaciones](https://aka.ms/armLangUpdates).

* **¿Hay un plan que admita la creación de plantillas en YAML?**

  En este momento, no hay ningún plan que admita YAML. Creemos que el nuevo lenguaje de plantilla ofrecerá una solución que será más fácil de usar que YAML o JSON.

* **¿Podré seguir escribiendo plantillas en JSON cuando se haya publicado el nuevo lenguaje de plantilla?**

  Sí, podrá seguir usando plantillas JSON.

* **¿Ofrecerán una herramienta para que pueda convertir mis plantillas JSON al nuevo lenguaje de plantilla?**

  Sí.

## <a name="template-specs"></a>Especificaciones de plantilla

* **¿Cómo puedo participar en la versión preliminar de Especificaciones de plantilla?**

  [Únase a la lista de espera](https://aka.ms/templateSpecsWaitlist) para las especificaciones de plantilla.

* **¿Qué relación existe entre las especificaciones de plantilla y Azure Blueprints?**

  Azure Blueprints usará especificaciones de plantilla en su implementación reemplazando el recurso `blueprint definition` por uno `template spec`. Se facilitará una ruta de migración para convertir la definición de plano técnico en una especificación de plantilla, pero las API de definición de plano técnico seguirán siendo compatibles. No hay que realizar ningún cambio en el recurso `blueprint assignment`. Los planos técnicos seguirán siendo una experiencia de usuario para crear un entorno controlado de Azure.

* **¿Las especificaciones de plantilla reemplazan a las plantillas vinculadas?**

  No, pero las especificaciones de plantilla están diseñadas para funcionar bien con plantillas vinculadas. No es necesario que mueva la plantilla vinculada a un punto de conexión accesible públicamente antes de implementar la plantilla principal. En su lugar, debe empaquetar la plantilla principal y sus artefactos juntos al crear la especificación de la plantilla.

* **¿Las especificaciones de plantilla se pueden compartir entre suscripciones?**

  Sí, se pueden usar en suscripciones siempre que el usuario tenga acceso de lectura a la especificación de plantilla. Las especificaciones de plantilla no se pueden usar en los inquilinos.

## <a name="scripts-in-templates"></a>Scripts en plantillas

* **¿Puedo incluir un script en mi plantilla para realizar tareas que no son posibles en una plantilla?**

  Sí, use [scripts de implementación](deployment-script-template.md). Puede incluir scripts de Azure PowerShell o la CLI de Azure en las plantillas. La característica se encuentra en versión preliminar.

* **¿Puedo seguir usando extensiones de script personalizadas con Desired State Configuration (DSC)?**

  Estas opciones siguen estando disponibles y no han cambiado. Los scripts de implementación están diseñados para realizar acciones que no están relacionadas con el invitado de la máquina virtual. Si ha de ejecutar un script en un sistema operativo host de una máquina virtual, la extensión de script personalizado o DSC sería una opción mejor. Sin embargo, los scripts de implementación tienen ventajas, como el establecimiento de la duración del tiempo de expiración.

* **¿Se admiten los scripts de implementación en Azure Government?**

  Sí, puede usar scripts de implementación en US Gov Arizona y US Gov Virginia.

## <a name="preview-changes-before-deployment"></a>Vista previa de los cambios antes de la implementación

* **¿Puedo obtener una vista previa de los cambios que se producirán antes implementar una plantilla?**

  Sí, use la [característica de hipótesis "what-if"](template-deploy-what-if.md). Dicha característica evalúa el estado actual del entorno y lo compara con el estado que habrá después de la implementación. Puede examinar los cambios resumidos para asegurarse de que la plantilla no tiene ningún resultado inesperado.

* **¿Puedo usar la característica de hipótesis con los modos incremental y completo?**

  Sí, se admiten ambos [modos de implementación](deployment-modes.md). Para obtener un ejemplo del uso del modo incremental, consulte el artículo sobre la [ejecución de la operación hipotética](template-deploy-what-if.md#run-what-if-operation). Para obtener un ejemplo del uso del modo completo, consulte el artículo sobre la [confirmación de la eliminación](template-deploy-what-if.md#confirm-deletion).

* **¿Funciona la característica de hipótesis con plantillas vinculadas?**

  Sí, la característica de hipótesis evalúa el estado de la plantilla principal y sus plantillas vinculadas.

* **¿Puedo usar la característica de hipótesis en una canalización de Azure?**

  Sí, puede usar la característica de hipótesis para comprobar que la canalización deba continuar.

* **Cuando uso la característica de hipótesis, veo cambios en las propiedades que no están en mi plantilla. ¿Es previsible este "ruido"?**

  La característica de hipótesis está en versión preliminar. Estamos trabajando para reducir el ruido. Para ayudarnos a mejorar, envíenos los problemas que tenga mediante nuestro repositorio de GitHub, en https://aka.ms/WhatIfIssues.

## <a name="template-visualizer"></a>Visualizador de plantilla

* **¿Hay alguna manera de visualizar la plantilla de Resource Manager y sus recursos?**

  Disponemos de una [extensión de VS Code aportada por la comunidad](https://aka.ms/ARMVisualizer) que realiza un gran trabajo de visualización de la plantilla de Resource Manager. Muestra los recursos que se van a implementar y las relaciones entre ellos.

* **¿Puedo usar el visualizador de plantillas fuera de VS Code?**

  El visualizador de plantillas se encuentra en versión preliminar en el portal. Para más información, vea esta [breve sesión de Build](https://mybuild.microsoft.com/sessions/0525094b-1fd2-4f69-bfd6-6d2fff6ffe5f?source=sessions).

## <a name="deployment-limits"></a>Límites de implementación

* **¿Cuántos grupos de recursos se pueden implementar en una sola operación de implementación?**

  En el pasado, este límite era de cinco grupos de recursos. Se ha aumentado recientemente a 800 grupos de recursos. Para más información, consulte [Creación de grupos de recursos y otros recursos en el nivel de suscripción](deploy-to-subscription.md).

* **Recibí un error sobre la limitación a 800 implementaciones en el historial de implementación, ¿qué debo hacer?**

  Estamos cambiando el modo en el que se mantiene el historial de implementación de un grupo de recursos. En el pasado, había que eliminar manualmente las implementaciones de este historial para evitar este error. A partir de junio de 2020, las implementaciones se eliminarán automáticamente del historial a medida que se acerque el límite. Para obtener más información, consulte [Eliminaciones automáticas del historial de implementaciones](deployment-history-deletions.md).

  La eliminación de una implementación del historial no afecta a los recursos implementados.

## <a name="templates-and-devops"></a>Plantillas y DevOps

* **¿Puedo integrar plantillas de Resource Manager en Azure Pipelines?**

  Sí. Para obtener una explicación de cómo usar la plantilla y las canalizaciones, consulte [Tutorial: Integración continua de plantillas de Azure Resource Manager en Azure Pipelines](deployment-tutorial-pipeline.md) e [Integración de plantillas Integración continua de plantillas de Azure Resource Manager con Azure Pipelines](add-template-to-azure-pipelines.md).

* **¿Puedo usar las acciones de GitHub para implementar una plantilla?**

  Sí, consulte [Implementación de plantillas de Azure Resource Manager mediante Acciones de GitHub](deploy-github-actions.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte [¿Qué son las plantillas de Resource Manager?](overview.md) para ver una introducción a las plantillas de Resource Manager.
