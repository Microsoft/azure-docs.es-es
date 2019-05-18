---
title: Límites y configuración - Azure Logic Apps | Microsoft Docs
description: Valores de límites y configuración del servicio para Azure Logic Apps
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 05/10/2019
ms.openlocfilehash: a320f584ff82f2b8a2b3d784e1995aa043004587
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597492"
---
# <a name="limits-and-configuration-information-for-azure-logic-apps"></a>Información de límites y configuración para Azure Logic Apps

En este artículo, se describen los límites y los detalles de configuración para crear y ejecutar flujos de trabajo automatizados con Azure Logic Apps. Para Microsoft Flow, consulte [Límites y configuración en Microsoft Flow](https://docs.microsoft.com/flow/limits-and-config).

<a name="definition-limits"></a>

## <a name="definition-limits"></a>Límites de definición

Estos son los límites de una definición de aplicación lógica:

| NOMBRE | Límite | Notas |
| ---- | ----- | ----- |
| Acciones por flujo de trabajo | 500 | Para ampliar este límite, puede agregar flujos de trabajo anidados según sea necesario. |
| Niveles de anidamiento permitidos para acciones | 8 | Para ampliar este límite, puede agregar flujos de trabajo anidados según sea necesario. |
| Flujos de trabajo por región y suscripción | 1000 | |
| Desencadenadores por flujo de trabajo | 10 | Al trabajar en la vista Código, no en el diseñador |
| Límite de cambio de ámbito de casos | 25 | |
| Variables por flujo de trabajo | 250 | |
| Caracteres por expresión | 8192 | |
| Tamaño máximo de `trackedProperties` | 16 000 caracteres |
| Nombre de `action` o `trigger` | 80 caracteres | |
| Longitud de `description` | 256 caracteres | |
| Máximo de `parameters` | 50 | |
| Máximo de `outputs` | 10 | |
||||

<a name="run-duration-retention-limits"></a>

## <a name="run-duration-and-retention-limits"></a>Límites de retención y duración de ejecución

Estos son los límites de ejecución de una única aplicación lógica:

| NOMBRE | Límite de varios inquilinos | Límite de entorno del servicio de integración | Notas |
|------|--------------------|---------------------------------------|-------|
| Duración de la ejecución | 90 días | 365 días | Para cambiar el límite predeterminado, consulte [duración de la ejecución de cambio](#change-duration). |
| Retención de almacenamiento | 90 días a partir de la hora de inicio de la ejecución | 365 días | Para cambiar el límite predeterminado, consulte [cambiar retención de almacenamiento](#change-retention). |
| Intervalo de periodicidad mínima | 1 segundo | 1 segundo ||
| Intervalo de periodicidad máxima | 500 días | 500 días ||
|||||

<a name="change-duration"></a>
<a name="change-retention"></a>

### <a name="change-run-duration-and-storage-retention"></a>Cambio de la duración de ejecución y la retención de almacenamiento

Para cambiar el límite predeterminado para la duración de ejecución y la retención de almacenamiento, siga estos pasos. Si necesita superar el límite máximo, [póngase en contacto con el equipo de Logic Apps](mailto://logicappsemail@microsoft.com) para obtener ayuda con sus requisitos.

1. En Azure Portal, en el menú de la aplicación lógica, elija **Configuración del flujo de trabajo**.

2. En **Opciones del entorno de ejecución**, en la lista **Retención del historial de ejecución en días** elija **Personalizado**.

3. Escriba o arrastre el control deslizante hasta el número de días que quiera.

<a name="looping-debatching-limits"></a>

## <a name="concurrency-looping-and-debatching-limits"></a>Límites de simultaneidad, bucle y anulación de procesamiento por lotes

Estos son los límites de ejecución de una única aplicación lógica:

| NOMBRE | Límite | Notas |
| ---- | ----- | ----- |
| Simultaneidad de desencadenadores | * Ilimitado cuando el control de simultaneidad está desactivado. <p><p>* 25 es el límite predeterminado cuando está activado el control de simultaneidad. No se puede revertir después de activar el control. Puede cambiar el valor predeterminado por otro entre 1 y 50, ambos incluidos. | Este límite describe el número más alto de instancias de aplicaciones lógicas que se pueden ejecutar al mismo tiempo o en paralelo. <p><p>Para cambiar el límite predeterminado a un valor comprendido entre 1 y 50 (ambos inclusive), consulte [Cambio en la simultaneidad de desencadenadores](../logic-apps/logic-apps-workflow-actions-triggers.md#change-trigger-concurrency) o [Desencadenamiento secuencial de instancias](../logic-apps/logic-apps-workflow-actions-triggers.md#sequential-trigger). |
| Número máximo de ejecuciones en espera | Cuando se activa el control de simultaneidad, el número mínimo de ejecuciones en espera es 10 más el número de ejecuciones simultáneas (simultaneidad del desencadenador). Puede cambiar el número máximo hasta 100, incluido. | Este límite describe el número más alto de instancias de aplicaciones lógicas que se pueden poner en espera de ejecución en caso de que la aplicación lógica ya esté ejecutando el número máximo de instancias simultáneas. <p><p>Para cambiar el límite predeterminado, consulte [Cambio del límite de ejecuciones en espera](../logic-apps/logic-apps-workflow-actions-triggers.md#change-waiting-runs). |
| Elementos de matriz de foreach | 100 000 | Este límite describe el número más alto de elementos de matriz que puede procesar un bucle "for each". <p><p>Para filtrar matrices más grandes, puede usar la [acción de consulta](../connectors/connectors-native-query.md). |
| Simultaneidad de foreach | El límite predeterminado es 20 cuando el control de simultaneidad está desactivado. Puede cambiar el valor predeterminado por otro entre 1 y 50, ambos incluidos. | Este límite es el número más alto de iteraciones de bucles "for each" que se pueden ejecutar al mismo tiempo o en paralelo. <p><p>Para cambiar el límite predeterminado a un valor comprendido entre 1 y 50 (ambos inclusive), consulte [Cambio en el límite de la simultaneidad de los bucles "for each"](../logic-apps/logic-apps-workflow-actions-triggers.md#change-for-each-concurrency) o [Ejecución secuencial de bucles "for each"](../logic-apps/logic-apps-workflow-actions-triggers.md#sequential-for-each). |
| Elementos SplitOn | 100 000 | Para los desencadenadores que devuelven una matriz, puede especificar una expresión que use la propiedad "SplitOn", la cual [divide o desagrupa los elementos de matriz en varias instancias de flujo de trabajo](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch) para su procesamiento, en lugar de usar un bucle "foreach". Esta expresión hace referencia a la matriz que se usará para crear y ejecutar una instancia de flujo de trabajo para cada elemento de la matriz. |
| Iteraciones Until | 5.000 | |
||||

<a name="throughput-limits"></a>

## <a name="throughput-limits"></a>Límites de rendimiento

Estos son los límites de ejecución de una única aplicación lógica:

### <a name="multi-tenant-logic-apps-service"></a>Servicio de Logic Apps de varios inquilinos

| NOMBRE | Límite | Notas |
| ---- | ----- | ----- |
| Acción: Ejecuciones por cada 5 minutos | 100.000 es el límite predeterminado, pero 300.000 es el límite máximo. | Para cambiar el límite predeterminado, consulte [Ejecución de la aplicación lógica en modo de "alto rendimiento"](../logic-apps/logic-apps-workflow-actions-triggers.md#run-high-throughput-mode), que está en versión preliminar. O bien puede distribuir la carga de trabajo entre varias aplicaciones lógicas según sea necesario. |
| Acción: Llamadas salientes simultáneas | Aproximadamente 2500 | Puede reducir el número de solicitudes simultáneas o la duración según sea necesario. |
| Punto de conexión en tiempo de ejecución: Llamadas entrantes simultáneas | ~1000 | Puede reducir el número de solicitudes simultáneas o la duración según sea necesario. |
| Punto de conexión en tiempo de ejecución: Llamadas de lectura cada 5 minutos  | 60 000 | Puede distribuir una carga de trabajo entre varias aplicaciones, según sea necesario. |
| Punto de conexión en tiempo de ejecución: Invocación de llamadas cada 5 minutos | 45 000 | Puede distribuir una carga de trabajo entre varias aplicaciones, según sea necesario. |
| Rendimiento de contenido cada 5 minutos | 600 MB | Puede distribuir una carga de trabajo entre varias aplicaciones, según sea necesario. |
||||

### <a name="integration-service-environment-ise"></a>Entorno de servicio de integración (ISE)

| NOMBRE | Límite | Notas |
|------|-------|-------|
| Límite de ejecución de la unidad base | Limitado del sistema cuando la capacidad de la infraestructura alcanza el 80% | Proporciona las ejecuciones de acción ~ 4.000 por minuto, que es de ~ 160 millones de ejecuciones de acción al mes | |
| Límite de ejecución de unidad de escalado | Limitado del sistema cuando la capacidad de la infraestructura alcanza el 80% | Cada unidad de escalado puede proporcionar aproximadamente 2.000 ejecuciones de acción adicional por minuto, que es aproximadamente 80 millones más ejecuciones de acción al mes | |
| Unidades de escala máxima que puede agregar | 10 | |
||||

Si quiere superar estos límites en el procesamiento normal, o ejecutar pruebas de carga que puedan superar estos límites, [póngase en contacto con el equipo de Logic Apps](mailto://logicappsemail@microsoft.com) para que podamos ayudarle con sus solicitudes.

<a name="request-limits"></a>

## <a name="http-limits"></a>Límites de HTTP

Estos son los límites de una única solicitud HTTP o llamada de conector sincrónica:

#### <a name="timeout"></a>Tiempo de espera

Algunas operaciones de conector realizan llamadas asincrónicas o escuchan las solicitudes de webhook, por lo que el tiempo de expiración de estas operaciones puede ser superior a estos límites. Para obtener más información, vea los detalles técnicos del conector concreto y también [Acciones y desencadenadores de flujos de trabajo](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action).

| NOMBRE | Límite de varios inquilinos | Límite de entorno del servicio de integración | Notas |
|------|--------------------|---------------------------------------|-------|
| Solicitud saliente | 120 segundos | 240 segundos | Para las operaciones de ejecución más largas, use un [patrón de sondeo asincrónico](../logic-apps/logic-apps-create-api-app.md#async-pattern) o un [bucle Until](../logic-apps/logic-apps-workflow-actions-triggers.md#until-action). |
| Respuesta sincrónica | 120 segundos | 240 segundos | Para que la solicitud original obtenga la respuesta, todos los pasos de la respuesta deben terminar dentro del límite, a menos que llame a otra aplicación lógica como un flujo de trabajo anidado. Para obtener más información, consulte [Llamada, desencadenamiento o anidación de aplicaciones lógicas](../logic-apps/logic-apps-http-endpoint.md). |
|||||

#### <a name="message-size"></a>Tamaño del mensaje

| NOMBRE | Límite de varios inquilinos | Límite de entorno del servicio de integración | Notas |
|------|--------------------|---------------------------------------|-------|
| Tamaño del mensaje | 100 MB | 200 MB | Para evitar este límite, consulte [Handle large messages with chunking](../logic-apps/logic-apps-handle-large-messages.md) (Controlar mensajes grandes con fragmentación). En cambio, puede que algunos conectores y API no admitan la fragmentación ni el límite predeterminado. |
| Tamaño del mensaje con fragmentación | 1 GB | 5 GB | Este límite se aplica a las acciones que admiten la fragmentación de forma nativa o que le permiten habilitar la opción de fragmentación en la configuración del entorno de ejecución. <p>Para el entorno de servicio de integración, el motor de Logic Apps es compatible con este límite, pero los conectores tienen sus propios límites fragmentación hasta el límite del motor, por ejemplo, vea [conector de Azure Blob Storage](/connectors/azureblob/). Para obtener más información fragmentación, consulte [controlar mensajes de gran tamaño con fragmentación](../logic-apps/logic-apps-handle-large-messages.md). |
| Límite de evaluación de expresiones | 131 072 caracteres. | 131 072 caracteres. | Las expresiones `@concat()`, `@base64()` y `@string()` no pueden superar este límite. |
|||||

#### <a name="retry-policy"></a>Directiva de reintentos

| NOMBRE | Límite | Notas |
| ---- | ----- | ----- |
| Intentos de reintento | 90 | El valor predeterminado es 4. Para cambiar el valor predeterminado, use el [parámetro de directiva de reintentos](../logic-apps/logic-apps-workflow-actions-triggers.md). |
| Retraso máximo entre reintentos | 1 día | Para cambiar el valor predeterminado, use el [parámetro de directiva de reintentos](../logic-apps/logic-apps-workflow-actions-triggers.md). |
| Retraso mínimo entre reintentos | 5 segundos | Para cambiar el valor predeterminado, use el [parámetro de directiva de reintentos](../logic-apps/logic-apps-workflow-actions-triggers.md). |
||||

<a name="custom-connector-limits"></a>

## <a name="custom-connector-limits"></a>Límites de conector personalizado

Estos son los límites para los conectores personalizados que puede crear a partir de las API web.

| NOMBRE | Límite de varios inquilinos | Límite de entorno del servicio de integración | Notas |
|------|--------------------|---------------------------------------|-------|
| Número de conectores personalizados | 1000 por cada suscripción de Azure | 1000 por cada suscripción de Azure ||
| Número de solicitudes por minuto para un conector personalizado | 500 solicitudes por minuto por conexión | 2.000 solicitudes por minuto por *conector personalizado* ||
|||

<a name="managed-identity"></a>

## <a name="managed-identities"></a>Identidades administradas

| NOMBRE | Límite |
| ---- | ----- |
| Número de aplicaciones lógicas con identidades administradas asignadas por el sistema por suscripción de Azure | 10 |
|||

<a name="integration-account-limits"></a>

## <a name="integration-account-limits"></a>Límites de cuenta de integración

<a name="artifact-number-limits"></a>

### <a name="artifact-limits-per-integration-account"></a>Límites de artefacto por cuenta de integración

Estos son los límites en cuanto al número de artefactos de cada cuenta de integración. Para obtener más información, consulte [Precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

> [!NOTE] 
> Use el nivel gratis solo para escenarios de exploración, no los escenarios de producción. Este nivel restringe el uso y el rendimiento, y no tiene ningún acuerdo de nivel de servicio (SLA).

| Artefacto | Gratis | Básica | Estándar |
|----------|------|-------|----------|
| Acuerdos comerciales de EDI | 10 | 1 | 500 |
| Socios comerciales de EDI | 25 | 2 | 500 |
| Mapas | 25 | 500 | 1000 |
| Esquemas | 25 | 500 | 1000 |
| Ensamblados | 10 | 25 | 50 |
| Certificados | 25 | 2 | 500 |
| Configuraciones por lotes | 5 | 1 | 50 |
||||

<a name="artifact-capacity-limits"></a>

### <a name="artifact-capacity-limits"></a>Límites de capacidad de artefactos

| Artefacto | Límite | Notas |
| -------- | ----- | ----- |
| Ensamblado | 8 MB | Para cargar archivos de más de 2 MB, use una [cuenta de almacenamiento y un contenedor de blobs de Azure](../logic-apps/logic-apps-enterprise-integration-schemas.md). |
| Asignación (archivo XSLT) | 8 MB | Para cargar archivos de más de 2 MB, use las [asignaciones de la API REST de Azure Logic Apps](https://docs.microsoft.com/rest/api/logic/maps/createorupdate). |
| Esquema | 8 MB | Para cargar archivos de más de 2 MB, use una [cuenta de almacenamiento y un contenedor de blobs de Azure](../logic-apps/logic-apps-enterprise-integration-schemas.md). |
||||

| Punto de conexión en tiempo de ejecución | Límite | Notas |
|------------------|-------|-------|
| Llamadas de lectura cada 5 minutos | 60 000 | Puede distribuir la carga de trabajo entre varias cuentas según sea necesario. |
| Invocación de llamadas cada 5 minutos | 45 000 | Puede distribuir la carga de trabajo entre varias cuentas según sea necesario. |
| Seguimiento de llamadas cada 5 minutos | 45 000 | Puede distribuir la carga de trabajo entre varias cuentas según sea necesario. |
| Bloqueo de llamadas simultáneas | ~1000 | Puede reducir el número de solicitudes simultáneas o la duración según sea necesario. |
||||

<a name="b2b-protocol-limits"></a>

### <a name="b2b-protocol-as2-x12-edifact-message-size"></a>Tamaño de mensaje de protocolo B2B (AS2, X12 y EDIFACT)

Estos son los límites de tamaño de mensaje que se aplican a los protocolos B2B:

| NOMBRE | Límite de varios inquilinos | Límite de entorno del servicio de integración | Notas |
|------|--------------------|---------------------------------------|-------|
| AS2 | v2 - 100 MB<br>v1 - 50 MB | v2 - 200 MB <br>v1 - 50 MB | Se aplica a la decodificación y la codificación |
| X12 | 50 MB | 50 MB | Se aplica a la decodificación y la codificación |
| EDIFACT | 50 MB | 50 MB | Se aplica a la decodificación y la codificación |
||||

<a name="disable-delete"></a>

## <a name="disabling-or-deleting-logic-apps"></a>Deshabilitación o eliminación de aplicaciones lógicas

Cuando se deshabilita una aplicación lógica, no se crean instancias de nuevas ejecuciones. Todo en curso y ejecuciones pendientes continúan hasta que finalice, lo cual puede tardar tiempo en completarse.

Cuando se elimina una aplicación lógica, no se crean instancias de nuevas ejecuciones. Todas las ejecuciones nuevas y pendientes se cancelan. Si tiene miles de ejecuciones, la cancelación puede tardar bastante tiempo en completarse.

<a name="configuration"></a>

## <a name="firewall-configuration-ip-addresses"></a>Configuración del firewall: Direcciones IP

Todas las aplicaciones lógicas de una misma región usan los mismos intervalos de direcciones IP. Para admitir las llamadas que sus aplicaciones lógicas realizan directamente con [HTTP](../connectors/connectors-native-http.md), [HTTP + Swagger](../connectors/connectors-native-http-swagger.md) y otras solicitudes HTTP, configure el firewall con *todas* las direcciones IP [entrantes](#inbound) *y* [salientes](#outbound) que usa el servicio Logic Apps en función de la región en la que estén las aplicaciones lógicas. Estas direcciones aparecen bajo los encabezados **Entrante** y **Saliente** de esta sección y están ordenadas por región.

Para admitir las llamadas que realizan los [conectores administrados de Microsoft](../connectors/apis-list.md), configure el firewall con *todas* las direcciones IP [salientes](#outbound) que usan dichos conectores en función de las regiones en las que estén las aplicaciones lógicas. Estas direcciones aparecen bajo el encabezado **Saliente** de esta sección y están ordenadas por región.

Para [Azure Government](../azure-government/documentation-government-overview.md) y [Azure China 21Vianet](/azure/china/china-welcome), las direcciones IP reservadas para los conectores no están disponibles actualmente.

> [!IMPORTANT]
>
> Si ya tiene algunas opciones de configuración, actualícelas **tan pronto como le sea posible antes del 1 de septiembre de 2018** para que incluyan y coincidan con las direcciones IP de estas listas de las regiones en las que están las aplicaciones lógicas.

Logic Apps no admite la conexión directa a las cuentas de Azure Storage a través de firewalls. Para acceder a estas cuentas de almacenamiento, use cualquiera de estas opciones:

* Cree un [entorno de servicio de integración](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) que pueda conectarse a los recursos de una red virtual de Azure.

* Si ya usa API Management, puede usar este servicio para el escenario. Para obtener más información, vea [Arquitectura de integración empresarial sencilla](https://aka.ms/aisarch).

<a name="inbound"></a>

### <a name="inbound-ip-addresses---logic-apps-service-only"></a>Direcciones IP de entrada: solo del servicio Logic Apps

| Área | IP |
|--------|----|
| Este de Australia | 13.75.153.66, 52.187.231.161, 104.210.89.222, 104.210.89.244 |
| Sudeste de Australia | 13.73.115.153, 40.115.78.70, 40.115.78.237, 52.189.216.28 |
| Sur de Brasil | 191.234.166.198, 191.235.86.199, 191.235.94.220, 191.235.95.229 |
| Centro de Canadá | 13.88.249.209, 40.85.241.105, 52.233.29.79, 52.233.30.218 |
| Este de Canadá | 40.86.202.42, 52.229.125.57, 52.232.129.143, 52.232.133.109 |
| India Central | 52.172.157.194, 52.172.184.192, 52.172.191.194, 104.211.73.195 |
| Centro de EE. UU. | 13.67.236.76, 40.77.31.87, 40.77.111.254, 104.43.243.39 |
| Asia oriental | 13.75.89.159, 23.97.68.172, 40.83.98.194, 168.63.200.173 |
| Este de EE. UU | 40.117.99.79, 40.117.100.228, 137.116.126.165, 137.135.106.54 |
| Este de EE. UU. 2 | 40.70.27.253, 40.79.44.7, 40.84.25.234, 40.84.59.136 |
| Este de Japón | 13.71.146.140, 13.78.43.164, 13.78.62.130, 13.78.84.187 |
| Oeste de Japón | 40.74.68.85, 40.74.81.13, 40.74.85.215, 40.74.140.173 |
| Centro-Norte de EE. UU | 65.52.9.64, 65.52.211.164, 168.62.249.81, 157.56.12.202 |
| Europa del Norte | 13.79.173.49, 40.112.90.39, 52.169.218.253, 52.169.220.174 |
| Centro-Sur de EE. UU | 13.65.98.39, 13.84.41.46, 13.84.43.45, 40.84.138.132 |
| Sur de la India | 52.172.9.47, 52.172.49.43, 52.172.51.140, 104.211.225.152 |
| Sudeste asiático | 52.163.93.214, 52.187.65.81, 52.187.65.155, 104.215.181.6 |
| Centro occidental de EE.UU. | 13.78.137.247, 52.161.8.128, 52.161.19.82, 52.161.26.172 |
| Europa occidental | 13.95.155.53, 51.144.176.185, 52.174.49.6, 52.174.54.218 |
| Oeste de la India | 104.211.157.237, 104.211.164.25, 104.211.164.112, 104.211.165.81 |
| Oeste de EE. UU. | 13.91.252.184, 52.160.90.237, 138.91.188.137, 157.56.160.212 |
| Oeste de EE. UU. 2 | 13.66.128.68, 13.66.224.169, 52.183.30.10, 52.183.39.67 |
| Sur de Reino Unido 2 | 51.140.78.71, 51.140.79.109, 51.140.84.39, 51.140.155.81 |
| Oeste de Reino Unido | 51.141.48.98, 51.141.51.145, 51.141.53.164, 51.141.119.150 |
| | |

<a name="outbound"></a>

### <a name="outbound-ip-addresses---logic-apps-service--managed-connectors"></a>Direcciones IP de salida: servicio Logic Apps y conectores administrados

| Área | IP de Logic Apps | IP de conectores administrados |
|--------|---------------|-----------------------|
| Este de Australia | 13.75.149.4, 52.187.226.96, 52.187.226.139, 52.187.227.245, 52.187.229.130, 52.187.231.184, 104.210.90.241, 104.210.91.55 | 13.70.72.192 - 13.70.72.207, 13.72.243.10, 40.126.251.213 |
| Sudeste de Australia | 13.70.159.205, 13.73.114.207, 13.77.3.139, 13.77.56.167, 13.77.58.136, 52.189.214.42, 52.189.220.75, 52.189.222.77 | 13.70.136.174, 13.77.50.240 - 13.77.50.255, 40.127.80.34 |
| Sur de Brasil | 191.234.161.28, 191.234.161.168, 191.234.162.131, 191.234.162.178, 191.234.182.26, 191.235.82.221, 191.235.91.7, 191.237.255.116 | 104.41.59.51, 191.232.38.129, 191.233.203.192 - 191.233.203.207 | 
| Centro de Canadá | 13.71.184.150, 13.71.186.1, 40.85.250.135, 40.85.250.212, 40.85.252.47, 52.233.29.92, 52.228.39.241, 52.228.39.244 | 13.71.170.208 - 13.71.170.223, 13.71.170.224 - 13.71.170.239, 52.228.33.76, 52.228.34.13, 52.228.42.205, 52.233.26.83, 52.233.31.197, 52.237.24.126 |
| Este de Canadá | 40.86.203.228, 40.86.216.241, 40.86.217.241, 40.86.226.149, 40.86.228.93, 52.229.120.45, 52.229.126.25, 52.232.128.155 | 40.69.106.240 - 40.69.106.255, 52.229.120.52, 52.229.120.131, 52.229.120.178, 52.229.123.98, 52.229.126.202, 52.242.35.152 |
| India Central | 52.172.154.168, 52.172.185.79, 52.172.186.159, 104.211.74.145, 104.211.90.162, 104.211.90.169, 104.211.101.108, 104.211.102.62 | 52.172.211.12, 104.211.81.192 - 104.211.81.207, 104.211.98.164 |
| Centro de EE. UU. | 13.67.236.125, 23.100.82.16, 23.100.86.139, 23.100.87.24, 23.100.87.56, 40.113.218.230, 40.122.170.198, 104.208.25.27 | 13.89.171.80 - 13.89.171.95, 40.122.49.51, 52.173.245.164 |
| Asia oriental | 13.75.94.173, 40.83.73.39, 40.83.75.165, 40.83.77.208, 40.83.100.69, 40.83.127.19, 52.175.33.254, 65.52.175.34 | 13.75.36.64 - 13.75.36.79, 23.99.116.181, 52.175.23.169 |
| Este de EE. UU | 13.92.98.111, 23.100.29.190, 23.101.132.208, 23.101.136.201, 23.101.139.153, 40.114.82.191, 40.121.91.41, 104.45.153.81 | 40.71.11.80 - 40.71.11.95, 40.71.249.205, 191.237.41.52 |
| Este de EE. UU. 2 | 40.70.26.154, 40.70.27.236, 40.70.29.214, 40.70.131.151, 40.84.30.147, 104.208.140.40, 104.208.155.200, 104.208.158.174 | 40.70.146.208 - 40.70.146.223, 52.232.188.154, 104.208.233.100 |
| Este de Japón | 13.71.158.3, 13.71.158.120, 13.73.4.207, 13.78.18.168, 13.78.20.232, 13.78.21.155, 13.78.35.229, 13.78.42.223 | 13.71.153.19, 13.78.108.0 - 13.78.108.15, 40.115.186.96 |
| Oeste de Japón | 40.74.64.207, 40.74.68.85, 40.74.74.21, 40.74.76.213, 40.74.77.205, 40.74.140.4, 104.214.137.243, 138.91.26.45 | 40.74.100.224 - 40.74.100.239, 40.74.130.77, 104.215.61.248 |
| Centro-Norte de EE. UU | 52.162.208.216, 52.162.213.231, 65.52.8.225, 65.52.9.96, 65.52.10.183, 157.55.210.61, 157.55.212.238, 168.62.248.37 | 52.162.107.160 - 52.162.107.175, 52.162.242.161, 65.52.218.230 |
| Europa del Norte | 40.112.92.104, 40.112.95.216, 40.113.1.181, 40.113.3.202, 40.113.4.18, 40.113.12.95, 52.178.165.215, 52.178.166.21 | 13.69.227.208 - 13.69.227.223, 52.178.150.68, 104.45.93.9 |
| Centro-Sur de EE. UU | 13.65.82.17, 13.66.52.232, 23.100.124.84, 23.100.127.172, 23.101.183.225, 70.37.54.122, 70.37.50.6, 104.210.144.48 | 13.65.86.57, 104.214.19.48 - 104.214.19.63, 104.214.70.191 |
| Sur de la India | 52.172.50.24, 52.172.52.0, 52.172.55.231, 104.211.227.229, 104.211.229.115, 104.211.230.126, 104.211.230.129, 104.211.231.39 | 13.71.125.22, 40.78.194.240 - 40.78.194.255, 104.211.227.225 |
| Sudeste asiático | 13.67.91.135, 13.67.107.128, 13.67.110.109, 13.76.4.194, 13.76.5.96, 13.76.133.155, 52.163.228.93, 52.163.230.166 | 13.67.8.240 - 13.67.8.255, 13.76.231.68, 52.187.68.19 |
| Centro occidental de EE.UU. | 13.78.129.20, 13.78.137.179, 13.78.141.75, 13.78.148.140, 13.78.151.161, 52.161.18.218, 52.161.9.108, 52.161.27.190 | 13.71.195.32 - 13.71.195.47, 52.161.24.128, 52.161.26.212, 52.161.27.108, 52.161.29.35, 52.161.30.5, 52.161.102.22 |
| Europa occidental | 13.95.147.65, 23.97.210.126, 23.97.211.179, 23.97.218.130, 40.68.209.23, 40.68.222.65, 51.144.182.201, 104.45.9.52 | 13.69.64.208 - 13.69.64.223, 40.115.50.13, 52.174.88.118 |
| Oeste de la India | 104.211.154.7, 104.211.154.59, 104.211.156.153, 104.211.158.123, 104.211.158.127, 104.211.162.205, 104.211.164.80, 104.211.164.136 | 104.211.146.224 - 104.211.146.239, 104.211.161.203, 104.211.189.218 |
| Oeste de EE. UU. | 40.83.164.80, 40.118.244.241, 40.118.241.243, 52.160.92.112, 104.42.38.32, 104.42.49.145, 157.56.162.53, 157.56.167.147 |40.112.243.160 - 40.112.243.175, 104.40.51.248, 104.42.122.49 |
| Oeste de EE. UU. 2 | 13.66.201.169, 13.66.210.167, 13.66.246.219, 13.77.149.159, 52.175.198.132, 52.183.29.132, 52.183.30.169 | 13.66.140.128 - 13.66.140.143, 13.66.218.78, 13.66.219.14, 13.66.220.135, 13.66.221.19, 13.66.225.219, 52.183.78.157 |
| Sur de Reino Unido 2 | 51.140.28.225, 51.140.73.85, 51.140.74.14, 51.140.78.44, 51.140.137.190, 51.140.142.28, 51.140.153.135, 51.140.158.24 | 51.140.80.51, 51.140.148.0 - 51.140.148.15 |
| Oeste de Reino Unido | 51.141.45.238, 51.141.47.136, 51.141.54.185, 51.141.112.112, 51.141.113.36, 51.141.114.77, 51.141.118.119, 51.141.119.63 | 51.140.211.0 - 51.140.211.15, 51.141.47.105 |
||||

## <a name="next-steps"></a>Pasos siguientes  

* Aprenda a [crear su primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md)  
* Obtenga información acerca de [ejemplos y escenarios comunes](../logic-apps/logic-apps-examples-and-scenarios.md)
