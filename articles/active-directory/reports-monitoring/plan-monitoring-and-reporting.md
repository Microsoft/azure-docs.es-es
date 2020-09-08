---
title: 'Planeamiento de informes y supervisión de la implementación: Azure AD'
description: Describe cómo planear y ejecutar la implementación de la supervisión y los informes.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: baselden
ms.reviewer: plenzke
ms.collection: M365-identity-device-management
ms.openlocfilehash: 20b83291dc37c6248761214654f99d3ce214b551
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229759"
---
# <a name="plan-an-azure-active-directory-reporting-and-monitoring-deployment"></a>Planeamiento de la implementación de la supervisión y los informes de Azure Active Directory

La solución de supervisión e informes de Azure Active Directory (Azure AD) depende de sus requisitos legales, de seguridad y operativos, así como del entorno y los procesos existentes. En este artículo se presentan las distintas opciones de diseño y se le guía para que elija la estrategia de implementación adecuada.

### <a name="benefits-of-azure-ad-reporting-and-monitoring"></a>Ventajas de la supervisión y los informes de Azure AD

Los informes de Azure AD proporcionan una vista completa y registros de la actividad de Azure AD en su entorno, lo que incluye los eventos de inicio de sesión, los eventos de auditoría y los cambios en el directorio.

Los datos proporcionados le permiten:

* determinar cómo se utilizan las aplicaciones y servicios,

* detectar posibles riesgos que afectan al estado del entorno,

* solucionar problemas que impiden a los usuarios finalizar su trabajo,

* obtener conclusiones tras ver los eventos de auditoría de los cambios en su directorio de Azure AD.

> [!IMPORTANT]
> La supervisión de Azure AD permite enrutar los registros que generan los informes de Azure AD a diferentes sistemas de destino. A continuación, puede conservarlo para su uso a largo plazo o integrarlo con herramientas de Administración de eventos e información de seguridad (SIEM) de terceros para obtener información sobre su entorno.

Con la supervisión de Azure AD, puede enrutar los registros a:

* una cuenta de Azure Storage con fines de archivo,
* registros de Azure Monitor, que anteriormente se conocían como área de trabajo de Azure Log Analytics, donde puede analizar los datos, crear paneles y alertar acerca de eventos específicos,
* un centro de eventos de Azure en el que puede realizar la integración con sus herramientas de SIEM existentes, como Splunk, Sumologic o QRadar.

> [!NOTE]
Recientemente hemos empezado a usar el término registros de Azure Monitor, en lugar de Log Analytics. Los datos de registro siguen almacenándose en un área de trabajo de Log Analytics y siguen recopilándose y analizándose por el mismo servicio de Log Analytics. Estamos actualizando la terminología para reflejar mejor el rol de los [registros de Azure Monitor](../../azure-monitor/platform/data-platform.md). Consulte [Azure Monitor terminology changes](../../azure-monitor/terminology.md) (Cambios en la terminología de Azure Monitor) para obtener más información.

[Más información sobre las directivas de retención de informes](./reference-reports-data-retention.md).

### <a name="licensing-and-prerequisites-for-azure-ad-reporting-and-monitoring"></a>Licencias y requisitos previos para los informes y la supervisión de Azure AD

Necesitará una licencia de Azure AD Premium para acceder a los registros de inicio de sesión de Azure AD.

Para más información acerca de las características y licencias, consulte la [guía de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

Para implementar la supervisión y los informes de Azure AD, necesitará un usuario que sea administrador global o administrador de seguridad del inquilino de Azure AD.

Según el destino final de los datos de registro, necesitará una de las opciones siguientes:

* Una cuenta de almacenamiento de Azure, para la que tenga permisos ListKeys. Le recomendamos utilizar una cuenta de almacenamiento general y no de almacenamiento de blobs. Para más información sobre precios de almacenamiento, consulte la [Calculadora de precios de Azure Storage](https://azure.microsoft.com/pricing/calculator/?service=storage).

* Un espacio de nombres de Azure Event Hubs para la integración con soluciones SIEM de terceros.

* Un área de trabajo de Azure Log Analytics para enviar registros a registros de Azure Monitor.

## <a name="plan-an-azure-reporting-and-monitoring-deployment-project"></a>Planeamiento de un proyecto de implementación de la supervisión y los informes de Azure

En este proyecto, definirá las audiencias que consumirán y supervisarán los informes, y definirá la arquitectura de supervisión de Azure AD.

### <a name="engage-the-right-stakeholders"></a>Interactuar con las partes interesadas adecuadas

Cuando fracasan los proyectos tecnológicos, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de que interactúa con las partes interesadas adecuadas](https://aka.ms/deploymentplans). Asegúrese también de que los roles de las partes interesadas en el proyecto se entienden bien; para ello, documente las partes interesadas y sus aportes y responsabilidades en el proyecto.

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para el éxito de cualquier servicio nuevo. Comunique de forma proactiva a los usuarios cómo y cuándo va a cambiar su experiencia, y cómo obtener soporte técnico si tienen cualquier problema.

### <a name="document-your-current-infrastructure-and-policies"></a>Documentación de la infraestructura y las directivas actuales

La infraestructura y las directivas actuales guiarán su diseño de los informes y la supervisión. Asegúrese de que conoce

* Cuáles son las herramientas SIEM que usa, en caso de que use alguna.

* La infraestructura de Azure, incluidas las cuentas de almacenamiento existentes y la supervisión que se está usando.

* Las directivas de retención de la organización para los registros, incluidos los marcos de cumplimiento aplicables necesarios. 

## <a name="plan-an-azure-ad-reporting-and-monitoring-deployment"></a>Planeamiento de la implementación de la supervisión y los informes de Azure AD

Los informes y la supervisión se usan para cumplir los requisitos empresariales, obtener información acerca de los patrones de uso y aumentar la postura de seguridad de la organización.

### <a name="business-use-cases"></a>Casos de uso empresarial

* Se requiere para que la solución cumpla las necesidades empresariales
* Conviene tenerlo para satisfacer las necesidades empresariales
* No aplicable

|Área |Descripción |
|-|-|
|Retención| **Retención del registro durante más de 30 días**. Los requisitos legales o empresariales obligan a almacenar tanto los registros de auditoría como los registros de inicio de sesión de Azure AD durante más de 30 días. |
|Análisis| **Es preciso poder realizar búsquedas en los registros**. ‎Es necesario realizar búsquedas en los registros almacenados con las herramientas de análisis. |
| Operational Insights| **Información para varios equipos**. La necesidad de conceder acceso a distintos usuarios para obtener una visión de las operaciones, como el uso de las aplicaciones, los errores de inicio de sesión, el uso de autoservicio, las tendencias, etc. |
| Información de seguridad| **Información para varios equipos**. La necesidad de conceder acceso a distintos usuarios para obtener una visión de las operaciones, como el uso de las aplicaciones, los errores de inicio de sesión, el uso de autoservicio, las tendencias, etc. |
| Integración en sistemas SIEM      | **Integración de SIEM**. ‎La necesidad de integrar y transmitir en secuencias los registros de inicio de sesión de Azure AD y los registros de auditoría a sistemas de SIEM existentes. |

### <a name="choose-a-monitoring-solution-architecture"></a>Elección de una arquitectura de soluciones de supervisión

Con la supervisión de Azure AD, puede enrutar los registros de actividad de Azure AD a un sistema que se adapte mejor a sus necesidades empresariales. Después, puede conservarlos para los informes y análisis a largo plazo para obtener información sobre su entorno e integrarlo con las herramientas SIEM.

#### <a name="decision-flow-chartan-image-showing-what-is-described-in-subsequent-sections"></a>Diagrama de flujo de decisión![Imagen en la que se muestra lo que se describe en las secciones posteriores](media/reporting-deployment-plan/deploy-reporting-flow-diagram.png)

#### <a name="archive-logs-in-a-storage-account"></a>Archivo de registros en una cuenta de almacenamiento

Si se enrutan los registros a una cuenta de Azure Storage, se pueden conservar durante más tiempo que el período de retención predeterminado que se describe en las [directivas de retención](./reference-reports-data-retention.md). Use este método si necesita archivar los registros, pero no necesita integrarlos en un sistema SIEM ni realizar consultas y análisis continuos. Aunque lo use puede seguir realizando búsquedas a petición.

Obtenga información sobre cómo [enrutar datos a la cuenta de almacenamiento](./quickstart-azure-monitor-route-logs-to-storage-account.md).

#### <a name="send-logs-to-azure-monitor-logs"></a>Envío de registros a registros de Azure Monitor

Los [registros de Azure Monitor](../../azure-monitor/log-query/log-query-overview.md) consolidan los datos de supervisión de distintos orígenes. También proporciona un lenguaje de consulta y un motor de análisis que ofrece información detallada acerca del funcionamiento de las aplicaciones y del uso de los recursos. Al enviar los registros de actividad de Azure AD a los registros de Azure Monitor, puede recuperar, supervisar y enviar alertas rápidamente de los datos recopilados. Use este método si no tiene una solución SIEM existente a la que desee enviar los datos directamente, pero desea realizar consultas y análisis. Una vez que los datos estén en los registros de Azure Monitor, puede enviarlos al centro de eventos y desde ahí a un SIEM si lo desea.

Aprenda a [enviar datos a los registros de Azure Monitor](./howto-integrate-activity-logs-with-log-analytics.md).

También puede instalar las vistas precompiladas de los registros de actividad de Azure AD para supervisar escenarios comunes que impliquen eventos de inicio de sesión y auditoría.

Aprenda a [instalar y utilizar las vistas de Log Analytics para los registros de actividad de Azure AD](./howto-install-use-log-analytics-views.md).

#### <a name="stream-logs-to-your-azure-event-hub"></a>Transmitir en secuencias de registros a un centro de eventos de Azure

El enrutamiento de registros a un centro de eventos de Azure permite la integración con herramientas SIEM de terceros. Esta integración le permite combinar los datos de registro de actividad de Azure AD con otros datos administrados por el SIEM, a fin de proporcionar una mejor comprensión del entorno. 

Aprenda cómo [transmitir registros a un centro de eventos](//azure/active-directory/reports-monitoring/tutorial-azure-monitor-stream-logs-to-event-hub).

## <a name="plan-operations-and-security-for-azure-ad-reporting-and-monitoring"></a>Planeamiento de las operaciones y la seguridad de los informes y la de Azure AD

Las partes interesadas necesitan acceder a los registros de Azure AD para obtener una visión de las operaciones. Entre los posibles usuarios se incluyen miembros del equipo de seguridad, auditores internos o externos, y el equipo de operaciones de administración de identidades y acceso.

Los roles de Azure AD le permiten delegar la capacidad de configurar y ver los informes de Azure AD en función de su rol. Identifique qué usuarios de la organización necesitan permiso para leer informes de Azure AD y qué rol sería adecuado para ellos. 

Los siguientes roles pueden leer informes de Azure AD:

* Administrador global

* Administrador de seguridad

* Lector de seguridad

* Lector de informes

Más información acerca de los [roles administrativos de Azure AD](../users-groups-roles/directory-assign-admin-roles.md).

*Aplique siempre el concepto de privilegios mínimos para reducir el riesgo de que una cuenta esté en peligro*. Considere la posibilidad de implementar [Privileged Identity Management](../privileged-identity-management/pim-configure.md) para aumentar la protección de su organización.

##  

## <a name="deploy-azure-ad-reporting-and-monitoring"></a>Implementación de la supervisión y los informes de Azure AD

En función de las decisiones que haya tomado anteriormente mediante la guía de diseño, esta sección le guiará a la documentación de las diferentes opciones de implementación.

### <a name="consume-and-archive-azure-ad-logs"></a>Consumo y archivo de registros de Azure AD

[Búsqueda de informes de actividad en Azure Portal](./howto-find-activity-reports.md)

[Archivo de registros de Azure AD en una cuenta de Azure Storage](./quickstart-azure-monitor-route-logs-to-storage-account.md)

### <a name="implement-monitoring-and-analytics"></a>Implementación de supervisión y análisis

[Envío de registros a Azure Monitor](./howto-integrate-activity-logs-with-log-analytics.md)

[Instalación y uso de las vistas de Log Analytics para Azure Active Directory](./howto-install-use-log-analytics-views.md)

[Análisis de registros de actividad de Azure AD con registros de Azure Monitor](./howto-analyze-activity-logs-log-analytics.md)

* [Interpretación del esquema de registros de auditoría en Azure Monitor](./reference-azure-monitor-audit-log-schema.md)

* [Interpretación del esquema de registros de inicio de sesión en Azure Monitor](./reference-azure-monitor-sign-ins-log-schema.md)

 * [Transmisión en secuencias de los registros de Azure AD a Azure Event Hubs](./tutorial-azure-monitor-stream-logs-to-event-hub.md)

* [Integración de registros de Azure AD con Splunk mediante Azure Monitor](./howto-integrate-activity-logs-with-splunk.md)

* [Integración de registros de Azure AD con SumoLogic mediante Azure Monitor](./howto-integrate-activity-logs-with-sumologic.md)

 

 

## <a name="next-steps"></a>Pasos siguientes

Considere la posibilidad de implementar [Privileged Identity Management](../privileged-identity-management/pim-configure.md) 

Considere la posibilidad de implementar el [control de acceso basado en rol de Azure (RBAC de Azure)](../../role-based-access-control/overview.md).