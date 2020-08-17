---
title: Escenarios y ejemplos comunes
description: Consulte ejemplos, escenarios comunes, tutoriales y guías detalladas de Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 02/28/2020
ms.openlocfilehash: 207b597bc865c8234d447759ab8b0f53dc35413c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090287"
---
# <a name="common-scenarios-examples-tutorials-and-walkthroughs-for-azure-logic-apps"></a>Escenarios comunes, ejemplos, tutoriales y guías detalladas de Azure Logic Apps

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) le ayuda a organizar e integrar servicios diferentes, ya que proporciona [cientos de conectores que se pueden usar en cualquier momento](../connectors/apis-list.md), que van de SQL Server o SAP a Azure Cognitive Services. El servicio Logic Apps es "sin servidor", por lo que no tiene que preocuparse de escalados o instancias. Lo único que debe hacer es definir el flujo de trabajo con un desencadenador y las acciones que realiza el flujo de trabajo. La plataforma subyacente controla la escala, la disponibilidad y el rendimiento. Logic Apps es especialmente útil para aquellos casos de uso y escenarios en los que hay que coordinar acciones en varios sistemas y servicios.

Para ayudarlo a obtener información sobre las funcionalidades y patrones que Azure Logic Apps admite, en este artículo se describen los escenarios, ejemplos y puntos iniciales más comunes.

## <a name="common-starting-points-for-logic-app-workflows"></a>Puntos iniciales comunes para flujos de trabajo de aplicaciones lógicas

Todas las aplicaciones lógica comienzan con un [*desencadenador*](../logic-apps/logic-apps-overview.md#logic-app-concepts), y solo un desencadenador, que inicia el flujo de trabajo de la aplicación lógica y pasa los datos como parte de dicho desencadenador. Algunos de los conectores proporcionan desencadenadores, que pueden ser de estos tipos:

* *Desencadenadores de sondeo*: comprueban de forma regular si un punto de conexión de servicio tiene datos nuevos. Cuando los haya, el desencadenador crea y ejecuta una nueva instancia de flujo de trabajo con dichos datos como entrada.

* *Desencadenadores de inserción*: escucha datos en un punto de conexión de servicio y espera hasta que se produce un evento específico. En ese momento, el desencadenador se activa inmediatamente y crear y ejecutar una nueva instancia de flujo de trabajo que utiliza como entrada todos los datos disponibles.

Estos son algunos ejemplos que describen los desencadenadores que se usan con frecuencia:

* Desencadenadores de *sondeo*:

  * El [desencadenador **Periodicidad**](../connectors/connectors-native-recurrence.md) permite establecer no solo la fecha y hora de inicio sino también la periodicidad con que se activa la aplicación lógica. Por ejemplo, puede seleccionar los días de la semana y las horas del día en que se activará la aplicación lógica. Para más información, consulte los temas siguientes:<p>

    * [Programación y ejecución de tareas, procesos y flujos de trabajo automatizados y periódicos con Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md)
    * [Tutorial: Creación de un flujo de trabajo automatizado, periódico y basado en una programación mediante Azure Logic Apps](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)

  * El desencadenador **Cuando llega un correo electrónico nuevo** permite que la aplicación lógica compruebe si hay mensajes de correo electrónico nuevos en todos los proveedores de correo electrónico compatibles con Logic Apps, por ejemplo, [Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md), [Gmail](/connectors/gmail/), [Outlook.com](/connectors/outlook/), etc.

    > [!IMPORTANT]
    > Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usarlo sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

    Para más información, consulte los temas siguientes:<p>

    * [Tutorial: Creación de flujos de trabajo automatizados basados en aprobación mediante Azure Logic Apps](../logic-apps/tutorial-process-mailing-list-subscriptions-workflow.md)
    * [Tutorial: Automatización de tareas para procesar correos electrónicos mediante Azure Logic Apps, Azure Functions y Azure Storage](../logic-apps/tutorial-process-email-attachments-workflow.md)

  * El [desencadenador **HTTP**](../connectors/connectors-native-http.md) puede llamar a un punto de conexión de servicio a través de HTTP o HTTPS. Para más información, consulte [Llamada, desencadenamiento o anidamiento de flujos de trabajo con puntos de conexión HTTP en Azure Logic Apps](../logic-apps/logic-apps-http-endpoint.md).

* Desencadenadores de *inserción*:

  * el [desencandenador **Request**](../connectors/connectors-native-reqres.md) puede recibir solicitudes HTTPS entrantes.

  * El [desencadenador **Webhook de HTTP**](../connectors/connectors-native-webhook.md) realiza la suscripción a un punto de conexión de servicio mediante el registro de una *dirección URL de devolución de llamada* en dicho servicio. De este modo, el servicio puede simplemente notificar el desencadenador cuando se produce el evento especificado, con el fin de que no sea preciso que este sondee el servicio.

Una vez que se produce el evento especificado, el desencadenador se activa, lo que crea una instancia nueva de flujo de trabajo de aplicación lógica y ejecuta las acciones en el flujo de trabajo. Para acceder a los datos del desencadenador se usa el flujo de trabajo. Por ejemplo, el desencadenador **Cuando se publica un tweet nuevo** de Twitter pasa el contenido del tweet a la aplicación lógica que se ejecuta. Para empezar a trabajar con Azure Logic Apps, pruebe estos temas de inicio rápido:

* [Inicio rápido: Creación del primer flujo de trabajo automatizado con Azure Logic Apps: Azure Portal](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [Inicio rápido: Creación y automatización de tareas, procesos y flujos de trabajo con Azure Logic Apps: Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)
* [Inicio rápido: Creación y administración de flujos de trabajo automatizados de aplicaciones lógicas mediante Visual Studio Code](../logic-apps/quickstart-create-logic-apps-visual-studio-code.md)

## <a name="control-flow-and-error-handling-capabilities"></a>Funcionalidades de flujo de control y control de errores

Las aplicaciones lógicas incluyen amplias funciones de flujo de control avanzado, como condiciones, modificadores, bucles y ámbitos. Para garantizar que las soluciones sean resistentes, también puede implementar el control de errores y excepciones en los flujos de trabajo.

* Realización de distintas acciones según las [instrucciones condicionales](../logic-apps/logic-apps-control-flow-conditional-statement.md) y las [instrucciones switch](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Repeat steps or process items in arrays and collections with loops](../logic-apps/logic-apps-control-flow-loops.md) (Repetición de pasos o procesamiento de elementos en matrices y colecciones con bucles)
* [Group actions together with scopes](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) (Agrupación de acciones con ámbitos)
* [Incorporación de control de errores y excepciones en un flujo de trabajo](../logic-apps/logic-apps-exception-handling.md)
* [Caso de uso: Cómo una empresa de atención sanitaria usa el control de excepciones de aplicaciones lógicas para los flujos de trabajo de HL7 FHIR](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)

## <a name="create-custom-apis-and-connectors"></a>Creación de conectores y API personalizadas

En el caso de los sistemas y servicios que no tienen conectores publicados, también puede extender las aplicaciones lógicas.

* [Creación de API personalizadas que se pueden llamar desde Azure Logic Apps](../logic-apps/logic-apps-create-api-app.md)
* [Comprobación periódica de nuevos datos o eventos con el patrón de desencadenador de sondeo](../logic-apps/logic-apps-create-api-app.md#polling-triggers)
* [Espera y escucha de nuevos datos o eventos con el patrón de desencadenador de webhook](../logic-apps/logic-apps-create-api-app.md#webhook-triggers)
* [Realización de tareas prolongadas con el patrón de acción de sondeo](../logic-apps/logic-apps-create-api-app.md#async-pattern)
* [Realización de tareas prolongadas con el patrón de acción de webhook](../logic-apps/logic-apps-create-api-app.md#webhook-actions)
* [Conectores personalizados en Azure Logic Apps](../logic-apps/custom-connector-overview.md)

## <a name="build-business-to-business-b2b-solutions"></a>Compilación de soluciones negocio a negocio (B2B)

En el caso de soluciones de integración empresarial y comunicación directa entre organizaciones, puede crear flujos de trabajo escalables automatizados para estos escenarios mediante el uso de Enterprise Integration Pack (EIP) con Azure Logic Apps. Aunque las organizaciones usan distintos protocolos y formatos, pueden intercambiar mensajes electrónicamente. EIP transforma distintos formato en un formato que los sistemas de las organizaciones pueden procesar y admiten protocolos estándar del sector, como AS2, X12, EDIFACT y RosettaNet. Para compilar estas soluciones, cree una cuenta de integración, que es un recurso de Azure independiente que proporciona un contenedor seguro,escalable y administrable para los artefactos que define y usa con los flujos de trabajo de la aplicación lógica. Por ejemplo, los artefactos incluyen entidades, acuerdos, mapas, esquemas, certificados y configuraciones de lote.

* [Información general: soluciones de integración empresarial B2B con Azure Logic Apps y Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
* [Creación y administración de cuentas de integración para la integración empresarial B2B en Azure Logic Apps](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)

## <a name="access-azure-virtual-network-resources"></a>Acceso a los recursos de una red virtual de Azure

A veces, sus cuentas de integración y aplicaciones lógicas necesitan tener acceso a recursos protegidos, como máquinas virtuales y otros sistemas o servicios que están en una red virtual de Azure. Para configurar este acceso, puede crear un entorno de servicio de integración (ISE) en el que puede compilar y ejecutar las aplicaciones lógicas. Un ISE es una instancia privada y aislada del servicio Logic Apps que usa recursos dedicados, como el almacenamiento, y se ejecuta de forma independiente del servicio Logic Apps público, "global" y de varios inquilinos. El hecho de separar la instancia privada aislada y la instancia global pública también ayuda a reducir el impacto que podrían tener otros inquilinos de Azure en el rendimiento de la aplicación, también conocido como el efecto "vecinos ruidosos".

* [Información general: Acceso a recursos de Azure Virtual Network desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)
* [Conexión a redes virtuales de Azure desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

## <a name="deploy-manage-and-monitor-logic-apps"></a>Implementación, administración y supervisión de aplicaciones lógicas

Puede desarrollar e implementar aplicaciones lógicas completamente con Visual Studio, Azure DevOps o cualquier otra herramienta de compilación automatizada o de control de código fuente. Con el fin de admitir la implementación para flujos de trabajos y conexiones dependientes en una plantilla de recursos, las aplicaciones lógicas usan las plantillas de implementación de recursos de Azure. Visual Studio Tools genera automáticamente estas plantillas, lo que puede comprobar en el control de código fuente para el control de versiones. Para los registros de notificación y diagnóstico del estado de ejecución del flujo de trabajo, Azure Logic Apps también ofrece supervisión y alertas.

### <a name="deploy"></a>Implementación

* [Inicio rápido: Creación y automatización de tareas, procesos y flujos de trabajo con Azure Logic Apps: Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)
* [Información general: Automatización de la implementación de aplicaciones lógicas](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)
* [Creación de plantillas de Azure Resource Manager para automatizar la implementación de Azure Logic Apps](../logic-apps/logic-apps-create-azure-resource-manager-templates.md)
* [Implementación de plantillas de Azure Resource Manager para Azure Logic Apps](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md)
* [Ejemplo: Conexión a colas de Azure Service Bus desde Azure Logic Apps e implementación con Azure Pipelines en Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-service-bus-queues-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Ejemplo: Conexión a cuentas de Azure Storage desde Azure Logic Apps e implementación con Azure Pipelines en Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-storage-accounts-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Ejemplo: Conexión a una función de aplicación de funciones para Azure Logic Apps e implementación con Azure Pipelines en Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/set-up-an-azure-function-app-action-for-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Ejemplo: Conexión a una cuenta de integración desde Azure Logic Apps e implementación con Azure Pipelines en Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-an-integration-account-from-azure-logic-apps-and-deploy-by-using-azure-devops-pipelines/)
* [Ejemplo: Organización de Azure Pipelines mediante Azure Logic Apps](/samples/azure-samples/azure-logic-apps-pipeline-orchestration/azure-devops-orchestration-with-logic-apps/)

### <a name="manage"></a>Administrar

* [Administración de aplicaciones lógicas mediante Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md)
* [Creación y administración de cuentas de integración para la integración empresarial B2B](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)
* [Administración del entorno del servicio de integración (ISE) en Azure Logic Apps](../logic-apps/ise-manage-integration-service-environment.md)

### <a name="monitor"></a>Supervisión

* [Supervisión del estado de ejecución, revisión del historial de los desencadenadores y configuración de alertas en Azure Logic Apps](../logic-apps/monitor-logic-apps.md)
* [Configuración de registros de Azure Monitor y recopilación de datos de diagnóstico para Azure Logic Apps](../logic-apps/monitor-logic-apps-log-analytics.md)
* [Configuración de registros de Azure Monitor y recopilación de datos de diagnóstico para mensajes B2B en Azure Logic Apps](../logic-apps/monitor-b2b-messages-log-analytics.md)
* [Visualización y creación de consultas para supervisar y realizar un seguimiento de los registros de Azure Monitor para Azure Logic Apps](../logic-apps/create-monitoring-tracking-queries.md)

## <a name="handle-content-types-conversions-and-transformations"></a>Control de tipos de contenido, conversiones y transformaciones

Puede obtener acceso a distintos tipos de contenido, convertirlos y transformarlos mediante las diversas funciones que existen en el [lenguaje de definición de flujo de trabajo de](https://aka.ms/logicappsdocs) Azure Logic Apps. Por ejemplo, puede convertir entre una cadena, JSON y XML con las expresiones de flujo de trabajo `@json()` y `@xml()`. El motor de Logic Apps conserva los tipos de contenido para admitir la transferencia de contenido sin pérdida de información entre los servicios.

* [Control de tipos de contenido en Azure Logic Apps](../logic-apps/logic-apps-content-type.md), como `application/`, `application/octet-stream` y `multipart/formdata`
* [Guía de referencia para usar las funciones en las expresiones para Azure Logic Apps y Power Automate](../logic-apps/workflow-definition-language-functions-reference.md)
* [Esquema del lenguaje de definición de flujo de trabajo - Azure Logic Apps](../logic-apps/logic-apps-workflow-definition-language.md)

## <a name="other-integrations-and-capabilities"></a>Otras integraciones y funcionalidades

Azure Logic Apps también ofrece integración con muchos servicios, como Azure Functions, Azure API Management, Azure App Service y puntos de conexión HTTP personalizados, por ejemplo, REST y SOAP.

* [Llamada a Azure Functions desde Azure Logic Apps](../logic-apps/logic-apps-azure-functions.md)
* [Tutorial: Llamada o desencadenamiento de aplicaciones lógicas con Azure Functions y Azure Service Bus](../logic-apps/logic-apps-scenario-function-sb-trigger.md)
* [Tutorial: Creación de un panel de streaming de Customer Insights con Azure Logic Apps y Azure Functions](../logic-apps/logic-apps-scenario-social-serverless.md)
* [Tutorial: Creación de una función que se integra con Azure Logic Apps y Azure Cognitive Services para analizar las intenciones de las publicaciones en Twitter](../azure-functions/functions-twitter-email.md)
* [Tutorial: Creación de un panel social con tecnología de inteligencia artificial mediante Power BI y Azure Logic Apps](https://aka.ms/logicappsdemo)
* [Tutorial: Supervisión de los cambios en las máquinas virtuales con Azure Event Grid y Logic Apps](../event-grid/monitor-virtual-machine-changes-event-grid-logic-app.md)
* [Tutorial: IoT remote monitoring and notifications with Azure Logic Apps connecting your IoT hub and mailbox](../iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps.md) (Tutorial: Supervisión remota y notificaciones de IoT con Azure Logic Apps conectando IoT Hub y el buzón de correo)
* [Blog: Llamada a servicios SOAP mediante Azure Logic Apps](/archive/blogs/logicapps/using-soap-services-with-logic-apps)

## <a name="end-to-end-scenarios"></a>Escenarios de un extremo a otro

* [Whitepaper: End-to-end case management integration with Azure services, such as Logic Apps](https://aka.ms/enterprise-integration-e2e-case-management-utilities-logic-apps) (Notas del producto: Integración de la administración de un caso completo con servicios de Azure, como Logic Apps)

## <a name="customer-stories"></a>Testimonios de clientes

Sepa cómo Azure Logic Apps, junto con otros servicios de Azure y productos de Microsoft, ayudó a que [estas empresas](https://aka.ms/logic-apps-customer-stories) mejoraran su agilidad y se centraran en sus negocios principales mediante la simplificación, organización, automatización y orquestación de procesos complejos.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [conectores para Logic Apps](../connectors/apis-list.md)
* Más información sobre [escenarios de integración empresarial B2B en Azure Logic Apps con Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
