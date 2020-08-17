---
title: 'Acceso a los registros de auditoría: CLI de Azure: Azure Database for MySQL'
description: En este artículo se describe cómo configurar los registros de auditoría de Azure Database for MySQL y cómo acceder a ellos desde la CLI de Azure.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 6/24/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 568f49565c6e6d8062f8869566cf3879b7c97eaa
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503334"
---
# <a name="configure-and-access-audit-logs-in-the-azure-cli"></a>Configuración y acceso a los registros de auditoría en la CLI de Azure

Puede configurar los [registros de auditoría de Azure Database for MySQL](concepts-audit-logs.md) desde la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

Para seguir esta guía, necesitará:

- [Servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> Esta guía de procedimientos requiere el uso de la CLI de Azure versión 2.0 o posterior. Para confirmar la versión, en el símbolo del sistema de la CLI de Azure, escriba `az --version`. Para la instalación o la actualización, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="configure-audit-logging"></a>Configuración del registro de auditoría

>[!IMPORTANT]
> Se recomienda registrar solo los tipos de evento y los usuarios necesarios con fines de auditoría para asegurarse de que el rendimiento del servidor no se vea afectado en gran medida.

Habilite y configure el registro de auditoría mediante los pasos siguientes:

1. Para activar los registros de auditoría, establezca el parámetro **audit_logs_enabled** en "ON". 
    ```azurecli-interactive
    az mysql server configuration set --name audit_log_enabled --resource-group myresourcegroup --server mydemoserver --value ON
    ```

1. Seleccione los [tipos de eventos](concepts-audit-logs.md#configure-audit-logging) que se registrarán actualizando el parámetro **audit_log_events**.
    ```azurecli-interactive
    az mysql server configuration set --name audit_log_events --resource-group myresourcegroup --server mydemoserver --value "ADMIN,CONNECTION"
    ```

1. Agregue a cualquier usuario de MySQL que se excluya del registro actualizando el parámetro **audit_log_exclude_users**. Especifique los usuarios proporcionando su nombre de usuario de MySQL.
    ```azurecli-interactive
    az mysql server configuration set --name audit_log_exclude_users --resource-group myresourcegroup --server mydemoserver --value "azure_superuser"
    ```

1. Agregue los usuarios de MySQL específicos que se deban incluir para el registro mediante la actualización del parámetro **audit_log_include_users**. Especifique los usuarios proporcionando su nombre de usuario de MySQL.
    ```azurecli-interactive
    az mysql server configuration set --name audit_log_include_users --resource-group myresourcegroup --server mydemoserver --value "sampleuser"
    ```

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre los [registros de auditoría](concepts-audit-logs.md) en Azure Database for MySQL.
- Aprenda a configurar los registros de auditoría en [Azure Portal](howto-configure-audit-logs-portal.md)
