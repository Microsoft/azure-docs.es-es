---
title: Configuración de Advanced Threat Protection
description: Advanced Threat Protection detecta actividades anómalas en la base de datos que indican posibles amenazas de seguridad para la base de datos en Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-dt-2019, sqldbrb=1
ms.topic: conceptual
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto
ms.date: 09/21/2020
ms.openlocfilehash: 19da4581ef44d7c7cc51eac839c7db018793e557
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885122"
---
# <a name="configure-advanced-threat-protection-for-azure-sql-database"></a>Configuración de Advanced Threat Protection para Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[Advanced Threat Protection](threat-detection-overview.md) para Azure SQL Database detecta actividades anómalas que indican intentos inusuales y potencialmente dañinos de acceso o ataque a las bases de datos. Advanced Threat Protection puede identificar una **posible inyección de código SQL**, **acceso desde un centro de datos o una ubicación inusuales**, **acceso desde una aplicación potencialmente peligrosa o entidad de seguridad desconocida** y **credenciales SQL por fuerza bruta**; consulte más detalles en [Alertas de Advanced Threat Protection](threat-detection-overview.md#alerts).

Puede recibir notificaciones sobre las amenazas detectadas mediante [notificaciones por correo electrónico](threat-detection-overview.md#explore-detection-of-a-suspicious-event) o en [Azure Portal](threat-detection-overview.md#explore-alerts-in-the-azure-portal).

[Advanced Threat Protection](threat-detection-overview.md) forma parte de la oferta [Azure Defender for SQL](azure-defender-for-sql.md), que es un paquete unificado de funcionalidades de seguridad avanzadas de SQL. Se puede acceder a Advanced Threat Protection y administrarlo a través del portal central de Azure Defender for SQL.

## <a name="set-up-advanced-threat-protection-in-the-azure-portal"></a>Configuración de Advanced Threat Protection en el Azure Portal

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
2. Vaya a la página de configuración del servidor que quiera proteger. En la configuración de seguridad, seleccione **Security Center**.
3. En la página **Azure Defender**:

   - Habilite Azure Defender en el servidor.
   - En **Configuración de Advanced Threat Protection**, en el cuadro de texto **Enviar alertas a**, proporcione la lista de correos electrónicos para recibir alertas de seguridad cuando se detecten actividades anómalas en la base de datos.

## <a name="set-up-advanced-threat-protection-using-powershell"></a>Configuración de Advanced Threat Protection mediante PowerShell

Para ver un script de ejemplo, consulte [Configuración de auditoría y Advanced Threat Protection mediante PowerShell](scripts/auditing-threat-detection-powershell-configure.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Advanced Threat Protection](threat-detection-overview.md).
- Más información sobre [Advanced Threat Protection en Instancia administrada de SQL](../managed-instance/threat-detection-configure.md)  
- Más información sobre [Azure Defender for SQL](azure-defender-for-sql.md).
- Más información sobre las [auditorías](../../azure-sql/database/auditing-overview.md)
- Más información sobre [Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-intro)
- Para más información sobre los precios, consulte la [página Precios de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/)  
