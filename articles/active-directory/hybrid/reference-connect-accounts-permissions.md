---
title: 'Azure AD Connect: Cuentas y permisos | Microsoft Docs'
description: En este tema se describen las cuentas usadas y creadas, así como los permisos necesarios.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.reviewer: cychua
ms.assetid: b93e595b-354a-479d-85ec-a95553dd9cc2
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 05/18/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1ccffe5e6e12d828acb4ced46bf3abf36113c588
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276497"
---
# <a name="azure-ad-connect-accounts-and-permissions"></a>Azure AD Connect: Cuentas y permisos

## <a name="accounts-used-for-azure-ad-connect"></a>Cuentas utilizadas para Azure AD Connect

![información general sobre las cuentas](media/reference-connect-accounts-permissions/account5.png)

Azure AD Connect usa 3 cuentas para sincronizar la información de un entorno local o de Windows Server Active Directory con Azure Active Directory.  Estas cuentas son:

- **Cuenta del conector de AD DS**: utilizada para leer y escribir información en Windows Server Active Directory

- **Cuenta del servicio ADSync**: utilizada para ejecutar el servicio de sincronización y acceder a la base de datos SQL

- **Cuenta del conector de Azure AD**: utilizada para escribir información en Azure AD

Además de estas tres cuentas usadas para ejecutar Azure AD Connect, también necesitará las siguientes cuentas adicionales para instalar Azure AD Connect.  Dichos componentes son:

- **Cuenta de administrador local**: Administrador que está instalando Azure AD Connect y que tiene permisos de administrador local en el equipo.

- **Cuenta de administrador de empresa de AD DS**: Se usa de manera opcional para crear la "cuenta de conector de AD DS" anterior.

- **Cuenta de administrador global de Azure AD**: utilizada para crear la cuenta del conector de Azure AD y configurar Azure AD.

- **Cuenta de SA de SQL (opcional)** : usada para crear la base de datos ADSync cuando se usa la versión completa de SQL Server.  Este servidor SQL Server puede ser local o remoto a la instalación de Azure AD Connect.  Esta cuenta puede ser la misma cuenta que la del administrador de empresa.  El administrador de SQL puede realizar ahora el aprovisionamiento de la base de datos fuera de banda y luego el administrador de Azure AD Connect puede instalarla con derechos de propietario de la base de datos.  Para más información, consulte [Instalación de Azure AD Connect con permisos de administrador delegado de SQL](how-to-connect-install-sql-delegation.md)


>[!IMPORTANT]
> A partir de la compilación 1.4.###.#, ya no se admite el uso de una cuenta de administrador de organización o de administrador de dominio como cuenta de AD DS Connector.  Si intenta especificar una cuenta que sea administrador de organización o administrador de dominio al especificar **usar cuenta existente**, recibirá un error.

> [!NOTE]
> Se admite la administración de las cuentas administrativas que se usan en Azure AD Connect desde un bosque administrativo ESAE (también conocido como "bosque rojo").
> Los bosques administrativos dedicados permiten a las organizaciones hospedar cuentas administrativas, estaciones de trabajo y grupos en un entorno que tiene controles de seguridad más seguros que el entorno de producción.
> Para más información acerca de los bosques administrativos dedicados, consulte [Enfoque de diseño de bosque administrativo ESAE](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material#esae-administrative-forest-design-approach).

> [!NOTE]
> El rol de administrador global no es necesario después de la configuración inicial y la única cuenta requerida será la cuenta de rol **Cuentas de sincronización de directorios** . Eso no significa necesariamente que solo desee eliminar la cuenta con el rol de Administrador global. Es mejor cambiar el rol a un rol menos poderoso, ya que la eliminación total de la cuenta puede presentar problemas si alguna vez necesita volver a ejecutar el asistente. Al reducir el privilegio del rol, siempre puede volver a elevar los privilegios si tiene que utilizar el asistente Azure AD Connect nuevamente. 

## <a name="installing-azure-ad-connect"></a>Instalación de Azure AD Connect
El asistente para instalación de Azure AD Connect ofrece dos itinerarios diferentes:

* En la Configuración rápida, el asistente necesita más privilegios.  Esto es para que pueda establecer la configuración con facilidad, sin necesidad de crear usuarios ni de configurar permisos.
* En la Configuración personalizada, el asistente ofrece más opciones. Sin embargo, existen algunas situaciones en que debe asegurarse de que tiene los permisos correctos.



## <a name="express-settings-installation"></a>Instalación de la configuración rápida
En la configuración rápida, el asistente para instalación solicita lo siguiente:

  - Credenciales del administrador de empresa de AD DS
  - Credenciales del administrador global de Azure AD

### <a name="ad-ds-enterprise-admin-credentials"></a>Credenciales del administrador de empresa de AD DS
La cuenta de administrador de empresa de AD DS se usa para configurar Active Directory local. Estas credenciales solo se usan durante la instalación y no se usan una vez completada la misma. El administrador de empresa, y no el del dominio, debe asegurarse de que se pueden establecer los permisos de Active Directory en todos los dominios.

Si va a actualizar desde DirSync, las credenciales de administradores de organización de AD DS se usan para restablecer la contraseña de la cuenta utilizada por DirSync. También necesitará credenciales de administrador global de Azure AD.

### <a name="azure-ad-global-admin-credentials"></a>Credenciales de administrador global de Azure AD
Estas credenciales solo se usan durante la instalación y no se usan una vez completada la misma. Se utiliza para crear la cuenta del conector de Azure AD usada para sincronizar los cambios con Azure AD. La cuenta también habilita la sincronización como una característica de Azure AD.

### <a name="ad-ds-connector-account-required-permissions-for-express-settings"></a>Permisos necesarios de la cuenta del conector de AD DS para la configuración rápida
La cuenta del conector de AD DS se crea para leer y escribir en Windows Server AD y tiene los permisos siguientes cuando se crea en la configuración rápida:

| Permiso | Se usa para |
| --- | --- |
| <li>Replicación de cambios de directorio</li><li>Replicación de todos los cambios de directorio |Sincronización de hash de contraseñas |
| Lectura y escritura de todas las propiedades Usuario |Importación y Exchange híbrido |
| Lectura y escritura de todas las propiedades iNetOrgPerson |Importación y Exchange híbrido |
| Lectura y escritura de todas las propiedades Grupo |Importación y Exchange híbrido |
| Lectura y escritura de todas las propiedades Contacto |Importación y Exchange híbrido |
| Restablecimiento de contraseña |Preparación para habilitar la escritura diferida de contraseñas |

### <a name="express-installation-wizard-summary"></a>Resumen del asistente para instalación rápida

![Instalación rápida](./media/reference-connect-accounts-permissions/express.png)

A continuación se ofrece un resumen de las páginas del asistente para instalación rápida, las credenciales recopiladas y para qué se utilizan.

| Página del asistente | Credenciales recopiladas | Permisos necesarios | Se usa para |
| --- | --- | --- | --- |
| N/D |Usuario que ejecuta el asistente para instalación |Administrador del servidor local |<li>Crea la cuenta del servicio ADSync que se usa para ejecutar el servicio de sincronización. |
| Conectarse a Azure |Credenciales de directorio de Azure AD |Rol de administrador global en Azure AD |<li>Habilitación de la sincronización en el directorio de Azure AD</li>  <li>Creación de la cuenta del conector de Azure AD que se usa para las operaciones de sincronización en curso en Azure AD.</li> |
| Conectarse a AD DS |Credenciales de Active Directory local |Miembro del grupo de administradores de empresa (EA) en Active Directory |<li>Crea la cuenta del conector de AD DS en Active Directory y concede permisos en la misma. Esta cuenta creada se usa para leer y escribir información de directorio durante la sincronización.</li> |


## <a name="custom-installation-settings"></a>Configuración de la instalación personalizada

Con la configuración de la instalación personalizada, el asistente ofrece más opciones. 

### <a name="custom-installation-wizard-summary"></a>Resumen del asistente para instalación personalizada

A continuación se ofrece un resumen de las páginas del asistente para instalación personalizada, las credenciales recopiladas y para qué se utilizan.

![Instalación rápida](./media/reference-connect-accounts-permissions/customize.png)

| Página del asistente | Credenciales recopiladas | Permisos necesarios | Se usa para |
| --- | --- | --- | --- |
| N/D |Usuario que ejecuta el asistente para instalación |<li>Administrador del servidor local</li><li>Si usa SQL Server completo, el usuario debe ser administrador del sistema (SA) en SQL.</li> |De forma predeterminada, crea la cuenta local que se usa como cuenta de servicio del motor de sincronización. La cuenta solo se crea cuando el administrador no especifica una cuenta determinada. |
| Instalar servicios de sincronización, opción de cuenta de servicio |Credenciales de cuenta de usuario local o de AD |Usuario, el asistente para instalación concede los permisos |Si el administrador especifica una cuenta, esta se usa como cuenta de servicio para el servicio de sincronización. |
| Conectarse a Azure |Credenciales de directorio de Azure AD |Rol de administrador global en Azure AD |<li>Habilitación de la sincronización en el directorio de Azure AD</li>  <li>Creación de la cuenta del conector de Azure AD que se usa para las operaciones de sincronización en curso en Azure AD.</li> |
| Conectar sus directorios |Credenciales de Active Directory local para cada bosque que se conecta a Azure AD |Los permisos dependen de las características que se habiliten y se pueden encontrar en Creación de la cuenta del conector de AD DS |Esta cuenta se usa para leer y escribir información de directorio durante la sincronización. |
| Servidores de AD FS |Para cada servidor de la lista, el asistente recopila credenciales cuando las credenciales de inicio de sesión del usuario que ejecuta el asistente no son suficientes para conectarse. |Administrador de dominio |Instalación y configuración del rol de servidor de AD FS. |
| Servidores proxy de aplicación web |Para cada servidor de la lista, el asistente recopila credenciales cuando las credenciales de inicio de sesión del usuario que ejecuta el asistente no son suficientes para conectarse. |Administrador local en la máquina de destino |Instalación y configuración del rol de servidor de WAP. |
| Credenciales de confianza del proxy |Credenciales de confianza del servicio de federación (las credenciales que el proxy usa para inscribirse para obtener un certificado de confianza de FS) |Cuenta de dominio que es un administrador local del servidor de AD FS |Inscripción inicial de certificados de confianza de FS WAP |
| Página de cuenta de servicio de AD FS, "Usar una opción de cuenta de usuario de dominio" |Credenciales de cuenta de usuario de AD |Usuario de dominio |La cuenta de usuario de Azure AD cuyas credenciales se especifiquen se usa como cuenta de inicio de sesión del servicio AD FS. |

### <a name="create-the-ad-ds-connector-account"></a>Creación de la cuenta del conector de AD DS

>[!IMPORTANT]
>Se introdujo un nuevo módulo de PowerShell denominado ADSyncConfig.psm1 con la compilación **1.1.880.0** (publicada en agosto de 2018) que incluye una colección de cmdlets que le ayudarán a configurar los permisos correctos de Active Directory para su cuenta de Azure AD DS.
>
>Para más información, consulte [Azure AD Connect: Configurar los permisos de cuenta del conector de AD DS](how-to-connect-configure-ad-ds-connector-account.md).

La cuenta especificada en la página **Conectar sus directorios** debe estar presente en Active Directory antes de la instalación.  Azure AD Connect 1.1.524.0 y las versiones posteriores ofrecen la opción de permitir que el asistente de Azure AD Connect cree la **cuenta del conector de AD DS** utilizada para conectarse a Active Directory.  

También debe tener los permisos necesarios concedidos. El asistente para instalación no comprueba los permisos y los problemas solo se encuentran durante la sincronización.

Los permisos que requiera dependen de las características opcionales que habilite. Si tiene varios dominios, se deben conceder los permisos para todos los dominios del bosque. Si no habilita ninguna de estas características, los permisos **Usuario de dominio** predeterminados son suficientes.

| Característica | Permisos |
| --- | --- |
| característica ms-DS-ConsistencyGuid |Permisos de escritura para el atributo msDS-ConsistencyGuid documentado en [Conceptos de diseño: Uso de msDS-ConsistencyGuid como sourceAnchor](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor). | 
| Sincronización de hash de contraseñas |<li>Replicación de cambios de directorio</li>  <li>Replicación de todos los cambios de directorio |
| Implementación híbrida de Exchange |Permisos de escritura en los atributos que se documentan en [Escritura diferida híbrida de Exchange](reference-connect-sync-attributes-synchronized.md#exchange-hybrid-writeback) para usuarios, grupos y contactos. |
| Carpeta pública de correo de Exchange |Permisos de lectura para los atributos que se documentan en [carpetas públicas de correo electrónico de Exchange](reference-connect-sync-attributes-synchronized.md#exchange-mail-public-folder) para las carpetas públicas. | 
| escritura diferida de contraseñas |Permisos de escritura en los atributos que se documentan en [Introducción a la administración de contraseñas](../authentication/tutorial-enable-sspr-writeback.md) para los usuarios. |
| Escritura diferida de dispositivos |Los permisos concedidos con un script de PowerShell como se describe en [Escritura diferida de dispositivos](how-to-connect-device-writeback.md). |
| Escritura diferida de grupos |Permite la escritura diferida de **grupos de Office 365** en un bosque con Exchange instalado.|

## <a name="upgrade"></a>Actualizar
Al actualizar desde una versión de Azure AD Connect a una nueva versión, necesita los siguientes permisos:

>[!IMPORTANT]
>A partir de la compilación 1.1.484, Azure AD Connect introdujo un error de regresión que requiere permisos de administrador del sistema para actualizar la base de datos SQL.  Este error se corrigió en la compilación 1.1.647.  Si va a actualizar a esta versión, necesitará permisos de administrador del sistema.  Los permisos de dbo no son suficientes.  Si intenta actualizar Azure AD Connect sin tener permisos de administrador del sistema, se producirá un error en la actualización y Azure AD Connect dejará de funcionar correctamente después.  Microsoft conoce este problema y está trabajando para corregir este problema.


| Principal | Permisos necesarios | Se usa para |
| --- | --- | --- |
| Usuario que ejecuta el asistente para instalación |Administrador del servidor local |Archivos binarios de la actualización. |
| Usuario que ejecuta el asistente para instalación |Miembro de ADSyncAdmins |Realice cambios en las reglas de sincronización y en otra configuración. |
| Usuario que ejecuta el asistente para instalación |Si usa una versión completa de SQL server: DBO (o similar) de la base de datos del motor de sincronización |Realice los cambios de nivel de base de datos, como actualizar tablas con nuevas columnas. |

## <a name="more-about-the-created-accounts"></a>Más información acerca de las cuentas creadas
### <a name="ad-ds-connector-account"></a>Cuenta del conector de AD DS
Si utiliza la configuración rápida, se crea una cuenta en Active Directory que se usa para sincronización. La cuenta creada se ubica en el dominio raíz del bosque en el contenedor Usuarios y su nombre tiene el prefijo **MSOL_** . La cuenta se crea con una contraseña larga compleja que no expira. Si tiene una directiva de contraseñas en el dominio, asegúrese de que se permitan contraseñas largas y complejas para esta cuenta.

![Cuenta de AD](./media/reference-connect-accounts-permissions/adsyncserviceaccount.png)

Si usa una configuración personalizada, es responsable de crear la cuenta antes de iniciar la instalación.  Consulte Creación de la cuenta del conector de AD DS.

### <a name="adsync-service-account"></a>Cuenta del servicio ADSync
El servicio de sincronización puede ejecutarse con diferentes cuentas. Puede ejecutarse con una **cuenta de servicio virtual** (VSA), una **cuenta de servicio administrada de grupo** (gMSA/sMSA), o una cuenta de usuario normal. Las opciones admitidas cambiaron con la versión de abril de 2017 de Connect cuando se realiza una instalación nueva. Si actualiza desde una versión anterior de Azure AD Connect, estas opciones adicionales no están disponibles.

| Tipo de cuenta | Opción de instalación | Descripción |
| --- | --- | --- |
| [Cuenta de servicio virtual](#virtual-service-account) | Rápida y personalizada, abril de 2017 y versiones posteriores | Es la opción utilizada para todas las instalaciones rápidas, excepto para las instalaciones en un controlador de dominio. Para las instalaciones personalizadas, es la opción predeterminada a menos que se use otra opción. |
| [Cuenta de servicio administrada de grupo](#group-managed-service-account) | Personalizada, abril de 2017 y versiones posteriores | Si utiliza un servidor SQL remoto, se recomienda usar una cuenta de servicio administrada de grupo. |
| [Cuenta de usuario](#user-account) | Rápida y personalizada, abril de 2017 y versiones posteriores | Durante la instalación, solo se crea una cuenta de usuario con el prefijo AAD_ cuando se instala en Windows Server 2008 y cuando se instala en un controlador de dominio. |
| [Cuenta de usuario](#user-account) | Rápida y personalizada, marzo de 2017 y versiones anteriores | Se crea una cuenta local con el prefijo AAD_ durante la instalación. Cuando se utiliza la instalación personalizada, se puede especificar otra cuenta. |

Si utiliza Connect con una compilación de marzo de 2017 o anterior, no debe restablecer la contraseña en la cuenta de servicio, ya que Windows destruye las claves de cifrado por motivos de seguridad. No se puede cambiar la cuenta a cualquier otra cuenta sin volver a instalar Azure AD Connect. Si se actualiza a una compilación de abril de 2017 o posterior, es posible cambiar la contraseña de la cuenta de servicio, pero no puede cambiar la cuenta utilizada.

> [!Important]
> Solo se puede establecer la cuenta de servicio en la primera instalación. No es posible cambiar la cuenta de servicio una vez completada la instalación.

Esta es la tabla de los valores predeterminados, recomendados y admitidos para la cuenta del servicio de sincronización.

Leyenda:

- **Negrita** indica la opción predeterminada y, en la mayoría de los casos, la opción recomendada.
- *Cursiva* indica la opción recomendada si no es la opción predeterminada.
- 2008: opción predeterminada cuando se instala en Windows Server 2008
- Sin negrita: opción admitida
- Cuenta local: cuenta de usuario local en el servidor
- Cuenta de dominio: cuenta de usuario de dominio
- sMSA: [cuenta de servicio administrada independiente](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd548356(v=ws.10))
- gMSA: [cuenta de servicio administrada de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831782(v=ws.11))

| | LocalDB</br>Express | LocalDB/LocalSQL</br>Personalizado | SQL remoto</br>Personalizado |
| --- | --- | --- | --- |
| **equipo unido a un dominio** | **VSA**</br>Cuenta local (2008) | **VSA**</br>Cuenta local (2008)</br>Cuenta local</br>Cuenta de dominio</br>sMSA, gMSA | **gMSA**</br>Cuenta de dominio |
| **Controlador de dominio** | **Cuenta de dominio** | *gMSA*</br>**Cuenta de dominio**</br>sMSA| *gMSA*</br>**Cuenta de dominio**|

#### <a name="virtual-service-account"></a>Cuenta de servicio virtual
Una cuenta de servicio virtual es un tipo especial de cuenta que no tiene contraseña y está administrada por Windows.

![VSA](./media/reference-connect-accounts-permissions/aadsyncvsa.png)

Una VSA está pensada para usarse en escenarios donde el motor de sincronización y SQL están en el mismo servidor. Si usa SQL remoto, es aconsejable utilizar una cuenta de servicio administrada de grupo en su lugar.

Esta característica requiere Windows Server 2008 R2 o versiones posteriores. Si instala Azure AD Connect en Windows Server 2008, la instalación vuelve a utilizar una [cuenta de usuario](#user-account) en su lugar.

#### <a name="group-managed-service-account"></a>Cuenta de servicio administrada de grupo
Si usa un servidor SQL remoto, se recomienda utilizar una **cuenta de servicio administrada de grupo**. Para más información sobre cómo preparar Active Directory para la cuenta de servicio administrada de grupo, consulte [Información general de las cuentas de servicio administradas de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831782(v=ws.11)).

Para usar esta opción, en la página [Instalar los componentes necesarios](how-to-connect-install-custom.md#install-required-components), seleccione **Usar una cuenta de servicio existente** y seleccione **Cuenta de servicio administrada**.  
![VSA](./media/reference-connect-accounts-permissions/serviceaccount.png)  
También se puede usar una [cuenta de servicio administrada independiente](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd548356(v=ws.10)). Sin embargo, solo se pueden utilizar en el equipo local y no hay ningún beneficio al usarlas en lugar de la cuenta de servicio virtual predeterminada.

Esta característica requiere Windows Server 2012 o posterior. Si necesita usar un sistema operativo anterior y usar SQL remoto, debe usar una [cuenta de usuario](#user-account).

#### <a name="user-account"></a>Cuenta de usuario
El asistente para instalación crea una cuenta de servicio local (a menos que especifique la cuenta que se desea usar en la configuración personalizada). La cuenta lleva delante **AAD_** y se usa para el servicio de sincronización real como cuenta de ejecución. Si instala Azure AD Connect en un controlador de dominio, la cuenta se crea en el dominio. La cuenta de servicio **AAD_** debe estar ubicada en el dominio si:
   - Se usa un servidor remoto que ejecuta SQL Server.
   - Se usa a un proxy que requiere autenticación.

![Cuenta de servicio de sincronización](./media/reference-connect-accounts-permissions/syncserviceaccount.png)

La cuenta se crea con una contraseña larga compleja que no expira.

Esta se utiliza para almacenar de forma segura las contraseñas de las otras cuentas, que se almacenan cifradas en la base de datos. Las claves privadas de las claves de cifrado se protegen con el cifrado de clave secreta de los servicios criptográficos mediante la API de protección de datos de Windows (DPAPI).

Si usa SQL Server completo, la cuenta de servicio es el DBO de la base de datos creada para el motor de sincronización. El servicio no funcionará como se pretende con ningún otro permiso. También se crea un inicio de sesión SQL.

A la cuenta también se le conceden permisos para archivos, claves del Registro y otros objetos relacionados con el motor de sincronización.

### <a name="azure-ad-connector-account"></a>Cuenta del conector de Azure AD
Se crea una cuenta de Azure AD para el uso del servicio de sincronización. Esta cuenta se puede identificar por su nombre para mostrar.

![Cuenta de AD](./media/reference-connect-accounts-permissions/aadsyncserviceaccount2.png)

El nombre del servidor en el que se usa la cuenta se puede identificar en la segunda parte del nombre de usuario. En la imagen, el nombre del servidor es DC1. Si tiene servidores de ensayo, cada servidor tiene su propia cuenta.

La cuenta se crea con una contraseña larga compleja que no expira. Se le concede el rol especial **Cuentas de sincronización de directorio** que solo tiene permisos para realizar tareas de sincronización de directorios. No se puede conceder este rol integrado especial fuera del asistente de Azure AD Connect. En Azure Portal se muestra esta cuenta con el rol **Usuario**.

Hay un límite de 20 cuentas de servicio de sincronización en Azure AD. Para obtener la lista de cuentas de servicio de Azure AD existentes en Azure AD, ejecute el siguiente cmdlet de Azure AD PowerShell: `Get-AzureADDirectoryRole | where {$_.DisplayName -eq "Directory Synchronization Accounts"} | Get-AzureADDirectoryRoleMember`

Para quitar las cuentas de servicio de Azure AD sin usar, ejecute el siguiente cmdlet de Azure AD PowerShell: `Remove-AzureADUser -ObjectId <ObjectId-of-the-account-you-wish-to-remove>`

>[!NOTE]
>Para poder usar los comandos de PowerShell anteriores deberá instalar el [módulo Azure Active Directory PowerShell for Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0#installing-the-azure-ad-module) y conectarse a la instancia de Azure AD mediante [Connect-AzureAD](/powershell/module/azuread/connect-azuread?view=azureadps-2.0)

Para obtener más información sobre cómo administrar o restablecer la contraseña de la cuenta del conector de Azure AD, consulte [Administración de la cuenta de Azure AD Connect](how-to-connect-azureadaccount.md)

## <a name="related-documentation"></a>documentación relacionada
Si no leyó la documentación que se encuentra en [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md), en la tabla siguiente se proporcionan vínculos a temas relacionados.

|Tema |Vínculo|  
| --- | --- |
|Descarga de Azure AD Connect | [Descarga de Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771)|
|Instalación mediante configuración rápida | [Instalación rápida de Azure AD Connect](how-to-connect-install-express.md)|
|Instalación mediante configuración personalizada | [Instalación personalizada de Azure AD Connect](./how-to-connect-install-custom.md)|
|Actualización desde DirSync | [Actualización desde la herramienta de sincronización de Azure AD (DirSync)](how-to-dirsync-upgrade-get-started.md)|
|Después de la instalación | [Comprobación de la instalación y asignación de licencias ](how-to-connect-post-installation.md)|

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).