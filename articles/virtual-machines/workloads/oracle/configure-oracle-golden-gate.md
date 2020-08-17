---
title: Implementación de Oracle Golden Gate en máquinas virtuales Linux de Azure | Microsoft Docs
description: Ponga en funcionamiento rápidamente una base de datos de Oracle Golden Gate en el entorno de Azure.
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
ms.openlocfilehash: 5ed99fd6a16743846033313fcf13702f69f3e728
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088366"
---
# <a name="implement-oracle-golden-gate-on-an-azure-linux-vm"></a>Implementación de Oracle Golden Gate en máquinas virtuales Linux de Azure 

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En esta guía se detalla cómo usar la CLI de Azure para la implementación de la base de datos de Oracle 12c desde la imagen de la galería de Azure Marketplace. 

En este documento se muestra paso a paso cómo crear, instalar y configurar Oracle Golden Gate en una máquina virtual de Azure. En este tutorial, se configuran dos máquinas virtuales en un conjunto de disponibilidad en una sola región. Se puede usar el mismo tutorial para configurar OracleGolden Gate para máquinas virtuales en diferentes Availability Zones en una sola región de Azure o para la configuración de máquinas virtuales en dos regiones diferentes.

Antes de empezar, asegúrese de que se ha instalado la CLI de Azure. Para obtener más información, consulte la [guía de instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="prepare-the-environment"></a>Preparación del entorno

Para llevar a cabo la instalación de Oracle Golden Gate, debe crear dos máquinas virtuales de Azure en el mismo conjunto de disponibilidad. La imagen de Marketplace que se usa para crear las máquinas virtuales es **Oracle:Oracle-Database-Ee:12.1.0.2:latest**.

También debe estar familiarizado con vi, el editor de Unix, y tener un conocimiento básico de x11 (Windows X).

Lo siguiente es un resumen de la configuración del entorno:
> 
> |  | **Sitio principal** | **Sitio de réplica** |
> | --- | --- | --- |
> | **Versión de Oracle** |Oracle 12c Release 2 – (12.1.0.2) |Oracle 12c Release 2 – (12.1.0.2)|
> | **Nombre de la máquina** |myVM1 |myVM2 |
> | **Sistema operativo** |Oracle Linux 6.x |Oracle Linux 6.x |
> | **SID de Oracle** |CDB1 |CDB1 |
> | **Esquema de replicación** |PRUEBA|PRUEBA |
> | **Propietario/réplica de Golden Gate** |C##GGADMIN |REPUSER |
> | **Proceso de Golden Gate** |EXTORA |REPORA|


### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure 

Inicie sesión en su suscripción de Azure con el comando [az login](/cli/azure/reference-index). Después, siga las instrucciones que aparecen en pantalla.

```azurecli
az login
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y desde el que se pueden administrar los recursos de Azure. 

En el ejemplo siguiente, se crea un grupo de recursos denominado `myResourceGroup` en la ubicación `westus`.

```azurecli
az group create --name myResourceGroup --location westus
```

### <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad

El paso siguiente es opcional pero recomendable. Para más información, consulte la [guía de conjuntos de disponibilidad de Azure](../../windows/infrastructure-example.md).

```azurecli
az vm availability-set create \
    --resource-group myResourceGroup \
    --name myAvailabilitySet \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

### <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Cree la máquina virtual con el comando [az vm create](/cli/azure/vm). 

En el ejemplo siguiente se crean dos máquinas virtuales, denominadas `myVM1` y `myVM2`. También se crean claves SSH, si aún no existen en una ubicación de claves predeterminada. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`.

#### <a name="create-myvm1-primary"></a>Cree myVM1 (principal):

```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM1 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --generate-ssh-keys \
```

Después de crear la máquina virtual, la CLI de Azure muestra información similar al ejemplo siguiente. (Anote el valor de `publicIpAddress`. Esta dirección se utiliza para acceder a la máquina virtual).

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

#### <a name="create-myvm2-replicate"></a>Cree myVM2 (réplica):

```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM2 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --generate-ssh-keys \
```

Tome nota de `publicIpAddress` también después de su creación.

### <a name="open-the-tcp-port-for-connectivity"></a>Apertura del puerto TCP para la conectividad

El siguiente paso consiste en configurar puntos de conexión externos, que le permiten tener acceso remoto a la base de datos de Oracle. Para configurar los puntos de conexión externos, ejecute los siguientes comandos.

#### <a name="open-the-port-for-myvm1"></a>Abra el puerto para myVM1:

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVm1NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

Los resultados tienen que ser similares a la siguiente respuesta:

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

#### <a name="open-the-port-for-myvm2"></a>Abra el puerto para myVM2:

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVm2NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

### <a name="connect-to-the-virtual-machine"></a>Conexión a la máquina virtual

Ejecute el comando siguiente para crear una sesión SSH con la máquina virtual. Reemplace la dirección IP por el valor de `publicIpAddress` de la máquina virtual.

```bash
ssh <publicIpAddress>
```

### <a name="create-the-database-on-myvm1-primary"></a>Creación de la base de datos en myVM1 (principal)

El software de Oracle ya está instalado en la imagen de Marketplace, por lo que el siguiente paso es instalar la base de datos. 

Ejecute el software como el superusuario "oracle":

```bash
sudo su - oracle
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
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/cdb1/cdb1.log" for more details.
```

Establezca las variables ORACLE_SID y ORACLE_HOME.

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
$ LD_LIBRARY_PATH=ORACLE_HOME/lib; export LD_LIBRARY_PATH
```

Opcionalmente, puede agregar ORACLE_HOME y ORACLE_SID al archivo .bashrc, para que esta configuración se guarde para futuros inicios de sesión.

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
# add Oracle library path
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

### <a name="start-oracle-listener"></a>Inicio del agente de escucha de Oracle

```bash
$ lsnrctl start
```

### <a name="create-the-database-on-myvm2-replicate"></a>Creación de la base de datos en myVM2 (réplica)

```bash
sudo su - oracle
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

Establezca las variables ORACLE_SID y ORACLE_HOME.

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
$ LD_LIBRARY_PATH=ORACLE_HOME/lib; export LD_LIBRARY_PATH
```

Opcionalmente, puede agregar ORACLE_HOME y ORACLE_SID al archivo .bashrc, para que esta configuración se guarde para futuros inicios de sesión.

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
# add Oracle library path
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

### <a name="start-oracle-listener"></a>Inicio del agente de escucha de Oracle

```bash
$ sudo su - oracle
$ lsnrctl start
```

## <a name="configure-golden-gate"></a>Configuración de Golden Gate 
Para configurar Golden Gate, realice los pasos de esta sección.

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
Habilite el registro forzado y asegúrese de que, al menos, haya un archivo de registro está presente.

```bash
SQL> ALTER DATABASE FORCE LOGGING;
SQL> ALTER SYSTEM SWITCH LOGFILE;
SQL> ALTER SYSTEM set enable_goldengate_replication=true;
SQL> ALTER PLUGGABLE DATABASE PDB1 OPEN;
SQL> ALTER SESSION SET CONTAINER=PDB1;
SQL> ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
SQL> EXIT;
```

### <a name="download-golden-gate-software"></a>Descargar el software de Golden Gate
Para descargar y preparar el software Oracle Golden Gate, complete los pasos siguientes:

1. Descargue el archivo **fbo_ggs_Linux_x64_shiphome.zip** desde la [página de descarga de Oracle Golden Gate](https://www.oracle.com/technetwork/middleware/goldengate/downloads/index.html). En el título de descarga **Oracle GoldenGate 12.x.x.x for Oracle Linux x86-64**, debe haber un conjunto de archivos .zip para descargar.

2. Después de descargar los archivos .zip en el equipo cliente, use el protocolo de copia segura (SCP) para copiar los archivos a la máquina virtual.

   ```bash
   $ scp fbo_ggs_Linux_x64_shiphome.zip <publicIpAddress>:<folder>
   ```

3. Mueva los archivos .zip a la carpeta **/opt**. A continuación, cambie el propietario de los archivos de la forma siguiente:

   ```bash
   $ sudo su -
   # mv <folder>/*.zip /opt
   ```

4. Descomprima los archivos (instale la utilidad de descompresión de Linux si aún no está instalada):

   ```bash
   # yum install unzip
   # cd /opt
   # unzip fbo_ggs_Linux_x64_shiphome.zip
   ```

5. Cambie el permiso:

   ```bash
   # chown -R oracle:oinstall /opt/fbo_ggs_Linux_x64_shiphome
   ```

### <a name="prepare-the-client-and-vm-to-run-x11-for-windows-clients-only"></a>Preparación del cliente y la máquina virtual para ejecutar x11 (solo para clientes Windows)
Se trata de un paso opcional. Puede omitir este paso si está utilizando un cliente Linux o ya ha configurado x11.

1. Descargue PuTTY y Xming en el equipo Windows:

   * [Descargar PuTTY](https://www.putty.org/)
   * [Descargar Xming](https://xming.en.softonic.com/)

2. Después de instalar PuTTY, en la carpeta PuTTY (por ejemplo, C:\Archivos de programa\PuTTY), ejecute puttygen.exe (PuTTY Key Generator).

3. En PuTTY Key Generator:

   - Para generar una clave, seleccione el botón **Generate** (Generar).
   - Copie el contenido de la clave (**Ctrl+C**).
   - Seleccione el botón **Save private key** (Guardar clave privada).
   - Pase por alto la advertencia que aparece y seleccione **OK** (Aceptar).

   ![Captura de pantalla de la página de PuTTY Key Generator](./media/oracle-golden-gate/puttykeygen.png)

4. En la máquina virtual, ejecute estos comandos:

   ```bash
   # sudo su - oracle
   $ mkdir .ssh (if not already created)
   $ cd .ssh
   ```

5. Cree un archivo denominado **authorized_keys**. Pegue el contenido de la clave en este archivo y guárdelo.

   > [!NOTE]
   > La clave debe contener la cadena `ssh-rsa`. Además, el contenido de la clave debe ser una sola línea de texto.
   >  

6. Inicie PuTTY. En el panel **Category** (Categoría), vaya a **Connection** > **SSH** > **Auth** (Conexión > SSH > Autenticación). En el cuadro **Private key file for authentication** (Archivo de clave privada para autenticación), vaya a la clave que generó antes.

   ![Captura de pantalla de la página para establecer la clave privada](./media/oracle-golden-gate/setprivatekey.png)

7. En el panel **Category** (Categoría), vaya a **Connection** > **SSH** > **X11** (Conexión > SSH > X11). Después, seleccione el cuadro **Enable X11 forwarding** (Habilitar reenvío a X11).

   ![Captura de pantalla de la página para habilitar X11](./media/oracle-golden-gate/enablex11.png)

8. En el panel **Category** (Categoría), vaya a **Session** (Sesión). Escriba la información del host y seleccione **Open** (Abrir).

   ![Captura de pantalla de la página Session (Sesión)](./media/oracle-golden-gate/puttysession.png)

### <a name="install-golden-gate-software"></a>Instalar el software de Golden Gate

Para instalar Oracle Golden Gate, complete los pasos siguientes:

1. Inicie sesión como oracle. (Debe poder iniciar sesión sin que se le solicite una contraseña). Asegúrese de que se esté ejecutando Xming antes de comenzar la instalación.

   ```bash
   $ cd /opt/fbo_ggs_Linux_x64_shiphome/Disk1
   $ ./runInstaller
   ```

2. Seleccione 'Oracle GoldenGate para Oracle Database 12c'. Después, seleccione **Siguiente** para continuar.

   ![Captura de pantalla de la página Select Installation (Seleccionar instalación) en el instalador](./media/oracle-golden-gate/golden_gate_install_01.png)

3. Cambie la ubicación del software. A continuación, active la casilla **Start Manager** (Iniciar administrador) y especifique la ubicación de la base de datos. Seleccione **Next** (Siguiente) para continuar.

   ![Captura de pantalla de la página Select Installation (Seleccionar instalación)](./media/oracle-golden-gate/golden_gate_install_02.png)

4. Cambie el directorio de inventario y, después, seleccione **Siguiente** para continuar.

   ![Captura de pantalla de la página Select Installation (Seleccionar instalación)](./media/oracle-golden-gate/golden_gate_install_03.png)

5. En la pantalla **Summary** (Resumen), seleccione **Install** (Instalar) para continuar.

   ![Captura de pantalla de la página Select Installation (Seleccionar instalación) en el instalador](./media/oracle-golden-gate/golden_gate_install_04.png)

6. Puede que se le pida que ejecute un script como 'raíz'. Si es así, abra una sesión independiente, envíe un ssh a la máquina virtual, un sudo a la raíz y, después, ejecute el script. Seleccione **Aceptar** para continuar.

   ![Captura de pantalla de la página Select Installation (Seleccionar instalación)](./media/oracle-golden-gate/golden_gate_install_05.png)

7. Cuando haya finalizado la instalación, seleccione **Cerrar** para completar el proceso.

   ![Captura de pantalla de la página Select Installation (Seleccionar instalación)](./media/oracle-golden-gate/golden_gate_install_06.png)

### <a name="set-up-service-on-myvm1-primary"></a>Configuración del servicio en myVM1 (principal)

1. Cree o actualice el archivo tnsnames.ora:

   ```bash
   $ cd $ORACLE_HOME/network/admin
   $ vi tnsnames.ora

   cdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=cdb1)
      )
    )

   pdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=pdb1)
      )
    )
   ```

2. Cree las cuentas de usuario y de propietario de Golden Gate.

   > [!NOTE]
   > La cuenta del propietario debe tener el prefijo C##.
   >

    ```bash
    $ sqlplus / as sysdba
    SQL> CREATE USER C##GGADMIN identified by ggadmin;
    SQL> EXEC dbms_goldengate_auth.grant_admin_privilege('C##GGADMIN',container=>'ALL');
    SQL> GRANT DBA to C##GGADMIN container=all;
    SQL> connect C##GGADMIN/ggadmin
    SQL> ALTER SESSION SET CONTAINER=PDB1;
    SQL> EXIT;
    ```

3. Cree las cuentas de usuario de prueba de Golden Gate:

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ sqlplus system/OraPasswd1@pdb1
   SQL> CREATE USER test identified by test DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
   SQL> GRANT connect, resource, dba TO test;
   SQL> ALTER USER test QUOTA 100M on USERS;
   SQL> connect test/test@pdb1
   SQL> @demo_ora_create
   SQL> @demo_ora_insert
   SQL> EXIT;
   ```

4. Configure el archivo de parámetros de extracción.

   Inicie la interfaz de la línea de comandos de Golden Gate (ggsci):

   ```bash
   $ sudo su - oracle
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci
   GGSCI> DBLOGIN USERID test@pdb1, PASSWORD test
   Successfully logged into database  pdb1
   GGSCI>  ADD SCHEMATRANDATA pdb1.test
   2017-05-23 15:44:25  INFO    OGG-01788  SCHEMATRANDATA has been added on schema test.
   2017-05-23 15:44:25  INFO    OGG-01976  SCHEMATRANDATA for scheduling columns has been added on schema test.

   GGSCI> EDIT PARAMS EXTORA
   ```

5. Agregue lo siguiente al archivo de parámetros de extracción (mediante comandos de vi). Presione la tecla Esc, ':wq!' para guardar el archivo. 

   ```bash
   EXTRACT EXTORA
   USERID C##GGADMIN, PASSWORD ggadmin
   RMTHOST 10.0.0.5, MGRPORT 7809
   RMTTRAIL ./dirdat/rt  
   DDL INCLUDE MAPPED
   DDLOPTIONS REPORT 
   LOGALLSUPCOLS
   UPDATERECORDFORMAT COMPACT
   TABLE pdb1.test.TCUSTMER;
   TABLE pdb1.test.TCUSTORD;
   ```

6. Register extract - extracción integrada:

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci

   GGSCI> dblogin userid C##GGADMIN, password ggadmin
   Successfully logged into database CDB$ROOT.

   GGSCI> REGISTER EXTRACT EXTORA DATABASE CONTAINER(pdb1)

   2017-05-23 15:58:34  INFO    OGG-02003  Extract EXTORA successfully registered with database at SCN 1821260.

   GGSCI> exit
   ```

7. Configure los puntos de control de extracción e inicie la extracción en tiempo real:

   ```bash
   $ ./ggsci
   GGSCI>  ADD EXTRACT EXTORA, INTEGRATED TRANLOG, BEGIN NOW
   EXTRACT (Integrated) added.

   GGSCI>  ADD RMTTRAIL ./dirdat/rt, EXTRACT EXTORA, MEGABYTES 10
   RMTTRAIL added.

   GGSCI>  START EXTRACT EXTORA

   Sending START request to MANAGER ...
   EXTRACT EXTORA starting

   GGSCI > info all

   Program     Status      Group       Lag at Chkpt  Time Since Chkpt

   MANAGER     RUNNING
   EXTRACT     RUNNING     EXTORA      00:00:11      00:00:04
   ```

   En este paso, busque el SCN inicial, que se usará más adelante, en otra sección:

   ```bash
   $ sqlplus / as sysdba
   SQL> alter session set container = pdb1;
   SQL> SELECT current_scn from v$database;
   CURRENT_SCN
   -----------
      1857887
   SQL> EXIT;
   ```

   ```bash
   $ ./ggsci
   GGSCI> EDIT PARAMS INITEXT
   ```

   ```bash
   EXTRACT INITEXT
   USERID C##GGADMIN, PASSWORD ggadmin
   RMTHOST 10.0.0.5, MGRPORT 7809
   RMTTASK REPLICAT, GROUP INITREP
   TABLE pdb1.test.*, SQLPREDICATE 'AS OF SCN 1857887'; 
   ```

   ```bash
   GGSCI> ADD EXTRACT INITEXT, SOURCEISTABLE
   ```

### <a name="set-up-service-on-myvm2-replicate"></a>Configuración del servicio en myVM2 (réplica)


1. Cree o actualice el archivo tnsnames.ora:

   ```bash
   $ cd $ORACLE_HOME/network/admin
   $ vi tnsnames.ora

   cdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=cdb1)
      )
    )

   pdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=pdb1)
      )
    )
   ```

2. Cree una cuenta de replicación:

   ```bash
   $ sqlplus / as sysdba
   SQL> alter session set container = pdb1;
   SQL> create user repuser identified by rep_pass container=current;
   SQL> grant dba to repuser;
   SQL> exec dbms_goldengate_auth.grant_admin_privilege('REPUSER',container=>'PDB1');
   SQL> connect repuser/rep_pass@pdb1 
   SQL> EXIT;
   ```

3. Cree una cuenta de usuario de prueba de Golden Gate:

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ sqlplus system/OraPasswd1@pdb1
   SQL> CREATE USER test identified by test DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
   SQL> GRANT connect, resource, dba TO test;
   SQL> ALTER USER test QUOTA 100M on USERS;
   SQL> connect test/test@pdb1
   SQL> @demo_ora_create
   SQL> EXIT;
   ```

4. Archivo de parámetros REPLICAT para replicar los cambios: 

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci
   GGSCI> EDIT PARAMS REPORA  
   ```

   Contenido del archivo de parámetros REPORA:

   ```bash
   REPLICAT REPORA
   ASSUMETARGETDEFS
   DISCARDFILE ./dirrpt/repora.dsc, PURGE, MEGABYTES 100
   DDL INCLUDE MAPPED
   DDLOPTIONS REPORT
   DBOPTIONS INTEGRATEDPARAMS(parallelism 6)
   USERID repuser@pdb1, PASSWORD rep_pass
   MAP pdb1.test.*, TARGET pdb1.test.*;
   ```

5. Configure un punto de control de réplica:

   ```bash
   GGSCI> ADD REPLICAT REPORA, INTEGRATED, EXTTRAIL ./dirdat/rt
   GGSCI> EDIT PARAMS INITREP

   ```

   ```bash
   REPLICAT INITREP
   ASSUMETARGETDEFS
   DISCARDFILE ./dirrpt/tcustmer.dsc, APPEND
   USERID repuser@pdb1, PASSWORD rep_pass
   MAP pdb1.test.*, TARGET pdb1.test.*;   
   ```

   ```bash
   GGSCI> ADD REPLICAT INITREP, SPECIALRUN
   ```

### <a name="set-up-the-replication-myvm1-and-myvm2"></a>Configuración de la replicación (myVM1 y myVM2)

#### <a name="1-set-up-the-replication-on-myvm2-replicate"></a>1. Configure la replicación en myVM2 (réplica).

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  GGSCI> EDIT PARAMS MGR
  ```

Actualice el archivo con lo siguiente:

  ```bash
  PORT 7809
  ACCESSRULE, PROG *, IPADDR *, ALLOW
  ```

Después, reinicie el servicio de administrador:

  ```bash
  GGSCI> STOP MGR
  GGSCI> START MGR
  GGSCI> EXIT
  ```

#### <a name="2-set-up-the-replication-on-myvm1-primary"></a>2. Configure la replicación en myVM1 (principal).

Inicie la carga inicial y busque errores:

```bash
$ cd /u01/app/oracle/product/12.1.0/oggcore_1
$ ./ggsci
GGSCI> START EXTRACT INITEXT
GGSCI> VIEW REPORT INITEXT
```

#### <a name="3-set-up-the-replication-on-myvm2-replicate"></a>3. Configure la replicación en myVM2 (réplica).

Cambie el número de SCN con el número que obtuvo antes:

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  START REPLICAT REPORA, AFTERCSN 1857887
  ```

Ha empezado la replicación y puede probarla mediante la inserción de nuevos registros a las tablas de prueba.


### <a name="view-job-status-and-troubleshooting"></a>Visualización del estado del trabajo y solución de problemas

#### <a name="view-reports"></a>Ver informes
Para ver los informes en myVM1, ejecute los comandos siguientes:

  ```bash
  GGSCI> VIEW REPORT EXTORA 
  ```
 
Para ver los informes en myVM2, ejecute los comandos siguientes:

  ```bash
  GGSCI> VIEW REPORT REPORA
  ```

#### <a name="view-status-and-history"></a>Visualización del estado e historial
Para ver el estado e historial en myVM1, ejecute los comandos siguientes:

  ```bash
  GGSCI> dblogin userid c##ggadmin, password ggadmin 
  GGSCI> INFO EXTRACT EXTORA, DETAIL
  ```

Para ver el estado e historial en myVM2, ejecute los comandos siguientes:

  ```bash
  GGSCI> dblogin userid repuser@pdb1 password rep_pass 
  GGSCI> INFO REP REPORA, DETAIL
  ```
Este paso completa la instalación y configuración de Golden Gate en Oracle Linux.


## <a name="delete-the-virtual-machine"></a>Eliminación de la máquina virtual

Cuando ya no los necesite, se puede usar el comando siguiente para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

[Tutorial de creación de máquinas virtuales de alta disponibilidad](../../linux/create-cli-complete.md)

[Ejemplos de la CLI de implementación de máquinas virtuales](../../linux/cli-samples.md)
