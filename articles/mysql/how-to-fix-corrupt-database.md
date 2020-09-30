---
title: 'Corrección de bases de datos dañadas: Azure Database for MySQL'
description: Obtenga información sobre cómo corregir los problemas de daños en bases de datos en Azure Database for MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 09/21/2020
ms.openlocfilehash: 815b24d25e674e5460cc50d7eb6871f740994893
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932192"
---
# <a name="troubleshoot-database-corruption-on-azure-database-for-mysql"></a>Solución de problemas de daños en bases de datos en Azure Database for MySQL
[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

Los daños en una base de datos pueden provocar tiempo de inactividad en la aplicación y es fundamental resolver el problema a tiempo para evitar la pérdida de datos. Cuando se produzcan daños en una base de datos, en los registros del servidor, verá el error **InnoDB: Página de base de datos dañada en el disco o un archivo con errores**.

En esta guía, aprenderá a corregir los daños en bases de datos o tablas. En Azure Database for MySQL se usa el motor InnoDB, que incluye operaciones automatizadas de comprobación y reparación de daños. InnoDB comprueba si hay páginas dañadas mediante la realización de sumas de comprobación en cada página que lee y, si encuentra una discrepancia en la suma de comprobación, detiene automáticamente el servidor de MySQL.

Pruebe cualquiera de las opciones que aparecen a continuación para mitigar rápidamente los problemas de daños en bases de datos.

## <a name="restart-your-mysql-server"></a>Reinicio del servidor de MySQL

Normalmente se dará cuenta de que una base de datos o una tabla está dañada cuando la aplicación acceda a ella. InnoDB incluye un mecanismo de recuperación tras un bloqueo que puede resolver la mayoría de los problemas al reiniciar el servidor. Por lo tanto, el reinicio del servidor debería ayudar a que este se recuperase de un bloqueo que hizo que la base de datos estuviera en mal estado.

##  <a name="resolve-data-corruption-with-dump-and-restore-method"></a>Corrección de datos dañados con el método de volcado y restauración

Se recomienda corregir los daños con un **método de volcado y restauración**. Para ello, se debe acceder a la tabla dañada, usar la utilidad **mysqldump** para crear una copia de seguridad lógica de la tabla (la cual conservará la estructura de la tabla y los datos que contiene) y, después, recargar la tabla en la base de datos.

### <a name="backup-your-database-or-tables"></a>Copia de seguridad de las tablas o bases de datos

> [!Important]
> - Configure una regla de firewall para acceder al servidor desde la máquina cliente. Consulte cómo configurar una [regla de firewall en un servidor único](howto-manage-firewall-using-portal.md) y una [regla de firewall en un servidor flexible](flexible-server/how-to-connect-tls-ssl.md).
> - Use la opción ```--ssl-cert``` de SSL para **mysqldump** si SSL está habilitado.

Use el siguiente comando para crear un archivo de copia de seguridad a partir de la línea de comandos mediante mysqldump:

```
$ mysqldump [--ssl-cert=/path/to/pem] -h [host] -u [uname] -p[pass] [dbname] > [backupfile.sql]
```

Los parámetros que se proporcionan son los siguientes:
- [ssl-cert=/path/to/pem]: descargue el certificado SSL en la máquina cliente y establezca la ruta de acceso en el comando. No use esta opción si SSL está deshabilitado.
- [host]: su servidor de Azure Database for MySQL.
- [uname]: su nombre de usuario de administrador del servidor.
- [pass]: la contraseña del usuario administrador.
- [dbname]: el nombre de la base de datos.
- [backupfile.sql]: el nombre de archivo de copia de seguridad de la base de datos.

> [!Important]
> - Para un servidor único, use el formato ```admin-user@servername``` para reemplazar ```myserveradmin``` en los siguientes comandos.
> - Para un servidor flexible, use el formato ```admin-user``` para reemplazar ```myserveradmin``` en los siguientes comandos.

Si solo están dañadas ciertas tablas, seleccione las tablas específicas de la base de datos para las que quiere realizar una copia de seguridad siguiendo este ejemplo:
```
$ mysqldump --ssl-cert=</path/to/pem> -h mydemoserver.mysql.database.azure.com -u myserveradmin -p testdb table1 table2 > testdb_tables_backup.sql
```

Para hacer una copia de seguridad de una o más bases de datos, use el conmutador --database y enumere los nombres de las bases de datos en una lista separados por espacios.

```
$ mysqldump --ssl-cert=</path/to/pem>  -h mydemoserver.mysql.database.azure.com -u myserveradmin -p --databases testdb1 testdb3 testdb5 > testdb135_backup.sql
```

###  <a name="restore-your-database-or-tables"></a>Restauración de tablas o bases de datos

En los pasos siguientes se muestra cómo restaurar bases de datos o tablas. Una vez creado el archivo de copia de seguridad, puede restaurar las tablas o las bases de datos mediante la utilidad ***mysql**. Ejecute el comando como se muestra a continuación:

```
mysql  --ssl-cert=</path/to/pem> -h [hostname] -u [uname] -p[pass] [db_to_restore] < [backupfile.sql]
```
Este es un ejemplo de la restauración de ```testdb``` a partir el archivo de copia de seguridad creado con **mysqldump**. 

> [!Important]
> - Para un servidor único, use el formato ```admin-user@servername``` para reemplazar ```myserveradmin``` en el siguiente comando.
> - Para un servidor flexible, use el formato ```admin-user``` para reemplazar ```myserveradmin``` en el siguiente comando. 

```
$ mysql --ssl-cert=</path/to/pem> -h mydemoserver.mysql.database.azure.com -u myserveradmin -p testdb < testdb_backup.sql
```

## <a name="next-steps"></a>Pasos a seguir
Si los pasos anteriores no le ayudan a resolver el problema, siempre puede restaurar todo el servidor.
- [Restauración de un servidor único de Azure Database for MySQL](howto-restore-server-portal.md)
- [Restauración de un servidor flexible de Azure Database for MySQL](flexible-server/how-to-restore-server-portal.md)



