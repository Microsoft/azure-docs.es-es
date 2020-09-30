---
title: 'Inicio rápido: Conexión (MySQL Workbench): Azure Database for MySQL'
description: En esta guía de inicio rápido se proporcionan los pasos para usar MySQL Workbench para conectarse a Azure Database for MySQL y consultar datos en este servicio.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 5/26/2020
ms.openlocfilehash: 5ec6aab8aaa63b848131d44f78867100f154d251
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90896329"
---
# <a name="quickstart-use-mysql-workbench-to-connect-and-query-data-in-azure-database-for-mysql"></a>Inicio rápido: Uso de MySQL Workbench para conectarse y consultar datos en Azure Database for MySQL

En esta guía de inicio rápido se muestra cómo conectarse a Azure Database for MySQL mediante la aplicación MySQL Workbench.

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial rápido se usan como punto de partida los recursos creados en una de estas guías:
- [Create an Azure Database for MySQL server using Azure Portal](./quickstart-create-mysql-server-database-using-azure-portal.md) (Creación de un servidor de Azure Database for MySQL mediante Azure Portal)
- [Create an Azure Database for MySQL server using Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md) (Creación de un servidor de Azure Database for MySQL mediante la CLI de Azure)

> [!IMPORTANT] 
> Asegúrese de que a la dirección IP desde la que se conecta se le han agregado las reglas de firewall del servidor desde [Azure Portal](./howto-manage-firewall-using-portal.md) o la [CLI de Azure](./howto-manage-firewall-using-cli.md)

## <a name="install-mysql-workbench"></a>Instalación de MySQL Workbench
Descargue e instale MySQL Workbench en el equipo desde [el sitio web de MySQL](https://dev.mysql.com/downloads/workbench/).

## <a name="get-connection-information"></a>Obtención de información sobre la conexión
Obtenga la información de conexión necesaria para conectarse a Azure Database for MySQL. Necesitará el nombre completo del servidor y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. En el menú izquierdo de Azure Portal, haga clic en **Todos los recursos** y, luego, busque el servidor que ha creado, por ejemplo, **mydemoserver**.

3. Haga clic en el nombre del servidor.

4. En el panel **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla en este panel.
 :::image type="content" source="./media/connect-php/1_server-overview-name-login.png" alt-text="Nombre del servidor de Azure Database for MySQL":::

## <a name="connect-to-the-server-by-using-mysql-workbench"></a>Conexión al servidor con MySQL Workbench 
Para conectarse al servidor Azure MySQL mediante la herramienta MySQL Workbench de la GUI, siga estos pasos:

1.    Inicie la aplicación MySQL Workbench en el equipo. 

2.    En el cuadro de diálogo **Setup New Connection** (Establecer nueva conexión), escriba la siguiente información en la pestaña **Parámetros**:

:::image type="content" source="./media/connect-workbench/2-setup-new-connection.png" alt-text="Configuración de una conexión nueva":::

| **Configuración** | **Valor sugerido** | **Descripción del campo** |
|---|---|---|
|    Nombre de la conexión | Conexión de demostración | Especifique una etiqueta para esta conexión. |
| Método de conexión | Estándar (TCP/IP) | Estándar (TCP/IP) es suficiente. |
| Hostname | *nombre del servidor* | Especifique el valor de nombre de servidor que se usó cuando creó el servidor de Azure Database for MySQL. Nuestro servidor de ejemplo es mydemoserver.mysql.database.azure.com. Use el nombre de dominio completo (\*.mysql.database.azure.com) tal como se muestra en el ejemplo. Siga los pasos de la sección anterior para obtener la información de conexión si no recuerda el nombre del servidor.  |
| Port | 3306 | Utilice siempre el puerto 3306 para conectarse a Azure Database for MySQL. |
| Nombre de usuario |  *nombre de inicio de sesión del administrador del servidor* | Escriba el valor de nombre de inicio de sesión del administrador del servidor que se usó al crear el servidor de Azure Database for MySQL. El nombre de usuario de nuestro ejemplo es myadmin@mydemoserver. Siga los pasos de la sección anterior para obtener la información de conexión si no recuerda el nombre de usuario. El formato es *nombre de usuario\@nombre de servidor*.
| Contraseña | la contraseña | Haga clic en el botón **Store in Vault...** (Almacenar en el almacén) para guardar la contraseña. |

3.   Haga clic en **Probar conexión** para probar si todos los parámetros están configurados correctamente. 

4.   Haga clic en **Aceptar** para guardar la conexión. 

5.   En la lista de **conexiones de MySQL**, haga clic en el icono que corresponde al servidor y, luego, espere que se establezca la conexión.

        Una pestaña SQL nueva se abre con un editor en blanco en el que puede escribir las consultas.
    
        > [!NOTE]
        > De forma predeterminada, la seguridad de conexión SSL es necesaria y se aplica al servidor de Azure Database para MySQL. Aunque normalmente no se necesita ninguna configuración adicional con certificados SSL para que MySQL Workbench se conecte al servidor, se recomienda enlazar el certificado SSL de entidad de certificación con MySQL Workbench. Para obtener más información sobre cómo descargar y enlazar el certificado, consulte [Configuración de la conectividad SSL en la aplicación para conectarse de forma segura a Azure Database for MySQL](./howto-configure-ssl.md).  Si necesita deshabilitar SSL, visite Azure Portal y haga clic en la página Seguridad de la conexión para deshabilitar el botón de alternancia Aplicar conexión SSL.

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
    
    :::image type="content" source="media/connect-workbench/3-workbench-sql-tab.png" alt-text="Pestaña SQL de MySQL Workbench para ejecutar código de SQL de ejemplo":::

2. Para ejecutar el código SQL de ejemplo, haga clic en el icono de rayo que aparece en la barra de herramientas de la pestaña **Archivo SQL**.
3. Observe los tres resultados en pestañas de la sección **Cuadrículas de resultados** del centro de la página. 
4. Observe la lista **Salida** que se encuentra en la parte inferior de la página. Se muestra el estado de cada comando. 

Ahora está conectado a Azure Database for MySQL con MySQL Workbench y ha consultado datos mediante el lenguaje SQL.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./concepts-migrate-import-export.md)
