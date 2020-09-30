---
title: 'Creación de bases de datos y usuarios: Azure Database for MySQL'
description: En este artículo se describe cómo puede crear nuevas cuentas de usuario para interactuar con un servidor de Azure Database for MySQL.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 4/2/2020
ms.openlocfilehash: 9b79a0f21135e91ab72a4c8a9e604b84b67df0a9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90902815"
---
# <a name="create-databases-and-users-in-azure-database-for-mysql-server"></a>Creación de bases de datos y usuarios en Azure Database for MySQL

[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

En este artículo se describe cómo puede crear usuarios en un servidor de Azure Database for MySQL.

> [!NOTE]
> Comunicación sin prejuicios
>
> Microsoft admite un entorno diverso e inclusivo. En este artículo se incluyen referencias a la palabra _esclavo_. En la [guía de estilo para la comunicación sin prejuicios](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) de Microsoft se reconoce que se trata de una palabra excluyente. Se usa en este artículo por coherencia, ya que actualmente es la palabra que aparece en el software. Cuando se actualice el software para quitarla, este artículo se actualizará para que esté alineado.
>

La primera vez que creó su instancia de Azure Database for MySQL, proporcionó un nombre de usuario y una contraseña de inicio de sesión de administrador del servidor. Para más información, puede seguir la [Guía de inicio rápido](quickstart-create-mysql-server-database-using-azure-portal.md). Puede encontrar su nombre de usuario de inicio de sesión de administrador del servidor en Azure Portal.

El usuario administrador del servidor obtiene ciertos privilegios para el servidor, según se indica a continuación: 

   SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER


Una vez creado el servidor de Azure Database for MySQL, puede usar la primera cuenta de usuario administrador del servidor para crear usuarios adicionales y concederles acceso de administrador. Además, la cuenta de administrador del servidor puede usarse para crear usuarios con menos privilegios que tengan acceso a esquemas de base de datos individuales.

> [!NOTE]
> El privilegio SUPER y el rol DBA no se admiten. Revise los [privilegios](concepts-limits.md#privilege-support) en el artículo que trata sobre las limitaciones para saber lo que no se admite en el servicio.

## <a name="how-to-create-database-with-non-admin-user-in-azure-database-for-mysql"></a>Creación de una base de datos con un usuario que no sea administrador en Azure Database for MySQL

1. Obtenga la información de conexión y el nombre de usuario administrador.
   Para conectarse al servidor de bases de datos, es preciso utilizar el nombre completo del servidor y las credenciales de inicio de sesión de administrador. Puede encontrar fácilmente el nombre del servidor y la información de inicio de sesión en la página **Información general** del servidor o en la página **Propiedades** de Azure Portal.

2. Use la cuenta y la contraseña de administrador para conectarse a su servidor de base de datos. Use la herramienta preferida de cliente, como MySQL Workbench, mysql.exe, HeidiSQL u otras.
   Si no sabe seguro cómo conectarse, consulte cómo usar MySQL Workbench para [conectarse a los datos y consultarlos para un servidor único](./connect-workbench.md) o [conectarse a los datos y consultarlos para un servidor flexible](./flexible-server/connect-workbench.md).

3. Modifique y ejecute el siguiente código SQL. Reemplace el valor de marcador de posición `db_user` por su nuevo nombre de usuario deseado y el valor de marcador de posición `testdb` por su propio nombre de base de datos.

   Esta sintaxis de código sql crea una nueva base de datos denominada testdb para que sirva de ejemplo. A continuación, crea un nuevo usuario en el servicio MySQL y concede todos los privilegios al nuevo esquema de base de datos (testdb.\*) de ese usuario.

   ```sql
   CREATE DATABASE testdb;

   CREATE USER 'db_user'@'%' IDENTIFIED BY 'StrongPassword!';

   GRANT ALL PRIVILEGES ON testdb . * TO 'db_user'@'%';

   FLUSH PRIVILEGES;
   ```

4. Compruebe las concesiones dentro de la base de datos.

   ```sql
   USE testdb;

   SHOW GRANTS FOR 'db_user'@'%';
   ```

5. Inicie sesión en el servidor mediante el nuevo nombre de usuario y contraseña, sin olvidarse de especificar la base de datos designada. En este ejemplo se muestra la línea de comandos de mysql. Con este comando, se le pedirá la contraseña del nombre de usuario. Reemplace su propio nombre de servidor, nombre de base de datos y nombre de usuario.

# <a name="single-server"></a>[Servidor único](#tab/single-server)

   ```azurecli-interactive
   mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user@mydemoserver -p
   ```
# <a name="flexible-server"></a>[Servidor flexible](#tab/flexible-server)

   ```azurecli-interactive
   mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user -p
   ```
 ---

## <a name="how-to-create-additional-admin-users-in-azure-database-for-mysql"></a>Creación de usuarios administradores adicionales en Azure Database for MySQL

1. Obtenga la información de conexión y el nombre de usuario administrador.
   Para conectarse al servidor de bases de datos, es preciso utilizar el nombre completo del servidor y las credenciales de inicio de sesión de administrador. Puede encontrar fácilmente el nombre del servidor y la información de inicio de sesión en la página **Información general** del servidor o en la página **Propiedades** de Azure Portal.

2. Use la cuenta y la contraseña de administrador para conectarse a su servidor de base de datos. Use la herramienta preferida de cliente, como MySQL Workbench, mysql.exe, HeidiSQL u otras.
   Si no está seguro de cómo conectarse, consulte [Uso de MySQL Workbench para conectarse y consultar datos](./connect-workbench.md).

3. Modifique y ejecute el siguiente código SQL. Reemplace el valor de marcador de posición `new_master_user` por el nuevo nombre de usuario. Esta sintaxis concede los privilegios enumerados en todos los esquemas de base de datos ( *.* ) al nombre de usuario (new_master_user en este ejemplo).

   ```sql
   CREATE USER 'new_master_user'@'%' IDENTIFIED BY 'StrongPassword!';

   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'new_master_user'@'%' WITH GRANT OPTION;

   FLUSH PRIVILEGES;
   ```

4. Comprobación de las concesiones

   ```sql
   USE sys;

   SHOW GRANTS FOR 'new_master_user'@'%';
   ```

## <a name="next-steps"></a>Pasos siguientes

Abra el firewall para las direcciones IP de las máquinas de los nuevos usuarios para permitirles conectarse:
- [Creación y administración de reglas de firewall en un servidor único](howto-manage-firewall-using-portal.md) 
- [Creación y administración de reglas de firewall en un servidor flexible](flexible-server/how-to-connect-tls-ssl.md)

Para más información respecto a la administración de cuentas de usuario, consulte la documentación del producto de MySQL relativa a la [administración de cuentas de usuario](https://dev.mysql.com/doc/refman/5.7/en/access-control.html), la [sintaxis GRANT](https://dev.mysql.com/doc/refman/5.7/en/grant.html) y los [privilegios](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html).
