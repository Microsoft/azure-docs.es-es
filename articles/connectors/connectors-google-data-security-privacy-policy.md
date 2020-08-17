---
title: Directivas de privacidad y seguridad de datos para conectores de Google
description: Más información sobre el impacto que tienen las directivas de privacidad y seguridad de Google en los conectores de Google, como Gmail, en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: conceptual
ms.date: 06/05/2020
ms.openlocfilehash: b82b79d076871b961d5c496bc70beac1b5067b2d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288112"
---
# <a name="data-security-and-privacy-policies-for-google-connectors-in-azure-logic-apps"></a>Directivas de privacidad y seguridad de datos para conectores de Google en Azure Logic Apps

A partir del **1 de mayo de 2020**, los cambios debidos a las [directivas de privacidad y seguridad de datos](https://www.blog.google/technology/safety-security/project-strobe/) de Google pueden afectar a los flujos de trabajo de aplicaciones lógicas que usan el [conector de Gmail](/connectors/gmail/). Si las aplicaciones lógicas usan el conector de Gmail con una cuenta de consumidor de Gmail (dirección de correo electrónico que termina con @gmail.com o @googlemail.com), las aplicaciones lógicas solo pueden usar [desencadenadores, acciones y conectores aprobados por Google](#approved-connectors) concretos.

> [!NOTE]
> Si las aplicaciones lógicas usan el conector de Gmail con una cuenta de negocio de G-Suite (dirección de correo con un dominio personalizado), las aplicaciones lógicas no se ven afectadas y no tienen restricciones en el uso del conector de Gmail.

## <a name="affected-logic-apps"></a>Aplicaciones lógicas afectadas

Si tiene aplicaciones lógicas que usan el conector de Gmail, recibirá un correo electrónico sobre las aplicaciones lógicas potencialmente afectadas. Sin embargo, a partir del **15 de junio de 2020**, se deshabilitará cualquier flujo de trabajo no compatible. Puede realizar cualquiera de estas acciones:

* Actualice las aplicaciones lógicas afectadas [siguiendo los pasos descritos en este tema](#update-affected-workflows). Debe crear una aplicación cliente de Google, que proporciona un identificador y un secreto de cliente que se usa para la autenticación en el desencadenador o la acción de Gmail.

* Actualice las aplicaciones lógicas afectadas para que usen solo los [conectores aprobados por Google](#approved-connectors) antes de volver a habilitar las aplicaciones lógicas deshabilitadas.

<a name="approved-connectors"></a>

## <a name="google-approved-connectors"></a>Conectores aprobados por Google

En esta directiva, cuando se usa una cuenta de consumidor de Gmail, puede usar el conector de Gmail solo con servicios aprobados por Google específicos, que están sujetos a cambios. Nuestros equipos de ingeniería continúan trabajando con Google para agregar más servicios a esta lista. Por ahora, estos son los desencadenadores, las acciones y los conectores aprobados por Google que puede usar en el mismo flujo de trabajo de aplicación lógica con el conector de Gmail al usar una cuenta de consumidor de Gmail:

* Desencadenadores y acciones integrados de Logic Apps: Batch, control, operaciones de datos, fecha y hora, archivo plano, Liquid, solicitud, programación, variables y XML.

  Los desencadenadores y acciones integrados que no están aprobados por Google, como los de HTTP, Azure Functions, Azure Logic Apps y otros, hacen que una aplicación lógica no sea compatible con el conector de Gmail, ya que la aplicación puede enviar o recibir datos desde cualquier lugar.

* Servicios de Google: Gmail, Google Calendar, Contactos de Google, Google Drive, Hojas de cálculo de Google y Tareas de Google.

* Servicios de Microsoft aprobados: Dynamics 365, Excel Online, Microsoft Teams, Office 365, OneDrive y SharePoint Online.

* Conectores para orígenes de datos administrados por el cliente: FTP, RSS, SFTP, SMTP y SQL Server.

## <a name="non-compliant-examples"></a>Ejemplos no compatibles

Estos son algunos ejemplos que usan el conector de Gmail con desencadenadores y acciones integrados o conectores administrados que no están aprobados por Google:

* Esta aplicación lógica usa el conector de Gmail con el desencadenador integrado de HTTP:

  ![Aplicación lógica no compatible: ejemplo 1](./media/connectors-google-data-security-privacy-policy/not-compliant-logic-app-1.png)
  
  La aplicación lógica también usa el conector de Google Calendar, que está aprobado.

* Esta aplicación lógica usa el conector de Gmail con el conector de Azure Blob Storage:

  ![Aplicación lógica no compatible: ejemplo 2](./media/connectors-google-data-security-privacy-policy/not-compliant-logic-app-2.png)

* Esta aplicación lógica usa el conector de Gmail con el conector de Twitter:

  ![Aplicación lógica no compatible: ejemplo 3](./media/connectors-google-data-security-privacy-policy/not-compliant-logic-app-3.png)

Para obtener la información más reciente, consulte la [documentación de referencia técnica del conector de Gmail](/connectors/gmail/).

<a name="update-affected-workflows"></a>

## <a name="steps-for-affected-logic-apps"></a>Pasos para las aplicaciones lógicas afectadas

Si tiene que usar el conector de Gmail con una cuenta de consumidor de Gmail y los conectores no aprobados de Google en una aplicación lógica, puede crear su propia aplicación de Google para su uso personal o interno en su empresa. En este escenario, estos son los pasos de alto nivel que debe seguir:

1. Cree una aplicación de cliente de Google mediante la [consola de la API de Google](https://console.developers.google.com).

1. En el conector de Gmail, use los valores de identificador y secreto de cliente de la aplicación cliente de Google.

Para obtener más información, consulte la [documentación de referencia técnica del conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application).

### <a name="create-google-client-app"></a>Creación de una aplicación cliente de Google

Para configurar un proyecto para la aplicación cliente, use el [asistente para la Consola de APIs de Google](https://console.developers.google.com/start/api?id=gmail&credential=client_key) y siga las instrucciones. O bien, para obtener pasos detallados, revise las instrucciones de la [documentación de referencia técnica del conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application).

Cuando haya terminado, la pantalla tendrá un aspecto similar al de este ejemplo, pero tendrá sus propios valores de **id. de cliente** y **secreto de cliente**, que, posteriormente usará en la aplicación lógica.

![Escriba el id. de cliente y el secreto de cliente de la aplicación de cliente de Google.](./media/connectors-google-data-security-privacy-policy/google-api-console.png)

### <a name="use-client-app-settings-in-logic-app"></a>Uso de la configuración de la aplicación cliente en la aplicación lógica

Para usar el id. de cliente y el secreto de cliente desde la aplicación de cliente de Google en el desencadenador o acción de Gmail, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. Si va a agregar un nuevo desencadenador o acción de Gmail y va a crear una conexión completamente nueva, continúe con el paso siguiente. En caso contrario, en el desencadenador o acción de Gmail, seleccione **Cambiar conexión** > **Agregar nueva**, por ejemplo:

   ![Seleccione "Cambiar conexión" > "Agregar nueva"](./media/connectors-google-data-security-privacy-policy/change-gmail-connection.png)

1. Proporcione la información de conexión; por ejemplo:

   ![Proporcionar la información de conexión](./media/connectors-google-data-security-privacy-policy/authentication-type-bring-your-own.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Tipo de autenticación** | **Traiga su propia aplicación** | Especifica que usará su propia aplicación cliente para la autenticación. |
   | **Id. de cliente** | <*client-ID*> | Identificador de cliente de la aplicación cliente de Google |
   | **Secreto de cliente** | <*secreto-de-cliente*> | Secreto de cliente de la aplicación cliente de Google |
   ||||

1. Al acabar, seleccione **Iniciar sesión**.

   Aparece una página que muestra la aplicación cliente que ha creado. Si usa una cuenta de consumidor de Gmail, es posible que aparezca una página que muestre que Google no ha verificado la aplicación de cliente y que le solicite que antes permita el acceso a su cuenta de Google.

   ![Mensaje para acceder a la cuenta de Google](./media/connectors-google-data-security-privacy-policy/allow-access-authorized-domain.png)

1. Si es necesario, seleccione **Permitir**.

   Ahora puede usar el conector de Gmail sin restricciones en la aplicación lógica.

## <a name="next-steps"></a>Pasos siguientes

Más información acerca del [conector de Gmail](/connectors/gmail/)

