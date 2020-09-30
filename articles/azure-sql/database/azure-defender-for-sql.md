---
title: Azure Defender para SQL
description: Conozca la funcionalidad para administrar los puntos vulnerables de la base de datos y detectar actividades anómalas que podrían indicar una amenaza para la base de datos en Azure SQL Database, Azure SQL Managed Instance o Azure Synapse.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.devlang: ''
ms.custom: sqldbrb=2
ms.topic: conceptual
ms.author: memildin
manager: rkarlin
author: memildin
ms.reviewer: vanto
ms.date: 09/21/2020
ms.openlocfilehash: b789cd423d2260ce709a02ffb3ac5ea500997609
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932130"
---
# <a name="azure-defender-for-sql"></a>Azure Defender para SQL
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]


Azure Defender para SQL es un paquete unificado de funcionalidades avanzadas de seguridad de SQL. Azure Defender está disponible para Azure SQL Database, Azure SQL Managed Instance y Azure Synapse Analytics. Incluye una funcionalidad para detectar y clasificar datos confidenciales, buscar y mitigar los potenciales puntos vulnerables de una base de datos y detectar actividades anómalas que puedan indicar una amenaza para dicha base de datos. Proporciona una ubicación única para habilitar y administrar estas funcionalidades.

## <a name="overview"></a>Información general

Azure Defender proporciona un conjunto de funcionalidades avanzadas de seguridad de SQL, entre las que se incluyen Evaluación de vulnerabilidad de SQL y Advanced Threat Protection.
- [Evaluación de vulnerabilidad](sql-vulnerability-assessment.md) es un servicio fácil de configurar que puede detectar, realizar el seguimiento y ayudarle a corregir vulnerabilidades de la base de datos. Permite ver el estado de la seguridad e incluye pasos interactivos para resolver problemas de seguridad y mejorar las defensas de la base de datos.
- [Advanced Threat Protection](threat-detection-overview.md) permite detectar actividades anómalas que indica intentos poco habituales y posiblemente dañinos de acceder a sus bases de datos o aprovecharse de ellas. Supervisa de forma continuada la base de datos para detectar actividades sospechosas y proporciona de forma inmediata alertas de seguridad sobre posibles vulnerabilidades, ataques por inyección de código SQL de Azure y patrones anómalos de acceso a la base de datos. Las alertas de Advanced Threat Protection proporcionan detalles de la actividad sospechosa y recomiendan acciones para investigar y mitigar la amenaza.

Si habilita Azure Defender para SQL una vez, habilitará todas estas características incluidas. Con un solo clic, puede habilitar Azure Defender para todas las bases de datos en el [servidor](logical-servers.md) en Azure o en SQL Managed Instance. Para habilitar o administrar la configuración de Azure Defender es necesario pertenecer al rol [administrador de seguridad SQL](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#sql-security-manager), o bien a uno de los roles de administrador de base de datos o de servidor.

Para obtener más información sobre los precios de Azure Defender para SQL, consulte la [página de precios de Azure Security Center](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="getting-started-with-azure-defender"></a>Introducción a Azure Defender

Estos pasos le ayudarán a empezar a usar Azure Defender.

## <a name="1-enable-azure-defender"></a>1. Habilitación de Azure Defender

Se puede acceder a Azure Defender a través de [Azure Portal](https://portal.azure.com). Para habilitar Azure Defender, vaya a **Security Center** bajo el encabezado **Seguridad** del servidor o la instancia administrada.

> [!NOTE]
> Una cuenta de almacenamiento se crea y se configura automáticamente para almacenar los resultados del examen de **evaluación de vulnerabilidades**. Si ya ha habilitado Azure Defender para otro servidor del mismo grupo de recursos y la misma región, se usa la cuenta de almacenamiento existente.
>
> El costo de Azure Defender se alinea al precio por nodo del nivel estándar de Azure Security Center, donde un nodo es la totalidad del servidor o la instancia administrada. Por tanto, se paga una sola vez por proteger con Azure Defender todas las bases de datos del servidor o la instancia administrada. Puede probar Azure Defender inicialmente con una evaluación gratuita.

## <a name="2-start-tracking-vulnerabilities-and-investigating-threat-alerts"></a>2. Inicio del seguimiento de puntos vulnerables y la investigación de alertas de amenazas

Haga clic en la tarjeta **Evaluación de vulnerabilidad** tarjeta no solo para ver y administrar informes y exámenes de vulnerabilidad, sino también realizar un seguimiento del estado de la seguridad. Si se han recibido alertas de seguridad, haga clic en la tarjeta **Advanced Threat Protection** para ver los detalles de las alertas y un informe consolidado de todas las alertas de su suscripción de Azure en la página de alertas de seguridad de Azure Security Center.

## <a name="3-manage-azure-defender-settings"></a>3. Administración de la configuración de Azure Defender

Para ver y administrar la configuración de Azure Defender, vaya a **Security Center** bajo el encabezado **Seguridad** del servidor o la instancia administrada. En esta página, puede habilitar o deshabilitar Azure Defender, y modificar la configuración de la evaluación de vulnerabilidades y Advanced Threat Protection para todo el servidor o la instancia administrada.

## <a name="4-manage-azure-defender-settings-for-a-database"></a>4. Administración de la configuración de Azure Defender en una base de datos

Para invalidar la configuración de Azure Defender en una base de datos determinada, active la casilla **Habilitar Azure Defender para SQL en el nivel de base de datos**. Use esta opción solo si tiene un requisito concreto para recibir alertas de Advanced Threat Protection o resultados de la evaluación de vulnerabilidades independientes para la base de datos individual, en lugar o además de las alertas y los resultados que recibe de todas las bases de datos en el servidor o la instancia administrada.

Una vez activada la casilla de verificación, puede configurar las opciones pertinentes para esta base de datos.

También se puede acceder a la configuración de Azure Defender para SQL del servidor o la instancia administrada desde el panel de base de datos de Azure Defender. En el panel principal de Azure Defender, haga clic en **Configuración** y, a continuación, haga clic en **Ver la configuración del servidor de Azure Defender para SQL**.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [evaluación de vulnerabilidades](sql-vulnerability-assessment.md)
- Más información sobre [Advanced Threat Protection](threat-detection-configure.md)
- Más información acerca de [Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-intro)
