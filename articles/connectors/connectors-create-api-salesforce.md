---
title: Conexión con Salesforce desde Azure Logic Apps
description: Automatización de las tareas y los flujos de trabajo que supervisan, crean y administran registros y trabajos de Salesforce mediante Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/24/2018
tags: connectors
ms.openlocfilehash: 9950951ab5189c8c7b72de78bca9465ec5f22748
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290587"
---
# <a name="monitor-create-and-manage-salesforce-resources-by-using-azure-logic-apps"></a>Supervisión, creación y administración de los recursos de Salesforce mediante Azure Logic Apps

Con Azure Logic Apps y el conector de Salesforce, puede crear flujos de trabajo y tareas automatizadas para los recursos de Salesforce, como registros, trabajos y objetos, por ejemplo:

* Supervisar cuándo se crean o cambian los registros. 
* Crear, obtener y administrar los trabajos y los registros, incluidas las acciones de inserción, actualización y eliminación.

Puede usar desencadenadores de Salesforce que obtengan respuestas de Salesforce y hagan que la salida esté disponible para otras acciones. Puede utilizar acciones en las aplicaciones lógicas para realizar tareas con los recursos de Salesforce. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). 

* Una [cuenta de Salesforce](https://salesforce.com/)

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica donde quiere acceder a su cuenta de Salesforce. Para comenzar con un desencadenador de Salesforce, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar una acción de Salesforce, inicie la aplicación lógica con otro desencadenador, por ejemplo, el de **periodicidad**.

## <a name="connect-to-salesforce"></a>Conexión con Salesforce

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. Elija una ruta de acceso: 

   * Para las aplicaciones lógicas en blanco, en el cuadro de búsqueda, escriba "salesforce" como filtro. 
   En la lista de desencadenadores, seleccione el que desee. 

     O bien

   * Para las aplicaciones lógicas existentes, en el paso donde desea agregar una acción, elija **Nuevo paso**. En el cuadro de búsqueda, escriba "salesforce" como filtro. En la lista de acciones, seleccione la que desee.

1. Si se le pide que inicie sesión en Salesforce, hágalo ahora y permita el acceso.

   Sus credenciales autorizan a la aplicación lógica a crear una conexión con Salesforce y acceder a sus datos.

1. Proporcione los detalles necesarios para el desencadenador o la acción seleccionados y continúe con la compilación del flujo de trabajo de la aplicación lógica.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener detalles técnicos acerca de desencadenadores, acciones y límites, que se describen en la descripción de OpenAPI (antes Swagger) del conector, consulte la [página de referencia](/connectors/salesforce/) del conector.

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Para enviar ideas sobre características o votar sobre ellas, visite el [sitio de comentarios de los usuarios de Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)
