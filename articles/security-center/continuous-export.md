---
title: Exportar alertas y recomendaciones de Azure Security Center a SIEM | Microsoft Docs
description: En este artículo se explica cómo configurar la exportación continua de alertas y recomendaciones de seguridad a SIEM
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: memildin
ms.openlocfilehash: cf8fdd8d91c035d374277c4752fb761c0c4e72c7
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905621"
---
# <a name="export-security-alerts-and-recommendations"></a>Exportación de alertas y recomendaciones de seguridad

Azure Security Center genera alertas y recomendaciones de seguridad detalladas. Puede verlas en el portal o mediante herramientas de programación. También puede exportar esta información o enviarla a otras herramientas de supervisión de su entorno. 

En este artículo se describe el conjunto de herramientas que le permiten exportar alertas y recomendaciones de forma manual o continua.

Con estas herramientas puede realizar lo siguiente:

* Exportación continua a las áreas de trabajo de Log Analytics
* Exportación continua a Azure Event Hubs (para integraciones con SIEM de terceros)
* Exportar a un archivo CSV (una vez)



## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|Gratuito|
|Roles y permisos necesarios:|**Rol de administrador de seguridad** en el grupo de recursos (o **Propietario**).<br>También debe tener permisos de escritura para el recurso de destino.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![Sí](./media/icons/yes-icon.png) China Gov (en el centro de eventos), otro Gov|
|||



## <a name="set-up-a-continuous-export"></a>Configuración de una exportación continua

Los pasos siguientes son necesarios si va a configurar una exportación continua a las área de trabajo de Log Analytics o a Azure Event Hubs.

1. En la barra lateral de Security Center, seleccione **Precios y configuración**.

1. Seleccione la suscripción específica para la que quiere configurar la exportación de datos.
    
1. En la barra lateral de la página de configuración de esa suscripción, seleccione **Exportación continua**.

    [![Opciones de exportación en Azure Security Center](media/continuous-export/continuous-export-options-page.png)](media/continuous-export/continuous-export-options-page.png#lightbox): aquí se ven las opciones de exportación. Hay una pestaña para cada destino de exportación disponible. 

1. Seleccione el tipo de datos que quiere exportar y elija los filtros que quiera de cada tipo (por ejemplo, exportar solo alertas de gravedad alta).

1. Opcionalmente, si la selección incluye una de estas cuatro recomendaciones, puede incluir los resultados de la evaluación de vulnerabilidades junto con ellas:

    - Se deben corregir las conclusiones de la evaluación de vulnerabilidades de las bases de datos SQL
    - Es necesario corregir las conclusiones de la evaluación de vulnerabilidades de los servidores SQL Server en las máquinas (versión preliminar)
    - Es necesario corregir las vulnerabilidades de las imágenes de Azure Container Registry (con tecnología de Qualys)
    - Es necesario corregir las vulnerabilidades de las máquinas virtuales

    Para incluir los resultados con estas recomendaciones, habilite la opción de **incluir resultados de seguridad**.

    :::image type="content" source="./media/continuous-export/include-security-findings-toggle.png" alt-text="Incluir la alternancia de los resultados de seguridad en la configuración de la exportación continua" :::


1. En el área "Destino de exportación", elija dónde quiere guardar los datos. Los datos se pueden guardar en el destino de una suscripción diferente (por ejemplo, en una instancia central del Centro de eventos o en un área de trabajo central de Log Analytics).

1. Seleccione **Guardar**.


## <a name="set-up-continuous-export-via-the-rest-api"></a>Configuración de la exportación continua mediante API REST

La característica de exportación continua se puede configurar y administrar mediante las [API de automatizaciones](https://docs.microsoft.com/rest/api/securitycenter/automations) de Security Center. Use esta API para crear o actualizar automatizaciones para exportarlas a cualquiera de los siguientes destinos posibles:

- Centro de eventos de Azure
- Área de trabajo de Log Analytics
- Azure Logic Apps 

La API proporciona funcionalidad adicional que no está disponible en Azure Portal, por ejemplo:

* **Mayor volumen**: la API permite crear varias configuraciones de exportación en una suscripción única. La página **Exportación continua** de la interfaz de usuario del portal de Security Center solo admite una configuración de exportación por suscripción.

* **Características adicionales**: la API ofrece parámetros adicionales que no se muestran en la interfaz de usuario. Por ejemplo, puede agregar etiquetas a su recurso de automatización, así como definir su exportación en función de un mayor conjunto de alertas y propiedades recomendadas que las que se ofrecen en la página **Exportación continua** de la interfaz de usuario del portal de Security Center.

* **Ámbito más centrado**: la API proporciona un nivel más detallado del ámbito de sus configuraciones de exportación. Cuando se define una exportación con la API, se puede hacer a nivel de grupo de recursos. Si va a usar la página **Exportación continua** en la interfaz de usuario del portal de Security Center, tiene que definirla en el nivel de suscripción.

    > [!TIP]
    > Si ha configurado la exportación múltiple mediante la API o si ha usado parámetros que son solo de la API, esas características adicionales no se mostrarán en la interfaz de usuario de Security Center. En su lugar, habrá un banner que le informará de que existen otras configuraciones.

En la [documentación de la API REST](https://docs.microsoft.com/rest/api/securitycenter/automations) encontrará más información sobre la API de automatizaciones.



## <a name="configure-siem-integration-via-azure-event-hubs"></a>Configuración de la integración de SIEM mediante Azure Event Hubs

Azure Event Hubs es una excelente solución para consumir mediante programación cualquiera de los datos de streaming. Para las alertas y recomendaciones de Azure Security Center, es la forma preferida de integrarse con un SIEM de terceros.

> [!NOTE]
> El método más efectivo en la mayoría de los casos para transmitir datos de supervisión a herramientas externas, es usar Azure Event Hubs. [En este artículo](https://docs.microsoft.com/azure/azure-monitor/platform/stream-monitoring-data-event-hubs) se proporciona una breve descripción de cómo puede transmitir datos de supervisión desde diferentes orígenes a un centro de eventos y a vínculos de instrucciones detalladas.

> [!NOTE]
> Si exportó anteriormente alertas de Security Center a un SIEM mediante el registro de actividad de Azure, el procedimiento que se indica a continuación reemplaza esa metodología.

Para ver los esquemas de eventos de los tipos de datos exportados, visite el artículo sobre los [esquemas de eventos del centro de eventos](https://aka.ms/ASCAutomationSchemas).


### <a name="to-integrate-with-a-siem"></a>Integración con un SIEM 

Después de configurar la exportación continua de los datos de Security Center seleccionados a Azure Event Hubs, puede configurar el conector adecuado para el SIEM:

* **Azure Sentinel**, use el [conector de datos](https://docs.microsoft.com/azure/sentinel/connect-azure-security-center) de las alertas nativas de Azure Security Center que se ofrecen ahí.
* **Splunk**: use el [complemento de Azure Monitor para Splunk](https://github.com/Microsoft/AzureMonitorAddonForSplunk/blob/master/README.md)
* **IBM QRadar**: use [un origen de registro configurado manualmente](https://www.ibm.com/support/knowledgecenter/SS42VS_DSM/com.ibm.dsm.doc/t_dsm_guide_microsoft_azure_enable_event_hubs.html)
* **ArcSight**: use [SmartConnector](https://community.microfocus.com/t5/ArcSight-Connectors/SmartConnector-for-Microsoft-Azure-Monitor-Event-Hub/ta-p/1671292)

Asimismo, si quiere trasladar los datos exportados continuamente desde el Centro de eventos configurado a Azure Data Explorer, siga las instrucciones que se detallan en la [ingesta de datos del Centro de eventos en Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-hub).



## <a name="continuous-export-to-a-log-analytics-workspace"></a>Exportación continua a un área de trabajo de Log Analytics

Si desea analizar datos de Azure Security Center dentro de un área de trabajo de Log Analytics o usar alertas de Azure junto con Security Center, configure la exportación continua a su área de trabajo de Log Analytics.

Para exportar un área de trabajo de Log Analytics, debe tener habilitadas las soluciones de Log Analytics de Security Center en el área de trabajo. Si usa Azure Portal, la solución gratuita de Security Center se habilita automáticamente al habilitar la exportación continua. Sin embargo, si va a configurar las opciones de exportación continua mediante programación, debe activar o desactivar Azure Defender manualmente en la página de **precios y configuración**.

### <a name="log-analytics-tables-and-schemas"></a>Tablas y esquemas de Log Analytics

Las alertas y recomendaciones de seguridad se almacenan en las tablas *SecurityAlert* y *SecurityRecommendations* respectivamente. El nombre de la solución Log Analytics que contiene estas tablas depende de si tiene habilitado Azure Defender: Security ("Security and Audit") o SecurityCenterFree.

![Tabla *SecurityAlert* en Log Analytics](./media/continuous-export/log-analytics-securityalert-solution.png)

Para ver los esquemas de eventos de los tipos de datos exportados, visite el artículo sobre los [esquemas de tabla de Log Analytics](https://aka.ms/ASCAutomationSchemas).

###  <a name="view-exported-security-alerts-and-recommendations-in-azure-monitor"></a>Visualización de las recomendaciones y alertas de seguridad exportadas en Azure Monitor

En algunos casos, puede elegir ver las alertas de seguridad exportadas o las recomendaciones en [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-overview). 

Azure Monitor proporciona un conjunto de alertas unificadas que consta de varias alertas de Azure, como el registro de diagnóstico, las alertas de métricas y las alertas personalizadas basadas en las consultas del área de trabajo de Log Analytics.

Para ver las alertas y recomendaciones de Security Center en Azure Monitor, configure una regla de alerta basada en consultas de Log Analytics (alerta de registro):

1. En la página **Alertas**de Azure Monitor, seleccione **Nueva regla de alerta**.

    ![Página de alertas de Azure Monitor](./media/continuous-export/azure-monitor-alerts.png)

1. En la página de creación de reglas, configure la nueva regla (de la misma manera que configuraría una [regla de alertas de registro en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-unified-log)):

    * En **Recurso**, seleccione el área de trabajo de Log Analytics a la que exportó las recomendaciones y las alertas de seguridad.

    * En **Condición**, seleccione **Custom log search** (Búsqueda de registros personalizada). En la página que aparece, configure la consulta, el período de retrospectiva y el período de frecuencia. En la consulta de búsqueda, puede escribir *SecurityAlert* o *SecurityRecommendation* para consultar los tipos de datos que Security Center exporta continuamente al habilitar la característica de exportación continua a Log Analytics. 
    
    * Opcionalmente, en [Grupo de acciones](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups), configure el grupo de acciones que desea desencadenar. Los grupos de acciones pueden desencadenar el envío de correo electrónico, los vales de ITSM, los webhooks, etc.
    ![Regla de alerta de Azure Monitor](./media/continuous-export/azure-monitor-alert-rule.png)

Ahora verá las nuevas alertas o recomendaciones de Azure Security Center (según la configuración) en las alertas de Azure Monitor, con el desencadenamiento automático de un grupo de acciones (si se proporciona).

## <a name="manual-one-time-export-of-security-alerts"></a>Exportación manual de un solo uso de alertas de seguridad

Para descargar un informe en formato CSV con alertas o recomendaciones, abra las páginas **Alertas de seguridad** o **Recomendaciones** y seleccione el botón para **Download CSV report** (Descargar informe en CSV).

[![Descargar datos de alertas como un archivo CSV](media/continuous-export/download-alerts-csv.png)](media/continuous-export/download-alerts-csv.png#lightbox)

> [!NOTE]
> Estos informes contienen alertas y recomendaciones para los recursos de las suscripciones seleccionadas actualmente.



## <a name="faq---continuous-export"></a>Preguntas frecuentes: exportación continua

### <a name="what-are-the-costs-involved-in-exporting-data"></a>¿Cuáles son los costos de la exportación de datos?

La habilitación de una exportación continua no tiene costo alguno. Sin embargo, la ingesta y retención de datos del área de trabajo de Log Analytics, pueden tener costo, en función de la configuración. 

Obtenga más información sobre los [precios del área de trabajo de Log Analytics](https://azure.microsoft.com/pricing/details/monitor/).

Obtenga más información sobre los [precios de Azure Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a configurar exportaciones continuas de sus recomendaciones y alertas. También aprendió a descargar los datos de alertas como un archivo CSV. 

Para obtener material relacionado, consulte la documentación siguiente: 

- [Documentación de Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/)
- [Documentación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/)
- [Documentación sobre Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/)
- [Esquemas de tipos de datos de exportación continua y automatización del flujo de trabajo](https://aka.ms/ASCAutomationSchemas)
