---
title: Conexión a SMTP desde Azure Logic Apps
description: Automatización de las tareas y los flujos de trabajo que envían correo electrónico a través de la cuenta de SMTP (protocolo simple de transferencia de correo) mediante Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/25/2018
tags: connectors
ms.openlocfilehash: 9989d0ebd95bfe5ee49be2ba76b73e07630b519a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283952"
---
# <a name="send-email-from-your-smtp-account-with-azure-logic-apps"></a>Envío de correo electrónico desde la cuenta de SMTP con Azure Logic Apps

Con Azure Logic Apps y el conector de SMTP (protocolo simple de transferencia de correo) puede crear tareas automatizadas y flujos de trabajo para el envío de correo electrónico desde la cuenta de SMTP. También puede hacer que otras acciones usen la salida de las acciones de SMTP. Por ejemplo, una vez enviado un correo electrónico desde SMTP, puede notificar al equipo de Slack con el conector correspondiente. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). 

* Cuenta de SMTP y credenciales del usuario

  Las credenciales autorizan a la aplicación lógica para que cree una conexión con la cuenta de SMTP y acceda a ella.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica desde donde quiere acceder a la cuenta de SMTP. Para usar una acción de SMTP, inicie la aplicación lógica con un desencadenador, por ejemplo, de Salesforce, si tiene una cuenta de Salesforce.

  Una opción es iniciar la aplicación lógica con el desencadenador **Al crear un registro** de Salesforce. 
  Este desencadenador se activa cada vez que se crea un nuevo registro, por ejemplo, un cliente potencial, en Salesforce. 
  A este desencadenador le puede seguir la acción **Send Email** de SMTP. De este modo, cuando se crea el nuevo registro, la aplicación lógica envía un correo electrónico desde la cuenta de SMTP para notificarlo.

## <a name="connect-to-smtp"></a>Conexión a SMTP

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. En el último paso para agregar una acción de SMTP, elija **Nuevo paso**. 

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. 
   Elija el signo más ( **+** ) que aparece y seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba "smtp" como filtro. En la lista de acciones, seleccione la que desee.

1. Cuando se le solicite, proporcione esta información de conexión:

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre de la conexión** | Sí | Un nombre para la conexión al servidor SMTP | 
   | **Dirección del servidor SMTP** | Sí | La dirección del servidor SMTP | 
   | **Nombre de usuario** | Sí | El nombre de usuario de la cuenta de SMTP | 
   | **Contraseña** | Sí | La contraseña de la cuenta de SMTP | 
   | **Puerto del servidor SMTP** | No | El puerto específico del servidor SMTP que desea usar | 
   | **¿Quiere habilitar SSL?** | No | Activar o desactivar el cifrado TLS/SSL. | 
   |||| 

1. Proporcione la información necesaria para la acción seleccionada. 

1. Guarde la aplicación lógica o continúe la compilación del flujo de trabajo de la aplicación lógica.

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como los desencadenadores, las acciones y los límites que se describen en el archivo de Swagger del conector, vea la [página de referencia del conector](/connectors/smtpconnector/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa en su lugar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)
