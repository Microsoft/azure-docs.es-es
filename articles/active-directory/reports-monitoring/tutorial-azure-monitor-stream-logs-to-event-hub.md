---
title: 'Tutorial: Transmisión de registros a un centro de eventos de Azure | Microsoft Docs'
description: Aprenda a configurar Azure Diagnostics para insertar registros de Azure Active Directory en un centro de eventos.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 045f94b3-6f12-407a-8e9c-ed13ae7b43a3
ms.service: active-directory
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0443dcb2bf3bd58f2474c507c9f9594fb6d8a7f0
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89469191"
---
# <a name="tutorial-stream-azure-active-directory-logs-to-an-azure-event-hub"></a>Tutorial: Transmisión de registros de Azure Active Directory a un centro de eventos de Azure

En este tutorial, aprenderá a configurar diagnósticos de Azure Monitor para transmitir registros de Azure Active Directory (Azure AD) a un centro de eventos de Azure. Use este mecanismo para integrar sus registros con las herramientas de Administración de eventos e información de seguridad (SIEM) de terceros, como Splunk y QRadar.

## <a name="prerequisites"></a>Prerrequisitos 

Para usar esta característica, necesita:

* Suscripción a Azure. Si no tiene ninguna suscripción de Azure, puede [registrarse para obtener una evaluación gratuita](https://azure.microsoft.com/free/).
* Un inquilino de Azure AD.
* Un usuario que sea *administrador global* o *administrador de seguridad* para el inquilino de Azure AD.
* Un espacio de nombres de Event Hubs y un centro de eventos en la suscripción de Azure. Aprenda a [crear un centro de eventos](../../event-hubs/event-hubs-create.md).

## <a name="stream-logs-to-an-event-hub"></a>Transmitir registros a un centro de eventos

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 

2. Seleccione **Azure Active Directory** > **Supervisión** > **Registros de auditoría**. 

3. Seleccione **Exportar configuración**.  
    
4. En el panel **Configuración de diagnóstico**, realice una de las siguientes acciones:
    * Para cambiar la configuración existente, seleccione **Editar configuración**.
    * Para agregar la nueva configuración, seleccione **Agregar configuración de diagnóstico**.  
      Puede tener un máximo de tres configuraciones.

      ![Exportar configuración](./media/quickstart-azure-monitor-stream-logs-to-event-hub/ExportSettings.png)

5. Seleccione la casilla **Stream to an event hub** (Hacer streaming a un centro de eventos) y haga clic en **Event Hub/Configure** (Centro de eventos/Configurar).

6. Seleccione la suscripción de Azure y el espacio de nombres de Event Hubs al que desea enrutar los registros.  
    Tanto la suscripción como el espacio de nombres de Event Hubs deben estar asociados al inquilino de Azure AD desde el que se envían los registros por streaming. También se puede especificar un centro de eventos en el espacio de nombres de Event Hubs al que se deben enviar los registros. Si no se especifica ningún centro de eventos, se crea uno en el espacio de nombres con el nombre predeterminado **insights-logs-audit**.

7. Seleccione **Aceptar** para salir de la configuración del centro de eventos.

8. Realice alguna de las siguientes acciones o ambas:
    * Para enviar los registros de auditoría al centro de eventos, seleccione la casilla **AuditLogs**. 
    * Para enviar los registros de inicio de sesión al centro de eventos, seleccione la casilla **SignInLogs**.

9. Seleccione **Guardar** para guardar la configuración.

    ![Configuración de diagnóstico](./media/quickstart-azure-monitor-stream-logs-to-event-hub/DiagnosticSettings.png)

10. Quince minutos después, compruebe que los eventos se muestran en el centro de eventos. Para ello, vaya al centro de eventos desde el portal y compruebe que el número de **mensajes entrantes** es mayor que cero. 

    ![Registros de auditoría](./media/quickstart-azure-monitor-stream-logs-to-event-hub/InsightsLogsAudit.png)

## <a name="access-data-from-your-event-hub"></a>Acceso a datos desde un centro de eventos

Después de mostrar los datos del centro de eventos, puede acceder y leer los datos de dos maneras:

* **Configurar una herramienta SIEM compatible**. Para leer los datos del centro de eventos, la mayoría de las herramientas requieren la cadena de conexión al centro de eventos y determinados permisos para la suscripción de Azure. Herramientas de terceros con la integración de Azure Monitor incluidas, entre otras:
    
    * **ArcSight**: para más información sobre cómo integrar los registros de Azure AD con Splunk, consulte [Integración de los registros de Azure Active Directory con ArcSight mediante Azure Monitor](howto-integrate-activity-logs-with-arcsight.md).
    
    * **Splunk**: para más información sobre cómo integrar los registros de Azure AD con Splunk, consulte [Integración de registros de Azure AD con Splunk mediante Azure Monitor](./howto-integrate-activity-logs-with-splunk.md).
    
    * **IBM QRadar**: DSM y el protocolo de Azure Event Hubs están disponibles para descarga en el [soporte técnico de IBM](https://www.ibm.com/support). Para más información sobre la integración con Azure, vaya al sitio [IBM QRadar Security Intelligence Platform 7.3.0](https://www.ibm.com/support/knowledgecenter/SS42VS_DSM/c_dsm_guide_microsoft_azure_overview.html?cp=SS42VS_7.3.0).
    
    * **Sumo Logic**: para configurar Sumo Logic para que consuma datos de un centro de eventos, consulte [Install the Azure AD app and view the dashboards](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Install_the_Azure_Active_Directory_App_and_View_the_Dashboards) (Instalación de la aplicación de Azure AD y visualización de los paneles). 

* **Configuración de herramientas personalizadas**. Si su herramienta SIEM actual aún no se admite en los diagnósticos de Azure Monitor, puede configurar herramientas personalizadas mediante las API de Event Hubs. Para más información, consulte la [Introducción a la recepción de mensajes con el Host del procesador de eventos en .NET Standard](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md).


## <a name="next-steps"></a>Pasos siguientes

* [Integración de los registros de Azure Active Directory con ArcSight mediante Azure Monitor](howto-integrate-activity-logs-with-arcsight.md)
* [Integración de registros de Azure AD con Splunk mediante Azure Monitor](./howto-integrate-activity-logs-with-splunk.md)
* [Integración de registros de Azure AD con SumoLogic mediante Azure Monitor](howto-integrate-activity-logs-with-sumologic.md)
* [Interpretación del esquema de registros de auditoría en Azure Monitor](reference-azure-monitor-audit-log-schema.md)
* [Interpretación del esquema de registros de inicio de sesión en Azure Monitor](reference-azure-monitor-sign-ins-log-schema.md)
