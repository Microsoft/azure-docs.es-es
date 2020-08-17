---
title: Análisis de usuarios, sesiones y eventos en Azure Application Insights
description: Este artículo trata sobre el análisis de los usuarios de su aplicación web.
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 01/24/2018
ms.reviewer: mbullwin
ms.openlocfilehash: d11b12ba37d543ec21985c52c4ffb3399bfc56d1
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323526"
---
# <a name="users-sessions-and-events-analysis-in-application-insights"></a>Análisis de usuarios, sesiones y eventos en Application Insights

Descubra cuándo utilizan los usuarios la aplicación web, en qué páginas están más interesados, en qué ubicación se encuentran dichos usuarios, y los sistemas operativos y exploradores que emplean. Analice la telemetría de uso y de negocio con [Azure Application Insights](./app-insights-overview.md).

![Captura de pantalla de usuarios de Application Insights](./media/usage-segmentation/0001-users.png)

## <a name="get-started"></a>Introducción

Si aún no ve los datos en las hojas de usuarios, sesiones o eventos de Application Insights, [obtenga información sobre cómo empezar a trabajar con las herramientas de uso](usage-overview.md).

## <a name="the-users-sessions-and-events-segmentation-tool"></a>La herramienta de segmentación de usuarios, sesiones y eventos

Tres de las hojas de uso usan la misma herramienta para segmentar y desglosar telemetría de su aplicación web desde tres perspectivas. Mediante el filtrado y la división de los datos, puede descubrir información detallada sobre el uso relativo de distintas páginas y características.

* **Herramienta Usuarios**: averigüe cuántas personas usan la aplicación y sus características.  Los usuarios se cuentan mediante el uso de identificadores anónimos almacenados en las cookies del explorador. Una sola persona que usa distintos exploradores o máquinas se contarán como más de un usuario.
* **Herramienta Sesiones**: la cantidad de sesiones de actividad de usuario que han incluido determinadas páginas y características de la aplicación. Una sesión se cuenta después de media hora de inactividad del usuario o después de 24 horas de uso continuo.
* **Herramienta Eventos**: la frecuencia con la que se usan ciertas páginas y características de la aplicación. Una vista de página se cuenta cuando un explorador carga una página de la aplicación, siempre que se haya [instrumentado](./javascript.md). 

    Un evento personalizado representa una repetición de algo que sucede en la aplicación, a menudo, una interacción del usuario como un clic en el botón o la realización de alguna tarea. Inserte código en su aplicación para [generar eventos personalizados](./api-custom-events-metrics.md#trackevent).

## <a name="querying-for-certain-users"></a>Consulta de determinados usuarios

Explore distintos grupos de usuarios mediante ajustando las opciones de consulta en la parte superior de la herramienta Usuarios:

* Mostrar: elija una cohorte de usuarios que se va a analizar.
* Que usaron: elija eventos personalizados y vistas de página.
* Durante: elija un intervalo de tiempo.
* Por: elija cómo se desglosarán los datos, por un periodo u otra propiedad como un explorador o una ciudad.
* Dividir por: elija una propiedad por la que dividir o segmentar los datos. 
* Agregar filtros: limite la consulta a determinados usuarios, sesiones o eventos en función de sus propiedades, como un explorador o una ciudad. 
 
## <a name="saving-and-sharing-reports"></a>Guardado e intercambio de informes 
Puede guardar los informes de Usuarios, de forma privada (solo para usted) en la sección Mis informes, o bien con todos aquellos que tengan acceso a este recurso de Application Insights en la sección Informes compartidos.

Para compartir un vínculo con un informe de usuarios, sesiones o eventos, haga clic en **Recurso compartido** en la barra de herramientas y, después, copie el vínculo.

Para compartir una copia de los datos de un informe de usuarios, sesiones o eventos, haga clic en **Recurso compartido** en la barra de herramientas y, después, en el **icono Word** para crear un documento de Word con los datos. O bien, haga clic en el **icono Word** encima del gráfico principal.

## <a name="meet-your-users"></a>Conozca a sus usuarios

La sección **Meet your users** (Conozca a sus usuarios) muestra información acerca de los cinco usuarios de ejemplo coincidentes con la consulta actual. Al tener en cuenta y explorar los comportamientos de los usuarios, además de los agregados, se puede proporcionar información sobre cómo se utiliza realmente la aplicación.

## <a name="next-steps"></a>Pasos siguientes

- Para habilitar las experiencias de uso, empiece por enviar [eventos personalizados](./api-custom-events-metrics.md#trackevent) o [vistas de páginas](./api-custom-events-metrics.md#page-views).
- Si ya ha enviado eventos personalizados o vistas de página, explore las herramientas de uso para obtener información sobre cómo los usuarios utilizan el servicio.
    - [Embudos](usage-funnels.md)
    - [Retención](usage-retention.md)
    - [Flujos de usuario](usage-flows.md)
    - [Libros](../platform/workbooks-overview.md)
    - [Adición de contexto de usuario](usage-send-user-context.md)

