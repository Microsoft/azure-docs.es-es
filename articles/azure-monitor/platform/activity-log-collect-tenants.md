---
title: Registros de actividad de Azure entre inquilinos en Azure Monitor
description: Use Event Hubs y Logic Apps para recopilar datos del registro de actividad de Azure y enviarlos a un área de trabajo de Log Analytics en Azure Monitor en un inquilino diferente.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/06/2019
ms.openlocfilehash: 7718bd5cbc3c3fc3c9632818f769c05cd1617361
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321877"
---
# <a name="collect-azure-activity-logs-into-azure-monitor-across-azure-active-directory-tenants-legacy"></a>Recopilación de registros de actividad de Azure en Azure Monitor en inquilinos de Azure Active Directory (heredado)

> [!NOTE]
> En este artículo se describe el método heredado para configurar el registro de actividad de Azure en los inquilinos de Azure para que se recopilen en un área de trabajo de Log Analytics.  Ahora puede recopilar el Registro de actividad en un área de trabajo Log Analytics mediante una configuración de diagnóstico similar a la de la recopilación de registros de recursos. Consulte [Collect and analyze Azure activity logs in Log Analytics workspace in Azure Monitor](./activity-log.md) (Recopilación y análisis de registros de actividad en un área de trabajo de Log Analytics en Azure Monitor).


En este artículo se explica un método para recopilar registros de actividad de Azure en un área de trabajo de Log Analytics en Azure Monitor mediante el conector del recopilador de datos de Azure Log Analytics para Logic Apps. Siga el proceso de este artículo si necesita enviar registros a un área de trabajo en un inquilino distinto de Azure Active Directory. Por ejemplo, si es un proveedor de servicios administrados, puede que desee recopilar registros de actividad de la suscripción de un cliente y almacenarlos en un área de trabajo de Log Analytics en su propia suscripción.

Si el área de trabajo de Log Analytics está en la misma suscripción de Azure, o bien en una suscripción diferente, pero en la misma instancia de Azure Active Directory, siga los pasos de [Recopilar y analizar los registros de actividad de Azure en el área de trabajo de Log Analytics en Azure Monitor](./activity-log.md) para recopilar registros de actividad de Azure.

## <a name="overview"></a>Información general

La estrategia utilizada en este escenario consiste en que el registro de actividad de Azure envíe eventos a un [centro de eventos](../../event-hubs/event-hubs-about.md) en el que una [aplicación lógica](../../logic-apps/logic-apps-overview.md) los remita a su área de trabajo de Log Analytics. 

![imagen del flujo de datos del registro de actividad hacia el área de trabajo de Log Analytics](media/collect-activity-logs-subscriptions/data-flow-overview.png)

Las ventajas de este enfoque incluyen:
- Baja latencia, ya que el registro de actividad de Azure se inserta en el centro de eventos.  Después se activa la aplicación lógica, que envía los datos al área de trabajo. 
- Se requiere un código mínimo y no hay que implementar ninguna infraestructura de servidor.

En este artículo se explica cómo:
1. Cree un centro de eventos. 
2. Exportar registros de actividad a un centro de eventos mediante el perfil de exportación del registro de actividad de Azure.
3. Crear una aplicación lógica que lea del centro de eventos y envíe los eventos al área de trabajo de Log Analytics.

## <a name="requirements"></a>Requisitos
A continuación, se especifican los requisitos para los recursos de Azure utilizados en este escenario.

- El espacio de nombres del centro de eventos no debe estar en la misma suscripción que la suscripción que emite los registros. El usuario que establece la configuración debe disponer de los permisos de acceso apropiados a ambas suscripciones. Si tiene varias suscripciones en la misma instancia de Azure Active Directory, puede enviar los registros de actividad de todas las suscripciones a un único centro de eventos.
- La aplicación lógica puede estar en una suscripción distinta a la del centro de eventos y no necesitar estar en la misma instancia de Azure Active Directory. La aplicación lógica lee del centro de eventos mediante la clave de acceso compartida del centro de eventos.
- El área de trabajo de Log Analytics puede estar en una suscripción diferente y en una instancia de Azure Active Directory distinta de donde se encuentra la aplicación lógica, pero para simplificar, se recomienda que estén en la misma suscripción. La aplicación lógica realiza los envíos al área de trabajo con la clave y el identificador del área de trabajo de Log Analytics.



## <a name="step-1---create-an-event-hub"></a>Paso 1: Cree un centro de eventos

<!-- Follow the steps in [how to create an Event Hubs namespace and Event Hub](../../event-hubs/event-hubs-create.md) to create your event hub. -->

1. En Azure Portal, seleccione **Crear un recurso** > **Internet de las cosas** > **Event Hubs**.

   ![Nuevo centro de eventos de Marketplace](media/collect-activity-logs-subscriptions/marketplace-new-event-hub.png)

3. En **Crear espacio de nombres**, escriba un nuevo espacio de nombres o seleccione alguno existente. El sistema realiza la comprobación automáticamente para ver si el nombre está disponible.

   ![imagen del cuadro de diálogo para crear un centro de eventos](media/collect-activity-logs-subscriptions/create-event-hub1.png)

4. Elija el plan de tarifa (Básico o Estándar), una suscripción de Azure, un grupo de recursos y la ubicación para el nuevo recurso.  Haga clic en **Crear** para crear el espacio de nombres. Puede que tenga que esperar unos minutos hasta que el sistema aprovisione totalmente los recursos.
6. Haga clic en el espacio de nombres que acaba de crear en la lista.
7. Seleccione **Directivas de acceso compartido** y después haga clic en **RootManageSharedAccessKey**.

   ![imagen de las directivas de acceso compartido al centro de eventos](media/collect-activity-logs-subscriptions/create-event-hub7.png)
   
8. Haga clic en el botón Copiar para copiar la cadena de conexión **RootManageSharedAccessKey** en el Portapapeles. 

   ![imagen de la clave de acceso compartido al centro de eventos](media/collect-activity-logs-subscriptions/create-event-hub8.png)

9. En una ubicación temporal, como el Bloc de notas, guarde una copia del nombre del centro de eventos y de la cadena de conexión del centro de eventos principal o secundaria. La aplicación lógica requiere estos valores.  Para la cadena de conexión del centro de eventos, puede usar la cadena de conexión **RootManageSharedAccessKey** o crear una distinta.  La cadena de conexión que use debe empezar por `Endpoint=sb://` y ser apta para una directiva que tenga la directiva de acceso **Administrar**.


## <a name="step-2---export-activity-logs-to-event-hub"></a>Paso 2: Exporte registros de actividad al centro de eventos

Para habilitar el streaming del registro de actividad, debe elegir un espacio de nombres del centro de eventos y una directiva de acceso compartido para dicho espacio. Se crea un centro de eventos en dicho espacio de nombres cuando se produce el primer evento nuevo del registro de actividad. 

Puede usar un espacio de nombres del centro de eventos que no esté en la misma suscripción que la suscripción que emite los registros; en cambio, las suscripciones deben estar en la misma instancia de Azure Active Directory. El usuario que configura los ajustes debe tener el acceso de RBAC adecuado a ambas suscripciones. 

1. En Azure Portal, seleccione **Supervisar** > **Registro de actividad**.
3. Haga clic en el botón **Exportar** en la parte superior de la página.

   ![imagen de la navegación por Azure Monitor](media/collect-activity-logs-subscriptions/activity-log-blade.png)

4. Seleccione la **suscripción** desde la que desea realizar la exportación y después haga clic en **Seleccionar todo** en la lista desplegable **Regiones** para seleccionar eventos para recursos en todas las regiones. Haga clic en la casilla **Exportar en un Centro de eventos**.
7. Haga clic en **Espacio de nombres de Service Bus** y después seleccione la **suscripción** con el centro de eventos, el **espacio de nombres del centro de eventos** y un **nombre de la directiva del centro de eventos**.

    ![imagen de la página de exportación al centro de eventos](media/collect-activity-logs-subscriptions/export-activity-log2.png)

11. Haga clic en **Aceptar** y en **Guardar** para guardar la configuración. La configuración se aplica inmediatamente a la suscripción.

<!-- Follow the steps in [stream the Azure Activity Log to Event Hubs](./activity-log.md#legacy-collection-methods) to configure a log profile that writes activity logs to an event hub. -->

## <a name="step-3---create-logic-app"></a>Paso 3: Cree una aplicación lógica

Una vez que los registros de actividad escriban en el centro de eventos, cree una aplicación lógica para que recopile los registros del centro de eventos y los escriba en el área de trabajo de Log Analytics.

La aplicación lógica incluye lo siguiente:
- Un desencadenador del [conector del centro de eventos](/connectors/eventhubs/) para leer en el centro de eventos.
- Una [acción de análisis JSON](../../logic-apps/logic-apps-content-type.md) para extraer los eventos JSON.
- Una [acción de redacción](../../logic-apps/logic-apps-workflow-actions-triggers.md#compose-action) para convertir el evento JSON en un objeto.
- Un [conector de envío de datos a Log Analytics](/connectors/azureloganalyticsdatacollector/) para publicar los datos en el área de trabajo de Log Analytics.

   ![imagen de la adición del desencadenador del centro de eventos a Logic Apps](media/collect-activity-logs-subscriptions/log-analytics-logic-apps-activity-log-overview.png)

### <a name="logic-app-requirements"></a>Requisitos de la aplicación lógica
Antes de crear la aplicación lógica, asegúrese de que dispone de la información siguiente de los pasos anteriores:
- Nombre del centro de eventos
- Cadena de conexión del centro de eventos (primaria o secundaria) para el espacio de nombres del centro de eventos
- Identificador del área de trabajo de Log Analytics
- Clave compartida de Log Analytics

Para obtener el nombre y la cadena de conexión del centro de eventos, siga los pasos descritos en [Comprobar los permisos del espacio de nombres de Event Hubs y buscar la cadena de conexión](../../connectors/connectors-create-api-azure-event-hubs.md#permissions-connection-string).


### <a name="create-a-new-blank-logic-app"></a>Creación de una aplicación lógica en blanco

1. En Azure Portal, elija **Crear un recurso** > **Integración empresarial** > **Aplicación lógica**.

    ![Nueva aplicación de lógica de Marketplace](media/collect-activity-logs-subscriptions/marketplace-new-logic-app.png)

2. Escriba la configuración en la tabla siguiente.

    ![Creación de la aplicación lógica](media/collect-activity-logs-subscriptions/create-logic-app.png)

   |Configuración | Descripción  |
   |:---|:---|
   | Nombre           | Nombre único para la aplicación lógica. |
   | Subscription   | Seleccione la suscripción de Azure que contendrá la aplicación lógica. |
   | Grupo de recursos | Seleccione un grupo de recursos de Azure existente o cree uno para la aplicación lógica. |
   | Location       | Seleccione la región del centro de datos para implementar la aplicación lógica. |
   | Log Analytics  | Seleccione si desea registrar el estado de cada ejecución de la aplicación lógica en el área de trabajo de Log Analytics.  |

    
3. Seleccione **Crear**. Cuando aparezca la notificación **Implementación correcta**, haga clic en **Ir al recurso** para abrir la aplicación lógica.

4. En **plantillas**, elija **Aplicación lógica en blanco**. 

El diseñador de aplicaciones lógicas ahora muestra los conectores disponibles y sus desencadenadores, que utilizará para iniciar el flujo de trabajo de la aplicación lógica.

<!-- Learn [how to create a logic app](../../logic-apps/quickstart-create-first-logic-app-workflow.md). -->

### <a name="add-event-hub-trigger"></a>Adición del desencadenador del centro de eventos

1. En el cuadro de búsqueda del diseñador de aplicaciones lógicas, escriba *centros de eventos* como filtro. Seleccione el desencadenador **Event Hubs: Cuando los eventos estén disponibles en el centro de eventos**.

   ![imagen de la adición del desencadenador del centro de eventos a Logic Apps](media/collect-activity-logs-subscriptions/logic-apps-event-hub-add-trigger.png)

2. Cuando se le soliciten las credenciales, conéctese al espacio de nombres de Event Hubs. Escriba un nombre para la conexión y después escriba la cadena de conexión que copió.  Seleccione **Crear**.

   ![imagen de la adición de la conexión del centro de eventos a Logic Apps](media/collect-activity-logs-subscriptions/logic-apps-event-hub-add-connection.png)

3. Una vez creada la conexión, edite la configuración del desencadenador. Empiece por seleccionar **insights-operational-logs** en el menú desplegable **Nombre del centro de eventos**.

   ![Cuadro de diálogo Cuando los eventos estén disponibles](media/collect-activity-logs-subscriptions/logic-apps-event-hub-read-events.png)

5. Expanda **Mostrar opciones avanzadas** y cambie **Tipo de contenido** a *application/json*.

   ![Cuadro de diálogo de configuración del centro de eventos](media/collect-activity-logs-subscriptions/logic-apps-event-hub-configuration.png)

### <a name="add-parse-json-action"></a>Adición de la acción Análisis del archivo JSON

La salida del centro de eventos contiene una carga JSON con una matriz de registros. La acción [Análisis del archivo JSON](../../logic-apps/logic-apps-content-type.md) se usa para extraer la matriz de registros para enviarlos al área de trabajo de Log Analytics.

1. Haga clic en **Nuevo paso** > **Agregar una acción**.
2. En el cuadro de búsqueda, escriba *análisis del archivo json* como filtro. Seleccione la acción **Operaciones de datos: análisis del archivo JSON**.

   ![Adición de una acción Análisis del archivo JSON en Logic Apps](media/collect-activity-logs-subscriptions/logic-apps-add-parse-json-action.png)

3. Haga clic en el campo **Contenido** y después seleccione *Cuerpo*.

4. Copie y pegue el siguiente esquema en el campo **Esquema**.  Este esquema coincide con el resultado de la acción del centro de eventos.  

   ![Cuadro de diálogo de análisis del archivo JSON](media/collect-activity-logs-subscriptions/logic-apps-parse-json-configuration.png)

``` json-schema
{
    "properties": {
        "body": {
            "properties": {
                "ContentData": {
                    "type": "string"
                },
                "Properties": {
                    "properties": {
                        "ProfileName": {
                            "type": "string"
                        },
                        "x-opt-enqueued-time": {
                            "type": "string"
                        },
                        "x-opt-offset": {
                            "type": "string"
                        },
                        "x-opt-sequence-number": {
                            "type": "number"
                        }
                    },
                    "type": "object"
                },
                "SystemProperties": {
                    "properties": {
                        "EnqueuedTimeUtc": {
                            "type": "string"
                        },
                        "Offset": {
                            "type": "string"
                        },
                        "PartitionKey": {},
                        "SequenceNumber": {
                            "type": "number"
                        }
                    },
                    "type": "object"
                }
            },
            "type": "object"
        },
        "headers": {
            "properties": {
                "Cache-Control": {
                    "type": "string"
                },
                "Content-Length": {
                    "type": "string"
                },
                "Content-Type": {
                    "type": "string"
                },
                "Date": {
                    "type": "string"
                },
                "Expires": {
                    "type": "string"
                },
                "Location": {
                    "type": "string"
                },
                "Pragma": {
                    "type": "string"
                },
                "Retry-After": {
                    "type": "string"
                },
                "Timing-Allow-Origin": {
                    "type": "string"
                },
                "Transfer-Encoding": {
                    "type": "string"
                },
                "Vary": {
                    "type": "string"
                },
                "X-AspNet-Version": {
                    "type": "string"
                },
                "X-Powered-By": {
                    "type": "string"
                },
                "x-ms-request-id": {
                    "type": "string"
                }
            },
            "type": "object"
        }
    },
    "type": "object"
}
```

>[!TIP]
> Puede obtener una carga de ejemplo si haga clic en **Ejecutar** y examina la **Salida sin procesar** del centro de eventos.  Después puede usar esta salida con **Usar una carga de ejemplo para generar el esquema** en la actividad **Análisis del archivo JSON** para generar el esquema.

### <a name="add-compose-action"></a>Adición de la acción de redacción
La acción [Redactar](../../logic-apps/logic-apps-workflow-actions-triggers.md#compose-action) usa la salida JSON y crea un objeto para que la acción de Log Analytics pueda usarlo.

1. Haga clic en **Nuevo paso** > **Agregar una acción**.
2. Escriba *redactar* como filtro y después seleccione la acción **Operaciones de datos: redactar**.

    ![Adición de la acción de redacción](media/collect-activity-logs-subscriptions/logic-apps-add-compose-action.png)

3. Haga clic en el campo **Entradas** y seleccione **Cuerpo** en la actividad **Análisis del archivo JSON**.


### <a name="add-log-analytics-send-data-action"></a>Adición de la acción de envío de datos a Log Analytics
El [recopilador de datos de Azure Log Analytics](/connectors/azureloganalyticsdatacollector/) usa el objeto de la acción de redacción y lo envía al área de trabajo de Log Analytics.

1. Haga clic en **Nuevo paso** > **Agregar una acción**.
2. Escriba *log analytics* como filtro y después seleccione la acción **Recopilador de datos de Azure Log Analytics: Enviar datos**.

   ![Adición de la acción de envío de datos a Log Analytics en Logic Apps](media/collect-activity-logs-subscriptions/logic-apps-send-data-to-log-analytics-connector.png)

3. Escriba un nombre para la conexión y pegue el **Id. de área de trabajo** y la **clave de área de trabajo** para el área de trabajo de Log Analytics.  Haga clic en **Crear**.

   ![Adición de la conexión a Log Analytics en Logic Apps](media/collect-activity-logs-subscriptions/logic-apps-log-analytics-add-connection.png)

4. Una vez creada la conexión, edite la configuración en la tabla siguiente. 

    ![Configuración de la acción de envío de datos](media/collect-activity-logs-subscriptions/logic-apps-send-data-to-log-analytics-configuration.png)

   |Configuración        | Value           | Descripción  |
   |---------------|---------------------------|--------------|
   |Cuerpo de la solicitud JSON  | **Salida** de la acción **Redactar** | Recupera los registros del cuerpo de la acción Redactar. |
   | Nombre de registro personalizado | AzureActivity | Nombre de la tabla de registro personalizada para crearla en el área de trabajo de Log Analytics para que almacene los datos importados. |
   | Time-generated-field | time | No seleccione el campo JSON para **time**, solo debe escribir la palabra time. Si selecciona el campo JSON, el diseñador coloca la acción **Enviar datos** en un bucle *For Each*, que no es lo que desea. |




10. Haga clic en **Guardar** para guardar los cambios realizados en la aplicación lógica.

## <a name="step-4---test-and-troubleshoot-the-logic-app"></a>Paso 4: Pruebe la aplicación lógica y solucione los problemas
Una vez completado el flujo de trabajo, puede probarlo en el diseñador para verificar que funciona sin errores.

En el diseñador de Logic Apps, haga clic en **Ejecutar** para probar la aplicación lógica. Cada paso de la aplicación lógica muestra un icono de estado, con una marca de verificación blanca en un círculo verde que indica que el paso se ha completado correctamente.

   ![Prueba de la aplicación lógica](media/collect-activity-logs-subscriptions/test-logic-app.png)

Para ver información detallada sobre cada paso, haga clic en el nombre del paso para expandirlo. Haga clic en **Mostrar entradas sin procesar** y **Mostrar salidas sin procesar** para obtener más información sobre los datos recibidos y enviados en cada paso.

## <a name="step-5---view-azure-activity-log-in-log-analytics"></a>Paso 5: Ver el registro de actividad de Azure en Log Analytics
El último paso consiste en comprobar el área de trabajo de Log Analytics para asegurarse de que los datos se recopilan según lo previsto.

1. En Azure Portal, haga clic en **Todos los servicios**, en la esquina superior izquierda. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Log Analytics**.
2. En la lista de áreas de trabajo de Log Analytics, seleccione su área de trabajo.
3.  Haga clic en el icono **Búsqueda de registros**. A continuación, en el campo de consulta del panel Búsqueda de registros, escriba `AzureActivity_CL` y presione Entrar o haga clic en el botón de búsqueda situado a la derecha del campo de consulta. Si no asignó el nombre *AzureActivity* al registro personalizado, escriba el que eligió y anexe `_CL`.

>[!NOTE]
> La primera vez que se envía un registro personalizado nuevo al área de trabajo de Log Analytics, puede transcurrir hasta una hora para poder buscar dicho registro.

>[!NOTE]
> Los registros de actividad se escriben en una tabla personalizada y no aparecen en la [solución de registro de actividad](./activity-log.md).


![Prueba de la aplicación lógica](media/collect-activity-logs-subscriptions/log-analytics-results.png)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado una aplicación lógica para leer los registros de actividad de Azure de un centro de eventos y enviarlos al área de trabajo de Log Analytics para su análisis. Para más información sobre cómo visualizar los datos en un área de trabajo, incluida la creación de paneles, revise el tutorial sobre la visualización de datos.

> [!div class="nextstepaction"]
> [Tutorial sobre la visualización de datos de búsqueda de registros](../learn/tutorial-logs-dashboards.md)

