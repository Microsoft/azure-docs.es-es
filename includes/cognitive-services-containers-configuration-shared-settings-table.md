---
author: diberry
ms.author: diberry
ms.service: cognitive-services
ms.topic: include
ms.date: 05/15/2019
ms.openlocfilehash: c4abc5fa89b48b6fc55637e9ff3b259387d0d410
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796392"
---
Este contenedor tiene las siguientes opciones de configuración:

|Obligatorio|Configuración|Propósito|
|--|--|--|
|Sí|[ApiKey](#apikey-configuration-setting)|Se usa para realizar un seguimiento de la información de facturación.|
|Sin |[Application Insights](#applicationinsights-setting)|Le permite agregar compatibilidad con los datos de telemetría de [Azure Application Insights](https://docs.microsoft.com/azure/application-insights) al contenedor.|
|Sí|[Facturación](#billing-configuration-setting)|Especifica el URI del punto de conexión del recurso de servicio en Azure.|
|Sí|[Eula](#eula-setting)| Indica que ha aceptado la licencia del contenedor.|
|Sin |[Fluentd](#fluentd-settings)|Escribe el registro y, opcionalmente, los datos de métricas en un servidor de Fluentd.|
|Sin |Proxy Http|Configure un proxy HTTP para hacer solicitudes salientes.|
|Sin |[Logging](#logging-settings)|Proporciona compatibilidad con el registro de ASP.NET Core al contenedor. |
|Sin |[Mounts](#mount-settings)|Leer y escribir datos desde el equipo host al contenedor y del contenedor al equipo host.|
