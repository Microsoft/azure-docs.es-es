---
title: Eliminación de Azure Sentinel | Microsoft Docs
description: Eliminación de la instancia de Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/16/2020
ms.author: yelevin
ms.openlocfilehash: f9c400b55b0da47495db4f1ff4ceb86aa39fe2cc
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885842"
---
# <a name="remove-azure-sentinel-from-your-workspace"></a>Eliminación de Azure Sentinel del área de trabajo

Si ya no quiere usar Azure Sentinel, en este artículo se explica cómo eliminarlo del área de trabajo.

## <a name="how-to-remove-azure-sentinel"></a>Eliminación de Azure Sentinel

Siga este proceso para eliminar Azure Sentinel del área de trabajo:

1. Vaya a **Azure Sentinel**, **Configuración** y seleccione la pestaña **Remove Azure Sentinel** (Eliminar Azure Sentinel).

1. Antes de eliminar Azure Sentinel, use las casillas para indicarnos el motivo de la eliminación.

1. Seleccione **Eliminación de Azure Sentinel del área de trabajo**.
    
    ![Eliminar la solución SecurityInsights](media/offboard/delete-solution.png)

## <a name="what-happens-behind-the-scenes"></a>¿Qué sucede en segundo plano?

Al eliminar la solución, Azure Sentinel tarda hasta 48 horas en completar la primera fase del proceso de eliminación.

Una vez identificada la desconexión, comienza el proceso de retirada.

**Se elimina la configuración de estos conectores:**
-   Office 365

-   AWS

-   Alertas de seguridad de servicios de Microsoft: Microsoft Defender for Identity (*anteriormente Azure ATP*), Microsoft Cloud App Security, incluidos la generación de informes de Cloud Discovery Shadow IT, Azure AD Identity Protection, Microsoft Defender para Endpoint (*anteriormente Microsoft Defender ATP*), alertas de Azure Defender de Azure Security Center

-   Información sobre amenazas

-   Registros de seguridad habituales (incluidos los registros basados en CEF, Barracuda y Syslog) (si obtiene las alertas de Azure Defender a partir de Azure Security Center, se seguirán recopilando estos registros).

-   Eventos de seguridad de Windows (si obtiene las alertas de Azure Defender a partir de Azure Security Center, se seguirán recopilando estos registros).

Durante las primeras 48 horas, los datos y las reglas de análisis (incluida la configuración de la automatización en tiempo real) dejarán de ser accesibles o consultables en Azure Sentinel.

**Después de 30 días, se eliminarán estos recursos:**

-   Incidentes (incluidos los metadatos de investigación)

-   Reglas de análisis

-   Marcadores

Los cuadernos de estrategias, libros guardados, consultas de búsqueda guardadas y cuadernos no se eliminan. **Algunos pueden generar errores debido a los datos que se han quitado. Deberá eliminarlos manualmente.**

Después de quitar el servicio, hay un período de gracia de 30 días durante el cual puede volver a habilitar la solución y se restaurarán los datos y las reglas de análisis, pero los conectores configurados que se desconectaron deberán volver a conectarse.

> [!NOTE]
> Si quita la solución, la suscripción se seguirá registrando con el proveedor de recursos de Azure Sentinel. **Puede eliminarlos manualmente.**




## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a eliminar el servicio Azure Sentinel. Si cambia de opinión y desea volver a instalarlo:
- Introducción a la [incorporación de Azure Sentinel](quickstart-onboard.md).
