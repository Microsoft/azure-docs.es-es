---
title: 'Tutorial: Configuración de los roles de Azure Analysis Services | Microsoft Docs'
description: En este tutorial, aprenderá a configurar los roles de usuario y administrador de Azure Analysis Services mediante Azure Portal o SQL Server Management Studio.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: owend
ms.reviewer: owend
ms.openlocfilehash: 0a19a0ed359cae61778866303d864d60e0e7606e
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229368"
---
# <a name="tutorial-configure-server-administrator-and-user-roles"></a>Tutorial: Configuración de los roles de administrador del servidor y de usuario

 En este tutorial se utiliza SQL Server Management Studio (SSMS) para conectarse al servidor de Azure para configurar los roles de administrador del servidor y de base de datos modelo. También es una introducción a [Tabular Model Scripting Language (TMSL)](https://docs.microsoft.com/analysis-services/tabular-model-programming-compatibility-level-1200/tabular-model-programming-for-compatibility-level-1200). TMSL es un lenguaje de scripting basado en JSON para modelos tabulares de los niveles de compatibilidad 1200 y superiores. Se puede utilizar para automatizar varias tareas de modelado tabular. TMSL se suele utilizar con PowerShell, pero en este tutorial se utilizará el editor de consultas XMLA de SSMS. Con este tutorial se completan las siguientes tareas: 
  
> [!div class="checklist"]
> * Obtención del nombre del servidor desde el portal
> * Conexión al servidor mediante SSMS
> * Incorporación de un usuario o grupo al rol de administrador del servidor 
> * Incorporación de un usuario o grupo al rol de administrador de la base de datos modelo
> * Incorporación de un nuevo rol de base de datos modelo y de un usuario o grupo

Para más información sobre la seguridad del usuario en Azure Analysis Services, consulte [Autenticación y permisos de usuario](../analysis-services-manage-users.md). 

## <a name="prerequisites"></a>Prerrequisitos

- Azure Active Directory en la suscripción.
- Un [servidor de Azure Analysis Services](../analysis-services-create-server.md) creado en la suscripción.
- Permisos de [administrador del servidor](../analysis-services-server-admins.md).
- [Modelo de ejemplo de adventureworks](../analysis-services-create-sample-model.md) agregado al servidor.
- La [versión más reciente de SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) instalada.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en el [portal](https://portal.azure.com/).

## <a name="get-server-name"></a>Obtención del nombre del servidor
Para poder conectarse al servidor desde SSMS, primero necesita el nombre de este. Puede obtener el nombre del servidor desde el portal.

En **Azure Portal** > servidor > **Información general** > **Nombre de servidor**, copie el nombre del servidor.
   
   ![Obtención del nombre del servidor en Azure](./media/analysis-services-tutorial-roles/aas-copy-server-name.png)

## <a name="connect-in-ssms"></a>Conectar en SSMS

En las tareas restantes, para la conexión y la administración el servidor se utiliza SSMS.

1. En SSMS > **Explorador de objetos**, haga clic en **Conectar** > **Analysis Services**.

    ![Conectar](./media/analysis-services-tutorial-roles/aas-ssms-connect.png)

2. En el cuadro de diálogo **Conectar con el servidor**, en **Nombre del servidor**, pegue el nombre del servidor que copió en el portal. En **Autenticación**, elija **Active Directory - Universal compatible con MFA**, escriba la cuenta de usuario y presione **Conectar**.
   
    ![Conectar en SSMS](./media/analysis-services-tutorial-roles/aas-connect-ssms-auth.png)

    > [!TIP]
    > Se recomienda elegir Active Directory - Universal compatible con MFA. Este tipo de autenticación es compatible con la [autenticación no interactiva y multifactor](../../azure-sql/database/authentication-mfa-ssms-overview.md). 

3. Expanda el **Explorador de objetos** para ver objetos de servidor. Haga clic con el botón derecho para ver las propiedades del servidor.
   
    ![Explorador de objetos en SSMS](./media/analysis-services-tutorial-roles/aas-connect-ssms-objexp.png)

## <a name="add-a-user-account-to-the-server-administrator-role"></a>Incorporación de una cuenta de usuario al rol de administrador del servidor

En esta tarea se agrega una cuenta de grupo o de usuario de Azure AD al rol de administrador del servidor. Si especifica un grupo de seguridad, use `obj:groupid@tenantid`.

1. En el **Explorador de objetos**, haga clic con el botón derecho en el nombre del servidor y haga clic en **Propiedades**. 
2. En la ventana **Propiedades de Analysis Server**, haga clic en **Seguridad** > **Agregar**.
3. En la ventana **Seleccione un usuario o grupo**, escriba una cuenta de grupo o de usuario en Azure AD y haga clic en **Agregar**. 
   
     ![Incorporación del administrador del servidor](./media/analysis-services-tutorial-roles/aas-add-server-admin.png)

4. Haga clic en **Aceptar** para cerrar **Propiedades de Analysis Server**.

    > [!TIP]
    > También puede agregar los administradores del servidor desde el portal, en **Administradores de Analysis Services**. 

## <a name="add-a-user-to-the-model-database-administrator-role"></a>Incorporación de un usuario al rol de administrador de la base de datos modelo

En esta tarea se agrega una cuenta de grupo o de usuario para el rol de administrador de ventas por Internet que ya existe en el modelo. Este rol tiene permisos de control total (administrador) para la base de datos del modelo de ejemplo de adventureworks. En esta tarea se usa el comando TMSL [CreateOrReplace](https://docs.microsoft.com/analysis-services/tmsl/createorreplace-command-tmsl) en un script creado automáticamente.

1. En el **Explorador de objetos**, expanda **Bases de datos** > **adventureworks** > **Roles**. 
2. Haga clic con el botón derecho en **Internet Sales Administrator** (Administrador de ventas por Internet) y haga clic en **Script Role as** > **CREATE OR REPLACE To** > **New Query Editor Window** (Crear rol en el script como > CREAR O REEMPLAZAR como) > (Nueva ventana del editor de consultas).

    ![Nueva ventana del editor de consultas](./media/analysis-services-tutorial-roles/aas-add-db-admin.png)

3. En **XMLAQuery**, cambie el valor de **"memberName":** por una cuenta de usuario o de grupo en Azure AD. De forma predeterminada se incluye la cuenta con la que ha iniciado sesión; sin embargo, no es necesario agregar su propia cuenta, porque ya es administrador del servidor.

    ![Script de TMSL en la consulta XMLA](./media/analysis-services-tutorial-roles/aas-add-db-admin-script.png)

4. Presione **F5** para ejecutar el script.


## <a name="add-a-new-model-database-role-and-add-a-user-or-group"></a>Incorporación de un nuevo rol de base de datos modelo y de un usuario o grupo

En esta tarea se debe usar el comando [Create](https://docs.microsoft.com/analysis-services/tmsl/create-command-tmsl) en un script TMSL para crear un nuevo rol Internet Sales Global, especificar permisos de *lectura* para el rol y agregar una cuenta de usuario o de grupo de Azure AD.

1. En el **Explorador de objetos**, haga clic con el botón derecho en **adventureworks** y, después, haga clic en **Nueva consulta** > **XMLA**. 
2. Copie y pegue el script TMSL siguiente en el editor de consultas:

    ```JSON
    {
    "create": {
      "parentObject": {
        "database": "adventureworks",
       },
       "role": {
         "name": "Internet Sales Global",
         "description": "All users can query model data",
         "modelPermission": "read",
         "members": [
           {
             "memberName": "globalsales@adventureworks.com",
             "identityProvider": "AzureAD"
           }
         ]
       }
      }
    }
    ```

3. Cambie el valor del objeto `"memberName": "globalsales@adventureworks.com"` por una cuenta de usuario o de grupo en Azure AD.
4. Presione **F5** para ejecutar el script.

## <a name="verify-your-changes"></a>Verificación de los cambios

1. En el **Explorador de objetos**, haga clic en el nombre de servidor y en **Actualizar** o presione **F5**.
2. Expanda **Bases de datos** > **adventureworks** > **Roles**. Verifique que aparecen los cambios en la cuenta de usuario y en el rol que agregó en la tarea anterior.   

    ![Verificación en el Explorador de objetos](./media/analysis-services-tutorial-roles/aas-connect-ssms-verify.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine las cuentas de usuario o de grupo y los roles. Para ello, use **Propiedades de rol** > **Pertenencia** para quitar las cuentas de usuario, o haga clic con el botón derecho en un rol y haga clic en **Eliminar**.


## <a name="next-steps"></a>Pasos siguientes
En este tutorial aprendió a conectarse al servidor de Azure AS y a explorar las bases de datos del modelo de ejemplo de adventureworks y las propiedades en SSMS. También aprendió a utilizar scripts SSMS y TMSL para agregar usuarios o grupos a nuevos roles o a roles existentes. Ahora que tiene permisos de usuario configurados para el servidor y la base de datos del modelo de ejemplo, usted y otros usuarios pueden conectarse a ellos mediante aplicaciones cliente como Power BI. Para más información, continúe con el siguiente tutorial. 

> [!div class="nextstepaction"]
> [Tutorial: Conexión con Power BI Desktop](analysis-services-tutorial-pbid.md)

