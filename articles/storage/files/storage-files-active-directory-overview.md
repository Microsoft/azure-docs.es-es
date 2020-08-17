---
title: Información general de la autorización basada en identidades de Azure Files
description: Azure Files admite la autenticación basada en identidades mediante SMB (Bloque de mensajes del servidor) a través de Azure Active Directory Domain Services (AD DS) y Active Directory. Las máquinas virtuales Windows unidas al dominio pueden entonces acceder a los recursos compartidos de archivos de Azure con las credenciales de Azure AD.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: conceptual
ms.date: 05/29/2020
ms.author: rogarana
ms.openlocfilehash: 65d7845763b60a66f396d9081f3c9c6e1d04c1bc
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87447256"
---
# <a name="overview-of-azure-files-identity-based-authentication-options-for-smb-access"></a>Introducción a las opciones de autenticación basada en la identidad de Azure Files con el acceso SMB
[!INCLUDE [storage-files-aad-auth-include](../../../includes/storage-files-aad-auth-include.md)]

Para más información sobre cómo habilitar la autenticación de Active Directory Domain Services local para recursos compartidos de archivos de Azure, vea [Habilitación de la autenticación de Azure Active Directory en SMB para recursos compartidos de archivos de Azure](storage-files-identity-auth-active-directory-enable.md).

Para aprender a habilitar la autenticación de Azure AD DS para recursos compartidos de archivos de Azure, consulte [Habilitación de la autenticación de Azure Active Directory Domain Services en Azure Files](storage-files-identity-auth-active-directory-domain-service-enable.md).

## <a name="glossary"></a>Glosario 
Es útil entender algunos términos clave relacionados con la autenticación de Azure AD Domain Service sobre SMB para recursos compartidos de archivos de Azure:

-   **Autenticación Kerberos**

    Kerberos es un protocolo de autenticación que se utiliza para comprobar la identidad de un usuario o un host. Para más información sobre Kerberos, consulte [Introducción a la autenticación Kerberos](https://docs.microsoft.com/windows-server/security/kerberos/kerberos-authentication-overview).

-  **Protocolo Bloque de mensajes del servidor (SMB)**

    SMB es un protocolo de uso compartido de archivos de red estándar del sector. SMB también se conoce como sistema de archivos de Internet común o CIFS. Para más información sobre SMB, consulte [Microsoft SMB Protocol and CIFS Protocol Overview](https://docs.microsoft.com/windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview) (Introducción a los protocolos CIFS y SMB de Microsoft).

-   **Azure Active Directory (Azure AD)**

    Azure Active Directory (Azure AD) es el directorio multiinquilino basado en la nube y el servicio de administración de identidades de Microsoft. Azure AD combina servicios de directorio fundamentales, administración del acceso a las aplicaciones y protección de identidades en una única solución. Las máquinas virtuales (VM) Windows unidas a Azure AD pueden acceder a los recursos compartidos de archivos de Azure con las credenciales de Azure AD. Para más información, consulte [¿Qué es Azure Active Directory?](../../active-directory/fundamentals/active-directory-whatis.md)

-   **Azure Active Directory Domain Services (Azure AD DS)**

    Azure AD DS proporciona servicios de dominio administrados, por ejemplo, unión a un dominio, directivas de grupo, LDAP y autenticación Kerberos/NTLM. Estos servicios son totalmente compatibles con Active Directory Domain Services. Para obtener más información, consulte [Azure Active Directory Domain Services](../../active-directory-domain-services/overview.md).

- **Active Directory Domain Services (AD DS) local**

    La integración de Active Directory Domain Services (AD DS) local en Azure Files proporciona los métodos para almacenar datos de directorio y poner dichos datos a disposición de los usuarios y administradores de la red. La seguridad se integra en AD DS mediante la autenticación de inicio de sesión y el control de acceso a los objetos del directorio. Con un único inicio de sesión de red, los administradores pueden administrar los datos del directorio y la organización a través de su red, y los usuarios de red autorizados pueden tener acceso a los recursos en cualquier parte de la red. Normalmente, son empresas en entornos locales las que adoptan AD DS, y usan las credenciales de AD DS como identidad para el control de acceso. Para obtener más información, consulte [Introducción a Active Directory Domain Services](https://docs.microsoft.com/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview).

-   **Control de acceso basado en roles de Azure (Azure RBAC)**

    El control de acceso basado en rol de Azure (Azure RBAC) permite realizar una administración detallada del acceso para Azure. Con RBAC, puede administrar el acceso a los recursos mediante la concesión a los usuarios del menor número de permisos necesarios para realizar su trabajo. Para más información acerca de RBAC, consulte [¿Qué es el control de acceso basado en rol (RBAC) de Azure?](../../role-based-access-control/overview.md).

## <a name="common-use-cases"></a>Casos de uso comunes

La autenticación basada en identidades y la compatibilidad con las ACL de Windows en Azure Files se aprovecha mejor para los siguientes casos de uso:

### <a name="replace-on-premises-file-servers"></a>Reemplazo de servidores de archivos locales

Dejar de usar y reemplazar servidores de archivos locales dispersos es un problema común que toda empresa encuentra en su recorrido de modernización de TI. Los recursos compartidos de archivos de Azure con la autenticación de AD DS local son los más idóneos aquí, cuando puede migrar los datos a Azure Files. Una migración completa le permitirá aprovechar las ventajas de alta disponibilidad y escalabilidad, a la vez que minimiza los cambios del lado cliente. Proporciona una experiencia de migración sin problemas a los usuarios finales, para que puedan seguir accediendo a sus datos con las mismas credenciales mediante sus equipos unidos a un dominio existente.

### <a name="lift-and-shift-applications-to-azure"></a>Migración "lift-and-shift" de aplicaciones a Azure

Al aplicar migraciones "lift-and-shift" de aplicaciones a la nube, querrá mantener el mismo modelo de autenticación para los datos. A medida que se amplía la experiencia de control de acceso basado en identidad a los recursos compartidos de archivos de Azure, se elimina la necesidad de cambiar la aplicación a los métodos de autenticación modernos y acelerar la adopción de la nube. Los recursos compartidos de archivos de Azure ofrecen la opción de integrarse en Azure AD DS o AD DS local para la autenticación. Si tiene pensado pasarse un 100 % a la nube nativa y minimizar los esfuerzos de administración de infraestructuras en la nube, Azure AD DS sería una mejor opción como servicio de dominio totalmente administrado. Si necesita compatibilidad total con las funcionalidades de AD DS, puede tener en cuenta la posibilidad de ampliar el entorno de AD DS a la nube mediante controladores de dominio autohospedados en VM. En cualquier caso, proporcionamos la flexibilidad para elegir los servicios de dominio que se adapten a sus necesidades empresariales.

### <a name="backup-and-disaster-recovery-dr"></a>Copia de seguridad y recuperación ante desastres (DR)

Si mantiene el almacenamiento de archivos principal en el entorno local, los recursos compartidos de archivos de Azure pueden servir como almacenamiento ideal para copias de seguridad o recuperación ante desastres, lo que mejora la continuidad empresarial. Puede usar recursos compartidos de archivos de Azure para realizar copias de seguridad de los datos a partir de servidores de archivos existentes, a la vez que conserva las DACL de Windows. En escenarios de recuperación ante desastres, puede configurar una opción de autenticación para admitir el cumplimiento adecuado del control de acceso en la conmutación por error.

## <a name="supported-scenarios"></a>Escenarios admitidos

En esta tabla se resumen los escenarios de autenticación de recursos compartidos de archivos de Azure compatibles con Azure AD DS y AD DS local. Se recomienda seleccionar el servicio de dominio que se adoptó para el entorno de cliente para la integración con Azure Files. Si tiene AD DS ya instalado de forma local o en Azure, donde los dispositivos están unidos a un dominio de AD, debe elegir aprovechar AD DS para la autenticación de recursos compartidos de archivos de Azure. Del mismo modo, si ya ha adoptado Azure AD DS, debe usarlo para recursos compartidos de archivos de Azure.


|Autenticación de Azure AD DS  | Autenticación de AD DS local  |
|---------|---------|
|Las máquinas Windows unidas a Azure AD DS pueden tener acceso a los recursos compartidos de archivos de Azure con las credenciales de Azure AD por SMB.     |Las máquinas Windows unidas a AD DS local o a Azure AD DS pueden acceder a los recursos compartidos de archivos de Azure con credenciales de Active Directory local que se sincronizan con Azure AD a través de SMB. El cliente debe tener una línea de visión en la instancia de AD DS.        |

### <a name="restrictions"></a>Restricciones

- Las autenticaciones de Azure AD DS y de AD DS local no admiten la autenticación en cuentas de equipo. En su lugar, puede pensar en usar una cuenta de inicio de sesión de servicio.
- La autenticación de Azure AD DS y la autenticación de AD DS local no se admiten en dispositivos unidos a Azure AD o en dispositivos registrados de Azure AD.
- Los recursos compartidos de archivos de Azure solo admiten la autenticación basada en identidades en uno de los siguientes servicios de dominio: [Azure Active Directory Domain Services (Azure AD DS)](#azure-ad-ds) o [Active Directory Domain Services (AD DS) local](#ad-ds).

## <a name="advantages-of-identity-based-authentication"></a>Ventajas de la autenticación basadas en identidad
La autenticación basada en identidad para Azure Files ofrece varias ventajas respecto al uso de la autenticación de clave compartida:

-   **Ampliar la experiencia tradicional de acceso de recursos compartidos de archivos basada en identidad en la nube con AD DS local y Azure AD DS**  
    Si tiene previsto realizar una migración "lift-and-shift" de su aplicación a la nube, mediante el reemplazo de los servidores de archivos tradicionales con recursos compartidos de archivos de Azure, tal vez quiera que la aplicación se autentique con las credenciales de AD DS local o Azure AD DS para acceder a los datos de los archivos. Azure Files admite el uso de credenciales de AD DS local y de Azure AD DS para tener acceso a los recursos compartidos de archivos de Azure a través de SMB desde VM unidas a un dominio de AD DS local o Azure AD DS.

-   **Aplicar el control de acceso granular en recursos compartidos de archivos de Azure**  
    Puede conceder permisos a una identidad específica en el nivel de recurso compartido, directorio o archivo. Por ejemplo, suponga que tiene varios equipos que utilizan un solo recurso compartido de archivos de Azure para la colaboración en proyectos. Puede conceder a todos los equipos acceso a directorios no confidenciales, al tiempo que limita el acceso a directorios que contienen datos financieros confidenciales únicamente a su equipo de financiero. 

-   **Realizar copias de seguridad de ACL de Windows (también conocidas como NTFS) junto con los datos**  
    Puede usar recursos compartidos de archivos de Azure para realizar copias de seguridad de los recursos compartidos de archivos locales existentes. Azure Files conserva las listas de control de acceso junto con los datos cuando se hace una copia de seguridad de un recurso compartido de archivos en recursos compartidos de archivos de Azure sobre SMB.

## <a name="how-it-works"></a>Funcionamiento

Los recursos compartidos de archivos de Azure aprovechan el protocolo Kerberos para la autenticación con AD DS local o Azure AD. Cuando una identidad asociada con un usuario o una aplicación que se ejecuta en un cliente intenta acceder a los datos de recursos compartidos de archivos de Azure, la solicitud se envía al servicio d dominio, ya sea AD DS o Azure AD DS para autenticar la identidad. Si la autenticación es correcta, devuelve un token de Kerberos. El cliente envía una solicitud que incluye el token de Kerberos, y los recursos compartidos de archivos de Azure usan ese token para autorizar la solicitud. Los recursos compartidos de archivos de Azure solo reciben el token de Kerberos, no las credenciales de acceso.

Para poder habilitar la autenticación basada en identidades en recursos compartidos de archivos de Azure, debe configurar el entorno del dominio.

### <a name="ad-ds"></a>AD DS

En el caso de la autenticación de AD DS local, primero debe configurar los controladores de dominio de AD y unir el dominio a sus máquinas o máquinas virtuales. Puede hospedar los controladores de dominio en máquinas virtuales de Azure o en un entorno local. En cualquier caso, los clientes unidos a un dominio deben tener una línea de visión en el servicio de dominio, por lo que deben estar en la red corporativa o en la red virtual (VNET) del servicio de dominio.

En este diagrama se muestra la autenticación de AD DS local en recursos compartidos de archivos de Azure a través de SMB. La instancia de AD DS local se debe sincronizar con Azure AD mediante Azure AD Connect Sync. Solo se pueden autenticar y autorizar a los usuarios híbridos que existen en el AD DS local y en Azure AD para el acceso a recursos compartidos de archivos de Azure. Esto se debe a que el permiso de nivel de recurso compartido se configura con respecto a la identidad representada en Azure AD donde se aplica el permiso de nivel de archivo o directorio con el de AD DS. Asegúrese de configurar los permisos correctamente con el mismo usuario híbrido.

:::image type="content" source="media/storage-files-active-directory-overview/Files-on-premises-AD-DS-Diagram.png" alt-text="Diagrama":::

### <a name="azure-ad-ds"></a>Azure AD DS

En el caso de la autenticación de Azure AD DS, debe habilitar Azure AD Domain Services y unir a un dominio las VM desde las que tiene pensado obtener acceso a los datos de los archivos. La VM unida a un dominio debe residir en la misma red virtual (Vnet) que la instancia de Azure AD DS. 

Este diagrama representa el flujo de trabajo para la autenticación de Azure AD DS en recursos compartidos de archivos de Azure a través de SMB. Sigue un patrón similar a la autenticación de AD DS local en recursos compartidos de archivos de Azure. Hay dos diferencias principales:

- En primer lugar, no es necesario crear la identidad en Azure AD DS para representar la cuenta de almacenamiento. Esto lo realiza el proceso de habilitación en segundo plano.

- En segundo lugar, todos los usuarios que existen en Azure AD se pueden autenticar y autorizar. El usuario puede ser híbrido o estar solo en la nube. La plataforma administra la sincronización de Azure AD con Azure AD DS sin necesidad de ninguna configuración de usuario. Pero el cliente debe estar unido a un dominio para Azure AD DS, no puede estar registrado o unido a Azure AD. 

:::image type="content" source="media/storage-files-active-directory-overview/Files-Azure-AD-DS-Diagram.png" alt-text="Diagrama":::

### <a name="enable-identity-based-authentication"></a>Habilitación de la autenticación basadas en identidad

Puede habilitar la autenticación basada en identidad con Azure AD DS o AD DS local para recursos compartidos de archivos de Azure en las cuentas de almacenamiento nuevas y existentes. Solo se puede usar un servicio de dominio para la autenticación de acceso a archivos en la cuenta de almacenamiento, lo que se aplica a todos los recursos compartidos de archivos de la cuenta. Para obtener instrucciones detalladas sobre cómo configurar los recursos compartidos de archivos para la autenticación con Azure AD DS, vea nuestro artículo [Habilitación de la autenticación de Azure Active Directory Domain Services en Azure Files](storage-files-identity-auth-active-directory-domain-service-enable.md); si lo que busca son instrucciones de AD DS local, vea este otro artículo [Habilitación de la autenticación de Azure Active Directory en SMB para recursos compartidos de archivos de Azure](storage-files-identity-auth-active-directory-enable.md).

### <a name="configure-share-level-permissions-for-azure-files"></a>Configuración de los permisos de nivel de recurso compartido para Azure Files

Cuando se ha habilitado la autenticación de Azure AD DS o AD DS local, puede usar roles integrados de Azure o configurar roles personalizados para las identidades de Azure AD y asignar derechos de acceso a cualquier recurso compartido de archivos en las cuentas de almacenamiento. El permiso asignado permite a la identidad concedida obtener acceso solo al recurso compartido, a nada más, ni siquiera al directorio raíz. Aún es necesario configurar por separado los permisos de nivel de archivo o directorio para los recursos compartidos de archivos de Azure.

### <a name="configure-directory-or-file-level-permissions-for-azure-files"></a>Configuración de los permisos de nivel de archivo o directorio para Azure Files

Los recursos compartidos de archivos de Azure aplican permisos estándar de archivo Windows en el nivel de archivo y directorio, incluido el directorio raíz. La configuración de permisos de nivel de archivo o directorio se admite sobre SMB y REST. Monte el recurso compartido de archivos de destino de la VM y configure los permisos mediante el Explorador de archivos de Windows, o el comando de Windows [icacls](https://docs.microsoft.com/windows-server/administration/windows-commands/icacls) o [Set-ACL](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/get-acl?view=powershell-6).

### <a name="use-the-storage-account-key-for-superuser-permissions"></a>Uso de la clave de cuenta de almacenamiento para los permisos de superusuario

Un usuario con la clave de cuenta de almacenamiento puede acceder a los recursos compartidos de archivos de Azure con permisos de superusuario. Los permisos de superusuario omiten todas las restricciones de control de acceso.

> [!IMPORTANT]
> Nuestro procedimiento recomendado de seguridad es evitar el uso compartido de las claves de cuenta de almacenamiento y aprovechar la autenticación basada en identidad siempre que sea posible.

### <a name="preserve-directory-and-file-acls-when-importing-data-to-azure-file-shares"></a>Conservación de las listas de control de acceso de archivos y directorios al importar datos a recursos compartidos de archivos de Azure

Azure Files admite la conservación de las listas de control de acceso de directorios o archivos al copiar datos a recursos compartidos de archivos de Azure. Puede copiar listas de control de acceso de un directorio o archivo en recursos compartidos de archivos de Azure mediante Azure File Sync o conjuntos de herramientas comunes para mover archivos. Por ejemplo, puede usar [robocopy](https://docs.microsoft.com/windows-server/administration/windows-commands/robocopy) con la marca `/copy:s` para copiar tanto los datos como las listas de control de acceso en un recurso compartido de archivos de Azure. Las listas de control de acceso se conservan de forma predeterminada, no es necesario habilitar la autenticación basada en identidad en la cuenta de almacenamiento para conservarlas.

## <a name="pricing"></a>Precios
No hay ningún cargo adicional por servicio para habilitar la autenticación basada en identidad sobre SMB en la cuenta de almacenamiento. Para obtener más información sobre los precios, consulte [Precios de Azure Files](https://azure.microsoft.com/pricing/details/storage/files/) y [Precios de Azure AD Domain Services](https://azure.microsoft.com/pricing/details/active-directory-ds/).

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre Azure Files y la autenticación basada en identidad a través de SMB, consulte estos recursos:

- [Planeamiento de una implementación de Azure Files](storage-files-planning.md)
- [Habilitación de la autenticación de Azure Active Directory en SMB para recursos compartidos de archivos de Azure](storage-files-identity-auth-active-directory-enable.md)
- [Habilitación de la autenticación de Azure Active Directory Domain Services en Azure Files](storage-files-identity-auth-active-directory-domain-service-enable.md)
- [P+F](storage-files-faq.md)
