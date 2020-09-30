---
title: Acceso a registros de consultas lentas mediante Azure Portal en Azure Database for MySQL
description: En este artículo se describe cómo configurar los registros de consultas lentas de Azure Database for MySQL y acceder a ellos mediante Azure Portal.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 4/13/2020
ms.openlocfilehash: 69368500a99b47238e74a960fdd5381c0339430a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905866"
---
# <a name="configure-and-access-slow-query-logs-from-the-azure-portal"></a>Configuración y acceso a los registros de consultas lentas en Azure Portal

Los [Registros de consultas lentas de Azure Database for MySQL](concepts-server-logs.md) se pueden configurar, enumerar y descargar desde Azure Portal.

## <a name="prerequisites"></a>Prerequisites
Los pasos descritos en este artículo requieren que tenga un [servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-portal.md).

## <a name="configure-logging"></a>registro
Configure el acceso al registro de consultas lentas de MySQL. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Seleccione un servidor de Azure Database for MySQL.

3. En la sección **Supervisión** de la barra lateral, seleccione **Registros de servidor**. 
   :::image type="content" source="./media/howto-configure-server-logs-in-portal/1-select-server-logs-configure.png" alt-text="Captura de pantalla de las opciones de registros de servidor":::

4. Seleccione **Haga clic aquí para habilitar los registros y configurar los parámetros** para ver los parámetros del servidor.

5. Sitúe **slow_query_log** en la posición **ON** (ACTIVADO).

6. Seleccione la ubicación en la que se van a generar los registros con **log_output**. Para enviar registros al almacenamiento local y a los registros de diagnóstico de Azure Monitor, seleccione **File** (Archivo). 

7. Cambie cualquier otro parámetro necesario. 

8. Seleccione **Guardar**. 

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/3-save-discard.png" alt-text="Captura de pantalla de parámetros de registro de consultas lentas y la acción de guardar.":::

Desde la página **Parámetros de servidor**, puede volver a la lista de los registros cerrando la página.

## <a name="view-list-and-download-logs"></a>Visualización de lista y descarga de registros
Una vez que comienza el registro, puede ver una lista de los registros de consultas lentas disponibles y descargar archivos de registro individuales.

1. Abra Azure Portal.

2. Seleccione un servidor de Azure Database for MySQL.

3. En la sección **Supervisión** de la barra lateral, seleccione **Registros de servidor**. La página muestra una lista de los archivos de registro.

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/4-server-logs-list.png" alt-text="Captura de pantalla de la página Registros de servidor con una lista de registros resaltados":::

   > [!TIP]
   > La convención de nomenclatura del registro es **mysql-slow-< nombre de su servidor>-yyyymmddhh.log**. La fecha y hora que se utilizan en el nombre de archivo indican el momento en que se emitió el registro. Los archivos de registro se rotan cada 24 horas o 7,5 GB, lo que ocurra primero. 

4. Si es necesario, utilice el cuadro de búsqueda para encontrar rápidamente un registro específico en función de la fecha y hora. La búsqueda se encuentra en el nombre del registro.

5. Para descargar los archivos de registro individuales, seleccione el icono de flecha hacia abajo que hay junto a cada archivo de registro en la fila de tabla.

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/5-download.png" alt-text="Captura de pantalla de la página Registros de servidor, con el icono de flecha hacia abajo resaltado":::

## <a name="set-up-diagnostic-logs"></a>Configuración de registros de diagnósticos

1. En la sección **Supervisión** de la barra lateral, seleccione **Configuración de diagnóstico** > **Add diagnostic settings** (Agregar configuración de diagnóstico).

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/add-diagnostic-setting.png" alt-text="Captura de pantalla de las opciones de diagnóstico":::

1. Proporcione un nombre de configuración de diagnóstico.

1. Especifique a qué receptores de datos se enviarán los registros de consultas lentas (cuenta de almacenamiento, centro de eventos o área de trabajo de Log Analytics).

1. Seleccione **MySqlSlowLogs** como tipo de registro.
:::image type="content" source="./media/howto-configure-server-logs-in-portal/configure-diagnostic-setting.png" alt-text="Captura de pantalla de las opciones de configuración de diagnóstico":::

1. Una vez que haya configurado los receptores de datos a los que canalizar los registros de consultas lentas, seleccione **Guardar**.
:::image type="content" source="./media/howto-configure-server-logs-in-portal/save-diagnostic-setting.png" alt-text="Captura de pantalla de las opciones de configuración de diagnóstico, con la opción Guardar resaltada":::

1. Acceda a los registros de consultas lentas explorándolos en los receptores de datos que configuró. Los registros pueden tardar hasta 10 minutos en aparecer.

## <a name="next-steps"></a>Pasos siguientes
- Consulte el artículo de [configuración y acceso a los registros de consultas lentas con la CLI de Azure](howto-configure-server-logs-in-cli.md) para más información sobre cómo descargar registros de consultas lentas mediante programación.
- Más información sobre los [registros de consultas lentas](concepts-server-logs.md) en Azure Database for MySQL.
- Para más información sobre las definiciones de parámetros y el registro de MySQL, consulte la documentación de MySQL sobre [registros](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html).