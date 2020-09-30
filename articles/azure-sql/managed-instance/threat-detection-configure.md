---
title: Configuración de Advanced Threat Protection
titleSuffix: Azure SQL Managed Instance
description: Advanced Threat Protection detecta actividades anómalas en la base de datos que indican posibles amenazas de seguridad para la base de datos en Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto
ms.date: 09/21/2020
ms.openlocfilehash: d81550f5c6906e409675002fde670db3734de883
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885056"
---
# <a name="configure-advanced-threat-protection-in-azure-sql-managed-instance"></a>Configuración de Advanced Threat Protection en Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

[Advanced Threat Protection](../database/threat-detection-overview.md) para una [Instancia administrada de Azure SQL](sql-managed-instance-paas-overview.md) detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de acceder a las bases de datos o de vulnerar su seguridad. Advanced Threat Protection puede identificar una **posible inyección de código SQL**, **acceso desde un centro de datos o una ubicación inusuales**, **acceso desde una aplicación potencialmente peligrosa o entidad de seguridad desconocida** y **credenciales SQL por fuerza bruta**; consulte más detalles en [Alertas de Advanced Threat Protection](../database/threat-detection-overview.md#alerts).

Puede recibir notificaciones sobre las amenazas detectadas mediante [notificaciones por correo electrónico](../database/threat-detection-overview.md#explore-detection-of-a-suspicious-event) o en [Azure Portal](../database/threat-detection-overview.md#explore-alerts-in-the-azure-portal).

[Advanced Threat Protection](../database/threat-detection-overview.md) forma parte de la oferta [Azure Defender for SQL](../database/azure-defender-for-sql.md), que es un paquete unificado de funcionalidades de seguridad avanzadas de SQL. Se puede acceder a Advanced Threat Protection y administrarlo a través del portal central de Azure Defender for SQL.

##  <a name="azure-portal"></a>Azure portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. Vaya a la página de configuración de la Instancia administrada de SQL que quiera proteger. En **Seguridad**, seleccione **Security Center**.
3. En la página de configuración de Azure Defender for SQL:
   - **Active** Azure Defender for SQL.
   - Configure la **lista de direcciones de correo electrónico** que recibirán alertas de seguridad cuando se detecten actividades anómalas en las bases de datos.
   - Seleccione la **cuenta de Azure Storage** donde se guardan los registros de anomalías de la auditoría de amenazas.
   - Seleccione los **tipos de Advanced Threat Protection** que quiere configurar. Más información sobre las [alertas de Advanced Threat Protection](../database/threat-detection-overview.md).
4. Haga clic en **Guardar** para guardar la directiva de Azure Defender for SQL nueva o actualizada.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Advanced Threat Protection](../database/threat-detection-overview.md).
- Más información sobre las instancias administradas en [¿Qué es una Instancia administrada de Azure SQL?](sql-managed-instance-paas-overview.md)
- Más información sobre [Advanced Threat Protection para Azure SQL Database](../database/threat-detection-configure.md)
- Más información sobre la [auditoría de Instancia administrada de SQL](https://go.microsoft.com/fwlink/?linkid=869430)
- Más información sobre [Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-intro).
