---
title: Registros de actividad de Azure Active Directory en Azure Monitor | Microsoft Docs
description: Introducción a los registros de actividad de Azure Active Directory en Azure Monitor
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/09/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: f4253fe52346890eaa993a18e8e9bc9b270bffd7
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229861"
---
# <a name="azure-ad-activity-logs-in-azure-monitor"></a>Registros de actividad de Azure AD en Azure Monitor

Puede enrutar los registros de actividad de Azure Active Directory (Azure AD) a varios puntos de conexión para conseguir conclusiones de datos y una retención a largo plazo. Esta característica le permite:

* Archivar los registros de actividad de Azure AD en una cuenta de almacenamiento de Azure para conservar los datos durante mucho tiempo.
* Hacer streaming de los registros de actividad de Azure AD a un centro de eventos de Azure para analizarlos con herramientas conocidas de Administración de eventos e información de seguridad (SIEM), como Splunk y QRadar.
* Integrar los registros de actividad de Azure AD con sus propias soluciones de registro personalizadas haciendo streaming a un centro de eventos.
* Envíe los registros de actividad de Azure AD a registros de Azure Monitor para permitir visualizaciones enriquecidas, supervisión y alertas sobre los datos conectados.

> [!VIDEO https://www.youtube.com/embed/syT-9KNfug8]

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="supported-reports"></a>Informes admitidos

Con esta característica, puede enrutar registros de auditoría y de inicios de sesión de Azure AD a la cuenta de almacenamiento de Azure, un centro de eventos, registros de Azure Monitor o una solución personalizada. 

* **Registros de auditoría**: el [informe de actividad de registros de auditoría](concept-audit-logs.md) le proporciona acceso al historial de todas las tareas llevadas a cabo en el inquilino.
* **Registros de inicio de sesión**: Con el [informe de actividad de inicios de sesión](concept-sign-ins.md), puede determinar quién ha realizado las tareas notificadas en el informe de registros de auditoría.

> [!NOTE]
> En este momento no se admiten los registros de actividad de auditoría e inicio de sesión relacionados con B2C.
>

## <a name="prerequisites"></a>Prerrequisitos

Para usar esta característica, necesita:

* Suscripción a Azure. Si no tiene ninguna suscripción de Azure, puede [registrarse para obtener una evaluación gratuita](https://azure.microsoft.com/free/).
* Una [licencia](https://azure.microsoft.com/pricing/details/active-directory/) Azure Active Directory Free, Basic, Premium 1 o Premium 2 para acceder a los registros de auditoría de Azure AD en Azure Portal. 
* Un inquilino de Azure AD.
* Un usuario que sea **administrador global** o **administrador de seguridad** para el inquilino de Azure AD.
* Una [licencia](https://azure.microsoft.com/pricing/details/active-directory/) Azure AD Premium 1 o Premium 2 para acceder a los registros de inicios de sesión de Azure AD en Azure Portal. 

Según de hacia dónde desee enrutar los datos de registro de auditoría, necesitará algo de lo siguiente:

* Una cuenta de almacenamiento de Azure, en la que tenga permisos *ListKeys*. Le recomendamos utilizar una cuenta de almacenamiento general y no de almacenamiento de blobs. Para más información sobre precios de almacenamiento, consulte la [Calculadora de precios de Azure Storage](https://azure.microsoft.com/pricing/calculator/?service=storage). 
* Un espacio de nombres de Azure Event Hubs para la integración con soluciones de terceros.
* Un área de trabajo de Azure Log Analytics para enviar registros a registros de Azure Monitor.

## <a name="cost-considerations"></a>Consideraciones sobre el costo

Si ya tiene una licencia de Azure AD, necesita una suscripción a Azure para configurar la cuenta de almacenamiento y el centro de eventos. La suscripción a Azure es gratuita, pero tiene que pagar para utilizar los recursos de Azure, incluida la cuenta de almacenamiento que se usa para archivar y el centro de eventos que se utiliza para el streaming. La cantidad de datos, y por lo tanto el costo incurrido, puede variar significativamente dependiendo del tamaño del inquilino. 

### <a name="storage-size-for-activity-logs"></a>Tamaño de almacenamiento para registros de actividad

Cada evento del registro de auditoría utiliza aproximadamente 2 KB de almacenamiento de datos. Los registros de eventos de inicio de sesión tienen aproximadamente 4 KB de almacenamiento de datos. Para un inquilino con 100 000 usuarios, que incurriría en alrededor de 1,5 millones de eventos por día, necesitaría alrededor de 3 GB de almacenamiento de datos al día. Dado que las escrituras se producen en lotes de aproximadamente cinco minutos, puede anticipar cerca de 9 000 operaciones de escritura al mes. 


La siguiente tabla contiene una estimación del costo, dependiendo del tamaño del inquilino, de una cuenta de almacenamiento de uso general v2 en la región Oeste de EE. UU. durante al menos un año de retención. Para crear una estimación más precisa del volumen de datos que prevé para la aplicación, use la [calculadora de precios de almacenamiento de Azure](https://azure.microsoft.com/pricing/details/storage/blobs/).


| Categoría del registro | Número de usuarios | Eventos por día | Volumen de datos al mes (aprox.) | Costo al mes (aprox.) | Costo al año (aprox.) |
|--------------|-----------------|----------------------|--------------------------------------|----------------------------|---------------------------|
| Auditoría | 100 000 | 1,5&nbsp;millones | 90 GB | 1,93 USD | 23,12 USD |
| Auditoría | 1,000 | 15,000 | 900 MB | 0,02 USD | 0,24 USD |
| Inicios de sesión | 1,000 | 34 800 | 4 GB | 0,13 USD | 1,56 USD |
| Inicios de sesión | 100 000 | 15&nbsp;millones | 1,7 TB | 35,41 USD | 424,92 USD |
 









### <a name="event-hub-messages-for-activity-logs"></a>Mensajes del centro de eventos para los registros de actividad

Los eventos se agrupan en intervalos de aproximadamente cinco minutos y se envían como un único mensaje que contiene todos los eventos dentro de ese período de tiempo. Un mensaje en el centro de eventos tiene un tamaño máximo de 256 KB y, si el tamaño total de todos los mensajes dentro del período de tiempo excede ese volumen, se envían múltiples mensajes. 

Por ejemplo, unos 18 eventos por segundo se producen normalmente para un inquilino de gran tamaño de más de 100 000 usuarios, una tasa que equivale a 5 400 eventos cada cinco minutos. Dado que los registros de auditoría son de aproximadamente 2 KB por evento, esto equivale a 10,8 MB de datos. Por lo tanto, se envían 43 mensajes al centro de eventos en ese intervalo de cinco minutos. 

En la tabla siguiente se incluyen los costos estimados al mes para un centro de eventos básico en la región Oeste de EE. UU., según el volumen de datos de eventos que pueden variar de un inquilino a otro en función de muchos factores, como el comportamiento de inicio de sesión de usuario, etc. Para calcular una estimación precisa del volumen de datos previstos para la aplicación, use la [calculadora de precios de Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

| Categoría del registro | Número de usuarios | Eventos por segundo | Eventos por intervalo de cinco minutos | Volumen por intervalo | Mensajes por intervalo | Mensajes al mes | Costo al mes (aprox.) |
|--------------|-----------------|-------------------------|----------------------------------------|---------------------|---------------------------------|------------------------------|----------------------------|
| Auditoría | 100 000 | 18 | 5 400 | 10,8 MB | 43 | 371,520 | 10,83 USD |
| Auditoría | 1,000 | 0,1 | 52 | 104 KB | 1 | 8 640 | 10,80 USD |
| Inicios de sesión | 100 000 | 18000 | 5 400 000 | 10,8 GB | 42188 | 364 504 320 | 23,9 USD |  
| Inicios de sesión | 1,000 | 178 | 53 400 | 106,8&nbsp;MB | 418 | 3 611 520 | 11,06 USD |  

### <a name="azure-monitor-logs-cost-considerations"></a>Consideraciones acerca del costo de registros de Azure Monitor



| Categoría del registro       | Número de usuarios | Eventos por día | Eventos al mes (30 días) | Costo por mes en USD (est.) |
| :--                | ---             | ---            | ---                        | --:                          |
| Auditoría e inicios de sesión | 100 000         | 16 500 000     | 495 000 000                |  1093,00 USD                       |
| Auditoría              | 100 000         | 1 500 000      | 45 000 000                 |  246,66 USD                     |
| Inicios de sesión           | 100 000         | 15 000 000     | 450 000 000                |  847,28 USD                     |










Para revisar los costos relacionados con la administración de registros de Azure Monitor, consulte [Administración de los costos mediante el control del volumen de datos y la retención en registros de Azure Monitor](../../azure-monitor/platform/manage-cost-storage.md).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

En esta sección se responden las preguntas más frecuentes y se abordan los problemas conocidos con los registros de Azure AD en Azure Monitor.

**P: ¿Qué registros se incluyen?**

**R.** : Los registros de actividad de inicio de sesión y los registros de auditoría se pueden enrutar mediante esta característica, aunque actualmente no se incluyen los eventos de auditoría relacionados con B2C. Para averiguar qué tipos de registros y qué registros basados en características se admiten en la actualidad, consulte el [esquema de registro de auditoría](reference-azure-monitor-audit-log-schema.md) y el [esquema de registro de inicio de sesión](reference-azure-monitor-sign-ins-log-schema.md). 

---

**P: Después de realizar una acción, ¿cuánto tiempo pasará antes de que se muestren los registros correspondientes en el centro de eventos?**

**R.** : Los registros deben aparecer en el centro de eventos de dos a cinco minutos después de realizada la acción. Para más información sobre Event Hubs, consulte [¿Qué es Azure Event Hubs?](../../event-hubs/event-hubs-about.md)

---

**P: Después de realizar una acción, ¿cuánto tiempo pasará antes de que se muestren los registros correspondientes en las cuentas de almacenamiento?**

**R.** : Para las cuentas de almacenamiento de Azure, la latencia está entre 5 y 15 minutos tras realizar la acción.

---

**P: ¿Qué ocurre si un administrador cambia el período de retención de una configuración de diagnóstico?**

**R.** : La nueva directiva de retención se aplicará a los registros recopilados después del cambio. Los registros que se hayan recopilado antes del cambio de directiva no se verán afectados.

---

**P: ¿Cuánto costará almacenar mis datos?**

**R.** : El costo de almacenamiento depende tanto del tamaño de sus registros como del período de retención que elija. Para ver una lista de los costos estimados para los inquilinos, que dependen del volumen de registros generados, consulte la sección [Tamaño de almacenamiento de los registros de actividad](#storage-size-for-activity-logs).

---

**P: ¿Cuánto costará hacer streaming de mis datos a un centro de eventos?**

**R.** : El costo del streaming depende del número de mensajes que reciba por minuto. En este artículo se describe cómo se calculan los costos y se enumeran las estimaciones de costos, que se basan en el número de mensajes. 

---

**P: ¿Cómo integro los registros de actividad de Azure AD con mi sistema SIEM?**

**R.** : Puede hacerlo de dos maneras:

- Use Azure Monitor con Event Hubs para hacer streaming de los registros en el sistema SIEM. En primer lugar, [haga streaming de los registros a un centro de eventos](tutorial-azure-monitor-stream-logs-to-event-hub.md) y, después, [configure la herramienta de SIEM](tutorial-azure-monitor-stream-logs-to-event-hub.md#access-data-from-your-event-hub) con el centro de eventos configurado. 

- Use [Reporting Graph API](concept-reporting-api.md) para acceder a los datos e insertarlos en el sistema SIEM mediante sus propios scripts.

---

**P: ¿Qué herramientas SIEM se admiten actualmente?** 

**R.** : **R.** : Actualmente, Azure Monitor es compatible con [Splunk](./howto-integrate-activity-logs-with-splunk.md), IBM QRadar, [Sumo Logic](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory), [ArcSight](./howto-integrate-activity-logs-with-arcsight.md), LogRhythm y Logz.io. Para más información sobre cómo funcionan los conectores, consulte [Flujo de datos de supervisión de Azure a un centro de eventos para que lo consuma una herramienta externa](../../azure-monitor/platform/stream-monitoring-data-event-hubs.md).

---

**P: ¿Cómo se integran los registros de actividad de Azure AD con mi instancia de Splunk?**

**R.** : En primer lugar, [enrute los registros de actividad de Azure AD a un centro de eventos](./tutorial-azure-monitor-stream-logs-to-event-hub.md) y, después, siga los pasos necesarios para [integrar los registros de actividad con Splunk](./howto-integrate-activity-logs-with-splunk.md).

---

**P: ¿Cómo se integran los registros de actividad de Azure AD con Sumo Logic?** 

**R.** : En primer lugar, [enrute los registros de actividad de Azure AD a un centro de eventos](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Collect_Logs_for_Azure_Active_Directory) y, después, siga los pasos necesarios para [instalar la aplicación de Azure AD y ver los paneles en SumoLogic](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Install_the_Azure_Active_Directory_App_and_View_the_Dashboards).

---

**P: ¿Puedo acceder a los datos desde un centro de eventos sin utilizar una herramienta SIEM externa?** 

**R.** : Sí. Para acceder a los registros desde la aplicación personalizada, puede usar la [API de Event Hubs](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md). 

---


## <a name="next-steps"></a>Pasos siguientes

* [Archivado de registros de actividad en una cuenta de almacenamiento](quickstart-azure-monitor-route-logs-to-storage-account.md)
* [Enrutamiento de registros de actividad a un centro de eventos](./tutorial-azure-monitor-stream-logs-to-event-hub.md)
* [Integración de registros de actividad con Azure Monitor](howto-integrate-activity-logs-with-log-analytics.md)