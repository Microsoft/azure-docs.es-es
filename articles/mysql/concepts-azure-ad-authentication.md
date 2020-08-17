---
title: Autenticación de Active Directory en Azure Database for MySQL
description: Conozca los conceptos de la autenticación de Azure Active Directory con Azure Database for MySQL.
author: lfittl-msft
ms.author: lufittl
ms.service: mysql
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: 942789d5541e796247af2dbdf9371cce4701105e
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171892"
---
# <a name="use-azure-active-directory-for-authenticating-with-mysql"></a>Uso de Azure Active Directory para la autenticación con MySQL

La autenticación de Microsoft Azure Active Directory (Azure AD) es un mecanismo de conexión a Azure Database for MySQL mediante identidades definidas en Azure AD.
Con la autenticación de Azure AD, puede administrar las identidades de los usuarios de la base de datos y otros servicios de Microsoft en una ubicación central, lo que simplifica la administración de permisos.

Entre las ventajas de usar Azure AD se incluyen:

- Autenticación de usuarios en los servicios de Azure de manera uniforme.
- Administración de directivas de contraseñas y rotación de contraseñas en un solo lugar.
- Varias formas de autenticación compatibles con Azure Active Directory, lo que puede eliminar la necesidad de almacenar contraseñas.
- Los clientes pueden administrar los permisos de la base de datos con grupos externos (Azure AD).
- La autenticación de Azure AD utiliza usuarios de base de datos MySQL para autenticar las identidades en el nivel de base de datos.
- Compatibilidad de la autenticación basada en tokens para las aplicaciones que se conectan a Azure Database for MySQL.

Para configurar y usar la autenticación de Azure Active Directory, siga el proceso a continuación:

1. Cree y rellene Azure Active Directory con identidades de usuario según sea necesario.
2. De manera opcional, asocie o cambie la instancia de Active Directory asociada actualmente a la suscripción de Azure.
3. Cree un administrador de Azure AD para el servidor de Azure Database for MySQL.
4. Cree usuarios de bases de datos en la base de datos, asignados a identidades de Azure AD.
5. Conéctese a la base de datos recuperando un token para una identidad de Azure AD e iniciando sesión.

> [!NOTE]
> Para aprender a crear y rellenar Azure AD y, posteriormente, configurar Azure AD con Azure Database for MySql, consulte [Configuración e inicio de sesión con Azure AD para Azure Database for MySQL](howto-configure-sign-in-azure-ad-authentication.md).

## <a name="architecture"></a>Architecture

En el siguiente diagrama de alto nivel se resume cómo funciona la autenticación de Azure AD con Azure Database for MySQL. Las flechas indican las rutas de comunicación.

![flujo de autenticación][1]

## <a name="administrator-structure"></a>Estructura del administrador

Cuando se usa la autenticación de Azure AD, existen dos cuentas de administrador para el servidor de MySQL: el administrador de MySQL original y el administrador de Azure AD. Solo el administrador basado en una cuenta de Azure AD puede crear el primer usuario de base de datos independiente en Azure AD en una base de datos de usuario. El inicio de sesión del administrador de Azure AD puede ser un usuario de Azure AD o un grupo de Azure AD. Cuando el administrador es una cuenta de grupo, cualquier miembro del grupo lo puede usar, lo que permite varios administradores de Azure AD para el servidor MySQL. Mediante el uso de una cuenta de grupo como administrador, se mejora la administración al permitir agregar y quitar miembros de grupo de forma centralizada en Azure AD sin cambiar los usuarios ni los permisos del servidor MySQL. Solo un administrador de Azure AD (un usuario o grupo) se puede configurar en cualquier momento.

![estructura de administración][2]

## <a name="permissions"></a>Permisos

Para crear nuevos usuarios que se puedan autenticar con Azure AD, debe ser el administrador de Azure AD designado. Este usuario se asigna mediante la configuración de la cuenta de administrador de Azure AD para un servidor de Azure Database for MySQL específico.

Para crear un nuevo usuario de base de datos de Azure AD, debe conectarse como administrador de Azure AD. Esto se muestra en [Configuración e inicio de sesión con Azure AD para Azure Database for MySQL](howto-configure-sign-in-azure-ad-authentication.md).

La autenticación de Azure AD solo es posible si se creó el administrador de Azure AD para Azure Database for MySQL. Si se ha quitado el administrador de Azure Active Directory del servidor, los usuarios de Azure Active Directory creados previamente ya no podrán conectarse a la base de datos utilizando sus credenciales de Azure Active Directory.

## <a name="connecting-using-azure-ad-identities"></a>Conexión mediante identidades de Azure AD

La autenticación de Azure Active Directory admite los siguientes métodos de conexión a una base de datos mediante identidades de Azure AD:

- Contraseña de Azure Active Directory
- Azure Active Directory integrado
- Azure Active Directory Universal con MFA
- Uso de certificados de aplicación o secretos de cliente de Active Directory
- [Identidad administrada](howto-connect-with-managed-identity.md)

Una vez que se ha autenticado en el Active Directory, se recupera un token. Este token es la contraseña para iniciar sesión.

Tenga en cuenta que las operaciones de administración, como agregar usuarios nuevos, solo se admiten para roles de usuario de Azure AD en este momento.

> [!NOTE]
> Para obtener más información sobre cómo conectarse con un token de Active Directory, consulte [Configuración e inicio de sesión con Azure AD para Azure Database for MySQL](howto-configure-sign-in-azure-ad-authentication.md).

## <a name="additional-considerations"></a>Consideraciones adicionales

- La autenticación de Azure Active Directory solo está disponible para MySQL 5.7 y versiones más recientes.
- Solo un administrador de Azure AD se puede configurar en un servidor de Azure Database for MySQL en cualquier momento.
- Inicialmente, solo un administrador de Azure AD para MySQL puede conectarse a Azure Database for MySQL con una cuenta de Azure Active Directory. El administrador de Active Directory puede configurar los usuarios de la base de datos de Azure AD sucesivos.
- Si se elimina un usuario de Azure AD, ese usuario ya no podrá autenticarse con Azure AD y, por tanto, ya no será posible adquirir un token de acceso para dicho usuario. En este caso, aunque el usuario coincidente seguirá estando en la base de datos, no será posible conectarse al servidor con ese usuario.
> [!NOTE]
> Aún es posible iniciar sesión con el usuario de Azure AD eliminado, hasta que expire el token (máximo de 60 minutos a partir de la emisión del token).  Si también quita el usuario de Azure Database for MySQL, este acceso se revocará inmediatamente.
- Si el administrador de Azure AD se quita del servidor, el servidor ya no estará asociado a un inquilino de Azure AD y, por tanto, se deshabilitarán todos los inicios de sesión de Azure AD para el servidor. Al agregar un nuevo administrador de Azure AD desde el mismo inquilino, se volverán a habilitar los inicios de sesión de Azure AD.
- Azure Database for MySQL asocia los tokens de acceso al usuario de Azure Database for MySQL mediante el identificador de usuario de Azure AD exclusivo del usuario, en lugar de usar el nombre de usuario. Esto significa que si un usuario de Azure AD se elimina en Azure AD y se crea un nuevo usuario con el mismo nombre, Azure Database for MySQL lo toma como un usuario diferente. Por lo tanto, si se elimina un usuario de Azure AD y, a continuación, se agrega un nuevo usuario con el mismo nombre, el nuevo usuario no podrá conectarse con el usuario existente.

## <a name="next-steps"></a>Pasos siguientes

- Para aprender a crear y rellenar Azure AD y, posteriormente, configurar Azure AD con Azure Database for MySql, consulte [Configuración e inicio de sesión con Azure AD para Azure Database for MySQL](howto-configure-sign-in-azure-ad-authentication.md).
- Para información general sobre los inicios de sesión y los usuarios de bases de datos de Azure Database for MySQL, consulte [Creación de usuarios en Azure Database for MySQL](howto-create-users.md).

<!--Image references-->

[1]: ./media/concepts-azure-ad-authentication/authentication-flow.png
[2]: ./media/concepts-azure-ad-authentication/admin-structure.png
