---
title: Creación de una base de datos Oracle en una VM de Azure | Microsoft Docs
description: Ponga en funcionamiento rápidamente una base de datos de Oracle Database 12c en el entorno de Azure.
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
ms.openlocfilehash: ca40fcb6a2e483e656058835f187dc50bf7bc9ab
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074070"
---
# <a name="create-an-oracle-database-in-an-azure-vm"></a>Creación de una base de datos Oracle en una VM de Azure

En esta guía se describe el uso de la CLI de Azure para implementar una máquina virtual de Azure a partir de la [imagen de la galería de Marketplace de Oracle](https://azuremarketplace.microsoft.com/marketplace/apps/Oracle.OracleDatabase12102EnterpriseEdition?tab=Overview) con el fin de crear una base de datos de Oracle 12c. Una vez implementado el servidor, el usuario conectará a través de SSH con el fin de configurar la base de datos Oracle. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Si decide instalar y usar la CLI localmente, para esta guía de inicio rápido es preciso que ejecute la CLI de Azure versión 2.0.4 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-virtual-machine"></a>Crear máquina virtual

Para crear una máquina virtual, use el comando [az vm create](/cli/azure/vm). 

En el ejemplo siguiente se crea una máquina virtual denominada `myVM`. También se crean claves SSH si aún no existen en una ubicación de claves predeterminada. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`.  

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
    --size Standard_DS2_v2 \
    --admin-username azureuser \
    --generate-ssh-keys
```

Después de crear la máquina virtual, la CLI de Azure muestra información similar al ejemplo siguiente. Tome nota del valor de `publicIpAddress`, Use esta dirección para acceder a la máquina virtual.

```output
{
  "fqdns": "",
  "id": "/subscriptions/{snip}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "westus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="connect-to-the-vm"></a>Conexión a la máquina virtual

Para crear una sesión de SSH con la máquina virtual, use el siguiente comando. Reemplace la dirección IP por el valor `publicIpAddress` para la máquina virtual.

```bash
ssh azureuser@<publicIpAddress>
```

## <a name="create-the-database"></a>Creación de la base de datos

El software de Oracle ya está instalado en la imagen de Marketplace. Cree una base de datos de ejemplo de la manera siguiente: 

1.  Cambie al superusuario *oracle* e inicialice el agente de escucha para el registro:

    ```bash
    $ sudo -su oracle
    $ lsnrctl start
    ```

    La salida será similar a la siguiente:

    ```output
    Copyright (c) 1991, 2014, Oracle.  All rights reserved.

    Starting /u01/app/oracle/product/12.1.0/dbhome_1/bin/tnslsnr: please wait...

    TNSLSNR for Linux: Version 12.1.0.2.0 - Production
    Log messages written to /u01/app/oracle/diag/tnslsnr/myVM/listener/alert/log.xml
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=myVM.twltkue3xvsujaz1bvlrhfuiwf.dx.internal.cloudapp.net)(PORT=1521)))

    Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
    STATUS of the LISTENER
    ------------------------
    Alias                     LISTENER
    Version                   TNSLSNR for Linux: Version 12.1.0.2.0 - Production
    Start Date                23-MAR-2017 15:32:08
    Uptime                    0 days 0 hr. 0 min. 0 sec
    Trace Level               off
    Security                  ON: Local OS Authentication
    SNMP                      OFF
    Listener Log File         /u01/app/oracle/diag/tnslsnr/myVM/listener/alert/log.xml
    Listening Endpoints Summary...
    (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=myVM.twltkue3xvsujaz1bvlrhfuiwf.dx.internal.cloudapp.net)(PORT=1521)))
    The listener supports no services
    The command completed successfully
    ```

2.  Cree la base de datos:

    ```bash
    dbca -silent \
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

    La operación de creación de la base de datos tarda unos minutos.

3. Establecimiento de las variables de Oracle

Antes de conectarse, tiene que definir dos variables de entorno: *ORACLE_HOME* y *ORACLE_SID*.

```bash
ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
ORACLE_SID=cdb1; export ORACLE_SID
```

También puede agregar ORACLE_HOME y ORACLE_SID al archivo .bashrc. De este modo, las variables de entorno se guardarían para futuros inicios de sesión. Confirme que las instrucciones siguientes se han agregado al archivo `~/.bashrc` mediante el editor que elija.

```bash
# Add ORACLE_HOME. 
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1 
# Add ORACLE_SID. 
export ORACLE_SID=cdb1 
```

## <a name="oracle-em-express-connectivity"></a>Conectividad con Oracle EM Express

Para disponer de una herramienta de administración de GUI con la que pueda explorar la base de datos, configure Oracle EM Express. Para conectarse a Oracle EM Express, primero debe configurar el puerto en Oracle. 

1. Conéctese a la base de datos mediante sqlplus:

    ```bash
    sqlplus / as sysdba
    ```

2. Una vez conectado, establezca el puerto 5502 para EM Express.

    ```bash
    exec DBMS_XDB_CONFIG.SETHTTPSPORT(5502);
    ```

3. Abra el contenedor PDB1 si no está ya abierto, pero antes compruebe el estado:

    ```bash
    select con_id, name, open_mode from v$pdbs;
    ```

    La salida será similar a la siguiente:

    ```output
      CON_ID NAME                           OPEN_MODE 
      ----------- ------------------------- ---------- 
      2           PDB$SEED                  READ ONLY 
      3           PDB1                      MOUNT
    ```

4. Si el valor de OPEN_MODE para `PDB1` no es READ WRITE, ejecute los comandos siguientes para abrir PDB1:

   ```bash
    alter session set container=pdb1;
    alter database open;
   ```

Debe escribir `quit` para finalizar la sesión de sqlplus y `exit` para cerrar la sesión del usuario de Oracle.

## <a name="automate-database-startup-and-shutdown"></a>Automatizar el arranque y el apagado de la base de datos

De forma predeterminada, la base de datos Oracle no se inicia automáticamente al reiniciar la máquina virtual. Para configurar la base de datos Oracle para iniciarse automáticamente, inicie primero una sesión como raíz. Después, cree y actualice algunos archivos del sistema.

1. Inicio de sesión como raíz

    ```bash
    sudo su -
    ```

2.  Con su editor favorito, edite el archivo `/etc/oratab` y cambie el valor predeterminado `N` a `Y`:

    ```bash
    cdb1:/u01/app/oracle/product/12.1.0/dbhome_1:Y
    ```

3.  Cree un archivo denominado `/etc/init.d/dbora` y pegue el siguiente contenido:

    ```bash
    #!/bin/sh
    # chkconfig: 345 99 10
    # Description: Oracle auto start-stop script.
    #
    # Set ORA_HOME to be equivalent to $ORACLE_HOME.
    ORA_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
    ORA_OWNER=oracle

    case "$1" in
    'start')
        # Start the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        # Remove "&" if you don't want startup as a background process.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbstart $ORA_HOME" &
        touch /var/lock/subsys/dbora
        ;;

    'stop')
        # Stop the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbshut $ORA_HOME" &
        rm -f /var/lock/subsys/dbora
        ;;
    esac
    ```

4.  Cambie los permisos de los archivos con *chmod* como se indica a continuación:

    ```bash
    chgrp dba /etc/init.d/dbora
    chmod 750 /etc/init.d/dbora
    ```

5.  Cree vínculos simbólicos para el inicio y el apagado de la manera siguiente:

    ```bash
    ln -s /etc/init.d/dbora /etc/rc.d/rc0.d/K01dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc3.d/S99dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc5.d/S99dbora
    ```

6.  Para probar los cambios, reinicie la máquina virtual:

    ```bash
    reboot
    ```

## <a name="open-ports-for-connectivity"></a>Abrir puertos para la conectividad

La tarea final consiste en configurar algunos puntos de conexión externos. Para configurar el grupo de seguridad de red de Azure que protege la máquina virtual, cierre primero su sesión de SSH en la máquina virtual (si se le expulsó de SSH al reiniciar en el paso anterior). 

1.  Para abrir el punto de conexión que usa para acceder a la base de datos Oracle de forma remota, cree una regla de grupo de seguridad de red con [az network nsg rule create](/cli/azure/network/nsg/rule) de la manera siguiente: 

    ```azurecli-interactive
    az network nsg rule create \
        --resource-group myResourceGroup\
        --nsg-name myVmNSG \
        --name allow-oracle \
        --protocol tcp \
        --priority 1001 \
        --destination-port-range 1521
    ```

2.  Para abrir el punto de conexión que usa para acceder a Oracle EM Express de forma remota, cree una regla de grupo de seguridad de red con [az network nsg rule create](/cli/azure/network/nsg/rule) de la manera siguiente:

    ```azurecli-interactive
    az network nsg rule create \
        --resource-group myResourceGroup \
        --nsg-name myVmNSG \
        --name allow-oracle-EM \
        --protocol tcp \
        --priority 1002 \
        --destination-port-range 5502
    ```

3. Si es necesario, obtenga de nuevo la dirección IP pública de la máquina virtual con [az network public-ip show](/cli/azure/network/public-ip) de la manera siguiente:

    ```azurecli-interactive
    az network public-ip show \
        --resource-group myResourceGroup \
        --name myVMPublicIP \
        --query [ipAddress] \
        --output tsv
    ```

4.  Conecte EM Express desde el explorador. Asegúrese de que el explorador sea compatible con Express EM (es necesario que Flash esté instalado): 

    ```https
    https://<VM ip address or hostname>:5502/em
    ```

Puede iniciar sesión mediante la cuenta **SYS** y activar la casilla **as sysdba**. Use la contraseña **OraPasswd1** que estableció durante la instalación. 

![Captura de pantalla de la página de inicio de sesión de Oracle OEM Express](./media/oracle-quick-start/oracle_oem_express_login.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de explorar la primera base de datos Oracle en Azure y la VM ya no se necesite, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, la VM y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Conozca otras [soluciones de Oracle en Azure](./oracle-overview.md). 

Pruebe el tutorial sobre la [instalación y configuración de Oracle Automated Storage Management](configure-oracle-asm.md).
