---
title: 'Configuración de la replicación de datos de entrada: Azure Database for MySQL'
description: En este artículo se describe la configuración de la replicación de datos internos para Azure Database for MySQL.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 8/7/2020
ms.openlocfilehash: f8dbdf87eef193540fd5c1bf9d9e7f3794ae46ce
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88168225"
---
# <a name="how-to-configure-azure-database-for-mysql-data-in-replication"></a>Configuración de la replicación de datos internos de Azure Database for MySQL

En este artículo se describe cómo configurar [la replicación de datos de entrada](concepts-data-in-replication.md) en Azure Database for MySQL mediante la configuración de los servidores maestros y de réplica. En este artículo se asume que tiene alguna experiencia previa con servidores y bases de datos MySQL.

> [!NOTE]
> Comunicación sin prejuicios
>
> Microsoft admite un entorno diverso e inclusivo. En este artículo se incluyen referencias a la palabra _esclavo_. En la [guía de estilo para la comunicación sin prejuicios](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) de Microsoft se reconoce que se trata de una palabra excluyente. Se usa en este artículo por coherencia, ya que actualmente es la palabra que aparece en el software. Cuando el software se actualice para quitarla, este artículo se actualizará en consecuencia.
>

Para crear una réplica en el servicio Azure Database for MySQL, [la replicación de datos de entrada](concepts-data-in-replication.md) sincroniza los datos que proceden de un servidor MySQL maestro local con máquinas virtuales (VM) o servicios de base de datos en la nube. Data-in Replication se basa en la replicación según la posición del archivo de registro binario (binlog), que es nativa de MySQL. Para obtener más información acerca de la replicación de binlog, consulte la [Introducción a la replicación de binlog de MySQL](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html).

Revise las [limitaciones y los requisitos](concepts-data-in-replication.md#limitations-and-considerations) de la replicación de datos de entrada antes de seguir los pasos de este artículo.

## <a name="create-a-mysql-server-to-be-used-as-replica"></a>Creación de un servidor de MySQL que se utilizará como réplica

1. Creación de un servidor de Azure Database for MySQL

   Cree un nuevo servidor MySQL (ejemplo, "replica.mysql.database.azure.com"). Consulte [Creación de un servidor de Azure Database for MySQL mediante Azure Portal](quickstart-create-mysql-server-database-using-azure-portal.md) para la creación del servidor. Este servidor es el servidor de "réplica" en la replicación de datos internos.

   > [!IMPORTANT]
   > El servidor de Azure Database for MySQL se debe crear en los planes de tarifa De uso general u Optimizada para memoria.
   > 

1. Creación de las mismas cuentas de usuario y los privilegios correspondientes

   Las cuentas de usuario no se replican desde el servidor maestro al servidor de réplica. Si planea proporcionar a los usuarios acceso al servidor de réplica, necesita crear manualmente todas las cuentas y privilegios correspondientes en este servidor recién creado de Azure Database for MySQL.

1. Agregue la dirección IP del servidor maestro a las reglas de firewall de la réplica. 

   Actualice las reglas de firewall mediante [Azure Portal](howto-manage-firewall-using-portal.md) o la [CLI de Azure](howto-manage-firewall-using-cli.md).

## <a name="configure-the-master-server"></a>Configurar el servidor maestro
En los siguientes pasos se prepara y configura el servidor MySQL en el entorno local, en una máquina virtual o en un servicio de base de datos hospedado por otros proveedores de nube para Data-in Replication. Este servidor es el "maestro" en Replicación de datos de entrada.


1. Revise los [requisitos del servidor maestro](concepts-data-in-replication.md#requirements) antes de continuar. 

   Por ejemplo, asegúrese de que el servidor maestro permite el tráfico entrante y saliente en el puerto 3306 y de que el servidor maestro tiene una **dirección IP pública**, el DNS es accesible públicamente o tiene un nombre de dominio completo (FQDN). 
   
   Pruebe la conectividad con el servidor maestro; para ello, pruebe a conectarse desde una herramienta como la línea de comandos de MySQL hospedada en otra máquina o desde la instancia de [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview) disponible en Azure Portal.

1. Activación del registro binario

   Compruebe si se ha habilitado el registro binario en el servidor maestro mediante la ejecución del comando siguiente: 

   ```sql
   SHOW VARIABLES LIKE 'log_bin';
   ```

   Si la variable [`log_bin`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_log_bin) se devuelve con el valor "ON", el registro binario está habilitado en el servidor. 

   Si `log_bin` se devuelve con el valor "OFF", active el registro binario mediante la edición del archivo my.cnf de modo que `log_bin=ON`, y reinicie el servidor para que el cambio se aplique.

1. Configuración del servidor maestro

   Replicación de datos de entrada requiere el parámetro `lower_case_table_names` para ser coherente entre los servidores maestro y de réplica. Este parámetro es 1 de forma predeterminada en Azure Database for MySQL. 

   ```sql
   SET GLOBAL lower_case_table_names = 1;
   ```

1. Creación de un nuevo rol de replicación y configuración de permisos

   Creación de una cuenta de usuario en el servidor maestro que está configurado con privilegios de replicación. Esto puede realizarse a través de los comandos SQL o una herramienta como MySQL Workbench. Tenga en cuenta si planea replicar con SSL, ya que necesitará especificarse al crear el usuario. Consulte la documentación de MySQL para entender cómo [agregar cuentas de usuario](https://dev.mysql.com/doc/refman/5.7/en/user-names.html) en el servidor maestro. 

   En los siguientes comandos, el nuevo rol de replicación creado es capaz de acceder al servidor maestro desde cualquier máquina, no solo desde la máquina que hospeda al propio servidor maestro en sí. Esto se hace especificando "syncuser@'%'" en el comando create user. Consulte la documentación de MySQL para más información acerca de la [especificación de nombres de cuenta](https://dev.mysql.com/doc/refman/5.7/en/account-names.html).

   **Comando SQL**

   *Replicación con SSL*

   Para requerir SSL a todas las conexiones de usuario, use el comando siguiente para crear un usuario: 

   ```sql
   CREATE USER 'syncuser'@'%' IDENTIFIED BY 'yourpassword';
   GRANT REPLICATION SLAVE ON *.* TO ' syncuser'@'%' REQUIRE SSL;
   ```

   *Replicación sin SSL*

   Si SSL no es necesario para todas las conexiones, use el comando siguiente para crear un usuario:

   ```sql
   CREATE USER 'syncuser'@'%' IDENTIFIED BY 'yourpassword';
   GRANT REPLICATION SLAVE ON *.* TO ' syncuser'@'%';
   ```

   **MySQL Workbench**

   Para crear el rol de replicación en MySQL Workbench, abra el panel **Users and Privileges** (Usuarios y privilegios) desde el panel **Management** (Administración). A continuación, haga clic en **Add Account** (Agregar cuenta). 
 
   ![Usuarios y privilegios](./media/howto-data-in-replication/users_privileges.png)

   Escriba el nombre de usuario en el campo **Login Name** (Nombre de inicio de sesión). 

   ![Usuario de sincronización](./media/howto-data-in-replication/syncuser.png)
 
   Haga clic en el panel **Administrative Roles** (Roles administrativos) y seleccione **Replication Slave** (Servidor subordinado de replicación) en la lista **Global Privileges** (Privilegios globales). A continuación, haga clic en **Apply** (Aplicar) para crear el rol de replicación.

   ![Servidor subordinado de replicación](./media/howto-data-in-replication/replicationslave.png)

1. Establecer el servidor maestro en el modo de solo lectura

   Antes de comenzar a volcar la base de datos, el servidor debe colocarse en modo de solo lectura. En modo de solo lectura, el servidor maestro podrá procesar cualquier transacción de escritura. Evalúe el impacto para su negocio y la programación de la ventana de solo lectura en un momento de poco tráfico, si es necesario.

   ```sql
   FLUSH TABLES WITH READ LOCK;
   SET GLOBAL read_only = ON;
   ```

1. Obtenga el nombre de archivo de registro binario y el desplazamiento

   Ejecute el comando [`show master status`](https://dev.mysql.com/doc/refman/5.7/en/show-master-status.html) para determinar el nombre de archivo de registro binario actual y el desplazamiento.
    
   ```sql
   show master status;
   ```
   Los resultados deben ser como el siguiente. Asegúrese de anotar el nombre del archivo binario que se utilizará en pasos posteriores.

   ![Resultados de estado del maestro](./media/howto-data-in-replication/masterstatus.png)
 
## <a name="dump-and-restore-master-server"></a>Volcar y restaurar el servidor maestro

1. Determine las bases de datos y tablas que desea replicar en Azure Database for MySQL y realice el volcado desde el servidor maestro.
 
    Puede usar mysqldump para volcar bases de datos desde el servidor maestro. Para más información, consulte [Volcado y restauración](concepts-migrate-dump-restore.md). No es necesario volcar la biblioteca de MySQL y la biblioteca de prueba.

1. Establecer el servidor maestro en modo de lectura/escritura

   Cuando se haya volcado la base de datos, cambie el servidor de MySQL maestro de nuevo al modo de lectura/escritura.

   ```sql
   SET GLOBAL read_only = OFF;
   UNLOCK TABLES;
   ```

1. Restauración del archivo de volcado en el nuevo servidor

   Restaure el archivo de volcado en el servidor creado en el servicio Azure Database for MySQL. Consulte [Volcado y restauración](concepts-migrate-dump-restore.md) para saber cómo restaurar un archivo de volcado en un servidor MySQL. Si el archivo de volcado es grande, cárguelo a una máquina virtual en Azure dentro de la misma región que el servidor de réplica. Restáurelo en el servidor de Azure Database for MySQL desde la máquina virtual.

## <a name="link-master-and-replica-servers-to-start-data-in-replication"></a>Vincular los servidores maestro y de réplica para iniciar Replicación de datos de entrada

1. Establecer el servidor maestro

   Todas las funciones de la replicación de datos internos se realizan mediante los procedimientos almacenados. Puede encontrar todos los procedimientos en [Procedimientos almacenados de replicación de datos internos](reference-data-in-stored-procedures.md). Los procedimientos almacenados se pueden ejecutar en el shell de MySQL o en MySQL Workbench. 

   Para vincular dos servidores e iniciar la replicación, inicie sesión en el servidor de réplica de destino en el servicio Azure Database for MySQL y establezca la instancia externa como servidor maestro. Esto se realiza mediante el procedimiento almacenado `mysql.az_replication_change_master` en el servidor Azure Database for MySQL.

   ```sql
   CALL mysql.az_replication_change_master('<master_host>', '<master_user>', '<master_password>', 3306, '<master_log_file>', <master_log_pos>, '<master_ssl_ca>');
   ```

   - master_host: nombre de host del servidor maestro
   - master_user: nombre de usuario para el servidor maestro
   - master_password: contraseña para el servidor maestro
   - master_log_file: nombre del archivo de registro binario procedente de la ejecución de `show master status`
   - master_log_pos: posición del registro binario procedente de la ejecución de `show master status`
   - master_ssl_ca: contexto del certificado de entidad de certificación. Si no usa SSL, pase una cadena vacía.
       - Se recomienda pasar este parámetro como una variable. Consulte los siguientes ejemplos para más información.

   > [!NOTE]
   > Si el servidor maestro se hospeda en una VM de Azure, establezca la opción "Permitir el acceso a servicios de Azure" en "Activado" para permitir que los servidores principal y réplica se comuniquen entre sí. Esta configuración se puede cambiar desde las opciones de **seguridad de conexión**. Consulte [Manage firewall rules using portal](howto-manage-firewall-using-portal.md) (Administrar reglas de firewall mediante el portal) para obtener más información.
      
   **Ejemplos**
   
   *Replicación con SSL*
   
   La variable `@cert` se crea mediante la ejecución de los siguientes comandos de MySQL: 
   
      ```sql
      SET @cert = '-----BEGIN CERTIFICATE-----
      PLACE YOUR PUBLIC KEY CERTIFICATE'`S CONTEXT HERE
      -----END CERTIFICATE-----'
      ```
   
   La replicación con SSL se configura entre un servidor maestro hospedado en el dominio "companya.com" y un servidor de réplica hospedado en Azure Database for MySQL. Este procedimiento almacenado se ejecuta en la réplica. 
   
      ```sql
      CALL mysql.az_replication_change_master('master.companya.com', 'syncuser', 'P@ssword!', 3306, 'mysql-bin.000002', 120, @cert);
      ```
   *Replicación sin SSL*
   
   La replicación sin SSL se configura entre un servidor maestro hospedado en el dominio "companya.com" y un servidor de réplica hospedado en Azure Database for MySQL. Este procedimiento almacenado se ejecuta en la réplica.
   
      ```sql
      CALL mysql.az_replication_change_master('master.companya.com', 'syncuser', 'P@ssword!', 3306, 'mysql-bin.000002', 120, '');
      ```

1. Filtrado 
 
   Si desea omitir la replicación de algunas tablas del maestro, actualice el parámetro de servidor `replicate_wild_ignore_table` en el servidor de réplica. Puede proporcionar más de un patrón de tabla mediante una lista separada por comas.

   Consulte la [documentación de MySQL](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-wild-ignore-table) para más información acerca de este parámetro. 
    
   Para actualizar el parámetro, use [Azure Portal](howto-server-parameters.md) o la [CLI de Azure](howto-configure-server-parameters-using-cli.md).

1. Inicio de la replicación

   Llame al procedimiento almacenado `mysql.az_replication_start` para iniciar la replicación.

   ```sql
   CALL mysql.az_replication_start;
   ```

1. Comprobación del estado de replicación

   Llame al comando [`show slave status`](https://dev.mysql.com/doc/refman/5.7/en/show-slave-status.html) en el servidor de réplica para ver el estado de replicación.
    
   ```sql
   show slave status;
   ```

   Si el estado de `Slave_IO_Running` y `Slave_SQL_Running` es "yes" y el valor de `Seconds_Behind_Master` es "0", la replicación funciona correctamente. `Seconds_Behind_Master` indica el tiempo de retraso de la réplica. Si el valor no es "0", significa que la réplica procesa actualizaciones. 

## <a name="other-stored-procedures"></a>Otros procedimientos almacenados

### <a name="stop-replication"></a>Detención replicación

Para detener la replicación entre el servidor maestro y de réplica, use el siguiente procedimiento almacenado:

```sql
CALL mysql.az_replication_stop;
```

### <a name="remove-replication-relationship"></a>Eliminación de la relación de replicación

Para quitar la relación entre el servidor maestro y de réplica, use el siguiente procedimiento almacenado:

```sql
CALL mysql.az_replication_remove_master;
```

### <a name="skip-replication-error"></a>Omisión de error de replicación

Para omitir un error de replicación y permitir que continúe la replicación, use el siguiente procedimiento almacenado:
    
```sql
CALL mysql.az_replication_skip_counter;
```

## <a name="next-steps"></a>Pasos siguientes
- Aprenda más sobre [Data-in Replication](concepts-data-in-replication.md) para Azure Database for MySQL. 
