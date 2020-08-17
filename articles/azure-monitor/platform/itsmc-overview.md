---
title: IT Service Management Connector en Azure Log Analytics | Microsoft Docs
description: En este artículo se proporciona información general sobre el Conector de Administración de servicios de TI (ITSMC) e información sobre cómo usar esta solución para supervisar y administrar de forma centralizada los elementos de trabajo de ITSM en Azure Log Analytics y resolver rápidamente cualquier problema.
ms.subservice: logs
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 05/24/2018
ms.openlocfilehash: 989399bcc86547b914deb442fa9793f6d79d0606
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325328"
---
# <a name="connect-azure-to-itsm-tools-using-it-service-management-connector"></a>Conectar Azure a las herramientas de ITSM mediante el Conector de Administración de servicios de TI

![Símbolo de IT Service Management Connector](media/itsmc-overview/itsmc-symbol.png)

El Conector de Administración de servicios de TI (ITSMC) le permite conectar Azure y un producto o servicio de Administración de servicios de TI (ITSM) compatibles.

Servicios de Azure como Log Analytics y Azure Monitor proporcionan herramientas para detectar, analizar y solucionar problemas relacionados con los recursos de Azure y de otros proveedores. Sin embargo, los elementos de trabajo relacionados con un problema suelen residir en un producto o servicio de ITSM. El conector de ITSM proporciona una conexión bidireccional entre las herramientas de Azure e ITSM para ayudarle a resolver problemas con mayor rapidez.

ITSMC admite conexiones con las siguientes herramientas ITSM:

-   ServiceNow
-   System Center Service Manager
-   Provance
-   Cherwell

Con ITSMC, puede:

-  Crear elementos de trabajo en la herramienta de ITSM, en función de las alertas de Azure (alertas de métricas, de Activity Log y de Log Analytics).
-  Sincronizar de forma opcional los datos de incidentes y solicitudes de cambio de la herramienta ITSM con un área de trabajo de Azure Log Analytics.

Más información sobre los [términos legales y la directiva de privacidad](https://go.microsoft.com/fwLink/?LinkID=522330&clcid=0x9).

Puede empezar a usar el conector de ITSM siguiendo estos pasos:

1.  [Agregue la solución del conector de ITSM](#adding-the-it-service-management-connector-solution).
2.  Cree una conexión de ITSM
3.  [Use la conexión](#using-the-solution).


##  <a name="adding-the-it-service-management-connector-solution"></a>Agregar la solución IT Service Management Connector

Para poder crear una conexión, debe agregar la solución del conector de ITSM.

1. En Azure Portal, haga clic en el icono **+ Nuevo**.

   ![Nuevo recurso de Azure](media/itsmc-overview/azure-add-new-resource.png)

2. Busque el **Conector de Administración de servicios de TI** en Marketplace y haga clic en **Crear**.

   ![Agregar la solución ITSMC](media/itsmc-overview/add-itsmc-solution.png)

3. En la sección **Área de trabajo de OMS**, seleccione el área de trabajo de Azure Log Analytics donde quiera instalar la solución.
   >[!NOTE]
   > * Como parte de la transición continuada de Microsoft Operations Management Suite (OMS) a Azure Monitor, las áreas de trabajo de OMS ahora se conocen como áreas de trabajo de Log Analytics.
   > * Solo se puede instalar el conector ITSM en áreas de trabajo de Log Analytics en las siguientes regiones: Este de EE. UU., Oeste de EE. UU.2, Centro Sur de EE. UU., Centro-oeste de EE. UU., Fairfax, Centro de Canadá, Oeste de Europa, Sur de Reino Unido, Sudeste de Asia, Este de Japón, Centro de la India y Sudeste de Australia.

4. En la sección **Configuración del área de trabajo de OMS**, seleccione el grupo de recursos donde quiera crear el recurso de la solución.

   ![Área de trabajo de ITSMC](media/itsmc-overview/itsmc-solution-workspace.png)
   >[!NOTE]
   >Como parte de la transición continuada de Microsoft Operations Management Suite (OMS) a Azure Monitor, las áreas de trabajo de OMS ahora se conocen como áreas de trabajo de Log Analytics.

5. Haga clic en **Crear**.

Cuando se implemente el recurso de la solución, aparecerá una notificación en la parte superior derecha de la ventana.


## <a name="creating-an-itsm--connection"></a>Crear una conexión de ITSM

Después de instalar la solución, puede crear una conexión.

Para crear una conexión, debe preparar la herramienta de ITSM para permitir la conexión a la solución del conector de ITSM.  

Según el producto de ITSM al que se conecte, siga estos pasos:

- [System Center Service Manager (SCSM)](./itsmc-connections.md#connect-system-center-service-manager-to-it-service-management-connector-in-azure)
- [ServiceNow](./itsmc-connections.md#connect-servicenow-to-it-service-management-connector-in-azure)
- [Provance](./itsmc-connections.md#connect-provance-to-it-service-management-connector-in-azure)  
- [Cherwell](./itsmc-connections.md#connect-cherwell-to-it-service-management-connector-in-azure)

Una vez que haya preparado sus herramientas ITSM, siga estos pasos para crear una conexión:

1. Vaya a **Todos los recursos** y busque **ServiceDesk(YourWorkspaceName)** .
2. En la opción **ORÍGENES DE DATOS DEL ÁREA DE TRABAJO** del panel izquierdo, haga clic en **Conexiones de ITSM** .
   ![Conexiones de ITSM](media/itsmc-overview/itsm-connections.png)

   En esta página se muestra la lista de conexiones.
3. Haga clic en **Agregar conexión**.

   ![Agregar una conexión de ITSM](media/itsmc-overview/add-new-itsm-connection.png)

4. Especifique la configuración de conexión tal como se describe en el artículo [Configuring the ITSMC connection with your ITSM products/services](./itsmc-connections.md) (Configurar la conexión ITSMC con los productos o servicios de ITSM).

   > [!NOTE]
   >
   > De forma predeterminada, ITSMC actualiza los datos de configuración de la conexión una vez cada 24 horas. Para actualizar los datos de la conexión al instante para cualquier modificación o actualizaciones de plantilla que se realicen, haga clic en el botón **Actualizar** que se muestra en la hoja de la conexión.

   ![Actualizar la conexión](media/itsmc-overview/itsmc-connections-refresh.png)


## <a name="using-the-solution"></a>Uso de la solución
   Al usar la solución del Conector ITSM, puede crear elementos de trabajo de alertas de Azure, alertas de Log Analytics y entradas de registros de Log Analytics.

## <a name="create-itsm-work-items-from-azure-alerts"></a>Creación de elementos de trabajo de ITSM a partir de alertas de Azure

Una vez que haya creado la conexión a ITSM, puede crear elementos de trabajo en la herramienta de ITSM según las alertas de Azure, mediante la **acción de ITSM** en **Grupos de acciones**.

Los grupos de acciones proporcionan una manera modular y reutilizable de desencadenar acciones para las alertas de Azure. Puede usar los grupos de acciones con alertas de métricas, de Activity Log y de Azure Log Analytics en Azure Portal.

Utilice el siguiente procedimiento:

1. En Azure Portal, haga clic en **Monitor**
2. En el panel izquierdo, haga clic en **Grupos de acciones**. Aparece la ventana **Agregar grupo de acciones**.

    ![Grupos de acciones](media/itsmc-overview/action-groups.png)

3. Complete los campos **Name** (Nombre) y **ShortName** (Nombre corto) para el grupo de acciones. Seleccione el **grupo de recursos** y la **suscripción** donde quiere crear el grupo de acciones.

    ![Detalle de los grupos de acciones](media/itsmc-overview/action-groups-details.png)

4. En la lista de acciones, seleccione **ITSM** en la lista desplegable **Tipo de acción**. Proporcione un **nombre** para la acción y haga clic en **Editar detalles**.
5. En **Subscription** (Suscripción), indique dónde se encuentra el área de trabajo de Log Analytics. Seleccione el nombre de la **conexión** (el nombre del conector ITSM) seguido del nombre del área de trabajo. Por ejemplo, "MiITSMMConnector(MiAreaDeTrabajo)".

    ![Detalles de acción de ITSM](media/itsmc-overview/itsm-action-details.png)

6. Seleccione el tipo **Elemento de trabajo** en el menú desplegable.
   Elija usar una plantilla existente o rellene los campos requeridos para el producto ITSM.
7. Haga clic en **OK**.

Al crear o editar una regla de alerta de Azure, use un grupo de acciones, que tiene una acción de ITSM. Cuando se desencadena la alerta, se crea o actualiza un elemento de trabajo en la herramienta ITSM.

> [!NOTE]
>
> Para obtener información sobre los precios de las acciones de ITSM, consulte la [página de precios](https://azure.microsoft.com/pricing/details/monitor/) de los grupos de acciones.


## <a name="visualize-and-analyze-the-incident-and-change-request-data"></a>Visualizar y analizar los datos de incidentes y solicitudes de cambios

Si se tienen en cuenta las opciones de configuración de una conexión, el Conector ITSM puede sincronizar hasta 120 días de datos referentes a incidentes y a solicitudes cambios. El esquema de registros de estos datos se proporciona en la [próxima sección](#additional-information).

Gracias al panel del Conector ITSM en la solución, se pueden visualizar los datos de incidentes y solicitudes de cambios.

![Pantalla de Log Analytics](media/itsmc-overview/itsmc-overview-sample-log-analytics.png)

Asimismo, el panel también proporciona información acerca del estado del conector que se puede usar como punto de partida para analizar cualquier problema con las conexiones.

También puede visualizar los incidentes que se sincronizan con los equipos afectados dentro de la solución Service Map.

Mapa de servicio detecta automáticamente los componentes de la aplicación en sistemas Windows y Linux y asigna la comunicación entre servicios. Permite ver los servidores a medida que piensa en ellos, como los sistemas interconectados que ofrecen servicios críticos. Mapa de servicio muestra las conexiones entre servidores, procesos y puertos en cualquier arquitectura conectada TCP sin una configuración necesaria que sea distinta a la instalación de un agente. [Más información](../insights/service-map.md).

Si usa la solución Service Map, puede ver los elementos de la consola de servicio creados en la solución ITSM tal como se muestra en el ejemplo siguiente:

![Pantalla de Log Analytics](media/itsmc-overview/itsmc-overview-integrated-solutions.png)

Más información: [Mapa de servicio](../insights/service-map.md)


## <a name="additional-information"></a>Información adicional

### <a name="data-synced-from-itsm-product"></a>Datos que se sincronizan del producto ITSM
Los incidentes y las solicitudes de cambio se sincronizan desde el producto ITSM con el área de trabajo de Log Analytics según la configuración de la conexión.

En la siguiente información se muestran ejemplos de datos recopilados por ITSMC:

> [!NOTE]
>
> En función del tipo de elemento de trabajo que se importa a Log Analytics, **ServiceDesk_CL** contiene los campos siguientes:

**Elemento de trabajo:** **Incidentes**  
ServiceDeskWorkItemType_s="Incidente"

**Fields**

- ServiceDeskConnectionName
- Service Desk ID
- State
- Urgencia
- Impacto
- Priority
- Escalado
- Creado por
- Resuelto por
- Cerrado por
- Source
- Asignado a
- Category
- Título
- Descripción
- Fecha de creación
- Fecha de cierre
- Fecha de resolución
- Fecha de última modificación
- Computer


**Elemento de trabajo:** **Solicitudes de cambio**

ServiceDeskWorkItemType_s="ChangeRequest"

**Fields**
- ServiceDeskConnectionName
- Service Desk ID
- Creado por
- Cerrado por
- Source
- Asignado a
- Título
- Tipo
- Category
- State
- Escalado
- Estado del conflicto
- Urgencia
- Priority
- Riesgo
- Impacto
- Asignado a
- Fecha de creación
- Fecha de cierre
- Fecha de última modificación
- Fecha de solicitud
- Fecha de inicio planeada
- Fecha de finalización planeada
- Fecha de inicio del trabajo
- Fecha de finalización del trabajo
- Descripción
- Computer

## <a name="output-data-for-a-servicenow-incident"></a>Datos de salida para un incidente de ServiceNow

| Campo de Log Analytics | Campo de ServiceNow |
|:--- |:--- |
| ServiceDeskId_s| Number |
| IncidentState_s | State |
| Urgency_s |Urgencia |
| Impact_s |Impacto|
| Priority_s | Priority |
| CreatedBy_s | Abierto por |
| ResolvedBy_s | Resuelto por|
| ClosedBy_s  | Cerrado por |
| Source_s| Tipo de contacto |
| AssignedTo_s | Asignado a  |
| Category_s | Category |
| Title_s|  Descripción breve |
| Description_s|  Notas |
| CreatedDate_t|  Abierto |
| ClosedDate_t| closed|
| ResolvedDate_t|Resuelto|
| Computer  | Elemento de configuración |

## <a name="output-data-for-a-servicenow-change-request"></a>Datos de salida para una solicitud de cambio de ServiceNow

| Log Analytics | Campo de ServiceNow |
|:--- |:--- |
| ServiceDeskId_s| Number |
| CreatedBy_s | Solicitado por |
| ClosedBy_s | Cerrado por |
| AssignedTo_s | Asignado a  |
| Title_s|  Descripción breve |
| Type_s|  Tipo |
| Category_s|  Category |
| CRState_s|  State|
| Urgency_s|  Urgencia |
| Priority_s| Priority|
| Risk_s| Riesgo|
| Impact_s| Impacto|
| RequestedDate_t  | Solicitado por fecha |
| ClosedDate_t | Fecha de cierre |
| PlannedStartDate_t  |     Fecha de inicio planeada |
| PlannedEndDate_t  |   Fecha de finalización planeada |
| WorkStartDate_t  | Fecha de inicio real |
| WorkEndDate_t | Fecha de finalización real|
| Description_s | Descripción |
| Computer  | Elemento de configuración |


## <a name="troubleshoot-itsm-connections"></a>Solución de problemas de conexión de ITSM
1. Si se produce un error en la conexión de la UI del origen conectado y recibe el mensaje **Error al guardar la conexión**, siga estos pasos:
   - En el caso de conexiones de ServiceNow, Cherwell y Provance,  
   - asegúrese de que ha introducido correctamente el nombre de usuario, la contraseña, el Id. de cliente y el secreto de cliente de cada una de las conexiones.  
   - compruebe si dispone de privilegios suficientes en el producto ITSM correspondiente para realizar la conexión.  
   - En el caso de conexiones de Service Manager,  
   - asegúrese de que la aplicación web se implementa correctamente y se crea la conexión híbrida. Para comprobar que la conexión se ha establecido correctamente con la máquina de Service Manager local, visite la dirección URL de la aplicación web como se detalla en la documentación para realizar la [conexión híbrida](./itsmc-connections.md#configure-the-hybrid-connection).  

2. Si los datos de ServiceNow no se sincronizan con Log Analytics, asegúrese de que la instancia de ServiceNow no esté suspendida. En algunas ocasiones, las instancias de desarrollo de ServiceNow se suspenden si están inactivas durante mucho tiempo. En caso contrario, notifique el problema.
3. Si se generan alertas de Log Analytics, pero no se crean elementos de trabajo en el producto de ITSM, no se crean elementos de configuración o no se vinculan a elementos de trabajo, o bien necesita información general, vea:
   -  ITSMC: La solución muestra un resumen de conexiones, elementos de trabajo, equipos, etc. Haga clic en el icono que muestra **Estado del conector**, que le lleva a **Búsqueda de registros** con la consulta correspondiente. Para obtener más información, consulte las entradas del registro con LogType_S como ERROR para obtener más información.
   - Página **Búsqueda de registros**: vea directamente los errores y la información relacionada con la consulta `*`ServiceDeskLog_CL`*`.

## <a name="troubleshoot-service-manager-web-app-deployment"></a>Solucionar problemas de implementación de aplicaciones web de Service Manager
1.  En caso de que se produzcan problemas con la implementación de la aplicación web, asegúrese de tener los permisos suficientes en la suscripción mencionada para crear o implementar recursos.
2.  Si aparece el mensaje de error **Referencia a objeto no establecida como instancia de un objeto"** al ejecutar el [script](itsmc-service-manager-script.md), asegúrese de que especificó valores válidos en la sección **Configuración de usuario**.
3.  Si no puede crear el espacio de nombres de retransmisión de Service Bus, asegúrese de que el proveedor de recursos necesario está registrado en la suscripción. Si no está registrado, cree el espacio de nombres de Bus Relay manualmente desde Azure Portal. También puede crearlo mientras [crea la conexión híbrida](./itsmc-connections.md#configure-the-hybrid-connection) desde Azure Portal.


## <a name="contact-us"></a>Ponerse en contacto con nosotros

Si tiene consultas o comentarios sobre IT Service Management, póngase en contacto con nosotros en [omsitsmfeedback@microsoft.com](mailto:omsitsmfeedback@microsoft.com).

## <a name="next-steps"></a>Pasos siguientes
[Incorporación de productos o servicios de ITSM a IT Service Management Connector](./itsmc-connections.md).

