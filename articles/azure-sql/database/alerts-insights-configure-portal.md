---
title: Configuración de alertas y notificaciones (Azure Portal)
description: Use Azure Portal para crear alertas, que pueden desencadenar notificaciones o automatización cuando se cumplen las condiciones especificadas.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: aamalvea
ms.author: aamalvea
ms.reviewer: jrasnik, carlrab
ms.date: 05/04/2020
ms.openlocfilehash: 2b7457ad5870e31d682f8727901795c9d6d73f06
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89442690"
---
# <a name="create-alerts-for-azure-sql-database-and-azure-synapse-analytics-using-the-azure-portal"></a>Creación de alertas para Azure SQL Database y Azure Synapse Analytics mediante Azure Portal
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]


## <a name="overview"></a>Información general

En este artículo se muestra cómo configurar alertas para bases de datos en Azure SQL Database y Azure Synapse Analytics (anteriormente SQL Data Warehouse) desde Azure Portal. Las alertas pueden enviarle un correo electrónico o llamar a un webhook cuando alguna métrica (por ejemplo, el tamaño de la base de datos o el uso de la CPU) alcanza el umbral.

> [!NOTE]
> Para obtener instrucciones específicas para Instancia administrada de Azure SQL, consulte [Creación de alertas para Instancia administrada de Azure SQL](../managed-instance/alerts-create.md).

Puede recibir una alerta basada en las métricas de supervisión para los servicios de Azure o los eventos sobre ellos.

* **Valores de métrica** : la alerta se desencadena cuando el valor de una métrica específica cruza un umbral asignado en cualquier dirección. Es decir, se desencadena tanto la primera vez que se cumple la condición como después, cuando dicha condición ya deja de cumplirse.
* **Eventos de registro de actividades** : una alerta puede desencadenarse en *cada* evento o solo cuando se produce una serie de eventos.

Puede configurar una alerta para hacer lo siguiente cuando se desencadena:

* Enviar notificaciones de correo electrónico al administrador y los coadministradores del servicio.
* Enviar un correo electrónico a direcciones de correo electrónico adicionales que especifique.
* Llamar a un webhook

Puede obtener información sobre las reglas de alerta y configurarlas mediante:

* [Portal de Azure](../../azure-monitor/platform/alerts-classic-portal.md)
* [PowerShell](../../azure-monitor/platform/alerts-classic-portal.md)
* [Interfaz de la línea de comandos (CLI)](../../azure-monitor/platform/alerts-classic-portal.md)
* [API de REST de Azure Monitor](https://msdn.microsoft.com/library/azure/dn931945.aspx)

## <a name="create-an-alert-rule-on-a-metric-with-the-azure-portal"></a>Creación de una regla de alerta de una métrica con Azure Portal

1. En el [portal](https://portal.azure.com/), busque el recurso que desea supervisar y selecciónelo.
2. Seleccione **Alertas** en la sección Supervisión. El texto y el icono pueden variar ligeramente en los distintos recursos.  

   ![Supervisión](./media/alerts-insights-configure-portal/Alerts.png)
  
3. Seleccione el botón **New alert rule** (Nueva regla de alertas) para abrir la página **Create rule** (Crear regla).
  ![Creación de una regla](./media/alerts-insights-configure-portal/create-rule.png)

4. En la sección **Condition** (Condición), haga clic en **Add** (Agregar).
  ![Definir condición](./media/alerts-insights-configure-portal/create-rule.png)
5. En la página **Configurar lógica de señal**, seleccione una señal.
  ![Selección de una señal](./media/alerts-insights-configure-portal/select-signal.png)
6. Después de seleccionar una señal, como **Porcentaje de CPU**, aparece la página **Configurar lógica de señal**.
  ![Configurar lógica de señal](./media/alerts-insights-configure-portal/configure-signal-logic.png)
7. En esta página, configure el tipo de umbral, operador, tipo de agregación, valor de umbral, granularidad de agregación y frecuencia de evaluación. A continuación, haga clic en **Hecho**.
8. En **Create rule** (Crear regla), seleccione un **grupo de acciones** existente o créelo. Un grupo de acciones le permite definir la acción que se va a realizar cuando se produce una condición de alerta.
  ![Definir un grupo de acciones](./media/alerts-insights-configure-portal/action-group.png)

9. Defina un nombre para la regla, especifique una descripción opcional, elija el nivel de gravedad de la regla, elija si desea habilitar la regla tras su creación y haga clic en **Create rule alert** (Crear alerta de regla) para crear la alerta de regla de la métrica.

En 10 minutos, la alerta se activa y se desencadena como se ha descrito anteriormente.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre cómo [configurar webhooks en las alertas](../../azure-monitor/platform/alerts-webhooks.md).
