---
title: 'Configuración de registros de auditoría mediante Azure Portal en Azure Database for MySQL: Servidor flexible'
description: 'En este artículo se describe cómo configurar los registros de auditoría en Azure Database for MySQL: Servidor flexible y acceder a ellos mediante Azure Portal.'
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 1d0687e6cdcc8208218183cb873f9620a5525d37
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932513"
---
# <a name="configure-and-access-audit-logs-for-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Configuración de registros de auditoría para Azure Database for MySQL: Servidor flexible y acceso a ellos mediante Azure Portal

> [!IMPORTANT]
> Actualmente, Azure Database for MySQL: Servidor flexible se encuentra en versión preliminar pública.

Puede configurar los [registros de auditoría](concepts-audit-logs.md) de Azure Database for MySQL: Servidor flexible y realizar un diagnóstico de la configuración desde Azure Portal.

## <a name="prerequisites"></a>Requisitos previos
Los pasos que se describen en este artículo requieren que tenga un [servidor flexible](quickstart-create-server-portal.md).

## <a name="configure-audit-logging"></a>Configuración del registro de auditoría

>[!IMPORTANT]
> Se recomienda registrar solo los tipos de evento y los usuarios necesarios con fines de auditoría para asegurarse de que el rendimiento del servidor no se ve afectado en gran medida.

Habilite y configure el registro de auditoría.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Seleccione el servidor flexible.

1. En la sección **Configuración**, seleccione **Parámetros del servidor**.
    <!--:::image type="content" source="./media/howto-configure-audit-logs-portal/server-parameters.png" alt-text="Server parameters":::-->

1. Actualice el parámetro **audit_log_enabled** a ACTIVADO.
    <!-- :::image type="content" source="./media/howto-configure-audit-logs-portal/audit-log-enabled.png" alt-text="Enable audit logs":::-->

1. Seleccione los [tipos de eventos](concepts-audit-logs.md#configure-audit-logging) que se registrarán actualizando el parámetro **audit_log_events**.
    <!-- :::image type="content" source="./media/howto-configure-audit-logs-portal/audit-log-events.png" alt-text="Audit log events":::-->

1. Agregue a cualquier usuario de MySQL que se excluya del registro actualizando el parámetro **audit_log_exclude_users**. Especifique los usuarios proporcionando su nombre de usuario de MySQL.
    <!--:::image type="content" source="./media/howto-configure-audit-logs-portal/audit-log-exclude-users.png" alt-text="Audit log exclude users":::-->

1. Cuando haya cambiado los parámetros, haga clic en **Guardar**. O bien puede **descartar** los cambios.
    <!--:::image type="content" source="./media/howto-configure-audit-logs-portal/save-parameters.png" alt-text="Save":::-->

## <a name="set-up-diagnostics"></a>Configuración de diagnósticos

Los registros de auditoría se integran en la configuración de diagnóstico de Azure Monitor para que pueda canalizar los registros a los registros de Azure Monitor, Event Hubs o Azure Storage.

1. En la sección **Supervisión** de la barra lateral, seleccione **Configuración de diagnóstico**.

1. Haga clic en "Agregar configuración de diagnóstico"  .<!-- :::image type="content" source="./media/howto-configure-audit-logs-portal/add-diagnostic-setting.png" alt-text="Add diagnostic setting":::-->

1. Proporcione un nombre de configuración de diagnóstico.

1. Especifique los destinos a los que se van a enviar los registros de auditoría (cuenta de almacenamiento, centro de eventos o área de trabajo de Log Analytics).

1. Seleccione **MySqlAuditLogs** como el tipo de registro.
    <!-- :::image type="content" source="./media/howto-configure-audit-logs-portal/configure-diagnostic-setting.png" alt-text="Configure diagnostic setting"::: -->

1. Una vez que haya configurado los receptores de datos para canalizar los registros de auditoría, puede hacer clic en **Guardar**.
    <!-- :::image type="content" source="./media/howto-configure-audit-logs-portal/save-diagnostic-setting.png" alt-text="Save diagnostic setting":::-->

1. Acceda a los registros de auditoría mediante los receptores de datos que configuró. Los registros pueden tardar hasta 10 minutos en aparecer.

Si ha canalizado sus registros de auditoría a los registros de Azure Monitor (Log Analytics), consulte varias [consultas de ejemplo](concepts-audit-logs.md#analyze-logs-in-azure-monitor-logs) que puede usar para el análisis.  

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [registros de auditoría](concepts-audit-logs.md)
- Más información sobre los [registros de consultas lentas](concepts-slow-query-logs.md)
<!-- - Learn how to configure audit logs in the [Azure CLI](howto-configure-audit-logs-cli.md)-->