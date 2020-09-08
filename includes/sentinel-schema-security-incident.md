---
title: archivo de inclusión
description: archivo de inclusión
services: azure-sentinel
author: yelevin
ms.service: azure-sentinel
ms.topic: include
ms.date: 06/28/2020
ms.author: yelevin
ms.custom: include file
ms.openlocfilehash: 63cb53dc60a718892d4bf86140e7fd51303bd61c
ms.sourcegitcommit: f1b18ade73082f12fa8f62f913255a7d3a7e42d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88761733"
---
### <a name="the-data-model-of-the-schema"></a>El modelo de datos del esquema

| Campo | Tipo de datos | Descripción |
| ---- | ---- | ---- |
| **AdditionalData** | dinámico | Recuento de alertas, recuento de marcadores, recuento de comentarios, tácticas y nombres de productos de alerta |
| **AlertIds** | dinámico | Alertas a partir de las que se creó el incidente |
| **BookmarkIds** | dinámico | Entidades marcadas |
| **Clasificación** | string | Clasificación de cierre de incidentes |
| **ClassificationComment** | string | Comentario de clasificación de cierre de incidentes |
| **ClassificationReason** | string | Motivo de clasificación de cierre de incidentes |
| **ClosedTime** | datetime | Marca de tiempo (UTC) de la última vez que se cerró el incidente |
| **Comentarios** | dinámico | Comentarios sobre el incidente |
| **CreatedTime** | datetime | Marca de tiempo (UTC) de cuando se creó el incidente |
| **Descripción** | cadena | Descripción del incidente |
| **FirstActivityTime** | datetime | Hora del primer evento |
| **FirstModifiedTime** | datetime | Marca de tiempo (UTC) de la primera vez que se modificó el incidente |
| **IncidentName** | string | GUID interno |
| **IncidentNumber** | int |  |
| **IncidentUrl** | string | Vínculo al incidente |
| **Etiquetas** | dinámico | Etiquetas |
| **LastActivityTime** | datetime | Hora del último evento |
| **LastModifiedTime** | datetime | Marca de tiempo (UTC) de la última vez que se modificó el incidente <br>(la modificación descrita por el registro actual) |
| **ModifiedBy** | string | Usuario o sistema que modificó el incidente |
| **Propietario** | dinámico |  |
| **RelatedAnalyticRuleIds** | dinámico | Reglas desde las que se desencadenaron las alertas del incidente |
| **Gravedad** | string | Gravedad del incidente (Alta/Media/Baja/Informativa) |
| **SourceSystem** | string | Constante ("Azure") |
| **Estado** | string |  |
| **TenantId** | string |  |
| **TimeGenerated** | datetime | Marca de tiempo (UTC) de cuando se creó el registro actual <br>(tras la modificación del incidente) |
| **Título** | string | 
| **Tipo** | string | Constante ("SecurityIncident") |
|
