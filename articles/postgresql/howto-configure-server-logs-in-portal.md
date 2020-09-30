---
title: 'Administración de registros de Azure Database for PostgreSQL: servidor único mediante Azure Portal'
description: 'En este artículo se describe cómo configurar los registros de servidor (archivos .log) de Azure Database for PostgreSQL: servidor único, y acceder a ellos, desde Azure Portal.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: how-to
ms.date: 5/6/2019
ms.openlocfilehash: cf22a165e9a6eadcd2e5d4cdb0efa301538be7e9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907530"
---
# <a name="configure-and-access-azure-database-for-postgresql---single-server-logs-from-the-azure-portal"></a>Configuración y acceso a los registros de Azure Database for PostgreSQL con un único servidor desde Azure Portal

Puede configurar, enumerar y descargar los [registros de Azure Database for PostgreSQL](concepts-server-logs.md) de Azure Portal.

## <a name="prerequisites"></a>Prerequisites
Los pasos descritos en este artículo requieren que tenga un [servidor de Azure Database for PostgreSQL](quickstart-create-server-database-portal.md).

## <a name="configure-logging"></a>registro
Configure el acceso a los registros de consulta y los registros de errores. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Seleccione su servidor de Azure Database for PostgreSQL.

3. En la sección **Supervisión** de la barra lateral, seleccione **Registros de servidor**. 

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/1-select-server-logs-configure.png" alt-text="Captura de pantalla de las opciones de registros de servidor":::

4. Seleccione **Haga clic aquí para habilitar los registros y configurar los parámetros** para ver los parámetros del servidor.

5. Cambie los parámetros que necesita ajustar. Todos los cambios que realice en esta sesión se resaltan en color púrpura.

   Después de cambiar los parámetros, seleccione **Guardar**. O bien puede descartar los cambios. 

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/3-save-discard.png" alt-text="Captura de pantalla de las opciones de los parámetros del servidor":::

Desde la página **Parámetros de servidor**, puede volver a la lista de los registros cerrando la página.

## <a name="view-list-and-download-logs"></a>Visualización de lista y descarga de registros
Una vez que comienza el registro, puede ver una lista de los registros disponibles y descargar archivos de registro individuales. 

1. Abra Azure Portal.

2. Seleccione su servidor de Azure Database for PostgreSQL.

3. En la sección **Supervisión** de la barra lateral, seleccione **Registros de servidor**. La página muestra una lista de los archivos de registro.

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/4-server-logs-list.png" alt-text="Captura de pantalla de la página Registros de servidor con una lista de registros resaltados":::

   > [!TIP]
   > La convención de nomenclatura del registro es **postgresql-aaaa-mm-dd_hh0000.log**. La fecha y hora que se utilizan en el nombre de archivo indican el momento en que se emitió el registro. Los archivos de registro rotan cada hora o 100 MB, lo que ocurra primero.

4. Si es necesario, utilice el cuadro de búsqueda para encontrar rápidamente un registro específico en función de la fecha y hora. La búsqueda se encuentra en el nombre del registro.

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/5-search.png" alt-text="Captura de pantalla de la página Registros de servidor, con el cuadro de búsqueda y los resultados resaltados":::

5. Para descargar los archivos de registro individuales, seleccione el icono de flecha hacia abajo que hay junto a cada archivo de registro en la fila de tabla.

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/6-download.png" alt-text="Captura de pantalla de la página Registros de servidor, con el icono de flecha hacia abajo resaltado":::

## <a name="next-steps"></a>Pasos siguientes
- Consulte [Access server logs in CLI](howto-configure-server-logs-using-cli.md) (Acceso a los registros de servidor mediante la CLI) para más información acerca de cómo descargar registros mediante programación.
- Obtenga más información sobre los [registros de servidor](concepts-server-logs.md) en Azure Database for PostgreSQL. 
- Para más información sobre las definiciones de parámetros y el registro de PostgreSQL, consulte la documentación de PostgreSQL sobre [informes y registro de errores](https://www.postgresql.org/docs/current/static/runtime-config-logging.html).

