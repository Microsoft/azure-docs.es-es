---
title: Conectores para Azure Logic Apps
description: Automatice los flujos de trabajo con conectores para Azure Logic Apps, como los conectores integrados, administrados, locales, de cuenta de integración, ISE y empresariales
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: article
ms.date: 06/11/2020
ms.openlocfilehash: 1bd3cb1c18d1bac078ac1344f574914dba73d07b
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871571"
---
# <a name="connectors-for-azure-logic-apps"></a>Conectores para Azure Logic Apps

Los conectores proporcionan acceso rápido desde Azure Logic Apps a eventos, datos y acciones entre otras aplicaciones, servicios, sistemas, protocolos y plataformas. El uso de conectores en las aplicaciones lógicas permite ampliar las funcionalidades de las aplicaciones locales y en la nube para realizar tareas con los datos que ya tiene y con los que cree.

Aunque Logic Apps ofrece [cientos de conectores](/connectors), en este artículo se describen los conectores *populares y más utilizados* que miles de aplicaciones y millones de ejecuciones usan correctamente para procesar datos e información. Para buscar la lista completa de conectores y la información de referencia de cada conector, como desencadenadores, acciones y límites, revise las páginas de referencia de conectores de [Introducción a los conectores](/connectors). Además, obtenga más información sobre [desencadenadores y acciones](#triggers-actions), el [modelo de precios de Logic Apps](../logic-apps/logic-apps-pricing.md) y los [detalles de precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

> [!TIP]
> Para integrarse con un servicio o API que no tiene conector, puede llamar directamente al servicio a través de un protocolo como HTTP, o bien crear un [conector personalizado](#custom).

## <a name="connector-types"></a>Tipos de conectores

Los conectores están disponibles como desencadenadores y acciones integrados, o como conectores administrados.

<a name="built-in"></a>

* [**Integrados**](#built-ins): las acciones y desencadenadores integrados son "nativos" en Azure Logic Apps y ayudan a realizar estas tareas para las aplicaciones lógicas:

  * Ejecutarse durante programaciones personalizadas y avanzadas.

  * Organizar y controlar el flujo de trabajo de la aplicación lógica (por ejemplo, bucles y condiciones), así como trabajar con variables y operaciones de datos.

  * Comunicarse con otros puntos de conexión.

  * Recibir y responder solicitudes.

  * Llamar a funciones de Azure, API Apps de Azure (aplicaciones web), sus API administradas o publicadas con Azure API Management y aplicaciones lógicas anidadas que pueden recibir solicitudes.

<a name="managed-connectors"></a>

* [**Conectores administrados**](#managed-api-connectors): Implementados y administrados por Microsoft, estos conectores proporcionan desencadenadores y acciones para acceder a servicios en la nube, sistemas locales o ambos, incluidos Office 365, Azure Blob Storage, SQL Server, Dynamics, Salesforce, SharePoint y muchos más. Algunos conectores admiten específicamente escenarios de comunicación de negocio a negocio (B2B) y requieren una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) vinculada a la aplicación lógica. Antes de usar algunos conectores, es posible que tenga que crear conexiones, que se administran mediante Azure Logic Apps.

  Por ejemplo, si usa Microsoft BizTalk Server, las aplicaciones lógicas podrán conectarse y comunicarse con BizTalk Server mediante el [conector local de BizTalk Server](#on-premises-connectors). Después, puede ampliar o realizar operaciones de BizTalk en las aplicaciones lógicas mediante los [conectores de la cuenta de integración](#integration-account-connectors).

  Los conectores se clasifican como estándar o empresariales. Los [conectores empresariales](#enterprise-connectors) proporcionan acceso a sistemas empresariales como SAP, IBM MQ e IBM 3270 por un costo adicional. Para determinar si un conector es Estándar o Empresarial, vea los detalles técnicos en la página de referencia de cada conector en [Introducción a los conectores](/connectors).

  También puede identificar los conectores mediante estas categorías, aunque algunos pueden existir en varias. Por ejemplo, SAP es un conector empresarial y un conector local:

  | Category | Descripción |
  |----------|-------------|
  | [**Conectores administrados**](#managed-api-connectors) | Cree aplicaciones lógicas que usan servicios tales como Azure Blob Storage, Office 365, Dynamics, Power BI, OneDrive, Salesforce, SharePoint Online y muchos más. |
  | [**Conectores locales**](#on-premises-connectors) | Después de instalar y configurar la [puerta de enlace de datos local][gateway-doc], estos conectores ayudan a las aplicaciones lógicas a acceder a los sistemas locales, como SQL Server, SharePoint Server, Oracle DB, recursos compartidos de archivos y otros. |
  | [**Conectores de la cuenta de integración**](#integration-account-connectors) | Disponibles cuando se crea y se paga una cuenta de integración, estos conectores transforman y validan el código XML, codifican y descodifican archivos sin formato y procesan mensajes de negocio a negocio (B2B) con protocolos AS2, EDIFACT y X12. |
  |||

<a name="integration-service-environment"></a>

### <a name="connect-from-an-integration-service-environment-ise"></a>Conexión desde un entorno del servicio de integración (ISE)

En el caso de las aplicaciones lógicas que necesitan acceso directo a los recursos de una red virtual de Azure, puede crear un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) dedicado en el que puede compilar, implementar y ejecutar aplicaciones lógicas en recursos dedicados. En el Diseñador de aplicación lógica, al examinar los conectores que quiere usar para aplicaciones lógicas en un ISE, aparece una etiqueta **CORE** en desencadenadores y acciones integrados, mientras que la etiqueta **ISE** aparece en algunos conectores.

> [!NOTE]
> Las aplicaciones lógicas que se ejecutan en un ISE, así como sus conectores, independientemente de la ubicación en que se ejecuten dichos conectores, usan un plan de precios fijo frente a un plan de precios basado en el consumo. Para más información, vea [Modelo de precios de Logic Apps](../logic-apps/logic-apps-pricing.md) y [Detalles de precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

| Etiqueta | Ejemplo | Descripción |
|-------|---------|-------------|
| **CORE** | ![Conector de ejemplo CORE](./media/apis-list/example-core-connector.png) | Los desencadenadores y acciones integrados con esta etiqueta se ejecutan en el mismo ISE que las aplicaciones lógicas. |
| **ISE** | ![Ejemplo de conector de ISE](./media/apis-list/example-ise-connector.png) | Los conectores administrados con esta etiqueta se ejecutan en el mismo ISE que las aplicaciones lógicas. Si tiene un sistema local que está conectado a una red virtual de Azure, un ISE permite a las aplicaciones lógicas obtener acceso directo a dicho sistema sin la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-connection.md). En su lugar, puede usar el conector de **ISE** si está disponible, una acción HTTP o un [conector personalizado](#custom). En el caso de los sistemas locales que no tienen conectores de **ISE**, use la puerta de enlace de datos local. Para revisar los conectores de ISE disponibles, consulte[conectores de ISE](#ise-connectors). |
| Sin etiqueta | ![Ejemplo de conector multiinquilino](./media/apis-list/example-multi-tenant-connector.png) | Todos los demás conectores que no tienen la etiqueta **CORE** o **ISE**, que puede seguir usando, se ejecutan en el servicio de Logic Apps global multiinquilino. |
|||

<a name="built-ins"></a>

## <a name="built-in"></a>Integrada

Logic Apps proporciona desencadenadores y acciones integrados para que pueda crear flujos de trabajo basados en programación, ayudar a las aplicaciones lógicas a comunicarse con otras aplicaciones y servicios, controlar el flujo de trabajo a través de las aplicaciones lógicas y administrar o manipular los datos.

| Nombre | Descripción |
|------|-------------|
| [![Conector integrado de programación][schedule-icon]<br>**Programación**][schedule-doc] | - Ejecute una aplicación lógica con una periodicidad específica, usando programaciones básicas y complejas, con el desencadenador [**Periodicidad**][schedule-recurrence-doc]. <br>- Ejecute una aplicación lógica que necesite controlar los datos en fragmentos continuos con el desencadenador [**Ventana deslizante**][schedule-sliding-window-doc]. <br>- Pause la aplicación lógica durante un tiempo especificado con la acción [**Retraso**][schedule-delay-doc]. <br>- Pause la aplicación lógica hasta la fecha y hora especificadas con la acción [**Retraso hasta**][schedule-delay-until-doc]. |
| [![Conector integrado por lotes][batch-icon]<br>**Lote**][batch-doc] | - Procese mensajes por lotes con el desencadenador **Mensajes del lote**. <br>- Llame a aplicaciones lógicas que tengan desencadenadores por lotes con la acción **Enviar mensajes al lote**. |
| [![Conector integrado de HTTP][http-icon]<br>**HTTP**][http-doc] | Llame a puntos de conexión HTTP o HTTPS con los desencadenadores y acciones para HTTP. Otras acciones y desencadenadores integrados HTTP incluyen [Conector integrado HTTP + Swagger][http-swagger-doc] y [HTTP + Webhook][http-webhook-doc]. |
| [![Conector integrado de solicitud][http-request-icon]<br>**Solicitud**][http-request-doc] | - Haga que su aplicación lógica se pueda llamar desde otras aplicaciones o servicios, desencadene ante eventos de recursos de Event Grid o desencadene ante respuestas a las alertas de Azure Security Center con el desencadenador **Solicitud**. <br>- Envíe respuestas a una aplicación o servicio con la acción **Respuesta**. |
| [![Conector integrado de Azure API Management][azure-api-management-icon]<br>**Azure API <br>Management**][azure-api-management-doc] | Llame a los desencadenadores y las acciones definidos por las API que administre y publique con Azure API Management. |
| [![Conector integrado deAzure App Services][azure-app-services-icon]<br>**Azure App <br>Services**][azure-app-services-doc] | Llame a las aplicaciones de API o las aplicaciones web de Azure hospedadas en Azure App Service. Los desencadenadores y las acciones definidos por estas aplicaciones aparecen como cualquier otro desencadenador y acción de primera clase cuando se incluye Swagger. |
| [![Conector integrado de Azure Logic Apps][azure-logic-apps-icon]<br>**Azure Logic <br>Apps**][nested-logic-app-doc] | Llame a otras aplicaciones lógicas que se inicien con el desencadenador **Solicitud**. |
|||

### <a name="run-code-from-logic-apps"></a>Ejecución de código de aplicaciones lógicas

Logic Apps proporciona acciones integradas para ejecutar código propio en el flujo de trabajo de la aplicación lógica:

| Nombre | Descripción |
|------|-------------|
| [![Conector integrado de Azure Functions][azure-functions-icon]<br>**Azure Functions**][azure-functions-doc] | Llame a funciones de Azure que ejecuten fragmentos de código personalizados (C# o Node.js) desde las aplicaciones lógicas. |
| [![Conector integrado de código en línea][inline-code-icon]<br>**Código en línea**][inline-code-doc] | Agregue y ejecute fragmentos de código JavaScript de las aplicaciones lógicas. |
|||

### <a name="control-workflow"></a>Controlar el flujo de trabajo

Logic Apps proporciona acciones integradas para estructurar y controlar las acciones en el flujo de trabajo de la aplicación lógica:

| Nombre | Descripción |
|------|-------------|
| [![Acción integrada de condición][condition-icon]<br>**Condición**][condition-doc] | Evalúa una condición y ejecuta acciones diferentes según si la condición es true o false. |
| [![Acción integrada For each][for-each-icon]<br>**For each**][for-each-doc] | Realiza las mismas acciones en todos los elementos de una matriz. |
| [![Acción integrada de ámbito][scope-icon]<br>**Ámbito**][scope-doc] | Agrupa las acciones en *ámbitos*, que obtienen su propio estado después de que las acciones del ámbito terminen de ejecutarse. |
| [![Acción integrada de modificador][switch-icon]<br>**Modificador**][switch-doc] | Agrupa las acciones en *casos* a los que se asignan valores únicos, excepto el caso predeterminado. Ejecuta solo el caso cuyo valor asignado coincida con el resultado de una expresión, objeto o token. Si no hay coincidencias, ejecuta el caso predeterminado. |
| [![Acción integrada de finalización][terminate-icon]<br>**Finalización**][terminate-doc] | Detiene el flujo de trabajo que la aplicación lógica está ejecutando activamente. |
| [![Acción integrada hasta][until-icon]<br>**Hasta**][until-doc] | Repite las acciones hasta que la condición especificada sea true o cambie algún estado. |
|||

### <a name="manage-or-manipulate-data"></a>Administrar o manipular datos

Logic Apps proporciona acciones integradas para trabajar con salidas de datos y sus formatos:

| Nombre | Descripción |
|------|-------------|
| [![Acción integrada operaciones de datos][data-operations-icon]<br>**Operaciones de datos**][data-operations-doc] | Realiza operaciones con datos: <p>- **Redactar**: crea una única salida a partir de varias entradas con varios tipos. <br>- **Crear tabla CSV**: crea una tabla con valores separados por comas (CSV) a partir de una matriz con objetos JSON. <br>- **Crear tabla HTML**: crea una tabla HTML a partir de una matriz con objetos JSON. <br>- **Filtrar matriz**: crea una matriz a partir de elementos de otra matriz que cumplen los criterios. <br>- **Unirse**: crea una cadena a partir de todos los elementos de una matriz y los separa con el delimitador especificado. <br>- **Analizar JSON**: crea tokens sencillos a partir de propiedades y sus valores en contenido JSON para que pueda usar esas propiedades en el flujo de trabajo. <br>- **Seleccionar**: crea una matriz con objetos JSON mediante la transformación de elementos o valores de otra matriz, y la asignación de esos elementos a propiedades especificadas. |
| ![Acción integrada de fecha y hora][date-time-icon]<br>**Fecha y hora** | Realiza operaciones con marcas de tiempo: <p>- **Agregar a la hora**: agrega el número especificado de unidades a una marca de tiempo. <br>- **Convertir la zona horaria**: Convierte una marca de tiempo de la zona horaria de origen a la zona horaria de destino. <br>- **Hora actual**: Devuelve la marca de tiempo actual como una cadena. <br>- **Obtener la hora futura**: Devuelve la marca de tiempo actual más las unidades de tiempo especificadas. <br>- **Obtener la hora pasada**: Devuelve la marca de tiempo actual menos las unidades de tiempo especificadas. <br>- **Restar de la hora**: Resta un número de unidades de tiempo de una marca de tiempo. |
| [![Acción integrada de variables][variables-icon]<br>**Variables**][variables-doc] | Realiza operaciones con variables: <p>- **Anexar a la variable de matriz**: inserta un valor como el último elemento en una matriz almacenada por una variable. <br>- **Anexar a la variable de cadena**: inserta un valor como el último carácter en una cadena almacenada por una variable. <br>- **Reducir variable**: reduce una variable por un valor constante. <br>- **Incrementar variable**: incrementa una variable por un valor constante. <br>- **Inicializar variable**: crea una variable y declara su tipo de datos y el valor inicial. <br>- **Establecer variable**: asigna otro valor a una variable existente. |
|||

<a name="managed-api-connectors"></a>

## <a name="managed-connectors"></a>Conectores administrados

Logic Apps proporciona estos conocidos conectores estándar para automatizar tareas, procesos y flujos de trabajo con estos servicios o sistemas:

| Nombre | Descripción |
|------|-------------|
| [![Conector administrado de Azure Service Bus][azure-service-bus-icon]<br>**Azure Service Bus**][azure-service-bus-doc] | Administre mensajes asincrónicos, sesiones y suscripciones a temas con el conector más usado en Logic Apps. |
| [![Conector administrado de SQL Server][sql-server-icon]<br>**SQL Server**][sql-server-doc] | Conéctese a SQL Server en el entorno local o a una base de datos de Azure SQL Database en la nube para poder administrar los registros, ejecutar procedimientos almacenados o realizar consultas. |
| [![Conector administrado de Azure Blob Storage][azure-blob-storage-icon]<br>**Azure Blob<br>Storage**][azure-blob-storage-doc] | Conéctese a su cuenta de almacenamiento para crear y administrar el contenido en blobs. |
| [![Conector administrado de Office 365 Outlook][office-365-outlook-icon]<br>**Office 365<br>Outlook**][office-365-outlook-doc] | Conéctese a su cuenta de correo electrónico de Office 365 para crear y administrar correos electrónicos, tareas, eventos de calendario y reuniones, contactos, solicitudes y más. |
| [![Conector administrado SFTP-SSH][sftp-ssh-icon]<br>**SFTP-SSH**][sftp-ssh-doc] | Conéctese a servidores SFTP accesibles desde internet mediante SSH para trabajar con archivos y carpetas. |
| [![Conector administrado de SharePoint Online][sharepoint-online-icon]<br>**SharePoint<br>Online**][sharepoint-online-doc] | Conéctese a SharePoint Online para administrar archivos, datos adjuntos, carpetas y mucho más. |
| [![Conector administrado de colas de Azure][azure-queues-icon]<br>**Colas <br>de Azure**][azure-queues-doc] | Conéctese a su cuenta de Azure Storage para poder crear y administrar colas y mensajes. |
| [![Conector administrado de FTP][ftp-icon]<br>**FTP**][ftp-doc] | Conéctese a servidores FTP accesibles desde internet para trabajar con archivos y carpetas. |
| [![Conector administrado del sistema de archivos][file-system-icon]<br>**Sistema<br> de archivos**][file-system-doc] | Conéctese al recurso compartido de archivos local para crear y administrar archivos. |
| [![Conector administrado de Azure Event Hubs][azure-event-hubs-icon]<br>**Azure Event Hubs**][azure-event-hubs-doc] | Consuma y publique eventos en un centro de eventos. Por ejemplo, obtenga una salida de su aplicación lógica con Event Hubs y enviarla luego a un proveedor de análisis en tiempo real. |
| [![Conector administrado de Azure Event Grid][azure-event-grid-icon]<br>**Azure Event**<br>**Grid**][azure-event-grid-doc] | Supervise los eventos publicados por Event Grid, por ejemplo, cuando cambian los recursos de Azure o los recursos de terceros. |
| [![Conector administrado de Salesforce][salesforce-icon]<br>**Salesforce**][salesforce-doc] | Conéctese a su cuenta de Salesforce para crear y administrar elementos tales como registros, trabajos, objetos y mucho más. |
|||

<a name="on-premises-connectors"></a>

## <a name="on-premises-connectors"></a>Conectores locales

Estos son algunos de los conectores estándar más usados que proporciona Logic Apps para acceder a datos y recursos en sistemas locales. Para poder crear una conexión a un sistema local, primero debe [descargar, instalar y configurar una puerta de enlace de datos local][gateway-doc]. Esta puerta de enlace proporciona un canal de comunicación seguro sin tener que configurar la infraestructura de red necesaria.

:::row:::
    :::column:::
        [![Conector de BizTalk Server][biztalk-server-icon]<br>**BizTalk** <br>**Server**][biztalk-server-doc]
    :::column-end:::
    :::column:::
        [![Conector del sistema de archivos][file-system-icon]<br>**Sistema<br> de archivos**][file-system-doc]
    :::column-end:::
    :::column:::
        [![Conector DB2][ibm-db2-icon]<br>**IBM DB2**][ibm-db2-doc]
    :::column-end:::
    :::column:::
        [![Conector de Informix][ibm-informix-icon]<br>**IBM** <br>**Informix**][ibm-informix-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Conector de MySQL][mysql-icon]<br>**MySQL**][mysql-doc]
    :::column-end:::
    :::column:::
        [![Conector de Oracle DB][oracle-db-icon]<br>**Oracle DB**][oracle-db-doc]
    :::column-end:::
    :::column:::
        [![Conector de PostgreSQL][postgre-sql-icon]<br>**PostgreSQL**][postgre-sql-doc]
    :::column-end:::
    :::column:::
        [![Conector de SharePoint Server][sharepoint-server-icon]<br>**SharePoint <br>Server**][sharepoint-server-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Conector de SQL Server][sql-server-icon]<br>**SQL <br>Server**][sql-server-doc]
    :::column-end:::
    :::column:::
        [![Conector de Teradata][teradata-icon]<br>**Teradata**][teradata-doc]
    :::column-end:::
    :::column:::
        
    :::column-end:::
    :::column:::
        
    :::column-end:::
:::row-end:::

<a name="integration-account-connectors"></a>

## <a name="integration-account-connectors"></a>Conectores de la cuenta de integración

Logic Apps proporciona conectores estándar para crear soluciones de negocio a negocio (B2B) con las aplicaciones lógicas cuando se crea y se paga por una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md), que está disponible a través de Enterprise Integration Pack (EIP) en Azure. Con esta cuenta, puede crear y almacenar artefactos B2B, tales como asociados comerciales, acuerdos, mapas, esquemas, certificados, etc. Para usar estos artefactos, asocie las aplicaciones lógicas con la cuenta de integración. Si actualmente usa BizTalk Server, estos conectores le resultará familiares ya.

:::row:::
    :::column:::
        [![Acción de descodificación de AS2][as2-icon]<br>**Descodificación<br> de AS2**][as2-doc]
    :::column-end:::
    :::column:::
        [![Acción de codificación de AS2][as2-icon]<br>**Codificación<br> de AS2**][as2-doc]
    :::column-end:::
    :::column:::
        [![Acción de descodificación de EDIFACT][edifact-icon]<br>**Descodificación<br> de EDIFACT**][edifact-decode-doc]
    :::column-end:::
    :::column:::
        [![Acción de codificación de EDIFACT][edifact-icon]<br>**Codificación<br> de EDIFACT**][edifact-encode-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Acción de descodificación de archivos planos][flat-file-decode-icon]<br>**Descodificación<br> de archivos planos**][flat-file-decode-doc]
    :::column-end:::
    :::column:::
        [![Acción de codificación de archivos planos][flat-file-encode-icon]<br>**Codificación<br> de archivos planos**][flat-file-encode-doc]
    :::column-end:::
    :::column:::
        [![Acción de la cuenta de integración][integration-account-icon]<br>**Acción <br> de integración**][integration-account-doc]
    :::column-end:::
    :::column:::
        [![Acción de transformaciones líquidas][liquid-icon]<br>**Transformaciones** <br>**líquidas**][json-liquid-transform-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Acción de descodificación de X12][x12-icon]<br>**Descodificación<br> de X12**][x12-decode-doc]
    :::column-end:::
    :::column:::
        [![Acción de codificación de X12][x12-icon]<br>**Codificación<br> de X12**][x12-encode-doc]
    :::column-end:::
    :::column:::
        [![Acción de transformaciones XML][xml-transform-icon]<br>**Transformaciones** <br> **XML**][xml-transform-doc]
    :::column-end:::
    :::column:::
        [![Acción de validación de XML][xml-validate-icon]<br>**Validación <br>de XML**][xml-validate-doc]
    :::column-end:::
:::row-end:::

<a name="enterprise-connectors"></a>

## <a name="enterprise-connectors"></a>Conectores de empresa

Logic Apps proporciona estos conectores empresariales para acceder a sistemas empresariales, como SAP e IBM MQ:

:::row:::
    :::column:::
        [![Conector de IBM 3270][ibm-3270-icon]<br>**IBM 3270**][ibm-3270-doc]
    :::column-end:::
    :::column:::
        [![Conector de MQ][ibm-mq-icon]<br>**IBM MQ**][ibm-mq-doc]
    :::column-end:::
    :::column:::
        [![Conector de SAP][sap-icon]<br>**SAP**][sap-connector-doc]
    :::column-end:::
    :::column:::
        
    :::column-end:::
:::row-end:::

<a name="ise-connectors"></a>

## <a name="ise-connectors"></a>Conectores del ISE

En el caso de las aplicaciones lógicas que se crean y ejecutan en un [entorno del servicio de integración (ISE)](#integration-service-environment) dedicado, el diseñador de aplicación lógica identifica los desencadenadores y las acciones integrados que se ejecutan en el ISE mediante la etiqueta **CORE**. Los conectores administrados que se ejecutan en un ISE muestran la etiqueta **ISE**, mientras que los conectores que se ejecutan en el servicio de Logic Apps global multiinquilino no muestran ninguna etiqueta. Esta lista muestra los conectores que actualmente tienen versiones ISE:

:::row:::
    :::column:::
        [![Conector de AS2 ISE][as2-icon]<br>**AS2**][as2-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure Automation][azure-automation-icon]<br>**Azure <br>Automation**][azure-automation-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure Blob Storage][azure-blob-storage-icon]<br>**Azure Blob<br>Storage**][azure-blob-storage-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure Cosmos DB][azure-cosmos-db-icon]<br>**Azure Cosmos <br> DB**][azure-cosmos-db-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Conector de ISE de Azure Event Hubs][azure-event-hubs-icon]<br>**Azure Event <br>Hubs**][azure-event-hubs-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure Event Grid][azure-event-grid-icon]<br>**Azure Event <br>Grid**][azure-event-grid-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure File Storage][azure-file-storage-icon]<br>**Azure File<br>Storage**][azure-file-storage-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure Key Vault][azure-key-vault-icon]<br>**Azure Key <br>Vault**][azure-key-vault-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Conector de ISE de registros de Azure Monitor ][azure-monitor-logs-icon]<br>**Registros<br> de Azure Monitor**][azure-monitor-logs-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure Service Bus ISE][azure-service-bus-icon]<br>**Azure Service <br>Bus**][azure-service-bus-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure SQL Data Warehouse][azure-sql-data-warehouse-icon]<br>**Azure SQL Data <br>Warehouse**][azure-sql-data-warehouse-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de Azure Table Storage][azure-table-storage-icon]<br>**Azure Table <br>Storage**][azure-table-storage-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Conector ISE de colas de Azure][azure-queues-icon]<br>**Colas<br> de Azure**][azure-queues-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de EDIFACT][edifact-icon]<br>**EDIFACT**][edifact-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE del sistema de archivos][file-system-icon]<br>**Sistema<br> de archivos**][file-system-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de FTP][ftp-icon]<br>**FTP**][ftp-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Conector de ISE de IBM 3270][ibm-3270-icon]<br>**IBM 3270**][ibm-3270-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de DB2][ibm-db2-icon]<br>**IBM DB2**][ibm-db2-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de MQ][ibm-mq-icon]<br>**IBM MQ**][ibm-mq-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de SAP][sap-icon]<br>**SAP**][sap-connector-doc]
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Conector de ISE de SFTP-SSH][sftp-ssh-icon]<br>**SFTP-SSH**][sftp-ssh-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de SMTP][smtp-icon]<br>**SMTP**][smtp-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de SQL Server][sql-server-icon]<br>**SQL <br>Server**][sql-server-doc]
    :::column-end:::
    :::column:::
        [![Conector de ISE de X12][x12-icon]<br>**X12**][x12-doc]
    :::column-end:::
:::row-end:::

Para más información, consulte los temas siguientes:

* [Acceso a recursos de Azure Virtual Network desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)
* [Modelo de precios de Logic Apps](../logic-apps/logic-apps-pricing.md)
* [Conexión a redes virtuales de Azure desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

<a name="triggers-actions"></a>

## <a name="triggers-and-action-types"></a>Tipos de desencadenadores y acciones

Los conectores pueden proporcionar *desencadenadores*, *acciones* o ambos. Un *desencadenador* es el primer paso de cualquier aplicación lógica, y normalmente especifica el evento que activa el desencadenador e inicia la ejecución de la aplicación lógica. Por ejemplo, el conector FTP tiene un desencadenador que inicia la aplicación lógica "cuando se agrega o modifica un archivo". Algunos desencadenadores comprueban de forma periódica el evento o los datos especificados, y después se activan cuando detectan el evento o los datos especificados. Otros desencadenadores esperan pero se activan al instante cuando se produce un evento específico o cuando hay nuevos datos disponibles. Los desencadenadores también pasan los datos requeridos para la aplicación lógica. La aplicación lógica puede leer y usar esos datos a lo largo del flujo de trabajo. Por ejemplo, el conector de Office 365 Outlook tiene un desencadenador, "Cuando llega un nuevo correo electrónico", que puede pasar el contenido de ese correo electrónico al flujo de trabajo de la aplicación lógica.

Una vez que se activa un desencadenador, Azure Logic Apps crea una instancia de la aplicación lógica y comienza a ejecutar las *acciones* en el flujo de trabajo de la aplicación lógica. Las acciones son los pasos que sigue el desencadenador y realizan tareas en el flujo de trabajo de la aplicación lógica. Por ejemplo, puede crear una aplicación lógica que obtenga datos del cliente desde una base de datos SQL y los procese en acciones posteriores.

Estos son los tipos generales de desencadenadores que proporciona Azure Logic Apps:

* *Desencadenador de periodicidad*: este desencadenador se ejecuta según una programación especificada y no está estrechamente asociado a un sistema o servicio determinado.

* *Desencadenador de sondeo*: este desencadenador sondea de forma periódica un servicio o sistema específico según la programación especificada, para buscar datos nuevos o si se ha producido un evento específico. Si hay datos nuevos disponibles o se ha producido el evento específico, el desencadenador crea y ejecuta una nueva instancia de la aplicación lógica, que ahora puede usar los datos que se han pasado como entrada.

* *Desencadenador de push*: este desencadenador espera y escucha datos nuevos, o que se produzca un evento. Cuando hay datos nuevos disponibles o se produce el evento, el desencadenador crea y ejecuta una nueva instancia de la aplicación lógica, que ahora puede usar los datos que se han pasado como entrada.

<a name="connections"></a>

## <a name="connector-configuration"></a>Configuración del conector

Los desencadenadores y las acciones de cada conector proporcionan sus propias propiedades que se pueden configurar. Muchos conectores también requieren que primero se cree una *conexión* al servicio o sistema de destino, y que proporcione credenciales de autenticación u otros detalles de configuración antes de poder usar un desencadenador o una acción en la aplicación lógica. Por ejemplo, para poder acceder a su cuenta de correo electrónico de Office 365 Outlook y trabajar con ella, debe autorizar una conexión a esa cuenta.

En el caso de los conectores que usan Azure Active Directory (Azure AD) OAuth, crear una conexión significa iniciar sesión en el servicio (por ejemplo, Office 365, Salesforce o GitHub), donde el token de acceso [se cifra](../security/fundamentals/encryption-overview.md) y se almacena de forma segura en un almacén de secretos de Azure. Otros conectores (como FTP y SQL) requieren una conexión con detalles de configuración como la dirección del servidor, el nombre de usuario y la contraseña. Estos detalles sobre la configuración de la conexión también se cifran y se almacenan de forma segura. Más información sobre el [cifrado en Azure ](../security/fundamentals/encryption-overview.md).

Las conexiones pueden acceder al servicio o sistema de destino siempre que ese servicio o sistema lo permita. En el caso de los servicios que usan conexiones de Azure AD OAuth, como Office 365 y Dynamics, Azure Logic Apps actualiza los tokens de acceso de forma indefinida. Es posible que otros servicios tengan límites con respecto a cuánto tiempo puede usar Azure Logic Apps un token sin actualizar. Por lo general, algunas acciones invalidarán todos los tokens de acceso, por ejemplo, el cambio de la contraseña.

<a name="custom"></a>

## <a name="custom-apis-and-connectors"></a>Conectores y API personalizadas

Para llamar a las API que ejecutan código personalizado o que no están disponibles como conectores, puede extender la plataforma de Logic Apps mediante la [creación de aplicaciones de API personalizadas](../logic-apps/logic-apps-create-api-app.md). También puede [crear conectores personalizados](../logic-apps/custom-connector-overview.md) para *cualquier* API REST o SOAP, para que esas API estén disponibles para cualquier aplicación lógica en su suscripción de Azure. Para que las aplicaciones de API o los conectores personalizados sean públicos y cualquier persona pueda usarlos en Azure, [envíe los conectores para que Microsoft los certifique](/connectors/custom-connectors/submit-certification).

> [!NOTE]
> Las aplicaciones lógicas que implemente y ejecute en un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) pueden acceder directamente a los recursos de una red virtual de Azure. Si tiene conectores personalizados que requieren la puerta de enlace de datos local y los ha creado fuera de un ISE, las aplicaciones lógicas en un ISE también pueden usar esos conectores.
>
> Los conectores personalizados creados dentro de un ISE no funcionan con la puerta de enlace de datos local. Pero estos conectores pueden acceder directamente a orígenes de datos locales que están conectados a una red virtual de Azure en la que se hospeda el ISE. Por tanto, es muy probable que las aplicaciones lógicas en un ISE no necesiten la puerta de enlace de datos cuando se comuniquen con esos recursos.
>
> Para más información sobre cómo crear ISE, vea [Conexión a redes virtuales de Azure desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

<a name="block-connections"></a>

## <a name="block-creating-connections"></a>Bloquear la creación de conexiones

Si su organización no permite la conexión a recursos específicos mediante el uso de sus conectores en Azure Logic Apps, puede [bloquear la capacidad para crear esas conexiones](../logic-apps/block-connections-connectors.md) para conectores específicos en flujos de trabajo de aplicaciones lógicas mediante el uso de [Azure Policy](../governance/policy/overview.md). Para obtener más información, consulte [Bloquear las conexiones creadas por conectores específicos en Azure Logic Apps](../logic-apps/block-connections-connectors.md).

## <a name="get-ready-for-deployment"></a>Preparación para la implementación

Aunque cree conexiones desde una aplicación lógica, estas son recursos de Azure independientes con sus propias definiciones de recursos. Para revisar estas definiciones de recursos de conexión, puede [descargar la aplicación lógica desde Azure en Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md), que es la forma más sencilla de crear una plantilla válida de aplicación lógica con parámetros prácticamente lista para la implementación.

## <a name="next-steps"></a>Pasos siguientes

* Consulte la [lista completa de conectores](/connectors)
* [Creación de una nueva aplicación lógica mediante la conexión de servicios de SaaS](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [Creación de conectores personalizados para aplicaciones lógicas](/connectors/custom-connectors/)
* [Creación de API personalizadas para las aplicaciones lógicas](../logic-apps/logic-apps-create-api-app.md)

<!-- Built-ins icons -->
[azure-api-management-icon]: ./media/apis-list/azure-api-management.png
[azure-app-services-icon]: ./media/apis-list/azure-app-services.png
[azure-functions-icon]: ./media/apis-list/azure-functions.png
[azure-logic-apps-icon]: ./media/apis-list/azure-logic-apps.png
[batch-icon]: ./media/apis-list/batch.png
[condition-icon]: ./media/apis-list/condition.png
[data-operations-icon]: ./media/apis-list/data-operations.png
[date-time-icon]: ./media/apis-list/date-time.png
[for-each-icon]: ./media/apis-list/for-each-loop.png
[http-icon]: ./media/apis-list/http.png
[http-request-icon]: ./media/apis-list/request.png
[http-response-icon]: ./media/apis-list/response.png
[http-swagger-icon]: ./media/apis-list/http-swagger.png
[http-webhook-icon]: ./media/apis-list/http-webhook.png
[inline-code-icon]: ./media/apis-list/inline-code.png
[schedule-icon]: ./media/apis-list/recurrence.png
[scope-icon]: ./media/apis-list/scope.png
[switch-icon]: ./media/apis-list/switch.png
[terminate-icon]: ./media/apis-list/terminate.png
[until-icon]: ./media/apis-list/until.png
[variables-icon]: ./media/apis-list/variables.png

<!--Managed connector icons-->
[appfigures-icon]: ./media/apis-list/appfigures.png
[asana-icon]: ./media/apis-list/asana.png
[azure-automation-icon]: ./media/apis-list/azure-automation.png
[azure-blob-storage-icon]: ./media/apis-list/azure-blob-storage.png
[azure-cognitive-services-text-analytics-icon]: ./media/apis-list/azure-cognitive-services-text-analytics.png
[azure-cosmos-db-icon]: ./media/apis-list/azure-cosmos-db.png
[azure-data-lake-icon]: ./media/apis-list/azure-data-lake.png
[azure-devops-icon]: ./media/apis-list/azure-devops.png
[azure-document-db-icon]: ./media/apis-list/azure-document-db.png
[azure-event-grid-icon]: ./media/apis-list/azure-event-grid.png
[azure-event-grid-publish-icon]: ./media/apis-list/azure-event-grid-publish.png
[azure-event-hubs-icon]: ./media/apis-list/azure-event-hubs.png
[azure-file-storage-icon]: ./media/apis-list/azure-file-storage.png
[azure-key-vault-icon]: ./media/apis-list/azure-key-vault.png
[azure-ml-icon]: ./media/apis-list/azure-ml.png
[azure-monitor-logs-icon]: ./media/apis-list/azure-monitor-logs.png
[azure-queues-icon]: ./media/apis-list/azure-queues.png
[azure-resource-manager-icon]: ./media/apis-list/azure-resource-manager.png
[azure-service-bus-icon]: ./media/apis-list/azure-service-bus.png
[azure-sql-data-warehouse-icon]: ./media/apis-list/azure-sql-data-warehouse.png
[azure-table-storage-icon]: ./media/apis-list/azure-table-storage.png
[basecamp-3-icon]: ./media/apis-list/basecamp.png
[bitbucket-icon]: ./media/apis-list/bitbucket.png
[bitly-icon]: ./media/apis-list/bitly.png
[biztalk-server-icon]: ./media/apis-list/biztalk.png
[blogger-icon]: ./media/apis-list/blogger.png
[campfire-icon]: ./media/apis-list/campfire.png
[common-data-service-icon]: ./media/apis-list/common-data-service.png
[dynamics-365-financials-icon]: ./media/apis-list/dynamics-365-financials.png
[dynamics-365-operations-icon]: ./media/apis-list/dynamics-365-operations.png
[easy-redmine-icon]: ./media/apis-list/easyredmine.png
[file-system-icon]: ./media/apis-list/file-system.png
[ftp-icon]: ./media/apis-list/ftp.png
[github-icon]: ./media/apis-list/github.png
[google-calendar-icon]: ./media/apis-list/google-calendar.png
[google-drive-icon]: ./media/apis-list/google-drive.png
[google-sheets-icon]: ./media/apis-list/google-sheet.png
[google-tasks-icon]: ./media/apis-list/google-tasks.png
[hipchat-icon]: ./media/apis-list/hipchat.png
[ibm-3270-icon]: ./media/apis-list/ibm-3270.png
[ibm-db2-icon]: ./media/apis-list/ibm-db2.png
[ibm-informix-icon]: ./media/apis-list/ibm-informix.png
[ibm-mq-icon]: ./media/apis-list/ibm-mq.png
[insightly-icon]: ./media/apis-list/insightly.png
[instagram-icon]: ./media/apis-list/instagram.png
[instapaper-icon]: ./media/apis-list/instapaper.png
[jira-icon]: ./media/apis-list/jira.png
[mandrill-icon]: ./media/apis-list/mandrill.png
[mysql-icon]: ./media/apis-list/mysql.png
[office-365-outlook-icon]: ./media/apis-list/office-365.png
[onedrive-icon]: ./media/apis-list/onedrive.png
[onedrive-for-business-icon]: ./media/apis-list/onedrive-business.png
[oracle-db-icon]: ./media/apis-list/oracle-db.png
[outlook.com-icon]: ./media/apis-list/outlook.png
[pagerduty-icon]: ./media/apis-list/pagerduty.png
[pinterest-icon]: ./media/apis-list/pinterest.png
[postgre-sql-icon]: ./media/apis-list/postgre-sql.png
[project-online-icon]: ./media/apis-list/projecton-line.png
[redmine-icon]: ./media/apis-list/redmine.png
[salesforce-icon]: ./media/apis-list/salesforce.png
[sap-icon]: ./media/apis-list/sap.png
[send-grid-icon]: ./media/apis-list/sendgrid.png
[sftp-ssh-icon]: ./media/apis-list/sftp.png
[sharepoint-online-icon]: ./media/apis-list/sharepoint-online.png
[sharepoint-server-icon]: ./media/apis-list/sharepoint-server.png
[slack-icon]: ./media/apis-list/slack.png
[smartsheet-icon]: ./media/apis-list/smartsheet.png
[smtp-icon]: ./media/apis-list/smtp.png
[sparkpost-icon]: ./media/apis-list/sparkpost.png
[sql-server-icon]: ./media/apis-list/sql.png
[teradata-icon]: ./media/apis-list/teradata.png
[todoist-icon]: ./media/apis-list/todoist.png
[twilio-icon]: ./media/apis-list/twilio.png
[vimeo-icon]: ./media/apis-list/vimeo.png
[wordpress-icon]: ./media/apis-list/wordpress.png
[youtube-icon]: ./media/apis-list/youtube.png

<!-- Enterprise Integration Pack icons -->
[as2-icon]: ./media/apis-list/as2.png
[edifact-icon]: ./media/apis-list/edifact.png
[flat-file-encode-icon]: ./media/apis-list/flat-file-encoding.png
[flat-file-decode-icon]: ./media/apis-list/flat-file-decoding.png
[integration-account-icon]: ./media/apis-list/integration-account.png
[liquid-icon]: ./media/apis-list/liquid-transform.png
[x12-icon]: ./media/apis-list/x12.png
[xml-validate-icon]: ./media/apis-list/xml-validation.png
[xml-transform-icon]: ./media/apis-list/xsl-transform.png

<!--Other doc links-->
[gateway-doc]: ../logic-apps/logic-apps-gateway-connection.md "Conexión a orígenes de datos locales desde aplicaciones lógicas con una puerta de enlace de datos local"

<!--Built-in doc links-->
[azure-api-management-doc]: ../api-management/get-started-create-service-instance.md "Creación de una instancia del servicio Azure API Management para administrar y publicar sus API"
[azure-app-services-doc]: ../logic-apps/logic-apps-custom-api-host-deploy-call.md "Integrar aplicaciones lógicas con App Service API Apps"
[azure-functions-doc]: ../logic-apps/logic-apps-azure-functions.md "Integración de aplicaciones lógicas con Azure Functions"
[batch-doc]: ../logic-apps/logic-apps-batch-process-send-receive-messages.md "Procesamiento de mensajes en grupos o lotes"
[condition-doc]: ../logic-apps/logic-apps-control-flow-conditional-statement.md "Evaluar una condición y ejecutar acciones diferentes según si la condición es true o false"
[for-each-doc]: ../logic-apps/logic-apps-control-flow-loops.md#foreach-loop "Realizar las mismas acciones en todos los elementos de una matriz"
[http-doc]: ./connectors-native-http.md "Llamar a puntos de conexión HTTP o HTTPS desde las aplicaciones lógicas"
[http-request-doc]: ./connectors-native-reqres.md "Recibir solicitudes HTTP en las aplicaciones lógicas"
[http-response-doc]: ./connectors-native-reqres.md "Responder a solicitudes HTTP desde las aplicaciones lógicas"
[http-swagger-doc]: ./connectors-native-http-swagger.md "Llamar a puntos de conexión REST desde las aplicaciones lógicas"
[http-webhook-doc]: ./connectors-native-webhook.md "Esperar eventos específicos de los puntos de conexión HTTP o HTTPS"
[inline-code-doc]: ../logic-apps/logic-apps-add-run-inline-code.md "Agregar y ejecutar fragmentos de código JavaScript de las aplicaciones lógicas"
[nested-logic-app-doc]: ../logic-apps/logic-apps-http-endpoint.md "Integrar aplicaciones lógicas con flujos de trabajo anidados"
[query-doc]: ../logic-apps/logic-apps-perform-data-operations.md#filter-array-action "Seleccionar y filtrar matrices con la acción Consulta"
[schedule-doc]: ../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md "Ejecutar aplicaciones lógicas en función de una programación"
[schedule-delay-doc]: ./connectors-native-delay.md "Retrasar la ejecución de la siguiente acción"
[schedule-delay-until-doc]: ./connectors-native-delay.md "Retrasar la ejecución de la siguiente acción"
[schedule-recurrence-doc]:  ./connectors-native-recurrence.md "Ejecutar aplicaciones lógicas según una programación periódica"
[schedule-sliding-window-doc]: ./connectors-native-sliding-window.md "Ejecutar aplicaciones lógicas que necesitan manipular datos en fragmentos contiguos"
[scope-doc]: ../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md "Organizar las acciones en grupos, que obtienen su propio estado después de que las acciones del grupo terminen de ejecutarse"
[switch-doc]: ../logic-apps/logic-apps-control-flow-switch-statement.md "Organizar las acciones en los casos, a los que se asignan valores únicos. Ejecutar solo el caso cuyo valor asignado coincida con el resultado de una expresión, objeto o token. Si no hay coincidencias, ejecutar el caso predeterminado"
[terminate-doc]: ../logic-apps/logic-apps-workflow-actions-triggers.md#terminate-action "Detener o cancelar un flujo de trabajo que se está ejecutando activamente para la aplicación lógica"
[until-doc]: ../logic-apps/logic-apps-control-flow-loops.md#until-loop "Repetir las acciones hasta que la condición especificada sea true o cambie algún estado"
[data-operations-doc]: ../logic-apps/logic-apps-perform-data-operations.md "Realizar operaciones de datos como el filtrado de matrices o la creación de tablas CSV y HTML"
[variables-doc]: ../logic-apps/logic-apps-create-variables-store-values.md "Realizar operaciones con variables, por ejemplo, inicializar, establecer, incrementar, reducir y anexar a la variable de cadena o matriz"

<!--Managed connector doc links-->
[azure-automation-doc]: /connectors/azureautomation/ "Creación y administración de trabajos de Automation para su infraestructura local y en la nube"
[azure-blob-storage-doc]: ./connectors-create-api-azureblobstorage.md "Administrar archivos del contenedor de blobs con el conector de Azure Blob Storage"
[azure-cosmos-db-doc]: /connectors/documentdb/ "Conectarse a Azure Cosmos DB para tener acceso a documentos y procedimientos almacenados"
[azure-event-grid-doc]: ../event-grid/monitor-virtual-machine-changes-event-grid-logic-app.md "Supervisar los eventos publicados por Event Grid, por ejemplo, cuando hay cambios en los recursos de Azure o de terceros"
[azure-event-hubs-doc]: ./connectors-create-api-azure-event-hubs.md "Conectarse a Azure Event Hubs para recibir y enviar eventos entre aplicaciones lógicas y Event Hubs"
[azure-file-storage-doc]: /connectors/azurefile/ "Conectarse a su cuenta de Azure Storage para crear, actualizar, obtener y eliminar archivos"
[azure-key-vault-doc]: /connectors/keyvault/ "Conexión a la instancia de Azure Key Vault para administrar sus secretos y claves"
[azure-monitor-logs-doc]: /connectors/azuremonitorlogs/ "Ejecución de consultas en los registros de Azure Monitor de las áreas de trabajo de Log Analytics y los componentes de Application Insights"
[azure-queues-doc]: /connectors/azurequeues/ "Conectarse a su cuenta de Azure Storage para crear y administrar colas y mensajes"
[azure-service-bus-doc]: ./connectors-create-api-servicebus.md "Envío de mensajes desde los temas y las colas de Service Bus y recepción de mensajes desde suscripciones y colas de Service Bus"
[azure-sql-data-warehouse-doc]: /connectors/sqldw/ "Conectarse a Azure SQL Data Warehouse para visualizar los datos"
[azure-table-storage-doc]: /connectors/azuretables/ "Conectarse a su cuenta de Azure Storage para crear, actualizar y consultar tablas y más"
[biztalk-server-doc]: /connectors/biztalk/ "Conectarse a su instancia de BizTalk Server para ejecutar aplicaciones basadas en BizTalk en paralelo con Azure Logic Apps"
[file-system-doc]: ../logic-apps/logic-apps-using-file-connector.md "Conectar con un sistema de archivos local"
[ftp-doc]: ./connectors-create-api-ftp.md "Conectar con un servidor FTP/FTPS para tareas de FTP, incluidas la carga, la obtención y la eliminación de archivos, entre otras"
[github-doc]: ./connectors-create-api-github.md "Conectar con GitHub y realizar un seguimiento de los problemas"
[google-calendar-doc]: ./connectors-create-api-googlecalendar.md "Conectarse con Google Calendar y administrar el calendario"
[google-sheets-doc]: ./connectors-create-api-googlesheet.md "Conectarse con Google Sheets para modificar sus hojas"
[google-tasks-doc]: ./connectors-create-api-googletasks.md "Conectarse con Google Tasks para administrar las tareas"
[ibm-3270-doc]: ./connectors-run-3270-apps-ibm-mainframe-create-api-3270.md "Conexión a aplicaciones de 3270 en sistemas centrales de IBM"
[ibm-db2-doc]: ./connectors-create-api-db2.md "Conectar con una instancia de IBM DB2 en la nube o local. Actualizar una fila, obtener una tabla, etc."
[ibm-informix-doc]: ./connectors-create-api-informix.md "Conectar con una instancia de Informix en la nube o local. Leer una fila, enumerar las tablas, etc."
[ibm-mq-doc]: ./connectors-create-api-mq.md "Conectarse a IBM MQ local o en Azure y enviar y recibir mensajes"
[instagram-doc]: ./connectors-create-api-instagram.md "Conectar con Instagram. Desencadenar eventos o realizar acciones en ellos"
[mandrill-doc]: ./connectors-create-api-mandrill.md "Conectar con Mandrill para realizar comunicaciones"
[mysql-doc]: /connectors/mysql/ "Conectarse con su base de datos de MySQL local para que pueda leer y escribir datos"
[office-365-outlook-doc]: ./connectors-create-api-office365-outlook.md "Conectarse a su cuenta de Office 365 para enviar y recibir correos electrónicos, administrar su calendario y contactos y mucho más"
[onedrive-doc]: ./connectors-create-api-onedrive.md "Conectarse a la instancia personal de Microsoft OneDrive para cargar, eliminar, mostrar archivos y mucho más"
[onedrive-for-business-doc]: ./connectors-create-api-onedriveforbusiness.md "Conectarse a la instancia de Microsoft OneDrive de su empresa para cargar, eliminar, mostrar archivos y mucho más"
[oracle-db-doc]: ./connectors-create-api-oracledatabase.md "Conectarse a una base de datos de Oracle para agregar, insertar, eliminar filas y mucho más"
[outlook.com-doc]: ./connectors-create-api-outlook.md "Conectarse a su buzón de correo de Outlook para administrar sus correos, calendarios, contactos y mucho más"
[postgre-sql-doc]: /connectors/postgresql/ "Conectarse a su base de datos de PostgreSQL para leer datos de las tablas"
[salesforce-doc]: ./connectors-create-api-salesforce.md "Conectar con su cuenta de Salesforce. Administrar cuentas, clientes potenciales, oportunidades, etc."
[sap-connector-doc]: ../logic-apps/logic-apps-using-sap-connector.md "Conectar con un servidor SAP local"
[sendgrid-doc]: ./connectors-create-api-sendgrid.md "Conectar con SendGrid. Enviar correo electrónico y administrar listas de destinatarios"
[sftp-ssh-doc]: ./connectors-sftp-ssh.md "Conectarse con su cuenta SFTP mediante SSH. Cargar, obtener y eliminar archivos, entre otras operaciones."
[sharepoint-server-doc]: ./connectors-create-api-sharepoint.md "Conectarse al servidor local de SharePoint. Administrar documentos, elementos de lista, etc."
[sharepoint-online-doc]: ./connectors-create-api-sharepoint.md "Conectar con SharePoint Online. Administrar documentos, elementos de lista, etc."
[slack-doc]: ./connectors-create-api-slack.md "Conectar con Slack y publicar mensajes en los canales de Slack"
[smtp-doc]: ./connectors-create-api-smtp.md "Conectar con un servidor SMTP y enviar correo con datos adjuntos"
[sparkpost-doc]: ./connectors-create-api-sparkpost.md "Conectar con SparkPost para la comunicación"
[sql-server-doc]: ./connectors-create-api-sqlazure.md "Conectar con Azure SQL Database o SQL Server. Crear, actualizar, obtener y eliminar entradas en una tabla de base de datos SQL"
[teradata-doc]: /connectors/teradata/ "Conectarse a su base de datos de Teradata para leer datos de las tablas"
[twilio-doc]: ./connectors-create-api-twilio.md "Conectar con Twilio. Enviar y recibir mensajes, obtener los números disponibles, administrar los números de teléfono entrantes y mucho más."
[youtube-doc]: ./connectors-create-api-youtube.md "Conectar con YouTube. Administrar sus vídeos y canales"

<!--Enterprise Intregation Pack doc links-->
[as2-doc]: ../logic-apps/logic-apps-enterprise-integration-as2.md "Codificar y descodificar mensajes que usan el protocolo AS2"
[edifact-doc]: ../logic-apps/logic-apps-enterprise-integration-edifact.md "Codificar y descodificar mensajes que usan el protocolo EDIFACT"
[edifact-decode-doc]: ../logic-apps/logic-apps-enterprise-integration-EDIFACT-decode.md "Descodificar mensajes que usan el protocolo EDIFACT"
[edifact-encode-doc]: ../logic-apps/logic-apps-enterprise-integration-EDIFACT-encode.md "Codificar mensajes que usan el protocolo EDIFACT"
[flat-file-decode-doc]:../logic-apps/logic-apps-enterprise-integration-flatfile.md "Más información sobre archivos planos para integración empresarial"
[flat-file-encode-doc]:../logic-apps/logic-apps-enterprise-integration-flatfile.md "Más información sobre archivos planos para integración empresarial"
[integration-account-doc]: ../logic-apps/logic-apps-enterprise-integration-metadata.md "Administrar metadatos de artefactos para cuentas de integración"
[json-liquid-transform-doc]: ../logic-apps/logic-apps-enterprise-integration-liquid-transform.md "Transformar JSON mediante plantillas de Liquid"
[x12-doc]: ../logic-apps/logic-apps-enterprise-integration-x12.md "Codificar y descodificar mensajes que usan el protocolo X12"
[x12-decode-doc]: ../logic-apps/logic-apps-enterprise-integration-X12-decode.md "Descodificar mensajes que usan el protocolo X12"
[x12-encode-doc]: ../logic-apps/logic-apps-enterprise-integration-X12-encode.md "Codificar mensajes que usan el protocolo X12"
[xml-transform-doc]: ../logic-apps/logic-apps-enterprise-integration-transform.md "Transformar mensajes XML"
[xml-validate-doc]: ../logic-apps/logic-apps-enterprise-integration-xml-validation.md "Validar mensajes XML"
