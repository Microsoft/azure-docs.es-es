---
title: Habilitación de AAD para Azure SSIS Integration Runtime
description: En este artículo se describe cómo habilitar la autenticación de Azure Active Directory con la identidad administrada para Azure Data Factory para crear una instancia de Azure-SSIS Integration Runtime.
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.devlang: powershell
ms.topic: conceptual
author: swinarko
ms.author: sawinark
manager: mflasko
ms.custom: seo-lt-2019
ms.date: 07/09/2020
ms.openlocfilehash: ffbb81fa56b87281199309d61ab3e2e59c1a5acd
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87564000"
---
# <a name="enable-azure-active-directory-authentication-for-azure-ssis-integration-runtime"></a>Habilitar la autenticación de Azure Active Directory para Azure-SSIS Integration Runtime

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se muestra cómo habilitar la autenticación de Azure Active Directory (Azure AD) con la identidad administrada para Azure Data Factory (ADF) y usarla en lugar de los métodos de autenticación convencionales (como la autenticación de SQL) para:

- Crear una instancia de Azure-SSIS Integration Runtime (IR) que, a su vez, aprovisionará la base de datos del catálogo de SSIS (SSISDB) en SQL Database o Instancia administrada de SQL en su nombre.

- Conectarse a varios recursos de Azure al ejecutar paquetes SSIS en Azure-SSIS IR.

Para más información sobre la identidad administrada de ADF, consulte [Identidad administrada de Data Factory](https://docs.microsoft.com/azure/data-factory/data-factory-service-identity).

> [!NOTE]
>
> - En este escenario, la autenticación de Azure AD con la identidad administrada para su instancia de ADF solo se usa en la creación y en las operaciones de inicio posteriores de la instancia de SSIS IR que, a su vez, aprovisionará y conectará SSISDB. Para las ejecuciones de paquetes de SSIS, la instancia de SSIS IR se conectará con SSISDB mediante la autenticación de SQL con cuentas totalmente administradas que se crean durante el aprovisionamiento de SSISDB.
> - Si ya ha creado una instancia de SSIS IR con la autenticación de SQL, no puede volver a configurarla para usar la autenticación de Azure AD con PowerShell en este momento, pero puede hacerlo en Azure Portal o la aplicación ADF. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="enable-azure-ad-on-azure-sql-database"></a>Habilitación de Azure AD en Azure SQL Database

SQL Database admite la creación de una base de datos con un usuario de Azure AD. En primer lugar, debe crear un grupo de Azure AD con la identidad administrada para ADF como miembro. A continuación, debe establecer un usuario de Azure AD como administrador de Active Directory para la instancia de SQL Database y luego conectarse desde SQL Server Management Studio (SSMS) con ese usuario. Por último, deberá crear un usuario contenido que represente al grupo de Azure AD, de modo que Azure-SSIS IR pueda usar la identidad administrada para ADF para crear SSISDB en su nombre.

### <a name="create-an-azure-ad-group-with-the-managed-identity-for-your-adf-as-a-member"></a>Crear un grupo de Azure AD con la identidad administrada para ADF como miembro

Puede utilizar un grupo de Azure AD existente o crear uno nuevo con Azure AD PowerShell.

1.  Instale el módulo de [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2).

2.  Inicie sesión con  `Connect-AzureAD`, ejecute el siguiente cmdlet para crear el grupo y guárdelo en una variable:

    ```powershell
    $Group = New-AzureADGroup -DisplayName "SSISIrGroup" `
                              -MailEnabled $false `
                              -SecurityEnabled $true `
                              -MailNickName "NotSet"
    ```

    El resultado es similar al del siguiente ejemplo, que también muestra el valor de la variable:

    ```powershell
    $Group

    ObjectId DisplayName Description
    -------- ----------- -----------
    6de75f3c-8b2f-4bf4-b9f8-78cc60a18050 SSISIrGroup
    ```

3.  Agregue la identidad administrada de la instancia de ADF al grupo. Puede seguir el artículo [Identidad administrada de Data Factory](https://docs.microsoft.com/azure/data-factory/data-factory-service-identity) para obtener el identificador de objeto de la identidad administrada de la entidad de seguridad (por ejemplo, 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc, pero no use el identificador de aplicación de la identidad administrada para este propósito).

    ```powershell
    Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc
    ```

    También puede comprobar la pertenencia al grupo más adelante.

    ```powershell
    Get-AzureAdGroupMember -ObjectId $Group.ObjectId
    ```

### <a name="configure-azure-ad-authentication-for-sql-database"></a>Configuración de la autenticación de Azure AD para SQL Database

Puede  [configurar y administrar la autenticación de Azure AD con SQL](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure) siguiendo estos pasos:

1.  En Azure Portal, seleccione **Todos los servicios** -> **Servidores SQL** en el panel de navegación izquierdo.

2.  Seleccione el servidor de SQL Database que quiera configurar con la autenticación de Azure AD.

3.  En la sección **Configuración** de la hoja, haga clic en **Administrador de Active Directory**.

4.  En la barra de comandos, seleccione **Establecer administrador**.

5.  Elija una cuenta de usuario de Azure AD para que sea el administrador del servidor y, después, haga clic en **Seleccionar**.

6.  En la barra de comandos, seleccione **Guardar**.

### <a name="create-a-contained-user-in-sql-database-representing-the-azure-ad-group"></a>Cree un usuario contenido en SQL Database que represente al grupo de Azure AD

En el paso siguiente, necesita  [Microsoft SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)  (SSMS).

1. Inicie SSMS.

2. En el cuadro de diálogo **Conectar al servidor**, escriba el nombre de su servidor en el campo **Nombre del servidor**.

3. En el campo **Autenticación**, seleccione **Active Directory - Universal compatible con MFA** (también puede usar los otros dos tipos de autenticación de Active Directory; consulte [Configuración y administración de la autenticación de Azure Active Directory con SQL](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure)).

4. En el campo **Nombre de usuario**, escriba el nombre de la cuenta de Azure AD que estableció como administrador del servidor, por ejemplo, testuser@xxxonline.com.

5. Seleccione **Conectar** y complete el proceso de inicio de sesión.

6. En el **Explorador de objetos**, expanda la carpeta **Bases de datos** -> **Bases de datos del sistema**.

7. Haga clic con el botón derecho en la base de datos **maestra** y seleccione **Nueva consulta**.

8. En la ventana de consulta, escriba el siguiente comando T-SQL y seleccione **Ejecutar** en la barra de herramientas.

   ```sql
   CREATE USER [SSISIrGroup] FROM EXTERNAL PROVIDER
   ```

   El comando debería completarse correctamente y crear el usuario contenido para representar al grupo.

9. Borre la ventana de consulta, escriba el siguiente comando T-SQL y seleccione **Ejecutar** en la barra de herramientas.

   ```sql
   ALTER ROLE dbmanager ADD MEMBER [SSISIrGroup]
   ```

   El comando debería completarse correctamente y conceder al usuario contenido la capacidad de crear una base de datos (SSISDB).

10. Si la SSISDB se ha creado mediante la autenticación de SQL y desea cambiar para usar la autenticación de Azure AD, con el fin de que Azure-SSIS Integration Runtime tenga acceso a ella, primero debe asegurarse de que los pasos para conceder el permiso a la base de datos **maestra** se han realizado correctamente. Después, haga clic con el botón derecho en la base de datos **SSISDB** y seleccione **Nueva consulta**.

11. En la ventana de consulta, escriba el siguiente comando T-SQL y seleccione **Ejecutar** en la barra de herramientas.

    ```sql
    CREATE USER [SSISIrGroup] FROM EXTERNAL PROVIDER
    ```

    El comando debería completarse correctamente y crear el usuario contenido para representar al grupo.

12. Borre la ventana de consulta, escriba el siguiente comando T-SQL y seleccione **Ejecutar** en la barra de herramientas.

    ```sql
    ALTER ROLE db_owner ADD MEMBER [SSISIrGroup]
    ```

    El comando debería completarse correctamente y conceder al usuario contenido la capacidad de acceder a SSISDB.

## <a name="enable-azure-ad-on-sql-managed-instance"></a>Habilitación de Azure AD en Instancia administrada de SQL

Instancia administrada de SQL admite la creación de una base de datos con la instancia administrada para ADF directamente. No necesita unir la identidad administrada para ADF a un grupo de Azure AD ni crear un usuario contenido que represente dicho grupo en Instancia administrada de SQL.

### <a name="configure-azure-ad-authentication-for-azure-sql-managed-instance"></a>Configuración de la autenticación de Azure AD para Instancia administrada de Azure SQL

Siga los pasos que aparecen en [Aprovisionamiento de un administrador de Azure Active Directory para Instancia administrada de SQL](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure#provision-an-azure-active-directory-administrator-for-your-managed-instance).

### <a name="add-the-managed-identity-for-your-adf-as-a-user-in-sql-managed-instance"></a>Adición de la identidad administrada para ADF como usuario en Instancia administrada de SQL

En el paso siguiente, necesita  [Microsoft SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)  (SSMS).

1.  Inicie SSMS.

2.  Conéctese a Instancia administrada de SQL con una cuenta de SQL Server que sea **sysadmin**. Esta es una limitación temporal que se quitará una vez que las entidades de seguridad (inicios de sesión) del servidor de Azure AD de Instancia administrada de Azure SQL estén disponibles con carácter general. Si intenta usar una cuenta de administrador de Azure AD para crear el inicio de sesión, verá el siguiente error: Mens. 15247, nivel 16, estado 1, línea 1 El usuario no tiene permiso para realizar esta acción.

3.  En el **Explorador de objetos**, expanda la carpeta **Bases de datos** -> **Bases de datos del sistema**.

4.  Haga clic con el botón derecho en la base de datos **maestra** y seleccione **Nueva consulta**.

5.  En la ventana de consulta, ejecute el siguiente script de T-SQL para agregar la identidad administrada para ADF como usuario.

    ```sql
    CREATE LOGIN [{your ADF name}] FROM EXTERNAL PROVIDER
    ALTER SERVER ROLE [dbcreator] ADD MEMBER [{your ADF name}]
    ALTER SERVER ROLE [securityadmin] ADD MEMBER [{your ADF name}]
    ```
    
    El comando debería completarse correctamente y conceder a la identidad administrada para ADF la capacidad de crear una base de datos (SSISDB).

6.  Si la SSISDB se ha creado mediante la autenticación de SQL y desea cambiar para usar la autenticación de Azure AD, con el fin de que Azure-SSIS Integration Runtime tenga acceso a ella, primero debe asegurarse de que los pasos para conceder el permiso a la base de datos **maestra** se han realizado correctamente. Después, haga clic con el botón derecho en la base de datos **SSISDB** y seleccione **Nueva consulta**.

7.  En la ventana de consulta, escriba el siguiente comando T-SQL y seleccione **Ejecutar** en la barra de herramientas.

    ```sql
    CREATE USER [{your ADF name}] FOR LOGIN [{your ADF name}] WITH DEFAULT_SCHEMA = dbo
    ALTER ROLE db_owner ADD MEMBER [{your ADF name}]
    ```

    El comando debería completarse correctamente y conceder a la identidad administrada para ADF la capacidad de acceder a SSISDB.

## <a name="provision-azure-ssis-ir-in-azure-portaladf-app"></a>Aprovisionar Azure-SSIS IR en Azure Portal o ADF

Al aprovisionar Azure-SSIS IR en Azure Portal o la aplicación AFD, en la página **Configuración de SQL**, seleccione la opción **Use AAD authentication with the managed identity for your ADF** (Usar autenticación de AAD con la identidad administrada de ADF). En la captura de pantalla siguiente se muestra la configuración del entorno de ejecución de integración con SQL Database que hospeda SSISDB. Para IR con Instancia administrada de SQL que hospeda SSISDB, los valores **Catalog Database Service Tier** (Nivel de servicio de base de datos del catálogo) y **Permitir el acceso a los servicios de Azure** no son aplicables, mientras que otros valores son los mismos.

Para más información sobre cómo crear una instancia de Azure-SSIS IR, consulte [Creación de una instancia de Integration Runtime de SSIS de Azure en Azure Data Factory](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime).

![Configuración de la instancia de Integration Runtime para la integración de SSIS en Azure](media/enable-aad-authentication-azure-ssis-ir/enable-aad-authentication.png)

## <a name="provision-azure-ssis-ir-with-powershell"></a>Aprovisionamiento de Azure-SSIS IR con PowerShell

Para aprovisionar el entorno de ejecución para la integración de SSIS en Azure con PowerShell, haga lo siguiente:

1.  Instale el módulo [Azure PowerShell](https://github.com/Azure/azure-powershell/releases/tag/v5.5.0-March2018) .

2.  En el script, no establezca el parámetro `CatalogAdminCredential`. Por ejemplo:

    ```powershell
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                               -DataFactoryName $DataFactoryName `
                                               -Name $AzureSSISName `
                                               -Description $AzureSSISDescription `
                                               -Type Managed `
                                               -Location $AzureSSISLocation `
                                               -NodeSize $AzureSSISNodeSize `
                                               -NodeCount $AzureSSISNodeNumber `
                                               -Edition $AzureSSISEdition `
                                               -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
                                               -CatalogServerEndpoint $SSISDBServerEndpoint `
                                               -CatalogPricingTier $SSISDBPricingTier

    Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                                 -DataFactoryName $DataFactoryName `
                                                 -Name $AzureSSISName
    ```

## <a name="run-ssis-packages-with-managed-identity-authentication"></a>Ejecución de paquetes SSIS con la autenticación de identidad administrada

Al ejecutar paquetes SSIS en Azure-SSIS IR, puede usar la autenticación de identidad administrada para conectarse a varios recursos de Azure. Actualmente ya se admite la autenticación de identidad administrada en los siguientes administradores de conexiones.

- [Administrador de conexiones OLE DB](https://docs.microsoft.com/sql/integration-services/connection-manager/ole-db-connection-manager#managed-identities-for-azure-resources-authentication)

- [Administrador de conexiones ADO.NET](https://docs.microsoft.com/sql/integration-services/connection-manager/ado-net-connection-manager#managed-identities-for-azure-resources-authentication)

- [Administrador de conexiones de Azure Storage](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-storage-connection-manager#managed-identities-for-azure-resources-authentication)
