---
title: 'Sincronización de Azure AD Connect: Scheduler | Microsoft Docs'
description: En este tema se describe la característica de Programador incorporada en la sincronización de Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 6b1a598f-89c0-4244-9b20-f4aaad5233cf
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/01/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: c2b65f8cd22e72e0ba90918121a02d66fe6bf3e7
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053055"
---
# <a name="azure-ad-connect-sync-scheduler"></a>Sincronización de Azure AD Connect: Scheduler
En este tema se describe el programador incorporado en la sincronización de Azure AD Connect (motor de sincronización).

Esta característica se introdujo con la compilación 1.1.105.0 (publicada en febrero de 2016).

## <a name="overview"></a>Información general
Azure AD Connect Sync sincronizará los cambios que se producen en su directorio local mediante un programador. Hay dos procesos de programador, uno para la sincronización de contraseñas y otro para la sincronización de objetos o atributos y tareas de mantenimiento. En este tema se trata el último.

En versiones anteriores, el programador de objetos y atributos era externo al motor de sincronización. Usaba el programador de tareas de Windows o un servicio de Windows independiente para desencadenar el proceso de sincronización. En las versiones 1.1 el programador viene integrado con el motor de sincronización y permite alguna personalización. La nueva frecuencia de sincronización predeterminada es de 30 minutos.

El programador es responsable de dos tareas:

* **Ciclo de sincronización**. El proceso para importar, sincronizar y exportar los cambios.
* **Tareas de mantenimiento**. Renueve las claves y certificados para el restablecimiento de contraseña y el servicio de registro de dispositivos (DRS). Purgue las entradas antiguas en el registro de operaciones.

El programador en sí, siempre está en ejecución, pero se puede configurar para que ejecute solo una o ninguna de estas tareas. Por ejemplo, si necesita tener su propio proceso de ciclo de sincronización, puede deshabilitar esta tarea en el programador y continuar ejecutando la tarea de mantenimiento.

>[!IMPORTANT]
>Un ciclo de sincronización se ejecuta cada 30 minutos de manera predeterminada. Si modificó el ciclo de sincronización, deberá asegurarse de que se ejecute un ciclo de sincronización al menos una vez cada siete días. 
>
>* Una sincronización diferencial debe ocurrir en un plazo de siete días desde la última sincronización diferencial.
>* Una sincronización diferencial (después de una sincronización completa) tiene que realizarse en un plazo de siete días a partir del momento en que se completó la última sincronización completa.
>
>Si no lo hace, pueden generarse problemas de sincronización que requerirán que ejecute una sincronización completa para resolverlos. Esto también se aplica a los servidores en modo de almacenamiento provisional.

## <a name="scheduler-configuration"></a>Configuración de Scheduler
Para ver la configuración actual, vaya a PowerShell y ejecute `Get-ADSyncScheduler`. Muestra algo parecido a esta imagen:

![GetSyncScheduler](./media/how-to-connect-sync-feature-scheduler/getsynccyclesettings2016.png)

Si al ejecutar este cmdlet, aparece el error **The sync command or cmdlet is not available** (El comando de sincronización o el cmdlet no están disponibles), significa que el módulo de PowerShell no está cargado. Este problema podría suceder si ejecuta Azure AD Connect en un controlador de dominio o en un servidor con niveles de restricción para PowerShell más elevados que la configuración predeterminada. Si ve este error, ejecute `Import-Module ADSync` para que esté disponible el cmdlet.

* **AllowedSyncCycleInterval**. El intervalo de tiempo más corto entre ciclos de sincronización permitido por Azure AD. No se puede sincronizar con más frecuencia que esta configuración y mantener la compatibilidad.
* **CurrentlyEffectiveSyncCycleInterval**. La programación en vigor actualmente. Tiene el mismo valor que CustomizedSyncInterval (si está establecido), si no es más frecuente que AllowedSyncInterval. Si usa una compilación anterior a 1.1.281 y cambia CustomizedSyncCycleInterval, esto surte efecto tras el próximo ciclo de sincronización. A partir de la compilación 1.1.281, el cambio surte efecto inmediatamente.
* **CustomizedSyncCycleInterval**. Si desea que el programador se ejecute con cualquier otra frecuencia distinta a la del valor predeterminado de 30 minutos, configure este valor. En la imagen anterior, el programador se ha establecido para que se ejecute cada hora. Si establece esta configuración en un valor inferior al de AllowedSyncInterval, se utilizará el último.
* **NextSyncCyclePolicyType**. Diferencial o inicial. Define si la siguiente ejecución debe procesar solo cambios diferenciales, o bien si debería hacer una importación y sincronización completas. En el último caso también vuelve a procesar las reglas nuevas o modificadas.
* **NextSyncCycleStartTimeInUTC**. La hora en que el programador inicia el siguiente ciclo de sincronización.
* **PurgeRunHistoryInterval**. El tiempo que deben mantenerse los registros de operación. Estos registros se pueden revisar en Synchronization Service Manager. El valor predeterminado mantener estos registros 7 días.
* **SyncCycleEnabled**. Indica si el programador está ejecutando los procesos de importación, sincronización y exportación como parte de su funcionamiento.
* **MaintenanceEnabled**. Muestra si está habilitado el proceso de mantenimiento. Actualiza los certificados y las claves y purga el registro de operaciones.
* **StagingModeEnabled**. Muestra si el [modo provisional](how-to-connect-sync-staging-server.md) está habilitado. Si esta opción está habilitada, suprime la ejecución de las exportaciones, pero ejecuta la importación y la sincronización.
* **SchedulerSuspended**. Configure Connect durante una actualización para impedir temporalmente que se ejecute el programador.

Puede cambiar algunos de estos valores con `Set-ADSyncScheduler`. Se pueden modificar los parámetros siguientes:

* CustomizedSyncCycleInterval
* NextSyncCyclePolicyType
* PurgeRunHistoryInterval
* SyncCycleEnabled
* MaintenanceEnabled

En compilaciones anteriores de Azure AD Connect, **isStagingModeEnabled** se expuso en Set-ADSyncScheduler. **No se puede** establecer esta propiedad. La propiedad **SchedulerSuspended** solo debe modificarse con Connect. **No se puede** establecer esto con PowerShell directamente.

La configuración del programador se almacena en Azure AD. Si tiene un servidor de ensayo, cualquier cambio realizado en el servidor principal también afectará a este servidor (excepto IsStagingModeEnabled).

### <a name="customizedsynccycleinterval"></a>CustomizedSyncCycleInterval
Sintaxis: `Set-ADSyncScheduler -CustomizedSyncCycleInterval d.HH:mm:ss`  
d: días; HH: horas; mm: minutos; ss: segundos

Ejemplo: `Set-ADSyncScheduler -CustomizedSyncCycleInterval 03:00:00`  
Cambia el programador para ejecutarse cada 3 horas.

Ejemplo: `Set-ADSyncScheduler -CustomizedSyncCycleInterval 1.0:0:0`  
Cambia el programador para ejecutarse a diario.

### <a name="disable-the-scheduler"></a>Deshabilitación del programador  
Si necesita realizar cambios en la configuración, es conveniente deshabilitar el programador. Por ejemplo, cuando [configura el filtrado](how-to-connect-sync-configure-filtering.md) o [realiza cambios en las reglas de sincronización](how-to-connect-sync-change-the-configuration.md).

Para deshabilitar el programador, ejecute `Set-ADSyncScheduler -SyncCycleEnabled $false`.

![Deshabilitación del programador](./media/how-to-connect-sync-feature-scheduler/schedulerdisable.png)

Una vez realizados los cambios, no olvide volver a habilitar el programador con `Set-ADSyncScheduler -SyncCycleEnabled $true`.

## <a name="start-the-scheduler"></a>Inicio del programador
De forma predeterminada, el programador se ejecuta cada 30 minutos. En algunos casos, es posible que quiera ejecutar un ciclo de sincronización entre los ciclos programados o necesite ejecutar un tipo diferente.

### <a name="delta-sync-cycle"></a>Ciclo de sincronización diferencial
Un ciclo de sincronización diferencial incluye los siguientes pasos:


- Importación diferencial en todos los conectores
- Sincronización diferencial en todos los conectores
- Exportación en todos los conectores

### <a name="full-sync-cycle"></a>Ciclo de sincronización completo
Un ciclo de sincronización completo incluye los pasos siguientes:

- Importación completa en todos los conectores
- Sincronización completa en todos los conectores
- Exportación en todos los conectores

Es posible que haya un cambio urgente que debe sincronizar inmediatamente, para lo que necesita ejecutar manualmente un ciclo. 

Si necesita ejecutar manualmente un ciclo de sincronización, ejecute `Start-ADSyncSyncCycle -PolicyType Delta`desde PowerShell.

Para iniciar un ciclo de sincronización completo, ejecute `Start-ADSyncSyncCycle -PolicyType Initial` desde un símbolo del sistema de PowerShell.   

La ejecución de un ciclo de sincronización completo puede llevar mucho tiempo; lea la siguiente sección para saber cómo optimizar este proceso.

### <a name="sync-steps-required-for-different-configuration-changes"></a>Pasos de sincronización requeridos para diferentes cambios de configuración
Los diferentes cambios de configuración requieren diferentes pasos de sincronización para garantizar que los cambios se aplican correctamente a todos los objetos.

- Se agregaron más objetos o atributos para importar desde un directorio de origen (esto es, agregando o modificando las reglas de sincronización).
    - Es necesario realizar una importación completa en el conector de ese directorio de origen.
- Realizó cambios en las reglas de sincronización
    - Es necesario realizar una sincronización completa en el conector para las reglas de sincronización cambiadas.
- Cambió el [filtrado](how-to-connect-sync-configure-filtering.md) para que se incluya un número diferente de objetos
    - Es necesario realizar una importación completa en el conector para cada conector de AD, A MENOS que esté usando una operación de filtrado basada en atributos que ya se están importando en el motor de sincronización.

### <a name="customizing-a-sync-cycle-run-the-right-mix-of-delta-and-full-sync-steps"></a>La personalización de un ciclo de sincronización ejecuta la combinación correcta de pasos de sincronización diferencial y completa.
Para evitar ejecutar un ciclo de sincronización completo, puede marcar conectores específicos para que ejecuten un paso completo mediante los siguientes cmdlets.

`Set-ADSyncSchedulerConnectorOverride -Connector <ConnectorGuid> -FullImportRequired $true`

`Set-ADSyncSchedulerConnectorOverride -Connector <ConnectorGuid> -FullSyncRequired $true`

`Get-ADSyncSchedulerConnectorOverride -Connector <ConnectorGuid>` 

Ejemplo:  Si realizó cambios en las reglas de sincronización del conector "AD Forest A" que no requieren la importación de nuevos atributos, puede ejecutar los siguientes cmdlets para ejecutar un ciclo de sincronización diferencial en el cual también se realizó un paso de sincronización completa para ese conector.

`Set-ADSyncSchedulerConnectorOverride -ConnectorName “AD Forest A” -FullSyncRequired $true`

`Start-ADSyncSyncCycle -PolicyType Delta`

Ejemplo:  Si realizó cambios en las reglas de sincronización del conector "AD Forest A" y ahora es necesario importar nuevos atributos, puede ejecutar los siguientes cmdlets para ejecutar un ciclo de sincronización diferencial en el cual también se realizó un paso de sincronización e importación completa para ese conector.

`Set-ADSyncSchedulerConnectorOverride -ConnectorName “AD Forest A” -FullImportRequired $true`

`Set-ADSyncSchedulerConnectorOverride -ConnectorName “AD Forest A” -FullSyncRequired $true`

`Start-ADSyncSyncCycle -PolicyType Delta`


## <a name="stop-the-scheduler"></a>Detención del programador
Si el programador está ejecutando actualmente un ciclo de sincronización, puede que necesite detenerlo. Por ejemplo, si inicia el Asistente para instalación y recibe este error:

![SyncCycleRunningError](./media/how-to-connect-sync-feature-scheduler/synccyclerunningerror.png)

Cuando se está ejecutando un ciclo de sincronización, no puede realizar cambios de configuración. Debe esperar hasta que el programador haya terminado el proceso, pero también es posible detenerlo para poder realizar los cambios inmediatamente. Detener el ciclo actual no es perjudicial y los cambios pendientes se procesarán en la próxima ejecución.

1. Para empezar, indique al programador que detenga el ciclo actual con el cmdlet `Stop-ADSyncSyncCycle`de PowerShell.
2. Si usa una compilación anterior a 1.1.281, detener el programador no hará que este conector detenga su tarea actual. Para forzar el conector a que se detenga, tome las medidas siguientes:  ![StopAConnector](./media/how-to-connect-sync-feature-scheduler/stopaconnector.png)
   * Inicie el **Servicio de sincronización** desde el menú Inicio. Vaya a **Conectores**, resalte el conector con el estado **En ejecución** y seleccione **Detener** en la lista de acciones.

El programador todavía permanece activo y se inicia de nuevo en la siguiente oportunidad.

## <a name="custom-scheduler"></a>Programador personalizado
Los cmdlets que se documentan en esta sección solo están disponibles en la compilación [1.1.130.0](reference-connect-version-history.md) y versiones posteriores.

Si el programador integrado no cumple con sus requisitos, puede programar los conectores con PowerShell.

### <a name="invoke-adsyncrunprofile"></a>Invoke-ADSyncRunProfile
Puede iniciar un perfil para un conector de esta manera:

```
Invoke-ADSyncRunProfile -ConnectorName "name of connector" -RunProfileName "name of profile"
```

Los nombres para utilizar como [nombres de conectores](how-to-connect-sync-service-manager-ui-connectors.md) y [nombres de perfil de ejecución](how-to-connect-sync-service-manager-ui-connectors.md#configure-run-profiles) pueden encontrarse en la [interfaz de usuario de Synchronization Service Manager](how-to-connect-sync-service-manager-ui.md).

![Invocar el perfil de ejecución](./media/how-to-connect-sync-feature-scheduler/invokerunprofile.png)  

El cmdlet `Invoke-ADSyncRunProfile` es sincrónico; es decir, no devuelve el control hasta que el conector ha completado la operación, ya sea correcta o incorrectamente.

Al programar los conectores, se recomienda hacerlo en el siguiente orden:

1. (Diferencial y completo) Importar desde directorios locales, como Active Directory
2. (Diferencial y completo) Importar desde Azure AD
3. (Diferencial y completo) Sincronización desde directorios locales, como Active Directory
4. (Diferencial y completo) Sincronización desde Azure AD
5. Exportar a Azure AD
6. Exportar a los directorios locales, como Active Directory

Este orden es en el que el programador integrado ejecuta los conectores.

### <a name="get-adsyncconnectorrunstatus"></a>Get-ADSyncConnectorRunStatus
También puede supervisar el motor de sincronización para ver si está ocupado o inactivo. Este cmdlet devuelve un resultado vacío si el motor de sincronización está inactivo y no está ejecutando ningún conector. Si se está ejecutando un conector, devuelve el nombre del conector.

```
Get-ADSyncConnectorRunStatus
```

![Estado de la ejecución de conector](./media/how-to-connect-sync-feature-scheduler/getconnectorrunstatus.png)  
En la ilustración anterior, la primera línea refleja un estado donde el motor de sincronización está inactivo. La segunda línea es de cuando se ejecuta el conector de Azure AD.

## <a name="scheduler-and-installation-wizard"></a>Scheduler y Asistente para instalación
Si inicia el Asistente para instalación, el programador se suspende temporalmente. Este comportamiento es debido a que se supone que realizará cambios en la configuración y estos cambios no se pueden aplicar si el motor de sincronización se está ejecutando activamente. Por este motivo, no deje el Asistente para instalación abierto, ya que impide que el motor de sincronización realizar ninguna acción.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la configuración de la [Sincronización de Azure AD Connect](how-to-connect-sync-whatis.md) .

Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).
