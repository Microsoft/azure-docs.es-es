---
title: Autenticación y permisos de usuario de Azure Analysis Services | Microsoft Docs
description: En este artículo se describe cómo Azure Analysis Services usa Azure Active Directory (Azure AD) para la administración de identidades y la autenticación de usuarios.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 15ae1ca6902c27732812a591f492430150c84ecb
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926185"
---
# <a name="authentication-and-user-permissions"></a>Autenticación y permisos de usuario

Azure Analysis Services usa Azure Active Directory (Azure AD) para la administración de identidades y la autenticación de usuarios. Todo usuario que cree, administre o se conecte a un servidor de Azure Analysis Services debe tener una identidad de usuario válida en un [inquilino de Azure AD](../active-directory/fundamentals/active-directory-administer.md) en la misma suscripción.

Azure Analysis Services admite la [colaboración B2B de Azure AD](../active-directory/active-directory-b2b-what-is-azure-ad-b2b.md). Con B2B, los usuarios fuera de una organización pueden ser usuarios invitados en un directorio de Azure AD. Los invitados pueden provenir de otro directorio de inquilino de Azure AD o de cualquier dirección de correo electrónico válida. Una vez que se invita a un usuario y este acepta la invitación que se envía por correo electrónico desde Azure, la identidad del usuario se agrega al directorio del inquilino. Esas identidades se pueden agregar a grupos de seguridad o como miembros de un rol de base de datos o administrador del servidor.

![Arquitectura de autenticación de Azure Analysis Services](./media/analysis-services-manage-users/aas-manage-users-arch.png)

## <a name="authentication"></a>Authentication

Todas las herramientas y aplicaciones cliente usan una o más de las [bibliotecas de cliente](https://docs.microsoft.com/analysis-services/client-libraries?view=azure-analysis-services-current) (AMO, MSOLAP, ADOMD) de Analysis Services para conectarse a un servidor. 

Las tres bibliotecas de cliente admiten el flujo interactivo de Azure AD y los métodos de autenticación no interactivos. Los dos métodos no interactivos, Autenticación de contraseña de Active Directory y Autenticación integrada de Active Directory, se pueden usar en aplicación con AMOMD y MSOLAP. Estos dos métodos nunca generan cuadros de diálogo emergentes.

Aplicaciones cliente, como Excel y Power BI Desktop, y herramientas como SSMS y la extensión de proyectos de Analysis Services para Visual Studio instalan las versiones más recientes de las bibliotecas cuando se actualizan a la versión más reciente. Power BI Desktop, SSMS y la extensión de proyectos de Analysis Services se actualizan mensualmente. Excel [se actualiza con Office 365](https://support.office.com/article/When-do-I-get-the-newest-features-in-Office-2016-for-Office-365-da36192c-58b9-4bc9-8d51-bb6eed468516). Las actualizaciones de Office 365 son menos frecuentes y algunas organizaciones usan el canal diferido, lo que significa que las actualizaciones se aplazan hasta por tres meses.

En función de la herramienta o la aplicación cliente que use, el tipo de autenticación y cómo se inicia sesión puede variar. Cada aplicación puede admitir distintas características para conectarse a servicios en la nube, como Azure Analysis Services.

Power BI Desktop, Visual Studio y SSMS admiten la Autenticación universal de Active Directory, un método de trabajo interactivo que también admite Azure Multi-Factor Authentication (MFA). Azure MFA ayuda a proteger el acceso a los datos y las aplicaciones, al tiempo que brinda un proceso de inicio de sesión simple. Ofrece una autenticación segura con varias opciones de comprobación: llamada de teléfono, mensaje de texto, tarjetas inteligentes con PIN o notificación de aplicación móvil. MFA interactivo con Azure AD puede generar un cuadro de diálogo emergente para la validación. **Se recomienda la autenticación universal**.

Si inicia sesión en Azure con una cuenta Windows y la autenticación universal no está seleccionada ni disponible (Excel), se requiere [Servicios de federación de Active Directory (AD FS)](../active-directory/hybrid/how-to-connect-fed-azure-adfs.md). Con la federación, los usuarios de Azure AD y Office 365 se autentican con credenciales locales y pueden acceder a los recursos de Azure.

### <a name="sql-server-management-studio-ssms"></a>SQL Server Management Studio (SSMS)

Los servidores de Azure Analysis Services admiten conexiones desde [SSMS V17.1](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y versiones superiores mediante el uso de autenticación de Windows, autenticación de contraseñas de Active Directory y autenticación universal de Active Directory. En general, se recomienda usar la autenticación universal de Active Directory por los motivos siguientes:

*  Admite los métodos de autenticación interactivos y no interactivos.

*  Admite usuarios invitados de Azure B2B en el inquilino de Azure AD. Cuando se conectan a un servidor, los usuarios invitados deben seleccionar Autenticación universal de Active Directory.

*  Admite Multi-Factor Authentication (MFA). Azure MFA ayuda a proteger el acceso a los datos y las aplicaciones con una variedad de opciones de comprobación: llamada telefónica, mensaje de texto, tarjetas inteligentes con PIN o notificación de aplicación móvil. MFA interactivo con Azure AD puede generar un cuadro de diálogo emergente para la validación.

### <a name="visual-studio"></a>Visual Studio

Visual Studio se conecta a Azure Analysis Services mediante la autenticación universal de Active Directory con compatibilidad de MFA. Los usuarios deben iniciar sesión en Azure en la primera implementación. Los usuarios deben iniciar sesión en Azure con una cuenta que tenga permisos de administrador del servidor en el servidor en que realizan la implementación. Cuando se inicia sesión en Azure por primera vez, se asigna un token. El token se almacena en caché en memoria para futuras reconexiones.

### <a name="power-bi-desktop"></a>Power BI Desktop

Power BI Desktop se conecta a Azure Analysis Services mediante la autenticación universal de Active Directory con compatibilidad de MFA. Los usuarios deben iniciar sesión en Azure en la primera conexión. Los usuarios deben iniciar sesión en Azure con una cuenta que esté incluida en un rol de base de datos o administrador del servidor.

### <a name="excel"></a>Excel

Los usuarios de Excel se pueden conectar a un servidor con una cuenta de Windows, un identificador de la organización (dirección de correo electrónico) o una dirección de correo electrónico externa. Las identidades de correo electrónico externas deben existir en Azure AD como usuario invitado.

## <a name="user-permissions"></a>Permisos de usuario

Los **administradores de servidor** son específicos para una instancia de servidor de Azure Analysis Services. Se conectan con herramientas como Azure Portal, SSMS y Visual Studio para realizar tareas como agregar bases de datos y administrar roles de usuario. De manera predeterminada, el usuario que crea el servidor se agrega automáticamente como administrador del servidor de Analysis Services. Es posible agregar otros administradores mediante Azure Portal o SSMS. Los administradores del servidor deben tener una cuenta en el inquilino de Azure AD en la misma suscripción. Para más información, consulte [Administración de administradores de servidor](analysis-services-server-admins.md). 

Los **usuarios de bases de datos** se conectan a las bases de datos modelo mediante aplicaciones cliente como Excel o Power BI. Los usuarios se deben agregar a los roles de base de datos. Los roles de base de datos definen permisos de administrador, proceso o lectura para una base de datos. Resulta importante comprender que los usuarios de base de datos en un rol con permisos de administración son distintos a los administradores de servidor. Sin embargo, de manera predeterminada, los administradores de servidor también son administradores de base de datos. Para más información, consulte [Administración de usuarios y roles de base de datos](analysis-services-database-users.md).

**Propietarios de recursos de Azure**. Los propietarios de recursos administran los recursos de una suscripción de Azure. Los propietarios de recursos pueden agregar identidades de usuario de Azure AD a los roles Propietario o Colaborador dentro de una suscripción mediante **Control de acceso** en Azure Portal o con plantillas de Azure Resource Manager. 

![Control de acceso en Azure Portal](./media/analysis-services-manage-users/aas-manage-users-rbac.png)

Los roles en este nivel se aplican a los usuarios o cuentas que necesitan realizar tareas que se pueden completar en el portal o mediante plantillas de Azure Resource Manager. Aprenda sobre el [control de acceso basado en roles de Azure (RBAC de Azure)](../role-based-access-control/overview.md). 

## <a name="database-roles"></a>Roles de base de datos

 Los roles definidos para un modelo tabular son roles de base de datos. Es decir, los roles contienen miembros que constan de grupos de seguridad y usuarios de Azure AD que tienen permisos específicos que definen la acción que esos miembros pueden realizar en una base de datos modelo. Un rol de base de datos se crea como objeto independiente en la base de datos y solo se aplica a la base de datos en que se crea ese rol.   
  
 De manera predeterminada, cuando crea un nuevo proyecto modelo tabular, el proyecto modelo no tiene ningún rol. Los roles se pueden definir mediante el cuadro de diálogo Administrador de roles en Visual Studio. Cuando los roles se definen durante el diseño del proyecto modelo, solo se aplican a la base de datos del área de trabajo modelo. Cuando se implementa el modelo, se aplican los mismos roles al modelo implementado. Una vez que se implementa un modelo, los administradores de servidor y de base de datos pueden administrar los roles y los miembros con SSMS. Para más información, consulte [Administración de usuarios y roles de base de datos](analysis-services-database-users.md).
  
## <a name="next-steps"></a>Pasos siguientes

[Administración del acceso a recursos con grupos de Azure Active Directory](../active-directory/fundamentals/active-directory-manage-groups.md)   
[Administración de usuarios y roles de base de datos](analysis-services-database-users.md)  
[Administración de administradores de servidor](analysis-services-server-admins.md)  
[Control de acceso basado en roles de Azure (Azure RBAC)](../role-based-access-control/overview.md)  