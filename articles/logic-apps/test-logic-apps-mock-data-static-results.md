---
title: Probar las aplicaciones lógicas con datos simulados - Azure Logic Apps
description: Configurar resultados estáticos para probar las aplicaciones lógicas con datos simulados sin que afecte a los entornos de producción
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, LADocs
ms.topic: article
ms.date: 05/13/2019
ms.openlocfilehash: 45eeb20e5c572ddd98244b2e751322fcce1d4b76
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597195"
---
# <a name="test-logic-apps-with-mock-data-by-setting-up-static-results"></a>Probar las aplicaciones lógicas con datos simulados mediante la configuración de resultados estáticos

Al probar las aplicaciones lógicas, podría no ser listo para llamar a realmente o tener acceso a las aplicaciones, servicios y sistemas por diversos motivos. Normalmente, en estos casos, es posible que deba ejecutar las rutas de acceso de la condición diferente, forzar errores, proporcionan los cuerpos de respuesta de mensaje específico o incluso intente omitir algunos pasos. Mediante la configuración de resultados estáticos para una acción en la aplicación lógica, puede simular datos de salida de esa acción. Resultados de estático en una acción no ejecutan la acción, pero devuelve los datos simulados en su lugar.

Por ejemplo, si configura resultados estáticos para la Outlook 365 enviar acción de correo electrónico, el motor de Logic Apps solo devuelve los datos ficticios que especificó como resultados estáticos, en lugar de llamar a Outlook y enviar un correo electrónico.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción de Azure. Si no tiene una suscripción de Azure, <a href="https://azure.microsoft.com/free/" target="_blank">regístrese para obtener una cuenta gratuita de Azure</a>.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica donde desea configurar resultados estáticos

<a name="set-up-static-results"></a>

## <a name="set-up-static-results"></a>Configurar resultados estáticos

1. Si no lo ha hecho ya, en el [portal Azure](https://portal.azure.com), abra la aplicación lógica en el Diseñador de Logic Apps.

1. En la acción donde desea configurar resultados estáticos, siga estos pasos: 

   1. En la esquina superior derecha de la acción, elija el botón de puntos suspensivos (*...* ) y seleccione **resultado estático**, por ejemplo:

      ![Seleccione "Resultado estático" > "Habilitar resultado estático"](./media/test-logic-apps-mock-data-static-results/select-static-result.png)

   1. Elija **habilitar resultado estático**. Para las propiedades necesarias (*), especifique los valores de salida ficticio que se desea devolver la respuesta de la acción.

      Por ejemplo, estas son las propiedades necesarias para la acción HTTP:

      | Propiedad | DESCRIPCIÓN |
      |----------|-------------|
      | **Estado** | Estado de la acción para devolver |
      | **Código de estado** | El código de estado específicos para devolver |
      | **Encabezados** | Para devolver el contenido del encabezado |
      |||

      ![Seleccione "Habilitar resultado estático"](./media/test-logic-apps-mock-data-static-results/enable-static-result.png)

      Para escribir los datos simulados en formato JavaScript Object Notation (JSON), elija **cambiar al modo JSON** (![elegir "Modo de conmutación a JSON"](./media/test-logic-apps-mock-data-static-results/switch-to-json-mode-button.png)).

   1. Para las propiedades opcionales, abra el **seleccione campos opcionales** enumerar y seleccionar las propiedades que desea simular.

      ![Seleccione las propiedades opcionales](./media/test-logic-apps-mock-data-static-results/optional-properties.png)

1. Cuando esté listo para guardar, elija **realiza**.

   En la esquina superior derecha de la acción, la barra de título ahora muestra un icono de probeta prueba (![icono para obtener resultados estáticos](./media/test-logic-apps-mock-data-static-results/static-results-test-beaker-icon.png)), lo que indica que ha habilitado resultados estáticos.

   ![Mostrar icono habilitado resultados estáticos](./media/test-logic-apps-mock-data-static-results/static-results-enabled.png)

   Para buscar las ejecuciones anteriores que usan datos simulados, consulte [encontrar ejecuciones que utilizan los resultados estáticos](#find-runs-mock-data) más adelante en este tema.

<a name="reuse-sample-outputs"></a>

## <a name="reuse-previous-outputs"></a>Volver a usar las salidas anteriores

Si la aplicación lógica tiene una anterior ejecutar con salidas, que puede volver a usar como salidas ficticias, puede copiar y pegar los resultados de la que se ejecutan.

1. Si no lo ha hecho ya, en el [portal Azure](https://portal.azure.com), abra la aplicación lógica en el Diseñador de Logic Apps.

1. En el menú principal de la aplicación lógica, seleccione **Introducción**.

1. En el **historial de ejecuciones** sección, seleccione la aplicación lógica ejecute desee.

1. En el flujo de trabajo de la aplicación lógica, busque y expanda la acción que tiene los resultados que desee.

1. Elija la **mostrar salidas sin procesar** vínculo.

1. Copiar el objeto de JavaScript Object Notation (JSON) completa o la subsección específica que desea usar, por ejemplo, la sección de salidas o incluso en la sección de encabezados.

1. Siga los pasos para abrir el **resultado estático** cuadro para la acción en [configurar resultados estáticos](#set-up-static-results).

1. Después de la **resultado estático** se abre el cuadro, elija el paso:

   * Para pegar un objeto JSON completo, elija **cambiar al modo JSON** (![elegir "Modo de conmutación a JSON"](./media/test-logic-apps-mock-data-static-results/switch-to-json-mode-button.png)):

     ![Elija "Cambiar a modo JSON" para el objeto completo](./media/test-logic-apps-mock-data-static-results/switch-to-json-mode-button-complete.png)

   * Para pegar solo una sección JSON, junto a la etiqueta de la sección, elija **cambiar al modo JSON** para esa sección, por ejemplo:

     ![Elija "Cambiar a modo JSON" para las salidas](./media/test-logic-apps-mock-data-static-results/switch-to-json-mode-button-outputs.png)

1. En el editor de JSON, pegar JSON copiada con anterioridad.

   ![Modo JSON](./media/test-logic-apps-mock-data-static-results/json-editing-mode.png)

1. Cuando haya finalizado, elija **Listo**. O bien, para volver al diseñador, elija **modo Editor de conmutador** (![elegir "Modo de Editor de conmutador"](./media/test-logic-apps-mock-data-static-results/switch-editor-mode-button.png)).

<a name="find-runs-mock-data"></a>

## <a name="find-runs-that-use-static-results"></a>Buscar ejecuciones que utilizan los resultados estáticos

Historial de ejecuciones de la aplicación lógica identifica las ejecuciones donde las acciones usar resultados estáticos. Para buscar estas ejecuciones, siga estos pasos:

1. En el menú principal de la aplicación lógica, seleccione **Introducción**. 

1. En el panel derecho, bajo **historial de ejecuciones**, busque el **resultados estático** columna. 

   Tiene cualquier serie que incluya las acciones con los resultados de la **resultados estático** columna establecida en **habilitado**, por ejemplo:

   ![Hist - columna estática resultados](./media/test-logic-apps-mock-data-static-results/run-history.png)

1. Para ver las acciones que usan resultados estáticos, seleccione la ejecución de dónde el **resultados estático** columna está establecida en **habilitado**.

   Acciones que usan estáticos resultados muestran el vaso de prueba (![icono para obtener resultados estáticos](./media/test-logic-apps-mock-data-static-results/static-results-test-beaker-icon.png)) icono, por ejemplo:

   ![Ejecute - historial de acciones que usan resultados estáticos](./media/test-logic-apps-mock-data-static-results/static-results-enabled-run-details.png)

## <a name="disable-static-results"></a>Deshabilitar resultados estáticos

Al desactivar resultados estáticos no deshacerse de los valores de la última configuración. Por lo tanto, al activar resultados estáticos la próxima vez, puede continuar utilizando los valores anteriores.

1. Encontrar la acción que desea deshabilitar salidas estáticas. En la esquina superior derecha de la acción, elija el icono de probeta prueba (![icono para obtener resultados estáticos](./media/test-logic-apps-mock-data-static-results/static-results-test-beaker-icon.png)).

   ![Deshabilitar resultados estáticos](./media/test-logic-apps-mock-data-static-results/disable-static-results.png)

1. Elija **deshabilitar resultado estático** > **realiza**.

   ![Deshabilitar resultados estáticos](./media/test-logic-apps-mock-data-static-results/disable-static-results-button.png)

## <a name="reference"></a>Referencia

Para obtener más información acerca de esta configuración en las definiciones de flujo de trabajo subyacente, vea [resultados estáticos: referencia de esquema de lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md#static-results) y [runtimeConfiguration.staticResult - tiempo de ejecución Opciones de configuración](../logic-apps/logic-apps-workflow-actions-triggers.md#runtime-configuration-settings)

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [Azure Logic Apps](../logic-apps/logic-apps-overview.md)