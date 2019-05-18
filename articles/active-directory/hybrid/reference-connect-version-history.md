---
title: 'Azure AD Connect: Historial de lanzamiento de versiones | Microsoft Docs'
description: En este artículo se muestran todas las versiones de Azure AD Connect y Sincronización de Azure AD
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ef2797d7-d440-4a9a-a648-db32ad137494
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/14/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 60453c320a66a8eebd7460b3930241f9e81b8a1b
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784321"
---
# <a name="azure-ad-connect-version-release-history"></a>Azure AD Connect: Historial de lanzamiento de versiones
El equipo de Azure Active Directory (Azure AD) actualiza periódicamente Azure AD Connect con nuevas características y funcionalidades. No todas las adiciones son aplicables a todas las audiencias.


Este artículo está diseñado para ayudarle a realizar un seguimiento de las versiones que se han publicado y para reconocer cuáles son los cambios en la última versión.

Esta tabla es una lista de temas relacionados:

Tema |  Detalles
--------- | --------- |
Pasos para actualizar desde Azure AD Connect | Diferentes métodos para [actualizar desde una versión anterior a la última](how-to-upgrade-previous-version.md) versión de Azure AD Connect.
Permisos necesarios | Para más información sobre los permisos necesarios para aplicar una actualización, consulte [cuentas y permisos](reference-connect-accounts-permissions.md#upgrade).

Descarga | [Descargar Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771).

>[!NOTE]
>Lanzar una versión nueva de Azure AD Connect es un proceso que requiere varios paso de control de calidad para garantizar la funcionalidad de la operación del servicio y, mientras que se van a través de este proceso se actualizará el número de versión de una nueva versión, así como el estado de lanzamiento para reflejar el estado más reciente.
Aunque vamos a través de este proceso, el número de versión de la versión se mostrará con una "X" en la posición de número de versión secundaria, como se muestra en "1.3.X.0" - Esto indica que las notas de la versión de este documento son válidas para todas las versiones a partir de "1.3.". Tan pronto como se ha finalizado el proceso de lanzamiento se actualizará el número de versión de lanzamiento para la versión más reciente y se actualizará el estado de lanzamiento a "Emitido para la descarga y la actualización automática".
No todas las versiones de Azure AD Connect estarán disponibles para la actualización automática. El estado de lanzamiento indicará si una versión está disponible para la actualización automática o solo para la descarga. Si la actualización automática estaba habilitada en el servidor de Azure AD Connect, dicho servidor se actualizará automáticamente a la versión más reciente de Azure AD Connect que se lanza para la actualización automática. Tenga en cuenta que no todas las configuraciones de Azure AD Connect son aptas para la actualización automática. Siga este vínculo para más información acerca de la [actualización automática](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-install-automatic-upgrade)

## <a name="13210"></a>1.3.21.0

### <a name="release-status"></a>Estado de la versión 

05/14/2019: TBD


### <a name="fixed-issues"></a>Problemas corregidos 

- Se ha corregido una vulnerabilidad de elevación de privilegios que existe en la compilación de Microsoft Azure Active Directory Connect 1.3.20.0.  Esta vulnerabilidad, bajo ciertas condiciones, puede permitir que un atacante ejecute dos cmdlets de powershell en el contexto de una cuenta con privilegios y realizar acciones con privilegios.  Actualización de seguridad soluciona el problema al deshabilitar estos cmdlets. Para obtener más información, consulte [actualización de seguridad](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2019-1000).

## <a name="13200"></a>1.3.20.0 

### <a name="release-status"></a>Estado de la versión 

04/24/2019: publicado para descarga.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras 

- Agregar compatibilidad para la actualización de dominio 
- La característica carpetas públicas de correo de Exchange deja de GA 
- Mejorar el Asistente para control de errores para errores de servicio 
- Agregó un vínculo advertencia para la interfaz de usuario anterior en la página de propiedades del conector. 
- La característica de reescritura de grupos unificados está ahora GA 
- Mensaje de error SSPR mejorada cuando el controlador de dominio no tiene un control LDAP 
- Mejorar el diagnóstico de errores del registro DCOM durante la instalación  
- Seguimiento de errores de RPC PHS mejorado 
- Permitir credenciales EA desde un dominio secundario 
- Permitir que se especificó durante la instalación (nombre predeterminado ADSync) el nombre de base de datos
- Actualizar a ADAL 3.19.8 para recoger una corrección de WS-Trust de Ping y agregar compatibilidad para nuevas instancias de Azure 
- Las reglas de sincronización Modificar grupo de flujo de samAccountName, DomainNetbios y DomainFQDN en la nube: es necesario para las notificaciones 
- Modificar control de regla de sincronización predeterminado: más [aquí](how-to-connect-fix-default-rules.md).
- Agrega a un nuevo agente que se ejecuta como un servicio de windows. Este agente, llamado a "Agente de administración", permite un diagnóstico más detallado remoto del servidor de Azure AD Connect para solucionar Microsoft Engineers al abrir una incidencia de soporte técnico. Este agente no está instalado y habilitado de forma predeterminada.  Para obtener más información sobre cómo instalar y habilitar el agente consulte [¿qué es el agente Azure AD Connect Admin?](whatis-aadc-admin-agent.md). 
- Actualiza el contrato de licencia de usuario final (CLUF) 
- Agrega compatibilidad con la actualización para implementaciones que usan AD FS como su tipo de inicio de sesión automático.  Esto también quita el requisito de actualizar el AD FS Azure AD de confianza como parte del proceso de actualización. 
- Agrega una tarea de administración de confianza de Azure AD que proporciona dos opciones: analizar o actualizar la confianza y restablecer la confianza. 
- Puede cambiar el comportamiento de la confianza de AD FS Azure AD de confianza para que siempre use el modificador - SupportMultipleDomain (incluye el dominio de Azure AD y de confianza actualizaciones). 
- Puede cambiar el comportamiento de granja de servidores de instalación nuevo AD FS para que requiera un certificado pfx mediante la eliminación de la opción de usar un certificado instalado previamente.
- Actualiza el flujo de trabajo de granja de servidores de instalación de nuevo AD FS para que solo permite la implementación de ADFS de 1 y 1 servidor WAP.  Todos los servidores adicionales se realizará después de la instalación inicial. 

### <a name="fixed-issues"></a>Problemas corregidos 


- Corrección de SQL lógica para el servicio ADSync de reconexión 
- Corrección para permitir el uso de una base de datos de AOA de SQL vacía una instalación limpia 
- Corrija los permisos de PS script para restringir permisos GWB 
- Corregir errores VSS con LocalDB  
- Corregir el mensaje de error engañoso cuando el tipo de objeto no está en ámbito 
- Se ha corregido un problema donde la instalación de PowerShell de Azure AD en un servidor podría ocasionar un conflicto de ensamblado con Azure AD Connect. 
- Se corrigió el error PHS en el servidor de ensayo cuando se actualizan las credenciales del conector en la interfaz de usuario anterior. 
- Se ha corregido algunas pérdidas de memoria 
- Varias correcciones de Autoupgrade 
- Varias correcciones de exportación e importación sin confirmar de procesamiento 
- Se ha corregido un error con el control de una barra diagonal inversa en dominios y unidades Organizativas filtrado 
- Se ha corregido un problema cuando el servicio ADSync tarda más de 2 minutos para detener y provoca un problema durante la actualización. 




## <a name="12700"></a>1.2.70.0

### <a name="release-status"></a>Estado de la versión

18/12/2018 publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos

Esta compilación actualiza los conectores no estándar (por ejemplo, el conector LDAP genérico y el conector SQL genérico) enviados con Azure AD Connect. Para obtener más información sobre los conectores aplicables, consulte la versión 1.1.911.0 en [Connector Version Release History](/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-version-history) (Historial de versiones del conector).


## <a name="12690"></a>1.2.69.0

### <a name="release-status"></a>Estado de la versión
11/12/2018: publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos
Esta compilación de revisión permite al usuario seleccionar un dominio de destino, en el bosque especificado, para el contenedor RegisteredDevices al habilitar la escritura diferida de dispositivo.  En las versiones anteriores que contienen la nueva funcionalidad de opciones de dispositivo (1.1.819.0 - 1.2.68.0), la ubicación del contenedor RegisteredDevices estaba limitada a la raíz del bosque y no permitía dominios secundarios.  Esta limitación solo se manifestaba en nuevas implementaciones, las actualizaciones locales no se veían afectadas.  

Si se implementaba alguna compilación que contenía la funcionalidad de opciones de dispositivo en un nuevo servidor y se habilitaba la escritura diferida de dispositivo, tenía que especificar manualmente la ubicación del contenedor si no la quería en la raíz del bosque.  Para ello, era necesario deshabilitar la escritura diferida de dispositivo y volver a habilitarla, lo que le permitía especificar la ubicación del contenedor en la página "Bosque de escritura diferida".



## <a name="12680"></a>1.2.68.0

### <a name="release-status"></a>Estado de la versión 

30/11/2018  publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos

Esta compilación de revisión soluciona un conflicto donde podría producirse un error de autenticación debido a la presencia independiente del módulo Galería de PowerShell de MSOnline en el servidor de sincronización.



## <a name="12670"></a>1.2.67.0

### <a name="release-status"></a>Estado de la versión 

19/11/2018  publicado para descarga.

### <a name="fixed-issues"></a>Problemas corregidos

Esta compilación de revisiones corrige una regresión en la compilación anterior donde la escritura diferida de contraseñas generaba un error cuando se usaba un controlador de dominio ADDS en Windows Server 2008/R2.

## <a name="12650"></a>1.2.65.0 

### <a name="release-status"></a>Estado de la versión 

25/10/2018: publicado para descargar

 
### <a name="new-features-and-improvements"></a>Nuevas características y mejoras 


- Se ha cambiado la funcionalidad de reescritura de atributos para asegurar que el correo de voz alojado funciona como se esperaba.  En determinados escenarios, Azure AD sobrescribía el atributo msExchUcVoicemailSettings durante la reescritura con un valor nulo.  Azure AD ya no borrará el valor local de este atributo si el valor de la nube no está establecido.
- Se agregaron diagnósticos en el asistente de Azure AD Connect para investigar e identificar problemas de conectividad con Azure AD. Estos diagnósticos también se pueden ejecutar directamente a través de Powershell con el cmdlet Test- AdSyncAzureServiceConnectivity. 
- Se agregaron diagnósticos en el asistente de Azure AD Connect para investigar e identificar problemas de conectividad con AD. Estos mismos diagnósticos también se pueden ejecutar directamente a través de Powershell con la función Start-ConnectivityValidation en el módulo ADConnectivityTools de Powershell.  Para obtener más información, vea [¿Qué es el módulo de PowerShell ADConnectivityTool?](how-to-connect-adconnectivitytools.md)
- Se agregó una comprobación previa de la versión del esquema de AD para Hybrid Azure Active Directory Join y la reescritura de dispositivos 
- Se cambió la búsqueda de atributos de la página Extensión de directorio para que no distinga mayúsculas de minúsculas.
-   Se ha agregado compatibilidad total con TLS 1.2. Esta versión admite que todos los demás protocolos se deshabiliten y que solo TLS 1.2 se habilite en la máquina donde está instalado Azure AD Connect.  Para obtener más información, consulte [TLS 1.2 enforcement for Azure AD Connect](reference-connect-tls-enforcement.md) (Aplicación de TLS 1.2 para Azure AD Connect)

 

### <a name="fixed-issues"></a>Problemas corregidos   

- Se ha corregido un error donde la actualización de Azure AD Connect produciría un error si se estaba usando SQL Always On. 
- Se ha corregido un error para poder analizar correctamente los nombres de UO que contenían una barra diagonal. 
- Se ha solucionado un problema por el que la Autenticación de paso a través se deshabilitaba al realizar una instalación limpia en el modo de ensayo. 
- Se ha corregido un error que impedía que el módulo de PowerShell se cargara al ejecutar las herramientas de solución de problemas 
- Se ha corregido un error que impedía que los clientes usaran valores numéricos en el primer carácter de un nombre de host. 
- Se ha corregido un error que hacía que Azure AD Connect permitiera particiones y selecciones de contenedor no válidas. 
- Se ha corregido el mensaje de error "Contraseña no válida" cuando está habilitado SSO de escritorio. 
- Se han corregido varios errores de Administración de confianza de AD FS  
- Al configurar la escritura diferida de dispositivo: se ha corregido la comprobación de esquema para buscar la clase de objeto msDs-DeviceContainer (introducida en WS2012 R2)

 
## <a name="118820"></a>1.1.882.0  

7/9/2018: Publicado para descarga, no será la versión de actualización automática. 

### <a name="fixed-issues"></a>Problemas corregidos  

La actualización de Azure AD Connect genera un error si la disponibilidad AlwaysOn de SQL está configurada para la base de datos de ADSync. Esta revisión soluciona este problema y permite que la actualización se realice correctamente. 

## <a name="118800"></a>1.1.880.0

### <a name="release-status"></a>Estado de la versión

21/8/2018 publicado para descarga y actualización automática. 

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

- La integración de PingFederate en Azure AD Connect ya tiene disponibilidad general. [Más información sobre cómo realizar la federación de Azure AD con PingFederate](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-user-signin#federation-with-pingfederate)
- Ahora, Azure AD Connect crea la copia de seguridad de confianza de Azure AD en AD FS cada vez que se realiza una actualización, y la almacena en un archivo independiente para una sencilla restauración, si esta fuese necesaria. [Obtener más información acerca de la nueva funcionalidad y Azure AD confía en la administración en Azure AD Connect](https://aka.ms/fedtrustinaadconnect).
- Las nuevas herramientas de solución de problemas le ayudan a resolver los problemas relacionados con el cambio de dirección de correo electrónico principal y la ocultación de la cuenta de la lista global de direcciones
- Azure AD Connect se ha actualizado para incluir al cliente nativo más reciente de SQL Server 2012
- Al cambiar del inicio de sesión de usuario a la sincronización de hash de contraseña o la autenticación de paso a través, en la tarea "Cambio de inicio de sesión de usuario", la casilla de verificación Inicio de sesión único de conexión directa está habilitada de forma predeterminada.
- Se ha agregado compatibilidad para Windows Server Essentials 2019
- El agente de Azure AD Connect Health se ha actualizado a la versión más reciente 3.1.7.0
- Durante una actualización, si el instalador detecta cambios en las reglas de sincronización predeterminadas, el administrador recibe una advertencia antes de sobrescribir las reglas modificadas. Esto permitirá al usuario llevar a cabo acciones correctivas y continuar más adelante. Comportamiento antiguo: si había alguna regla integrada modificada, la actualización manual sobrescribía esas reglas sin proporcionar ningún mensaje de advertencia al usuario, y el programador de sincronización se deshabilitaba sin informar al usuario. Nuevo comportamiento: se proporciona al usuario una advertencia antes de sobrescribir las reglas de sincronización integradas modificadas. El usuario tendrá la opción de detener el proceso de actualización y reanudarlo más adelante, tras llevar a cabo una acción correctiva.
- Se proporciona un mejor control de los problemas de cumplimiento de la norma FIPS, proporcionándose un mensaje de error para la generación de hash MD5 en un entorno compatible con FIPS, y un vínculo a documentación que proporciona una solución alternativa para este problema.
- Actualización de la interfaz de usuario para mejorar las tareas de federación en el asistente, que ahora están en un subgrupo independiente para la federación. 
- Todas las tareas adicionales de la federación se agrupan ahora bajo un submenú único para facilitar su uso.
- Un nuevo y renovado módulo Posh de ADSyncConfig (AdSyncConfig.psm1) con nuevas funciones de los permisos de AD que se han movido desde el antiguo ADSyncPrep.psm1 (que posiblemente se deje de usar en breve)

### <a name="fixed-issues"></a>Problemas corregidos 

- Se ha corregido un error en el servidor de AAD Connect mostraría uso elevado de CPU después de actualizar a .NET 4.7.2
- Se ha corregido un error que de forma intermitente producía un mensaje de error para un problema de interbloqueo de SQL de resolución automática
- Se han corregido varios problemas de accesibilidad del editor de reglas de sincronización y el servicio de administrador de sincronización  
- Se ha corregido un error en Azure AD Connect por el que no se podía obtener información de configuración del registro
- Se ha corregido un error que creaba problemas cuando el usuario va hacia delante y hacia atrás en el asistente
- Se ha corregido un error para evitar un error que se produce debido a incorrecto multiproceso detenida en el Asistente
- Cuando la página Group Sync Filtering (filtros de sincronización de grupos) encuentra un error LDAP al resolver los grupos de seguridad, Azure AD Connect devuelve ahora la excepción con fidelidad total.  La causa raíz de la excepción de referencia aún no se conoce y se tratará en un error diferente.
-  Se ha corregido un error en el que los permisos para las claves STK y NGC (atributo ms-DS-KeyCredentialLink en objetos de usuario o dispositivo para WHfB) no se establecían correctamente.     
- Se ha corregido un error en el que no se llamaba correctamente a "Set-ADSyncRestrictedPermissions"
-  Se ha agregado compatibilidad para el permiso que se concede para la escritura diferida de grupos en el asistente para la instalación de AADConnect
- Al cambiar el método de inicio de sesión de sincronización de hash de contraseña a AD FS, no se deshabilitaba la sincronización de hash de contraseña.
- Se ha agregado la comprobación para las direcciones IPv6 en la configuración de AD FS
- Se ha actualizado el mensaje de notificación para informar de que ya existe una configuración.
- La escritura diferida de dispositivo no detecta el contenedor en el bosque sin confianza. Se ha actualizado para proporcionar un mensaje de error mejorado y un vínculo a la documentación adecuada
- La anulación de la selección de una unidad organizativa y, a continuación, la sincronización/escritura diferida que corresponde a esa unidad organizativa da un error genérico de sincronización. Se ha cambiado para crear un mensaje de error más comprensible.

## <a name="118190"></a>1.1.819.0

### <a name="release-status"></a>Estado de la versión

14/5/2018: publicado para actualización automática y descarga.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

Nuevas características y mejoras

- Esta versión incluye la versión preliminar pública de la integración de PingFederate en Azure AD Connect. Con esta versión, los clientes pueden configurar su entorno de Azure Active Directory de forma fácil y confiable para aprovechar PingFederate como proveedor de federación. Para obtener más información sobre cómo utilizar esta nueva característica, visite nuestra [documentación en línea](plan-connect-user-signin.md#federation-with-pingfederate). 
- Se ha actualizado la utilidad de solución de problemas del asistente de Azure AD Connect y ahora se analizan más escenarios de error, como buzones vinculados y grupos dinámicos de AD. Hay más información disponible sobre la utilidad de solución de problemas [aquí](tshoot-connect-objectsync.md).
- Ahora, la configuración de escritura diferida de dispositivo se administra únicamente desde el Asistente de Azure AD Connect.
- Se ha agregado un nuevo módulo de PowerShell llamado ADSyncTools.psm1 que puede utilizarse para solucionar problemas de conectividad de SQL y otras utilidades de solución de problemas. Hay más información disponible sobre el módulo ADSyncTools [aquí](tshoot-connect-tshoot-sql-connectivity.md). 
- Se ha agregado una nueva tarea adicional: Configurar opciones de dispositivo. Puede usar la tarea para configurar las dos operaciones siguientes: 
  - **Unión a Azure AD híbrido**: Si su entorno tiene un uso local de AD y también desea aprovechar las funcionalidades proporcionadas por Azure Active Directory, puede implementar dispositivos híbridos unidos a Azure AD. Se trata de dispositivos que están unidos tanto al entorno local de Active Directory como a Azure Active Directory.
  - **Escritura diferida de dispositivo**: la escritura diferida de dispositivo se usa para permitir el acceso condicional basado en dispositivos protegidos con AD FS (2012 R2 o posterior).

    >[!NOTE] 
    > - La opción para habilitar la escritura diferida de dispositivos de Personalizar las opciones de sincronización se mostrará atenuada. 
    > -  El módulo de PowerShell para ADPrep está en desuso en esta versión.



### <a name="fixed-issues"></a>Problemas corregidos 

- Esta versión actualiza la instalación de SQL Server Express a SQL Server 2012 SP4, que, entre otras cosas, ofrece correcciones para varias vulnerabilidades de seguridad.  Consulta [aquí](https://support.microsoft.com/help/4018073/sql-server-2012-service-pack-4-release-information) para obtener más información sobre SQL Server 2012 SP4.
- Procesamiento de reglas de sincronización: las reglas de sincronización de unión de salida sin condición de unión se deben dejar de aplicar si la regla de sincronización principal ya no es aplicable.
- Se han aplicado varias correcciones de accesibilidad a la interfaz de usuario de Synchronization Service Manager y al Editor de reglas de sincronización
- Asistente para Azure AD Connect: se genera un error al crear la cuenta del conector de AD si Azure AD Connect está en un grupo de trabajo.
- Asistente para Azure AD Connect: en la página de inicio de sesión de Azure AD, se muestra la casilla de verificación siempre que haya incoherencias entre los dominios de AD y los dominios de Azure AD verificados.
- Corrección de actualización automática de PowerShell para establecer el estado de actualización automática correctamente en ciertos casos después de intentar una actualización automática.
- Asistente para Azure AD Connect: se actualizó la telemetría para capturar la información que faltaba antes.
- Asistente para Azure AD Connect: los siguientes cambios se han realizado cuando se usa la tarea **Cambiar inicio de sesión de usuario** para cambiar de AD FS a autenticación de paso a través:
    - El agente de autenticación de paso a través está instalado en el servidor de Azure AD Connect y la característica de autenticación de paso a través está habilitada, antes de convertir dominios de federados a administrados.
    - Los usuarios ya no se convierten de federados a administrados. Solo se convierten los dominios.
- Asistente para Azure AD Connect: regex multidominio de AD FS no es correcto cuando el UPN del usuario tiene la actualización Regex del carácter especial ' para admitir caracteres especiales.
- Asistente para Azure AD Connect: se ha eliminado el mensaje falso para configurar el atributo de delimitador de origen cuando no hay ningún cambio. 
- Asistente para Azure AD Connect: compatibilidad de AD FS con el escenario de federación dual.
- Asistente para Azure AD Connect: las notificaciones de AD FS no se actualizan para el dominio agregado al convertir un dominio administrado en federado.
- Asistente para Azure AD Connect: Durante la detección de los paquetes instalados, encontramos productos relacionados con Dirsync, Sincronización de Azure AD y Azure AD Connect obsoletos. Ahora intentaremos desinstalar los productos obsoletos.
- Asistente para Azure AD Connect: se corrigió la asignación de mensajes de error cuando se produce un error de instalación del agente de autenticación de paso.
- Asistente para Azure AD Connect: se eliminó el contenedor "Configuración" de la página de filtrado de la unidad organizativa de dominio.
- Instalación de motor de sincronización: se quitó la lógica heredada innecesaria que, en ocasiones, provocaba errores del archivo msi de instalación del motor de sincronización.
- Asistente para Azure AD Connect: se corrigió el texto de ayuda emergente en la página Características opcionales para la sincronización de hash de contraseña.
- Entorno en tiempo de ejecución del motor de sincronización: se corrigió el escenario en que un objeto CS tiene una eliminación importada y las reglas de sincronización intentan volver a aprovisionarlo.
- Entorno en tiempo de ejecución del motor de sincronización: se agregó un vínculo de ayuda para la guía de solución de problemas de conectividad en línea al registro de eventos por un error de importación.
- Entorno en tiempo de ejecución del motor de sincronización: se redujo el uso de memoria del programador de sincronización al enumerar los conectores.
- Asistente para Azure AD Connect: se corrigió un problema para resolver una cuenta de servicio de sincronización personalizada que no tiene privilegios de lectura de AD.
- Asistente para Azure AD Connect: se mejoró el registro de selecciones de filtrado de dominio y unidad organizativa.
- Asistente para Azure AD Connect: se agregaron notificaciones predeterminadas de AD FS a la confianza de federación creada para el escenario de MFA.
- Asistente para Azure AD Connect: WAP de implementación de AD FS: al agregar el servidor no se puede usar un nuevo certificado.
- Asistente para Azure AD Connect: excepción de DSSO cuando no se inicializa onPremCredentials para un dominio. 
- Flujo preferente del atributo distinguishedName de AD desde el objeto de usuario activo.
- Se corrigió un error cosmético en el que la precedencia de la primera regla de sincronización OOB estaba definida como 99, en lugar de 100.



## <a name="117510"></a>1.1.751.0
Estado 12/4/2018: publicado solo para descarga.

>[!NOTE]
>Esta versión es una revisión para Azure AD Connect

### <a name="azure-ad-connect-sync"></a>Sincronización de Azure AD Connect
#### <a name="fixed-issues"></a>Problemas corregidos
Se ha corregido un problema en el que la detección automática de instancias de Azure para los inquilinos de China producía error en algunas ocasiones.  

### <a name="ad-fs-management"></a>Administración de AD FS
#### <a name="fixed-issues"></a>Problemas corregidos

Había un problema en la lógica de reintento de la configuración que podía provocar una excepción ArgumentException que indicaba que "Ya se agregó un elemento con la misma clave".  Esto podía provocar que todas las operaciones de reintento produjeran error.

## <a name="117500"></a>1.1.750.0
Estado 22/3/2018: publicado para actualización automática y descarga.
>[!NOTE]
>Cuando se haya completado la actualización a esta nueva versión, se desencadenará automáticamente una sincronización completa y una importación completa para el conector de Azure AD y una sincronización completa para el conector de AD. Puesto que esta acción puede tardar algún tiempo en función del tamaño del entorno de Azure AD Connect, asegúrese de que ha seguido los pasos necesarios para admitir esto o aplace la actualización hasta que haya encontrado un momento adecuado para hacerlo.

>[!NOTE]
>La funcionalidad AutoUpgrade se deshabilitó incorrectamente para algunos inquilinos que implementaron compilaciones posteriores a 1.1.524.0. Para asegurarse de que la instancia de Azure AD Connect aún es apta para AutoUpgrade, ejecute el siguiente cmdlet de PowerShell: "Set-ADSyncAutoUpgrade -AutoupGradeState Enabled"


### <a name="azure-ad-connect"></a>Azure AD Connect
#### <a name="fixed-issues"></a>Problemas corregidos

* Set-ADSyncAutoUpgrade cmdlet bloqueaba anteriormente la actualización automática si el estado de esta se configuraba en Suspendido. Ahora se ha cambiado esta funcionalidad para que no bloquee la función AutoUpgrade de futuras compilaciones.
* Se ha cambiado la opción de la página de **inicio de sesión de usuario** "Sincronización de contraseña" a "Sincronización de hash de contraseñas".  Azure AD Connect sincroniza los hash de contraseñas, no las contraseñas, por lo que esto se adapta a lo que realmente está ocurriendo.  Para más información, consulte [Implementación de la sincronización de hash de contraseñas con la sincronización de Azure AD Connect](how-to-connect-password-hash-synchronization.md).

## <a name="117490"></a>1.1.749.0
Estado: publicado para clientes exclusivos.

>[!NOTE]
>Cuando se haya completado la actualización a esta nueva versión, se desencadenará automáticamente una sincronización completa y una importación completa para el conector de Azure AD y una sincronización completa para el conector de AD. Puesto que esta acción puede tardar algún tiempo en función del tamaño del entorno de Azure AD Connect, asegúrese de que ha seguido los pasos necesarios para admitir esto o aplace la actualización hasta que haya encontrado un momento adecuado para hacerlo.

### <a name="azure-ad-connect"></a>Azure AD Connect
#### <a name="fixed-issues"></a>Problemas corregidos
* Corrección de la ventana de sincronización en las tareas en segundo plano para la página de filtrado de particiones al cambiar a la página siguiente.

* Se ha corregido un error que provocó una infracción de acceso durante la acción personalizada ConfigDB.

* Se ha corregido un error para recuperarse del tiempo de espera de la conexión de SQL.

* Se corrigió un error que provocaba que los certificados con caracteres comodín de SAN generaran un error al realizar una comprobación de requisitos previos.

* Se ha corregido un error que provocaba que miiserver.exe se bloqueara durante una exportación del conector de Azure AD.

* Se corrigió un error cuyo intento de contraseña incorrecta se registró en el controlador de dominio al ejecutar el asistente de Azure AD Connect para cambiar la configuración.


#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

* Adición de una configuración de privacidad para la reglamento general de protección de datos (GDPR).  Para obtener más información, consulte el siguiente [artículo](reference-connect-user-privacy.md).

[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]  

* Telemetría de aplicaciones: el administrador puede activar o desactivar este tipo de datos como desee

* Datos de mantenimiento de Azure AD: el administrador debe visitar el portal de mantenimiento para controlar la configuración de mantenimiento.
   Una vez se haya cambiado la directiva del servicio, los agentes la leerán y la aplicarán.

* Se han agregado acciones de configuración de reescritura de dispositivos y una barra de progreso para la página de inicialización.

* Se han mejorado los diagnósticos generales con un informe HTML y una recopilación completa de datos en un informe HTML o de texto ZIP.

* Se ha mejorado la confiabilidad de la actualización automática y se ha agregado telemetría adicional para asegurarse de que se puede determinar el mantenimiento del servidor.

* Se han restringido los permisos disponibles para las cuentas con privilegios de la cuenta del conector AD.

  * Para las nuevas instalaciones, el asistente restringirá los permisos que las cuentas con privilegios tiene en la cuenta de MSOL tras la creación de dicha cuenta.

Los cambios se encargarán de lo siguiente:
1. Instalaciones rápidas
2. Instalaciones personalizadas con la cuenta de creación automática
3. Se ha cambiado el instalador, por lo que no se requiere el privilegio de asociación de seguridad en una instalación limpia de Azure AD Connect.

* Se ha agregado una utilidad nueva para solucionar problemas de sincronización de un objeto específico. Está disponible en la opción "Solucionar problemas de sincronización de objetos" de la tarea adicional de solución de problemas del asistente de Azure AD Connect. Actualmente, la utilidad comprueba lo siguiente:

  * Error de coincidencia de UserPrincipalName entre el objeto de usuario sincronizado y la cuenta de usuario del inquilino de Azure AD.
  * Si se filtra el objeto de sincronización debido al filtrado de dominio
  * Si se filtra el objeto de sincronización debido al filtrado de unidad organizativa (UO)

* Se ha agregado una utilidad nueva para sincronizar el hash de contraseña actual almacenado en el Active Directory local para una cuenta de usuario específica.

La utilidad no requiere un cambio de contraseña. Está disponible en la opción "Solucionar problemas de sincronización del hash de contraseña" de la tarea adicional de solución de problemas del asistente de Azure AD Connect.






## <a name="116540"></a>1.1.654.0
Estado: 12 de diciembre de 2017

>[!NOTE]
>Esta versión es una revisión relacionada con la seguridad para Azure AD Connect

### <a name="azure-ad-connect"></a>Azure AD Connect
Se ha agregado un mejora a Azure AD Connect versión 1.1.654.0 (y posteriores) para garantizar que los cambios de permisos recomendados que se describen en la sección [Bloquear el acceso a la cuenta de AD DS](#lock) se aplican automáticamente cuando Azure AD Connect crea la cuenta de AD DS. 

- Al instalar Azure AD Connect, el administrador de instalación puede proporcionar una cuenta de AD DS existente o dejar que Azure AD Connect cree la cuenta automáticamente. Los cambios de permisos se aplican automáticamente a la cuenta de AD DS que crea Azure AD Connect durante la instalación. No se aplican a la cuenta de AD DS existente proporcionada por el administrador de instalación.
- Para los clientes que han actualizado desde una versión anterior de Azure AD Connect a 1.1.654.0 (o posterior), los cambios de permisos no se aplicarán con carácter retroactivo a las cuentas de AD DS existentes creadas antes de la actualización. Solo se aplicarán a las cuentas de AD DS nuevas creadas después de la actualización. Esto se produce cuando se agregan nuevos bosques de AD para sincronizarlos con Azure AD.

>[!NOTE]
>Esta versión solo quita la vulnerabilidad para las nuevas instalaciones de Azure AD Connect donde la cuenta de servicio se crea mediante el proceso de instalación. Para las instalaciones existentes o en los casos en que proporciona la cuenta usted mismo, debe asegurarse de que esta vulnerabilidad no existe.

#### <a name="lock"></a> Bloquear el acceso a la cuenta de AD DS
Bloquee el acceso a la cuenta de AD DS mediante la implementación de los siguientes cambios de permisos en las instancias de AD locales:  

*   Deshabilite la herencia en el objeto especificado.
*   Quite todas las ACE del objeto específico, excepto las ACE específicas de SELF. Deseamos mantener intactos los permisos predeterminados cuando se trata de SELF.
*   Asigne estos permisos específicos:

Type     | NOMBRE                          | Acceso               | Se aplica a
---------|-------------------------------|----------------------|--------------|
PERMITIR    | SYSTEM                        | Control total         | Este objeto  |
PERMITIR    | Administradores de empresas             | Control total         | Este objeto  |
PERMITIR    | Administradores de dominio                 | Control total         | Este objeto  |
PERMITIR    | Administradores                | Control total         | Este objeto  |
PERMITIR    | Enterprise Domain Controllers | Contenido de la lista        | Este objeto  |
PERMITIR    | Enterprise Domain Controllers | Lectura de todas las propiedades  | Este objeto  |
PERMITIR    | Enterprise Domain Controllers | Permisos de lectura     | Este objeto  |
PERMITIR    | Usuarios autenticados           | Contenido de la lista        | Este objeto  |
PERMITIR    | Usuarios autenticados           | Lectura de todas las propiedades  | Este objeto  |
PERMITIR    | Usuarios autenticados           | Permisos de lectura     | Este objeto  |

Para reforzar la configuración de la cuenta de AD DS, puede ejecutar [este script de PowerShell](https://gallery.technet.microsoft.com/Prepare-Active-Directory-ef20d978). El script de PowerShell asignará los permisos mencionados anteriormente a la cuenta de AD DS.

#### <a name="powershell-script-to-tighten-a-pre-existing-service-account"></a>Script de PowerShell para reforzar una cuenta de servicio existente

Para usar el script de PowerShell para aplicar esta configuración a una cuenta de AD DS existente (tanto si la proporciona la organización como si la crea una instalación anterior de Azure AD Connect), descargue el script desde el vínculo proporcionado anteriormente.

##### <a name="usage"></a>Uso:

```powershell
Set-ADSyncRestrictedPermissions -ObjectDN <$ObjectDN> -Credential <$Credential>
```

Dónde 

**$ObjectDN** = cuenta de Active Directory cuyos permisos se deben reforzar.

**$Credential** = credencial de administrador que tiene los privilegios necesarios para restringir los permisos de la cuenta $ObjectDN. Normalmente, el administrador de la empresa o del dominio mantiene estos privilegios. Use el nombre de dominio completo de la cuenta de administrador para evitar errores de búsqueda de cuenta. Ejemplo: contoso.com\admin.

>[!NOTE] 
>$credential.NombreDeUsuario debe tener el formato FQDN\nombreDeUsuario. Ejemplo: contoso.com\admin. 

##### <a name="example"></a>Ejemplo:

```powershell
Set-ADSyncRestrictedPermissions -ObjectDN "CN=TestAccount1,CN=Users,DC=bvtadwbackdc,DC=com" -Credential $credential 
```
### <a name="was-this-vulnerability-used-to-gain-unauthorized-access"></a>¿Se usó esta vulnerabilidad para obtener acceso no autorizado?

Para ver si esta vulnerabilidad se usó para poner en peligro su configuración de Azure AD Connect, debe comprobar la última fecha de restablecimiento de contraseña de la cuenta de servicio.  Si la marca de tiempo es inesperada, se debe llevar a cabo una investigación más extensa para ese evento de restablecimiento de contraseña mediante el registro de eventos.

Para obtener más información, vea [Microsoft Security Advisory 4056318](https://technet.microsoft.com/library/security/4056318).

## <a name="116490"></a>1.1.649.0
Estado: 27 de octubre de 2017

>[!NOTE]
>Esta compilación no está disponible para los clientes a través de la característica de actualización automática de Azure AD Connect.

### <a name="azure-ad-connect"></a>Azure AD Connect
#### <a name="fixed-issue"></a>Problema corregido
* Se ha solucionado un problema de compatibilidad de versiones entre Azure AD Connect y el agente Azure AD Connect Health (para la sincronización). Este problema afecta a los clientes que realizan la actualización local de Azure AD Connect a la versión 1.1.647.0, pero actualmente tienen la versión del agente de Health 3.0.127.0. Después de la actualización, el agente de Health ya no puede enviar datos de estado sobre el servicio de sincronización de Azure AD Connect al servicio Azure AD Health. Con esta corrección, la versión 3.0.129.0 del agente de mantenimiento se instala durante la actualización local de Azure AD Connect. La versión 3.0.129.0 del agente de Health no tiene el problema de compatibilidad con la versión 1.1.649.0 de Azure AD Connect.


## <a name="116470"></a>1.1.647.0
Estado: 19 de octubre de 2017

> [!IMPORTANT]
> Hay un problema de compatibilidad conocido entre Azure AD Connect versión 1.1.647.0 y el agente Azure AD Connect Health (para la sincronización) versión 3.0.127.0. Este problema impide que al agente de Health envíe datos de estado sobre el servicio de sincronización de Azure AD Connect (incluidos errores de sincronización de objetos y datos de historial de ejecución) a Azure AD Health Service. Antes de actualizar manualmente la implementación de Azure AD Connect a la versión 1.1.647.0, compruebe la versión actual del agente de Azure AD Connect Health que está instalado en el servidor de Azure AD Connect. Para ello puede ir a *Panel de Control → Agregar o quitar programas* y buscar la aplicación del *agente de Microsoft Azure AD Connect Health para la sincronización*. Si su versión es 3.0.127.0, se recomienda que espere a que la próxima versión de Azure AD Connect esté disponible antes de la actualización. Si la versión del agente Health no es 3.0.127.0, no hay problema en continuar con la actualización local manual. Tenga en cuenta que este problema no afecta a la actualización oscilante o a los clientes que realicen una instalación nueva de Azure AD Connect.
>
>

### <a name="azure-ad-connect"></a>Azure AD Connect
#### <a name="fixed-issues"></a>Problemas corregidos
* Se corrigió un problema con la tarea *Cambiar inicio de sesión de usuario* en el asistente de Azure AD Connect:

  * El problema se produce cuando tiene una implementación de Azure AD Connect existente con sincronización de contraseña **habilitada**, y está intentando establecer el método de inicio de sesión de usuario como *Autenticación de paso a través*. Antes de aplicar el cambio, el asistente muestra incorrectamente la solicitud "*Deshabilitar la sincronización de contraseña*". Sin embargo, la sincronización de contraseña sigue estando habilitada después de aplicar el cambio. Con esta corrección, el asistente ya no muestra la solicitud.

  * De forma predeterminada, el asistente no deshabilita la sincronización de contraseña cuando se actualiza el método de inicio de sesión de usuario usando la tarea *Cambiar inicio de sesión de usuario*. Esto sirve para evitar trastornos a los clientes que desean mantener la sincronización de contraseña, aunque vayan a habilitar la autenticación de paso a través o la federación como su método principal de inicio de sesión de usuario.
  
  * Si desea deshabilitar la sincronización de contraseña después de actualizar el método de inicio de sesión de usuario, tiene que ejecutar la tarea *Personalizar la configuración de sincronización* en el asistente. Cuando se desplaza a la página *Características opcionales* desactive la opción *Sincronización de contraseña*.
  
  * Tenga en cuenta que el mismo problema también se produce si intenta habilitar o deshabilitar un inicio de sesión único de conexión directa. De forma específica cuando tiene una implementación de Azure AD Connect existente con sincronización de contraseña habilitada, y el método de inicio de sesión de usuario ya está configurado como *Autenticación de paso a través*. Mediante la tarea *Cambiar inicio de sesión de usuario* puede activar o desactivar la opción *Habilitar el Inicio de sesión único de conexión directa* mientras el método de inicio de sesión de usuario se mantiene configurado como "Autenticación de paso a través". Antes de aplicar el cambio, el asistente muestra incorrectamente la solicitud "*Deshabilitar la sincronización de contraseña*". Sin embargo, la sincronización de contraseña sigue estando habilitada después de aplicar el cambio. Con esta corrección, el asistente ya no muestra la solicitud.

* Se corrigió un problema con la tarea *Cambiar inicio de sesión de usuario* en el asistente de Azure AD Connect:

  * El problema se produce cuando tiene una implementación de Azure AD Connect existente con sincronización de contraseña **deshabilitada**, y está intentando establecer el método de inicio de sesión de usuario como *Autenticación de paso a través*. Cuando se aplica el cambio, el asistente habilita tanto la autenticación de paso a través como la sincronización de contraseña. Con esta corrección, el asistente ya no habilita la sincronización de contraseña.

  * Anteriormente, la sincronización de contraseña era un requisito previo para habilitar la autenticación de paso a través. Al establecer el método de inicio de sesión de usuario como *Autenticación de paso a través*, permitiría tanto la autenticación de paso a través como la sincronización de contraseña. Recientemente, la sincronización de contraseña se ha eliminado como un requisito previo. Como parte de Azure AD Connect versión 1.1.557.0, se realizó un cambio en Azure AD Connect para no habilitar la sincronización de contraseña al establecer el método de inicio de sesión de usuario como *Autenticación de paso a través*. Sin embargo, el cambio se aplicó solamente a la instalación de Azure AD Connect. Con esta corrección, el mismo cambio se aplica también a la tarea *Cambiar inicio de sesión de usuario*.
  
  * Tenga en cuenta que el mismo problema también se produce si intenta habilitar o deshabilitar un inicio de sesión único de conexión directa. De forma específica cuando tiene una implementación de Azure AD Connect existente con sincronización de contraseña deshabilitada, y el método de inicio de sesión de usuario ya está configurado como *Autenticación de paso a través*. Mediante la tarea *Cambiar inicio de sesión de usuario* puede activar o desactivar la opción *Habilitar el Inicio de sesión único de conexión directa* mientras el método de inicio de sesión de usuario se mantiene configurado como "Autenticación de paso a través". Cuando se aplica el cambio, el asistente habilita la sincronización de contraseña. Con esta corrección, el asistente ya no habilita la sincronización de contraseña. 

* Se corrigió un problema que provocaba el fallo de la actualización de Azure AD Connect con el error "*No se pudo actualizar el componente servicio de sincronización*". Además, ya no se puede iniciar el servicio de sincronización con el error de evento "*No se pudo iniciar este servicio porque la versión de la base de datos es más reciente que la versión de los binarios instalados.*". El problema se produce cuando el administrador que realiza la actualización no tiene privilegios sysadmin para el servidor SQL Server que Azure AD Connect está usando. Con esta corrección, Azure AD Connect solo requiere que el administrador tenga privilegios db_owner en la base de datos de ADSync durante la actualización.

* Corregido un problema con la actualización de Azure AD Connect que afectaba a los clientes que han activado el [Inicio de sesión único de conexión directa](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso). Cuando se actualiza Azure AD Connect, el Inicio de sesión único de conexión directa aparece incorrectamente como deshabilitado en el asistente de Azure AD Connect, aunque la característica permanezca habilitada y totalmente funcional. Con esta solución, la característica ahora aparece correctamente como habilitada en el asistente.

* Se corrigió un problema que provocaba que el asistente de Azure AD Connect mostrase siempre la solicitud "*Configuración del delimitador de origen*" en la página *Listo para configurar*, incluso si no se había realizado ningún cambio relacionado con el delimitador de origen.

* Al realizar la actualización local manual de Azure AD Connect, el cliente debe proporcionar las credenciales de administrador global del correspondiente inquilino de Azure AD. Anteriormente, se podía realizar la actualización incluso si las credenciales de administrador global pertenecían a un inquilino de Azure AD diferente. Aunque la actualización parecía completarse sin problemas, determinadas configuraciones no se almacenaban correctamente tras la actualización. Con este cambio, el asistente evitará que la actualización continúe si las credenciales proporcionadas no coinciden con el inquilino de Azure AD.

* Eliminada la lógica redundante que reiniciaba innecesariamente el servicio Azure AD Connect Health al principio de una actualización manual.


#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
* Se ha agregado la lógica para simplificar los pasos necesarios para configurar Azure AD Connect con Microsoft Germany Cloud. Anteriormente, era necesario actualizar ciertas claves de registro específicas en el servidor de Azure AD Connect para que funcionase con Microsoft Germany Cloud, tal como se describe en este artículo. Ahora, Azure AD Connect puede detectar automáticamente si el inquilino está en Microsoft Germany Cloud basándose en las credenciales de administrador global proporcionadas durante la configuración.

### <a name="azure-ad-connect-sync"></a>Azure AD Connect Sync
> [!NOTE]
> Nota: El servicio de sincronización tiene una interfaz WMI que le permite desarrollar su propio programador personalizado. Esta interfaz está ahora en desuso y se eliminará en futuras versiones de Azure AD Connect que se suministren después del 30 de junio de 2018. Los clientes que deseen personalizar la programación de sincronización deben utilizar el [programador integrado (https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-feature-scheduler).

#### <a name="fixed-issues"></a>Problemas corregidos
* Cuando el asistente de Azure AD Connect crea la cuenta de AD Connector necesaria para sincronizar los cambios de la instancia local de Active Directory, no asigna correctamente a la cuenta el permiso necesario para leer objetos PublicFolder. Este problema afecta a la instalación rápida y a la instalación personalizada. Este cambio corrige el problema.

* Se ha corregido un problema que provocaba que la página de solución de problemas del asistente de Azure AD Connect no se represente correctamente para los administradores que la ejecuten desde Windows Server 2016.

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
* Cuando solucione problemas de sincronización de contraseña mediante la página de solución de problemas del asistente de Azure AD Connect, la página de solución de problemas devuelve ahora el estado específico de dominio.

* Anteriormente, si intentaba habilitar la sincronización de hash de contraseña, Azure AD Connect no comprobaba si la cuenta de AD Connector había necesitado permisos para sincronizar hashes de contraseña desde instancias locales de AD. Ahora, el asistente de Azure AD Connect comprobará y le advertirá si la cuenta de AD Connector no tiene permisos suficientes.

### <a name="ad-fs-management"></a>Administración de AD FS
#### <a name="fixed-issue"></a>Problema corregido
* Se corrigió un problema relacionado con el uso de la característica [ms-DS-ConsistencyGuid como sourceAnchor](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-design-concepts#using-ms-ds-consistencyguid-as-sourceanchor). Este problema afecta a los clientes que han configurado *Federación con AD FS* como método de inicio de sesión de usuario. Cuando se ejecuta la tarea *Configuración del delimitador de origen* en el asistente, Azure AD Connect pasa a utilizar * ms-DS-ConsistencyGuid como atributo de origen para immutableId. Como parte de este cambio, Azure AD Connect intenta actualizar las reglas de notificación para ImmutableId en AD FS. Sin embargo, este paso fallaba porque Azure AD Connect no tenía las credenciales de administrador necesarias para configurar AD FS. Con esta corrección, Azure AD Connect ahora le pide que escriba las credenciales de administrador para AD FS cuando se ejecuta la tarea *Configuración delimitador de origen*.



## <a name="116140"></a>1.1.614.0
Estado: 5 de septiembre de 2017

### <a name="azure-ad-connect"></a>Azure AD Connect

#### <a name="known-issues"></a>Problemas conocidos
* Se corrigió un problema que provocaba el fallo de la actualización de Azure AD Connect con el error "*No se pudo actualizar el componente servicio de sincronización*". Además, ya no se puede iniciar el servicio de sincronización con el error de evento "*No se pudo iniciar este servicio porque la versión de la base de datos es más reciente que la versión de los binarios instalados.*". El problema se produce cuando el administrador que realiza la actualización no tiene privilegios sysadmin para el servidor SQL Server que Azure AD Connect está usando. Los permisos de dbo no son suficientes.

* Hay un problema conocido con la actualización de Azure AD Connect que afecta a los clientes que han activado [Inicio de sesión único de conexión directa](how-to-connect-sso.md). Después de actualizar Azure AD Connect, la característica aparece como deshabilitada en el asistente, aunque realmente sigue estando habilitada. Se proporcionará una solución a este problema en una versión futura. Los clientes preocupados por este problema de presentación pueden solucionarlo manualmente habilitando la opción Inicio de sesión único de conexión directa en el asistente.

#### <a name="fixed-issues"></a>Problemas corregidos
* Se corrigió un problema que impedía que Azure AD Connect actualizara las reglas de notificaciones en AD FS local al tiempo que habilita la característica [ms-DS-ConsistencyGuid como delimitador de origen](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-design-concepts#using-ms-ds-consistencyguid-as-sourceanchor). El problema se produce si intenta habilitar la característica de una implementación de Azure AD Connect existente que tenga configurado AD FS como método de inicio de sesión. El problema se produce porque el asistente no solicita las credenciales de AD FS antes de intentar actualizar las reglas de notificaciones de AD FS.
* Se corrigió un problema que provocaba un error durante la instalación de Azure AD Connect si el bosque de AD local tenía NTLM deshabilitado. El problema se debe a que el asistente de Azure AD Connect no proporciona credenciales de acceso completas al crear los contextos de seguridad necesarios para la autenticación de Kerberos. Esto hace que se produzca un error durante la autenticación de Kerberos y que el asistente de Azure AD Connect recurra de nuevo al uso de NTLM.

### <a name="azure-ad-connect-sync"></a>Azure AD Connect Sync
#### <a name="fixed-issues"></a>Problemas corregidos
* Se ha corregido un problema por el que no se podía crear una nueva regla de sincronización si no estaba relleno el valor del atributo Tag.
* Se ha corregido un problema que provocaba que Azure AD Connect se conectara a una instancia de AD local para la sincronización de contraseña mediante NTLM, incluso aunque Kerberos estuviera disponible. Este problema se produce si la topología de AD local tiene uno o varios controladores de dominio que se restauraron a partir de una copia de seguridad.
* Se ha corregido un problema que provocaba que se repitieran todos los pasos de sincronización de forma innecesaria después de una actualización. Por lo general, realizar todos los pasos de sincronización es obligatorio después de una actualización si hay cambios en las reglas de sincronización integradas. El problema se producía por un error en la lógica de detección de cambios que incorrectamente detectaba un cambio al encontrar una expresión de una regla de sincronización con caracteres de nueva línea. Los caracteres de nueva línea se insertan en la expresión de una regla de sincronización para mejorar la legibilidad.
* Se ha corregido un problema que provocaba que el servidor de Azure AD Connect no funcionara correctamente después de la actualización automática. Este problema afecta a los servidores de Azure AD Connect de la versión 1.1.443.0 (o anterior). Para más información sobre el problema, consulte el artículo [Azure Connect de AD no funciona correctamente después de una actualización automática](https://support.microsoft.com/help/4038479/azure-ad-connect-is-not-working-correctly-after-an-automatic-upgrade).
* Se ha corregido un problema que puede provocar que se vuelva a intentar la actualización automática cada 5 minutos si se detectan errores. Con la corrección, la actualización automática se vuelve a intentar con retroceso exponencial cuando se producen errores.
* Se ha corregido un problema por el que el evento de sincronización de contraseña 611 se mostraba incorrectamente en los registros de eventos de aplicación de Windows como **informativo** en lugar de como **error**. El evento 611 se genera siempre que la sincronización de contraseña encuentra un problema. 
* Se ha corregido un problema en el asistente de Azure AD Connect que permitía que la característica de escritura diferida de grupos se habilitara sin seleccionar una unidad organizativa que es obligatoria para esta.

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
* Se ha agregado una tarea de solución de problemas al asistente de Azure AD Connect en Tareas adicionales. Los clientes pueden aprovechar esta tarea para solucionar problemas relacionados con la sincronización de contraseña y recopilar diagnósticos generales. En el futuro, la tarea de solución de problemas se ampliará para incluir otros problemas relacionados con la sincronización de directorios.
* Azure AD Connect ahora es compatible con un nuevo modo de instalación denominado **Usar base de datos existente**. Este modo de instalación permite a los clientes instalar una instancia de Azure AD Connect que especifica una base de datos existente de ADSync. Para más información sobre esta característica, consulte el artículo [Uso de una base de datos existente](how-to-connect-install-existing-database.md).
* Para una mayor seguridad, Azure AD Connect usa ahora TLS 1.2 como valor predeterminado para conectarse a Azure AD para la sincronización de directorios. Anteriormente, el valor predeterminado era TLS 1.0.
* Cuando se inicia el agente de sincronización de contraseña de Azure AD Connect, este intenta conectarse a un conocido punto de conexión de Azure AD para realizar la sincronización de contraseña. Después de conectarse correctamente, se le redirige a un punto de conexión específico de una región. Anteriormente, el agente de sincronización de contraseña almacenaba en caché el punto de conexión específico de la región hasta que se reiniciaba. Ahora, el agente borra la memoria caché y lo vuelve a intentar con el punto de conexión conocido si encuentra un problema de conexión con el punto de conexión específico de la región. Este cambio garantiza que la sincronización de contraseña pueda realizar una conmutación por error a un punto de conexión específico de la región diferente si el punto de conexión específico de la región almacenado en la memoria caché ya no está disponible.
* Para sincronizar los cambios desde un bosque de AD local, se necesita una cuenta de AD DS. Puede (i) crear la cuenta de AD DS usted mismo y proporcionar sus credenciales a Azure AD Connect o (ii) proporcionar credenciales del administrador de la empresa y dejar que Azure AD Connect cree la cuenta de AD DS en su nombre. Anteriormente, (i) era la opción predeterminada en el asistente de Azure AD Connect. Ahora, (ii) es la opción predeterminada.

### <a name="azure-ad-connect-health"></a>Azure AD Connect Health

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
* Se agregó compatibilidad para Microsoft Azure Government Cloud y Microsoft Cloud Germany.

### <a name="ad-fs-management"></a>Administración de AD FS
#### <a name="fixed-issues"></a>Problemas corregidos
* El cmdlet Initialize-ADSyncNGCKeysWriteBack del módulo de PowerShell de preparación de AD estaba aplicando de forma incorrecta los ACL al contenedor de registros de dispositivo y, por tanto, solo heredaba los permisos existentes.  Esto se ha actualizado para que la cuenta del servicio de sincronización tenga los permisos correctos.

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
* La tarea Comprobar inicio de sesión de ADFS de AAD Connect se ha actualizado de forma que ahora comprueba los inicios de sesión con Microsoft Online y no usa simplemente la recuperación de token de ADFS.
* Al configurar una nueva granja de ADFS mediante AAD Connect, se ha movido la página que solicitaba las credenciales de ADFS para que ahora aparezca antes de pedir al usuario que proporcione los servidores ADFS y WAP.  Esto permite que AAD Connect compruebe que la cuenta especificada tiene los permisos correctos.
* Durante la actualización de AAD Connect, ya no se producirá ningún error de la actualización aunque la confianza de ADFS AAD sufra un error al actualizarse.  Si eso sucede, se le mostrará al usuario el mensaje de advertencia correspondiente y este deberá continuar para restablecer la confianza mediante la tarea adicional de AAD Connect.

### <a name="seamless-single-sign-on"></a>Inicio de sesión único de conexión directa
#### <a name="fixed-issues"></a>Problemas corregidos
* Se ha corregido un problema que provocaba que el asistente de Azure AD Connect devolviera un error si intentaba habilitar [Inicio de sesión único de conexión directa](how-to-connect-sso.md). El mensaje de error es *"Se ha producido un error en la configuración del agente de autenticación de Microsoft Azure AD Connect".* Este problema afectaba a los clientes existentes que habían actualizado manualmente la versión preliminar de los agentes de autenticación de la [autenticación de paso a través](how-to-connect-sso.md) según los pasos descritos en este [artículo](how-to-connect-pta-upgrade-preview-authentication-agents.md).


## <a name="115610"></a>1.1.561.0
Estado: 23 de julio de 2017

### <a name="azure-ad-connect"></a>Azure AD Connect

#### <a name="fixed-issue"></a>Problema corregido

* Se ha corregido un problema que hacía que se quitara la regla de sincronización predeterminada "Out to AD - User ImmutableId":

  * El problema se produce cuando se actualiza Azure AD Connect o cuando se usa la opción de tarea *Actualizar configuración de sincronización* del asistente de Azure AD Connect para actualizar la configuración de sincronización de Azure AD Connect.
  
  * Esta regla de sincronización se aplica a los clientes que han habilitado la característica [ms-DS-ConsistencyGuid como delimitador de origen](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor). Esta característica se presentó en la versión 1.1.524.0 y posteriores. Cuando se quita la regla de sincronización, Azure AD Connect ya no puede rellenar el atributo local de AD ms-DS-ConsistencyGuid con el valor del atributo ObjectGuid. No evita que se aprovisionen nuevos usuarios en Azure AD.
  
  * La corrección garantiza que la regla de sincronización ya no se quite durante la actualización ni durante el cambio de configuración, siempre y cuando la característica esté habilitada. En el caso de los clientes existentes que se han visto afectados por este problema, la corrección también garantiza que la regla de sincronización se vuelva a agregar después de la actualización a esta versión de Azure AD Connect.

* Se ha corregido un problema que hace que las reglas de sincronización predeterminadas tengan un valor de precedencia inferior a 100:

  * En general, los valores de precedencia entre 0 y 99 están reservados a las reglas de sincronización personalizadas. Durante la actualización, se actualizan los valores de precedencia de las reglas de sincronización predeterminadas para incluir los cambios de la regla de sincronización. A causa de este problema, a las reglas de sincronización predeterminadas se les puede asignar un valor de precedencia inferior a 100.
  
  * La corrección evita que se produzca el problema durante la actualización, aunque no restaura los valores de precedencia de los clientes existentes que se han visto afectados por él. En el futuro se proporcionará una corrección independiente para ayudar con la restauración.

* Se ha corregido un problema que consiste en que la [pantalla Filtrado de dominios y unidades organizativas](how-to-connect-install-custom.md#domain-and-ou-filtering) del asistente de Azure AD Connect muestra como seleccionada la opción *Sincronizar todos los dominios y unidades organizativas* aun cuando está habilitado el filtrado basado en unidades organizativas.

*   Se ha corregido un problema que provocaba que la [pantalla de configuración de particiones de directorio](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering) de Synchronization Service Manager devolviera un error si se hacía clic en el botón *Actualizar*. El mensaje de error es *"An error was encountered while refreshing domains: Unable to cast object of type ‘System.Collections.ArrayList’ to type ‘Microsoft.DirectoryServices.MetadirectoryServices.UI.PropertySheetBase.MaPropertyPages.PartitionObject."* (Se encontró un error al actualizar los dominios: no se puede convertir el objeto de tipo ‘System.Collections.ArrayList’ en el tipo ‘Microsoft.DirectoryServices.MetadirectoryServices.UI.PropertySheetBase.MaPropertyPages.PartitionObject'): El error se produce si se ha agregado un nuevo dominio de AD a un bosque existente de AD y se está intentando actualizar Azure AD Connect con el botón Actualizar.

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

* La [característica Actualización automática](how-to-connect-install-automatic-upgrade.md) se ha ampliado para admitir clientes con las siguientes configuraciones:
  * Se ha habilitado la característica Reescritura de dispositivos.
  * Se ha habilitado la característica Reescritura de grupos.
  * La instalación no es una configuración rápida ni una actualización de DirSync.
  * Tiene más de 100.000 objetos en el metaverso.
  * Se está conectando a más de un bosque. La configuración rápida solo se conecta a un bosque.
  * La cuenta del conector AD ya no es la cuenta de MSOL_ predeterminada.
  * El servidor está establecido en modo provisional.
  * Se ha habilitado la característica Reescritura de usuarios.
  
  >[!NOTE]
  >La ampliación del ámbito de la característica Actualización automática afecta a los clientes con la compilación 1.1.105.0 y posteriores de Azure AD Connect. Si no quiere que el servidor de Azure AD Connect se actualice automáticamente, debe ejecutar el siguiente cmdlet en el servidor de Azure AD Connect: `Set-ADSyncAutoUpgrade -AutoUpgradeState disabled`. Para más información sobre la habilitación o deshabilitación de actualizaciones automáticas, consulte el artículo [Azure AD Connect: Actualización automática](how-to-connect-install-automatic-upgrade.md)

## <a name="115580"></a>1.1.558.0
Estado: no se publicará. Los cambios en esta compilación se incluyen en la versión 1.1.561.0.

### <a name="azure-ad-connect"></a>Azure AD Connect

#### <a name="fixed-issue"></a>Problema corregido

* Se ha corregido un problema que hacía que se quitara la regla de sincronización predeterminada "Out to AD - User ImmutableId" al actualizar la configuración de filtrado basado en unidades organizativas. Esta regla de sincronización es necesaria para la característica [ms-DS-ConsistencyGuid como delimitador de origen](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor).

* Se ha corregido un problema que consiste en que la [pantalla Filtrado de dominios y unidades organizativas](how-to-connect-install-custom.md#domain-and-ou-filtering) del asistente de Azure AD Connect muestra como seleccionada la opción *Sincronizar todos los dominios y unidades organizativas* aun cuando está habilitado el filtrado basado en unidades organizativas.

*   Se ha corregido un problema que provocaba que la [pantalla de configuración de particiones de directorio](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering) de Synchronization Service Manager devolviera un error si se hacía clic en el botón *Actualizar*. El mensaje de error es *"An error was encountered while refreshing domains: Unable to cast object of type ‘System.Collections.ArrayList’ to type ‘Microsoft.DirectoryServices.MetadirectoryServices.UI.PropertySheetBase.MaPropertyPages.PartitionObject."* (Se encontró un error al actualizar los dominios: no se puede convertir el objeto de tipo ‘System.Collections.ArrayList’ en el tipo ‘Microsoft.DirectoryServices.MetadirectoryServices.UI.PropertySheetBase.MaPropertyPages.PartitionObject'): El error se produce si se ha agregado un nuevo dominio de AD a un bosque existente de AD y se está intentando actualizar Azure AD Connect con el botón Actualizar.

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

* La [característica Actualización automática](how-to-connect-install-automatic-upgrade.md) se ha ampliado para admitir clientes con las siguientes configuraciones:
  * Se ha habilitado la característica Reescritura de dispositivos.
  * Se ha habilitado la característica Reescritura de grupos.
  * La instalación no es una configuración rápida ni una actualización de DirSync.
  * Tiene más de 100.000 objetos en el metaverso.
  * Se está conectando a más de un bosque. La configuración rápida solo se conecta a un bosque.
  * La cuenta del conector AD ya no es la cuenta de MSOL_ predeterminada.
  * El servidor está establecido en modo provisional.
  * Se ha habilitado la característica Reescritura de usuarios.
  
  >[!NOTE]
  >La ampliación del ámbito de la característica Actualización automática afecta a los clientes con la compilación 1.1.105.0 y posteriores de Azure AD Connect. Si no quiere que el servidor de Azure AD Connect se actualice automáticamente, debe ejecutar el siguiente cmdlet en el servidor de Azure AD Connect: `Set-ADSyncAutoUpgrade -AutoUpgradeState disabled`. Para más información sobre la habilitación o deshabilitación de actualizaciones automáticas, consulte el artículo [Azure AD Connect: Actualización automática](how-to-connect-install-automatic-upgrade.md)

## <a name="115570"></a>1.1.557.0
Estado: Julio de 2017

>[!NOTE]
>Esta compilación no está disponible para los clientes a través de la característica de actualización automática de Azure AD Connect.

### <a name="azure-ad-connect"></a>Azure AD Connect

#### <a name="fixed-issue"></a>Problema corregido
* Se ha corregido un problema con el cmdlet Initialize-ADSyncDomainJoinedComputerSync que provocaba que el dominio comprobado configurado en el objeto de punto de conexión de servicio existente se cambiara incluso cuando todavía era un dominio válido. Este problema se produce cuando el inquilino de Azure AD tiene más de un dominio comprobado que puede usarse para configurar el punto de conexión de servicio.

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
* La reescritura de contraseñas ahora está disponible en versión preliminar con la nube de Microsoft Azure Government y Microsoft Cloud Germany. Para más información sobre la compatibilidad de Azure AD Connect con las diferentes instancias de servicio, consulte el artículo [Azure AD Connect: consideraciones especiales para instancias](reference-connect-instances.md)

* Ahora el cmdlet Initialize-ADSyncDomainJoinedComputerSync tiene un nuevo parámetro opcional denominado AzureADDomain. Este parámetro le permite especificar qué dominio comprobado debe usarse para configurar el punto de conexión de servicio.

### <a name="pass-through-authentication"></a>Autenticación de paso a través

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
* El nombre del agente necesario para la autenticación de paso a través se ha cambiado de *Conector del proxy de la aplicación de Microsoft Azure AD* a *Agente de autenticación de Microsoft Azure AD Connect*.

* Al habilitar la autenticación de paso a través ya no se habilita la sincronización de hash de contraseña de manera predeterminada.


## <a name="115530"></a>1.1.553.0
Estado: junio de 2017

> [!IMPORTANT]
> Se produjeron cambios en las reglas de sincronización y el esquema en esta compilación. El servicio de sincronización de Azure AD Connect activará pasos de importación completa y sincronización completa después de la actualización. A continuación, se describen los detalles de los cambios. Para suspender temporalmente los pasos de Importación completa y Sincronización completa después de la actualización, vea el artículo [Cómo suspender la sincronización completa después de la actualización](how-to-upgrade-previous-version.md#how-to-defer-full-synchronization-after-upgrade).
>
>

### <a name="azure-ad-connect-sync"></a>Azure AD Connect Sync

#### <a name="known-issue"></a>Problema conocido
* Existe un problema que afecta a los clientes que están usando el [filtrado basado en la unidad organizativa](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering) con la sincronización de Azure AD Connect. Cuando se dirige a la [página Filtrado de dominios y unidades organizativas](how-to-connect-install-custom.md#domain-and-ou-filtering) en el Asistente de Azure AD Connect, se espera el siguiente comportamiento:
  * Si el filtrado basado en la unidad organizativa está habilitado, la opción **Sincronizar los dominios y las unidades organizativas seleccionados** está seleccionada.
  * De otro modo, la opción **Sincronizar todos los dominios y unidades organizativas** está seleccionada.

El problema que surge es que la **opción Sincronizar todos los dominios y unidades organizativas** siempre está seleccionada cuando ejecuta el asistente.  Esto sucede incluso si el filtrado basado en la unidad organizativa se ha configurado anteriormente. Antes de guardar cualquier cambio de configuración de AAD Connect, asegúrese de que la **opción Sincronizar los dominios y las unidades organizativas seleccionados está seleccionada** y confirme que todas las unidades organizativas que necesitan sincronizarse están habilitadas de nuevo. De otro modo, el filtrado basado en la unidad organizativa se deshabilitará.

#### <a name="fixed-issues"></a>Problemas corregidos

* Se ha corregido un problema con la reescritura de contraseñas que permite que un administrador de Azure AD restablezca la contraseña de una cuenta de usuario con privilegios de AD local. El problema sucede cuando a Azure AD Connect se le concede el permiso Restablecer contraseña en la cuenta con privilegios. El problema se trata en esta versión de Azure AD Connect no permitiendo que un administrador de Azure AD restablezca la contraseña de una cuenta de usuario con privilegios de AD local arbitraria a no ser que el administrador sea el propietario de esa cuenta. Para obtener más información, vea [Aviso de seguridad 4033453](https://technet.microsoft.com/library/security/4033453).

* Se ha corregido un problema relacionado con la característica [ms-DS-ConsistencyGuid como sourceAnchor](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-design-concepts#using-ms-ds-consistencyguid-as-sourceanchor) donde Azure AD Connect no reescribe en el atributo ms-DS-ConsistencyGuid de AD local. Este problema sucede cuando hay varios bosques de AD local agregados a Azure AD Connect y la *opción Existen identidades de usuario entre varios directorios* está seleccionada. Cuando se usa dicha configuración, las reglas de sincronización resultantes no rellenan el atributo sourceAnchorBinary en el metaverso. El atributo sourceAnchorBinary se usa como el atributo de origen para el atributo ms-DS-ConsistencyGuid. Como resultado, no se produce la reescritura en el atributo ms-DSConsistencyGuid. Para corregir el problema, las siguientes reglas de sincronización se han actualizado para garantizar que siempre se rellene el atributo sourceAnchorBinary en el metaverso:
  * In from AD - InetOrgPerson AccountEnabled.xml
  * In from AD - InetOrgPerson Common.xml
  * In from AD - User AccountEnabled.xml
  * In from AD - User Common.xml
  * In from AD - User Join SOAInAAD.xml

* Anteriormente, aunque la característica [ms-DS-ConsistencyGuid como sourceAnchor](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-design-concepts#using-ms-ds-consistencyguid-as-sourceanchor) no esté habilitada, la regla de sincronización "Out to AD – User ImmutableId" todavía se agrega a Azure AD Connect. El efecto es benigno y no provoca que se produzca la reescritura del atributo ms-DS-ConsistencyGuid. Para evitar confusión, se ha agregado lógica para garantizar que la regla de sincronización solo se agrega cuando la característica está habilitada.

* Se ha corregido un problema que provocaba un error en la sincronización de hash de contraseña con el evento de error 611. Este problema sucede después de que uno o más controladores de dominio se hayan quitado de AD local. Al final de cada ciclo de sincronización de contraseña, la cookie de sincronización que AD local ha emitido contiene identificadores de invocación de los controladores de dominio que se han quitado con el valor USN (Números de secuencias actualizadas) de 0. El administrador de sincronización de contraseña no puede conservar la cookie de sincronización que contiene el valor USN de 0 y produce un error con el evento de error 611. Durante el siguiente ciclo de sincronización, el administrador de sincronización de contraseña vuelve a usar la última cookie de sincronización conservada que no contiene el valor USN de 0. Esto provoca que la misma contraseña cambie para volver a sincronizarse. Con esta corrección, el administrador de sincronización de contraseña conserva la cookie de sincronización correctamente.

* Anteriormente, incluso si la actualización automática se había deshabilitado con el cmdlet Set-ADSyncAutoUpgrade, el proceso de actualización automática seguía buscando actualizaciones periódicamente, y se basaba en el instalador descargado para asignar la inhabilitación. Con esta corrección, el proceso de actualización automática ya no busca actualizaciones periódicamente. La corrección se aplica automáticamente cuando el instalador de actualización para esta versión de Azure AD Connect se ejecuta una vez.

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

* Anteriormente, la característica [ms-DS-ConsistencyGuid como sourceAnchor](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-design-concepts#using-ms-ds-consistencyguid-as-sourceanchor) estaba disponible solo para nuevas implementaciones. Ahora, está disponible para las implementaciones existentes. Más concretamente:
  * Para tener acceso a la característica, inicie el asistente de Azure AD Connect y pulse la opción *Update Source Anchor* (Actualizar delimitador de origen).
  * Esta opción solo está visible para las implementaciones existentes que están usando objectGuid como el atributo sourceAnchor.
  * Al configurar la opción, el asistente valida el estado del atributo ms-DS-ConsistencyGuid en Active Directory local. Si el atributo no está configurado en ningún objeto de usuario del directorio, el asistente usa ms-DS-ConsistencyGuid como atributo sourceAnchor. Si el atributo está configurado en uno o varios objetos de usuario del directorio, el asistente concluye que el atributo se está usando en otras aplicaciones, no es adecuado como atributo sourceAnchor y no permite el cambio de sourceAnchor para continuar. Si está seguro de que el atributo no se está usando en aplicaciones existentes, debe ponerse en contacto con el soporte técnico para obtener información sobre cómo suprimir el error.

* En relación al atributo **userCertificate** en los objetos de dispositivo, Azure AD Connect ahora busca los valores de certificado necesarios para [Conectar dispositivos unidos a dominio a Azure AD para la experiencia de Windows 10](https://docs.microsoft.com/azure/active-directory/active-directory-azureadjoin-devices-group-policy) y filtra el resto antes de sincronizarse con Azure AD. Para permitir este comportamiento, la regla de sincronización inmediata "Out to AAD - Device Join SOAInAD" se ha actualizado.

* Azure AD Connect ahora admite la reescritura del atributo **cloudPublicDelegates** de Exchange Online en el atributo **publicDelegates** de AD local. Esto permite el escenario donde se pueden conceder derechos SendOnBehalfTo a un buzón de Exchange Online para los usuarios con buzones de Exchange locales. Para admitir esta característica, se ha agregado una nueva regla de sincronización inmediata "Out to AD – User Exchange Hybrid PublicDelegates writeback". Esta regla de sincronización solo se agrega a Azure AD Connect cuando la característica híbrida de Exchange está habilitada.

* Ahora Azure AD Connect admite la sincronización del atributo **altRecipient** desde Azure AD. Para admitir este cambio, se han actualizado las siguientes reglas de sincronización inmediatas para incluir el flujo de atributo necesario:
  * In from AD – User Exchange
  * Out to AAD – User ExchangeOnline
  
* El atributo **cloudSOAExchMailbox** en el metaverso indica si un usuario determinado tiene un buzón de Exchange Online o no. Su definición se ha actualizado para incluir elementos RecipientDisplayTypes de Exchange Online adicionales como buzones de sala de conferencias y equipamiento. Para permitir este cambio, la definición del atributo cloudSOAExchMailbox, que se encuentra en la regla de sincronización inmediata "In from AAD – User Exchange Hybrid" se ha actualizado desde:

```
CBool(IIF(IsNullOrEmpty([cloudMSExchRecipientDisplayType]),NULL,BitAnd([cloudMSExchRecipientDisplayType],&amp;HFF) = 0))
```

...para lo siguiente:

```
CBool(
  IIF(IsPresent([cloudMSExchRecipientDisplayType]),(
    IIF([cloudMSExchRecipientDisplayType]=0,True,(
      IIF([cloudMSExchRecipientDisplayType]=2,True,(
        IIF([cloudMSExchRecipientDisplayType]=7,True,(
          IIF([cloudMSExchRecipientDisplayType]=8,True,(
            IIF([cloudMSExchRecipientDisplayType]=10,True,(
              IIF([cloudMSExchRecipientDisplayType]=16,True,(
                IIF([cloudMSExchRecipientDisplayType]=17,True,(
                  IIF([cloudMSExchRecipientDisplayType]=18,True,(
                    IIF([cloudMSExchRecipientDisplayType]=1073741824,True,(
                       IF([cloudMSExchRecipientDisplayType]=1073741840,True,False)))))))))))))))))))),False))

```

* Se ha agregado el siguiente conjunto de funciones compatibles con X509Certificate2 para crear expresiones de reglas de sincronización para controlar los valores de certificado en el atributo userCertificate:

    ||||
    | --- | --- | --- |
    |CertSubject|CertIssuer|CertKeyAlgorithm|
    |CertSubjectNameDN|CertIssuerOid|CertNameInfo|
    |CertSubjectNameOid|CertIssuerDN|IsCert|
    |CertFriendlyName|CertThumbprint|CertExtensionOids|
    |CertFormat|CertNotAfter|CertPublicKeyOid|
    |CertSerialNumber|CertNotBefore|CertPublicKeyParametersOid|
    |CertVersion|CertSignatureAlgorithmOid|Seleccionar|
    |CertKeyAlgorithmParams|CertHashString|Dónde|
    |||With|

* Se han presentado los siguientes cambios de esquema para permitir que los clientes creen reglas de sincronización personalizadas para el flujo de sAMAccountName, domainNetBios, y domainFQDN para los objetos de grupo, así como distinguishedName para los objetos de usuario:

  * Los siguientes atributos se han agregado al esquema de metaverso:
    * Group: Nombre de cuenta
    * Group: domainNetBios
    * Group: domainFQDN
    * Person: distinguishedName

  * Los siguientes atributos se han agregado al esquema de Azure AD Connector:
    * Group: OnPremisesSamAccountName
    * Group: NetBiosName
    * Group: DnsDomainName
    * User: OnPremisesDistinguishedName

* Ahora el script de cmdlet ADSyncDomainJoinedComputerSync tiene un nuevo parámetro opcional denominado AzureEnvironment. El parámetro se usa para especificar en qué región se hospeda el correspondiente inquilino de Azure Active Directory. Los valores válidos son:
  * AzureCloud (predeterminado)
  * AzureChinaCloud
  * AzureGermanyCloud
  * USGovernment
 
* Editor de reglas de sincronización actualizado para usar Unir (en lugar de Aprovisionar) como el valor predeterminado de tipo de vínculo durante la creación de las reglas de sincronización.

### <a name="ad-fs-management"></a>Administración de AD FS

#### <a name="issues-fixed"></a>Problemas corregidos

* Las siguientes direcciones URL son nuevos puntos de conexión de WS-Federation que ha presentado Azure AD para mejorar la resistencia frente a la interrupción de la autenticación y se agregarán a la configuración de relación de confianza para usuario autenticado de AD FS local:
  * https://ests.login.microsoftonline.com/login.srf
  * https://stamp2.login.microsoftonline.com/login.srf
  * https://ccs.login.microsoftonline.com/login.srf
  * https://ccs-sdf.login.microsoftonline.com/login.srf
  
* Se ha corregido un problema que provocaba que AD FS generara un valor de notificación incorrecto para IssuerID. Este problema se produce si hay varios dominios comprobados en el inquilino de Azure AD y el sufijo del dominio del atributo userPrincipalName que se ha usado para generar la notificación IssuerID tiene al menos 3 niveles de profundidad (por ejemplo, johndoe@us.contoso.com). El problema se resuelve actualizando la expresión regular que han usado las reglas de notificación.

#### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
* Anteriormente, la característica Administración de certificados de ADFS proporcionada por Azure AD Connect solo podía usarse con granjas de ADFS administradas mediante Azure AD Connect. Ahora, puede usar la característica con granjas de ADFS que no se administran mediante Azure AD Connect.

## <a name="115240"></a>1.1.524.0
Fecha de publicación: mayo de 2017

> [!IMPORTANT]
> Se produjeron cambios en las reglas de sincronización y el esquema en esta compilación. El servicio de sincronización de Azure AD Connect activará pasos de importación completa y sincronización completa después de la actualización. A continuación, se describen los detalles de los cambios.
>
>

**Problemas corregidos:**

Sincronización de Azure AD Connect

* Se ha corregido un problema que hacía que se produjera la actualización automática en el servidor de Azure AD Connect incluso si el cliente había deshabilitado la característica mediante el cmdlet Set-ADSyncAutoUpgrade. Con esta corrección, el proceso de actualización automática en el servidor sigue comprobando periódicamente si hay que actualizar, pero el instalador descargado respeta la configuración de actualización automática.
* Durante la actualización local de DirSync, Azure AD Connect crea una cuenta de servicio de Azure AD que el conector de Azure AD va a usar para sincronizar con Azure AD. Una vez creada la cuenta, Azure AD Connect la usa para autenticarse con Azure AD. En ocasiones, la autenticación genera un error por problemas transitorios, lo que a su vez hace que la actualización local de DirSync produzca un *error al ejecutar la tarea de configuración de la sincronización de AAD: AADSTS50034: Para iniciar sesión en esta aplicación, la cuenta debe agregarse al directorio xxx.onmicrosoft.com.* Para mejorar la resistencia de la actualización de DirSync, Azure AD Connect ahora reintenta el paso de autenticación.
* Había un problema con la compilación 443 que hacía que la actualización local de DirSync se realizara correctamente, pero no se creaban perfiles de ejecución necesarios para la sincronización de directorios. Se incluye lógica de recuperación en esta compilación de Azure AD Connect. Cuando el cliente actualiza a esta compilación, Azure AD Connect detecta que faltan perfiles de ejecución y los crea.
* Se corrigió un problema que hacía que el proceso de sincronización de contraseña no se iniciara con el id. de evento 6900 y el error *"Ya se agregó un elemento con la misma clave"*. Este problema se producía si actualizaba la configuración del filtrado por unidad organizativa para incluir la partición de configuración de AD. Para corregir este problema, ahora el proceso de sincronización de contraseña sincroniza los cambios de contraseña solo desde particiones de dominio de AD. Se omiten las particiones que no son de dominio, como la partición de configuración.
* Durante la instalación rápida, Azure AD Connect crea una cuenta de AD DS local que el conector de AD usará para comunicarse con la instancia local de AD. Antes, la cuenta se creaba con la marca PASSWD_NOTREQD establecida en el atributo user-Account-Control y se establecía una contraseña aleatoria en la cuenta. Ahora, Azure AD Connect quita explícitamente la marca PASSWD_NOTREQD una vez que se establece la contraseña en la cuenta.
* Se corrigió un problema que causaba un error de actualización de DirSync que indicaba que *se había producido un interbloqueo en SQL Server al intentar adquirir un bloqueo de aplicación* cuando se encontraba el atributo mailNickname en el esquema de AD local, pero no estaba enlazado a la clase de objeto de usuario de AD.
* Se ha corregido un problema que hacía que la característica Reescritura de dispositivos se deshabilitara automáticamente cuando un administrador estaba actualizando la configuración de Azure AD Connect Sync mediante el asistente de Azure AD Connect. Este problema se debía a que el asistente realizaba una comprobación de requisitos previos para la configuración de escritura diferida de dispositivo existente en la instancia local de AD y se producía un error en la comprobación. La solución consiste en omitir la comprobación si Reescritura de dispositivos ya se había habilitado antes.
* Para configurar el filtrado por unidad organizativa, puede usar el asistente de Azure AD Connect o Synchronization Service Manager. Anteriormente, si usaba al asistente de Azure AD Connect para configurar el filtrado por unidad organizativa, las unidades organizativas nuevas creadas después se incluían en la sincronización de directorios. Si no desea que se incluyan las unidades organizativas nuevas, debe configurar el filtrado por unidad organizativa mediante Synchronization Service Manager. Ahora, puede lograr el mismo comportamiento con el asistente de Azure AD Connect.
* Se ha corregido un problema que hacía que los procedimientos almacenados que Azure AD Connect necesitaba se crearan bajo el esquema del administrador que llevaba a cabo la instalación, en lugar de bajo el esquema dbo.
* Se ha corregido un problema que hacía que el atributo TrackingId devuelto por Azure AD se omitiera en los registros de eventos del servidor de AAD Connect. El problema se producía si Azure AD Connect recibía un mensaje de redirección de Azure AD y Azure AD Connect no se podía conectar al punto de conexión proporcionado. Los ingenieros de soporte técnico usan TrackingId para correlacionarlo con los registros en el servicio durante la solución de problemas.
* Cuando Azure AD Connect recibe un error LargeObject de Azure AD, Azure AD Connect genera un evento con el id. 6941 y el mensaje *"El objeto aprovisionado es demasiado grande. Recorte el número de valores de atributo de este objeto"*. Al mismo tiempo, Azure AD Connect también genera un evento confuso con EventID 6900 y el mensaje *"Microsoft.Online.Coexistence.ProvisionRetryException: Unable to communicate with the Windows Azure Active Directory service.”* (No es posible comunicar con el servicio Azure Active Directory) Para minimizar las confusiones, Azure AD Connect ya no genera este último evento cuando se recibe el error LargeObject.
* Se ha corregido un problema que hacía que Synchronization Service Manager dejara de responder al intentar actualizar la configuración para un conector LDAP genérico.

**Nuevas características y mejoras:**

Sincronización de Azure AD Connect
* Cambios en las reglas de sincronización: se han implementado los siguientes cambios en reglas de sincronización:
  * Se ha actualizado el conjunto de reglas de sincronización predeterminado para que no se exporten los atributos **userCertificate** ni **userSMIMECertificate** si tienen más de 15 valores.
  * Ahora los atributos **employeeID** y **msExchBypassModerationLink** de AD están incluidos en el conjunto de reglas de sincronización predeterminado.
  * Se ha quitado el atributo **photo** de AD del conjunto de reglas de sincronización predeterminado.
  * Se ha agregado **preferredDataLocation** al esquema de metaverso y al del conector de AAD. Los clientes que deseen actualizar cualquiera de los atributos en Azure AD pueden implementar reglas de sincronización personalizadas para hacerlo. 
  * Agregue **userType** al esquema de metaverso y al del conector de AAD. Los clientes que deseen actualizar cualquiera de los atributos en Azure AD pueden implementar reglas de sincronización personalizadas para hacerlo.

* Ahora Azure AD Connect permite automáticamente el uso del atributo ConsistencyGuid como el atributo sourceAnchor para los objetos de AD local. Además, Azure AD Connect rellena el atributo ConsistencyGuid con el valor del atributo objectGuid si está vacío. Esta característica solo es aplicable a nueva implementaciones. Para más información sobre esta característica, consulte la sección del artículo [Azure AD Connect: Conceptos de diseño: Uso de msDS-ConsistencyGuid como sourceAnchor](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor).
* Se ha agregado un nuevo cmdlet de solución de problemas, Invoke-ADSyncDiagnostics, para ayudar a diagnosticar problemas relacionados con la sincronización de hash de contraseña. Para más información sobre el uso del cmdlet, consulte el artículo [Solución de problemas de sincronización de hash de contraseñas con la sincronización de Azure AD Connect](tshoot-connect-password-hash-synchronization.md).
* Azure AD Connect es compatible ahora con la sincronización de objetos de carpeta pública habilitada para correo de una instancia local de AD a Azure AD. Puede habilitar la característica con el asistente de Azure AD Connect en Características opcionales. Para obtener más información sobre esta característica, vea el artículo [Compatibilidad del bloqueo perimetral basado en directorios de Office 365 para las carpetas públicas habilitadas para correo locales](https://blogs.technet.microsoft.com/exchange/2017/05/19/office-365-directory-based-edge-blocking-support-for-on-premises-mail-enabled-public-folders).
* Azure AD Connect necesita una cuenta de AD DS para sincronizar desde la instancia local de AD. Anteriormente, si se instalaba Azure AD Connect con el modo rápido, se podían proporcionar las credenciales de una cuenta de administrador de organización y Azure AD Connect creaba la cuenta de AD DS necesaria. Pero para una instalación personalizada y para agregar bosques a una implementación existente, había que proporcionar la cuenta de AD DS. Ahora, también tiene la opción de proporcionar las credenciales de una cuenta de administrador de organización durante una instalación personalizada y dejar que Azure AD Connect cree la cuenta de AD DS necesaria.
* Azure AD Connect ahora es compatible con AOA de SQL. Debe habilitar AOA de SQL antes de instalar Azure AD Connect. Durante la instalación, Azure AD Connect detecta si la instancia de SQL proporcionada está habilitada para AOA de SQL o no. Si AOA de SQL está habilitada, Azure AD Connect averigua si AOA de SQL está configurada para usar la replicación sincrónica o asincrónica. Cuando configure la escucha de grupo de disponibilidad, se recomienda que establezca la propiedad RegisterAllProvidersIP en 0. Esta recomendación se debe a que Azure AD Connect actualmente usa SQL Native Client para conectarse a SQL y SQL Native Client no admite el uso de la propiedad MultiSubNetFailover.
* Si usa LocalDB como base de datos para el servidor de Azure AD Connect y ha alcanzado su límite de 10 GB de tamaño, el servicio de sincronización deja de iniciarse. Antes, tenía que realizar la operación ShrinkDatabase en LocalDB para recuperar el suficiente espacio de base de datos para iniciar el servicio de sincronización. Después, podía usar Synchronization Service Manager para eliminar el historial de ejecución para reclamar más espacio de base de datos. Ahora, puede usar el cmdlet Start-ADSyncPurgeRunHistory para purgar los datos del historial de LocalDB y recuperar espacio de base de datos. Además, este cmdlet admite un modo sin conexión (especificando el parámetro -offline) que se puede usar cuando no se está ejecutando el servicio de sincronización. Nota: El modo sin conexión solo se puede usar si el servicio de sincronización no se está ejecutando y la base de datos usada es LocalDB.
* Para reducir la cantidad de espacio de almacenamiento necesario, ahora Azure AD Connect comprime los detalles de errores de sincronización antes de almacenarlos en las bases de datos LocalDB y SQL. Al actualizar desde una versión anterior de Azure AD Connect a esta versión, Azure AD Connect realiza una compresión única de los detalles de errores de sincronización existentes.
* Antes, después de actualizar la configuración del filtrado por unidad organizativa, había que ejecutar manualmente la importación completa para asegurarse de que los objetos existentes quedaran correctamente incluidos o excluidos de la sincronización de directorios. Ahora, Azure AD Connect desencadena automáticamente la importación completa durante el próximo ciclo de sincronización. Además, la importación completa solo se aplica a los conectores de AD afectados por la actualización. Nota: Esta mejora es aplicable a las actualizaciones del filtrado por unidad organizativa realizadas con el asistente de Azure AD Connect únicamente. No es aplicable al filtrado por unidad organizativa realizado mediante Synchronization Service Manager.
* Antes, el filtrado basado en grupo solo admitía usuarios, grupos y objetos de contacto. Ahora, también admite objetos de equipo.
* Antes, podía eliminar datos de espacio conector sin deshabilitar el programador de Azure AD Connect Sync. Ahora, Synchronization Service Manager bloquea la eliminación de datos del espacio conector si detecta que el programador está habilitado. Además, se devuelve una advertencia para informar a los clientes acerca de la posible pérdida de datos si se eliminan los datos del espacio conector.
* Anteriormente, tenía que deshabilitar la transcripción de PowerShell para que el asistente de Azure AD Connect se ejecutara correctamente. Este problema se ha resuelto en parte. Puede habilitar la transcripción de PowerShell si usa el asistente de Azure AD Connect para administrar la configuración de sincronización. Debe deshabilitarla si usa el asistente de Azure AD Connect para administrar la configuración de AD FS.



## <a name="114860"></a>1.1.486.0
Fecha de publicación: Abril de 2017

**Problemas corregidos:**
* Se ha corregido el problema según el cual Azure AD Connect no se instala correctamente en una versión localizada de Windows Server.

## <a name="114840"></a>1.1.484.0
Fecha de publicación: Abril de 2017

**Problemas conocidos**:

* Esta versión de Azure AD Connect no se instalará correctamente si se cumplen las condiciones siguientes:
   1. Realiza una actualización de DirSync o una instalación nueva de Azure AD Connect.
   2. Utiliza una versión localizada de Windows Server, donde el nombre del grupo de administradores integrado en el servidor no es "Administradores".
   3. Está usando la instancia predeterminada de Local DB incluida en SQL Server 2012 Express que se instala con Azure AD Connect en lugar de proporcionar su propia versión completa de SQL Server.

**Problemas corregidos:**

Sincronización de Azure AD Connect
* Se ha corregido un problema según el cual el programador de sincronización omite todo el paso de sincronización si uno o varios conectores no tienen un perfil de ejecución para ese paso de sincronización. Por ejemplo, puede agregar manualmente un conector con Synchronization Service Manager sin crear un perfil de ejecución de importación diferencial para él. Esta revisión garantiza que el programador de sincronización sigue ejecutando la importación diferencial para los demás conectores.
* Se corrigió un problema según el cual el servicio de sincronización detiene inmediatamente el procesamiento de un perfil de ejecución cuando es encuentra un problema con uno de los pasos de ejecución. Esta revisión garantiza que el servicio de sincronización omite ese paso de ejecución y continúa procesando el resto. Por ejemplo, tiene un perfil de ejecución de importación diferencial para el conector AD con varios pasos de ejecución (uno para cada dominio de Active Directory local). El servicio de sincronización ejecutará la importación diferencial con los demás dominios de AD, incluso si uno de ellos tiene problemas de conectividad de red.
* Se ha corregido un problema que hace que la actualización del conector Azure AD se omita durante la actualización automática.
* Se corrigió un problema que hace que Azure AD Connect determine incorrectamente si el servidor es un controlador de dominio durante la instalación, que a su vez provoca un error de actualización en DirSync.
* Se ha corregido un problema que hace que la actualización en contexto de DirSync no cree ningún perfil de ejecución para el conector Azure AD.
* Se ha corregido un problema según el cual la interfaz de usuario de Synchronization Service Manager deja de responder al intentar configurar el conector LDAP genérico.

Administración de AD FS
* Se ha corregido un problema en el que se produce un error en el Asistente para Azure AD Connect si el nodo principal de AD FS se ha movido a otro servidor.

SSO de escritorio
* Se ha corregido un problema en el Asistente para Azure AD Connect en el que la pantalla de inicio de sesión no le permite habilitar la característica de SSO de escritorio si ha elegido la sincronización de contraseña como opción de inicio de sesión durante la instalación nueva.

**Nuevas características y mejoras:**

Sincronización de Azure AD Connect
* Sincronización de Azure AD Connect ahora admite el uso de la cuenta de servicio virtual, la cuenta de servicio administrado y la cuenta de servicio administrado de grupo como su cuenta de servicio. Esto se aplica únicamente a la nueva instalación de Azure AD Connect. Al instalar Azure AD Connect:
    * De forma predeterminada, el Asistente para Azure AD Connect creará una cuenta de servicio virtual y la utilizará como su cuenta de servicio.
    * Si realiza la instalación en un controlador de dominio, Azure AD Connect vuelve al comportamiento anterior donde se creará una cuenta de usuario de dominio y se utilizará como cuenta de servicio.
    * Puede invalidar el comportamiento predeterminado al proporcionar una de las opciones siguientes:
      * Una cuenta de servicio administrada de grupo
      * Una cuenta de servicio administrada
      * Una cuenta de usuario de dominio
      * Una cuenta de usuario local
* Anteriormente, si actualizaba a una nueva versión de Azure AD Connect que contenía la actualización de los conectores o cambios de las reglas de sincronización, Azure AD Connect desencadenaba un ciclo completo de sincronización. Ahora, Azure AD Connect desencadena selectivamente el paso de importación completa solo para los conectores con actualización y el paso de sincronización completa solo para los conectores con cambios de la regla de sincronización.
* Anteriormente, el umbral de eliminación de exportación solo se aplicaba a las exportaciones desencadenadas mediante el programador de sincronización. Ahora, la característica se extiende para incluir las exportaciones desencadenadas manualmente por el cliente con Synchronization Service Manager.
* En su inquilino de Azure AD, hay una configuración del servicio que indica si la característica de sincronización de contraseña está habilitada para el inquilino o no. Anteriormente, era fácil que la configuración de servicio no se configurara correctamente por Azure AD Connect cuando tenía un servidor provisional y activo. Ahora, Azure AD Connect intenta mantener la coherencia de la configuración del servicio solo con su servidor de Azure AD Connect activo.
* El Asistente para Azure AD Connect ahora detecta y devuelve una advertencia si la instancia de AD local no tiene la papelera de reciclaje de AD habilitada.
* Anteriormente, se agotaba el tiempo de espera de la exportación a Azure AD y se producía un error si el tamaño combinado de los objetos en el lote superaba cierto umbral. Ahora, el servicio de sincronización intentará volver a enviar los objetos en lotes más pequeños e independientes si se encuentra el problema.
* La aplicación de administración de claves del servicio de sincronización se quitó del menú Inicio de Windows. La administración de la clave de cifrado continuará admitiéndose a través de la interfaz de línea de comandos mediante miiskmu.exe. Para más información sobre cómo administrar la clave de cifrado, consulte el artículo [Abandonar la clave de cifrado de sincronización de Azure AD Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-change-serviceacct-pass#abandoning-the-azure-ad-connect-sync-encryption-key).
* Anteriormente, si cambiaba la contraseña de la cuenta del servicio de sincronización de Azure AD Connect, el servicio de sincronización no se iniciaba correctamente hasta que hubiera abandonado la clave de cifrado y reinicializado la contraseña de la cuenta del servicio de sincronización de Azure AD Connect. Ahora, este proceso ya no es necesario.

SSO de escritorio

* El Asistente para Azure AD Connect ya no requiere que el puerto 9090 esté abierto en la red al configurar la autenticación de paso a través y SSO de escritorio. Solo se requiere el puerto 443. 

## <a name="114430"></a>1.1.443.0
Fecha de publicación: Marzo de 2017

**Problemas corregidos:**

Sincronización de Azure AD Connect
* Se ha corregido un problema que hace el Asistente para Azure AD Connect deje de funcionar si el nombre para mostrar del conector de Azure AD no contiene el dominio inicial onmicrosoft.com que se asignan al inquilino de Azure AD.
* Se ha corregido un problema que hace que el Asistente para Azure AD Connect genere un error al realizar la conexión a la base de datos SQL cuando la contraseña de la cuenta de servicio de sincronización tenga caracteres especiales, como un apóstrofo, una coma y un espacio.
* Se corrigió un problema que genera el error "la imagen tiene un delimitador que es diferente de la imagen" que se produzca en un servidor de Azure AD Connect en modo de ensayo después de que ha excluido temporalmente un uso local de la sincronización de objetos de AD y, a continuación, lo incluye de nuevo para la sincronización.
* Se corrigió un problema que hace que el error que indica que el objeto que ha localizado DN es un fantasma se produzca en un servidor de Azure AD Connect en modo de almacenamiento provisional, una vez que ha excluido temporalmente un objeto AD de sincronización local de la sincronización y, luego, lo incluye de nuevo para realizar la sincronización.

Administración de AD FS
* Se ha corregido un problema donde el Asistente para Azure AD Connect no actualizar la configuración de AD FS y establece las notificaciones adecuadas en el usuario de confianza después de configurar el identificador de inicio de sesión alternativo.
* Se ha corregido un problema donde el Asistente para Azure AD Connect no puede administrar correctamente los servidores de AD FS cuyas cuentas de servicio se configuran mediante el formato userPrincipalName en lugar de sAMAccountName.

Autenticación de paso a través
* Se ha corregido un problema que hace que el Asistente para Azure AD Connect deje de funcionar si se selecciona la autenticación de paso a través, pero se produce un error de registro de su conector.
* Se ha corregido un problema que hace que el Asistente para Azure AD Connect pase por alto las comprobaciones de validación en el método de inicio de sesión seleccionado cuando se habilita la característica de SSO de escritorio.

Restablecimiento de contraseña
* Se ha corregido un problema que puede hacer que el servidor de Azure AAD Connect no intente volver a conectarse si un firewall o un proxy termina la conexión.

**Nuevas características y mejoras:**

Sincronización de Azure AD Connect
* El cmdlet Get-ADSyncScheduler ahora devuelve una nueva propiedad booleana denominada "SyncCycleInProgress". Si el valor devuelto es True, significa que hay un ciclo de sincronización programada en curso.
* La carpeta de destino para almacenar la instalación de Azure AD Connect y los registros de instalación se han movido desde %localappdata%\AADConnect a %programdata%\AADConnect para mejorar la accesibilidad a los archivos de registro.

Administración de AD FS
* Se agregó compatibilidad para actualizar el certificado SSL de granja de servidores de AD FS.
* Se agregó compatibilidad para administrar AD FS 2016.
* Ahora puede especificar gMSA existente (cuenta de servicio administrada de grupo) durante la instalación de AD FS.
* Ahora puede configurar SHA-256 como el algoritmo de hash de firma para usuarios de confianza de Azure AD.

Restablecimiento de contraseña
* Se han introducido mejoras para permitir que el producto funcione en entornos con reglas de firewall más estrictas.
* Se ha mejorado la confiabilidad de la conexión a Azure Service Bus.

## <a name="113800"></a>1.1.380.0
Fecha de publicación: Diciembre de 2016

**Problema corregido:**

* Se ha corregido el problema por el cual falta la regla de notificación de issuerid para Active Directory Federation Services (AD FS) en esta compilación.

>[!NOTE]
>Esta compilación no está disponible para los clientes a través de la característica de actualización automática de Azure AD Connect.

## <a name="113710"></a>1.1.371.0
Fecha de publicación: Diciembre de 2016

**Problema conocido:**

* Falta la regla de notificación de issuerid para AD FS en esta compilación. La regla de notificación de issuerid es necesaria si se está realizando una federación de varios dominios con Azure Active Directory (Azure AD). Si utiliza Azure AD Connect para administrar la implementación de AD FS local, la actualización a esta compilación quitará la regla de notificación de issuerid existente de la configuración de AD FS. Puede solucionar el problema agregando la regla de notificación de issuerid después de la instalación o actualización. Para más información sobre cómo agregar la regla de notificación de issuerid, consulte el artículo [Compatibilidad con varios dominios para la federación con Azure AD](how-to-connect-install-multiple-domains.md).

**Problema corregido:**

* Si el puerto 9090 no está abierto para las conexiones salientes, la instalación o actualización de Azure AD Connect producirá un error.

>[!NOTE]
>Esta compilación no está disponible para los clientes a través de la característica de actualización automática de Azure AD Connect.

## <a name="113700"></a>1.1.370.0
Fecha de publicación: Diciembre de 2016

**Problemas conocidos**:

* Falta la regla de notificación de issuerid para AD FS en esta compilación. La regla de notificación de issuerid es necesaria si se está realizando una federación de varios dominios con Azure AD. Si utiliza Azure AD Connect para administrar la implementación de AD FS local, la actualización a esta compilación quitará la regla de notificación de issuerid existente de la configuración de AD FS. Puede solucionar el problema agregando la regla de notificación de issuerid después de la instalación o actualización. Para más información sobre cómo agregar la regla de notificación de issuerid, consulte el artículo [Compatibilidad con varios dominios para la federación con Azure AD](how-to-connect-install-multiple-domains.md).
* El puerto 9090 debe estar abierto para las conexiones salientes a fin de completar la instalación.

**Nuevas características:**

* Autenticación de paso a través (versión preliminar)

>[!NOTE]
>Esta compilación no está disponible para los clientes a través de la característica de actualización automática de Azure AD Connect.

## <a name="113430"></a>1.1.343.0
Fecha de publicación: Noviembre de 2016

**Problema conocido:**

* Falta la regla de notificación de issuerid para AD FS en esta compilación. La regla de notificación de issuerid es necesaria si se está realizando una federación de varios dominios con Azure AD. Si utiliza Azure AD Connect para administrar la implementación de AD FS local, la actualización a esta compilación quitará la regla de notificación de issuerid existente de la configuración de AD FS. Puede solucionar el problema agregando la regla de notificación de issuerid después de la instalación o actualización. Para más información sobre cómo agregar la regla de notificación de issuerid, consulte el artículo [Compatibilidad con varios dominios para la federación con Azure AD](how-to-connect-install-multiple-domains.md).

**Problemas corregidos:**

* A veces, Azure AD Connect no se puede instalar porque no puede crear una cuenta de servicio local cuya contraseña cumpla el nivel de complejidad especificado por la directiva de contraseñas de la organización.
* Se corrigió un problema por el que las reglas de unión no se vuelven a evaluar si un objeto en el espacio del conector se convierte simultáneamente en un objeto fuera de ámbito para una regla de unión y dentro de ámbito para otra regla. Esto puede ocurrir si tiene dos o más reglas de unión cuyas condiciones de unión son mutuamente excluyentes.
* Se ha corregido un problema por el que no se procesan las reglas de sincronización entrantes (desde Azure AD) que no contienen reglas de unión si tienen valores de prioridad inferiores que aquellas que sí que las contienen.

**Mejoras:**

* Se agregó compatibilidad para la instalación de Azure AD Connect en Windows Server 2016 estándar o superior.
* Se agregó compatibilidad para utilizar SQL Server 2016 como la base de datos remota para Azure AD Connect.

## <a name="112810"></a>1.1.281.0
Fecha de publicación: Agosto de 2016

**Problemas corregidos:**

* Los cambios en el intervalo de sincronización no se realizarán hasta que finalice el siguiente ciclo de sincronización.
* El Asistente de Azure AD Connect no acepta cuentas de Azure AD cuyo nombre de usuario comience con un guion bajo (\_).
* Si la contraseña contiene demasiados caracteres especiales, se producirá un error en el Asistente de Azure AD Connect al autenticar la cuenta de Azure AD. Se mostrará el mensaje de error: No se pueden validar las credenciales. Se produjo un error inesperado" .
* Al desinstalar el servidor de ensayo, se deshabilita la sincronización de contraseñas del inquilino de Azure AD y se produce un error de sincronización de contraseñas en el servidor activo.
* Se produce un error de sincronización de contraseñas en casos raros cuando no hay ningún valor de hash de contraseña almacenado en el usuario.
* Cuando se habilita el servidor de Azure AD Connect para el modo provisional, la escritura diferida de contraseñas se deshabilita temporalmente.
* El Asistente de Azure AD Connect no muestra la sincronización de contraseñas y la configuración de escritura diferida de contraseñas reales cuando el servidor está en modo provisional. Siempre se muestran como deshabilitadas.
* Los cambios de configuración en la sincronización de contraseñas y la escritura diferida de contraseñas no se almacenan en el Asistente de Azure AD Connect cuando el servidor está en modo provisional.

**Mejoras:**

* Se ha actualizado el cmdlet Start-ADSyncSyncCycle para indicar si se puede iniciar correctamente un nuevo ciclo de sincronización.
* Se ha agregado el cmdlet Stop-ADSyncSyncCycle para finalizar el ciclo de sincronización y la operación actualmente en curso.
* Se ha actualizado el cmdlet Stop-ADSyncScheduler para finalizar el ciclo de sincronización y la operación actualmente en curso.
* Al configurar [extensiones de directorio](how-to-connect-sync-feature-directory-extensions.md) en el Asistente de Azure AD Connect, ahora se podrá seleccionar el atributo de Azure AD de tipo cadena Teletexto.

## <a name="111890"></a>1.1.189.0
Fecha de publicación: Junio de 2016

**Problemas corregidos y mejoras:**

* Azure AD Connect ahora puede instalarse en un servidor conforme a FIPS.
  * Para la sincronización de contraseñas, consulte [Sincronización de hash de contraseñas y FIPS](how-to-connect-password-hash-synchronization.md#password-hash-synchronization-and-fips).
* Se ha corregido un problema por el cual un nombre NetBIOS no se pudo resolver en el FQDN en el conector de Active Directory.

## <a name="111800"></a>1.1.180.0
Fecha de publicación: Mayo de 2016

**Nuevas características:**

* Le advierte y ayuda a comprobar los dominios, si no lo hizo antes de ejecutar Azure AD Connect.
* Se ha agregado compatibilidad con [Microsoft Cloud Germany](reference-connect-instances.md#microsoft-cloud-germany).
* Se ha agregado compatibilidad con la versión más reciente de la infraestructura de [Microsoft Azure Government Cloud](reference-connect-instances.md#microsoft-azure-government-cloud) con los nuevos requisitos de dirección URL.

**Problemas corregidos y mejoras:**

* Agrega el filtrado en el Editor de reglas de sincronización para facilitar la búsqueda de reglas de sincronización.
* Mejora el rendimiento en la eliminación de un espacio de conector.
* Se corrigió un problema cuando el mismo objeto se eliminaba y se agregaba en la misma ejecución (llamado eliminar o agregar).
* Una regla de sincronización deshabilitada ya no vuelve a habilitar objetos y atributos incluidos durante la actualización del esquema de directorio o una actualización.

## <a name="111300"></a>1.1.130.0
Fecha de publicación: abril de 2016

**Nuevas características:**

* Se ha agregado compatibilidad con atributos multivalor en las [extensiones de directorio](how-to-connect-sync-feature-directory-extensions.md).
* Se ha agregado compatibilidad con más variaciones de configuración para que la [actualización automática](how-to-connect-install-automatic-upgrade.md) se considere apta para la actualización.
* Se han agregado algunos cmdlets para el [programador personalizado](how-to-connect-sync-feature-scheduler.md#custom-scheduler).

## <a name="111190"></a>1.1.119.0
Fecha de publicación: marzo de 2016

**Problemas corregidos:**

* Nos aseguramos de que la instalación rápida no se pueda usar en Windows Server 2008 (versión preliminar 2) porque la sincronización de contraseñas no es compatible con este sistema operativo.
* La actualización desde DirSync con una configuración de filtro personalizada no funcionó como se esperaba.
* Cuando se actualiza a una versión más reciente y no hay ningún cambio en la configuración, no se debe programar una importación o sincronización completa.

## <a name="111100"></a>1.1.110.0
Fecha de publicación: Febrero de 2016

**Problemas corregidos:**

* La actualización desde versiones anteriores no funciona si la instalación no está en la carpeta predeterminada C:\Archivos de programa.
* Si efectúa la instalación y anula la selección de **Inicie el proceso de sincronización** al final del asistente para instalación, el programador no se habilitará al volver a ejecutar el asistente.
* El programador no funciona según lo previsto en los servidores en los que no se use el formato de fecha y hora US-en. Además, impedirá que `Get-ADSyncScheduler` devuelva las horas correctas.
* Si ha instalado una versión anterior de Azure AD Connect con AD FS como opción de inicio de sesión y actualización, no puede volver a ejecutar el asistente para instalación.

## <a name="111050"></a>1.1.105.0
Fecha de publicación: Febrero de 2016

**Nuevas características:**

* [Automatic upgrade](how-to-connect-install-automatic-upgrade.md) para los clientes de configuración rápida.
* Compatibilidad con el administrador global mediante Azure Multi-Factor Authentication y Privileged Identity Management en el asistente para la instalación.
  * Si utiliza Multi-Factor Authentication, debe permitir que el servidor proxy también permita el tráfico a https://secure.aadcdn.microsoftonline-p.com.
  * Debe agregar https://secure.aadcdn.microsoftonline-p.com a la lista de sitios de confianza para que Multi-Factor Authentication funcione correctamente.
* Permite cambiar el método de inicio de sesión del usuario después de la instalación inicial.
* Permita el [filtrado de dominios y unidades organizativas](how-to-connect-install-custom.md#domain-and-ou-filtering) en el Asistente para instalación. Esto también permite conectar con bosques donde no todos los dominios están disponibles.
* [Scheduler](how-to-connect-sync-feature-scheduler.md) está integrado en el motor de sincronización.

**Características promocionadas desde la vista previa a GA:**

* [Escritura diferida de dispositivos](how-to-connect-device-writeback.md)
* [Extensiones de directorio](how-to-connect-sync-feature-directory-extensions.md)

**Nuevas características de la versión preliminar:**

* El nuevo intervalo de ciclo de sincronización predeterminado es de 30 minutos. Solía ser tres horas en todas las versiones anteriores. Agrega compatibilidad para cambiar el comportamiento del [programador](how-to-connect-sync-feature-scheduler.md) .

**Problemas corregidos:**

* La página de comprobación de dominios DNS no siempre reconocía los dominios.
* Solicita las credenciales de administrador de dominio al configurar AD FS.
* El Asistente para instalación no reconoce las cuentas de AD locales si están ubicadas en un dominio con un árbol DNS diferente al dominio raíz.

## <a name="1091310"></a>1.0.9131.0
Fecha de publicación: diciembre de 2015

**Problemas corregidos:**

* La sincronización de contraseñas podría no funcionar al cambiar las contraseñas en Active Directory Domain Services (AD DS), pero funciona al establecer una contraseña.
* Con un servidor proxy, la autenticación en Azure AD puede producir errores durante la instalación o si se cancela una actualización en la página de configuración.
* La actualización desde una versión anterior de Azure AD Connect con una instalación completa de una instancia de SQL Server produce errores si no es administrador del sistema de SQL Server (SA).
* La actualización desde una versión anterior de Azure AD Connect con una instalación remota de SQL Server muestra el error "Unable to access the ADSync SQL database" (No se puede acceder a la base de datos SQL de ADSync).

## <a name="1091250"></a>1.0.9125.0
Fecha de publicación: Noviembre de 2015

**Nuevas características:**

* Puede volver a configurar AD FS para la confianza de Azure AD.
* Puede actualizar el esquema de Active Directory y volver a generar reglas de sincronización.
* Puede deshabilitar una regla de sincronización.
* Puede definir "AuthoritativeNull" como un nuevo literal en una regla de sincronización.

**Nuevas características de la versión preliminar:**

* [Azure AD Connect Health para sincronización](how-to-connect-health-sync.md)
* Compatibilidad para sincronización de contraseñas de [Servicios de dominio de Azure AD](../user-help/active-directory-passwords-update-your-own-password.md) .

**Nuevo escenarios admitido:**

* Admite varias organizaciones de Exchange locales. Consulte [Implementaciones híbridas con varios bosques de Active Directory](https://technet.microsoft.com/library/jj873754.aspx) para más información.

**Problemas corregidos:**

* Problemas de sincronización de contraseñas:
  * Un objeto movido desde fuera de ámbito a dentro de ámbito no tendrá su contraseña sincronizada. Esto incluye tanto UO como el filtrado de atributos.
  * La selección de una nueva UO para incluir en sincronización no requiere una sincronización de contraseñas completa.
  * Cuando un usuario deshabilitado se habilita, la contraseña no se sincroniza.
  * La cola de reintentos de contraseña es infinita y el límite anterior de 5.000 objetos para retirar anterior se ha quitado.
* No se puede conectar con Active Directory con el nivel funcional de bosque de Windows Server 2016.
* No se puede cambiar el grupo usado para el filtrado de grupo tras la instalación inicial.
* Ya no se crea un nuevo perfil de usuario en el servidor de Azure AD Connect para cada usuario al hacer un cambio de contraseña con la escritura diferida de contraseñas habilitada.
* No se pueden usar valores enteros largos en ámbitos de reglas de sincronización.
* La casilla de verificación "Reescritura de dispositivos" permanece deshabilitada si hay controladores de dominio inalcanzables.

## <a name="1086670"></a>1.0.8667.0
Fecha de publicación: Agosto de 2015

**Nuevas características:**

* Ahora, el asistente para la instalación de Azure AD Connect se adapta a todos los idiomas de Windows Server.
* Se ha agregado compatibilidad con el desbloqueo de cuentas cuando se usa la administración de contraseñas de Azure AD.

**Problemas corregidos:**

* El asistente para la instalación de Azure AD Connect se bloquea si otro usuario continúa la instalación, y no la persona que la inició por primera vez.
* Si una desinstalación anterior de Azure AD Connect no desinstala limpiamente la sincronización de Azure AD Connect, no es posible volver a instalarlo.
* No se puede instalar Azure AD Connect mediante la instalación rápida si el usuario no está en el dominio raíz del bosque o si se usa una versión distinta del inglés de Active Directory.
* Si no se puede resolver el FQDN de la cuenta de usuario de Active Directory, se muestra un mensaje de error engañoso para indicar que no se pudo confirmar el esquema.
* Si se cambia la cuenta usada en el conector de Active Directory fuera del asistente, este producirá errores en ejecuciones posteriores.
* Azure AD Connect no se puede instalar en ocasiones en un controlador de dominio.
* No se puede habilitar y deshabilitar el "Modo provisional" si se han agregado atributos de extensión.
* La escritura diferida de contraseñas produce errores en alguna configuración debido a una contraseña incorrecta en el conector de Active Directory.
* No se puede actualizar la sincronización de directorios si se usa en el filtrado de atributos el nombre distintivo (DN).
* Uso excesivo de CPU al utilizar el restablecimiento de contraseña.

**Características de versión la preliminar eliminadas:**

* La característica en vista previa [Reescritura de usuarios](how-to-connect-preview.md#user-writeback) se ha eliminado temporalmente a raíz de los comentarios de nuestros clientes de vista previa. Se volverá a agregar después de que se hayan examinado los comentarios proporcionados.

## <a name="1086410"></a>1.0.8641.0
Fecha de publicación: Junio de 2015

**Versión inicial de Azure AD Connect.**

Ha cambiado el nombre de Azure AD Sync a Azure AD Connect.

**Nuevas características:**

* Instalación de la [configuración rápida](how-to-connect-install-express.md)
* Posibilidad de [configurar AD FS](how-to-connect-install-custom.md#configuring-federation-with-ad-fs)
* Posibilidad de [actualizar desde DirSync](how-to-dirsync-upgrade-get-started.md)
* [Evitar eliminaciones accidentales](how-to-connect-sync-feature-prevent-accidental-deletes.md)
* Se ha introducido el [modo de ensayo](how-to-connect-sync-staging-server.md)

**Nuevas características de la versión preliminar:**

* [Reescritura de usuarios](how-to-connect-preview.md#user-writeback)
* [Escritura diferida de grupos](how-to-connect-preview.md#group-writeback)
* [Escritura diferida de dispositivos](how-to-connect-device-writeback.md)
* [Extensiones de directorio](how-to-connect-preview.md)

## <a name="104940501"></a>1.0.494.0501
Fecha de publicación: Mayo de 2015

**Nuevo requisito**

* Ahora, Azure AD Sync requiere que se instale la versión 4.5.1 de .NET Framework.

**Problemas corregidos:**

* La escritura diferida de contraseñas de Azure AD produce errores de conectividad de Service Bus.

## <a name="104910413"></a>1.0.491.0413
Fecha de publicación: Abril de 2015

**Problemas corregidos y mejoras:**

* El conector de Active Directory no procesa las eliminaciones correctamente si está habilitada la Papelera de reciclaje y hay varios dominios en el bosque.
* Se ha mejorado el rendimiento de las operaciones de importación para el conector de Azure Active Directory.
* Cuando un grupo superaba el límite de pertenencia (de forma predeterminada, el límite se establece en 50 000 objetos), el grupo se eliminaba de Azure Active Directory. Con el nuevo comportamiento, no se elimina el grupo, se produce un error y no se exportan los cambios de pertenencia.
* No se puede aprovisionar un nuevo objeto si una eliminación preconfigurada con el mismo DN ya está presente en el espacio del conector.
* Algunos objetos se marcan para que se sincronicen durante una sincronización delta, aunque no haya ningún cambio preconfigurado en el objeto.
* Forzar una sincronización de contraseñas también elimina la lista preferida de controladores de dominio.
* CSExportAnalyzer tiene problemas con algunos estados de objetos.

**Nuevas características:**

* Una combinación puede conectarse ahora a "CUALQUIER" tipo de objeto en la MV.

## <a name="104850222"></a>1.0.485.0222
Fecha de publicación: Febrero de 2015

**Mejoras:**

* Rendimiento de importación mejorada.

**Problemas corregidos:**

* La sincronización de contraseñas respeta el atributo cloudFiltered que usa el filtrado de atributos. Los objetos filtrados ya no pertenecen al ámbito de la sincronización de contraseñas.
* En las raras ocasiones donde la topología tenía muchos controladores de dominio, la sincronización de contraseñas no funcionaba.
* Se ha habilitado en Azure AD/Intune "Servidor detenido" al importar desde el conector de Azure AD después de la administración de dispositivos.
* La unión de entidades de seguridad externas (FSP) desde varios dominios del mismo bosque produce un error de unión ambigua.

## <a name="104751202"></a>1.0.475.1202
Fecha de publicación: Diciembre de 2014

**Nuevas características:**

* Ahora es posible realizar la sincronización de contraseñas con filtrado basado en atributos. Para más información, consulte el artículo sobre la [sincronización de contraseñas con filtrado](how-to-connect-sync-configure-filtering.md).
* El atributo ms-DS-ExternalDirectoryObjectID se reescribe en Active Directory. Esta característica agrega compatibilidad para las aplicaciones de Office 365. Utiliza OAuth2 para acceder a los buzones de correo en línea y locales en una implementación híbrida de Exchange.

**Problemas de actualización corregidos:**

* Hay una versión más reciente del ayudante de inicio de sesión en el servidor.
* Se usó una ruta de acceso de instalación personalizada para instalar Azure AD Sync.
* Un criterio de combinación personalizado no válido bloquea la actualización.

**Otras correcciones:**

* Se han corregido las plantillas para Office Pro Plus.
* Se han corregido los problemas de instalación ocasionados por nombres de usuario que comienzan con un guión.
* Se ha corregido la pérdida de la configuración de sourceAnchor cuando se ejecuta al asistente para la instalación por una segunda vez.
* Se ha corregido el seguimiento ETW para la sincronización de contraseñas.

## <a name="104701023"></a>1.0.470.1023
Fecha de publicación: Octubre de 2014

**Nuevas características:**

* Sincronización de contraseñas desde varias instancias de Active Directory locales a Azure AD.
* Interfaz de usuario de instalación localizada para todos los idiomas de Windows Server.

**Actualización de AADSync 1.0 GA**

Si ya tiene instalado Azure AD Sync, hay un paso adicional que debe seguir en caso de que haya cambiado alguna de las reglas de sincronización inmediata. Después de haber actualizado a la versión 1.0.470.1023, las reglas de sincronización que ha modificado se duplican. Para cada regla de sincronización modificada, haga lo siguiente:

1. Busque la regla de sincronización modificada y anote los cambios.
1. Elimine la regla de sincronización.
1. Busque la nueva regla de sincronización creada por Azure AD Sync y vuelva a aplicar los cambios.

**Permisos para la cuenta de Active Directory**

Se deben conceder permisos adicionales a la cuenta de Active Directory para poder leer los hash de contraseña de Active Directory. Los permisos que se deben conceder se denominan "Replicating Directory Changes" (Replicar cambios de directorio) y "Replicating Directory Changes All" (Replicar todos los cambios de directorio). Ambos permisos son necesarios para poder leer los hash de contraseña.

## <a name="104190911"></a>1.0.419.0911
Fecha de publicación: Septiembre de 2014

**Versión inicial de Azure AD Sync.**

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).
