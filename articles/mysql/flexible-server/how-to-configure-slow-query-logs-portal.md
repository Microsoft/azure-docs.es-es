---
title: Configuración de registros de consultas lentas de un servidor flexible de Azure Database for MySQL en Azure Portal
description: En este artículo se describe cómo configurar los registros de consultas lentas en el servidor flexible de Azure Database for MySQL y acceder a ellos mediante Azure Portal.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 1416efaa325ced623b5c7514f2f9953dc0bd2781
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932508"
---
# <a name="configure-and-access-slow-query-logs-for-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Configuración de registros de consultas lentas de un servidor flexible de Azure Database for MySQL y acceso a ellos mediante Azure Portal

> [!IMPORTANT]
> Actualmente, la opción de implementación Servidor flexible de Azure Database for MySQL se encuentra en versión preliminar pública.

Los [registros de consultas lentas](concepts-slow-query-logs.md) de Azure Database for MySQL se pueden configurar, enumerar y descargar desde Azure Portal.

## <a name="prerequisites"></a>Requisitos previos
Los pasos que se describen en este artículo requieren que tenga un [servidor flexible](quickstart-create-server-portal.md).

## <a name="configure-logging"></a>registro
Configure el acceso al registro de consultas lentas de MySQL. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Seleccione el servidor flexible.

1. En la sección **Configuración**, seleccione **Parámetros del servidor**.
   <!-- :::image type="content" source="./media/howto-configure-server-logs-in-portal/1-select-server-logs-configure.png" alt-text="Screenshot of Server logs options":::-->

1. Actualice el parámetro **slow_query_log** a **ON**.

1. Cambie cualquier otro parámetro necesario (por ejemplo, `long_query_time`, `log_slow_admin_statements`). Consulte la documentación de los [registros de consultas lentas](./concepts-slow-query-logs.md#configure-slow-query-logging) para conocer más parámetros.  

1. Seleccione **Guardar**. 

   <!-- :::image type="content" source="./media/howto-configure-server-logs-in-portal/3-save-discard.png" alt-text="Screenshot of slow query log parameters and save."::: -->

Desde la página **Parámetros de servidor**, puede volver a la lista de los registros cerrando la página.

## <a name="set-up-diagnostics"></a>Configuración de diagnósticos

Los registros de consultas lentas se integran en la configuración de diagnóstico de Azure Monitor para que pueda canalizar los registros a los registros de Azure Monitor, Event Hubs o Azure Storage.

1. En la sección **Supervisión** de la barra lateral, seleccione **Configuración de diagnóstico** > **Add diagnostic settings** (Agregar configuración de diagnóstico).

   <!--:::image type="content" source="./media/howto-configure-server-logs-in-portal/add-diagnostic-setting.png" alt-text="Screenshot of Diagnostic settings options":::-->

1. Proporcione un nombre de configuración de diagnóstico.

1. Especifique a qué destinos se envían los registros de consultas lentas (cuenta de almacenamiento, centro de eventos o área de trabajo de Log Analytics).

1. Seleccione **MySqlSlowLogs** como tipo de registro.
    <!--:::image type="content" source="./media/howto-configure-server-logs-in-portal/configure-diagnostic-setting.png" alt-text="Screenshot of Diagnostic settings configuration options":::-->

1. Una vez que haya configurado los receptores de datos a los que canalizar los registros de consultas lentas, seleccione **Guardar**.
    <!--:::image type="content" source="./media/howto-configure-server-logs-in-portal/save-diagnostic-setting.png" alt-text="Screenshot of Diagnostic settings configuration options, with Save highlighted":::-->

1. Acceda a los registros de consultas lentas explorándolos en los receptores de datos que configuró. Los registros pueden tardar hasta 10 minutos en aparecer.

Si ha canalizado sus registros de auditoría a los registros de Azure Monitor (Log Analytics), consulte varias [consultas de ejemplo](concepts-audit-logs.md#analyze-logs-in-azure-monitor-logs) que puede usar para el análisis. 

## <a name="next-steps"></a>Pasos siguientes
<!-- - See [Access slow query Logs in CLI](howto-configure-server-logs-in-cli.md) to learn how to download slow query logs programmatically.-->
- Más información sobre los [registros de consultas lentas](concepts-slow-query-logs.md)
- Para más información sobre las definiciones de parámetros y el registro de MySQL, consulte la documentación de MySQL sobre [registros](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html).
- Más información sobre los [registros de auditoría](concepts-audit-logs.md)