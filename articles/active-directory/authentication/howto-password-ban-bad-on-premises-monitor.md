---
title: Supervisión de la protección con contraseña de Azure AD local
description: Aprenda a supervisar y revisar los registros de la protección con contraseña de Azure AD para un entorno de Active Directory Domain Services local.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 02937c22cbc16defb0b7672ac7ebc56c2ae2beb5
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89068786"
---
# <a name="monitor-and-review-logs-for-on-premises-azure-ad-password-protection-environments"></a>Supervisión y revisión de los registros de los entornos de protección con contraseña de Azure AD local

Una vez realizada la implementación de protección con contraseña de Azure AD, la supervisión y la notificación son tareas fundamentales. En este artículo se ofrece información detallada para ayudarle a conocer las distintas técnicas de supervisión, además de explicar dónde registra cada servicio la información y cómo envía notificaciones sobre el uso de la protección con contraseña de Azure AD.

La supervisión y el registro se realizan mediante mensajes de registro de eventos o mediante la ejecución de cmdlets de PowerShell. Los servicios del agente de controlador de dominio y de proxy registran mensajes en el registro de eventos. Todos los cmdlets de PowerShell que se describe a continuación solo están disponibles en el servidor proxy (consulte el módulo de PowerShell AzureADPasswordProtection). El software del agente de controlador de dominio no instala ningún módulo de PowerShell.

## <a name="dc-agent-event-logging"></a>Registro de eventos del agente de controlador de dominio

En cada controlador de dominio, el software del servicio del agente de controlador de dominio escribe los resultados de cada operación de validación de contraseña individual y otros estados en un registro de eventos local:

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Admin`

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Operational`

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Trace`

El registro de administración del agente de controlador de dominio es la principal fuente de información de cómo se comporta el software.

Tenga en cuenta que el registro de seguimiento está desactivado de forma predeterminada.

Los eventos registrados por los distintos componentes del agente de controlador de dominio se engloban en los siguientes intervalos:

|Componente |Intervalo de identificadores de evento|
| --- | --- |
|DLL de filtro de contraseña del agente de controlador de dominio| 10000-19999|
|Proceso de hospedaje del servicio de agente de controlador de dominio| 20000-29999|
|Lógica de validación de directivas de servicio del agente de controlador de dominio| 30000-39999|

## <a name="dc-agent-admin-event-log"></a>Registro de eventos de administración del agente de controlador de dominio

### <a name="password-validation-outcome-events"></a>Eventos de resultado de validación de contraseña

En cada controlador de dominio, el software del servicio del agente de controlador de dominio escribe los resultados de cada validación de contraseña individual en un registro de eventos de administración del agente de controlador de dominio.

Para una operación de validación de contraseñas correcta se registra, por lo general, un evento desde la DLL de filtro de contraseña del agente de controlador de dominio. Si se producen errores durante la validación, se registrarán, por lo general, dos eventos, uno desde el servicio de agente de controlador de dominio y otro desde la DLL de filtro de contraseña de este agente.

Se registran eventos discretos para capturar estas situaciones en función de los siguientes factores:

* Si se está estableciendo o cambiando una contraseña determinada.
* Si la validación de una contraseña se ha realizado correctamente o se han producido errores.
* Si se produjo un error durante la validación debido al uso de la directiva global de Microsoft, la directiva de la organización o una combinación de ambas.
* Si el modo de solo auditoría está actualmente activado o desactivado para la directiva de contraseñas actual.

Los principales eventos relacionados con la validación de contraseñas son los siguientes:

| Evento |Cambio de contraseña |Establecimiento de contraseña|
| --- | :---: | :---: |
|Pass (pasado) |10014 |10015|
|Error (debido a la directiva de contraseñas del cliente)| 10016, 30002| 10017, 30003|
|Error (debido a la directiva de contraseñas de Microsoft)| 10016, 30004| 10017, 30005|
|Error (debido a las directivas de Microsoft y de cliente combinadas)| 10016, 30026| 10017, 30027|
|Superó la fase de solo auditoría (la directiva de contraseñas del cliente habría generado errores)| 10024, 30008| 10025, 30007|
|Superó la fase de solo auditoría (la directiva de contraseñas de Microsoft habría generado errores)| 10024, 30010| 10025, 30009|
|Superó la fase de solo auditoría (las directivas de contraseña de Microsoft y del cliente combinadas habrían generado errores)| 10024, 30028| 10025, 30029|

Los casos expuestos en la tabla anterior que hacen referencia a "directivas combinadas" hacen referencia a situaciones en las que se detectó que la contraseña de un usuario contenía al menos un token tanto de la lista de contraseñas prohibidas de Microsoft como de la lista de contraseñas prohibidas del cliente.

Cuando un par de eventos se registran juntos, ambos se asocian de manera explícita al tener el mismo parámetro CorrelationId.

### <a name="password-validation-summary-reporting-via-powershell"></a>Información de resumen de validación de contraseña mediante PowerShell

Se puede usar el cmdlet `Get-AzureADPasswordProtectionSummaryReport` para generar una vista resumida de la actividad de validación de contraseña. A continuación se muestra un ejemplo de salida de este cmdlet:

```powershell
Get-AzureADPasswordProtectionSummaryReport -DomainController bplrootdc2
DomainController                : bplrootdc2
PasswordChangesValidated        : 6677
PasswordSetsValidated           : 9
PasswordChangesRejected         : 10868
PasswordSetsRejected            : 34
PasswordChangeAuditOnlyFailures : 213
PasswordSetAuditOnlyFailures    : 3
PasswordChangeErrors            : 0
PasswordSetErrors               : 1
```

Se puede influenciar el ámbito de los informes del cmdlet con uno de estos parámetros: -Forest, -Domain o -DomainController. No especificar un parámetro implica –Forest.

El cmdlet `Get-AzureADPasswordProtectionSummaryReport` funciona mediante la consulta del registro de eventos de administración del agente de controlador de dominio y el posterior recuento del número total de eventos correspondiente a cada categoría de resultados mostrada. La tabla siguiente contiene las asignaciones entre cada resultado y su identificador de evento correspondiente:

|Propiedad Get-AzureADPasswordProtectionSummaryReport |Identificador de evento correspondiente|
| :---: | :---: |
|PasswordChangesValidated |10014|
|PasswordSetsValidated |10015|
|PasswordChangesRejected |10016|
|PasswordSetsRejected |10017|
|PasswordChangeAuditOnlyFailures |10024|
|PasswordSetAuditOnlyFailures |10025|
|PasswordChangeErrors |10012|
|PasswordSetErrors |10013|

Tenga en cuenta que el cmdlet `Get-AzureADPasswordProtectionSummaryReport` se suministra en forma de script de PowerShell y, si es necesario, se puede hacer referencia a él directamente en la siguiente ubicación:

`%ProgramFiles%\WindowsPowerShell\Modules\AzureADPasswordProtection\Get-AzureADPasswordProtectionSummaryReport.ps1`

> [!NOTE]
> Este cmdlet funciona al abrir una sesión de PowerShell en cada controlador de dominio. Para que la operación se realice correctamente, se debe habilitar la compatibilidad con la sesión remota de PowerShell en cada controlador de dominio y el cliente debe tener privilegios suficientes. Para más información sobre los requisitos de la sesión remota de Powershell, ejecute "Get-Help about_Remote_Troubleshooting" en una ventana de PowerShell.

> [!NOTE]
> Este cmdlet funciona mediante la consulta remota del registro de eventos de administración de cada servicio de agente de controlador de dominio. Si los registros de eventos contienen un gran número de eventos, el cmdlet puede tardar mucho tiempo en completarse. Además, las consultas de red masivas de grandes conjuntos de datos pueden afectar al rendimiento del controlador de dominio. Por lo tanto, este cmdlet debe usarse con cuidado en entornos de producción.

### <a name="sample-event-log-message-for-event-id-10014-successful-password-change"></a>Ejemplo de mensaje de registro de evento para el evento con identificador 10014 (cambio correcto de contraseña)

```text
The changed password for the specified user was validated as compliant with the current Azure password policy.

 UserName: BPL_02885102771
 FullName:
```

### <a name="sample-event-log-message-for-event-id-10017-and-30003-failed-password-set"></a>Ejemplo de mensaje de registro de evento para el evento con identificador 10017 y 30003 (error en el establecimiento de contraseña)

10017:

```text
The reset password for the specified user was rejected because it did not comply with the current Azure password policy. Please see the correlated event log message for more details.

 UserName: BPL_03283841185
 FullName:
```

30003:

```text
The reset password for the specified user was rejected because it matched at least one of the tokens present in the per-tenant banned password list of the current Azure password policy.

 UserName: BPL_03283841185
 FullName:
```

### <a name="sample-event-log-message-for-event-id-30001-password-accepted-due-to-no-policy-available"></a>Ejemplo de mensaje de registro de evento para el evento con identificador 30001 (contraseña aceptada ya que no hay ninguna directiva disponible)

```text
The password for the specified user was accepted because an Azure password policy is not available yet

UserName: SomeUser
FullName: Some User

This condition may be caused by one or more of the following reasons:%n

1. The forest has not yet been registered with Azure.

   Resolution steps: an administrator must register the forest using the Register-AzureADPasswordProtectionForest cmdlet.

2. An Azure AD password protection Proxy is not yet available on at least one machine in the current forest.

   Resolution steps: an administrator must install and register a proxy using the Register-AzureADPasswordProtectionProxy cmdlet.

3. This DC does not have network connectivity to any Azure AD password protection Proxy instances.

   Resolution steps: ensure network connectivity exists to at least one Azure AD password protection Proxy instance.

4. This DC does not have connectivity to other domain controllers in the domain.

   Resolution steps: ensure network connectivity exists to the domain.
```

### <a name="sample-event-log-message-for-event-id-30006-new-policy-being-enforced"></a>Ejemplo de mensaje de registro de evento para el evento con identificador 30006 (nueva directiva aplicada)

```text
The service is now enforcing the following Azure password policy.

 Enabled: 1
 AuditOnly: 1
 Global policy date: ‎2018‎-‎05‎-‎15T00:00:00.000000000Z
 Tenant policy date: ‎2018‎-‎06‎-‎10T20:15:24.432457600Z
 Enforce tenant policy: 1
```

### <a name="sample-event-log-message-for-event-id-30019-azure-ad-password-protection-is-disabled"></a>Mensaje de registro de eventos de ejemplo para el evento con identificador 30019 (protección con contraseña de Azure AD deshabilitada)

```text
The most recently obtained Azure password policy was configured to be disabled. All passwords submitted for validation from this point on will automatically be considered compliant with no processing performed.

No further events will be logged until the policy is changed.%n

```

## <a name="dc-agent-operational-log"></a>Registro operativo del agente de controlador de dominio

El servicio del agente de controlador de dominio también registrará los eventos relacionados con operaciones en el registro siguiente:

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Operational`

## <a name="dc-agent-trace-log"></a>Registro de seguimiento del agente de controlador de dominio

El servicio del agente de controlador de dominio también puede registrar eventos de seguimiento de nivel de depuración detallado en el registro siguiente:

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Trace`

El registro de seguimiento está deshabilitado de manera predeterminada.

> [!WARNING]
> Cuando se habilita, el registro de seguimiento recibe un gran volumen de eventos y esto puede afectar al rendimiento del controlador de dominio. Por tanto, este registro mejorado solo se debe habilitar si un problema requiere una investigación más profunda y solo durante un período mínimo de tiempo.

## <a name="dc-agent-text-logging"></a>Registro de texto del agente de controlador de dominio

El servicio del agente de controlador de dominio puede configurarse para escribir en un registro de texto estableciendo el valor de registro siguiente:

```text
HKLM\System\CurrentControlSet\Services\AzureADPasswordProtectionDCAgent\Parameters!EnableTextLogging = 1 (REG_DWORD value)
```

El registro de texto está deshabilitado de manera predeterminada. El reinicio del servicio del agente de controlador de dominio es necesario para que los cambios realizados en este valor surtan efecto. Cuando se habilita, el servicio del agente de controlador de dominio escribirá en un archivo de registro ubicado en:

`%ProgramFiles%\Azure AD Password Protection DC Agent\Logs`

> [!TIP]
> El registro de texto recibe las mismas entradas de nivel de depuración que se pueden registrar en el registro de seguimiento, pero suelen tener un formato más fácil para revisar y analizar.

> [!WARNING]
> Cuando se habilita, este registro recibe un gran volumen de eventos y esto puede afectar al rendimiento del controlador de dominio. Por tanto, este registro mejorado solo se debe habilitar si un problema requiere una investigación más profunda y solo durante un período mínimo de tiempo.

## <a name="dc-agent-performance-monitoring"></a>Supervisión del rendimiento del agente de controlador de dominio

El software del servicio de agente de controlador de dominio instala un objeto de contador de rendimiento denominado **Protección con contraseña de Azure AD**. Los siguientes contadores de rendimiento están disponibles actualmente:

|Nombre del contador de rendimiento | Descripción|
| --- | --- |
|Contraseñas procesadas |Este contador muestra el número total de contraseñas que se procesan (aceptadas o rechazadas) desde el último reinicio.|
|Contraseñas aceptadas |Este contador muestra el número total de contraseñas que se aceptaron desde el último reinicio.|
|Contraseñas rechazadas |Este contador muestra el número total de contraseñas que se rechazaron desde el último reinicio.|
|Solicitudes de filtro de contraseñas en curso |Este contador muestra el número de solicitudes de filtro de contraseñas actualmente en curso.|
|Máximo de solicitudes de filtro de contraseñas |Este contador muestra el número máximo de solicitudes de filtro de contraseña simultáneas desde el último reinicio.|
|Errores de solicitudes de filtro de contraseñas |Este contador muestra el número total de solicitudes de filtro de contraseñas en las que se produjo algún error desde el último reinicio. Se pueden producir errores si el servicio de agente de controlador de dominio de protección con contraseña de Azure AD no se ejecuta.|
|Solicitudes de filtro de contraseñas por segundo |Este contador muestra la velocidad a la que se procesan las contraseñas.|
|Tiempo de procesamiento de la solicitud de filtro de contraseñas |Este contador muestra el tiempo promedio necesario para procesar una solicitud de filtro de contraseñas.|
|Tiempo máximo de procesamiento de la solicitud de filtro de contraseñas |Este contador muestra el tiempo máximo de procesamiento de solicitudes de filtro de contraseñas desde el último reinicio.|
|Contraseñas que se aceptan debido al modo de auditoría |Este contador muestra el número total de contraseñas que normalmente se habrían rechazado, pero que se aceptaron porque la directiva de contraseñas se configuró para estar en modo de auditoría (desde el último reinicio).|

## <a name="dc-agent-discovery"></a>Detección del agente de controlador de dominio

Se puede usar el cmdlet `Get-AzureADPasswordProtectionDCAgent` para mostrar información básica sobre los diversos agentes de controlador de dominio que se están ejecutando en un dominio o bosque. Esta información se recupera de los objetos serviceConnectionPoint registrados por los servicios de agente de controlador de dominio en ejecución.

A continuación se muestra un ejemplo de salida de este cmdlet:

```powershell
Get-AzureADPasswordProtectionDCAgent
ServerFQDN            : bplChildDC2.bplchild.bplRootDomain.com
Domain                : bplchild.bplRootDomain.com
Forest                : bplRootDomain.com
PasswordPolicyDateUTC : 2/16/2018 8:35:01 AM
HeartbeatUTC          : 2/16/2018 8:35:02 AM
```

Cada servicio de agente de controlador de dominio actualiza las diversas propiedades una vez cada hora aproximadamente. Los datos siguen estando sujetos a la latencia de replicación de Active Directory.

Se puede influenciar el ámbito de la consulta del cmdlet con los parámetros -Forest o -Domain.

Si el valor HeartbeatUTC se queda obsoleto, puede ser indicativo de que el agente de controlador de dominio de protección con contraseña de Azure AD de dicho controlador de dominio no se está ejecutando o de que la máquina se ha degradado y ya no es un controlador de dominio.

Si el valor PasswordPolicyDateUTC se queda obsoleto, puede ser indicativo de que el agente de controlador de dominio de protección con contraseña de Azure AD de dicha máquina no funciona correctamente.

## <a name="dc-agent-newer-version-available"></a>Versión más reciente del agente de controlador de dominio disponible

El servicio del agente de controlador de dominio registrará un evento de advertencia 30034 en el registro operativo cuando detecte que hay disponible una versión más reciente del software de dicho agente, por ejemplo:

```text
An update for Azure AD Password Protection DC Agent is available.

If autoupgrade is enabled, this message may be ignored.

If autoupgrade is disabled, refer to the following link for the latest version available:

https://aka.ms/AzureADPasswordProtectionAgentSoftwareVersions

Current version: 1.2.116.0
```

El evento anterior no especifica la versión del software más reciente. Debe ir al vínculo en el mensaje de evento para esa información.

> [!NOTE]
> A pesar de las referencias a la actualización automática en el mensaje de evento anterior, el software del agente de controlador de dominio no admite actualmente esta característica.

## <a name="proxy-service-event-logging"></a>Registros de eventos del servicio de proxy

El servicio de proxy emite un conjunto mínimo de eventos para los registros de eventos siguientes:

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\ProxyService\Admin`

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\ProxyService\Operational`

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\ProxyService\Trace`

Tenga en cuenta que el registro de seguimiento está desactivado de forma predeterminada.

> [!WARNING]
> Cuando se habilita, el registro de seguimiento recibe un gran volumen de eventos y esto puede afectar al rendimiento del host del proxy. Por tanto, este registro solo se debe habilitar si un problema requiere una investigación más profunda y solo durante un período mínimo de tiempo.

Los diversos componentes de proxy registran los eventos mediante los siguientes intervalos:

|Componente |Intervalo de identificadores de evento|
| --- | --- |
|Proceso de hospedaje del servicio de proxy| 10000-19999|
|Lógica de negocios principal del servicio de proxy| 20000-29999|
|Cmdlets de PowerShell| 30000-39999|

## <a name="proxy-service-text-logging"></a>Registro de texto del servicio de proxy

El servicio de proxy puede configurarse para escribir en un registro de texto estableciendo el valor de registro siguiente:

HKLM\System\CurrentControlSet\Services\AzureADPasswordProtectionProxy\Parameters!EnableTextLogging = 1 (REG_DWORD value)

El registro de texto está deshabilitado de manera predeterminada. El reinicio del servicio de proxy es necesario para que los cambios realizados en este valor surtan efecto. Cuando se habilita, el servicio de proxy escribirá en un archivo de registro ubicado en:

`%ProgramFiles%\Azure AD Password Protection Proxy\Logs`

> [!TIP]
> El registro de texto recibe las mismas entradas de nivel de depuración que se pueden registrar en el registro de seguimiento, pero suelen tener un formato más fácil para revisar y analizar.

> [!WARNING]
> Cuando se habilita, este registro recibe un gran volumen de eventos y esto puede afectar al rendimiento de la máquina. Por tanto, este registro mejorado solo se debe habilitar si un problema requiere una investigación más profunda y solo durante un período mínimo de tiempo.

## <a name="powershell-cmdlet-logging"></a>Registro del cmdlet de PowerShell

Los cmdlets de PowerShell que producen un cambio de estado (por ejemplo, Register-AzureADPasswordProtectionProxy) normalmente registrarán un evento de resultado en el registro operativo.

Además, la mayoría de los cmdlets de PowerShell de la protección con contraseña de Azure AD escribirán en un registro de texto que se encuentra en:

`%ProgramFiles%\Azure AD Password Protection Proxy\Logs`

Si se produce un error de cmdlet y la causa o la solución no son evidentes, también se pueden consultar estos registros de texto.

## <a name="proxy-discovery"></a>Detección de proxy

Se puede usar el cmdlet `Get-AzureADPasswordProtectionProxy` para mostrar información básica sobre los diversos servicios de proxy de la protección con contraseña de Azure AD que se están ejecutando en un dominio o un bosque. Esta información se recupera de los objetos serviceConnectionPoint registrados por los servicios de proxy en ejecución.

A continuación se muestra un ejemplo de salida de este cmdlet:

```powershell
Get-AzureADPasswordProtectionProxy
ServerFQDN            : bplProxy.bplchild2.bplRootDomain.com
Domain                : bplchild2.bplRootDomain.com
Forest                : bplRootDomain.com
HeartbeatUTC          : 12/25/2018 6:35:02 AM
```

Cada servicio de proxy actualiza las diversas propiedades una vez cada hora aproximadamente. Los datos siguen estando sujetos a la latencia de replicación de Active Directory.

Se puede influenciar el ámbito de la consulta del cmdlet con los parámetros -Forest o -Domain.

Si el valor HeartbeatUTC se queda obsoleto, puede ser indicativo de que el proxy de la protección con contraseña de Azure AD de dicha máquina no está en ejecución o se ha desinstalado.

## <a name="proxy-agent-newer-version-available"></a>Versión más reciente del agente proxy disponible

El servicio Proxy registrará un evento de advertencia 20002 en el registro operativo cuando detecte que hay disponible una versión más reciente del software del proxy, por ejemplo:

```text
An update for Azure AD Password Protection Proxy is available.

If autoupgrade is enabled, this message may be ignored.

If autoupgrade is disabled, refer to the following link for the latest version available:

https://aka.ms/AzureADPasswordProtectionAgentSoftwareVersions

Current version: 1.2.116.0
.
```

El evento anterior no especifica la versión del software más reciente. Debe ir al vínculo en el mensaje de evento para esa información.

Este evento se emitirá incluso si el agente proxy está configurado con la opción de actualización automática habilitada.

## <a name="next-steps"></a>Pasos siguientes

[Solución de problemas de la protección con contraseña de Azure AD](howto-password-ban-bad-on-premises-troubleshoot.md)

Para más información sobre las listas de contraseñas prohibidas globales y personalizadas, consulte el artículo [Ban bad passwords](concept-password-ban-bad.md) (Prohibición de contraseñas incorrectas).
