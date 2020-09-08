---
title: Acerca de la copia de seguridad de bases de datos SAP HANA en máquinas virtuales de Azure
description: En este artículo obtendrá información sobre cómo realizar copias de seguridad de bases de datos SAP HANA que se ejecutan en máquinas virtuales de Azure.
ms.topic: conceptual
ms.date: 12/11/2019
ms.openlocfilehash: e30510817401fd8db23dc9f1d62fab495fac7ab2
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022316"
---
# <a name="about-sap-hana-database-backup-in-azure-vms"></a>Acerca de la copia de seguridad de bases de datos SAP HANA en máquinas virtuales de Azure

Las bases de datos de SAP HANA son cargas de trabajo críticas que requieren un objetivo de punto de recuperación (RPO) bajo y un objetivo de tiempo de recuperación (RTO) rápido. Ahora puede [hacer una copia de seguridad de las bases de datos de SAP HANA que se ejecutan en máquinas virtuales de Azure](./tutorial-backup-sap-hana-db.md) mediante [Azure Backup](./backup-overview.md).

Azure Backup tiene [certificación Backint](https://www.sap.com/dmc/exp/2013_09_adpd/enEN/#/d/solutions?id=8f3fd455-a2d7-4086-aa28-51d8870acaa5) de SAP, para proporcionar compatibilidad con copias de seguridad nativas aprovechando las API nativas de SAP HANA. Esta oferta de Azure Backup se alinea con el mantra de copias de seguridad de **infraestructura cero**, lo que elimina la necesidad de implementar y administrar la infraestructura de copia de seguridad. Ahora puede realizar copias de seguridad de bases de datos de SAP HANA que se ejecutan en máquinas virtuales de Azure (ya se admiten las [de la serie M](../virtual-machines/m-series.md)) y restaurarlas, y aprovechar las funciones de administración empresarial que proporciona Azure Backup.

## <a name="added-value"></a>Valor agregado

El uso de Azure Backup para realizar copias de seguridad de bases de datos de SAP HANA y restaurarlas, ofrece las ventajas siguientes:

* **Objetivo de punto de recuperación (RPO) de 15 minutos**: ahora se pueden recuperar datos críticos hasta un máximo de 15 minutos.
* **Restauraciones a un momento dado con un clic**: se ha facilitado la restauración de los datos de producción en servidores HANA alternativos. Azure administra en segundo plano el encadenamiento de copias de seguridad y catálogos para realizar restauraciones.
* **Retención a largo plazo**: para satisfacer las necesidades rigurosas de cumplimiento y auditoría. Conserve las copias de seguridad durante años, en función de la duración de la retención, más allá de la cual los puntos de recuperación se eliminarán de forma automática mediante la función integrada de administración del ciclo de vida.
* **Administración de copias de seguridad desde Azure**: use las funciones de administración y supervisión de Azure Backup para mejorar la experiencia de administración. También se admite la CLI de Azure.

Para ver los escenarios de copia de seguridad y restauración que se admiten en la actualidad, consulte la [matriz de compatibilidad de escenarios de SAP HANA](./sap-hana-backup-support-matrix.md#scenario-support).

## <a name="backup-architecture"></a>Arquitectura de copia de seguridad

![Diagrama de la arquitectura de copia de seguridad](./media/sap-hana-db-about/backup-architecture.png)

* El proceso de copia de seguridad comienza por la [creación de un almacén de Recovery Services](./tutorial-backup-sap-hana-db.md#create-a-recovery-services-vault) en Azure. Este almacén se usará para almacenar las copias de seguridad y los puntos de recuperación creados con el tiempo.
* La máquina virtual de Azure que ejecuta el servidor SAP HANA se registra con el almacén y se [detectan](./tutorial-backup-sap-hana-db.md#discover-the-databases) las bases de datos de las que se va a realizar la copia de seguridad. A fin de habilitar el servicio Azure Backup para que detecte las bases de datos, se debe ejecutar un [script de preregistro](https://aka.ms/scriptforpermsonhana) en el servidor HANA como usuario raíz.
* Este script crea el usuario de base de datos **AZUREWLBACKUPHANAUSER** y una clave correspondiente con el mismo nombre en **hdbuserstore**. Consulte la sección [Qué hace el script de registro previo](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does) para más información sobre lo que hace el script.
* Ahora el servicio Azure Backup instala el **complemento de Azure Backup para HANA** en el servidor SAP HANA registrado.
* El **complemento de Azure Backup para HANA** utiliza el usuario de base de datos **AZUREWLBACKUPHANAUSER** creado por el script de preregistro para realizar todas las operaciones de copia de seguridad y restauración. Si intenta configurar la copia de seguridad de las bases de datos de SAP HANA sin ejecutar este script, es posible que reciba el error siguiente: **UserErrorHanaScriptNotRun**.
* Para [configurar la copia de seguridad](./tutorial-backup-sap-hana-db.md#configure-backup) en las bases de datos que se han detectado, elija la directiva de copia de seguridad necesaria y habilite las copias de seguridad.

* Una vez que se haya configurado la copia de seguridad, el servicio Azure Backup configura los parámetros de Backint siguientes en el nivel de base de datos del servidor de SAP HANA protegido:
  * [catalog_backup_using_backint:true]
  * [enable_accumulated_catalog_backup:false]
  * [parallel_data_backup_backint_channels:1]
  * [log_backup_timeout_s:900)]
  * [backint_response_timeout:7200]

>[!NOTE]
>Asegúrese de que estos parámetros *no* estén presentes en el nivel de host. Los parámetros en el nivel de host invalidarán estos parámetros y pueden causar un comportamiento inesperado.
>

* El **complemento de Azure Backup para HANA** mantiene todas las programaciones de copia de seguridad y los detalles de la directiva. Desencadena las copias de seguridad programadas y se comunica con el **motor de copia de seguridad de HANA** a través de las API de Backint.
* El **motor de copia de seguridad de HANA** devuelve una secuencia de Backint con los datos de los que se va a realizar la copia de seguridad.
* Todas las copias de seguridad programadas y las copias de seguridad a petición (desencadenadas desde Azure Portal) que son completas o diferenciales se inician mediante el **complemento de Azure Backup para HANA**. Pero el propio **motor de copia de seguridad de HANA** administra y desencadena las copias de seguridad de registros.
* Azure Backup para SAP HANA, que es una solución con certificación BackInt, no depende de los tipos de máquina virtual o disco subyacente. La copia de seguridad se realiza mediante secuencias generadas por HANA.

## <a name="using-azure-vm-backup-with-azure-sap-hana-backup"></a>Uso de copia de seguridad de máquina virtual de Azure con la copia de seguridad de Azure SAP HANA

Además de usar la copia de seguridad de SAP HANA en Azure, que proporciona copia de seguridad y recuperación en el nivel de base de datos, puede usar la solución de copia de seguridad de máquina virtual de Azure para realizar copias de seguridad de discos que no sean de base de datos y del sistema operativo.

La [solución de copia de seguridad de Azure SAP HANA con certificación BackInt](#backup-architecture) puede usarse para la copia de seguridad y recuperación de bases de datos.

La [copia de seguridad de máquina virtual de Azure](backup-azure-vms-introduction.md) se puede usar para realizar copias de seguridad del sistema operativo y de otros discos que no sean de base de datos. La copia de seguridad de la VM se lleva a cabo una vez al día y realiza una copia de seguridad de todos los discos (excepto los **discos habilitados para el Acelerador de escritura** y los **discos Ultra**). Dado que se realiza una copia de seguridad de la base de datos mediante la solución de copia de seguridad de Azure SAP HANA, puede realizar una copia de seguridad coherente con el archivo solo del sistema operativo y de los discos que no sean de la base de datos mediante la característica de [copias de seguridad y restauración de disco selectivas para las VM de Azure](selective-disk-backup-restore.md).

>[!NOTE]
> El uso de scripts previos y posteriores con la copia de seguridad de máquina virtual de Azure permitirá copias de seguridad coherentes con la aplicación de los volúmenes de datos de la base de datos. Sin embargo, si el área de registro reside en discos habilitados para el Acelerador de escritura, es posible que la toma de una instantánea de estos discos no garantice la coherencia del área de registro. HANA tiene una forma explícita de generar copias de seguridad de registros por este preciso motivo. Habilítela en SAP HANA para poder realizar una copia de seguridad mediante la copia de seguridad de SAP HANA de Azure.

Para restaurar una máquina virtual que ejecuta SAP HANA, siga estos pasos:

* [Restaure una nueva máquina virtual desde la copia de seguridad de máquina virtual de Azure](backup-azure-arm-restore-vms.md) desde el punto de recuperación más reciente. También puede crear una nueva máquina virtual vacía y conectar los discos desde el punto de recuperación más reciente.
* Como no se realizan copias de seguridad de los discos habilitados para el Acelerador de escritura, no se restauran. Cree el área de registro y discos habilitados para el Acelerador de escritura vacíos.
* Una vez establecidas todas las demás configuraciones (como la dirección IP, el nombre del sistema, etc.), la máquina virtual se establece para recibir datos de base de datos de Azure Backup.
* Ahora restaure la base de datos en la máquina virtual desde la [copia de seguridad de base de datos de SAP HANA de Azure](sap-hana-db-restore.md#restore-to-a-point-in-time-or-to-a-recovery-point) en el momento deseado.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [restaurar bases de datos de SAP HANA que se ejecutan en una máquina virtual de Azure](./sap-hana-db-restore.md).
* Obtenga información acerca de cómo [administrar bases de datos de SAP HANA de las que se realizan copias de seguridad mediante Azure Backup](./sap-hana-db-manage.md).
