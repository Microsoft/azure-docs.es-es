---
title: Instalación de TmaxSoft OpenFrame en Azure Virtual Machines
description: Rehospede las cargas de trabajo del sistema central IBM z/OS con el entorno TmaxSoft OpenFrame en Azure Virtual Machines (VM).
services: virtual-machines-linux
documentationcenter: ''
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: virtual-machines-linux
ms.openlocfilehash: 41b31d5c4a01183d4620dcd6ec6f4729f078d382
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082399"
---
# <a name="install-tmaxsoft-openframe-on-azure"></a>Instalación TmaxSoft OpenFrame en Azure

Obtenga información sobre cómo configurar un entorno OpenFrame en Azure adecuado para cargas de trabajo de desarrollo, demostraciones, pruebas o producción. Este tutorial le guiará a través de estos pasos.

OpenFrame incluye varios componentes que crean el entorno de emulación del sistema central en Azure. Por ejemplo, los servicios en línea de OpenFrame reemplazan el middleware del sistema central, como el sistema de control de información de clientes (CICS) de IBM, mientras que OpenFrame Batch, con su componente TJES, reemplaza el subsistema de entrada de trabajo (JES) del sistema central de IBM.

OpenFrame funciona con cualquier base de datos relacional, incluidos Oracle Database, Microsoft SQL Server, IBM Db2 y MySQL. Para esta instalación de OpenFrame se usa la base de datos relacional TmaxSoft Tibero. Tanto OpenFrame como Tibero funcionan con sistemas operativos Linux. En este tutorial se instala CentOS 7.3, aunque puede usar otras distribuciones de Linux compatibles. El servidor de aplicaciones OpenFrame y la base de datos Tibero se instalan en una máquina virtual (VM).

El tutorial le guiará en la instalación de los componentes del conjunto OpenFrame. Algunos deben instalarse por separado.

Componentes principales de OpenFrame:

- Paquetes de instalación obligatorios.
- Base de datos Tibero.
- Las aplicaciones de OpenFrame utilizan la conectividad abierta de bases de datos (ODBC) para comunicarse con la base de datos Tibero.
- OpenFrame Base, el middleware que administra todo el sistema.
- OpenFrame Batch, la solución que reemplaza los sistemas de lotes del sistema central.
- TACF, un módulo de servicio que controla el acceso del usuario a los sistemas y recursos.
- ProSort, una herramienta de ordenación para las transacciones por lotes.
- OFCOBOL, un compilador que interpreta los programas de COBOL del sistema central.
- OFASM, un compilador que interpreta los programas del ensamblador del sistema central.
- Servidor OpenFrame del tipo C (OSC), la solución que reemplaza el middleware del sistema central y el CICS de IBM.
- Java Enterprise User Solution (JEUS ), un servidor de aplicaciones web certificado para Java Enterprise Edition 6.
- OFGW, el componente de puerta de enlace de OpenFrame que proporciona un cliente de escucha 3270.
- OFManager, una solución que proporciona funciones de administración y operación de OpenFrame en el entorno web.

Otros componentes de OpenFrame obligatorios:

- OSI, la solución que reemplaza el middleware del sistema central y el DC de IMS.
- TJES, la solución que proporciona el entorno JES del sistema central.
- OFTSAM, la solución que permite el uso de archivos (V)SAM en el sistema abierto.
- OFHiDB, la solución que reemplaza el DB de IMS del sistema central.
- OFCOBOL, un compilador que interpreta los programas PL/I del sistema central.
- PROTRIEVE, una solución que ejecuta el lenguaje CA-Easytrieve del sistema central.
- OFMiner, una solución que analiza los recursos de los sistemas centrales y después los migra a Azure.

## <a name="architecture"></a>Architecture

La ilustración siguiente proporciona información general sobre los componentes de la arquitectura OpenFrame 7.0 en este tutorial:

![Componentes de OpenFrame](media/openframe-02.png)

## <a name="azure-system-requirements"></a>Requisitos del sistema Azure

En la tabla siguiente se enumeran los requisitos para la instalación en Azure.
<!-- markdownlint-disable MD033 -->

<table>
<thead>
    <tr><th>Requisito</th><th>Descripción</th></tr>
</thead>
<tbody>
<tr><td>Distribuciones de Linux admitidas en Azure
</td>
<td>
Linux x86 2.6 (32 bits, 64 bits)<br/>
Red Hat 7.x<br/>
CentOS 7.x<br/>
</td>
</tr>
<tr><td>Hardware
</td>
<td>Núcleos: 2 (como mínimo)<br/>
Memoria: 4 GB (como mínimo)<br/>
Espacio de intercambio: 1 GB (como mínimo)<br/>
Disco duro: 100 GB (como mínimo)<br/>
</td>
</tr>
<tr><td>Software opcional para los usuarios de Windows
</td>
<td>PuTTY: se usa en esta guía para configurar las características de la máquina virtual<br/>
WinSCP: un cliente SFTP popular y un cliente FTP que puede usar<br/>
Eclipse para Windows: una plataforma de desarrollo compatible con TmaxSoft<br/>
(de momento no se admite Microsoft Visual Studio)
</td>
</tr>
</tbody>
</table>

<!-- markdownlint-enable MD033 -->

## <a name="prerequisites"></a>Prerrequisitos

Tendrá que dedicar unos días para ensamblar todo el software necesario y completar todos los procesos manuales.

Antes de empezar, haga lo siguiente:

- Obtenga el soporte de instalación de OpenFrame de TmaxSoft. Si ya es cliente de TmaxSoft, póngase en contacto con su representante de TmaxSoft para obtener una copia con licencia. En caso contrario, solicite una versión de prueba en [TmaxSoft](https://www.tmaxsoft.com/contact/).

- Para solicitar la documentación de OpenFrame, envíe un correo a <support@tmaxsoft.com>.

- Suscríbase a Azure, si todavía no lo está. También puede crear una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.

- Opcional. Configure un túnel VPN de sitio a sitio o un jumpbox que restrinja el acceso a la máquina virtual de Azure a los usuarios permitidos en su organización. Este paso no es obligatorio, pero sí recomendable.

## <a name="set-up-a-vm-on-azure-for-openframe-and-tibero"></a>Configurar una máquina virtual en Azure para OpenFrame y Tibero

Puede configurar el entorno OpenFrame con diversos patrones de implementación. En el procedimiento siguiente se muestra cómo implementar el servidor de aplicaciones OpenFrame y la base de datos Tibero en una máquina virtual. En entornos más grandes y para cargas de trabajo considerables, se recomienda implementar la base de datos por separado en su propia máquina virtual para mejorar el rendimiento.

**Para crear una máquina virtual**

1. Vaya a Azure Portal (<https://portal.azure.com>) e inicie sesión en su cuenta.

2. Haga clic en **Máquinas virtuales**.

    ![Lista de recursos en Azure Portal](media/vm-01.png)

3. Haga clic en **Agregar**.

    ![Opción para agregar en Azure Portal](media/vm-02.png)

4. A la derecha de **Sistemas operativos**, haga clic en **Más**.

     ![Opción Más en Azure Portal](media/vm-03.png)

5. Haga clic en **Basado en CentOS 7.3** para seguir este tutorial exactamente o elija otra distribución de Linux admitida.

     ![Opciones de sistema operativo en Azure Portal](media/vm-04.png)

6. En la configuración de **Basics**, rellene los campos **Nombre**, **Nombre de usuario**, **Tipo de autenticación**, **Suscripción** (pago por uso es el tipo de pago de AWS) y **Grupo de recursos** (use uno existente o cree un grupo de TmaxSoft).

7. Cuando haya terminado (incluyendo el par de claves pública y privada para **Tipo de autenticación**), haga clic en **Enviar**.

> [!NOTE]
> Si utiliza una clave pública SSH para **Tipo de autenticación**, consulte los pasos descritos en la sección siguiente para generar el par de claves pública y privada y luego reanude los pasos siguientes.

### <a name="generate-a-publicprivate-key-pair"></a>Generación de un par de claves públicas y privadas

Si utiliza un sistema operativo de Windows, necesitará PuTTYgen para generar un par de claves pública y privada.

La clave pública se puede compartir libremente, pero la clave privada debe mantenerse completamente en secreto y nunca debe compartirse con nadie. Después de generar las claves, debe pegar la **clave pública SSH** en la configuración, de hecho, debe cargarla en la máquina virtual de Linux. Esta se almacenará dentro de authorized\_keys en el directorio \~/.ssh del directorio principal de la cuenta de usuario. Después, la máquina virtual de Linux podrá reconocer y validar la conexión una vez que haya proporcionado la **clave privada SSH** asociada en el cliente SSH (en nuestro caso, PuTTY).

Al proporcionar acceso a la máquina virtual a nuevos individuos: 

- Cada nuevo individuo generará sus propias claves pública y privada con PuTTYgen.
- Los usuarios almacenarán sus propias claves privadas de forma independiente y enviarán la información de la clave pública al administrador de la máquina virtual.
- El administrador pegará el contenido de la clave pública en el archivo \~/.ssh/authorized\_keys.
- El nuevo usuario se conectará mediante PuTTY.

**Para generar un par de claves pública y privada**

1.  Descargue PuTTYgen desde <https://www.putty.org/> e instálelo usando todos los valores predeterminados.

2.  Para abrir PuTTYgen, busque el directorio de instalación de PuTTY en C:\\Archivos de programa\\PuTTY.

    ![Interfaz de PuTTY](media/puttygen-01.png)

3.  Haga clic en **Generar**.

    ![Cuadro de diálogo del generador de claves PuTTY](media/puttygen-02.png)

4.  Después de la generación, guarde la clave pública y la privada. Pegue el contenido de la clave pública en la sección **SSH public key** (Clave pública SSH) del panel **Create virtual machine \> Basics** (Crear máquina virtual > Aspectos básicos), como se muestra en los pasos 6 y 7 de la sección anterior.

    ![Cuadro de diálogo del generador de claves PuTTY](media/puttygen-03.png)

### <a name="configure-vm-features"></a>Configuración de características de la máquina virtual

1. En Azure Portal, en la hoja **Elegir un tamaño**, elija la configuración de hardware de la máquina Linux que desee. Los requisitos *mínimos* para instalar Tibero y OpenFrame son 2 CPU y 4 GB de RAM, tal como se muestra en esta instalación de ejemplo:

    ![Creación de una máquina virtual: aspectos básicos](media/create-vm-01.png)

2. Haga clic en **3 Configuración** y use la configuración predeterminada para configurar características opcionales.
3. Revise los detalles de pago.

    ![Creación de una máquina virtual: compra](media/create-vm-02.png)

4. Envíe lo que ha seleccionado. Azure comienza a implementar la máquina virtual. Este proceso acostumbra a tardar unos minutos.

5. Una vez implementada la máquina virtual, se muestra su panel, en el que aparecen todas las opciones de configuración seleccionadas durante la configuración. Anote la **Dirección IP pública**.

    ![tMax en el panel de Azure](media/create-vm-03.png)

6. Abra PuTTY.

7. En **Host name** (Nombre de host), escriba su nombre de usuario y la dirección IP pública que ha copiado. Por ejemplo, **nombreusuario\@ippública**.

    ![Cuadro de diálogo de configuración de PuTTY](media/putty-01.png)

8. En el cuadro **Category** (Categoría), haga clic en **Connection\> SSH \> Auth** (Conexión > SSH > Autenticación). Proporcione la ruta de acceso a su archivo de **clave privada**.

    ![Cuadro de diálogo de configuración de PuTTY](media/putty-02.png)

9. Haga clic en **Open** (Abrir) para abrir la ventana de PuTTY. Si ha realizado los pasos correctamente, tendrá conexión a su nueva máquina virtual de CentOS en Azure.

10. Para iniciar sesión como usuario raíz, escriba **sudo bash**.

    ![Inicio de sesión como usuario raíz en la ventana de comandos](media/putty-03.png)

## <a name="set-up-the-environment-and-packages"></a>Configuración del entorno y los paquetes

Cuando haya creado la máquina virtual y haya iniciado sesión, deberá seguir algunos pasos de configuración e instalar los paquetes de preinstalación necesarios.

1. Asigne el nombre **ofdemo** a la dirección IP local usando vi para modificar el archivo de hosts (`vi /etc/hosts`). Suponiendo que nuestra IP es 192.168.96.148 ofdemo, antes del cambio se vería así:

    ```vi
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
    ::1              localhost localhost.localdomain localhost6 localhost6.localdomain 
    <IP Address>    <your hostname>
    ```

     Y así después del cambio:

    ```vi
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
    ::1              localhost localhost.localdomain localhost6 localhost6.localdomain 
    192.168.96.148   ofdemo
    ```

2. Cree grupos y usuarios:

    ```vi
    [root@ofdemo ~]# adduser -d /home/oframe7 oframe7 
    [root@ofdemo ~]# passwd oframe7
    ```

3. Cambie la contraseña del usuario oframe7:

    ```vi
    New password: 
    Retype new password: 
    passwd: all authentication tokens updated successfully.
    ```

4. Actualice los parámetros de kernel en /etc/sysctl.conf:

    ```vi
    [root@ofdemo ~]# vi /etc/sysctl.conf
    kernel.shmall = 7294967296 
    kernel.sem = 10000 32000 10000 10000
    ```

5. Actualice los parámetros de kernel dinámicamente sin reiniciar:

    ```vi
    [root@ofdemo ~]# /sbin/sysctl -p
    ```

6. Consiga los paquetes obligatorios: (asegúrese de que el servidor está conectado a Internet, descargue los paquetes siguientes e instálelos):

     - dos2unix
     - glibc
     - glibc.i686 glibc.x86\_64
     - libaio
     - ncurses

          > [!NOTE]
          > Después de instalar el paquete ncurses, cree los vínculos simbólicos siguientes:
         ```
         ln -s /usr/lib64/libncurses.so.5.9 /usr/lib/libtermcap.so
         ln -s /usr/lib64/libncurses.so.5.9 /usr/lib/libtermcap.so.2
         ```

     - gcc
     - gcc-c++
     - libaio-devel.x86\_64
     - strace
     - ltrace
     - gdb

7. En el caso de una instalación de RPM de Java, haga lo siguiente:

```
root@ofdemo ~]# rpm -ivh jdk-7u79-linux-x64.rpm
[root@ofdemo ~]# vi .bash_profile

# JAVA ENV
export JAVA_HOME=/usr/java/jdk1.7.0_79/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar

[root@ofdemo ~]# source /etc/profile
[root@ofdemo ~]# java –version

java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

[root@ofdemo ~]# echo $JAVA_HOME /usr/java/jdk1.7.0_79/
```

## <a name="install-the-tibero-database"></a>Instalar la base de datos Tibero

Tibero proporciona la diferentes funciones clave en el entorno OpenFrame en Azure:

- Tibero sirve como almacén de datos interno de OpenFrame para distintas funciones del sistema.
- Los archivos VSAM, incluidos KSDS RRDS y ESDS, usan la base de datos Tibero internamente para almacenar los datos.
- El repositorio de datos TACF se almacena en Tibero.
- La información del catálogo OpenFrame se almacena en Tibero.
- La base de datos Tibero puede usarse como sustituta de la Db2 de IBM para almacenar datos de la aplicación.

**Para instalar Tibero**

1. Compruebe que el archivo de instalador binario Tibero está presente y revise el número de versión.
2. Copie el software Tibero en la cuenta de usuario de Tibero (oframe). Por ejemplo:

    ```
    [oframe7@ofdemo ~]$ tar -xzvf tibero6-bin-6_rel_FS04-linux64-121793-opt-tested.tar.gz 
    [oframe7@ofdemo ~]$ mv license.xml /opt/tmaxdb/tibero6/license/
    ```

3. Abra .bash\_profile en vi (`vi .bash_profile`) y pegue lo siguiente:

    ```
    # Tibero6 ENV
    export TB_HOME=/opt/tmaxdb/tibero6 
    export TB_SID=TVSAM export TB_PROF_DIR=$TB_HOME/bin/prof 
    export LD_LIBRARY_PATH=$TB_HOME/lib:$TB_HOME/client/lib:$LD_LIBRARY_PATH 
    export PATH=$TB_HOME/bin:$TB_HOME/client/bin:$PATH
    ```

4. Para ejecutar el perfil de bash, escriba lo siguiente en el símbolo del sistema:

    ```
    source .bash_profile
    ```

5. Genere el archivo TIP (un archivo de configuración de Tibero) y ábralo en vi. Por ejemplo:

    ```
    [oframe7@ofdemo ~]$ sh $TB_HOME/config/gen_tip.sh
    [oframe7@ofdemo ~]$ vi $TB_HOME/config/$TB_SID.tip
    ```

6. Modifique \$TB\_HOME/client/config/tbdsn.tbr e introduzca 127.0.0.1 en vez de localhost, tal como se muestra a continuación:

    ```
    TVSAM=( 
    (INSTANCE=(HOST=127.0.0.1)
        (PT=8629)
    (DB_NAME=TVSAM)
          )
     )
    ```

7. Cree la base de datos. Se mostrará lo siguiente:

    ```
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof.
    Listener port = 8629
    Tibero 6
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NOMOUNT mode).
     /--------------------- newmount sql ------------------------/
    create database character set MSWIN949 national character set UTF16;
    /-----------------------------------------------------------/
    Database created.
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof.
    Listener port = 8629
    Tibero 6
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NORMAL mode).
    /opt/tmaxdb/tibero6/bin/tbsvr
    ………………………..
    Creating agent table...
    Done.
    For details, check /opt/tmaxdb/tibero6/instance/TVSAM/log/system_init.log.
    ************************************************** 
    * Tibero Database TVSAM is created successfully on Fri Aug 12 19:10:43 UTC 2016.
    *     Tibero home directory ($TB_HOME) =
    *         /opt/tmaxdb/tibero6
    *     Tibero service ID ($TB_SID) = TVSAM
    *     Tibero binary path =
    *         /opt/tmaxdb/tibero6/bin:/opt/tmaxdb/tibero6/client/bin
    *     Initialization parameter file =
    *         /opt/tmaxdb/tibero6/config/TVSAM.tip
    * 
    * Make sure that you always set up environment variables $TB_HOME and
    * $TB_SID properly before you run Tibero.
     ******************************************************************************
    ```

8. Para reciclar Tibero, primero apáguelo utilizando el comando `tbdown`. Por ejemplo:

    ```
    [oframe7@ofdemo ~]$$ tbdown 
    Tibero instance terminated (NORMAL mode).
    ```

9. Ahora arranque Tibero usando `tbboot`. Por ejemplo:

    ```
    [oframe7@ofdemo ~]$ tbboot
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof. Listener port = 8629

    Tibero 6  
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NORMAL mode).
    ```

10. Para crear un espacio de tabla, acceda a la base de datos mediante el usuario SYS (sys/tmax) y después cree el espacio de tabla necesario para el volumen predeterminado y TACF:

    ```
    [oframe7@ofdemo ~]$ tbsql tibero/tmax
    tbSQL 6  
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Connected to Tibero.
    ```

11. Ahora escriba los comandos SQL siguientes:

    ```
    SQL> create tablespace "DEFVOL" datafile 'DEFVOL.dbf' size 500M autoextend on; create tablespace "TACF00" datafile 'TACF00.dbf' size 500M autoextend on; create tablespace "OFM_REPOSITORY" datafile 'ofm_repository.dbf' size 300M autoextend on;
    SQL> Tablespace 'DEFVOL' created.
    SQL> Tablespace 'TACF00' created.
    SQL> Tablespace ' OFM_REPOSITORY ' created.
    SQL> SQL> Disconnected.
    ```

12. Arranque Tibero y compruebe que se ejecutan los procesos de Tibero:

    ```
    [oframe7@ofdemo ~]$ tbboot 
    ps -ef | egrep tbsvr
    ```

Salida:

![Salida de Tibero](media/tibero-01.png)

## <a name="install-odbc"></a>Instalación de ODBC

Las aplicaciones de OpenFrame se comunican con la base de datos Tibero con la API de ODBC proporcionada por el proyecto de código abierto unixODBC.

Para instalar ODBC:

1. Compruebe que el archivo de instalación unixODBC 2.3.4.tar.gz está presente o use el comando `wget unixODBC-2.3.4.tar.gz`. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ wget ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.4.tar.gz
     ```

2. Descomprima el binario. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ tar -zxvf unixODBC-2.3.4.tar.gz
     ```

3. Vaya al directorio unixODBC-2.3.4 y genere el archivo Make utilizando la información de comprobación de la máquina. Por ejemplo:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ ./configure --prefix=/opt/tmaxapp/unixODBC/ --sysconfdir=/opt/tmaxapp/unixODBC/etc
     ```

     De forma predeterminada, unixODBC se instala en /usr /local, por lo que `--prefix` pasa un valor para cambiar la ubicación. De forma similar, los archivos de configuración se instalan en /etc de forma predeterminada, por lo que `--sysconfdir` pasa el valor de la ubicación deseada.

4. Ejecute el archivo Make: `[oframe7@ofdemo unixODBC-2.3.4]$ make`

5. Copie el archivo ejecutable en el directorio del programa después de la compilación. Por ejemplo:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ make install
     ```

6. Utilice vi para editar el perfil de bash (`vi ~/.bash_profile`) y agregue lo siguiente:

     ```
     # UNIX ODBC ENV 
     export ODBC_HOME=$HOME/unixODBC 
     export PATH=$ODBC_HOME/bin:$PATH 
     export LD_LIBRARY_PATH=$ODBC_HOME/lib:$LD_LIBRARY_PATH 
     export ODBCINI=$HOME/unixODBC/etc/odbc.ini 
     export ODBCSYSINI=$HOME
     ```

7. Aplique la conectividad abierta de bases de datos. Edite los archivos siguientes según corresponda. Por ejemplo:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ source ~/.bash_profile

     [oframe7@ofdemo ~]$ cd

     [oframe7@ofdemo ~]$ odbcinst -j unixODBC 2.3.4
     DRIVERS............: /home/oframe7/odbcinst.ini
     SYSTEM DATA SOURCES: /home/oframe7/odbc.ini
     FILE DATA SOURCES..: /home/oframe7/ODBCDataSources
     USER DATA SOURCES..: /home/oframe7/unixODBC/etc/odbc.ini
     SQLULEN Size.......: 8
     SQLLEN Size........: 8
     SQLSETPOSIROW Size.: 8

     [oframe7@ofdemo ~]$ vi odbcinst.ini

     [Tibero]
     Description = Tibero ODBC driver for Tibero6
     Driver = /opt/tmaxdb/tibero6/client/lib/libtbodbc.so
     Setup = 
     FileUsage = 
     CPTimeout = 
     CPReuse = 
     Driver Logging = 7

     [ODBC]
     Trace = NO 
     TraceFile = /home/oframe7/odbc.log 
     ForceTrace = Yes 
     Pooling = No 
     DEBUG = 1

     [oframe7@ofdemo ~]$ vi odbc.ini

     [TVSAM]
     Description = Tibero ODBC driver for Tibero6 
     Driver = Tibero 
     DSN = TVSAM 
     SID = TVSAM 
     User = tibero 
     password = tmax
     ```

8. Cree un vínculo simbólico y valide la conexión de base de datos Tibero:

     ```
     [oframe7@ofdemo ~]$ ln $ODBC_HOME/lib/libodbc.so $ODBC_HOME/lib/libodbc.so.1 [oframe7@ofdemo ~]$ ln $ODBC_HOME/lib/libodbcinst.so 
     $ODBC_HOME/lib/libodbcinst.so.1

     [oframe7@ofdemo lib]$ isql TVSAM tibero tmax
     ```

Se muestra el siguiente resultado:

![Salida de ODBC que se muestra conectada a SQL](media/odbc-01.png)

## <a name="install-openframe-base"></a>Instalación de OpenFrame Base

El servidor de la aplicación Base se instala antes que los servicios individuales que OpenFrame usa para administrar el sistema en Azure, incluidos los procesos del servidor de control de transacciones.

**Para instalar OpenFrame Base**

1. Asegúrese de que la instalación de Tibero se ha realizado correctamente y, después, compruebe que el archivo de instalación OpenFrame\_Base7\_0\_Linux\_x86\_64.bin y el archivo de configuración base.properties están presentes.

2. Actualice el perfil de bash con la siguiente información específica de Tibero:

     ```bash
     alias ofhome='cd $OPENFRAME_HOME'
     alias ulog='cd $OPENFRAME_HOME/log/tmax/ulog'
     alias sysjcl='cd $OPENFRAME_HOME/volume_default/SYS1.JCLLIB'
     alias sysload='cd $OPENFRAME_HOME/volume_default/SYS1.LOADLIB'
     alias sysproc='cd $OPENFRAME_HOME/volume_default/SYS1.PROCLIB'
     alias oscsrc='cd $OPENFRAME_HOME/osc/oivp'
     alias osisrc='cd $OPENFRAME_HOME/osi/oivp'
     alias defvol='cd $OPENFRAME_HOME/volume_default'
     ```

3. Ejecute el perfil de bash:`[oframe7@ofdemo ~]$ . .bash_profile`
4. Asegúrese de que los procesos de Tibero se están ejecutando. Por ejemplo:

     ```linux
     [oframe7@ofdemo ~]$ ps -ef|grep tbsvr
     ```

    ![Base](media/base-01.png)

     > [!IMPORTANT]
     > Inicie Tibero antes de la instalación.

5. Genere licencias en [technet.tmaxsoft.com](https://technet.tmaxsoft.com/en/front/main/main.do) y coloque las licencias de OpenFrame Base, Batch, TACF y OSC en la carpeta correspondiente:

     ```
     [oframe7@ofdemo ~]$ cp license.dat /opt/tmaxapp/OpenFrame/core/license/
     [oframe7@ofdemo ~]$ cp lictjes.dat lictacf.dat licosc.dat $OPENFRAME_HOME/license/
     ```

6. Descargue los archivos binarios y base.properties de OpenFrame Base:

     ```
     [oframe7@ofdemo ~]$ vi base.properties
     OPENFRAME_HOME= <appropriate location for installation> ex. /opt/tmaxapp/OpenFrame TP_HOST_NAME=<your IP Hostname> ex. ofdemo 
     TP_HOST_IP=<your IP Address> ex. 192.168.96.148 
     TP_SHMKEY=63481 
     TP_TPORTNO=6623 
     TP_UNBLOCK_PORT=6291 
     TP_NODE_NAME=NODE1 
     TP_NODE_LIST=NODE1 
     MASCAT_NAME=SYS1.MASTER.ICFCAT 
     MASCAT_CREATE=YES 
     DEFAULT_VOLSER=DEFVOL 
     VOLADD_DEFINE=YES TSAM_USERNAME=tibero 
     TSAM_PASSWORD=tmax 
     TSAM_DATABASE=oframe 
     DATASET_SHMKEY=63211 
     DSLOCK_DATA=SYS1.DSLOCK.DATA 
     DSLOCK_LOG=SYS1.DSLOCK.LOG 
     DSLOCK_SEQ=dslock_seq.dat 
     DSLOCK_CREATE=YES 
     OPENFRAME_LICENSE_PATH=/opt/tmaxapp/license/OPENFRAME TMAX_LICENSE_PATH=/opt/tmaxapp/license/TMAX
     ```

7. Ejecute el programa de instalación mediante el archivo base.properties. Por ejemplo:

    ```
    [oframe7@ofdemo ~]$ chmod a+x OpenFrame_Base7_0_Linux_x86_64.bin 
    [oframe7@ofdemo ~]$ ./OpenFrame_Base7_0_Linux_x86_64.bin -f base.properties
    ```

    Cuando haya finalizado, se mostrara el mensaje que indica que la instalación ha finalizado.

8. Compruebe la estructura del directorio de OpenFrame Base con el comando `ls -ltr`. Por ejemplo:

     ```
     [oframe7@ofdemo OpenFrame]$ ls -ltr
     total 44

     drwxrwxr-x. 4 oframe7 oframe7 61 Nov 30 16:57 UninstallerData 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 bin 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 cpm drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 data 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 include 
     drwxrwxr-x. 2 oframe7 oframe7 8192 Nov 30 16:57 lib 
     drwxrwxr-x. 6 oframe7 oframe7 48 Nov 30 16:57 log 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 profile 
     drwxrwxr-x. 7 oframe7 oframe7 62 Nov 30 16:57 sample 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 schema 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 temp 
     drwxrwxr-x. 3 oframe7 oframe7 16 Nov 30 16:57 shared 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 license 
     drwxrwxr-x. 23 oframe7 oframe7 4096 Nov 30 16:58 core 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 config 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 scripts 
     drwxrwxr-x. 2 oframe7 oframe7 25 Nov 30 16:58 volume_default
     ```

9. Inicie OpenFrame Base:

     ```
     [oframe7@ofdemo ~]$ cp /usr/lib/libtermcap.so.2 $TMAXDIR/lib
     Startup Tmax Server
     [oframe7@ofdemo ~]$ tmboot
     ```

     ![Resultado del comando tmboot](media/base-02.png)

10. Compruebe que el estado del proceso está listo usando el comando tmadmin en si. RDY se muestra en la columna **estado** para cada uno de los procesos:

     ![Resultado del comando tmadmin](media/base-03.png)

11. Apague OpenFrame Base:

     ```
     [oframe7@ofdemo ~]$ tmdown 
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(vtammgr:43) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofruisvr:41) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrsmlog:42) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: CLH downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: CLL downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TLM downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TMM downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-openframe-batch"></a>Instalación de OpenFrame Base

OpenFrame Batch consta de varios componentes que simulan entornos por lotes del sistema central y se usa para ejecutar trabajos por lotes en Azure.

**Para instalar Batch**

1. Asegúrese de que la instalación base se ha realizado correctamente y, después, compruebe que el archivo de instalación OpenFrame\_Batch7\_0\_Fix2\_MVS\_Linux\_x86\_64.bin y el archivo de configuración base.properties están presentes:

2. En el símbolo del sistema, escriba `vi batch.properties` para editar el archivo batch.properties mediante vi.

3. Modifique los parámetros de la forma siguiente:

     ```
     OPENFRAME_HOME = /opt/tmaxapp/OpenFrame
     DEFAULT_VOLSER=DEFVOL 
     TP_NODE_NAME=NODE1 
     TP_NODE_LIST=NODE1 
     RESOURCE_SHMKEY=66991 
     #JOBQ_DATASET_CREATE=YES 
     #OUTPUTQ_DATASET_CREATE=YES 
     DEFAULT_JCLLIB_CREATE=YES 
     DEFAULT_PROCLIB_CREATE=YES 
     DEFAULT_USERLIB_CREATE=YES 
     TJES_USERNAME=tibero 
     TJES_PASSWORD=tmax 
     TJES_DATABASE=oframe 
     BATCH_TABLE_CREATE=YES
     ```

4. Para ejecutar el instalador por lotes, escriba lo siguiente en el símbolo del sistema:

     ```
     ./OpenFrame_Batch7_0_Fix2_MVS_Linux_x86_64.bin -f batch.properties
     ```

5. Una vez completada la instalación, inicie los conjuntos de OpenFrame instalados escribiendo `tmboot` en el símbolo del sistema.

    ![Resultado de tmboot](media/tmboot-01.png)

6. Escriba `tmadmin` en el símbolo del sistema para comprobar el proceso de OpenFrame.

    ![Pantalla de administración de tMax](media/tmadmin-01.png)

7. Ejecute los siguientes comandos:

     ```
     $$2 NODE1 (tmadm): quit 
     ADM quit for node (NODE1)
     ```

8. Use el comando `tmdown` para iniciar y apagar Batch:

     ```
     [oframe7@ofdemo ~]$tmdown
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:44) downed: Wed Sep  7 16:01:46 2016
     TMDOWN: SERVER(vtammgr: 43) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:45) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:46) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:47) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjschd:54) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjinit:55) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:48) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjspbk:57) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:49) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:50) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:51) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:52) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:53) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjhist:56) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofruisvr:41) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmtsmgr:59) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrpmsvr:58) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrsmlog:42) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: CLL downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TLM downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: CLH downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TMM downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-tacf"></a>Instalación de TACF

TACF Manager es un módulo de servicio de OpenFrame que controla el acceso de los usuarios a los sistemas y los recursos a través de la seguridad RACF.

**Para instalar TACF**

1. Compruebe que el archivo de instalación OpenFrame\_Tacf7\_0\_Fix2\_Linux\_x86\_64.bin y el archivo de configuración tacf.properties están presentes.
2. Asegúrese de que la instalación de Batch se realizó correctamente y, después, use vi para abrir el archivo tacf.properties (`vi tacf.properties`).
3. Modifique los parámetros de TACF:

     ```
     OPENFRAME_HOME=/opt/tmaxapp/OpenFrame 
     USE_OS_AUTH=NO 
     TACF_USERNAME=tibero 
     TACF_PASSWORD=tmax 
     TACF_DATABASE=oframe 
     TACF_TABLESPACE=TACF00 
     TACF_TABLE_CREATE=YES
     ```

4. Después de completar el instalador de TACF, aplique las variables del entorno TACF. En el símbolo del sistema, escriba:

     ```
     source \~/.bash\_profile
     ```

5. Ejecute al instalador de TACF. En el símbolo del sistema, escriba:

     ```
     ./OpenFrame_Tacf7_0_Fix2_Linux_x86_64.bin -f tacf.properties
     ```

     El resultado es parecido a este:

     ```
     Wed Dec 07 17:36:42 EDT 2016
     Free Memory: 18703 kB 
     Total Memory: 28800 kB

     4 Command Line Args: 
     0:  -f 1:  tacf.properties 
     2:  -m 
     3:  SILENT 
     java.class.path: 
     /tmp/install.dir.41422/InstallerData 
     /tmp/install.dir.41422/InstallerData/installer.zip 
     ZGUtil.CLASS_PATH: 
     /tmp/install.dir.41422/InstallerData 
     tmp/install.dir.41422/InstallerData/installer.zip 
     sun.boot.class.path: 
     /tmp/install.dir.41422/Linux/resource/jre/lib/resources.jar /tmp/install.dir.41422/Linux/resource/jre/lib/rt.jar /tmp/install.dir.41422/Linux/resource/jre/lib/sunrsasign.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jsse.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jce.jar /tmp/install.dir.41422/Linux/resource/jre/lib/charsets.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jfr.jar /tmp/install.dir.41422/Linux/resource/jre/classes
     ```

6. En el símbolo del sistema, escriba `tmboot` para reiniciar OpenFrame. El resultado es parecido a este:

     ```
     TMBOOT for node(NODE1) is starting: 
     Welcome to Tmax demo system: it will expire 2016/11/4 
     Today: 2016/9/7 
     TMBOOT: TMM is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: CLL is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: CLH is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: TLM(tlm) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrsasvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrlhsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrdmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrdsedt) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrcmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofruisvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrsmlog) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(vtammgr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjschd) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjinit) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjhist) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjspbk) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrpmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmtsmgr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(tmsvr) is starting: Wed Sep  7 17:48:53 2016
     ```

7. Compruebe que el estado del proceso esté listo usando `tmadmin` en el comando `si`. Por ejemplo:

     ```
     [oframe7\@ofdemo \~]\$ tmadmin
     ```

     En la columna **estado**, aparece RDY:

    ![RDY en la columna de estado](media/tmboot-02.png)

8. Ejecute los siguientes comandos:

     ```
     $$2 NODE1 (tmadm): quit 
     DM quit for node (NODE1)

     [oframe7@ofdemo ~]$ tacfmgr 
     Input USERNAME  : ROOT 
     Input PASSWORD  : SYS1

     TACFMGR: TACF MANAGER START!!!
     QUIT TACFMGR: TACF MANAGER END!!!

     [oframe7@ofdemo ~]$ tmdow
     ```

9. Apague el servidor mediante el comando `tmdown`. El resultado es parecido a este:

     ```
     [oframe7@ofdemo ~]$ tmdown 
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjschd:54) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:47) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:48) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:50) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjhist:56) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjspbk:57) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(tmsvr:60) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrpmsvr:58) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmtsmgr:59) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: CLL downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: CLH downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TLM downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TMM downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-prosort"></a>Instalación de ProSort

ProSort es una utilidad que se usan en transacciones por lotes para ordenar los datos.

**Para instalar ProSort**

1. Asegúrese de que la instalación por lotes es correcta y después compruebe que el archivo de instalación **prosort-bin-prosort\_2sp3-linux64-2123-opt.tar.gz** está presente.

2. Ejecute el programa de instalación mediante el archivo de propiedades. En el símbolo del sistema, escriba:

     ```
     tar -zxvf prosort-bin-prosort\_2sp3-linux64-2123-opt.tar.gz
     ```

3. Mueva el directorio prosort a la ubicación principal. En el símbolo del sistema, escriba:

     ```
     mv prosort /opt/tmaxapp/prosort
     ```

4. Cree un subdirectorio de licencia y copie allí el archivo de licencia. Por ejemplo:

     ```
     cd /opt/tmaxapp/prosort 
     mkdir license 
     cp /opt/tmaxsw/oflicense/prosort/license.xml /opt/tmaxapp/prosort/license
     ```

5. Abra bash.profile en vi (`vi .bash_profile`) y actualícelo como se indica a continuación:

     ```bash
     #       PROSORT

     PROSORT_HOME=/opt/tmaxapp/prosort 
     PROSORT_SID=gbg 
     PATH=$PATH:$PROSORT_HOME/bin LD_LIBRARY_PATH=$PROSORT_HOME/lib:$LD_LIBRARY_PATH LIBPATH$PROSORT_HOME/lib:$LIBPATH 
     export PROSORT_HOME PROSORT_SID 
     PATH LD_LIBRARY_PATH LIBPATH 
     PATH=$PATH:$OPENFRAME_HOME/shbin 
     export PATH
     ```

6. Para ejecutar el perfil de bash, escriba lo siguiente en el símbolo del sistema: `. .bash_profile`

7. Cree el archivo de configuración. Por ejemplo:

     ```
     oframe@oframe7: cd /opt/tmaxapp/prosort/config 
     oframe@oframe7: ./gen_tip.sh 
     Using PROSORT_SID "gbg"
      /home/oframe7/prosort/config/gbg.tip generated
     ```

8. Cree el vínculo simbólico. Por ejemplo:

     ```
     oframe@oframe7: cd /opt/tmaxapp/OpenFrame/util/ 
     oframe@oframe7home/oframe7/OpenFrame/util :  ln -s DFSORT SORT
     ```

9. Compruebe la instalación de ProSort ejecutando el comando `prosort -h`. Por ejemplo:

     ```
     oframe@oframe7: prosort -h

     Usage: prosort [options] [sort script files]
     options ------
     -h             Display this information 
     -v             Display version information 
     -s             Display state information 
     -j             Display profile information 
     -x             Use SyncSort compatible mode
     ```

## <a name="install-ofcobol"></a>Instalación de OFCOBOL

OFCOBOL es el compilador de OpenFrame que interpreta los programas de COBOL del sistema central. 

**Para instalar OFCOBOL**

1. Asegúrese de que la instalación en línea o por lotes se realizó correctamente y después compruebe que el archivo de instalación OpenFrame\_COBOL3\_0\_40\_Linux\_x86\_64.bin está presente.

2. Para ejecutar el instalador de OFCOBOL, escriba lo siguiente en el símbolo del sistema:

     ```
      ./OpenFrame\_COBOL3\_0\_40\_Linux\_x86\_64.bin
     ```

3. Lea el contrato de licencia y presione Entrar para continuar.

4. Acepte el contrato de licencia. Cuando la instalación haya finalizado, se mostrará lo siguiente:

     ```
     Choose Install Folder 
     --------------------
     Where would you like to install?
     Default Install Folder: /home/oframe7/OFCOBOL

     ENTER AN ABSOLUTE PATH, OR PRESS <ENTER> TO ACCEPT THE DEFAULT : /opt/tmaxapp/OFCOBOL

     INSTALL FOLDER IS: /opt/tmaxapp/OFCOBOL 
     IS THIS CORRECT? (Y/N): Y[oframe7@ofdemo ~]$ vi .bash_profile

     ============================================================================ Installing... 
     ------------
     [==================|==================|==================|==================] 
     [------------------|------------------|------------------|------------------]

     =============================================================================== Installation Complete 
     --------------------
     Congratulations. OpenFrame_COBOL has been successfully installed
     PRESS <ENTER> TO EXIT THE INSTALLER
     ```

5. Abra el perfil de bash en vi (`vi .bash_profile`) y compruebe que se actualiza con las variables de OFCOBOL.
6. Ejecute el perfil de bash. En el símbolo del sistema, escriba:

     ```
      source ~/.bash_profile
     ```

7. Copie la licencia de OFCOBOL en la carpeta de instalación. Por ejemplo:
     ```
     mv licofcob.dat $OFCOB_HOME/license
     ```
8. Vaya al archivo de configuración tjclrun.conf de OpenFrame y ábralo en vi. Por ejemplo:
     ```
     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/config 
     [oframe7@ofdemo ~]$ vi tjclrun.conf
     ```

   Esta es la sección de SYSLIB anterior al cambio:
     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bin LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${COBDIR}/lib:/ usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib
     ```
   Esta es la sección de SYSLIB posterior al cambio:
     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bin LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${COBDIR}/lib:/ usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib :${ODBC_HOME}/lib 
     :${OFCOB_HOME}/lib
     ```
9. Revise el archivo OpenFrame\_COBOL\_InstallLog.log en vi y compruebe que no hay ningún error. Por ejemplo:
     ```
     [oframe7@ofdemo ~]$ vi $OFCOB_HOME/UninstallerData/log/OpenFrame_COBOL_InstallLog.log 
     …….. 
     Summary 
     ------
     Installation: Successful. 
     131 Successes 
     0 Warnings 
     0 NonFatalErrors 
     0 FatalError
     ```
10. Use el comando `ofcob --version` y revise el número de versión para comprobar la instalación. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ ofcob --version 
     OpenFrame COBOL Compiler 3.0.54 
     CommitTag:: 645f3f6bf7fbe1c366a6557c55b96c48454f4bf
     ```

11. Reinicie OpenFrame utilizando el comando `tmdown/tmboot`.

## <a name="install-ofasm"></a>Instalación de OFASM

OFASM es el compilador de OpenFrame que interpreta los programas del ensamblador del sistema central.

**Para instalar OFASM**

1. Asegúrese de que la instalación en línea o por lotes se realizó correctamente y después compruebe que el archivo de instalación OpenFrame\_ASM3\_0\_Linux\_x86\_64.bin está presente.

2. Ejecute el instalador. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ ./OpenFrame_ASM3_0_Linux_x86_64.bin
     ```

3. Lea el contrato de licencia y presione Entrar para continuar.
4. Acepte el contrato de licencia.
5. Compruebe que el perfil de bash se actualiza con las variables de OFASM. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ source .bash_profile
     [oframe7@ofdemo ~]$ ofasm --version 
     # TmaxSoft OpenFrameAssembler v3 r328 
     (3ff35168d34f6e2046b96415bbe374160fcb3a34)

     [oframe7@ofdemo OFASM]$ vi .bash_profile

     # OFASM ENV 
     export OFASM_HOME=/opt/tmaxapp/OFASM 
     export OFASM_MACLIB=$OFASM_HOME/maclib/free_macro 
     export PATH="${PATH}:$OFASM_HOME/bin:" 
     export LD_LIBRARY_PATH="./:$OFASM_HOME/lib:$LD_LIBRARY_PATH"
     ```

6. Abra el archivo de configuración tjclrun.conf de OpenFrame en vi y modifíquelo como sigue:

     ```
     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/config 
     [oframe7@ofdemo ~]$ vi tjclrun.conf
     ```

     Esta es la sección de [SYSLIB] *anterior* al cambio:

     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bi n:${OPENFRAME_HOME}/volume_default/SYS1.LOADLIB LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${CO BDIR}/lib:/usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib:${OFCOB_HOM E}/lib:${ODBC_HOME}/lib:${OFPLI_HOME}/lib
     ```

     Esta es la sección de [SYSLIB] *posterior* al cambio:

     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bi n:${OPENFRAME_HOME}/volume_default/SYS1.LOADLIB LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${CO BDIR}/lib:/usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib:${OFCOB_HOM E}/lib:${ODBC_HOME}/lib:${OFPLI_HOME}/lib:${OFASM_HOME}/lib
     ```

7. Abra el archivo OpenFrame\_ASM\_InstallLog.log en vi y compruebe que no hay ningún error. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ vi 
     $OFASM_HOME/UninstallerData/log/OpenFrame_ASM_InstallLog.log 
     …….. 
     Summary 
     ------

     Installation: Successful.

     55 Successes 
     0 Warnings 
     0 NonFatalErrors 
     0 FatalErrors
     ```

8. Reinicie OpenFrame mediante uno de los siguientes comandos:

     ```
     tmdown / tmboot
     ```

     O bien:

     ```
     oscdown / oscboot
     ```

## <a name="install-osc"></a>Instalación de OSC

OSC es el entorno de OpenFrame similar al CICS de IBM que admite transacciones OLTP a alta velocidad y otras funciones de administración.

**Para instalar OSC**

1. Asegúrese de que la instalación base se ha realizado correctamente y, después, compruebe que el archivo de instalación OpenFrame\_OSC7\_0\_Fix2\_Linux\_x86\_64.bin y el archivo de configuración osc.properties están presentes.
2. Edite los parámetros siguientes en el archivo osc.properties:
     ```
     OPENFRAME_HOME=/opt/tmaxapp/OpenFrame OSC_SYS_OSC_NCS_PATH=/opt/tmaxapp/OpenFrame/temp/OSC_NCS OSC_APP_OSC_TC_PATH=/opt/tmaxapp/OpenFrame/temp/OSC_TC
     ```

3. Ejecute el programa de instalación mediante el archivo de propiedades como se muestra:

     ```
     [oframe7@ofdemo ~]$ chmod a+x OpenFrame_OSC7_0_Fix2_Linux_x86_64.bin [oframe7@ofdemo ~]$ ./OpenFrame_OSC7_0_Fix2_Linux_x86_64.bin -f osc.properties
     ```

     Cuando haya finalizado, se mostrará el mensaje que indica que la instalación ha finalizado.

4. Compruebe que el perfil de bash se actualiza con las variables de OSC.
5. Revise el archivo OpenFrame\_OSC7\_0\_Fix2\_InstallLog.log. Debe tener el siguiente aspecto:

     ```
     Summary 
     ------ 
     Installation: Successful.

     233 Successes
     0 Warnings
     0 NonFatalErrors
     0 FatalError
     ```

6. Utilice vi para abrir el archivo de configuración ofsys.seq. Por ejemplo:

     ```
     vi $OPENFRAME_HOME/config/ofsys.seq
     ```

7. En las secciones \#BASE y \#BATCH, edite los parámetros como se muestra.

     ```
     Before changes
     #BASE
     ofrsasvr
     ofrlhsvr
     ofrdmsvr
     ofrdsedt
     ofrcmsvr
     ofruisvr
     ofrsmlog
     vtammgr
     TPFMAGENT

     #BATCH 
     #BATCH#obmtsmgr
     #BATCH#ofrpmsvr
     #BATCH#obmjmsvr
     #BATCH#obmjschd
     #BATCH#obmjinit
     #BATCH#obmjhist
     #BATCH#obmjspbk
     #TACF #TACF#tmsvr

     After changes  #BATCH
     #BASE          obmtsmgr 
     ofrsasvr       ofrpmsvr
     ofrlhsvr       obmjmsvr
     ofrdmsvr       obmjschd
     ofrdsedt       obmjinit
     ofrcmsvr       obmjhist
     ofruisvr       obmjspbk
     ofrsmlog
     vtammgr        #TACF
     TPFMAGENT      tmsvr
    ```

8. Copie el archivo de licencia. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ cp /home/oframe7/oflicense/ofonline/licosc.dat 

     $OPENFRAME_HOME/license

     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/license 
     oframe@oframe7/OpenFrame/license / ls -l 
     -rwxr-xr-x. 1 oframe mqm 80 Sep 12 01:37 licosc.dat 
     -rwxr-xr-x. 1 oframe mqm 80 Sep  8 09:40 lictacf.dat 
     -rwxrwxr-x. 1 oframe mqm 80 Sep  3 11:54 lictjes.da
     ```

9. Para iniciar y apagar OSC, inicialice la memoria compartida de la región CICS escribiendo `osctdlinit OSCOIVP1` en el símbolo del sistema.

10. Ejecute `oscboot` para iniciar OSC. El resultado es parecido a este:

     ```
     OSCBOOT : pre-processing       [ OK ]

     TMBOOT for node(NODE1) is starting: 
     Welcome to Tmax demo system: it will expire 2016/11/4 
     Today: 2016/9/12 
          TMBOOT: TMM is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: CLL is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: CLH is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: TLM(tlm) is starting: Mon Sep 12 01:40:25 2016 
     ```

11. Para comprobar que el estado del proceso esté listo usa el comando `tmadmin` en si. Todos los procesos deben mostrar RDY en la columna **estado**.

    ![Procesos que muestran RDY](media/tmadmin-02.png)

12. Cierre OSC mediante el comando `oscdown`.

## <a name="install-jeus"></a>Instalación de JEUS

JEUS (solución para usuarios de Java Enterprise) proporciona la capa de presentación del servidor de aplicaciones web OpenFrame.

Antes de instalar JEUS, instale el paquete de Apache Ant, que proporciona las bibliotecas y herramientas de línea de comandos necesarias para instalar JEUS.

**Para instalar Apache Ant**

1. Descargue el binario de Ant mediante el comando `wget`. Por ejemplo:

     ```
     wget http://apache.mirror.cdnetworks.com/ant/binaries/apacheant-1.9.7-bin.tar.gz
     ```

2. Use la utilidad `tar` para extraer el archivo binario y muévalo a una ubicación adecuada. Por ejemplo:

     ```
     tar -xvzf apache-ant-1.9.7-bin.tar.gz
     ```

3. Para mejorar la eficacia, cree un vínculo simbólico:

     ```
     ln -s apache-ant-1.9.7 ant
     ```

4. Abra el perfil de bash en vi (`vi .bash_profile`) y actualícelo con las siguientes variables:

     ```
     # Ant ENV
     export ANT_HOME=$HOME/ant 
     export PATH=$HOME/ant/bin:$PATH
     ```

5.  Aplique la variable de entorno modificada. Por ejemplo:

     ```
     [oframe7\@ofdemo \~]\$ source \~/.bash\_profile
     ```

**Para instalar JEUS**

1. Expanda el instalador con la utilidad `tar`. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ tar -zxvf jeus704.tar.gz
     ```

2. Cree una carpeta **jeus** (`mkdir jeus7`) y descomprima el archivo binario.
3. Cambie al directorio de **configuración** (o use el parámetro JEUS para su entorno). Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ cd jeus7/setup/
     ```

4. Ejecute `ant clean-all` antes de realizar la compilación. El resultado es parecido a este:

     ```
     Buildfile: /home/oframe7jeus7/setup/build.xml

     clean-bin:
     delete-domain:
     [echo] Deleting a domain configuration: domain = jeus_domain
     delete-nodesxml:
     clean-config:
     clean-all:
     BUILD SUCCESSFUL
     Total time: 0 seconds
     ```

5.  Haga una copia de seguridad del archivo domain-config-template.properties. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ cp domain-config-template.properties domain-configtemplate.properties.bkp
     ```

6. Abra el archivo domain-config-template.properties en vi:

     ```
     [oframe7\@ofdemo setup]\$ vi domain-config-template.properties
     ```

7. Cambie `jeus.password=jeusadmin nodename=Tmaxsoft` a `jeus.password=tmax1234 nodename=ofdemo`.

8. Ejecute el comando `ant install` para compilar JEUS.
9.  Actualice el archivo .bash\_profile con las variables JEUS tal como se muestra:

     ```
     # JEUS ENV 
     export JEUS_HOME=/opt/tmaxui/jeus7 PATH="/opt/tmaxui/jeus7/bin:/opt/tmaxui/jeus7/lib/system:/opt/tmaxui/jeus7/webserver/bin:$ {PATH}" 
     export PATH
     ```

10. Ejecute el perfil de bash. Por ejemplo:

     ```
     [oframe7@ofdemo setup]$ . .bash_profile
     ```

11. *Opcional*. Cree un alias para cerrar y arrancar los componentes de JEUS fácilmente:

     ```     
     # JEUS alias

     alias dsboot='startDomainAdminServer -domain jeus_domain -u administrator -p jeusadmin'
     alias msboot='startManagedServer -domain jeus_domain -server server1 -u administrator -p jeusadmin' 
     alias msdown=‘jeusadmin -u administrator -p tmax1234 "stop-server server1“’ 
     alias dsdown=‘jeusadmin -domain jeus_domain -u administrator -p tmax1234 "local-shutdown“’
     ```

12. Para comprobar la instalación, inicie el servidor de administración de dominio como se muestra:

     ```
     [oframe7@ofdemo ~]$ startDomainAdminServer -domain jeus_domain -u administrator -p jeusadmin
     ```

13. Compruebe por inicio de sesión web mediante la sintaxis:

     ```
     http://<IP>:<port>/webadmin/login
     ```

     Por ejemplo, <http://192.168.92.133:9736/webadmin/login.> Aparece la pantalla de inicio de sesión:
    
     ![Pantalla de inicio de sesión WebAdmin de JEUS](media/jeus-01.png)

     > [!NOTE]
     > Si tiene problemas con la seguridad del puerto, abra el puerto 9736 o deshabilite el firewall (`systemctl stop firewall`).

14. Para cambiar el nombre de host para server1, haga clic en **Lock & Edit** (Bloquear y editar) y después haga clic en **server1**. En la ventana del servidor, cambie el nombre de host de la manera siguiente:

    1.  Cambie **Nodename** por **ofdemo**.
    2.  Haga clic en **OK** (Aceptar) a la derecha de la ventana.
    3.  Haga clic en **Apply changes** (Aplicar cambios) en la parte inferior izquierda de la ventana y, en la descripción, escriba *Hostname change* (Cambio de nombre de host).

    ![Pantalla de WebAdmin de JEUS](media/jeus-02.png)

15. Compruebe que la configuración es correcta en la pantalla de confirmación.

    ![Pantalla del servidor jeus_domain](media/jeus-03.png)

16. Inicie el proceso de servidor administrado “server1” con el siguiente comando:

     ```
     [oframe7@ofdemo ~]$ startManagedServer -domain jeus_domain -server server1 -u administrator -p jeusadmin
     ```

## <a name="install-ofgw"></a>Instalación de OFGW

OFGW es la puerta de enlace de OpenFrame que admite la comunicación entre el emulador terminal 3270 y la base OSI, y administra las sesiones entre el emulador terminal y OSI.

**Para instalar OFGW**

1. Asegúrese de que JEUS se ha instalado correctamente y, después, compruebe que el archivo de instalación OFGW7\_0\_1\_Generic.bin está presente.
2. Ejecute el instalador. Por ejemplo:

     ```
     [oframe7@ofdemo ~]$ ./OFGW7_0_1_Generic.bin
     ````

3. Utilice las siguientes ubicaciones para los mensajes correspondientes:
     -   Directorio de inicio de JEUS
     -   Nombre de dominio de JEUS
     -   Nombre del servidor JEUS
     -   Controlador de Tibero
     -   Id. de nodo tMax ofdemo

4. Acepte el resto de valores predeterminados y presione Entrar para salir del instalador.

5. Compruebe que la dirección URL de OFGW funciona según lo esperado:

     ```
     Type URL 
     http://192.168.92.133:8088/webterminal/ and press enter
      < IP >               :8088/webterminal/
     ```

     Aparece la siguiente pantalla:

    ![WebTerminal de OpenFrame](media/ofgw-01.png)

## <a name="install-ofmanager"></a>Instalación de OFManager

OFManager proporciona funciones de administración y funcionamiento para OpenFrame en el entorno web.

**Para instalar OFManager**

1. Compruebe que el archivo de instalación OFManager7\_Generic.bin está presente.
2. Ejecute el instalador. Por ejemplo:

     ```
     OFManager7_Generic.bin
     ```

3.  Presione Entrar para continuar y, después, acepte el contrato de licencia.
4.  Elija la carpeta de instalación.
5.  Acepte los valores predeterminados.
6.  Elija Tibero como base de datos.
7.  Presione Entrar para salir del instalador.
8.  Compruebe que la dirección URL de OFManager funciona según lo esperado:

     ```
     Type URL http://192.168.92.133:8088/ofmanager and press enter <  IP >  : < PORT >  ofmanager Enter ID:   ROOT 
     Password: SYS1
     ```

Aparecerá la pantalla de inicio:

![Pantalla de inicio de sesión de tMax OpenFrame Manager](media/ofmanager-01.png)

Con esto finaliza la instalación de los componentes de OpenFrame.

## <a name="next-steps"></a>Pasos siguientes

Si se está planteando migrar un sistema central, nuestro ecosistema de asociados en expansión está disponible para ayudarle. Para más información sobre cómo elegir una solución de asociados, consulte [Platform Modernization Alliance](https://datamigration.microsoft.com/).

-   [Comience a usar Azure](../../../../index.yml)
-   [Documentación de Host Integration Server (HIS)](/host-integration-server/)
-   [Azure Virtual Data Center Lift and Shift Guide](/archive/blogs/azurecat/new-whitepaper-azure-virtual-datacenter-lift-and-shift-guide) (Centro de datos virtual de Azure: guía para una migración lift-and-shift)
