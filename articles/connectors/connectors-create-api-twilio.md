---
title: Conexión a Twilio desde Azure Logic Apps
description: Automatización de tareas y flujos de trabajo que administran mensajes IP, MMS y SMS globales a través de la cuenta de Twilio con Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/25/2018
tags: connectors
ms.openlocfilehash: d144960972f5a1b45e88cc3a0ea015925cae3b91
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288147"
---
# <a name="manage-messages-in-twilio-with-azure-logic-apps"></a>Administración de mensajes en Twilio con Azure Logic Apps

Con Azure Logic Apps y el conector de Twilio, puede crear tareas automatizadas y flujos de trabajo que obtienen, envían y enumeran mensajes en Twilio, entre los que se incluyen mensajes SMS, MMS e IP globales. Puede usar estas acciones para realizar tareas con su cuenta de Twilio. También puede hacer que otras acciones usen la salida de las acciones de Twilio. Por ejemplo, cuando llega un nuevo mensaje, puede enviar el contenido del mensaje con el conector de Slack. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). 

* De [Twilio](https://www.twilio.com/): 

  * El identificador de cuenta de Twilio y el [token de autenticación](https://support.twilio.com/hc/en-us/articles/223136027-Auth-Tokens-and-How-to-Change-Them), que puede encontrar en el panel de Twilio.

    Las credenciales autorizan a la aplicación lógica para que cree una conexión con la cuenta de Twilio y acceda a ella desde la aplicación lógica. 
    Si usa una cuenta de prueba de Twilio, solamente podrá enviar SMS a números de teléfono *comprobados*.

  * Un número de teléfono de Twilio comprobado que pueda enviar SMS

  * Un número de teléfono de Twilio comprobado que pueda recibir SMS

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica desde donde quiere acceder a la cuenta de Twilio. Para usar una acción de Twilio, inicie la aplicación lógica con otro desencadenador, por ejemplo, el de **periodicidad**.

## <a name="connect-to-twilio"></a>Conexión a Twilio

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. Elija una ruta de acceso: 

     * En el último paso para agregar una acción, elija **Nuevo paso**. 

       O bien

     * Entre los pasos en los que desee agregar una acción, mueva el puntero sobre la flecha. 
     Elija el signo más ( **+** ) que aparece y seleccione **Agregar una acción**.
     
       En el cuadro de búsqueda, escriba "twilio" como filtro. 
       En la lista de acciones, seleccione la que desee.

1. Proporcione la información necesaria para la conexión y, a continuación, seleccione **Crear**:

   * El nombre que usar para su conexión
   * El identificador de su cuenta de Twilio 
   * El token de acceso (autenticación) de Twilio

1. Proporcione los detalles necesarios para la acción seleccionada y continúe con la creación del flujo de trabajo de la aplicación lógica.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener detalles técnicos acerca de desencadenadores, acciones y límites, que se describen en la descripción de OpenAPI (antes Swagger) del conector, consulte la [página de referencia](/connectors/twilio/) del conector.

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Para enviar ideas sobre características o votar sobre ellas, visite el [sitio de comentarios de los usuarios de Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)
