---
title: Conexión a Slack desde Azure Logic Apps
description: Automatice tareas y flujos de trabajo que supervisan archivos y administran canales, grupos y mensajes en la cuenta de Slack con Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/25/2018
tags: connectors
ms.openlocfilehash: 75f4cea0e5cd2b0bf42c28cbd81e199115834e11
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283969"
---
# <a name="monitor-and-manage-slack-with-azure-logic-apps"></a>Supervisión y administración de Slack con Azure Logic Apps

Con Azure Logic Apps y el conector Slack, puede crear tareas y flujos de trabajo automatizados que supervisan los archivos de Slack y administran los canales, mensajes o grupos de Slack. Por ejemplo:

* Supervisar cuándo se crean nuevos archivos
* Crear, enumerar y unirse a canales 
* Publicar mensajes
* Crear grupos y establecer "no molestar"

Puede usar desencadenadores que obtengan respuestas de su cuenta de Slack y permitan que la salida esté disponible para otras acciones. Puede usar acciones que realicen tareas con su cuenta de Slack. También puede hacer que otras acciones usen la salida de las acciones de Slack. Por ejemplo, cuando se crea un nuevo archivo, puede enviar un correo electrónico con el conector Office 365 Outlook. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). 

* La cuenta y las credenciales de usuario de [Slack](https://slack.com/).

  Las credenciales autorizan a la aplicación lógica a crear una conexión con la cuenta de Slack y acceder a ella.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica desde donde quiere acceder a la cuenta de Slack. Para comenzar con un desencadenador de Slack, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar una acción de Slack, inicie la aplicación lógica con un desencadenador, por ejemplo, uno de Slack u otro desencadenador, como **Recurrence**.

## <a name="connect-to-slack"></a>Conexión a Slack

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. Para las aplicaciones lógicas en blanco, en el cuadro de búsqueda, escriba "slack" como filtro. En la lista de desencadenadores, seleccione el que desee. 

   O bien

   Para las aplicaciones lógicas existentes, en el último paso donde desea agregar una acción, elija **Nuevo paso**. 
   En el cuadro de búsqueda, escriba "slack" como filtro. 
   En la lista de acciones, seleccione la que desee.

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. 
   Elija el signo más ( **+** ) que aparece y seleccione **Agregar una acción**.

1. Si se le pide que inicie sesión en Slack, inicie sesión en el área de trabajo de Slack. 

   ![Inicio de sesión en el área de trabajo de Slack](./media/connectors-create-api-slack/slack-sign-in-workspace.png)

1. Autorice el acceso para la aplicación lógica.

   ![Autorización del acceso a Slack](./media/connectors-create-api-slack/slack-authorize-access.png)

1. Proporcione la información necesaria para el desencadenador o la acción seleccionados. Para continuar con la creación del flujo de trabajo de la aplicación lógica, agregue más acciones.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener detalles técnicos acerca de desencadenadores, acciones y límites, que se describen en la descripción de OpenAPI (antes Swagger) del conector, consulte la [página de referencia](/connectors/slack/) del conector.

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Para enviar ideas sobre características o votar sobre ellas, visite el [sitio de comentarios de los usuarios de Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

