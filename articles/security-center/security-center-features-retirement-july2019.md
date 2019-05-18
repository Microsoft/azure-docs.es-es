---
title: Retirada del centro de seguridad de las características (julio de 2019) | Microsoft Docs
description: En este artículo se describe las características en el centro de seguridad que se retirará el 31 de julio de 2019.
services: security-center
author: yoavfrancis
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.date: 4/16/2019
ms.author: yoafr
ms.openlocfilehash: 5ac6603be087e887e3304f6a7e14aa063863e1dd
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796468"
---
# <a name="retirement-of-security-center-features-july-2019"></a>Retirada de las características de Security Center (julio de 2019)

Hemos realizado varias [mejoras](https://azure.microsoft.com/updates/?product=security-center) a Azure Security Center durante los últimos seis meses.
Con estas capacidades mejoradas, nos estamos quitando algunas características redundantes y las API relacionadas desde el centro de seguridad en el 31 de julio de 2019.  

La mayoría de estas características de retirada puede reemplazarse con nueva funcionalidad en Azure Security Center o Azure Log Analytics. Otras características se pueden implementar mediante [Centinela de Azure (versión preliminar)](https://azure.microsoft.com/services/azure-sentinel/).

Las características de Security Center se retirarán incluyen:

- [Panel eventos](#menu_events)
- [Entrada de menú de búsqueda](#menu_search)
- [Vínculos clásico de identidad y acceso de vista sobre la identidad y acceso (versión preliminar)](#menu_classicidentity)
- [Botón en el mapa de las alertas de seguridad (versión preliminar) asignan los eventos de seguridad](#menu_securityeventsmap)
- [Reglas de alerta personalizadas (versión preliminar)](#menu_customalerts)
- [Investigar el botón en las alertas de seguridad de protección contra amenazas](#menu_investigate)
- [Un subconjunto de las soluciones de seguridad](#menu_solutions)
- [Editar configuraciones de seguridad para las directivas de seguridad](#menu_securityconfigurations)
- [Panel seguridad y auditoría (usado originalmente en el portal de OMS) para las áreas de trabajo de Log Analytics](#menu_securityomsdashboard)

En este artículo se proporciona información detallada para cada característica retirada y los pasos que puede seguir para implementar funciones de reemplazo.

## Panel eventos<a name="menu_events"></a>

Security Center usa a Microsoft Monitoring Agent para recopilar distintas configuraciones relacionadas con la seguridad y los eventos de las máquinas. Estos eventos almacena en las áreas de trabajo. El [panel eventos](https://docs.microsoft.com/azure/security-center/security-center-events-dashboard) le permite ver los datos y le ofrece un punto de entrada a Log Analytics.

Estamos retirando el panel de eventos que aparece al seleccionar un área de trabajo:

![Panel Eventos][2]

### <a name="events-dashboard---the-new-experience"></a>Panel de eventos - la nueva experiencia

Se recomienda usar las capacidades nativas de Azure Log Analytics para ver los eventos importantes en las áreas de trabajo.

Si ha creado los eventos importantes personalizados en Security Center, que sean accesibles. En Log Analytics, vaya a **seleccionar área de trabajo** > **búsquedas guardadas**. No se pierde o modifica los datos. Eventos importantes nativos también están disponibles en la misma pantalla de Log Analytics.

![Las búsquedas guarda del área de trabajo][3]

## Entrada de menú de búsqueda<a name="menu_search"></a>

Azure Security Center usa actualmente la búsqueda de registros de Azure Monitor para recuperar y analizar los datos de seguridad. Esta pantalla sirve como una ventana de la página de búsqueda de Log Analytics y permite a los usuarios ejecutar consultas de búsqueda en su área de trabajo seleccionada. Para obtener más información, consulte [búsqueda de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-search). Estamos retirando esta ventana de búsqueda:

![Página de búsqueda][4]

### <a name="search-menu-entry---the-new-experience"></a>Entrada de menú de búsqueda: la nueva experiencia

Le animamos a usar las capacidades nativas de Azure Log Analytics para realizar consultas de búsqueda en las áreas de trabajo. Vaya a Azure Log Analytics y seleccione **registros**.

![Página de registros de log Analytics][5]

## Clásico identidad y acceso (versión preliminar)<a name="menu_classicidentity"></a>

Actualmente, la experiencia clásica de identidad y acceso en Security Center muestra un panel de información de identidad y acceso en Log Analytics. Para ver este panel:

1. Seleccione **vista clásica identidad y acceso**.

   ![Página de identidad][6]

1. Ver el **panel identidad y acceso**.

    ![Página de identidad: selección del área de trabajo][7]

1. Seleccione un área de trabajo para abrir el **identidad y acceso** panel en Log Analytics para ver la identidad y acceso a la información en el área de trabajo.

   ![Página de identidad: panel][8]

Estamos retirando todas las tres pantallas que se muestra en los pasos anteriores. Los datos seguirán estando disponibles en la solución de seguridad de Log Analytics y no puede modificar ni quitar.

### <a name="classic-identity--access-preview---the-new-experience"></a>Clásica identidad y acceso (versión preliminar): la nueva experiencia

El panel de Log Analytics ha demostrado insights en un área de trabajo. Sin embargo, las capacidades nativas de Security Center proporcionan visibilidad en todas las suscripciones y todas las áreas de trabajo asociados con ellos. Puede tener acceso a una forma sencilla de-para usar la vista que le permite centrarse en lo que es importante con las recomendaciones se clasifican según su puntuación segura.

Todas las características de la **identidad y acceso** panel de Log Analytics puede ponerse seleccionando **identidad y acceso (versión preliminar)** dentro de Security Center.

![Retirada de la experiencia de página identidad - clásico][9]

## Mapa de eventos de seguridad<a name="menu_securityeventsmap"></a>

Security Center proporciona un [mapa de las alertas de seguridad](https://docs.microsoft.com/azure/security-center/security-center-threat-intel) para ayudar a identificar las amenazas de seguridad. El **ir al mapa de eventos de seguridad** botón en el que se asignan abre un panel que le permite ver los eventos de seguridad sin procesar en el área de trabajo seleccionada.

Nos estamos quitando el **ir al mapa de eventos de seguridad** botón y el panel por área de trabajo.

![Botón de mapa: las alertas de seguridad][10]

Cuando se selecciona el **ir al mapa de eventos de seguridad** botón, abra el panel de inteligencia de amenazas. Estamos retirando el panel de inteligencia de amenazas.  

![Panel Inteligencia de amenazas][11]

Al elegir un área de trabajo para ver su panel de inteligencia de amenazas, abra la pantalla de mapa (versión preliminar) de las alertas de seguridad de registros en Log Analytics. Estamos retirando esta pantalla.

![Mapa de las alertas de seguridad en Log Analytics][12]

Los datos existentes seguirán estando disponibles en la solución de seguridad de Log Analytics y no se pueden modificar ni quitar.

### <a name="security-events-map---the-new-experience"></a>Mapa de eventos de seguridad: la nueva experiencia

Le recomendamos que use la funcionalidad de asignación de las alertas integrada en el centro de seguridad: **Mapa de las alertas de seguridad (versión preliminar)**. Esta funcionalidad proporciona una experiencia optimizada y funciona en todas las suscripciones y áreas de trabajo asociados. Proporciona una visión general en todo el entorno y no se centra en un área de trabajo.

## Reglas de alerta personalizadas (versión preliminar)<a name="menu_customalerts"></a>

Estamos [retirar personalizado experiencia de alertas](https://docs.microsoft.com/azure/security-center/security-center-custom-alert) el 30 de junio de 2019 porque se eliminará su infraestructura subyacente. Hasta entonces, puede editar reglas de alerta personalizadas existentes, pero no puede agregar otros nuevos. Se recomienda que habilite [Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/) automáticamente migrar las alertas existentes y crear otros nuevos. Como alternativa, puede crear las alertas con alertas de registro de Azure Monitor.

Para mantener las alertas existentes y migrarlas a Azure centinela:

1. Abra Azure Sentinel y seleccione el área de trabajo donde se almacenan las alertas personalizadas.
1. Seleccione **Analytics** en el menú para migrar automáticamente las alertas.

![Alertas personalizadas][13]

Si no está interesado en realizar la transición a Azure Sentinel, le recomendamos que cree las alertas con alertas de registro de Azure Monitor. Para obtener instrucciones, consulte [crear, ver y administrar las alertas del registro mediante el uso de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-log) y [alertas de registro en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-unified-log).

Para obtener más información sobre la retirada de alertas personalizadas, vea [reglas de alerta personalizadas en Azure Security Center (versión preliminar)](https://docs.microsoft.com/azure/security-center/security-center-custom-alert).

## Investigación de alertas de seguridad<a name="menu_investigate"></a>

[La característica de investigación](https://docs.microsoft.com/azure/security-center/security-center-investigation) en Security Center ayuda a evaluar las prioridades un incidente de seguridad potenciales. La característica permite entender el ámbito de un incidente y localizar la causa raíz. Lo que estamos quita esta característica desde el centro de seguridad ya que se ha reemplazado por una experiencia mejorada en [Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/).

![Incidente de seguridad][14]

Cuando se selecciona el **investigar** botón desde un **incidente de seguridad** pantalla, abra el panel de investigación (versión preliminar) en Log Analytics. Estamos retirando el panel de investigación.  

Los datos existentes seguirán estando disponibles en la solución de seguridad de Log Analytics y no se pueden modificar ni quitar.

![Panel de investigación en Log Analytics][15]

### <a name="investigation---the-new-experience"></a>Investigación: la nueva experiencia

Le recomendamos que pase al [Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/) para una experiencia enriquecida de investigación. Sentinel Azure proporciona eficaces herramientas de búsqueda y consulta buscando amenazas de seguridad a través de orígenes de datos de su organización.  

## Subconjunto de las soluciones de seguridad<a name="menu_solutions"></a>

Puede habilitar Security Center [soluciones de seguridad de Azure integradas](https://docs.microsoft.com/azure/security-center/security-center-partner-integration). Estamos retirando las siguientes soluciones de asociados desde Security Center. Estas soluciones están habilitadas en [Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/) junto con un número de orígenes de datos adicionales.

- [Próxima generación web y firewall de aplicación las soluciones de firewall](https://docs.microsoft.com/azure/sentinel/connect-data-sources)
- [Integración de soluciones de seguridad que admite Common Event Format (CEF)](https://docs.microsoft.com/azure/sentinel/connect-common-event-format)
- [Microsoft Advanced Threat Analytics](https://docs.microsoft.com/azure/sentinel/connect-azure-atp)
- [Azure AD Identity Protection](https://docs.microsoft.com/azure/sentinel/connect-azure-ad-identity-protection)

Después de la retirada, no podrá agregar o modificar cualquiera de los tipos de soluciones mencionados en la lista anterior, ya sea desde la interfaz de usuario o la API.

Si tiene soluciones conectadas existentes, le animamos a mover a Azure Sentinel.

![Soluciones de centros de seguridad][16]

## Editar configuraciones de seguridad para las directivas de seguridad<a name="menu_securityconfigurations"></a>

Azure Security Center supervisa las configuraciones de seguridad mediante la aplicación de un conjunto de [más de 150 reglas recomendadas](https://gallery.technet.microsoft.com/Azure-Security-Center-a789e335). para proteger el sistema operativo. Estas reglas se aplican a los firewalls, auditoría, las directivas de contraseñas y mucho más. Si se detecta que la configuración de una maquina es vulnerable, Security Center genera una recomendación de seguridad. El [pantalla de configuración de seguridad de edición](https://docs.microsoft.com/azure/security-center/security-center-customize-os-security-config) permite a los clientes personalizar la configuración de seguridad predeterminada del sistema operativo en Security Center.

Estamos retirando esta característica de vista previa.

![Edición de las opciones de configuración de seguridad][17]

### <a name="edit-security-configurations---the-new-experience"></a>Editar configuraciones de seguridad: la nueva experiencia

Tenemos previsto habilitar Security Center admitir la [agente de configuración de invitado](https://docs.microsoft.com/azure/governance/policy/concepts/guest-configuration). Dicha actualización permitirá que un conjunto de características mucho más completo, incluida la compatibilidad con varios sistemas operativos y la integración de las directivas de invitado de Azure para las configuraciones de invitado. Después de que estos cambios están habilitados, también tendrá la capacidad de controlar las configuraciones de escalado y aplicar automáticamente a los nuevos recursos.

## Panel seguridad y auditoría para las áreas de trabajo de Log Analytics<a name="menu_securityomsdashboard"></a>

El panel security and audit se utilizó originalmente en el portal de OMS. En Log Analytics, el panel proporciona una introducción al área de trabajo de eventos de seguridad importantes y las amenazas, un mapa de inteligencia sobre amenazas y una evaluación de identidad y acceso de eventos de seguridad que se guardan en el área de trabajo. Nos estamos quitando el panel. Como ya se han recomendado en el panel de interfaz de usuario, le recomendamos que, para realizar la transición a Azure Security Center.

![Panel de seguridad de log Analytics][18]

### <a name="security-and-audit-dashboard---the-new-experience"></a>Panel de Security and audit - la nueva experiencia

Le recomendamos que cambie a Azure Security Center. Proporciona la misma información general de seguridad en varias suscripciones y las áreas de trabajo asociados a ellos, además de establece una característica más completa.

Puede obtener las consultas de Log Analytics originales que rellenan el panel seguridad y auditoría en el [repositorio de GitHub](https://github.com/Azure/Azure-Security-Center/tree/master/Legacy%20Log%20Analytics%20dashboards) de Security Center.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Azure Security Center](https://docs.microsoft.com/azure/security-center/)
- Obtenga más información sobre [Azure Sentinel](https://docs.microsoft.com/azure/sentinel).

<!--Image references - events-->
[1]: ./media/security-center-features-retirement-july2019/asc_events_dashboard.png
[2]: ./media/security-center-features-retirement-july2019/asc_events_dashboard_inner.png
[3]: ./media/security-center-features-retirement-july2019/workspace_saved_searches.png
<!--Image references - search-->
[4]: ./media/security-center-features-retirement-july2019/asc_search.png
[5]: ./media/security-center-features-retirement-july2019/workspace_logs.png
<!--Image references - classic identity and access-->
[6]: ./media/security-center-features-retirement-july2019/asc_identity.png
[7]: ./media/security-center-features-retirement-july2019/asc_identity_workspace_selection.png
[8]: ./media/security-center-features-retirement-july2019/loganalytics_dashboard_identity.png
[9]: ./media/security-center-features-retirement-july2019/asc_identity_nobuttonhighlight.png
<!--Image references - alerts map-->
[10]: ./media/security-center-features-retirement-july2019/asc_security_alerts_map.png
[11]: ./media/security-center-features-retirement-july2019/asc_threat_intellignece_dashboard.png
[12]: ./media/security-center-features-retirement-july2019/loganalytics_security_alerts_map.png
<!--Image references - custom alerts-->
[13]: ./media/security-center-features-retirement-july2019/asc_custom_alerts.png
<!--Image references - Investigation-->
[14]: ./media/security-center-features-retirement-july2019/asc-security-incident.png
[15]: ./media/security-center-features-retirement-july2019/loganalytics_investigation_dashboard.png
<!--Image references - Solutions-->
[16]: ./media/security-center-features-retirement-july2019/asc_security_solutions.png
<!--Image references - Edit security configurations-->
[17]: ./media/security-center-features-retirement-july2019/asc_edit_security_configurations.png
<!--Image references - Security dashboard in log analytics-->
[18]: ./media/security-center-features-retirement-july2019/loganalytics_security_dashboard.png
