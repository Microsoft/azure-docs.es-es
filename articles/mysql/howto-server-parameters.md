---
title: Configuración de parámetros de servidor de Azure Database for MySQL mediante Azure Portal
description: En este artículo se explica cómo configurar parámetros del servidor MySQL en Azure Database for MySQL mediante Azure Portal.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 6/11/2020
ms.openlocfilehash: a37fbee4361d4a87c43a42cae66c425eba1e0877
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90887051"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-using-the-azure-portal"></a>Configuración de parámetros del servidor en Azure Database for MySQL mediante Azure Portal

Azure Database para MySQL admite la configuración de algunos parámetros del servidor. En este artículo se explica cómo configurar estos parámetros mediante Azure Portal. No es posible ajustar todos los parámetros del servidor.

## <a name="configure-server-parameters"></a>Configuración de parámetros del servidor

1. Inicie sesión en Azure Portal y luego localice su servidor de Azure Database for MySQL.
2. En la sección **CONFIGURACIÓN**, haga clic en **Parámetros del servidor** para abrir la página Parámetros del servidor de Azure Database for MySQL.
:::image type="content" source="./media/howto-server-parameters/auzre-portal-server-parameters.png" alt-text="Página Parámetros del servidor de Azure Portal":::
3. Localice cualquier configuración que deba ajustar. Revise la columna **Descripción** para entender el propósito y los valores permitidos.
:::image type="content" source="./media/howto-server-parameters/3-toggle_parameter.png" alt-text="Menú desplegable de enumeración":::
4. Haga clic en **Guardar** para guardar los cambios.
:::image type="content" source="./media/howto-server-parameters/4-save_parameters.png" alt-text="Guardar o descartar cambios":::
5. Si ha guardado los nuevos valores para los parámetros, siempre puede revertir todos los elementos a los valores predeterminados; para ello, seleccione **Restablecer todos los valores predeterminados**.
:::image type="content" source="./media/howto-server-parameters/5-reset_parameters.png" alt-text="Restablecer todos los valores predeterminados":::

## <a name="setting-parameters-not-listed"></a>Ajustar parámetros que no aparecen en la lista

Si el parámetro de servidor que desea actualizar no aparece en Azure Portal, también puede establecer el parámetro en el nivel de conexión mediante `init_connect`. De este modo se establecen los parámetros del servidor para cada cliente con conexión al servidor. 

1. En la sección **CONFIGURACIÓN**, haga clic en **Parámetros del servidor** para abrir la página Parámetros del servidor de Azure Database for MySQL.
2. Busque `init_connect`.
3. Agregue los parámetros del servidor con el formato: `SET parameter_name=YOUR_DESIRED_VALUE` en valor la columna de valor.

    Por ejemplo, puede cambiar el conjunto de caracteres del servidor estableciendo `init_connect` en `SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;`.
4. Haga clic en **Guardar** para guardar los cambios.

>[!Note]
> `init_connect` se puede usar para cambiar los parámetros que no requieren privilegios elevados en el nivel de sesión. Para comprobar si puede establecer el parámetro mediante `init_connect`, ejecute el comando `set session parameter_name=YOUR_DESIRED_VALUE;` y si se producen errores que indican **acceso denegado y que necesita los privilegios elevados**, no puede establecer el parámetro con "init_connect".

## <a name="working-with-the-time-zone-parameter"></a>Trabajo con el parámetro de zona horaria

### <a name="populating-the-time-zone-tables"></a>Relleno de las tablas de la zona horaria

Las tablas de la zona horaria del servidor se pueden rellenar mediante una llamada al procedimiento almacenado `mysql.az_load_timezone` desde una herramienta como la línea de comandos de MySQL o MySQL Workbench.

> [!NOTE]
> Si ejecuta el comando `mysql.az_load_timezone` de MySQL Workbench, es posible que primero tenga que desactivar el modo de actualización segura mediante `SET SQL_SAFE_UPDATES=0;`.

```sql
CALL mysql.az_load_timezone();
```

> [!IMPORTANT]
> Debe reiniciar el servidor para asegurarse de que las tablas de zona horaria se rellenen correctamente. Para reiniciar el servidor, use [Azure Portal](howto-restart-server-portal.md) o la [CLI](howto-restart-server-cli.md).

Para ver los valores de zonas horarias disponibles, ejecute el comando siguiente:

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>Establecimiento de la zona horaria de nivel global

La zona horaria de nivel global se puede establecer desde la página **Parámetros del servidor** en Azure Portal. En el ejemplo siguiente se establece la zona horaria global en el valor "US/Pacific".

:::image type="content" source="./media/howto-server-parameters/timezone.png" alt-text="Establecimiento del parámetro de zona horaria":::

### <a name="setting-the-session-level-time-zone"></a>Establecimiento de la zona horaria de nivel de sesión

La zona horaria de nivel de sesión se puede establecer mediante la ejecución del comando `SET time_zone` desde una herramienta como la línea de comandos de MySQL o MySQL Workbench. En el ejemplo siguiente se establece la zona horaria en **US/Pacific**.

```sql
SET time_zone = 'US/Pacific';
```

Consulte [Date and Time Functions](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz) (Funciones de fecha y hora) en la documentación de MySQL.

## <a name="next-steps"></a>Pasos siguientes

- [Bibliotecas de conexiones de Azure Database for MySQL](concepts-connection-libraries.md)
