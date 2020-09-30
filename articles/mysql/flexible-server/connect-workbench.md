---
title: 'Inicio rápido: Conexión mediante MySQL Workbench a Azure Database for MySQL con la opción Servidor flexible'
description: En esta guía de inicio rápido se proporcionan los pasos para usar MySQL Workbench para conectarse a Azure Database for MySQL con la opción Servidor flexible y consultar datos en este servicio.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 9/21/2020
ms.openlocfilehash: d604ca8c8979ec98b990f8002ce29d0df92ac2af
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946234"
---
# <a name="quickstart-use-mysql-workbench-to-connect-and-query-data-in-azure-database-for-mysql---flexible-server-preview"></a>Inicio rápido: Use MySQL Workbench para conectarse a Azure Database for MySQL con la opción Servidor flexible (versión preliminar) y consultar datos en este servicio.


> [!IMPORTANT] 
> Actualmente, Azure Database for MySQL con la opción Servidor flexible se encuentra en versión preliminar pública.

En este inicio rápido se muestra cómo conectarse a una instancia de Azure Database for MySQL con la opción Servidor flexible mediante la aplicación MySQL Workbench.

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial rápido se usan como punto de partida los recursos creados en una de estas guías:

- [Creación de una instancia de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./quickstart-create-server-portal.md).
- [Creación de una instancia de Azure Database for MySQL con opción Servidor flexible mediante la CLI de Azure](./quickstart-create-server-cli.md)

## <a name="preparing-your-client-workstation"></a>Preparación de la estación de trabajo de cliente
- Si creó el servidor flexible con la opción *Acceso privado (integración con red la virtual)* , deberá conectarse a su servidor desde un recurso de la misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada con el servidor flexible. Consulte [Creación y administración de la red virtual de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure](./how-to-manage-virtual-network-cli.md).
- Si creó el servidor flexible con la opción *Acceso público (direcciones IP permitidas)* , puede agregar la dirección IP local a la lista de reglas de firewall del servidor. Consulte [Creación y administración de reglas de firewall de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure](./how-to-manage-firewall-cli.md).

- Descargue e instale MySQL Workbench en el equipo desde [el sitio web de MySQL](https://dev.mysql.com/downloads/workbench/).

## <a name="get-connection-information"></a>Obtención de información sobre la conexión

Obtenga la información de conexión necesaria para conectarse al servidor flexible. Necesitará el nombre completo del servidor y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el menú izquierdo de Azure Portal, seleccione **Todos los recursos** y, luego, busque el servidor que ha creado, por ejemplo, **mydemoserver**.
3. Seleccione el nombre del servidor.
4. En el panel **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla en este panel.
<!--- :::image type="content" source="./media/connect-php/1_server-overview-name-login.png" alt-text="Azure Database for MySQL Flexible Server name":::--->

## <a name="connect-to-the-server-using-mysql-workbench"></a>Conexión al servidor con MySQL Workbench

Para conectarse a Azure Database for MySQL con la opción Servidor flexible mediante MySQL Workbench, haga lo siguiente:

1. Inicie la aplicación MySQL Workbench en el equipo.

2. En el cuadro de diálogo **Setup New Connection** (Establecer nueva conexión), escriba la siguiente información en la pestaña **Parámetros**:

    :::image type="content" source="./media/connect-workbench/2-setup-new-connection.png" alt-text="Configuración de una conexión nueva":::

    | **Parámetros** | **Valor sugerido** | **Descripción del campo** |
    |---|---|---|
    |    Nombre de la conexión | Conexión de demostración | Especifique una etiqueta para esta conexión. |
    | Método de conexión | Estándar (TCP/IP) | Estándar (TCP/IP) es suficiente. |
    | Hostname | *nombre del servidor* | Especifique el valor de nombre de servidor que se usó cuando creó el servidor de Azure Database for MySQL. Nuestro servidor de ejemplo es mydemoserver.mysql.database.azure.com. Use el nombre de dominio completo (\*.mysql.database.azure.com) tal como se muestra en el ejemplo. Siga los pasos de la sección anterior para obtener la información de conexión si no recuerda el nombre del servidor.  |
    | Port | 3306 | Utilice siempre el puerto 3306 para conectarse a Azure Database for MySQL. |
    | Nombre de usuario |  *nombre de inicio de sesión del administrador del servidor* | Escriba el valor de nombre de inicio de sesión del administrador del servidor que se usó al crear el servidor de Azure Database for MySQL. El nombre de usuario del ejemplo es myadmin. Siga los pasos de la sección anterior para obtener la información de conexión si no recuerda el nombre de usuario.
    | Contraseña | la contraseña | Haga clic en el botón **Store in Vault...** (Almacenar en el almacén) para guardar la contraseña. |

3. Haga clic en **Probar conexión** para probar si todos los parámetros están configurados correctamente.

4. Haga clic en **Aceptar** para guardar la conexión.

5. En la lista de **conexiones de MySQL**, haga clic en el icono que corresponde al servidor y, luego, espere que se establezca la conexión.

    Una pestaña SQL nueva se abre con un editor en blanco en el que puede escribir las consultas.

> [!NOTE]
> La conexión cifrada mediante TLS 1.2 es necesaria y se aplica a la instancia de Azure Database for MySQL con la opción Servidor flexible. Aunque normalmente no se necesita ninguna configuración adicional con los certificados TLS/SSL para que MySQL Workbench se conecte al servidor, se recomienda enlazar la certificación de entidad de certificación de TLS/SSL con MySQL Workbench. Para obtener más información, consulte cómo [conectarse mediante TLS/SSL](./how-to-connect-tls-ssl.md).

## <a name="create-a-table-insert-data-read-data-update-data-delete-data"></a>Creación de una tabla, inserción de datos, lectura de datos, actualización de datos, eliminación de datos

1. Copie y pegue el código SQL de ejemplo en una pestaña SQL en blanco para mostrar algunos datos de ejemplo.

    Este código crea una base de datos vacía llamada quickstartdb y luego crea una tabla de ejemplo llamada inventario. Inserta algunas filas y luego las lee. Cambia los datos con una instrucción de actualización y vuelve a leer las filas. Finalmente, elimina una fila y vuelve a leer las filas.

    ```sql
    -- Create a database
    -- DROP DATABASE IF EXISTS quickstartdb;
    CREATE DATABASE quickstartdb;
    USE quickstartdb;

    -- Create a table and insert rows
    DROP TABLE IF EXISTS inventory;
    CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);
    INSERT INTO inventory (name, quantity) VALUES ('banana', 150);
    INSERT INTO inventory (name, quantity) VALUES ('orange', 154);
    INSERT INTO inventory (name, quantity) VALUES ('apple', 100);

    -- Read
    SELECT * FROM inventory;

    -- Update
    UPDATE inventory SET quantity = 200 WHERE id = 1;
    SELECT * FROM inventory;

    -- Delete
    DELETE FROM inventory WHERE id = 2;
    SELECT * FROM inventory;
    ```

    La captura de pantalla muestra un ejemplo del código SQL en SQL Workbench y el resultado una vez que se ejecuta.

    :::image type="content" source="./media/connect-workbench/3-workbench-sql-tab.png" alt-text="Pestaña SQL de MySQL Workbench para ejecutar código de SQL de ejemplo":::

2. Para ejecutar el código SQL de ejemplo, haga clic en el icono de rayo que aparece en la barra de herramientas de la pestaña **Archivo SQL**.
3. Observe los tres resultados en pestañas de la sección **Cuadrículas de resultados** del centro de la página.
4. Observe la lista **Salida** que se encuentra en la parte inferior de la página. Se muestra el estado de cada comando.

Ahora está conectado a Azure Database for MySQL con la opción Servidor flexible mediante MySQL Workbench y ha consultado datos con el lenguaje SQL.

## <a name="next-steps"></a>Pasos siguientes
- [Conectividad cifrada con Seguridad de la capa de transporte (TLS 1.2) en Azure Database for MySQL con la opción Servidor flexible](./how-to-connect-tls-ssl.md).
- Obtenga más información sobre las [redes de Azure Database for MySQL con la opción Servidor flexible](./concepts-networking.md).
- [Creación y administración de reglas de firewall de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./how-to-manage-firewall-portal.md).
- [Creación y administración de la red virtual de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./how-to-manage-virtual-network-portal.md).
