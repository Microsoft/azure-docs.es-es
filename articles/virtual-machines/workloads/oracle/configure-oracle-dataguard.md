---
title: Implementación de Oracle Data Guard en una máquina virtual Linux en Azure | Microsoft Docs
description: Ponga en funcionamiento rápidamente una instancia de Oracle Data Guard en el entorno de Azure.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: rgardler
manager: ''
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/02/2018
ms.author: rogardle
ms.openlocfilehash: c6b064df1fe8943f9202446fb2857d50bcb4e0e1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083385"
---
# <a name="implement-oracle-data-guard-on-an-azure-linux-virtual-machine"></a>Implementación de Oracle Data Guard en una máquina virtual Linux en Azure 

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En este artículo se describe cómo usar la CLI de Azure para implementar una base de datos de Oracle Database 12c desde una imagen de la imagen de Azure Marketplace. Por lo tanto, este artículo le muestra paso a paso cómo instalar y configurar Data Guard en una máquina virtual (VM) de Azure.

Antes de comenzar, asegúrese de que esté instalada la CLI de Azure. Para obtener más información, vea la [guía de instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="prepare-the-environment"></a>Preparación del entorno
### <a name="assumptions"></a>Supuestos

Para instalar Oracle Data Guard, debe crear dos máquinas virtuales de Azure en el mismo conjunto de disponibilidad:

- La máquina virtual principal (myVM1) tiene una instancia de Oracle en ejecución.
- La máquina virtual en espera (myVM2) solo tiene el software de Oracle instalado.

La imagen de Marketplace que se usa para crear las máquinas virtuales es Oracle:Oracle-Database-Ee:12.1.0.2:latest.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure 

Inicie sesión en la suscripción de Azure con el comando [az login](/cli/azure/reference-index) y siga las instrucciones de la pantalla.

```azurecli
az login
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

En el ejemplo siguiente se crea un grupo de recursos denominado `myResourceGroup` en la ubicación `westus`:

```azurecli
az group create --name myResourceGroup --location westus
```

### <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad

La creación de un conjunto de disponibilidad es opcional, pero es recomendable. Para más información, consulte las [directrices de conjuntos de disponibilidad de Azure](../../windows/infrastructure-example.md).

```azurecli
az vm availability-set create \
    --resource-group myResourceGroup \
    --name myAvailabilitySet \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

### <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Cree una máquina virtual con el comando [az vm create](/cli/azure/vm). 

En el ejemplo siguiente se crean dos máquinas virtuales, denominadas `myVM1` y `myVM2`. También se crean claves SSH si aún no existen en una ubicación de claves predeterminada. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`.

Cree myVM1 (principal):
```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM1 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --admin-username azureuser \
     --generate-ssh-keys \
```

Después de crear la máquina virtual, la CLI de Azure muestra información similar al ejemplo siguiente. Tome nota del valor de `publicIpAddress`. Use esta dirección para acceder a la máquina virtual.

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "westus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "myResourceGroup"
}
```

Cree myVM2 (en espera):

```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM2 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --admin-username azureuser \
     --generate-ssh-keys \
```

Tenga en cuenta el valor de `publicIpAddress` después de crear myVM2.

### <a name="open-the-tcp-port-for-connectivity"></a>Apertura del puerto TCP para la conectividad

En este paso se configuran puntos de conexión externos, lo que permite el acceso remoto a la base de datos de Oracle.

Abra el puerto para myVM1:

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVM1NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

El resultado tiene que ser similar a la siguiente respuesta:

```output
{
  "access": "Allow",
  "description": null,
  "destinationAddressPrefix": "*",
  "destinationPortRange": "1521",
  "direction": "Inbound",
  "etag": "W/\"bd77dcae-e5fd-4bd6-a632-26045b646414\"",
  "id": "/subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myVmNSG/securityRules/allow-oracle",
  "name": "allow-oracle",
  "priority": 999,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sourceAddressPrefix": "*",
  "sourcePortRange": "*"
}
```

Abra el puerto para myVM2:

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVM2NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

### <a name="connect-to-the-virtual-machine"></a>Conexión a la máquina virtual

Ejecute el comando siguiente para crear una sesión SSH con la máquina virtual. Reemplace la dirección IP por el valor de `publicIpAddress` de la máquina virtual.

```bash 
$ ssh azureuser@<publicIpAddress>
```

### <a name="create-the-database-on-myvm1-primary"></a>Creación de la base de datos en myVM1 (principal)

El software de Oracle ya está instalado en la imagen de Marketplace, por lo que el siguiente paso es instalar la base de datos. 

Cambie al superusuario de Oracle:

```bash
$ sudo su - oracle
```

Cree la base de datos:

```bash
$ dbca -silent \
   -createDatabase \
   -templateName General_Purpose.dbc \
   -gdbname cdb1 \
   -sid cdb1 \
   -responseFile NO_VALUE \
   -characterSet AL32UTF8 \
   -sysPassword OraPasswd1 \
   -systemPassword OraPasswd1 \
   -createAsContainerDatabase true \
   -numberOfPDBs 1 \
   -pdbName pdb1 \
   -pdbAdminPassword OraPasswd1 \
   -databaseType MULTIPURPOSE \
   -automaticMemoryManagement false \
   -storageType FS \
   -ignorePreReqs
```

Los resultados tienen que ser similares a la siguiente respuesta:

```output
Copying database files
1% complete
2% complete
8% complete
13% complete
19% complete
27% complete
Creating and starting Oracle instance
29% complete
32% complete
33% complete
34% complete
38% complete
42% complete
43% complete
45% complete
Completing Database Creation
48% complete
51% complete
53% complete
62% complete
70% complete
72% complete
Creating Pluggable Databases
78% complete
100% complete
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/cdb1/cdb1.log" for further details.
```

Establezca las variables ORACLE_SID y ORACLE_HOME:

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
```

Opcionalmente, puede agregar ORACLE_HOME y ORACLE_SID al archivo /home/oracle/.bashrc para que esta configuración se guarde para futuros inicios de sesión:

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
```

## <a name="configure-data-guard"></a>Configuración de Data Guard

### <a name="enable-archive-log-mode-on-myvm1-primary"></a>Habilitar el modo de archivar registro en myVM1 (principal)

```bash
$ sqlplus / as sysdba
SQL> SELECT log_mode FROM v$database;

LOG_MODE
------------
NOARCHIVELOG

SQL> SHUTDOWN IMMEDIATE;
SQL> STARTUP MOUNT;
SQL> ALTER DATABASE ARCHIVELOG;
SQL> ALTER DATABASE OPEN;
```

Habilite el registro forzado y asegúrese de que, al menos, haya un archivo de registro está presente:

```bash
SQL> ALTER DATABASE FORCE LOGGING;
SQL> ALTER SYSTEM SWITCH LOGFILE;
```

Cree registros de rehacer en espera:

```bash
SQL> ALTER DATABASE ADD STANDBY LOGFILE ('/u01/app/oracle/oradata/cdb1/standby_redo01.log') SIZE 50M;
SQL> ALTER DATABASE ADD STANDBY LOGFILE ('/u01/app/oracle/oradata/cdb1/standby_redo02.log') SIZE 50M;
SQL> ALTER DATABASE ADD STANDBY LOGFILE ('/u01/app/oracle/oradata/cdb1/standby_redo03.log') SIZE 50M;
SQL> ALTER DATABASE ADD STANDBY LOGFILE ('/u01/app/oracle/oradata/cdb1/standby_redo04.log') SIZE 50M;
```

Active la base de datos flashback (que anteriormente se utilizó numerosas veces para la recuperación) y establezca el valor de STANDBY\_FILE\_MANAGEMENT en Automático. Salga de SQL * Plus a continuación.

```bash
SQL> ALTER DATABASE FLASHBACK ON;
SQL> ALTER SYSTEM SET STANDBY_FILE_MANAGEMENT=AUTO;
SQL> EXIT;
```

### <a name="set-up-service-on-myvm1-primary"></a>Configuración del servicio en myVM1 (principal)

Edite o cree el archivo tnsnames.ora, que se encuentra en la carpeta $ORACLE_HOME\network\admin.

Agregue las siguientes entradas:

```bash
cdb1 =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = myVM1)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = cdb1)
    )
  )

cdb1_stby =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = myVM2)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = cdb1)
    )
  )
```

Edite o cree el archivo listener.ora, que se encuentra en la carpeta $ORACLE_HOME\network\admin.

Agregue las siguientes entradas:

```bash
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = myVM1)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = cdb1_DGMGRL)
      (ORACLE_HOME = /u01/app/oracle/product/12.1.0/dbhome_1)
      (SID_NAME = cdb1)
    )
  )

ADR_BASE_LISTENER = /u01/app/oracle
```

Habilite el agente de Data Guard:

```bash
$ sqlplus / as sysdba
SQL> ALTER SYSTEM SET dg_broker_start=true;
SQL> EXIT;
```

Inicie el agente de escucha:

```bash
$ lsnrctl stop
$ lsnrctl start
```

### <a name="set-up-service-on-myvm2-standby"></a>Configuración del servicio en myVM2 (en espera)

Establezca SSH en myVM2:

```bash 
$ ssh azureuser@<publicIpAddress>
```

Inicie sesión como Oracle:

```bash
$ sudo su - oracle
```

Edite o cree el archivo tnsnames.ora, que se encuentra en la carpeta $ORACLE_HOME\network\admin.

Agregue las siguientes entradas:

```bash
cdb1 =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = myVM1)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = cdb1)
    )
  )

cdb1_stby =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = myVM2)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = cdb1)
    )
  )
```

Edite o cree el archivo listener.ora, que se encuentra en la carpeta $ORACLE_HOME\network\admin.

Agregue las siguientes entradas:

```bash
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = myVM2)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = cdb1_DGMGRL)
      (ORACLE_HOME = /u01/app/oracle/product/12.1.0/dbhome_1)
      (SID_NAME = cdb1)
    )
  )

ADR_BASE_LISTENER = /u01/app/oracle
```

Inicie el agente de escucha:

```bash
$ lsnrctl stop
$ lsnrctl start
```


### <a name="restore-the-database-to-myvm2-standby"></a>Restauración de la base de datos myVM2 (en espera)

Cree un archivo de parámetros /tmp/initcdb1_stby.ora con el contenido siguiente:

```bash
*.db_name='cdb1'
```

Cree carpetas:

```bash
mkdir -p /u01/app/oracle/oradata/cdb1/pdbseed
mkdir -p /u01/app/oracle/oradata/cdb1/pdb1
mkdir -p /u01/app/oracle/fast_recovery_area/cdb1
mkdir -p /u01/app/oracle/admin/cdb1/adump
```

Cree un archivo de contraseña:

```bash
$ orapwd file=/u01/app/oracle/product/12.1.0/dbhome_1/dbs/orapwcdb1 password=OraPasswd1 entries=10
```

Inicie la base de datos en myVM2:

```bash
$ export ORACLE_SID=cdb1
$ sqlplus / as sysdba

SQL> STARTUP NOMOUNT PFILE='/tmp/initcdb1_stby.ora';
SQL> EXIT;
```

Restaure la base de datos mediante la herramienta RMAN:

```bash
$ rman TARGET sys/OraPasswd1@cdb1 AUXILIARY sys/OraPasswd1@cdb1_stby
```

Ejecute los siguientes comandos en RMAN:

```bash
DUPLICATE TARGET DATABASE
  FOR STANDBY
  FROM ACTIVE DATABASE
  DORECOVER
  SPFILE
    SET db_unique_name='CDB1_STBY' COMMENT 'Is standby'
  NOFILENAMECHECK;
```

Verá mensajes similares al siguiente cuando se complete el comando. Salga de RMAN.

```output
media recovery complete, elapsed time: 00:00:00
Finished recover at 29-JUN-17
Finished Duplicate Db at 29-JUN-17
```

```bash
RMAN> EXIT;
```

Opcionalmente, puede agregar ORACLE_HOME y ORACLE_SID al archivo /home/oracle/.bashrc para que esta configuración se guarde para futuros inicios de sesión:

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
```

Habilite el agente de Data Guard:
```bash
$ sqlplus / as sysdba
SQL> ALTER SYSTEM SET dg_broker_start=true;
SQL> EXIT;
```

### <a name="configure-data-guard-broker-on-myvm1-primary"></a>Configuración del agente de Data Guard en myVM1 (principal)

Inicie el administrador de Data Guard e inicie sesión mediante SYS y una contraseña. (No utilice la autenticación de sistema operativo). Lleve a cabo los siguientes pasos:

```bash
$ dgmgrl sys/OraPasswd1@cdb1
DGMGRL for Linux: Version 12.1.0.2.0 - 64bit Production

Copyright (c) 2000, 2013, Oracle. All rights reserved.

Welcome to DGMGRL, type "help" for information.
Connected as SYSDBA.
DGMGRL> CREATE CONFIGURATION my_dg_config AS PRIMARY DATABASE IS cdb1 CONNECT IDENTIFIER IS cdb1;
Configuration "my_dg_config" created with primary database "cdb1"
DGMGRL> ADD DATABASE cdb1_stby AS CONNECT IDENTIFIER IS cdb1_stby MAINTAINED AS PHYSICAL;
Database "cdb1_stby" added
DGMGRL> ENABLE CONFIGURATION;
Enabled.
```

Revise la configuración:

```bash
DGMGRL> SHOW CONFIGURATION;

Configuration - my_dg_config

  Protection Mode: MaxPerformance
  Members:
  cdb1      - Primary database
    cdb1_stby - Physical standby database

Fast-Start Failover: DISABLED

Configuration Status:
SUCCESS   (status updated 26 seconds ago)
```

Ha completado la instalación de Oracle Data Guard. En la sección siguiente se muestra cómo probar la conectividad y el cambio.

### <a name="connect-the-database-from-the-client-machine"></a>Conexión de la base de datos desde el equipo cliente

Actualice o cree el archivo tnsnames.ora en el equipo cliente. Este archivo suele encontrarse en $ORACLE_HOME\network\admin.

Reemplace las direcciones IP por sus valores de `publicIpAddress` para myVM1 y myVM2:

```bash
cdb1=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=<myVM1 IP address>)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVER=dedicated)
      (SERVICE_NAME=cdb1)
    )
  )

cdb1_stby=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=<myVM2 IP address>)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVER=dedicated)
      (SERVICE_NAME=cdb1_stby)
    )
  )
```

Inicie SQL *Plus:

```bash
$ sqlplus sys/OraPasswd1@cdb1
SQL*Plus: Release 12.2.0.1.0 Production on Wed May 10 14:18:31 2017

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL>
```

## <a name="test-the-data-guard-configuration"></a>Prueba de la configuración de Data Guard

### <a name="switch-over-the-database-on-myvm1-primary"></a>Cambio de la base de datos en myVM1 (principal)

Para cambiar de la principal a la que en espera (de cdb1 a cdb1_stby):

```bash
$ dgmgrl sys/OraPasswd1@cdb1
DGMGRL for Linux: Version 12.1.0.2.0 - 64bit Production

Copyright (c) 2000, 2013, Oracle. All rights reserved.

Welcome to DGMGRL, type "help" for information.
Connected as SYSDBA.
DGMGRL> SWITCHOVER TO cdb1_stby;
Performing switchover NOW, please wait...
Operation requires a connection to instance "cdb1" on database "cdb1_stby"
Connecting to instance "cdb1"...
Connected as SYSDBA.
New primary database "cdb1_stby" is opening...
Operation requires start up of instance "cdb1" on database "cdb1"
Starting instance "cdb1"...
ORACLE instance started.
Database mounted.
Switchover succeeded, new primary is "cdb1_stby"
DGMGRL>
```

Ahora puede conectarse a la base de datos en espera.

Inicie SQL *Plus:

```bash

$ sqlplus sys/OraPasswd1@cdb1_stby
SQL*Plus: Release 12.2.0.1.0 Production on Wed May 10 14:18:31 2017

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL>
```

### <a name="switch-over-the-database-on-myvm2-standby"></a>Cambio de la base de datos en myVM2 (en espera)

Para cambiar, ejecute lo siguiente en myVM2:

```bash
$ dgmgrl sys/OraPasswd1@cdb1_stby
DGMGRL for Linux: Version 12.1.0.2.0 - 64bit Production

Copyright (c) 2000, 2013, Oracle. All rights reserved.

Welcome to DGMGRL, type "help" for information.
Connected as SYSDBA.
DGMGRL> SWITCHOVER TO cdb1;
Performing switchover NOW, please wait...
Operation requires a connection to instance "cdb1" on database "cdb1"
Connecting to instance "cdb1"...
Connected as SYSDBA.
New primary database "cdb1" is opening...
Operation requires start up of instance "cdb1" on database "cdb1_stby"
Starting instance "cdb1"...
ORACLE instance started.
Database mounted.
Switchover succeeded, new primary is "cdb1"
```

Una vez más, debería poder conectarse a la base de datos principal.

Inicie SQL *Plus:

```bash

$ sqlplus sys/OraPasswd1@cdb1
SQL*Plus: Release 12.2.0.1.0 Production on Wed May 10 14:18:31 2017

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL>
```

Ha completado la instalación y configuración de Data Guard en Oracle Linux.


## <a name="delete-the-virtual-machine"></a>Eliminación de la máquina virtual

Cuando ya no necesite la máquina virtual, puede usar el comando siguiente para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados:

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

[Tutorial: Creación de máquinas virtuales de alta disponibilidad](../../linux/create-cli-complete.md)

[Ejemplos de la CLI de Azure para implementación de máquinas virtuales](../../linux/cli-samples.md)
