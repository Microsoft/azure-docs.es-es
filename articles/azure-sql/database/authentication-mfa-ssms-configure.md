---
title: Configuración de Multi-Factor Authentication
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Aprenda a usar la autenticación multifactor con SSMS para Azure SQL Database, Azure SQL Managed Instance y Azure Synapse Analytics.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: has-adal-ref, sqldbrb=3
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 08/27/2019
ms.openlocfilehash: 791402f5d9648c8d235f8853de1b6c41f8082e1b
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018337"
---
# <a name="configure-multi-factor-authentication-for-sql-server-management-studio-and-azure-ad"></a>Configuración de la autenticación multifactor para SQL Server Management Studio y Azure AD
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

En este artículo se muestra cómo usar la autenticación multifactor (MFA) de Azure Active Directory (Azure AD) con SQL Server Management Studio (SSMS). MFA de Azure AD se puede usar cuando se conecta SSMS o SqlPackage.exe a [Azure SQL Database](sql-database-paas-overview.md), [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) y [Azure Synapse Analytics (antes, Azure SQL Data Warehouse)](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md). Para ver una introducción a la autenticación multifactor, consulte [Autenticación universal con SQL Database, Instancia administrada de SQL y Azure Synapse (compatibilidad de SSMS con MFA)](../database/authentication-mfa-ssms-overview.md).

> [!IMPORTANT]
> A las bases de datos de Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse (que anteriormente se conoce como Azure SQL Data Warehouse) en el resto de este artículo se les denomina en conjunto "bases de datos" y el servidor hace referencia al [servidor](logical-servers.md) que hospeda las bases de datos de Azure SQL Database y Azure Synapse.

## <a name="configuration-steps"></a>Pasos de configuración

1. **Configuración de una instancia de Azure Active Directory**: para más información, consulte [Administración de su directorio de Azure AD](https://msdn.microsoft.com/library/azure/hh967611.aspx), [Integración de identidades locales con Azure Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md), [Add your own domain name to Azure AD](https://azure.microsoft.com/blog/20../../windows-azure-now-supports-federation-with-windows-server-active-directory/) (Incorporación de su nombre de dominio a Azure AD), [Microsoft Azure now supports federation with Windows Server Active Directory](https://azure.microsoft.com/blog/20../../windows-azure-now-supports-federation-with-windows-server-active-directory/) (Windows ahora admite la federación con Windows Server Active Directory) y [Administrar Azure AD mediante Windows PowerShell](https://msdn.microsoft.com/library/azure/jj151815.aspx).
2. **Configurar MFA**: para obtener instrucciones detalladas, consulte [¿Qué es Azure Multi-Factor Authentication?](../../active-directory/authentication/multi-factor-authentication.md), [Acceso condicional (MFA) con Azure SQL Database y Data Warehouse](conditional-access-configure.md). (El acceso condicional completo requiere la versión Premium de Azure Active Directory. MFA limitado está disponible con la versión estándar de Azure AD).
3. **Configuración de la Autenticación de Azure AD**: para instrucciones detalladas, consulte [Conexión a SQL Database, Instancia administrada de SQL o a Azure Synapse mediante Autenticación de Azure Active Directory](authentication-aad-overview.md).
4. **Descarga de SSMS**: en el equipo cliente, descargue la versión más reciente de SSMS, desde [Descarga de SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/library/mt238290.aspx).

## <a name="connecting-by-using-universal-authentication-with-ssms"></a>Conexión mediante autenticación universal con SSMS

En los pasos siguientes se muestra cómo conectarse con la versión más reciente de SSMS.

1. Para conectarse con Autenticación universal, en el cuadro de diálogo **Conectar con el servidor** de SQL Server Management Studio (SSMS), seleccione **Active Directory: Universal compatible con MFA**. (Si ve **Autenticación universal de Active Directory**, no tiene la versión más reciente de SSMS).

   ![1mfa-universal-connect](./media/authentication-mfa-ssms-configure/mfa-no-tenant-ssms.png)  
2. Rellene el cuadro **Nombre de usuario** con las credenciales de Azure Active Directory, con el formato `user_name@domain.com`.

   ![1mfa-universal-connect-user](./media/authentication-mfa-ssms-configure/1mfa-universal-connect-user.png)
3. Si se conecta como usuario invitado, ya no necesita completar el campo Nombre de dominio de AD o el campo Id. del inquilino para usuarios invitados porque SSMS 18.x o posterior lo reconoce automáticamente. Para más información, consulte [Autenticación universal con SQL Database, Instancia administrada de SQL y Azure Synapse (compatibilidad de SSMS con MFA)](../database/authentication-mfa-ssms-overview.md).

   ![mfa-no-tenant-ssms](./media/authentication-mfa-ssms-configure/mfa-no-tenant-ssms.png)

   Sin embargo, si va a conectarse como un usuario invitado mediante SSMS 17.x o una versión posterior, debe hacer clic en **Opciones** y, en el cuadro de diálogo **Propiedad de conexión**, rellenar el cuadro **Nombre de dominio o ID de inquilino de AD**.

   ![mfa-tenant-ssms](./media/authentication-mfa-ssms-configure/mfa-tenant-ssms.png)

4. Seleccione **Opciones** y especifique la base de datos en el cuadro de diálogo **Opciones**. (Si el usuario conectado es un invitado (por ejemplo, joe@outlook.com), debe activar la casilla y agregar el nombre de dominio de AD actual o el identificador de inquilino como parte de las opciones). Consulte [Autenticación universal con SQL Database y SQL Data Warehouse (compatibilidad de SSMS con MFA)](../database/authentication-mfa-ssms-overview.md). Haga clic en **Conectar**.  
5. Cuando aparezca el cuadro de diálogo **Iniciar sesión con su cuenta** , escriba la cuenta y la contraseña de su identidad de Azure Active Directory. Si un usuario pertenece a un dominio federado con Azure AD, no se requiere contraseña.

   ![2mfa-sign-in](./media/authentication-mfa-ssms-configure/2mfa-sign-in.png)  

   > [!NOTE]
   > Para Autenticación universal con una cuenta que no requiere MFA, conéctese en este momento. Para los usuarios que requieren MFA, continúe con los pasos siguientes:
   >  

6. Pueden aparecer dos cuadros de diálogo de configuración de MFA. Esta operación de una sola vez depende de la configuración del administrador de MFA y, por tanto, puede ser opcional. En el caso de un dominio habilitado para MFA, a veces este paso está definido previamente (por ejemplo, el dominio requiere que los usuarios usen una tarjeta inteligente y un PIN).

   ![3mfa-setup](./media/authentication-mfa-ssms-configure/3mfa-setup.png)
  
7. El segundo cuadro de diálogo de un único uso que puede que aparezca le permite seleccionar los detalles del método de autenticación. El administrador configura las opciones posibles.

   ![4mfa-verify-1](./media/authentication-mfa-ssms-configure/4mfa-verify-1.png)  
8. Azure Active Directory le envía la información de confirmación. Cuando reciba el código de verificación, escríbalo en el cuadro **Escribir código de comprobación** y haga clic en **Iniciar sesión**.

   ![5mfa-verify-2](./media/authentication-mfa-ssms-configure/5mfa-verify-2.png)  

Cuando se complete la comprobación, SSMS se conectará de manera habitual, siempre que haya acceso de firewall y credenciales válidas.

## <a name="next-steps"></a>Pasos siguientes

- Para ver una introducción a la autenticación multifactor, consulte [Autenticación universal con SQL Database, Instancia administrada de SQL y Azure Synapse (compatibilidad de SSMS con MFA)](../database/authentication-mfa-ssms-overview.md).  
- Conceda a otros usuarios acceso a la base de datos: [Autenticación y autorización de SQL Database: concesión de acceso](logins-create-manage.md)  
- Asegúrese de que otros usuarios puedan conectarse mediante el firewall: [Configuración de una regla de firewall de nivel de servidor mediante Azure Portal](https://docs.microsoft.com/azure/azure-sql/database/firewall-configure)  
- Cuando se usa la autenticación de **Active Directory - Universal compatible con MFA**, el seguimiento de ADAL está disponible a partir de [SSMS 17.3](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms). Desactivado de forma predeterminada, puede activar el seguimiento de ADAL mediante el menú **Herramientas**, **Opciones**, en **Azure Services**, **Azure Cloud**, **Nivel de seguimiento de la ventana de salida de ADAL**, y habilitar seguidamente **Salida** en el menú **Ver**. Los seguimientos están disponibles en la ventana de salida al seleccionar la opción **Azure Active Directory**.
