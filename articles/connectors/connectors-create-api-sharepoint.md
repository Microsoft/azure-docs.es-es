---
title: Conexión a SharePoint desde Azure Logic Apps
description: Automatización de tareas y flujos de trabajo que supervisan y administran los recursos en SharePoint Online o SharePoint Server en el entorno local con Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/25/2018
tags: connectors
ms.openlocfilehash: c72330792e508361830c1bf391f85eefe78bdd1e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283986"
---
# <a name="monitor-and-manage-sharepoint-resources-with-azure-logic-apps"></a>Supervisión y administración de los recursos de SharePoint con Azure Logic Apps

Con Azure Logic Apps y el conector de SharePoint puede crear tareas automatizadas y flujos de trabajo que supervisen y administren los recursos, como archivos, carpetas, listas, elementos, personas, etc., en SharePoint Online o SharePoint Server en el entorno local, como por ejemplo:

* Supervisar cuándo se crean, se cambian o se eliminan archivos o elementos.
* Crear, obtener, actualizar o eliminar elementos.
* Agregar, obtener o eliminar datos adjuntos. Obtener el contenido de los datos adjuntos.
* Crear, copiar, actualizar o eliminar archivos. 
* Actualizar propiedades de archivo. Obtener el contenido, los metadatos o las propiedades de un archivo.
* Enumerar o extraer carpetas.
* Obtener listas o vistas de lista.
* Definir el estado de aprobación del contenido.
* Resolver personas.
* Enviar solicitudes HTTP a SharePoint.
* Obtener valores de entidad.

Puede usar desencadenadores que obtengan respuestas de SharePoint y hagan que la salida esté disponible para otras acciones. Puede utilizar acciones en las aplicaciones lógicas para realizar tareas en SharePoint. También puede hacer que otras acciones usen la salida de las acciones de SharePoint. Por ejemplo, si suele capturar archivos de SharePoint, puede enviar mensajes a su equipo mediante el conector de Slack.
Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). 

* Dirección del sitio de SharePoint y credenciales de usuario

  Las credenciales autorizan a la aplicación lógica para que cree una conexión con la cuenta de SharePoint y acceda a ella. 

* Para que pueda conectar las aplicaciones lógicas con sistemas locales como SharePoint Server, debe [instalar y configurar una puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md). De este modo, cuando cree la conexión de SharePoint Server para la aplicación lógica, puede especificar que se use esta instalación de puerta de enlace.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica desde donde quiere acceder a la cuenta de SharePoint. Para comenzar con un desencadenador de SharePoint, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar una acción de SharePoint, inicie la aplicación lógica con un desencadenador, por ejemplo, de Salesforce, si tiene una cuenta de Salesforce.

  Una opción es iniciar la aplicación lógica con el desencadenador **Al crear un registro** de Salesforce. 
  Este desencadenador se activa cada vez que se crea un nuevo registro, por ejemplo, un cliente potencial, en Salesforce. 
  A este desencadenador le puede seguir la acción **Create file** de SharePoint. De este modo, cuando se crea el nuevo registro, la aplicación lógica crea un archivo en SharePoint con información sobre él.

## <a name="connect-to-sharepoint"></a>Conexión a SharePoint

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. Para las aplicaciones lógicas en blanco, en el cuadro de búsqueda, escriba "sharepoint" como filtro. En la lista de desencadenadores, seleccione el que desee. 

   O bien

   Para las aplicaciones lógicas existentes, en el último paso para agregar una acción de SharePoint, elija **Nuevo paso**. 
   En el cuadro de búsqueda, escriba "sharepoint" como filtro. 
   En la lista de acciones, seleccione la que desee.

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. 
   Elija el signo más ( **+** ) que aparece y seleccione **Agregar una acción**.

1. Cuando se le solicite el inicio de sesión, proporcione la información de conexión necesaria. Si usa SharePoint Server, asegúrese de seleccionar **Conectarse mediante una puerta de enlace de datos local**. Cuando termine, seleccione **Crear**.

1. Proporcione los detalles necesarios para el desencadenador o la acción seleccionados y continúe con la compilación del flujo de trabajo de la aplicación lógica.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener detalles técnicos acerca de desencadenadores, acciones y límites, que se describen en la descripción de OpenAPI (antes Swagger) del conector, consulte la [página de referencia](/connectors/sharepoint/) del conector.

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Para enviar ideas sobre características o votar sobre ellas, visite el [sitio de comentarios de los usuarios de Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

