---
title: Conexión a Office 365 Outlook
description: Automatización de tareas y flujos de trabajo que administran el correo electrónico, los contactos y los calendarios en Office 365 Outlook con Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: article
ms.date: 07/27/2020
tags: connectors
ms.openlocfilehash: e0e152b3c0e10c34bc6213ddf867b79c58d1d767
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87832984"
---
# <a name="manage-email-contacts-and-calendars-in-office-365-outlook-by-using-azure-logic-apps"></a>Administración del correo electrónico, los contactos y los calendarios en Office 365 Outlook con Azure Logic Apps

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el [conector de Office 365 Outlook](/connectors/office365connector/), puede crear tareas y flujos de trabajo automatizados que administren su cuenta de Office 365 mediante la compilación de aplicaciones lógicas. Por ejemplo, puede automatizar estas tareas:

* Recibir, enviar y responder mensajes de correo electrónico. 
* Programar reuniones en el calendario.
* Agregar y editar contactos. 

Puede usar cualquier desencadenador para iniciar el flujo de trabajo (por ejemplo, cuando llega un nuevo correo electrónico, cuando se actualiza un elemento del calendario o cuando se produce un evento en un servicio diferencial, como Salesforce). Puede usar acciones que respondan al evento desencadenador, como enviar un correo electrónico o crear un nuevo evento de calendario. 

> [!NOTE]
> Para automatizar tareas para una cuenta @outlook.com o @hotmail.com, use el [conector de Outlook.com](../connectors/connectors-create-api-outlook.md).

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 

* Una [cuenta de Office 365](https://www.office.com/).

* La aplicación lógica donde quiere acceder a su cuenta de Office 365 Outlook. Para iniciar el flujo de trabajo con un desencadenador de Office 365 Outlook, necesita una [aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para agregar una acción de Office 365 Outlook al flujo de trabajo, la aplicación lógica ya debe tener un desencadenador.

## <a name="add-a-trigger"></a>Incorporación de un desencadenador

Un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) es un evento que inicia el flujo de trabajo en la aplicación lógica. Esta aplicación lógica de ejemplo usa un desencadenador de "sondeo" que comprueba cualquier evento de calendario actualizado en la cuenta de correo electrónico, según el intervalo y la frecuencia especificados.

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en blanco en el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba `office 365 outlook` como filtro. En este ejemplo se selecciona **Cuando un evento próximo va a comenzar pronto**.
   
   ![Seleccionar el desencadenador para iniciar la aplicación lógica](./media/connectors-create-api-office365-outlook/office365-trigger.png)

1. Si se le pide que inicie sesión, proporcione sus credenciales de Office 365 para que la aplicación lógica pueda conectarse a la cuenta. O bien, si la conexión ya existe, especifique la información de las propiedades del desencadenador.

   > [!NOTE]
   > La conexión no expira hasta que se revoca, incluso si cambia las credenciales de inicio de sesión. Para obtener más información, consulte [Vigencia de tokens configurables de Azure Active Directory (versión preliminar pública)](../active-directory/develop/active-directory-configurable-token-lifetimes.md).

   En este ejemplo se selecciona el calendario que comprueba el desencadenador, por ejemplo:

   ![Configurar las propiedades del desencadenador](./media/connectors-create-api-office365-outlook/select-calendar.png)

1. En el desencadenador, establezca los valores de **Frecuencia** e **Intervalo**. Para agregar otras propiedades del desencadenador disponibles, como **Zona horaria**, selecciónelas en la lista **Agregar nuevo parámetro**.

   Por ejemplo, si desea que el desencadenador compruebe el calendario cada 15 minutos, establezca el valor de **Frecuencia** en **Minuto** y el de **Intervalo** en `15`. 

   ![Establecer la frecuencia y el intervalo del desencadenador](./media/connectors-create-api-office365-outlook/calendar-settings.png)

1. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

Ahora agregue una acción que se ejecute cuando se active el desencadenador. Por ejemplo, puede agregar la acción **Enviar mensaje** de Twilio, que envía un mensaje de texto cuando faltan 15 minutos para que comience un evento de calendario.

## <a name="add-an-action"></a>Agregar una acción

Una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) es una operación que se ejecuta mediante el flujo de trabajo de la aplicación lógica. Esta aplicación lógica de ejemplo crea un nuevo contacto en Office 365 Outlook. Para crear el contacto puede utilizar la salida de otro desencadenador u otra acción. Por ejemplo, supongamos que la aplicación lógica usa el desencadenador Dynamics 365, **Al crear un registro**. Después, agregue la acción **Crear contacto** de Office 365 Outlook y utilice las salidas del desencadenador de SalesForce para crear el contacto nuevo.

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. Para agregar una acción como último paso del flujo de trabajo, seleccione **Nuevo paso**. 

   Para agregar una acción entre un paso y otro, mueva el puntero por encima de la flecha entre ellos. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba `office 365 outlook` como filtro. En este ejemplo se selecciona **Crear contacto**.

   ![Seleccione la acción que se ejecutará en la aplicación lógica](./media/connectors-create-api-office365-outlook/office365-actions.png) 

1. Si se le pide que inicie sesión, proporcione sus credenciales de Office 365 para que la aplicación lógica pueda conectarse a la cuenta. O bien, si la conexión ya existe, especifique la información de las propiedades de la acción.

   > [!NOTE]
   > La conexión no expira hasta que se revoca, incluso si cambia las credenciales de inicio de sesión. Para obtener más información, consulte [Vigencia de tokens configurables de Azure Active Directory (versión preliminar pública)](../active-directory/develop/active-directory-configurable-token-lifetimes.md).

   En este ejemplo se selecciona la carpeta de contactos en la que la acción crea el nuevo contacto, por ejemplo:

   ![Configurar las propiedades de la acción](./media/connectors-create-api-office365-outlook/select-contacts-folder.png)

   Para agregar otras propiedades de la acción disponibles, selecciónelas en la lista **Agregar nuevo parámetro**.

1. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

## <a name="connector-reference"></a>Referencia de conectores

Si necesita detalles técnicos sobre este conector, como los desencadenadores, las acciones y los límites que se describen en el archivo de Swagger del conector, consulte la [página de referencia del conector](/connectors/office365/). 

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)