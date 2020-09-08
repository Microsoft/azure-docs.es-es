---
title: 'Azure AD Connect: Historial de lanzamiento de versiones | Microsoft Docs'
description: En este artículo se muestran todas las versiones de Azure AD Connect y Sincronización de Azure AD
services: active-directory
author: billmath
manager: daveba
ms.assetid: ef2797d7-d440-4a9a-a648-db32ad137494
ms.service: active-directory
ms.topic: reference
ms.workload: identity
ms.date: 08/07/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d790c7c4c8d184999ff073e97613d663988c34e7
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89275953"
---
# <a name="azure-ad-connect-version-release-history"></a>Azure AD Connect: Historial de lanzamiento de versiones
El equipo de Azure Active Directory (Azure AD) actualiza periódicamente Azure AD Connect con nuevas características y funcionalidades. No todas las adiciones son aplicables a todas las audiencias.

Este artículo está diseñado para ayudarle a realizar un seguimiento de las versiones que se han publicado y para reconocer cuáles son los cambios en la última versión.



Esta tabla es una lista de temas relacionados:

Tema |  Detalles
--------- | --------- |
Pasos para actualizar desde Azure AD Connect | Diferentes métodos para [actualizar desde una versión anterior a la última](how-to-upgrade-previous-version.md) versión de Azure AD Connect.
Permisos necesarios | Para más información sobre los permisos necesarios para aplicar una actualización, consulte [cuentas y permisos](reference-connect-accounts-permissions.md#upgrade).
Descargar| [Descarga de Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771).

>[!NOTE]
>Lanzar una versión nueva de Azure AD Connect es un proceso que requiere varios pasos de control de calidad para garantizar la funcionalidad operativa del servicio y, mientras realizamos este proceso, tanto el número de cada versión nueva como su estado se actualizan para reflejar los cambios más recientes.
Mientras realizamos este proceso, el número de la versión se mostrará con una "X" en la posición del número de versión secundaria; por ejemplo, "1.3.X.0". Esto indica que las notas de la versión de este documento son válidas para todas las versiones que empiezan por "1.3". En cuanto finalice el proceso de lanzamiento, se actualizará el número de versión a la versión publicada más reciente y el estado de la versión se actualizará a "publicado para descarga y actualización automática".
No todas las versiones de Azure AD Connect estarán disponibles para la actualización automática. El estado de lanzamiento indicará si una versión está disponible para la actualización automática o solo para la descarga. Si la actualización automática estaba habilitada en el servidor de Azure AD Connect, dicho servidor se actualizará automáticamente a la versión más reciente de Azure AD Connect que se lanza para la actualización automática. Tenga en cuenta que no todas las configuraciones de Azure AD Connect son aptas para la actualización automática. Siga este vínculo para más información acerca de la [actualización automática](how-to-connect-install-automatic-upgrade.md)

>[!IMPORTANT]
> A partir del 1 de noviembre de 2020, comenzaremos a implementar un proceso de retirada en el que las versiones de Azure AD Connect que se publicaron hace más de 18 meses dejarán de usarse. En ese momento se iniciará el proceso con la retirada de todas las versiones de Azure AD Connect 1.3.20.0 (publicada el 24/4/2019) y anteriores, y se pasará a evaluar la retirada de las versiones anteriores de Azure AD Connect cada vez que se publique una nueva versión.
>
> Debe asegurarse de que está ejecutando una versión reciente de Azure AD Connect para lograr una experiencia de soporte técnico óptima. 
>
>Si ejecuta una versión en desuso de Azure AD Connect es posible que no tenga las últimas correcciones de seguridad, mejoras de rendimiento, herramientas de diagnóstico y solución de problemas, y mejoras de servicio. Asimismo, si necesita soporte técnico, es posible que no podamos proporcionarle el nivel de servicio que requiere su organización.
>
>Si ha habilitado Azure AD Connect para la sincronización, pronto comenzará a recibir notificaciones de estado automáticamente, que le avisarán sobre las próximas retiradas si ejecuta una de las versiones anteriores.
>
>Consulte [este artículo](./how-to-upgrade-previous-version.md) para obtener más información sobre cómo actualizar Azure AD Connect a la versión más reciente.
>
>Para obtener información sobre el historial de versiones en desuso, consulte [Archivo del historial de lanzamiento de versiones de Azure AD Connect](reference-connect-version-history-archive.md).

## <a name="15450"></a>1.5.45.0

### <a name="release-status"></a>Estado de la versión
29/07/2020: publicado para descarga.

### <a name="functional-changes"></a>Cambios funcionales
Se trata de una versión de corrección de errores. No hay cambios funcionales en esta versión.

### <a name="fixed-issues"></a>Problemas corregidos

- Se ha corregido un problema por el que el administrador no podía habilitar el "Inicio de sesión único de conexión directa" si la cuenta de equipo AZUREADSSOACC ya está presente en la instancia de "Active Directory".
- Se ha corregido un problema que provocaba un error de almacenamiento provisional durante la importación diferencial de la API v2 para un objeto conflictivo que se ha reparado mediante el portal de mantenimiento.
- Se ha corregido un problema en la configuración de importación y exportación, en el que una regla personalizada deshabilitada se importaba como habilitada.

## <a name="15420"></a>1.5.42.0

### <a name="release-status"></a>Estado de la versión
07/10/2020: publicado para descarga.

### <a name="functional-changes"></a>Cambios funcionales
Esta versión incluye una versión preliminar pública de la funcionalidad para exportar la configuración de un servidor de Azure AD Connect existente a un archivo .JSON que se puede usar al instalar un nuevo servidor de Azure AD Connect para crear una copia del servidor original.

Puede encontrar una descripción detallada de esta nueva característica en [este artículo](./how-to-connect-import-export-config.md).

### <a name="fixed-issues"></a>Problemas corregidos
- Se corrigió un error por el que habría una advertencia falsa sobre el tamaño de la base de datos local en las compilaciones localizadas durante la actualización.
- Se corrigió un error por el que habría un error falso en los eventos de la aplicación para el cambio de nombre de cuenta/nombre de dominio.
- Se corrigió un error por el que Azure AD Connect no podía instalarse en un controlador de dominio, con el mensaje "no se encontró el miembro".


## <a name="15300"></a>1.5.30.0.

### <a name="release-status"></a>Estado de la versión
07/05/2020: publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos
Esta compilación de revisiones corrige un problema por el que los dominios no seleccionados se seleccionaban de forma incorrecta en la interfaz de usuario del asistente si solo se seleccionaban contenedores secundarios.


>[!NOTE]
>Esta versión incluye la nueva API de punto de conexión de sincronización de Azure AD Connect V2.  Actualmente, este nuevo punto de conexión V2 se encuentra en versión preliminar pública.  Se requiere esta versión o una posterior para usar la nueva API de punto de conexión V2.  Sin embargo, el punto de conexión V2 no se habilita simplemente mediante la instalación de esta versión. Seguirá usando el punto de conexión V1, a menos que habilite el punto de conexión V2.  Debe seguir los pasos descritos en [API de punto de conexión de sincronización de Azure AD Connect V2 (versión preliminar pública)](how-to-connect-sync-endpoint-api-v2.md) para habilitarlo y participar en la versión preliminar pública.  

## <a name="15290"></a>1.5.29.0

### <a name="release-status"></a>Estado de la versión
23/04/2020: publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos
Esta compilación de revisiones corrige un problema presentado en la compilación 1.5.20.0 en el que un administrador de inquilinos con MFA no podía habilitar DSSO.

## <a name="15220"></a>1.5.22.0

### <a name="release-status"></a>Estado de la versión
20/04/2020: publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos
Esta compilación de revisión corrige un problema en la compilación 1.5.20.0 si ha clonado la regla **In from AD - Group Join** pero no la regla **In from AD - Group Common**.

## <a name="15200"></a>1.5.20.0

### <a name="release-status"></a>Estado de la versión
09/04/2020: publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos
- Esta compilación de revisiones corrige un problema con la compilación 1.5.18.0 si tiene habilitada la característica de filtrado de grupos y usa mS-DS-ConsistencyGuid como delimitador de origen.
- Se corrigió un problema en el módulo de PowerShell de ADSyncConfig, en el que la invocación del comando DSACLS usado en todos los cmdlets de permisos Set-ADSync* daba lugar a uno de los siguientes errores:
     - `GrantAclsNoInheritance : The parameter is incorrect.   The command failed to complete successfully.`
     - `GrantAcls : No GUID Found for computer …`

> [!IMPORTANT]
> Si ha clonado la regla de sincronización **In from AD - Group Join** pero no la regla de sincronización **In from AD - Group Common** y planea actualizar, complete los pasos siguientes como parte de la actualización:
> 1. Durante la actualización, desactive la opción **Inicie el proceso de sincronización cuando se complete la configuración**.
> 2. Edite la regla de sincronización de combinación clonada y agregue las dos transformaciones siguientes:
>     - Establezca el valor de `objectGUID` del flujo directo en `sourceAnchorBinary`.
>     - Establezca el valor `ConvertToBase64([objectGUID])` del flujo de expresión en `sourceAnchor`.     
> 3. Habilite el programador mediante `Set-ADSyncScheduler -SyncCycleEnabled $true`.



## <a name="15180"></a>1.5.18.0

### <a name="release-status"></a>Estado de la versión
02/04/2020: publicado para descarga.

### <a name="functional-changes-adsyncautoupgrade"></a>ADSyncAutoUpgrade para cambios funcionales 

- Se ha agregado compatibilidad con la característica mS-DS-ConsistencyGuid para objetos de grupo. Esto le permite trasladar grupos entre bosques o volver a conectar grupos en AD a Azure AD donde el objectID del grupo de AD ha cambiado, por ejemplo, cuando se vuelve a compilar un servidor de AD después de un catástrofe. Para más información, vea [Mover grupos entre bosques](how-to-connect-migrate-groups.md).
- El atributo mS-DS-ConsistencyGuid se establece automáticamente en todos los grupos sincronizados y no es necesario hacer nada para habilitar esta característica. 
- Se quitó Get-ADSyncRunProfile porque ya no se usa. 
- Se cambió la advertencia que se ve al intentar usar una cuenta de administrador de organización o de administrador de dominio para que la cuenta del conector de AD DS proporcione más contexto. 
- Se agregó un nuevo cmdlet para quitar objetos del espacio del conector que se ha quitado de la herramienta CSDelete.exe anterior y se ha reemplazado por el nuevo cmdlet Remove-ADSyncCSObject. El cmdlet Remove-ADSyncCSObject adopta un CsObject como entrada. Este objeto se puede recuperar mediante el cmdlet Get-ADSyncCSObject.

>[!NOTE]
>La herramienta CSDelete.exe anterior se ha quitado y se ha reemplazado por el nuevo cmdlet Remove-ADSyncCSObject. 

### <a name="fixed-issues"></a>Problemas corregidos

- Se corrigió un error en el selector de bosque/unidad organizativa de escritura diferida de grupo al volver a ejecutar el Asistente para Azure AD Connect después de deshabilitar la característica. 
- Se presentó una nueva página de error que se mostrará si faltan los valores del registro DCOM necesarios con un nuevo vínculo de ayuda. La información también se escribe en los archivos de registro. 
- Se corrigió un problema con la creación de la cuenta de sincronización de Azure Active Directory en la que se puede producir un error al habilitar las extensiones de directorio o PHS porque la cuenta no se propagó en todas las réplicas de servicio antes de que se intentara usar. 
- Se corrigió un error en la utilidad de compresión de errores de sincronización que no estaba utilizando los caracteres suplentes correctamente. 
- Se corrigió un error en la actualización automática que dejó el servidor en el estado suspendido del programador. 

## <a name="14380"></a>1.4.38.0
### <a name="release-status"></a>Estado de la versión
9/12/2019: versión para descarga. No está disponible mediante la actualización automática.
### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
- Se ha actualizado la sincronización de hash de contraseña para Azure AD Domain Services con el fin de tener en cuenta correctamente el relleno en los hashes de Kerberos.  Esta actualización proporcionará una mejora del rendimiento durante la sincronización de contraseñas de Azure AD con Azure AD Domain Services.
- Se ha agregado compatibilidad con sesiones confiables entre el agente de autenticación y Service Bus.
- Esta versión exige TLS 1.2 para la comunicación entre el agente de autenticación y los servicios en la nube.
- Se ha agregado una caché DNS para las conexiones WebSocket entre el agente de autenticación y los servicios en la nube.
- Se ha agregado la capacidad de dirigirse a un agente específico desde la nube para probar la conectividad del agente.

### <a name="fixed-issues"></a>Problemas corregidos
- La versión 1.4.18.0 tenía un error en el que el cmdlet de PowerShell para DSSO utilizaba las credenciales de inicio de sesión de Windows en lugar de las credenciales de administrador proporcionadas durante la ejecución de PS. Como resultado, no era posible habilitar DSSO en varios bosques mediante la interfaz de usuario de Azure AD Connect. 
- Se ha realizado una corrección para habilitar DSSO simultáneamente en todos los bosques mediante la interfaz de usuario de Azure AD Connect.

## <a name="14320"></a>1.4.32.0
### <a name="release-status"></a>Estado de la versión
08/11/2019: publicado para descarga. No está disponible mediante la actualización automática.

>[!IMPORTANT]
>Debido a un cambio de esquema interno en esta versión de Azure AD Connect, si administra los valores de configuración de relación de confianza de AD FS mediante MSOnline PowerShell, debe actualizar el módulo de MSOnline PowerShell a la versión 1.1.183.57 o posterior.

### <a name="fixed-issues"></a>Problemas corregidos

Esta versión corrige un problema con los dispositivos unidos a Azure AD híbrido existentes. Contiene una nueva regla de sincronización de dispositivos que corrige este problema.
Tenga en cuenta que este cambio de regla puede hacer que se eliminen de Azure AD los dispositivos obsoletos. No debe preocuparse, ya que estos objetos de dispositivos no se usan en Azure AD durante la autorización de acceso condicional. Para algunos clientes, el número de dispositivos que se eliminarán mediante esta regla puede superar el umbral de eliminación. Si observa que la eliminación de objetos de dispositivo de Azure AD supera el umbral de eliminación de exportación, se aconseja permitir que las eliminaciones sigan adelante. [Procedimiento para permitir que fluyan las eliminaciones cuando superan el umbral de eliminación](how-to-connect-sync-feature-prevent-accidental-deletes.md)

## <a name="14250"></a>1.4.25.0

### <a name="release-status"></a>Estado de la versión
28/9/2019: Publicado para la actualización automática para seleccionar inquilinos. No disponible para descargar.

Esta versión corrige un error en el que algunos servidores que se actualizaron automáticamente de una versión anterior a 1.4.18.0 y experimentaron problemas con el autoservicio de restablecimiento de contraseña (SSPR) y la escritura diferida de contraseñas.

### <a name="fixed-issues"></a>Problemas corregidos

En determinadas circunstancias, los servidores que se actualizaron automáticamente a la versión 1.4.18.0 no volvieron a habilitar el autoservicio de restablecimiento de contraseña y la escritura diferida de contraseñas de autoservicio una vez completada la actualización. Esta versión de actualización automática corrige el problema y vuelve a habilitar el autoservicio de restablecimiento de contraseña y la escritura diferida de contraseñas.

Se corrigió un error en la utilidad de compresión de errores de sincronización que no estaba utilizando los caracteres suplentes correctamente.

## <a name="14180"></a>1.4.18.0

>[!WARNING]
>Estamos investigando un incidente por el que algunos clientes tienen incidencias con los dispositivos existentes unidos a Azure AD híbrido después de actualizar a esta versión de Azure AD Connect. Se aconseja a los clientes que han implementado la unión a Azure AD híbrido que pospongan la actualización a esta versión hasta que se comprenda y solucione definitivamente la causa principal de estas incidencias. Se proporcionará más información en cuanto sea posible.

>[!IMPORTANT]
>Con esta versión de Azure AD Connect, algunos clientes pueden ver que algunos o todos sus dispositivos Windows desaparecen de Azure AD. No hay motivo para preocuparse, ya que estas identidades de dispositivo no se usan en Azure AD durante la autorización de acceso condicional. Para obtener más información, consulte [Descripción de la desaparición del dispositivo Azure AD Connect 1.4.XX.x](reference-connect-device-disappearance.md)


### <a name="release-status"></a>Estado de la versión
25/9/2019: publicada solo para actualización automática.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
- La nueva herramienta de solución de problemas ayuda a solucionar escenarios de falta de sincronización de usuarios, grupos o miembros de grupos.
- Compatibilidad agregada con nubes nacionales en el script de solución de problemas de Azure AD Connect. 
- Se debe informar a los clientes de que ahora se han quitado los puntos de conexión WMI en desuso para MIIS_Service. Las operaciones de WMI ahora deben realizarse mediante cmdlets de PS.
- Mejora de la seguridad mediante el restablecimiento de la delegación restringida en el objeto AZUREADSSOACC.
- Al agregar o editar una regla de sincronización, si hay atributos que se usan en la regla que se encuentran en el esquema del conector pero no se agregan al conector, los atributos se agregan al conector. Lo mismo se aplica al tipo de objeto al que afecta la regla. Si se agrega algo al conector, este se marcará para la importación completa en el siguiente ciclo de sincronización.
- El uso de un administrador de organización o dominio como la cuenta del conector ya no se admite en las nuevas implementaciones de Azure AD Connect. Las implementaciones actuales de Azure AD Connect que usan un administrador de dominio o de empresa como cuenta de conector no se verán afectadas por esta versión.
- En el administrador de sincronización, se ejecuta una sincronización completa durante la creación, la edición y la eliminación de la regla. Con cualquier cambio de la regla aparecerá una ventana emergente que notifica al usuario si se va a ejecutar la importación completa o la sincronización completa.
- Se han agregado pasos para la mitigación de los errores de contraseña a la página "conectores > propiedades > conectividad".
- Se ha agregado una advertencia de desuso para el administrador de servicios de sincronización en la página de propiedades del conector. Esta advertencia informa al usuario de que deben realizarse cambios mediante el asistente para Azure AD Connect.
- Se ha agregado un nuevo error para problemas con la directiva de contraseñas de un usuario.
- Se impide el error de configuración del filtrado de grupos por dominio y filtros de unidad organizativa. El filtrado de grupos mostrará un error cuando el dominio o la unidad organizativa del grupo especificado ya se hayan filtrado y evitará que el usuario prosiga hasta que no se resuelva el problema.
- Los usuarios ya no pueden crear un conector para Active Directory Domain Services o Windows Azure Active Directory en la interfaz de usuario de Synchronization Service Manager.
- Se corrigió la accesibilidad de los controles de interfaz de usuario personalizados en Synchronization Service Manager.
- Se han habilitado seis tareas de administración de federación para todos los métodos de inicio de sesión en Azure AD Connect.  (Antes, solo estaba disponible la tarea “Actualización del certificado TLS/SSL de AD FS” en todos los inicios de sesión).
- Se ha agregado una advertencia al cambiar el método de inicio de sesión de federación a PHS o PTA que indica que todos los dominios de Azure AD y usuarios se convertirán a la autenticación administrada.
- Se han quitado los certificados de firma de tokens de la tarea "Restablecer Azure AD y confianza de AD FS" y se ha agregado una subtarea independiente para actualizar estos certificados.
- Se ha agregado una nueva tarea de administración de federación denominada “Administrar certificados” que tiene subtareas para actualizar los certificados de firma de tokens o TLS de la granja de servidores de AD FS.
- Se ha agregado una nueva subtarea de administración de federación llamada "Especificar servidor principal" que permite a los administradores especificar un nuevo servidor principal para la granja de servidores de AD FS.
- Se ha agregado una nueva tarea de administración de federación denominada "Administrar servidores" que tiene subtareas para implementar un servidor de AD FS, implementar un servidor proxy de aplicación web y especificar el servidor principal.
- Se ha agregado una nueva tarea de administración de federación denominada "Ver configuración de federación" que muestra la configuración actual de AD FS.  (Debido a esta incorporación, se ha quitado la configuración de AD FS de la página "Revisar su solución").

### <a name="fixed-issues"></a>Problemas corregidos
- Se ha resuelto un problema de error de sincronización en el escenario donde un objeto de usuario que asume su objeto de contacto correspondiente tiene una referencia automática (por ejemplo, el usuario es su propio administrador).
- Las ventanas emergentes de ayuda ahora se muestran en el foco del teclado.
- Para la actualización automática, si alguna aplicación en conflicto se ejecuta a partir de 6 horas, se finaliza y se sigue con la actualización.
- Limitación del número de atributos que un cliente puede seleccionar a 100 por objeto al seleccionar extensiones de directorio. Esto evitará que se produzca el error durante la exportación, ya que Azure tiene un máximo de 100 atributos de extensión por objeto.
- Se ha corregido un error para que el script de conectividad de AD sea más sólido.
- Se ha corregido un error para aportar una mayor solidez a la instalación de Azure AD Connect en una máquina mediante un servicio WCF de canalizaciones con nombre existente.
- Se han mejorado los diagnósticos y la solución de problemas de directivas de grupo que no permiten que el servicio ADSync se inicie cuando se instala inicialmente.
- Se ha corregido un error en el que el nombre para mostrar de un equipo Windows se escribía incorrectamente.
- Se ha corregido un error en el que el tipo de sistema operativo de un equipo Windows se escribía incorrectamente.
- Se ha corregido un error en el que los equipos que no son de Windows 10 se sincronizaban de forma inesperada. Observe que el efecto de este cambio es que ahora se eliminarán los equipos que no sean de Windows 10 que se hayan sincronizado previamente. Esto no afecta a ninguna característica, ya que la sincronización de equipos Windows solo se usa para la unión a un dominio de Azure AD híbrido, que solo funciona en dispositivos Windows 10.
- Se han agregaron varios cmdlets nuevos (internos) al módulo de ADSync PowerShell.


## <a name="13210"></a>1.3.21.0
>[!IMPORTANT]
>Hay un problema conocido con la actualización de Azure AD Connect desde una versión anterior a 1.3.21.0, donde el portal de Office 365 no refleja la versión actualizada, aunque Azure AD Connect se actualizara correctamente.
>
> Para resolver esto, deberá importar el módulo **AdSync** y ejecutar el cmdlet `Set-ADSyncDirSyncConfiguration` de PowerShell en el servidor de Azure AD Connect.  Puede usar los pasos siguientes:
>
>1. Abra PowerShell en modo de administrador.
>2. Ejecute `Import-Module "ADSync"`.
>3. Ejecute `Set-ADSyncDirSyncConfiguration -AnchorAttribute ""`.
 
### <a name="release-status"></a>Estado de la versión 

14/05/2019: publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos 

- Se ha corregido una vulnerabilidad de elevación de privilegios que existe en la compilación de Microsoft Azure Active Directory Connect 1.3.20.0.  Esta vulnerabilidad, bajo ciertas condiciones, puede permitir que un atacante ejecute dos cmdlets de PowerShell en el contexto de una cuenta con privilegios y realizar acciones con privilegios.  Esta actualización de seguridad inhabilita estos cmdlets para solucionar el problema. Para obtener más información, consulte la [actualización de seguridad](https://portal.msrc.microsoft.com/security-guidance/advisory/CVE-2019-1000).


## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).