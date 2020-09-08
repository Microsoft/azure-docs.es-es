---
title: Uso de Azure AD Connect Health con AD DS | Microsoft Docs
description: Esta es la página de Azure AD Connect Health donde se describe cómo supervisar la sincronización de AD DS.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: daveba
editor: curtand
ms.assetid: 19e3cf15-f150-46a3-a10c-2990702cd700
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/18/2017
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d75b439c2753db17cffecdc97b6389a7b31b833e
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279880"
---
# <a name="using-azure-ad-connect-health-with-ad-ds"></a>Uso de Azure AD Connect Health con AD DS
La siguiente documentación es específica de la supervisión de Active Directory Domain Services con Azure AD Connect Health. Las versiones compatibles de AD DS son: Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2 y Windows Server 2016.

Para más información sobre la supervisión de AD FS con Azure AD Connect Health, consulte [Uso de Azure AD Connect Health con AD FS](how-to-connect-health-adfs.md). Para obtener información adicional sobre la supervisión de Azure AD Connect (Sync) con Azure AD Connect Health, consulte [Uso de Azure AD Connect Health para sincronización](how-to-connect-health-sync.md).

![Azure AD Connect Health para AD DS](./media/how-to-connect-health-adds/domainservicesnapshot.PNG)

## <a name="alerts-for-azure-ad-connect-health-for-ad-ds"></a>Alertas de Azure AD Connect Health para AD DS
La sección Alertas de Azure AD Connect Health para AD DS le proporciona una lista de alertas activas y resueltas relacionadas con los controladores de dominio. Si selecciona una alerta activa o resuelta se abrirá una nueva hoja con información adicional, junto con los pasos para su resolución y vínculos a documentación adicional. Cada tipo de alerta puede tener una o más instancias que se corresponden con cada uno de los controladores de dominio afectados por esa alerta en concreto. En la parte inferior de la hoja de la alerta, puede hacer doble clic sobre un controlador de dominio afectado para abrir una hoja adicional con más detalles sobre esa instancia de alerta.

En esta hoja, puede habilitar las notificaciones por correo electrónico para las alertas y cambiar el intervalo de tiempo en la vista. Si aumenta el intervalo de tiempo se le permitirá ver alertas resueltas anteriores.

![Error de sincronización de Azure AD Connect](./media/how-to-connect-health-adds/aadconnect-health-adds-alerts.png)

## <a name="domain-controllers-dashboard"></a>Panel de controladores de dominio
Este panel proporciona una vista topológica del entorno, junto con métricas clave de funcionamiento y el estado de mantenimiento de cada uno de los controladores de dominio supervisados. Las métricas presentadas ayudan a identificar rápidamente los controladores de dominio que podrían requerir una mayor investigación. De forma predeterminada, se muestra solo un subconjunto de las columnas. No obstante, puede encontrar todo el conjunto de columnas disponibles haciendo doble clic en el comando de columnas. Poder seleccionar las columnas que más le interesan convierte a este panel en un lugar único y sencillo para ver el mantenimiento del entorno de AD DS.

![Controladores de dominio](./media/how-to-connect-health-adds/aadconnect-health-adds-domainsandsites-dashboard.png)

Los controladores de dominio se pueden agrupar según su respectivo dominio o sitio, lo cual resulta útil para entender la topología del entorno. Por último, si hace doble clic en el encabezado de la hoja, se maximizará el panel para utilizar el estado real disponible de la pantalla. Esta vista más grande es útil cuando se muestran varias columnas.

## <a name="replication-status-dashboard"></a>Panel del estado de replicación
Este panel proporciona una vista del estado y la topología de la replicación de los controladores de dominio supervisados. Se muestra el estado del intento de replicación más reciente, junto con documentación útil para cualquier error encontrado. Haga doble clic en un controlador de dominio con un error para que se abra una nueva hoja con información como: detalles acerca del error, pasos de resolución recomendados y vínculos a documentación de solución de problemas.

![Estado de replicación](./media/how-to-connect-health-adds/aadconnect-health-adds-replication.png)

## <a name="monitoring"></a>Supervisión
Esta característica proporciona tendencias gráficas de diferentes contadores de rendimiento que se recopilan continuamente desde cada uno de los controladores de dominio supervisados. El rendimiento de un controlador de dominio se puede comparar fácilmente con el de todos los demás controladores de dominio supervisados del bosque. Además, puede ver varios contadores de rendimiento en paralelo, lo cual resulta útil al solucionar problemas en su entorno.

![Supervisión](./media/how-to-connect-health-adds/aadconnect-health-adds-monitoring.png)

De forma predeterminada, hemos preseleccionado cuatro contadores de rendimiento. Sin embargo, puede incluir otros haciendo clic en el comando de filtros y seleccionando o anulando la selección de cualquier contador de rendimiento que desee. Además, puede hacer doble clic en un gráfico del contador de rendimiento para que se abra una nueva hoja que incluye los puntos de datos correspondientes para cada uno de los controladores de dominio supervisados.

## <a name="related-links"></a>Vínculos relacionados
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Instalación del agente de Azure AD Connect Health](how-to-connect-health-agent-install.md)
* [Operaciones de Azure AD Connect Health](how-to-connect-health-operations.md)
* [Uso de Azure AD Connect Health con AD FS](how-to-connect-health-adfs.md)
* [Uso de Azure AD Connect Health para sincronización](how-to-connect-health-sync.md)
* [Preguntas más frecuentes de Azure AD Connect Health](reference-connect-health-faq.md)
* [Historial de versiones de Azure AD Connect Health](reference-connect-health-version-history.md)