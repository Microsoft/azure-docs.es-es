---
title: Uso de la autenticación multifactor de Azure Active Directory
description: Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics admiten conexiones procedentes de SQL Server Management Studio (SSMS) mediante la Autenticación universal de Active Directory.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
ms.custom: seoapril2019, has-adal-ref, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 07/27/2020
tags: azure-synapse
ms.openlocfilehash: b2d179121b05d7bf3493937a9ff72e302fd31f3d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281147"
---
# <a name="using-multi-factor-azure-active-directory-authentication"></a>Uso de la autenticación multifactor de Azure Active Directory
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Azure SQL Database, Azure SQL Managed Instance y Azure Synapse Analytics admiten conexiones de [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) en las que se utiliza la autenticación *Azure Active Directory - Universal con MFA*. En este artículo, se explican las diferencias entre las distintas opciones de autenticación, así como las limitaciones asociadas al uso de la autenticación universal en Azure Active Directory (Azure AD) para Azure SQL.

**Descarga de la versión de SSMS más reciente**: en el equipo cliente, descargue la versión más reciente de SSMS desde [Descarga de SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/library/mt238290.aspx).

Para todas las características que se tratan en este artículo, utilice al menos la versión 17.2 de julio de 2017. El cuadro de diálogo de conexión más reciente debe ser similar a la siguiente imagen:

  ![1mfa-universal-connect](./media/authentication-mfa-ssms-overview/1mfa-universal-connect.png "Complete el cuadro Nombre de usuario.")  

## <a name="authentication-options"></a>Opciones de autenticación

Existen dos modelos de autenticación no interactiva en Azure AD que pueden utilizarse en muchas aplicaciones diferentes (ADO.NET, JDCB, ODC, etc.). Estos dos métodos nunca generan cuadros de diálogo emergentes:

- `Azure Active Directory - Password`
- `Azure Active Directory - Integrated`

El método interactivo que también admite Azure Multi-Factor Authentication (MFA) es: 

- `Azure Active Directory - Universal with MFA`

Azure MFA ayuda a proteger el acceso a los datos y las aplicaciones mientras se cumple la exigencia del usuario en cuanto a un proceso de inicio de sesión simple. Ofrece una autenticación segura con una gran variedad de opciones sencillas de verificación, como llamadas telefónicas, mensajes de texto, tarjetas inteligentes con PIN o notificaciones de aplicaciones móviles, lo que permite a los usuarios elegir el mecanismo que prefieran. MFA interactivo con Azure AD puede generar un cuadro de diálogo emergente para la validación.

Para obtener una descripción de Azure Multi-Factor Authentication, vea [Multi-Factor Authentication](../../active-directory/authentication/multi-factor-authentication.md).
Para ver los pasos de configuración, consulte [Configure Azure SQL Database multi-factor authentication for SQL Server Management Studio](authentication-mfa-ssms-configure.md) (Configuración de la autenticación multifactor de Azure SQL Database para SQL Server Management Studio).

### <a name="azure-ad-domain-name-or-tenant-id-parameter"></a>Nombre de dominio de Azure AD o parámetro de identificador de inquilino

A partir de la [versión 17 de SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms), los usuarios que se importan como usuarios invitados en la instancia actual de Azure AD desde otras instancias de Azure Active Directory, pueden proporcionar un identificador de inquilino o un nombre de dominio de Azure AD cuando se conectan. Los usuarios invitados incluyen los que lo son desde otras instancias de Azure AD, cuentas de Microsoft como outlook.com, hotmail.com o live.com, u otras cuentas como gmail.com. De este modo, la autenticación `Azure Active Directory - Universal with MFA` puede identificar a la entidad de autenticación correcta. Esta opción también se requiere para admitir cuentas de Microsoft (MSA), como outlook.com, hotmail.com o live.com, y otras cuentas que no son MSA. 

Todos los usuarios invitados que deseen autenticarse con la autenticación universal deben especificar el identificador de inquilino o el nombre de dominio de Azure AD. Este parámetro representa el identificador de inquilino o el nombre de dominio de Azure AD actual al que está vinculado el servidor lógico de Azure SQL. Por ejemplo, si el servidor lógico de Azure SQL está asociado al dominio `contosotest.onmicrosoft.com` de Azure AD, en el que el usuario `joe@contosodev.onmicrosoft.com` está hospedado como un usuario importado desde el dominio `contosodev.onmicrosoft.com` de Azure AD, el nombre de dominio necesario para autenticar este usuario será `contosotest.onmicrosoft.com`. Cuando el usuario es un usuario nativo de Azure AD que está asociado al servidor lógico de SQL y no es una cuenta de MSA, el identificador de inquilino y el nombre de dominio no son necesarios. Para especificar el parámetro (a partir de la versión 17.2 de SSMS):


1. Abra una conexión en SSMS. Escriba el nombre del servidor y seleccione la autenticación **Azure Active Directory - Universal con MFA**. Agregue el **nombre de usuario** con el que desee iniciar sesión.
1. Active la casilla **Opciones** y vaya a la pestaña **Propiedades de conexión**. En el cuadro de diálogo **Conectar con base de datos**, especifique los detalles de su base de datos. Active la casilla **Nombre de dominio o ID de inquilino de AD** y proporcione la entidad de autenticación, por ejemplo, el nombre de dominio (**contosoprueba.enmicrosoft.com**) o el GUID del identificador del inquilino. 

   ![mfa-tenant-ssms](./media/authentication-mfa-ssms-overview/mfa-tenant-ssms.png)

Si ejecuta SSMS 18.x o una versión posterior, ya no son necesarios el nombre de dominio de AD ni el identificador de inquilino para los usuarios invitados porque a partir de esta versión se reconocen de forma automática.

   ![mfa-tenant-ssms](./media/authentication-mfa-ssms-overview/mfa-no-tenant-ssms.png)

### <a name="azure-ad-business-to-business-support"></a>Compatibilidad con aplicaciones empresariales de Azure AD

> [!IMPORTANT]
> Actualmente, la funcionalidad que permite que los usuarios invitados se conecten a Azure SQL Database, SQL Managed Instance y Azure Synapse sin necesidad de formar parte de un grupo está en **versión preliminar pública**. Para más información, consulte [Crear usuarios invitados de Azure AD y establecerlos como administradores de Azure AD](authentication-aad-guest-users.md).

Los usuarios de Azure AD que se pueden utilizar como invitados en escenarios empresariales (B2B) de Azure AD (consulte [Qué es la colaboración B2B de Azure](../../active-directory/active-directory-b2b-what-is-azure-ad-b2b.md)) solamente pueden conectarse a Azure SQL Database y Azure Synapse si son miembros de un grupo que se haya creado en la instancia de Azure AD asociada y que se haya asignado manualmente mediante la instrucción [CREATE USER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-user-transact-sql) en una base de datos determinada. Por ejemplo, si `steve@gmail.com` es invitado a `contosotest` de Azure AD (con el dominio de Azure AD `contosotest.onmicrosoft.com`), debe crearse un grupo de Azure AD, como `usergroup` en la instancia de en Azure AD que contenga el miembro `steve@gmail.com`. Después, el administrador de Azure AD SQL o el propietario de la base de datos de Azure AD tendrá que crear este grupo en una base de datos específica (por ejemplo, `MyDatabase`) ejecutando la instrucción `CREATE USER [usergroup] FROM EXTERNAL PROVIDER` de Transact-SQL. 

Una vez creado el usuario de base de datos, el usuario `steve@gmail.com` podrá iniciar sesión en `MyDatabase` utilizando la opción de autenticación `Azure Active Directory – Universal with MFA` de SSMS. De forma predeterminada, `usergroup` solo tiene permiso de conexión. Para acceder a cualquier otro dato, será necesario que un usuario con privilegios suficientes [conceda](https://docs.microsoft.com/sql/t-sql/statements/grant-transact-sql) permiso. 

> [!NOTE]
> En SSMS 17.x, si va a utilizar `steve@gmail.com` como usuario invitado, deberá activar la casilla **Nombre de dominio o id. de inquilino de AD** y agregar el nombre de dominio `contosotest.onmicrosoft.com` de AD en el cuadro de diálogo **Propiedades de conexión**. La opción **Nombre de dominio o id. de inquilino de AD** solo puede usarse con la autenticación **Azure Active Directory - Universal con MFA**. De lo contrario, la casilla estará atenuada.

## <a name="universal-authentication-limitations"></a>Limitaciones de la autenticación universal

- SSMS y SqlPackage.exe son las únicas herramientas que actualmente están habilitadas para MFA a través de Autenticación universal de Active Directory.
- La versión 17.2 de SSMS admite el acceso simultáneo de varios usuarios mediante Autenticación universal con MFA. En las versiones 17.0 y 17.1 de SSMS, la herramienta solo permite utilizar una sola cuenta de Azure Active Directory para iniciar sesión en una instancia de SSMS con la autenticación universal. Para iniciar sesión con otra cuenta de Azure AD, debe usar otra instancia de SSMS. Esta restricción solo se aplica a la autenticación universal de Active Directory; puede iniciar sesión en un servidor diferente utilizando la autenticación `Azure Active Directory - Password`, la autenticación `Azure Active Directory - Integrated` o `SQL Server Authentication`.
- SSMS admite Autenticación universal de Active Directory para la visualización de Explorador de objetos, Editor de consultas y Almacén de consultas.
- La versión 17.2 de SSMS proporciona compatibilidad con el asistente de DacFx para la base de datos de exportación, extracción e implementación. Una vez que se autentica un usuario específico mediante el cuadro de diálogo de autenticación inicial con Autenticación universal, el asistente de DacFx funciona del mismo modo que con todos los demás métodos de autenticación.
- El Diseñador de tablas de SSMS no admite Autenticación universal.
- No hay ningún requisito de software adicional para Autenticación universal de Active Directory, excepto el uso de una versión compatible de SSMS.  
- Consulte el enlace siguiente para ver la última versión de la Biblioteca de autenticación de Active Directory (ADAL) con autenticación universal: [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).  

## <a name="next-steps"></a>Pasos siguientes

- Para ver los pasos de configuración, consulte [Configure Azure SQL Database multi-factor authentication for SQL Server Management Studio](authentication-mfa-ssms-configure.md) (Configuración de la autenticación multifactor de Azure SQL Database para SQL Server Management Studio).
- Conceda a otros usuarios acceso a la base de datos: [Autenticación y autorización de SQL Database: concesión de acceso](logins-create-manage.md)  
- Asegúrese de que otros usuarios puedan conectarse mediante el firewall: [Configuración de una regla de firewall de nivel de servidor mediante Azure Portal](firewall-configure.md)  
- [Configuración y administración de la autenticación de Azure Active Directory con SQL Database y Azure Synapse](authentication-aad-configure.md)
- [Crear usuarios invitados de Azure AD y establecerlos como administradores de Azure AD](authentication-aad-guest-users.md) 
- [Microsoft SQL Server Data-Tier Application Framework (17.0.0 GA)](https://www.microsoft.com/download/details.aspx?id=55088)  
- [SQLPackage.exe](https://docs.microsoft.com/sql/tools/sqlpackage)  
- [Importación de un archivo BACPAC en una nueva base de datos](database-import.md)  
- [Exportación de una base de datos a un archivo BACPAC](database-export.md)  
- Interfaz de C# [Interfaz IUniversalAuthProvider](https://msdn.microsoft.com/library/microsoft.sqlserver.dac.iuniversalauthprovider.aspx)  
- Cuando se utiliza la autenticación **Azure Active Directory- Universal con MFA**, a partir de [SSMS 17.3](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms), está disponible el seguimiento de ADAL. Desactivado de forma predeterminada, puede activar el seguimiento de ADAL mediante el menú **Herramientas**, **Opciones**, en **Azure Services**, **Azure Cloud**, **Nivel de seguimiento de la ventana de salida de ADAL**, y habilitar seguidamente **Salida** en el menú **Ver**. Los seguimientos están disponibles en la ventana de salida al seleccionar la opción **Azure Active Directory**.  
