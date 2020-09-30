---
title: Restauración de bases de datos de SAP HANA en máquinas virtuales de Azure
description: En este artículo se describe cómo restaurar bases de datos de SAP HANA que se ejecutan en máquinas virtuales de Azure.
ms.topic: conceptual
ms.date: 11/7/2019
ms.openlocfilehash: 68858db6f89221e1a3a8f0955d5e009d56e2d365
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89375319"
---
# <a name="restore-sap-hana-databases-on-azure-vms"></a>Restauración de bases de datos de SAP HANA en máquinas virtuales de Azure

En este artículo se describe cómo restaurar bases de datos de SAP HANA que se ejecutan en una máquina virtual (VM) de Azure de la que el servicio Azure Backup ha realizado una copia de seguridad en un almacén de Recovery Services. Las restauraciones se pueden usar para crear copias de los datos para escenarios de desarrollo y pruebas, o para volver a un estado anterior.

Para más información sobre cómo hacer una copia de seguridad de bases de datos de SAP HANA, consulte [Copia de seguridad de bases de datos de SAP HANA en máquinas virtuales de Azure](./backup-azure-sap-hana-database.md).

## <a name="restore-to-a-point-in-time-or-to-a-recovery-point"></a>Restauración a un momento dado o a un punto de recuperación

Azure Backup puede restaurar bases de datos de SAP HANA que se ejecutan en máquinas virtuales de Azure como se indica a continuación:

* Restaurar a una fecha u hora específicas (con precisión de segundos) mediante copias de seguridad de registros. Azure Backup determina automáticamente la copia de seguridad diferencial o completa apropiada, y la cadena de copias de seguridad de registros necesarias para restaurar los datos en función del tiempo seleccionado.

* Restaurar una copia de seguridad completa o diferencial específica para restaurar a un punto de recuperación específico.

## <a name="prerequisites"></a>Requisitos previos

Antes de restaurar una base de datos, tenga en cuenta lo siguiente:

* Solo puede restaurar la base de datos en una instancia de SAP HANA que se encuentre en la misma región.

* La instancia de destino debe estar registrada en el mismo almacén que la de origen.

* Azure Backup no puede identificar dos instancias de SAP HANA diferentes en la misma máquina virtual. Por lo tanto, no es posible restaurar los datos de una instancia en otra de la misma VM.

* Para asegurarse de que la instancia de SAP HANA de destino está lista para la restauración, compruebe su estado de **Preparación para la copia de seguridad**:

  1. Abra el almacén en el que está registrada la instancia de SAP HANA de destino.

  1. En el panel del almacén, en **Introducción**, elija **Copia de seguridad**.

      ![Copia de seguridad en el panel del almacén](media/sap-hana-db-restore/getting-started-backup.png)

  1. En **Copia de seguridad**, en **¿De qué quiere realizar una copia de seguridad?** , elija **SAP HANA en Azure VM**.

      ![Elección de SAP HANA en Azure VM](media/sap-hana-db-restore/sap-hana-backup.png)

  1. En **Detectar bases de datos en máquinas virtuales**, Seleccione **Ver detalles**.

      ![Ver detalles](media/sap-hana-db-restore/view-details.png)

  1. Revise el estado de **preparación para la copia de seguridad** de la máquina virtual de destino.

      ![Los servidores protegidos.](media/sap-hana-db-restore/protected-servers.png)

* Para más información acerca de los tipos de restauración que admite SAP HANA, consulte la nota [1642148](https://launchpad.support.sap.com/#/notes/1642148) de SAP HANA.

## <a name="restore-a-database"></a>Restaurar una base de datos

Para realizar la restauración, necesita los siguientes permisos:

* Permisos de **operador de copia de seguridad** en el almacén en el que se realiza la restauración.
* Acceso de **colaborador (escritura)** a la máquina virtual de origen de la que se realiza la copia de seguridad.
* Acceso de **colaborador (escritura**) a la VM de destino:
  * Si va a restaurar en la misma VM, esta será la VM de origen.
  * Si va a restaurar en una ubicación alternativa, esta será la nueva VM de destino.

1. Abra el almacén en el que está registrada la base de datos de SAP HANA que se va a restaurar.

1. En el panel del almacén, en **Elementos protegidos**, elija **Elementos de copia de seguridad**.

    ![Elementos de copia de seguridad](media/sap-hana-db-restore/backup-items.png)

1. En **Elementos de copia de seguridad**, en **Tipo de administración de copias de seguridad**, seleccione **SAP HANA en Azure VM**.

    ![Tipo de administración de copias de seguridad](media/sap-hana-db-restore/backup-management-type.png)

1. Seleccione la base de datos que se va a restaurar.

    ![Base de datos para restaurar](media/sap-hana-db-restore/database-to-restore.png)

1. Revise el menú de la base de datos. Proporciona información sobre la copia de seguridad de la base de datos, incluidos:

    * Los puntos de restauración más antiguos y más recientes

    * El estado de la copia de seguridad de registros de la base de datos para las últimas 24 y 72 horas

    ![Menú de la base de datos](media/sap-hana-db-restore/database-menu.png)

1. Seleccione **Restaurar base de datos**.

1. En **Configuración de la restauración**, especifique dónde o cómo quiere restaurar los datos:

    * **Ubicación alternativa**: restaura la base de datos en una ubicación alternativa y mantiene la base de datos de origen original.

    * **Sobrescribir la base de datos**: restaura los datos en la misma instancia de SAP HANA que el origen. Esta opción sobrescribe la base de datos original.

      ![Configuración de la restauración](media/sap-hana-db-restore/restore-configuration.png)

### <a name="restore-to-alternate-location"></a>Restauración a una ubicación alternativa

1. En el menú **Restaurar configuración**, en **Where to Restore** (Ubicación de restauración), seleccione **Ubicación alternativa**.

    ![Restauración a una ubicación alternativa](media/sap-hana-db-restore/restore-alternate-location.png)

1. Seleccione el nombre de host de SAP HANA y el nombre de la instancia en la que desea restaurar la base de datos.
1. Compruebe si la instancia de SAP HANA de destino está lista para la restauración; para ello, compruebe su estado de **Preparación para la copia de seguridad**. Consulte la sección [Requisitos previos](#prerequisites) para más información.
1. En el cuadro de diálogo **Nombre de la base de datos restaurada**, escriba el nombre de la base de datos de destino.

    > [!NOTE]
    > Las restauraciones de Contenedor de base de datos única (SDC) deben seguir estas [comprobaciones](backup-azure-sap-hana-database-troubleshoot.md#single-container-database-sdc-restore).

1. Si procede, seleccione **Sobrescribir si ya existe una base de datos con el mismo nombre en la instancia de HANA seleccionada**.
1. Seleccione **Aceptar**.

    ![Configuración de la restauración: pantalla final](media/sap-hana-db-restore/restore-configuration-last.png)

1. En **Seleccionar punto de restauración**, seleccione **Registros (a un momento dado)** a [Restauración a un momento dado](#restore-to-a-specific-point-in-time). O bien, seleccione **Completa y diferencial** para [restaurar a un punto de recuperación específico](#restore-to-a-specific-recovery-point).

### <a name="restore-and-overwrite"></a>Restauración y sobrescritura

1. En el menú **Restaurar configuración**, en **Where to Restore** (Ubicación de restauración), seleccione **Sobrescribir la base de datos** > **Aceptar**.

    ![Sobrescribir la base de datos](media/sap-hana-db-restore/overwrite-db.png)

1. En **Seleccionar punto de restauración**, seleccione **Registros (a un momento dado)** a [Restauración a un momento dado](#restore-to-a-specific-point-in-time). O bien, seleccione **Completa y diferencial** para [restaurar a un punto de recuperación específico](#restore-to-a-specific-recovery-point).

### <a name="restore-as-files"></a>Restaurar como archivos

Para restaurar datos de copia de seguridad como archivos en lugar de una base de datos, seleccione **Restaurar como archivos**. Cuando los archivos se vuelcan en una ruta de acceso especificada, puede llevar estos archivos a cualquier máquina de SAP HANA en la que quiera restaurarlos como base de datos. Dado que puede mover estos archivos a cualquier máquina, ahora puede restaurar los datos entre suscripciones y regiones.

1. En el menú **Restaurar configuración**, en **¿Dónde y cómo se realiza la restauración?** , seleccione **Restaurar como archivos**.
1. Seleccione el nombre de servidor de HANA o **host** en el que quiere restaurar los archivos de copia de seguridad.
1. En **Ruta de acceso de destino en el servidor**, especifique la ruta de acceso de carpetas del servidor seleccionado en el paso 2. Se trata de la ubicación en la que el servicio volcará todos los archivos de copia de seguridad necesarios.

    Los archivos que se vuelcan son:

    * Archivos de copia de seguridad de bases de datos
    * Archivos de catálogo
    * Archivos de metadatos JSON (para cada archivo de copia de seguridad implicado)

    Típicamente, una ruta de acceso a un recurso compartido de red o una ruta de acceso de un recurso compartido de archivos de Azure montado cuando se especifica como una ruta de acceso de destino facilita el acceso a estos archivos de parte de otras máquinas en la red o en el mismo recurso compartido de archivos de Azure montado en ellas.

    >[!NOTE]
    >Para restaurar los archivos de copia de seguridad de base de datos en un recurso compartido de archivos de Azure montado en la VM registrada de destino, asegúrese de que la cuenta raíz tenga accesos de lectura o escritura en el recurso compartido de archivos de Azure.

    ![Elección de la ruta de acceso de destino](media/sap-hana-db-restore/restore-as-files.png)

1. Seleccione el **Punto de restauración** correspondiente en el que se restaurarán todas las carpetas y archivos de copia de seguridad.

    ![Seleccionar punto de restauración](media/sap-hana-db-restore/select-restore-point.png)

1. Todos los archivos de copia de seguridad asociados con el punto de restauración seleccionado se vuelcan en la ruta de acceso de destino.
1. En función del tipo de punto de restauración elegido (**Momento dado** o **Completa y diferencial**), se crearán una o varias carpetas en la ruta de acceso de destino. Una de las carpetas, denominada `Data_<date and time of restore>`, contiene las copias de seguridad completas y diferenciales y, la otra, llamada `Log`, contiene las copias de seguridad de registros.
1. Mueva estos archivos restaurados al servidor de SAP HANA en el que quiere restaurarlos como base de datos.
1. A continuación, siga estos pasos:
    1. Establezca permisos en la carpeta o el directorio en que se almacenan los archivos de copia de seguridad mediante el comando siguiente:

        ```bash
        chown -R <SID>adm:sapsys <directory>
        ```

    1. Ejecute el siguiente conjunto de comandos como `<SID>adm`.

        ```bash
        su - <sid>adm
        ```

    1. Genere el archivo de catálogo para la restauración. Extraiga **BackupId** del archivo de metadatos JSON para la copia de seguridad completa, que se usará más adelante en la operación de restauración. Asegúrese de que las copias de seguridad completas y de registros se encuentran en carpetas diferentes y elimine los archivos de catálogo y los archivos de metadatos JSON de estas carpetas.

        ```bash
        hdbbackupdiag --generate --dataDir <DataFileDir> --logDirs <LogFilesDir> -d <PathToPlaceCatalogFile>
        ```

        En el comando anterior:

        * `<DataFileDir>`: carpeta que contiene las copias de seguridad completas
        * `<LogFilesDir>`: carpeta que contiene las copias de seguridad de registros
        * `<PathToPlaceCatalogFile>`: carpeta donde se debe colocar el archivo de catálogo generado

    1. Realice la restauración con el archivo de catálogo recién generado mediante HANA Studio o ejecute la consulta de restauración HDBSQL con este catálogo recién generado. Las consultas HDBSQL se enumeran a continuación:

    * Para restaurar a un momento dado:

        Si está creando una nueva base de datos restaurada, ejecute el comando HDBSQL para crear una nueva base de datos `<DatabaseName>` y, a continuación, detenga la base de datos para la restauración. Sin embargo, si solo está restaurando una base de datos existente, ejecute el comando HDBSQL para detenerla.

        A continuación, ejecute el siguiente comando para restaurar la base de datos:

        ```hdbsql
        RECOVER DATABASE FOR <DatabaseName> UNTIL TIMESTAMP '<TimeStamp>' CLEAR LOG USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING LOG PATH (' <LogFileDir>') USING DATA PATH ('<DataFileDir>') USING BACKUP_ID <BackupIdFromJsonFile> CHECK ACCESS USING FILE
        ```

        * `<DatabaseName>`: nombre de la base de datos nueva o existente que quiere restaurar
        * `<Timestamp>`: marca de tiempo exacta de la restauración a un momento dado
        * `<DatabaseName@HostName>`: nombre de la base de datos cuya copia de seguridad se usa para la restauración y nombre del servidor de SAP HANA o **host** en que reside esta base de datos. La opción `USING SOURCE <DatabaseName@HostName>` especifica que la copia de seguridad de datos (usada para la restauración) es de una base de datos con un SID o nombre diferente al de la máquina de SAP HANA de destino. Por lo tanto, no es necesario especificarla para las restauraciones realizadas en el mismo servidor de HANA del que se toma la copia de seguridad.
        * `<PathToGeneratedCatalogInStep3>`: ruta de acceso al archivo de catálogo generado en el **paso C**
        * `<DataFileDir>`: carpeta que contiene las copias de seguridad completas
        * `<LogFilesDir>`: carpeta que contiene las copias de seguridad de registros
        * `<BackupIdFromJsonFile>`: **BackupId** extraído en el **paso C**

    * Para restaurar una copia de seguridad completa o diferencial determinada:

        Si está creando una nueva base de datos restaurada, ejecute el comando HDBSQL para crear una nueva base de datos `<DatabaseName>` y, a continuación, detenga la base de datos para la restauración. Sin embargo, si solo está restaurando una base de datos existente, ejecute el comando HDBSQL para detenerla:

        ```hdbsql
        RECOVER DATA FOR <DatabaseName> USING BACKUP_ID <BackupIdFromJsonFile> USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING DATA PATH ('<DataFileDir>')  CLEAR LOG
        ```

        * `<DatabaseName>`: nombre de la base de datos nueva o existente que quiere restaurar
        * `<Timestamp>`: marca de tiempo exacta de la restauración a un momento dado
        * `<DatabaseName@HostName>`: nombre de la base de datos cuya copia de seguridad se usa para la restauración y nombre del servidor de SAP HANA o **host** en que reside esta base de datos. La opción `USING SOURCE <DatabaseName@HostName>` especifica que la copia de seguridad de datos (usada para la restauración) es de una base de datos con un SID o nombre diferente al de la máquina de SAP HANA de destino. Por lo tanto, no es necesario especificarla para las restauraciones realizadas en el mismo servidor de HANA del que se toma la copia de seguridad.
        * `<PathToGeneratedCatalogInStep3>`: ruta de acceso al archivo de catálogo generado en el **paso C**
        * `<DataFileDir>`: carpeta que contiene las copias de seguridad completas
        * `<LogFilesDir>`: carpeta que contiene las copias de seguridad de registros
        * `<BackupIdFromJsonFile>`: **BackupId** extraído en el **paso C**

### <a name="restore-to-a-specific-point-in-time"></a>Restauración a un momento dado

Si ha seleccionado **Registros (punto en el tiempo)** como el tipo de restauración, haga lo siguiente:

1. Seleccione un punto de recuperación en el gráfico del registro y seleccione **Aceptar** para elegir el punto de restauración.

    ![Punto de restauración](media/sap-hana-db-restore/restore-point.png)

1. En el menú **Restaurar**, seleccione **Restaurar** para iniciar el trabajo de restauración.

    ![Selección de restauración](media/sap-hana-db-restore/restore-restore.png)

1. Realice un seguimiento del progreso de la restauración en el área **Notificaciones** o seleccione **Trabajos de restauración** en el menú de la base de datos.

    ![Restauración desencadenada correctamente](media/sap-hana-db-restore/restore-triggered.png)

### <a name="restore-to-a-specific-recovery-point"></a>Restauración a un punto de recuperación específico

Si ha seleccionado **Completo y diferencial** como el tipo de restauración, haga lo siguiente:

1. Seleccione un punto de recuperación en la lista y seleccione **Aceptar** para elegir el punto de restauración.

    ![Restauración a un punto de recuperación específico](media/sap-hana-db-restore/specific-recovery-point.png)

1. En el menú **Restaurar**, seleccione **Restaurar** para iniciar el trabajo de restauración.

    ![Inicio del trabajo de restauración](media/sap-hana-db-restore/restore-specific.png)

1. Realice un seguimiento del progreso de la restauración en el área **Notificaciones** o seleccione **Trabajos de restauración** en el menú de la base de datos.

    ![Progreso de la restauración](media/sap-hana-db-restore/restore-progress.png)

    > [!NOTE]
    > En las restauraciones de Contenedor de varias bases de datos (MDC), cuando la base de datos del sistema se restaura en una instancia de destino, es necesario volver a ejecutar el script de registro previo. Solo entonces las restauraciones de bases de datos de inquilino subsiguientes se realizarán correctamente. Para más información, consulte [Solución de problemas: restauración de MDC](backup-azure-sap-hana-database-troubleshoot.md#multiple-container-database-mdc-restore).

## <a name="next-steps"></a>Pasos siguientes

* [Obtenga información acerca de cómo](sap-hana-db-manage.md) administrar bases de datos de SAP HANA de las que se realizan copias de seguridad mediante Azure Backup.
