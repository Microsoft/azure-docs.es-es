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
ms.openlocfilehash: 7b0fbb7c4f680f9d732a63fff7b0b317c6cf1511
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86519704"
---
# <a name="export-security-alerts-and-recommendations"></a>Exportación de alertas y recomendaciones de seguridad

Azure Security Center genera alertas y recomendaciones de seguridad detalladas. Puede verlas en el portal o mediante herramientas de programación. También puede exportar esta información o enviarla a otras herramientas de supervisión de su entorno. 

En este artículo se describe el conjunto de herramientas que le permiten exportar alertas y recomendaciones de forma manual o continua.

Con estas herramientas puede realizar lo siguiente:

* Exportación continua a las áreas de trabajo de Log Analytics
* Exportación continua a Azure Event Hubs (para integraciones con SIEM de terceros)
* Exportar a un archivo CSV (una vez)



## <a name="availability"></a>Disponibilidad

- Estado de la versión: **Disponibilidad general**
- Roles y permisos necesarios:
    - **Lector** en la suscripción que contiene la configuración de exportación.
    - **Rol de administrador de seguridad** en el grupo de recursos, o **Propietario**.
    - También debe tener permisos de escritura para el recurso de destino.
- Nubes:   ✔ Nubes comerciales   ✔ US Gov   ✘ China Gov, otros gobiernos


## <a name="setting-up-a-continuous-export"></a>Configuración de una exportación continua

Los pasos siguientes son necesarios si va a configurar una exportación continua a las área de trabajo de Log Analytics o a Azure Event Hubs.

1. En la barra lateral de Security Center, seleccione **Precios y configuración**.

1. Seleccione la suscripción específica para la que quiere configurar la exportación de datos.
    
1. En la barra lateral de la página de configuración de esa suscripción, seleccione **Exportación continua**.

    [![Opciones de exportación en Azure Security Center](media/continuous-export/continuous-export-options-page.png)](media/continuous-export/continuous-export-options-page.png#lightbox): aquí se ven las opciones de exportación. Hay una pestaña para cada destino de exportación disponible. 

1. Seleccione el tipo de datos que quiere exportar y elija los filtros que quiera de cada tipo (por ejemplo, exportar solo alertas de gravedad alta).

1. En el área "Destino de exportación", elija dónde quiere guardar los datos. Los datos se pueden guardar en el destino de una suscripción diferente (por ejemplo, en una instancia central del Centro de eventos o en un área de trabajo central de Log Analytics).

1. Haga clic en **Save**(Guardar).



## <a name="configuring-siem-integration-via-azure-event-hubs"></a>Configuración de la integración de SIEM a través de Azure Event Hubs

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

Para exportar un área de trabajo de Log Analytics, debe tener habilitadas las soluciones de Log Analytics de Security Center en el área de trabajo. Si usa Azure Portal, la solución de nivel gratuito de Security Center se habilita automáticamente al habilitar la exportación continua. Sin embargo, si va a configurar las opciones de exportación continua mediante programación, debe seleccionar manualmente el plan de tarifa gratuito o estándar para el área de trabajo necesaria desde **Precios y configuración**.  

### <a name="log-analytics-tables-and-schemas"></a>Tablas y esquemas de Log Analytics

Las alertas y recomendaciones de seguridad se almacenan en las tablas *SecurityAlert* y *SecurityRecommendations* respectivamente. El nombre de la solución de Log Analytics que contiene estas tablas cambiará dependiendo de si está en el nivel gratuito o estándar (consulte los [precios](security-center-pricing.md)): Security ("Security and Audit") o SecurityCenterFree.

![Tabla *SecurityAlert* en Log Analytics](./media/continuous-export/log-analytics-securityalert-solution.png)

Para ver los esquemas de eventos de los tipos de datos exportados, visite el artículo sobre los [esquemas de tabla de Log Analytics](https://aka.ms/ASCAutomationSchemas).

###  <a name="view-exported-security-alerts-and-recommendations-in-azure-monitor"></a>Visualización de las recomendaciones y alertas de seguridad exportadas en Azure Monitor

En algunos casos, puede elegir ver las alertas de seguridad exportadas o las recomendaciones en [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-overview). 

Azure Monitor proporciona un conjunto de alertas unificadas que consta de varias alertas de Azure, como el registro de diagnóstico, las alertas de métricas y las alertas personalizadas basadas en las consultas del área de trabajo de Log Analytics.

Para ver las alertas y recomendaciones de Security Center en Azure Monitor, configure una regla de alerta basada en consultas de Log Analytics (alerta de registro):

1. En la página **Alertas** de Azure Monitor, haga clic en **Nueva regla de alertas**.

    ![Página de alertas de Azure Monitor](./media/continuous-export/azure-monitor-alerts.png)

1. En la página de creación de reglas, configure la nueva regla (de la misma manera que configuraría una [regla de alertas de registro en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-unified-log)):

    * En **Recurso**, seleccione el área de trabajo de Log Analytics a la que exportó las recomendaciones y las alertas de seguridad.

    * En **Condición**, seleccione **Custom log search** (Búsqueda de registros personalizada). En la página que aparece, configure la consulta, el período de retrospectiva y el período de frecuencia. En la consulta de búsqueda, puede escribir *SecurityAlert* o *SecurityRecommendation* para consultar los tipos de datos que Security Center exporta continuamente al habilitar la característica de exportación continua a Log Analytics. 
    
    * Opcionalmente, en [Grupo de acciones](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups), configure el grupo de acciones que desea desencadenar. Los grupos de acciones pueden desencadenar el envío de correo electrónico, los vales de ITSM, los webhooks, etc.
    ![Regla de alerta de Azure Monitor](./media/continuous-export/azure-monitor-alert-rule.png)

Ahora verá las nuevas alertas o recomendaciones de Azure Security Center (según la configuración) en las alertas de Azure Monitor, con el desencadenamiento automático de un grupo de acciones (si se proporciona).

## <a name="manual-one-time-export-of-security-alerts"></a>Exportación manual de un solo uso de alertas de seguridad

Para descargar un informe en formato CSV con alertas o recomendaciones, abra la página de **alertas de seguridad** o de **recomendaciones** y haga clic en el botón para **descargar el informe CSV**.

[![Descargar datos de alertas como un archivo CSV](media/continuous-export/download-alerts-csv.png)](media/continuous-export/download-alerts-csv.png#lightbox)

> [!NOTE]
> Estos informes contienen alertas y recomendaciones para los recursos de las suscripciones seleccionadas actualmente.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a configurar exportaciones continuas de sus recomendaciones y alertas. También aprendió a descargar los datos de alertas como un archivo CSV. 

Para obtener material relacionado, consulte la documentación siguiente: 

- [Documentación de Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/)
- [Documentación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/)
- [Documentación sobre Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/)
- [Esquemas de tipos de datos de exportación continua y automatización del flujo de trabajo](https://aka.ms/ASCAutomationSchemas)
