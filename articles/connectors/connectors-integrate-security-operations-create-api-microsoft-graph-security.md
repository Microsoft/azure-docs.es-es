---
title: Integración y administración de las operaciones de seguridad y Microsoft Graph Security
description: Mejore la protección, detección y respuesta frente amenazas de su aplicación con Microsoft Graph Security y Azure Logic Apps
services: logic-apps
ms.suite: integration
author: preetikr
ms.author: preetikr
ms.reviewer: v-ching, estfan, logicappspm
ms.topic: article
ms.date: 02/21/2020
tags: connectors
ms.openlocfilehash: b08b5db5639d498aa6a6a47b7f7121cad565fe02
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87986375"
---
# <a name="improve-threat-protection-by-integrating-security-operations-with-microsoft-graph-security--azure-logic-apps"></a>Mejore la protección frente a amenazas mediante la integración de las operaciones de seguridad con Microsoft Graph Security y Azure Logic Apps.

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el conector [Microsoft Graph Security](/graph/security-concept-overview), puede mejorar cómo la aplicación detecta, protege y responde frente a las amenazas mediante la creación de flujos de trabajo automatizados para la integración de productos, servicios y asociados de seguridad de Microsoft. Por ejemplo, puede crear [cuadernos de estrategias de Azure Security Center](../security-center/workflow-automation.md) que supervisen y administren las entidades de seguridad de Microsoft Graph, como las alertas. Estos son algunos de los escenarios admitidos por el conector Microsoft Graph Security:

* Obtención de alertas basadas en consultas o por identificador de alerta. Por ejemplo, puede obtener una lista que incluya alertas de gravedad alta.

* Actualización de alertas. Por ejemplo, puede actualizar asignaciones de alertas, agregar comentarios a alertas o etiquetar alertas.

* Supervisión de cuándo se crean las alertas o se cambian mediante la creación de [suscripciones de alertas (webhooks)](/graph/api/resources/webhooks).

* Administración de las suscripciones de alertas. Por ejemplo, puede obtener las suscripciones activas, ampliar el tiempo de expiración de una suscripción o eliminar suscripciones.

El flujo de trabajo de la aplicación lógica puede usar acciones que obtengan respuestas del conector Microsoft Graph Security y permitir que esa salida esté disponible para otras acciones del flujo de trabajo. También puede tener otras acciones en el flujo de trabajo mediante la salida de las acciones del conector Microsoft Graph Security. Por ejemplo, si recibe alertas de gravedad alta a través del conector Microsoft Graph Security, puede enviar esas alertas en un mensaje de correo electrónico mediante el conector de Outlook. 

Para más información sobre Microsoft Graph Security, consulte la [introducción a Microsoft Graph Security API](https://aka.ms/graphsecuritydocs). Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) Si busca Microsoft Flow o PowerApps, consulte [¿Qué es Flow?](https://flow.microsoft.com/) o [¿Qué es PowerApps?](https://powerapps.microsoft.com/)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). 

* Para usar el conector Microsoft Graph Security, debe haber *otorgado explícitamente* consentimiento al administrador de inquilinos de Azure Active Directory (AD), que es parte de los [requisitos de autenticación de Microsoft Graph Security](https://aka.ms/graphsecurityauth). Este consentimiento requiere el identificador y el nombre de la aplicación del conector Microsoft Graph Security, que también se puede encontrar en [Azure Portal](https://portal.azure.com):

  | Propiedad | Value |
  |----------|-------|
  | **Nombre de la aplicación** | `MicrosoftGraphSecurityConnector` |
  | **Identificador de la aplicación** | `c4829704-0edc-4c3d-a347-7c4a67586f3c` |
  |||

  Para consentir el conector, el administrador de inquilinos de Azure AD puede seguir cualquiera de estos pasos:

  * [Conceda al administrador el consentimiento para aplicaciones de Azure AD](../active-directory/develop/v2-permissions-and-consent.md).

  * Durante la primera ejecución de la aplicación lógica, la aplicación puede solicitar el consentimiento del administrador de inquilinos de Azure AD mediante la [experiencia de consentimiento de aplicación](../active-directory/develop/application-consent-experience.md).
   
* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica a la que quieren acceder las entidades de Microsoft Graph Security, como las alertas. Para usar un desencadenador de seguridad de Microsoft Graph, necesita una aplicación lógica en blanco. Para usar una acción de seguridad de Microsoft Graph, necesita una aplicación lógica que comience por el desencadenador adecuado para su escenario.

## <a name="connect-to-microsoft-graph-security"></a>Conexión con Microsoft Graph Security 

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. Para las aplicaciones lógicas en blanco, agregue el desencadenador y todas las demás acciones que desee antes de agregar una acción de seguridad de Microsoft Graph.

   O bien

   Para aplicaciones lógicas existentes, en el último paso donde quiera agregar una acción de seguridad de Microsoft Graph, seleccione **New step** (Nuevo paso).

   O bien

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. Seleccione el signo más (+) que aparece y seleccione **Add an action** (Agregar una acción).

1. En el cuadro de búsqueda, escriba "microsoft graph security" como filtro. En la lista de acciones, seleccione la acción que desee.

1. Inicie sesión con sus credenciales de Microsoft Graph Security.

1. Proporcione los detalles necesarios para la acción seleccionada y continúe con la creación del flujo de trabajo de la aplicación lógica.

## <a name="add-triggers"></a>Incorporación de desencadenadores

En Azure Logic Apps, cada aplicación lógica debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts), que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica y empieza a ejecutar el flujo de trabajo de la aplicación.

> [!NOTE] 
> Cuando se activa un desencadenador, este procesa todas las nuevas alertas. Si no se reciben alertas, se omite la ejecución del desencadenador. El siguiente sondeo del desencadenador ocurre en función del intervalo de periodicidad especificado en las propiedades del desencadenador.

En este ejemplo se muestra cómo iniciar un flujo de trabajo de aplicaciones lógicas cuando se envían nuevas alertas a la aplicación.

1.  En Azure Portal o Visual Studio, cree una aplicación lógica en blanco que abra el Diseñador de aplicaciones lógicas. En este ejemplo se usa Azure Portal.

1.  En el cuadro de búsqueda del diseñador, escriba "microsoft graph security" como filtro. En la lista de desencadenadores, seleccione este desencadenador: **On all new alerts**

1.  En el desencadenador, proporcione información sobre las alertas que quiera supervisar. Para más propiedades, abra la lista **Agregar nuevo parámetro** y seleccione un parámetro para agregar esa propiedad al desencadenador.

   | Propiedad | Propiedad (JSON) | Obligatorio | Tipo | Descripción |
   |----------|-----------------|----------|------|-------------|
   | **Intervalo** | `interval` | Sí | Entero | Entero positivo que describe la frecuencia con la que se ejecuta el flujo de trabajo. Estos son los intervalos mínimo y máximo: <p><p>- Mes: 1-16 meses <br>- Día: 1-500 días <br>- Hora: 1-12 000 horas <br>- Minuto: 1-72 000 minutos <br>- Segundo: 1-9 999 999 segundos <p>Por ejemplo, si el intervalo es 6 y la frecuencia es "month", la periodicidad es cada 6 meses. |
   | **Frecuencia** | `frequency` | Sí | String | Unidad de tiempo que se usa para la periodicidad: **Segundo**, **Minuto**, **Hora**, **Día**, **Semana** o **Mes** |
   | **Zona horaria** | `timeZone` | No | String | Solo se aplica cuando se especifica una hora de inicio porque este desencadenador no acepta [diferencia horaria con UTC](https://en.wikipedia.org/wiki/UTC_offset). Seleccione la zona horaria que desea aplicar. |
   | **Hora de inicio** | `startTime` | No | String | Proporcione una fecha y hora de inicio en este formato: <p><p>AAAA-MM-DDThh:mm:ss si selecciona una zona horaria <p>O bien <p>AAAA-MM-DDThh:mm:ssZ si no selecciona una zona horaria <p>Por ejemplo, si desea la fecha del 18 de septiembre de 2017 a las 14:00, especifique "2017-09-18T14:00:00" y seleccione una zona horaria, como la hora estándar del Pacífico. O bien, especifique "2017-09-18T14:00:00Z" sin una zona horaria. <p>**Nota:** Esta hora de inicio tiene un máximo de 49 años en el futuro y debe seguir la [especificación de fecha y hora ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) en [formato de fecha y hora UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), pero sin una [diferencia horaria con UTC](https://en.wikipedia.org/wiki/UTC_offset). Si no se selecciona una zona horaria, debe agregar la letra "Z" al final sin espacios. Esta "Z" se refiere al equivalente de [hora náutica](https://en.wikipedia.org/wiki/Nautical_time). <p>Para las programaciones simples, la hora de inicio es la primera aparición, mientras que para programaciones complejas, el desencadenador no se activa antes de la hora de inicio. [ *¿De qué formas puedo usar la fecha y hora de inicio?* ](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#start-time) |
   ||||||

1.  Cuando esté listo, seleccione **Guardar** en la barra de herramientas del diseñador.

1.  Ahora, agregue a la aplicación lógica una o varias acciones, en función de las tareas que desea realizar con los resultados del desencadenador.

## <a name="add-actions"></a>Incorporación de una acción

Estos son detalles más específicos sobre el uso de las distintas acciones disponibles con el conector Microsoft Graph Security.

### <a name="manage-alerts"></a>Administrar alertas

Para filtrar, ordenar, u obtener los resultados más recientes, *solo* proporcione los [parámetros de consulta de ODATA admitidos por Microsoft Graph](/graph/query-parameters). *No especifique* la dirección URL base completa o la acción HTTP, por ejemplo, `https://graph.microsoft.com/v1.0/security/alerts`, ni la operación `GET` o `PATCH`. Este es un ejemplo específico que muestra los parámetros para una acción **Obtener alertas** cuando quiere una lista con alertas de gravedad alta:

`Filter alerts value as Severity eq 'high'`

Para más información sobre las consultas que puede usar con este conector, consulte la [documentación de referencia de alertas de Microsoft Graph Security](/graph/api/alert-list). Para crear experiencias mejoradas con este conector, aprenda más sobre las [alertas de propiedades de esquema](/graph/api/resources/alert) que admite el conector.

| Acción | Descripción |
|--------|-------------|
| **Obtener alertas** | Obtenga alertas filtrada por una o varias [propiedades de alerta](/graph/api/resources/alert), por ejemplo, `Provider eq 'Azure Security Center' or 'Palo Alto Networks'`. | 
| **Obtener alerta por identificador** | Obtenga una alerta específica según el identificador de alerta. | 
| **Actualizar alerta** | Actualice una alerta específica según el identificador de alerta. Para asegurarse de que pasa las propiedades obligatorias y modificables en la solicitud, consulte las [propiedades editables para alertas](/graph/api/alert-update). Por ejemplo, para asignar una alerta a un analista de seguridad para que puede investigarla, puede actualizar la propiedad **Assigned to** (Asignado a) de la alerta. |
|||

### <a name="manage-alert-subscriptions"></a>Administración de suscripciones de alertas

Microsoft Graph admite [*suscripciones*](/graph/api/resources/subscription) o [*webhooks*](/graph/api/resources/webhooks). Para obtener, actualizar, o eliminar suscripciones, proporcione los [parámetros de consulta de ODATA admitidos por Microsoft Graph](/graph/query-parameters) a la construcción de entidad de Microsoft Graph e incluya `security/alerts` seguido de la consulta de ODATA. *No incluya* la URL base, por ejemplo, `https://graph.microsoft.com/v1.0`. En cambio, use el formato de este ejemplo:

`security/alerts?$filter=status eq 'NewAlert'`

| Acción | Descripción |
|--------|-------------|
| **Creación de suscripciones** | [Crea una suscripción](/graph/api/subscription-post-subscriptions) que le informe sobre cualquier cambio. Puede filtrar esta suscripción por los tipos de alerta específicos que desee. Por ejemplo, puede crear una suscripción que le informe sobre las alertas de gravedad alta. |
| **Obtener suscripciones activas** | [Obtiene las suscripciones no expiradas](/graph/api/subscription-list). | 
| **Actualizar suscripción** | [Actualiza una suscripción](/graph/api/subscription-update) cuando se proporciona el identificador de suscripción. Por ejemplo, para ampliar su suscripción, puede actualizar la propiedad `expirationDateTime` de la suscripción. | 
| **Eliminar suscripción** | [Elimina una suscripción](/graph/api/subscription-delete) cuando se proporciona el identificador de la suscripción. | 
||| 

### <a name="manage-threat-intelligence-indicators"></a>Administración de indicadores de inteligencia sobre amenazas

Para filtrar, ordenar, u obtener los resultados más recientes, *solo* proporcione los [parámetros de consulta de ODATA admitidos por Microsoft Graph](/graph/query-parameters). *No especifique* la dirección URL base completa o la acción HTTP, por ejemplo, `https://graph.microsoft.com/beta/security/tiIndicators`, ni la operación `GET` o `PATCH`. Este es un ejemplo específico que muestra los parámetros de una acción **Obtener indicadores de inteligencia sobre amenazas** cuando se quiere una lista que tenga el tipo de amenaza `DDoS`:

`Filter threat intelligence indicator value as threatType eq 'DDoS'`

Para obtener más información sobre las consultas que puede usar con este conector, vea ["Parámetros de consulta opcionales" en la documentación de referencia del indicador de inteligencia sobre amenazas de seguridad de Microsoft Graph](/graph/api/tiindicators-list?tabs=http&view=graph-rest-beta). Para crear experiencias mejoradas con este conector, obtenga más información sobre el [indicador de inteligencia sobre amenazas de propiedades de esquema](/graph/api/resources/tiindicator?view=graph-rest-beta) que admite el conector.

| Acción | Descripción |
|--------|-------------|
| **Obtener indicadores de inteligencia sobre amenazas** | Obtenga indicadores de inteligencia sobre amenazas filtrados por una o varias [propiedades de indicadores de inteligencia sobre amenazas](/graph/api/resources/tiindicator?view=graph-rest-beta), por ejemplo, `threatType eq 'MaliciousUrl' or 'DDoS'`. |
| **Obtener indicadores de inteligencia sobre amenazas por identificador** | Obtenga un indicador de inteligencia sobre amenazas específico en función del identificador. | 
| **Crear un indicador de inteligencia sobre amenazas** | Cree un nuevo indicador de inteligencia sobre amenazas mediante su publicación en la colección. Para asegurarse de pasar las propiedades necesarias en la solicitud, vea las [propiedades necesarias para crear un indicador de inteligencia sobre amenazas](/graph/api/tiindicators-post?tabs=http&view=graph-rest-beta). |
| **Enviar varios indicadores de inteligencia sobre amenazas** | Cree varios indicadores de inteligencia sobre amenazas mediante la publicación de una colección. Para asegurarse de pasar las propiedades necesarias en la solicitud, vea las [propiedades necesarias para enviar varios indicadores de inteligencia sobre amenazas](/graph/api/tiindicator-submittiindicators?tabs=http&view=graph-rest-beta). |
| **Actualizar un indicador de inteligencia sobre amenazas** | Actualice un indicador de inteligencia sobre amenazas específico en función del identificador. Para asegurarse de pasar las propiedades necesarias y editables en la solicitud, vea las [propiedades editables para indicadores de inteligencia sobre amenazas](/graph/api/tiindicator-update?tabs=http&view=graph-rest-beta). Por ejemplo, para actualizar la acción que se va a aplicar si el indicador se combina desde la herramienta de seguridad targetProduct, puede actualizar la propiedad **action** del indicador de inteligencia sobre amenazas. |
| **Actualizar varios indicadores de inteligencia sobre amenazas** | Actualice varios indicadores de inteligencia sobre amenazas. Para asegurarse de pasar las propiedades necesarias en la solicitud, vea las [propiedades necesarias para actualizar varios indicadores de inteligencia sobre amenazas](/graph/api/tiindicator-updatetiindicators?tabs=http&view=graph-rest-beta). |
| **Eliminar indicadores de inteligencia sobre amenazas por identificador** | Elimine un indicador de inteligencia sobre amenazas específico en función del identificador. |
| **Eliminar varios indicadores de inteligencia sobre amenazas por identificador** | Elimine varios indicadores de inteligencia sobre amenazas por sus identificadores. Para asegurarse de pasar las propiedades necesarias en la solicitud, vea las [propiedades necesarias para eliminar varios indicadores de inteligencia sobre amenazas por identificador](/graph/api/tiindicator-deletetiindicators?tabs=http&view=graph-rest-beta). |
| **Eliminar varios indicadores de inteligencia sobre amenazas por identificador externo** | Elimine varios indicadores de inteligencia sobre amenazas por los identificadores externos. Para asegurarse de pasar las propiedades necesarias en la solicitud, vea las [propiedades necesarias para eliminar varios indicadores de inteligencia sobre amenazas por identificador externo](/graph/api/tiindicator-deletetiindicatorsbyexternalid?tabs=http&view=graph-rest-beta). |
|||

## <a name="connector-reference"></a>Referencia de conectores

Para obtener detalles técnicos acerca de desencadenadores, acciones y límites, que se describen en la descripción de OpenAPI (antes Swagger) del conector, consulte la [página de referencia](https://aka.ms/graphsecurityconnectorreference) del conector.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

