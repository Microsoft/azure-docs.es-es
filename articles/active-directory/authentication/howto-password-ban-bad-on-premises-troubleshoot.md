---
title: Solución de problemas de la protección con contraseña de Azure AD local
description: Aprenda a solucionar problemas de la protección con contraseña de Azure AD para un entorno de Active Directory Domain Services local
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 79ebf543a3880a4f2c8ee8c0d706c268ef3f08d2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87035492"
---
# <a name="troubleshoot-on-premises-azure-ad-password-protection"></a>Solución de problemas: Protección con contraseña de Azure AD local

Después de la implementación de Protección de contraseña de Azure AD, puede ser necesario solucionar problemas. Este artículo entra en detalles para ayudarle a entender algunos pasos de solución de problemas comunes.

## <a name="the-dc-agent-cannot-locate-a-proxy-in-the-directory"></a>El agente de controlador de dominio no puede localizar un proxy en el directorio

El síntoma principal de este problema son eventos 30017 en el registro de eventos de administración del agente de controlador de dominio.

La causa habitual de este problema es que no se ha registrado aún un proxy. Si se ha registrado un proxy, puede haber algún retraso debido a la latencia de replicación de AD hasta que un agente de controlador de dominio determinado pueda ver el proxy.

## <a name="the-dc-agent-is-not-able-to-communicate-with-a-proxy"></a>El agente de controlador de dominio no es capaz de comunicarse con un proxy

El síntoma principal de este problema son eventos 30018 en el registro de eventos de administración del agente de controlador de dominio. Este problema puede deberse a varias causas:

1. El agente de controlador de dominio se encuentra en una parte aislada de la red que no permite la conectividad de red con el proxy registrado. Este problema puede ser inofensivo siempre que otros agentes de controlador de dominio se puedan comunicar con el proxy para descargar las directivas de contraseña de Azure. Una vez descargadas, el controlador de dominio aislado obtendrá dichas directivas a través de la replicación de los archivos de las directivas en el recurso compartido Sysvol.

1. El equipo host de proxy está bloqueando el acceso para el punto de conexión del mapeador del punto de conexión de RPC (puerto 135)

   El instalador de proxy de protección de contraseña de Azure AD crea automáticamente una regla de entrada de firewall de Windows que permite el acceso al puerto 135. Si más adelante se elimina o se deshabilita esta regla, los agentes de controlador de dominio no pueden comunicarse con el servicio de proxy. Si se ha deshabilitado el firewall de Windows integrado en lugar de otro producto de firewall, debe configurar el firewall para permitir el acceso al puerto 135.

1. La máquina host de proxy está bloqueando el acceso al punto de conexión de RPC (dinámico o estático) que escucha el servicio de proxy

   El instalador de proxy de protección de contraseña de Azure AD crea automáticamente una regla de entrada de firewall de Windows que permite el acceso a cualquier puerto de entrada que escuche el servicio de proxy de Protección con contraseña de Azure AD. Si más adelante se elimina o se deshabilita esta regla, los agentes de controlador de dominio no pueden comunicarse con el servicio de proxy. Si se ha deshabilitado el firewall de Windows integrado en lugar de otro producto de firewall, debe configurar el firewall para permitir el acceso a cualquier puerto de entrada que escuche el servicio de proxy de Protección con contraseña de Azure AD. Esta configuración puede ser más específica si se ha configurado el servicio de proxy para que escuche un puerto estático de RPC específico (mediante el cmdlet `Set-AzureADPasswordProtectionProxyConfiguration`).

1. La máquina host del proxy no está configurada para permitir que los controladores de dominio puedan iniciar sesión en la máquina. Este comportamiento se controla a través de la asignación del privilegio de usuario "Tener acceso a este equipo desde la red". Se debe conceder este privilegio a todos los controladores de dominio de todos los dominios del bosque. Esta configuración suele estar restringida como parte de un esfuerzo más amplio de protección de la red.

## <a name="proxy-service-is-unable-to-communicate-with-azure"></a>El servicio proxy no se puede comunicar con Azure

1. Asegúrese de que la máquina de proxy dispone de conectividad con los puntos de conexión que aparecen en los [requisitos de implementación](howto-password-ban-bad-on-premises-deploy.md).

1. Asegúrese de que el bosque y todos los servidores proxy estén registrados en el mismo inquilino de Azure.

   Para comprobar este requisito, puede ejecutar los cmdlets de PowerShell `Get-AzureADPasswordProtectionProxy` u `Get-AzureADPasswordProtectionDCAgent` y, luego, comparar la propiedad `AzureTenant` de cada elemento devuelto. Para un funcionamiento correcto, el nombre de inquilino declarado debe ser el mismo dentro de todos los agentes de controlador de dominio y servidores proxy.

   Si existe una condición de error de coincidencia del registro de inquilino de Azure, este problema puede corregirse ejecutando los cmdlets de PowerShell `Register-AzureADPasswordProtectionProxy` o `Register-AzureADPasswordProtectionForest` según sea necesario y asegurándose de usar credenciales desde el mismo inquilino de Azure para todos los registros.

## <a name="dc-agent-is-unable-to-encrypt-or-decrypt-password-policy-files"></a>El agente de controlador de dominio no puede cifrar ni descifrar los archivos de directivas de contraseña

Protección con contraseña de Azure AD tiene una dependencia crítica en la funcionalidad de cifrado y descifrado proporcionada por el servicio de distribución de claves de Microsoft. Los errores de cifrado o descifrado pueden manifestar varios síntomas y tener varias causas posibles.

1. Asegúrese de que el servicio KDS esté habilitado y funcione en todos controladores de dominio de Windows Server 2012 y versiones posteriores en un dominio.

   De forma predeterminada, el modo de inicio de servicio del servicio KDS está configurado como Manual (inicio de desencadenador). Esta configuración significa que la primera vez que un cliente intenta usar el servicio, se inicia a petición. Este modo de inicio del servicio predeterminado es aceptable para que Protección de contraseñas de Azure AD funcione.

   Si el modo de inicio del servicio KDS se ha configurado como Deshabilitado, esta configuración debe corregirse para que Protección de contraseñas de Azure AD funcione correctamente.

   Una prueba sencilla para este problema consiste en iniciar manualmente el servicio KDS, ya sea a través de la consola MMC de administración de servicios, o con otras herramientas de administración (por ejemplo, ejecute "net start kdssvc" desde una consola de línea de comandos). Se espera que el servicio KDS se inicie correctamente y permanezca en ejecución.

   La causa más común de que el servicio KDS no se pueda iniciar es que el objeto de controlador de dominio de Active Directory se encuentra fuera de la unidad organizativa controladores de dominio predeterminada. Esta configuración no es compatible con el servicio KDS y no es una limitación impuesta por Protección con contraseña de Azure AD. La solución para esta condición consiste en mover el objeto del controlador de dominio a una ubicación en la unidad organizativa de controladores de dominio predeterminada.

1. El cambio de formato del búfer cifrado de KDS no es compatible de Windows Server 2012 R2 a Windows Server 2016.

   En Windows Server 2016 se presentó una corrección de seguridad KDS que modifica el formato de los búferes cifrados de KDS; en ocasiones, estos búferes no se descifrarán en Windows Server 2012 y Windows Server 2012 R2. La dirección inversa es correcta: los búferes que están cifrados con KDS en Windows Server 2012 y Windows Server 2012 R2 siempre se descifrarán correctamente en Windows Server 2016 y versiones posteriores. Si los controladores de dominio de los dominios de Active Directory ejecutan una combinación de estos sistemas operativos, es posible que se notifiquen los errores ocasionales del descifrado de la protección con contraseña de Azure AD. No es posible predecir con precisión el tiempo o los síntomas de estos errores debido a la naturaleza de la corrección de seguridad, ya que no es determinista qué agente del controlador de dominio de protección con contraseña de Azure AD cifrará los datos en un momento dado.

   Microsoft está investigando una corrección para este problema, pero no hay ninguna fecha estimada disponible todavía. Mientras tanto, no hay ninguna solución alternativa para este problema que no sea ejecutar una combinación de estos sistemas operativos incompatibles en los dominios de Active Directory. En otras palabras, solo debe ejecutar los controladores de dominio de Windows Server 2012 y Windows Server 2012 R2, o solo ejecutar los controladores de dominio de Windows Server 2016 y versiones posteriores.

## <a name="weak-passwords-are-being-accepted-but-should-not-be"></a>Las contraseñas débiles se aceptan, pero no debería ser así

Este problema puede tener varias causas.

1. Los agentes de controlador de dominio ejecutan una versión de software de versión preliminar pública que ha expirado. Consulte [El software de los agentes de controlador de dominio de versión preliminar pública expiró](howto-password-ban-bad-on-premises-troubleshoot.md#public-preview-dc-agent-software-has-expired).

1. Sus agentes de controlador de dominio no pueden descargar una directiva o no pueden descifrar las directivas existentes. Busque posibles causas en los temas anteriores.

1. El modo Exigir de la directiva de contraseñas sigue establecido en Auditoría. Si esta configuración está aplicada, vuelva a configurarla en Exigir con el portal de Protección de contraseña de Azure AD. Para más información, consulte [Modos de funcionamiento](howto-password-ban-bad-on-premises-operations.md#modes-of-operation).

1. Se ha deshabilitado la directiva de contraseñas. Si esta configuración está aplicada, vuelva a configurarla en Habilitada con el portal de Protección de contraseña de Azure AD. Para más información, consulte [Modos de funcionamiento](howto-password-ban-bad-on-premises-operations.md#modes-of-operation).

1. No ha instalado el software del agente de controlador de dominio en todos los controladores de dominio en el dominio. En esta situación, es difícil garantizar que los clientes remotos de Windows se dirijan a un controlador de dominio determinado durante una operación de cambio de contraseña. Si cree que se ha dirigido correctamente un controlador de dominio determinado en el que está instalado el software del agente de controlador de dominio, puede asegurarse de ello volviendo a comprobar el registro de eventos de administración del agente de controlador de dominio; independientemente del resultado, habrá al menos un evento para documentar el resultado de la validación de contraseña. Si no hay ningún evento presente para el usuario para el que se ha cambiado la contraseña, es probable que el cambio de contraseña lo procese otro controlador de dominio.

   Como una prueba alternativa, intente configurar/cambiar contraseñas al iniciar sesión directamente en un controlador de dominio donde esté instalado el software del agente de controlador de dominio. Esta técnica no es recomendable para los dominios de Active Directory de producción.

   Aunque se admite la implementación incremental del software del agente de controlador de dominio sujeta a estas limitaciones, Microsoft recomienda encarecidamente que se instale el software del agente de controlador de dominio en todos los controladores de dominio en un dominio tan pronto como sea posible.

1. Puede que el algoritmo de validación de contraseña funcione realmente según lo esperado. Consulte [Cómo se evalúan las contraseñas](concept-password-ban-bad.md#how-are-passwords-evaluated).

## <a name="ntdsutilexe-fails-to-set-a-weak-dsrm-password"></a>Ntdsutil.exe no puede establecer una contraseña de DSRM no segura

Active Directory validará siempre una nueva contraseña del modo de reparación de servicios de directorio para asegurarse de que cumple los requisitos de complejidad de la contraseña del dominio; esta validación también llama a los archivos dll del filtro de contraseñas, como protección de contraseña de Azure AD. Si se rechaza la nueva contraseña de DSRM, se genera el siguiente mensaje de error:

```text
C:\>ntdsutil.exe
ntdsutil: set dsrm password
Reset DSRM Administrator Password: reset password on server null
Please type password for DS Restore Mode Administrator Account: ********
Please confirm new password: ********
Setting password failed.
        WIN32 Error Code: 0xa91
        Error Message: Password doesn't meet the requirements of the filter dll's
```

Cuando protección de contraseña de Azure AD registra los eventos del registro de eventos de validación de contraseñas para una contraseña de DSRM de Active Directory, se espera que los mensajes del registro de eventos no incluyan un nombre de usuario. Este comportamiento se produce porque la cuenta de DSRM es una cuenta local que no forma parte del dominio de Active Directory real.  

## <a name="domain-controller-replica-promotion-fails-because-of-a-weak-dsrm-password"></a>No se puede realizar la promoción de la réplica del controlador de dominio debido a una contraseña de DSRM no segura

Durante el proceso de promoción del controlador de dominio, la nueva contraseña del modo de reparación de servicios de directorio se enviará a un controlador de dominio existente en el dominio para la validación. Si se rechaza la nueva contraseña de DSRM, se genera el siguiente mensaje de error:

```powershell
Install-ADDSDomainController : Verification of prerequisites for Domain Controller promotion failed. The Directory Services Restore Mode password does not meet a requirement of the password filter(s). Supply a suitable password.
```

Al igual que en el problema anterior, cualquier evento de resultado de validación de la contraseña de protección de contraseña de Azure AD tendrá nombres de usuario vacíos para este escenario.

## <a name="domain-controller-demotion-fails-due-to-a-weak-local-administrator-password"></a>Se produce un error de degradación del controlador de dominio debido a una contraseña de administrador local no segura

Se admite la degradación de un controlador de dominio en el que todavía se ejecuta el software del agente de controlador de dominio. No obstante, los administradores deben tener en cuenta que el software del agente de controlador de dominio sigue aplicando la directiva de contraseñas actual durante el procedimiento de degradación. La nueva contraseña de la cuenta de administrador local (que se especifica como parte de la operación de degradación) se valida como cualquier otra contraseña. Microsoft recomienda que se elijan contraseñas seguras para las cuentas de administrador local como parte de un procedimiento de degradación del controlador de dominio.

Una vez que se ha realizado correctamente la degradación y el controlador de dominio se ha reiniciado y está de nuevo en ejecución como un servidor miembro normal, el software del agente de controlador de dominio se vuelve a ejecutar en modo pasivo. A partir de ese momento se puede desinstalar en cualquier momento.

## <a name="booting-into-directory-services-repair-mode"></a>Arranque en el modo de reparación de servicios de directorio

Si el controlador de dominio se inicia en el modo de reparación de servicios de directorio, la DLL de filtro de contraseña del agente de controlador de dominio detecta esta anomalía y hará que se deshabiliten todas las actividades de validación de contraseñas o de aplicación de estas, independientemente de la configuración de directivas activa actualmente. La DLL de filtro de contraseña del agente de controlador de dominio registrará un evento de advertencia 10023 en el registro de eventos de administración, por ejemplo:

```text
The password filter dll is loaded but the machine appears to be a domain controller that has been booted into Directory Services Repair Mode. All password change and set requests will be automatically approved. No further messages will be logged until after the next reboot.
```
## <a name="public-preview-dc-agent-software-has-expired"></a>El software de los agentes de controlador de dominio de versión preliminar pública expiró

Durante el período de versión preliminar de Protección con contraseña de Azure AD, el software de agente de controlador de dominio estaba codificado de forma rígida para dejar de procesar las solicitudes de validación de contraseñas en las fechas siguientes:

* La versión 1.2.65.0 dejará de procesar las solicitudes de validación de contraseñas el 1 de septiembre de 2019.
* La versión 1.2.25.0 y versiones anteriores dejaron de procesar las solicitudes de validación de contraseñas el 1 de julio de 2019.

A medida que se aproxima la fecha límite, todas las versiones de agentes de controlador de dominio con límite de tiempo emitirán un evento 10021 en el registro de eventos de administración del agente de controlador de dominio durante el arranque que tienen un aspecto similar al siguiente:

```text
The password filter dll has successfully loaded and initialized.

The allowable trial period is nearing expiration. Once the trial period has expired, the password filter dll will no longer process passwords. Please contact Microsoft for an newer supported version of the software.

Expiration date:  9/01/2019 0:00:00 AM

This message will not be repeated until the next reboot.
```

Cuando se ha pasado la fecha límite, todas las versiones de agentes de controlador de dominio con límite de tiempo emitirán un evento 10022 en el registro de eventos de administración del agente de controlador de dominio durante el arranque que tienen un aspecto similar al siguiente:

```text
The password filter dll is loaded but the allowable trial period has expired. All password change and set requests will be automatically approved. Please contact Microsoft for a newer supported version of the software.

No further messages will be logged until after the next reboot.
```

Como la fecha límite solo se comprueba en el arranque inicial, es posible que no vea estos eventos hasta mucho después de la fecha límite. Una vez conocido el plazo, no se producirán efectos negativos ni en el controlador de dominio ni en el entorno más grande, salvo que todas las contraseñas se aprobarán de manera automática.

> [!IMPORTANT]
> Microsoft recomienda que los agentes de controlador de dominio de la versión preliminar pública expirados se actualicen de inmediato a la versión más reciente.

Una manera sencilla de detectar un agente de controlador de dominio en un entorno que se debe actualizar es mediante la ejecución del cmdlet `Get-AzureADPasswordProtectionDCAgent`, por ejemplo:

```powershell
PS C:\> Get-AzureADPasswordProtectionDCAgent

ServerFQDN            : bpl1.bpl.com
SoftwareVersion       : 1.2.125.0
Domain                : bpl.com
Forest                : bpl.com
PasswordPolicyDateUTC : 8/1/2019 9:18:05 PM
HeartbeatUTC          : 8/1/2019 10:00:00 PM
AzureTenant           : bpltest.onmicrosoft.com
```

En este tema, el campo SoftwareVersion obviamente es la propiedad de clave que se debe examinar. También puede usar el filtrado de PowerShell para filtrar los agentes de controlador de dominio que ya tienen la versión de línea de base necesaria o una versión superior, por ejemplo:

```powershell
PS C:\> $LatestAzureADPasswordProtectionVersion = "1.2.125.0"
PS C:\> Get-AzureADPasswordProtectionDCAgent | Where-Object {$_.SoftwareVersion -lt $LatestAzureADPasswordProtectionVersion}
```

Ninguna versión del software de proxy de Protección con contraseña de Azure AD tiene límite de tiempo. Microsoft sigue recomendando que tanto los agentes de controlador de dominio como los de proxy se actualicen a las versiones más recientes a medida que se lanzan. El cmdlet `Get-AzureADPasswordProtectionProxy` se puede usar para buscar los agentes de proxy que requieren actualizaciones, similar a lo que sucede en el ejemplo anterior para los agentes de controlador de dominio.

Consulte [Actualización del agente de controlador de dominio](howto-password-ban-bad-on-premises-deploy.md#upgrading-the-dc-agent) y [Actualización del servicio proxy](howto-password-ban-bad-on-premises-deploy.md#upgrading-the-proxy-service) para más detalles sobre procedimientos de actualización específicos.

## <a name="emergency-remediation"></a>Corrección de emergencia

Si se produce una situación en la que el servicio del agente de controlador de dominio causa problemas, este servicio se puede apagar inmediatamente. La DLL de filtro de contraseña del agente de controlador de dominio sigue intentando llamar al servicio que no está en ejecución, y registrará los eventos de advertencia (10012, 10013), pero durante ese tiempo se aceptan todas las contraseñas entrantes. El servicio de agente de controlador de dominio se puede también configurar mediante el Administrador de control de servicios de Windows con un tipo de inicio "Deshabilitado" según sea necesario.

Otra medida de corrección sería establecer el modo Habilitar en No en el portal de Protección de contraseña de Azure AD. Una vez que se ha descargado la directiva actualizada, cada servicio del agente de controlador de dominio entrará en un modo inactivo donde todas las contraseñas se aceptan tal cual. Para más información, consulte [Modos de funcionamiento](howto-password-ban-bad-on-premises-operations.md#modes-of-operation).

## <a name="removal"></a>Eliminación

Realice los pasos siguientes si decide desinstalar el software de Protección de contraseñas de Azure AD y borrar todos los estados relacionados de los dominios y el bosque:

> [!IMPORTANT]
> Es importante que realice estos pasos en orden. Si se deja cualquier instancia del servicio de proxy en ejecución, esta volverá a crear periódicamente su objeto serviceConnectionPoint. Si se deja cualquier instancia del servicio del agente de controlador de dominio en ejecución, esta volverá a crear periódicamente su objeto serviceConnectionPoint y el estado sysvol.

1. Desinstale el software del proxy de todas las máquinas. En este paso **no** es necesario reiniciar.
2. Desinstale el software del agente de controlador de dominio de todos los controladores de dominio. En este paso es **necesario** reiniciar.
3. Quite manualmente todos los puntos de conexión del servicio de proxy en cada contexto de nomenclatura de dominio. La ubicación de estos objetos se puede detectar con el siguiente comando de Active Directory PowerShell:

   ```powershell
   $scp = "serviceConnectionPoint"
   $keywords = "{ebefb703-6113-413d-9167-9f8dd4d24468}*"
   Get-ADObject -SearchScope Subtree -Filter { objectClass -eq $scp -and keywords -like $keywords }
   ```

   No omita el asterisco ("*") al final del valor de variable $keywords.

   El objeto u objetos resultantes que se encuentran a través del comando `Get-ADObject`, se pueden canalizar hacia `Remove-ADObject`, o eliminar manualmente.

4. Quite manualmente todos los puntos de conexión del agente de controlador de dominio de cada contexto de nomenclatura de dominio. Puede haber uno estos objetos por cada controlador de dominio del bosque, dependiendo del grado de extensión de la implementación del software. La ubicación de ese objeto se puede detectar con el siguiente comando de Active Directory PowerShell:

   ```powershell
   $scp = "serviceConnectionPoint"
   $keywords = "{2bac71e6-a293-4d5b-ba3b-50b995237946}*"
   Get-ADObject -SearchScope Subtree -Filter { objectClass -eq $scp -and keywords -like $keywords }
   ```

   El objeto u objetos resultantes que se encuentran a través del comando `Get-ADObject`, se pueden canalizar hacia `Remove-ADObject`, o eliminar manualmente.

   No omita el asterisco ("*") al final del valor de variable $keywords.

5. Quite manualmente el estado de configuración en el nivel de bosque. El estado de configuración del bosque se conserva en un contenedor en el contexto de nomenclatura de configuración de Active Directory. Se puede detectar y eliminar de la siguiente manera:

   ```powershell
   $passwordProtectionConfigContainer = "CN=Azure AD Password Protection,CN=Services," + (Get-ADRootDSE).configurationNamingContext
   Remove-ADObject -Recursive $passwordProtectionConfigContainer
   ```

6. Quite todos los estados relacionados con Sysvol mediante la eliminación manual de la siguiente carpeta y de todo su contenido:

   `\\<domain>\sysvol\<domain fqdn>\AzureADPasswordProtection`

   Si es necesario, también se puede acceder localmente a esta ruta en un determinado controlador de dominio. La ubicación predeterminada debe ser parecida a la siguiente:

   `%windir%\sysvol\domain\Policies\AzureADPasswordProtection`

   Esta ruta de acceso será diferente si se ha configurado el recurso compartido de Sysvol en una ubicación distinta a la predeterminada.

## <a name="next-steps"></a>Pasos siguientes

[Preguntas más frecuentes de la protección con contraseña de Azure AD](howto-password-ban-bad-on-premises-faq.md)

Para más información sobre las listas de contraseñas prohibidas globales y personalizadas, consulte el artículo [Ban bad passwords](concept-password-ban-bad.md) (Prohibición de contraseñas incorrectas).
