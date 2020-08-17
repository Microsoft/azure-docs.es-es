---
title: Creación de un flujo de trabajo de integración automatizado
description: 'Inicio rápido: Cree el primer flujo de trabajo automatizado mediante Azure Logic Apps para soluciones de integración de aplicaciones empresariales (EAI) e integración de sistemas'
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: quickstart
ms.custom: mvc
ms.date: 07/23/2020
ms.openlocfilehash: 980e3e036257bbf5aa9743025bbfb55065176a39
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87133306"
---
# <a name="quickstart-create-your-first-automated-integration-workflow-by-using-azure-logic-apps---azure-portal"></a>Inicio rápido: Creación del primer flujo de trabajo de integración automatizado con Azure Logic Apps: Azure Portal

En esta guía de inicio rápido se presentan los conceptos generales básicos sobre cómo compilar el primer flujo de trabajo mediante [Azure Logic Apps](logic-apps-overview.md), como la creación de una aplicación lógica en blanco, la adición de un desencadenador y una acción y, a continuación, la prueba de la aplicación lógica. En esta guía de inicio rápido, creará una aplicación lógica que comprobará periódicamente la fuente RSS de un sitio web para detectar nuevos elementos. Si existe algún elemento nuevo, la aplicación lógica envía un correo electrónico por cada elemento. Cuando haya terminado, la aplicación lógica se parecerá a este flujo de trabajo, en un alto nivel:

![Mapa conceptual en el que se muestra un flujo de trabajo de aplicación lógica de ejemplo de alto nivel.](./media/quickstart-create-first-logic-app-workflow/quickstart-workflow-overview.png)

Para este escenario, necesita una suscripción a Azure o [registrarse para una cuenta de Azure gratis](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), una cuenta de correo electrónico de un servicio compatible con Azure Logic Apps, como Office 365 Outlook, Outlook.com o Gmail. Para otros servicios de correo electrónico compatibles, [revise la lista de conectores aquí](/connectors/). En este ejemplo, la aplicación lógica usa cuenta de Office 365 Outlook. Si usa un servicio de correo electrónico diferente, los pasos generales son los mismos, pero la interfaz de usuario podría diferir ligeramente.

> [!IMPORTANT]
> Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

## <a name="create-your-logic-app"></a>Creación de una aplicación lógica

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el cuadro de búsqueda de Azure Portal, escriba `logic apps` y seleccione **Logic Apps**.

   ![Captura de pantalla del cuadro de búsqueda de Azure Portal con "Logic apps" como término de la búsqueda y "Logic Apps" como el resultado de la búsqueda seleccionado.](./media/quickstart-create-first-logic-app-workflow/find-select-logic-apps.png)

1. En la página **Logic Apps**, seleccione en **Agregar**.

   ![Captura de pantalla que muestra la lista de aplicaciones lógicas y el botón seleccionado, "Agregar".](./media/quickstart-create-first-logic-app-workflow/add-new-logic-app.png)

1. En el panel **Aplicación lógica**, proporcione los detalles sobre la aplicación lógica, como se muestra a continuación.

   ![Captura de pantalla que muestra el panel de creación de aplicaciones lógicas con detalles de la nueva aplicación lógica.](./media/quickstart-create-first-logic-app-workflow/create-logic-app-settings.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Nombre** | <*nombre-de-la-aplicación-lógica*> | El nombre de la aplicación lógica (que debe ser exclusivo entre las regiones) solo puede contener letras, números, guiones (`-`), caracteres de subrayado (`_`), paréntesis (`(`, `)`) y puntos (`.`). En este ejemplo se usa "My-First-Logic-App". |
   | **Suscripción** | <*Azure-subscription-name*> | El nombre de la suscripción de Azure |
   | **Grupos de recursos** | <*nombre del grupo de recursos de Azure*> | El nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) (que debe ser exclusivo entre las regiones) se utiliza para organizar recursos relacionados. En este ejemplo se usa "My-First-LA-RG". |
   | **Ubicación** | <*Azure-region*> | La región en la que desea almacenar la información de la aplicación lógica. En este ejemplo se utiliza "Oeste de EE. UU.". |
   | **Log Analytics** | Off | Mantenga el valor **Off** para el registro de diagnóstico. |
   ||||

1. Seleccione **Revisar y crear** cuando esté preparado. Confirme los detalles que proporcionó y seleccione **Crear**.

1. Una vez que Azure implemente correctamente la aplicación, seleccione **Ir al recurso**.

   ![Captura de pantalla que muestra la página de implementación del recurso y el botón seleccionado para "Ir al recurso".](./media/quickstart-create-first-logic-app-workflow/go-to-new-logic-app-resource.png)

   O también puede buscar y seleccionar la aplicación lógica escribiendo el nombre en el cuadro de búsqueda.

   El Diseñador de aplicación lógica se abre y muestra una página con un vídeo de introducción y desencadenadores utilizados frecuentemente. En **Plantillas**, elija **Blank Logic App**.

   ![Captura de pantalla que muestra la galería de plantillas del diseñador de aplicaciones lógicas y la plantilla seleccionada, "Blank Logic App".](./media/quickstart-create-first-logic-app-workflow/choose-logic-app-template.png)

A continuación, añada un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) que se active cuando aparezca un nuevo elemento en la fuente RSS. Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Azure Logic Apps crea una instancia de aplicación lógica que inicia y ejecuta el flujo de trabajo.

<a name="add-rss-trigger"></a>

## <a name="add-the-rss-trigger"></a>Agregar el desencadenador de RSS

1. En el **Diseñador de aplicación lógica**, debajo del cuadro de búsqueda, seleccione **Todo**.

1. En el cuadro de búsqueda, escriba `rss` para buscar el conector de RSS. En la lista de desencadenadores, seleccione el desencadenador de RSS **Cuando se publica un elemento de fuente**.

   ![Captura de pantalla que muestra el diseñador de aplicaciones lógicas con "RSS" en el cuadro de búsqueda y el desencadenador seleccionado "Cuando se publica un elemento de fuente".](./media/quickstart-create-first-logic-app-workflow/add-rss-trigger-new-feed-item.png)

1. Proporcione la información del desencadenador tal y como se describe en este paso:

   ![Captura de pantalla que muestra el diseñador de aplicaciones lógicas con la configuración del desencadenador de RSS, la cual incluye la dirección URL, la frecuencia y el intervalo de RSS.](./media/quickstart-create-first-logic-app-workflow/add-rss-trigger-settings.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **URL de fuente RSS** | <*URL-de-fuente-RSS*> | Vínculo de la fuente RSS que desea supervisar En este ejemplo se usa la fuente RSS del Wall Street Journal en `https://feeds.a.dj.com/rss/RSSMarketsMain.xml` pero, si lo desea, puede usar su propia dirección URL de la fuente RSS. |
   | **Intervalo** | 1 | Número de intervalos que se espera entre comprobaciones |
   | **Frecuencia** | Minute | La unidad de tiempo de cada intervalo entre comprobaciones |
   ||||

   Juntos, el intervalo y la frecuencia definen la programación para el desencadenador de la aplicación lógica. Esta aplicación lógica comprueba la fuente cada minuto.

1. Para contraer los detalles del desencadenador por ahora, haga clic dentro de la barra de título del desencadenador.

   ![Captura de pantalla que muestra el diseñador de aplicaciones lógicas con la forma contraída de la aplicación lógica.](./media/quickstart-create-first-logic-app-workflow/collapse-trigger-shape.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

La aplicación lógica estará activa, pero no hace más que comprobar la fuente RSS. Por lo tanto, agregue una acción que responda cuando se active el desencadenador.

## <a name="add-the-send-email-action"></a>Adición de la acción "enviar correo electrónico"

Ahora, agregue una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) que envíe un correo electrónico cuando aparezca un nuevo elemento en la fuente RSS.

1. En el desencadenador **Cuando se publica un elemento de fuente**, elija **Nuevo paso**.

   ![Captura de pantalla que muestra el diseñador de aplicaciones lógicas con "Nuevo paso".](./media/quickstart-create-first-logic-app-workflow/add-new-step-under-trigger.png)

1. En **Elegir una acción** y en el cuadro de búsqueda, seleccione **Todas**.

1. En el cuadro de búsqueda, escriba `send an email` para buscar conectores que ofrezcan esta acción. Para filtrar la lista de acciones de una aplicación o un servicio específicos, puede seleccionar la aplicación o servicio primero.

   Por ejemplo, si usa una cuenta profesional o educativa de Microsoft y desea usar Office 365 Outlook, seleccione **Office 365 Outlook**. O, si usa una cuenta de Microsoft personal, puede seleccionar Outlook.com. Este ejemplo continúa con Office 365 Outlook:

   ![Captura de pantalla que muestra el diseñador de aplicaciones lógicas y el conector de Office 365 Outlook seleccionado.](./media/quickstart-create-first-logic-app-workflow/select-connector.png)

   Ahora puede buscar más fácilmente y seleccionar la acción que desea usar, por ejemplo, `send an email`:

   ![Captura de pantalla del diseñador de aplicaciones lógicas y una lista con acciones filtradas.](./media/quickstart-create-first-logic-app-workflow/filtered-actions-list.png)

1. Si el conector de correo electrónico seleccionado le pide que autentique su identidad, complete ese paso ahora para crear una conexión entre la aplicación lógica y el servicio de correo electrónico.

   > [!NOTE]
   > En este ejemplo en concreto, la identidad se autentica manualmente. Sin embargo, los conectores que requieren autenticación difieren en los tipos de autenticación que admiten. También tiene opciones para configurar la manera en que desea controlar la autenticación. Por ejemplo, si se usan plantillas de Azure Resource Manager para la implementación, es posible parametrizar y mejorar la seguridad de las entradas que se cambian con frecuencia o con facilidad, como la información de la conexión. Para más información, consulte los temas siguientes:
   >
   > * [Parámetros de plantilla para la implementación](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters)
   > * [Autorización de conexiones de OAuth](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md#authorize-oauth-connections)
   > * [Autenticación y acceso con identidades administradas](../logic-apps/create-managed-service-identity.md)
   > * [Autenticación de conexiones para la implementación de aplicaciones lógicas](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#authenticate-connections)

1. En la acción **Enviar un correo electrónico**, especifique la información que desea incluir en el correo electrónico.

   1. En el cuadro **Para**, escriba la dirección de correo electrónico del destinatario. Para realizar pruebas, puede usar su dirección de correo electrónico.

      Por ahora, ignore la lista **Agregar contenido dinámico** que aparece. Al hacer clic en algunos cuadros de edición, esta lista aparece y muestra las salidas disponibles del paso anterior que se pueden incluir como entradas para la acción actual.

   1. En el cuadro **Asunto**, escriba este texto con un espacio en blanco final: `New RSS item: `

      ![Captura de pantalla del diseñador de aplicaciones lógicas con la acción "Enviar un correo electrónico" y el cursor dentro del cuadro de propiedad "Asunto".](./media/quickstart-create-first-logic-app-workflow/send-email-subject.png)

   1. En la lista **Agregar contenido dinámico**, seleccione **Título de fuente**, que es la salida del desencadenador "Cuando se publica un elemento de fuente", que hace que el título del elemento RSS esté disponible para su uso.

      ![Captura de pantalla del diseñador de aplicaciones lógicas con la acción "Enviar un correo electrónico" y el cursor dentro del cuadro de propiedad "Asunto" con una lista de contenido dinámico abierta y la salida seleccionada, "Título de fuente".](./media/quickstart-create-first-logic-app-workflow/send-email-subject-dynamic-content.png)

      > [!TIP]
      > En la lista de contenido dinámico, si no aparece ninguna salida del desencadenador "Cuando se publica un elemento de fuente", junto al encabezado de la acción, seleccione **Ver más**.
      > 
      > ![Captura de pantalla del diseñador de aplicaciones lógicas con la lista de contenido dinámico abierta y la opción "Ver más" seleccionada para el desencadenador.](./media/quickstart-create-first-logic-app-workflow/dynamic-content-list-see-more-actions.png)

      Cuando haya terminado, el asunto del correo electrónico será similar a este ejemplo:

      ![Captura del diseñador de aplicaciones lógicas con la acción "Enviar un correo electrónico" y un asunto de correo electrónico de ejemplo con la propiedad "Título de fuente" incluida.](./media/quickstart-create-first-logic-app-workflow/send-email-feed-title.png)

      Si un bucle "Para cada uno" aparece en el diseñador, habrá seleccionado un token de una matriz, como el token **categorías-elemento**. Para este tipo de token, el diseñador añade automáticamente este bucle alrededor de la acción que hace referencia a ese token. De este modo, la aplicación lógica realiza la misma acción en cada elemento de la matriz. Para quitar el bucle, elija el botón de  **puntos suspensivos** ( **...** ) de la barra de título del bucle y luego **Eliminar**.

   1. En el cuadro **Cuerpo**, escriba este texto y seleccione estos tokens para el cuerpo del correo electrónico. Para agregar líneas en blanco en el cuadro de edición, presione Mayús + Entrar.

      ![Captura de pantalla del diseñador de aplicaciones lógicas con la acción "Enviar un correo electrónico" y las propiedades seleccionadas dentro del cuadro "Cuerpo".](./media/quickstart-create-first-logic-app-workflow/send-email-body.png)

      | Propiedad | Descripción |
      |----------|-------------|
      | **Título de fuente** | Título del elemento |
      | **Fuente publicada el** | Fecha y hora de publicación del elemento |
      | **Vínculo de fuente principal** | Dirección URL del elemento |
      |||

1. Guarde la aplicación lógica.

A continuación, pruebe la aplicación lógica.

## <a name="run-your-logic-app"></a>Ejecución de la aplicación lógica

Para iniciar manualmente la aplicación lógica, en la barra de herramientas del diseñador, elija **Ejecutar**. También puede esperar a que la aplicación lógica compruebe la fuente RSS basada en la programación especificada (cada minuto).

![Captura de pantalla del diseñador de aplicaciones lógicas con el botón "Ejecutar" seleccionado en la barra de herramientas del diseñador.](./media/quickstart-create-first-logic-app-workflow/run-logic-app-test.png)

Si la fuente RSS tiene nuevos elementos, la aplicación lógica envía un correo electrónico para cada uno de ellos. En caso contrario, la aplicación lógica espera hasta el siguiente intervalo antes de volver a comprobar. Si no obtiene los mensajes de correo electrónico, compruebe la carpeta de correo electrónico no deseado.

Por ejemplo, este es un correo electrónico de ejemplo que envía esta aplicación lógica.

![Captura de pantalla del correo electrónico de ejemplo recibido cuando aparece el nuevo elemento de fuente RSS.](./media/quickstart-create-first-logic-app-workflow/monitor-rss-feed-email.png)

Técnicamente, cuando el desencadenador comprueba la fuente RSS y encuentra nuevos elementos, el desencadenador se activa y el motor de Azure Logic Apps crea una instancia de flujo de trabajo de la aplicación lógica que ejecuta las acciones en el flujo de trabajo. Si el desencadenador no encuentra nuevos elementos, no se activa y "omite" crear instancias de flujo de trabajo.

Enhorabuena, acaba de compilar y ejecutar correctamente su primera aplicación lógica con Azure Portal.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite este ejemplo, elimine el grupo de recursos que contiene la aplicación lógica y los recursos relacionados.

1. En el cuadro de búsqueda de Azure, escriba `resource groups` y seleccione **Grupos de recursos**.

   ![Captura de pantalla del cuadro de búsqueda de Azure Portal con el término de búsqueda "Grupos de recursos".](./media/quickstart-create-first-logic-app-workflow/find-resource-groups.png)

1. Busque y seleccione el grupo de recursos de la aplicación lógica. En el panel **Información general**, elija **Eliminar grupo de recursos**.

   ![Captura de pantalla que muestra Azure Portal con el grupo de recursos seleccionado y el botón para "Eliminar grupo de recursos".](./media/quickstart-create-first-logic-app-workflow/delete-resource-group.png)

1. Cuando aparezca el panel de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

   ![Captura de pantalla de Azure Portal con el panel de confirmación y el nombre del grupo de recursos que se va a eliminar.](./media/quickstart-create-first-logic-app-workflow/delete-resource-group-2.png)

> [!NOTE]
> Cuando se elimina una aplicación lógica, no se crean instancias de nuevas ejecuciones. Todas las ejecuciones nuevas y pendientes se cancelan. Si tiene miles de ejecuciones, la cancelación puede tardar bastante tiempo en completarse.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado su primera aplicación lógica que comprueba si hay actualizaciones de RSS según la programación especificada (cada minuto) y realiza una acción (envía un correo electrónico) cuando hay actualizaciones. Para más información, continúe con este tutorial para crear flujos de trabajo basados en programación más avanzados:

> [!div class="nextstepaction"]
> [Comprobación del tráfico con una aplicación lógica basada en una programación](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)
