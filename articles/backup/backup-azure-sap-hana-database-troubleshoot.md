---
title: Solución de problemas de errores de copia de seguridad de bases de datos de SAP HANA
description: En este artículo se describe cómo se solucionan los errores comunes que pueden producirse al usar Azure Backup para realizar copias de seguridad de bases de datos de SAP HANA.
ms.topic: troubleshooting
ms.date: 11/7/2019
ms.openlocfilehash: 5cdad55ef849b9ced31646466e2c2c170ebf0827
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89377691"
---
# <a name="troubleshoot-backup-of-sap-hana-databases-on-azure"></a>Solución de problemas al realizar copias de seguridad de bases de datos de SAP HANA en Azure

En este artículo se proporciona información para la solución de problemas al realizar copias de seguridad de bases de datos de SAP HANA en Azure Virtual Machines. Para más información sobre los escenarios de copia de seguridad de SAP HANA actualmente compatibles, consulte [Compatibilidad con los escenarios](sap-hana-backup-support-matrix.md#scenario-support).

## <a name="prerequisites-and-permissions"></a>Requisitos previos y permisos

Consulte las secciones sobre los [requisitos previos](tutorial-backup-sap-hana-db.md#prerequisites) y [Qué hace el script de registro previo](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does) antes de configurar las copias de seguridad.

## <a name="common-user-errors"></a>Errores de usuario comunes

### <a name="usererrorhanainternalrolenotpresent"></a>UserErrorHANAInternalRoleNotPresent

| **Mensaje de error**      | <span style="font-weight:normal">Azure Backup no tiene los privilegios de rol necesarios para llevar a cabo la copia de seguridad</span>    |
| ---------------------- | ------------------------------------------------------------ |
| **Causas posibles:**    | Es posible que se haya sobrescrito el rol.                          |
| **Acción recomendada** | Para resolver el problema, ejecute el script desde el panel **Discover DB** (Detectar BD) o descárguelo [aquí](https://aka.ms/scriptforpermsonhana). También puede agregar el rol "SAP_INTERNAL_HANA_SUPPORT " al usuario de copia de seguridad de la carga de trabajo (AZUREWLBACKUPHANAUSER). |

### <a name="usererrorinopeninghanaodbcconnection"></a>UserErrorInOpeningHanaOdbcConnection

| Mensaje de error      | <span style="font-weight:normal">No se pudo conectar con el sistema HANA</span>                        |
| ------------------ | ------------------------------------------------------------ |
| **Causas posibles:**    | La instancia de SAP HANA puede estar fuera de servicio.<br/>No se han establecido los permisos necesarios para que Azure Backup interactúe con la base de datos de HANA. |
| **Acción recomendada** | Compruebe si la base de datos de SAP HANA está activa. Si la base de datos está en funcionamiento, compruebe si se establecieron todos los permisos necesarios. Si falta alguno de los permisos, ejecute el [script de registro previo](https://aka.ms/scriptforpermsonhana) para agregar los permisos que faltan. |

### <a name="usererrorhanainstancenameinvalid"></a>UserErrorHanaInstanceNameInvalid

| Mensaje de error      | <span style="font-weight:normal">La instancia de SAP HANA especificada no es válida ni se puede encontrar</span>  |
| ------------------ | ------------------------------------------------------------ |
| **Causas posibles:**    | No se puede realizar una copia de seguridad de varias instancias de SAP HANA en una máquina virtual de Azure única. |
| **Acción recomendada** | Ejecute el [script de registro previo](https://aka.ms/scriptforpermsonhana) en la instancia de SAP HANA de la que quiere hacer una copia de seguridad. Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Microsoft. |

### <a name="usererrorhanaunsupportedoperation"></a>UserErrorHanaUnsupportedOperation

| Mensaje de error      | <span style="font-weight:normal">No se admite la operación de SAP HANA especificada</span>              |
| ------------------ | ------------------------------------------------------------ |
| **Causas posibles:**    | Azure Backup para SAP HANA no es compatible con la copia de seguridad incremental ni con las acciones realizadas en clientes nativos de SAP HANA (Studio/Cockpit/DBA Cockpit). |
| **Acción recomendada** | Para más información, consulte [aquí](./sap-hana-backup-support-matrix.md#scenario-support). |

### <a name="usererrorhanapodoesnotsupportbackuptype"></a>UserErrorHANAPODoesNotSupportBackupType

| Mensaje de error      | <span style="font-weight:normal">Esta base de datos de SAP HANA no admite el tipo de copia de seguridad solicitada</span>  |
| ------------------ | ------------------------------------------------------------ |
| **Causas posibles:**    | Azure Backup no es compatible con la copia de seguridad incremental ni con la copia de seguridad mediante instantáneas. |
| **Acción recomendada** | Para más información, consulte [aquí](./sap-hana-backup-support-matrix.md#scenario-support). |

### <a name="usererrorhanalsnvalidationfailure"></a>UserErrorHANALSNValidationFailure

| Mensaje de error      | <span style="font-weight:normal">La cadena de registros de copia de seguridad se interrumpió</span>                                    |
| ------------------ | ------------------------------------------------------------ |
| **Causas posibles:**    | Es posible que el destino de la copia de seguridad de registros se haya actualizado desde BackInt al sistema de archivos o puede que el ejecutable de BackInt se haya modificado. |
| **Acción recomendada** | Desencadene una copia de seguridad completa para resolver el problema.                   |

### <a name="usererrorsdctomdcupgradedetected"></a>UserErrorSDCtoMDCUpgradeDetected

| Mensaje de error      | <span style="font-weight:normal">Actualización de SDC a MDC detectada</span>                                   |
| ------------------ | ------------------------------------------------------------ |
| **Causas posibles:**    | La instancia de SAP HANA se actualizó de SDC a MDC. Las copias de seguridad presentarán un error después de la actualización. |
| **Acción recomendada** | Siga los pasos indicados en [actualización de SDC a MDC](#sdc-to-mdc-upgrade-with-a-change-in-sid) para resolver el problema. |

### <a name="usererrorinvalidbackintconfiguration"></a>UserErrorInvalidBackintConfiguration

| Mensaje de error      | <span style="font-weight:normal">Se detectó una configuración de tipo Backint no válida</span>                       |
| ------------------ | ------------------------------------------------------------ |
| **Causas posibles:**    | Los parámetros de respaldo están incorrectamente especificados para Azure Backup. |
| **Acción recomendada** | Compruebe si se establecieron estos parámetros (BackInt):<br/>\* [catalog_backup_using_backint:true]<br/>\* [enable_accumulated_catalog_backup:false]<br/>\* [parallel_data_backup_backint_channels:1]<br/>\* [log_backup_timeout_s:900)]<br/>\* [backint_response_timeout:7200]<br/>Si los parámetros basados en BackInt están presentes en el host, quítelos. Si los parámetros no están presentes en el nivel del host, pero se han modificado manualmente en el nivel de la base de datos, reviértalos a los valores apropiados como se describió anteriormente. O bien, ejecute [Detener la protección y conservar los datos de copia de seguridad](./sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database) en Azure Portal y, después, seleccione **Reanudar copia de seguridad**. |

### <a name="usererrorincompatiblesrctargetsystemsforrestore"></a>UserErrorIncompatibleSrcTargetSystemsForRestore

|Mensaje de error  |Los sistemas de origen y destino para la restauración no son compatibles.  |
|---------|---------|
|Causas posibles   | Los sistemas de origen y destino que seleccionó para la restauración no son compatibles        |
|Acción recomendada   |   Asegúrese de que el escenario de restauración no está incluido en la siguiente lista de posibles restauraciones incompatibles: <br><br>   **Caso 1:** No se puede cambiar el nombre de SYSTEMDB durante la restauración.  <br><br> **Caso 2:** Origen SDC y destino MDC: La base de datos de origen no se puede restaurar como SYSTEMDB o base de datos de inquilino en el destino. <br><br> **Caso 3:** Origen MDC y destino SDC: La base de datos de origen (SYSTEMDB o base de datos de inquilino) no se puede restaurar en el destino. <br><br>  Para obtener más información, consulte la nota **1642148** en [Launchpad de soporte técnico de SAP](https://launchpad.support.sap.com). |

## <a name="restore-checks"></a>Comprobaciones de restauración

### <a name="single-container-database-sdc-restore"></a>Restauración de una base de datos de contenedor único (SDC)

Ocúpese de las entradas mientras se realiza la restauración de una base de datos de contenedor único (SDC) para HANA a otra máquina de SDC. El nombre de la base de datos debe estar en minúscula con "sdc" anexado entre corchetes. La instancia de HANA se mostrará en mayúsculas.

Supongamos que hay una copia de seguridad de una instancia "H21" de HANA de SDC. En la página de elementos de copia de seguridad aparecerá el nombre del elemento de copia de seguridad como **"h21(sdc)"** . Si intenta restaurar esta base de datos en otra SDC de destino, por ejemplo, H11, es necesario proporcionar las entradas siguientes.

![Nombre de base de datos SDC restaurada](media/backup-azure-sap-hana-database/hana-sdc-restore.png)

Tenga en cuenta los siguientes puntos:

- De manera predeterminada, el nombre de la base de datos restaurada se rellenará con el nombre del elemento de copia de seguridad. En este caso, h21(sdc).
- Si selecciona H11 como destino, el nombre de la base de datos restaurada no cambiará de manera automática. **Se debe editar para cambiarlo a h11(sdc)** . Con respecto a SDC, el nombre de la base de datos restaurada será el identificador de la instancia de destino en minúsculas más "sdc" entre corchetes.
- Dado que SDC solo puede tener una base de datos única, también debe seleccionar la casilla para permitir reemplazar los datos existentes de la base de datos por los datos del punto de recuperación.
- Linux distingue mayúsculas de minúsculas. De este modo, tiene que tener cuidado y respetar su uso.

### <a name="multiple-container-database-mdc-restore"></a>Restauración de base de datos de varios contenedores (MDC)

En las bases de datos de varios contenedores para HANA, la configuración estándar es SYSTEMDB + 1 o más bases de datos de inquilino. Restaurar toda una instancia de SAP HANA significa restaurar tanto SYSTEMDB como las bases de datos de inquilino. Primero se restaura SYSTEMDB y, luego, la base de datos de inquilino. Básicamente, restaurar la base de datos del sistema significa reemplazar la información del sistema del destino seleccionado. Esta restauración también reemplaza la información relacionada con BackInt en la instancia de destino. Así pues, una vez que la base de datos del sistema se restaura en una instancia de destino, vuelva a ejecutar el script de registro previo. Solo entonces las restauraciones de bases de datos de inquilino subsiguientes se realizarán correctamente.

## <a name="back-up-a-replicated-vm"></a>Copia de seguridad de una VM replicada

### <a name="scenario-1"></a>Escenario 1

La máquina virtual original se replicó mediante Azure Site Recovery o una copia de seguridad de la máquina virtual de Azure. La nueva máquina virtual se creó para simular la máquina virtual anterior. Es decir, la configuración es exactamente la misma. (Esto se debe a que la máquina virtual original se eliminó y la restauración se realizó desde la copia de seguridad de la máquina virtual o Azure Site Recovery).

Este escenario podría incluir dos posibles casos. Obtenga información sobre cómo realizar una copia de seguridad de la máquina virtual replicada en ambos casos:

1. La máquina virtual recién creada tiene el mismo nombre y se encuentra en el mismo grupo de recursos y en la misma suscripción que la máquina virtual que se eliminó.

    - La extensión ya está presente en la máquina virtual, pero ninguno de los servicios puede verla.
    - Ejecute el script de registro previo.
    - Vuelva a registrar la extensión para la misma máquina en Azure Portal (**Copia de seguridad** -> **Ver detalles** -> seleccione la máquina virtual de Azure pertinente -> Volver a registrar).
    - Se debe comenzar a crear correctamente la copia de seguridad de las bases de datos que ya tienen una copia de seguridad (de la máquina virtual eliminada).

2. La máquina virtual recién creada tiene:

    - un nombre diferente al de la máquina virtual eliminada.
    - el mismo nombre que la máquina virtual eliminada, pero se encuentra en otro grupo de recursos o suscripción (en comparación con la máquina virtual eliminada).

    Si es este el caso, siga estos pasos:

    - La extensión ya está presente en la máquina virtual, pero ninguno de los servicios puede verla.
    - Ejecute el script de registro previo.
    - Si detecta y protege las bases de datos nuevas, comenzarán a mostrarse bases de datos activas duplicadas en el portal. Para evitar esta situación, [detenga la protección con la conservación de datos](sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database) para las bases de datos anteriores. Después, continúe con los pasos restantes.
    - Detecte las bases de datos para habilitar la copia de seguridad.
    - Habilite las copias de seguridad en estas bases de datos.
    - Las bases de datos existentes con copia de seguridad (de la máquina virtual eliminada) seguirán almacenadas en el almacén (y sus copias de seguridad se conservarán según la directiva).

### <a name="scenario-2"></a>Escenario 2

La máquina virtual original se replicó mediante Azure Site Recovery o una copia de seguridad de la máquina virtual de Azure. La nueva máquina virtual se creó a partir del contenido, con el fin de usarse como plantilla. Se trata de una nueva máquina virtual con un nuevo SID.

Siga estos pasos para habilitar las copias de seguridad en la máquina virtual nueva:

- La extensión ya está presente en la máquina virtual, pero ninguno de los servicios puede verla.
- Ejecute el script de registro previo. En función del SID de la nueva máquina virtual, pueden surgir dos escenarios:
  - La máquina virtual original y la nueva tienen el mismo SID. El script de registro previo se ejecutará correctamente.
  - La máquina virtual original y la nueva tienen SID diferentes. El script de registro previo generará un error. Póngase en contacto con el soporte técnico para obtener ayuda en este caso.
- Detecte las bases de datos de las que quiere realizar copias de seguridad.
- Habilite las copias de seguridad en estas bases de datos.

## <a name="sdc-version-upgrade-or-mdc-version-upgrade-on-the-same-vm"></a>Actualización de la versión de SDC o MDC en la misma máquina virtual

Las actualizaciones del sistema operativo, el cambio de versión de SDC o el cambio de versión de MDC que no ocasionan un cambio de SID se pueden administrar de la siguiente manera:

- Asegúrese de que la nueva versión del sistema operativo, de SDC o de MDC [sea compatible actualmente con Azure Backup](sap-hana-backup-support-matrix.md#scenario-support).
- [Detenga la protección de datos con la conservación de datos](sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database) para la base de datos.
- Realice la actualización.
- Vuelva a ejecutar el script de registro previo. Normalmente, el proceso de actualización elimina los roles necesarios. La ejecución del script de registro previo le ayudará a comprobar todos los roles necesarios.
- Reanude la protección de la base de datos nuevamente.

## <a name="sdc-to-mdc-upgrade-with-no-change-in-sid"></a>Actualización de SDC a MDC sin cambios en el SID

Las actualizaciones de SDC a MDC que no provocan un cambio de SID se pueden administrar de la siguiente manera:

- Asegúrese de que la nueva versión de MDC [sea compatible actualmente con Azure Backup](sap-hana-backup-support-matrix.md#scenario-support).
- [Detenga la protección con la conservación de datos](sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database) para la base de datos SDC anterior.
- Realice la actualización. Después de la finalización, el sistema HANA es ahora MDC con una base de datos de sistema y bases de datos de inquilino.
- Vuelva a ejecutar el [script de registro previo](https://aka.ms/scriptforpermsonhana).
- Vuelva a registrar la extensión para la misma máquina en Azure Portal (**Copia de seguridad** -> **Ver detalles** -> seleccione la máquina virtual de Azure pertinente -> Volver a registrar).
- Seleccione **Volver a detectar bases de datos** para la misma VM. Esta acción debería mostrar las nuevas bases de datos en el paso 3 como SYSTEMDB y base de datos de inquilino, no SDC.
- La base de datos SDC anterior seguirá existiendo en el almacén y se conservarán los datos de copia de seguridad antiguos según la directiva.
- Configure la copia de seguridad de estas bases de datos.

## <a name="sdc-to-mdc-upgrade-with-a-change-in-sid"></a>Actualización de SDC a MDC con un cambios de SID

Las actualizaciones de SDC a MDC que provocan un cambio de SID se pueden administrar de la siguiente manera:

- Asegúrese de que la nueva versión de MDC [sea compatible actualmente con Azure Backup](sap-hana-backup-support-matrix.md#scenario-support).
- **Detenga la protección con la conservación de datos** para la base de datos SDC anterior.
- Realice la actualización. Después de la finalización, el sistema HANA es ahora MDC con una base de datos de sistema y bases de datos de inquilino.
- Vuelva a ejecutar el [script de registro previo](https://aka.ms/scriptforpermsonhana) con los detalles correctos (nuevo SID y MDC). Debido al cambio en el SID, puede que surjan problemas con la ejecución correcta del script. Póngase en contacto con el soporte técnico de Azure Backup si tiene problemas.
- Vuelva a registrar la extensión para la misma máquina en Azure Portal (**Copia de seguridad** -> **Ver detalles** -> seleccione la máquina virtual de Azure pertinente -> Volver a registrar).
- Seleccione **Volver a detectar bases de datos** para la misma VM. Esta acción debería mostrar las nuevas bases de datos en el paso 3 como SYSTEMDB y base de datos de inquilino, no SDC.
- La base de datos SDC anterior seguirá existiendo en el almacén y se conservarán los datos de copia de seguridad antiguos según la directiva.
- Configure la copia de seguridad de estas bases de datos.

## <a name="re-registration-failures"></a>Errores de repetición del registro

Compruebe uno o varios de los siguientes síntomas antes de desencadenar la operación de repetición del registro:

- Se producen errores en todas las operaciones (tales como copia de seguridad, restauración y configuración de copia de seguridad) en la máquina virtual con uno de los códigos de error siguientes: **WorkloadExtensionNotReachable, UserErrorWorkloadExtensionNotInstalled, WorkloadExtensionNotPresent, WorkloadExtensionDidntDequeueMsg**.
- Si el área **Estado de copia de seguridad** del elemento de copia de seguridad se muestra como **no accesible**, descarte todas las demás causas que podrían dar lugar al mismo estado:

  - Falta de permiso para realizar operaciones relacionadas con la copia de seguridad en la máquina virtual
  - La máquina virtual está apagada, por lo que no se pueden realizar copias de seguridad.
  - Problemas de red

Estos síntomas pueden surgir por uno o varios de los siguientes motivos:

- Se ha eliminado o desinstalado una extensión del portal.
- Se ha restaurado la máquina virtual retrocediendo en el tiempo a través de la restauración del disco o discos en contexto.
- Se ha apagado la máquina virtual durante un largo período, por lo que la configuración de la extensión en ella ha caducado.
- Se ha eliminado la máquina virtual, y se ha creado otra con el mismo nombre y en el mismo grupo de recursos que la máquina virtual eliminada.

En los escenarios anteriores, se recomienda desencadenar una operación de nuevo registro en la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

- Revise las [preguntas frecuentes](./sap-hana-faq-backup-azure-vm.md) sobre cómo hacer copias de seguridad de las bases de datos SAP HANA en máquinas virtuales de Azure.
