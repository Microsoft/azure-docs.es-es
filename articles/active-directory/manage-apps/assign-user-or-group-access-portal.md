---
title: Asignación de un usuario o un grupo a una aplicación empresarial en Azure Active Directory | Microsoft Docs
description: Procedimiento para seleccionar una aplicación empresarial para asignarla a un usuario o un grupo en la versión preliminar de Azure Active Directory
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 04/11/2019
ms.author: mimart
ms.reviewer: luleon
ms.collection: M365-identity-device-management
ms.openlocfilehash: 702eb5008c60764cd84e2787a8e2c2818d3f9268
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780930"
---
# <a name="assign-a-user-or-group-to-an-enterprise-app-in-azure-active-directory"></a>Asignación de un usuario o un grupo a una aplicación empresarial en Azure Active Directory
Para asignar un usuario o un grupo a una aplicación empresarial, debe contar con los permisos adecuados para administrar dicha aplicación, y debe ser administrador global del directorio.

> [!NOTE]
> Para los requisitos de licencia de las características tratadas en este artículo, vea la [página de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory).

> [!NOTE]
> Con aplicaciones de Microsoft (por ejemplo, aplicaciones de Office 365), use PowerShell para asignar usuarios a una aplicación empresarial.


## <a name="assign-a-user-to-an-app---portal"></a>Asignación de un usuario a una aplicación: portal
1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta que tenga el rol de administrador global en el directorio.
1. Seleccione **Todos los servicios**, escriba Azure Active Directory en el cuadro de texto y seleccione **Entrar**.
1. Seleccione **Aplicaciones empresariales**.
1. En el **aplicaciones empresariales - todas las aplicaciones** panel, verá una lista de las aplicaciones que puede administrar. Seleccione una aplicación.
1. En el ***appname*** panel (es decir, el panel con el nombre de la aplicación seleccionada en el título), seleccione **usuarios y grupos**.
1. En el ***appname*** **-grupos de usuarios y** panel, seleccione **Agregar usuario**.
1. En el **Agregar asignación** panel, seleccione **usuarios y grupos**.

    ![Asignación de un usuario o un grupo a la aplicación](./media/assign-user-or-group-access-portal/assign-users.png)
1. En el **usuarios y grupos** panel, seleccione uno o más usuarios o grupos de la lista y, a continuación, elija el **seleccione** situado en la parte inferior del panel.
1. En el **Agregar asignación** panel, seleccione **rol**. A continuación, en el **Seleccionar rol** panel, seleccione un rol que se aplicará a los usuarios o grupos seleccionados, a continuación, seleccione **Aceptar** en la parte inferior del panel.
1. En el **Agregar asignación** panel, seleccione el **asignar** situado en la parte inferior del panel. Los usuarios o grupos asignados tienen definidos los permisos por el rol seleccionado para esta aplicación empresarial.

## <a name="allow-all-users-to-access-an-app---portal"></a>Permitir a los usuarios tener acceso a una aplicación: portal
Para permitir a los usuarios tener acceso a una aplicación:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta que tenga el rol de administrador global en el directorio.
1. Seleccione **Todos los servicios**, escriba Azure Active Directory en el cuadro de texto y seleccione **Entrar**.
1. Seleccione **Aplicaciones empresariales**.
1. En el panel **Aplicaciones empresariales**, seleccione **Todas las aplicaciones**. A continuación se muestran las aplicaciones que se pueden administrar.
1. En el panel **Aplicaciones empresariales - Todas las aplicaciones**, seleccione una aplicación.
1. En el ***appname*** panel, seleccione **propiedades**.
1. En el  ***appname* -propiedades** panel, establezca la **asignación de usuarios necesaria?** si se establece en **No**. 

La opción **¿Asignación de usuarios?**:

- No afecta a si una aplicación aparece en el panel de acceso de la aplicación o no. Para mostrar la aplicación en el panel de acceso, debe asignar un grupo o usuario adecuado a la aplicación.
- Solo las funciones con las aplicaciones en la nube configuradas para el inicio de sesión único de SAML, y aplicaciones locales configuradas con el proxy de aplicación. Vea [Inicio de sesión único para aplicaciones](what-is-single-sign-on.md).
- Requiere que los usuarios den su consentimiento a una aplicación. Un administrador puede conceder consentimiento para todos los usuarios.  Vea [Configure el modo en que los usuarios finales dan su consentimiento a una aplicación](configure-user-consent.md).


## <a name="assign-a-user-to-an-app---powershell"></a>Asignación de un usuario a una aplicación: PowerShell

1. Abra un símbolo del sistema de Windows PowerShell con privilegios elevados.

    >[!NOTE] 
    > Debe instalar el módulo de AzureAD (use el comando `Install-Module -Name AzureAD`). Si se le pide que instale un módulo de NuGet o el nuevo módulo de PowerShell para Azure Active Directory V2, escriba S y presione ENTRAR.

1. Ejecute `Connect-AzureAD` e inicie sesión con una cuenta de usuario administrador global.
1. Use el siguiente script para asignar un usuario y un rol a una aplicación:

    ```powershell
    # Assign the values to the variables
    $username = "<You user's UPN>"
    $app_name = "<Your App's display name>"
    $app_role_name = "<App role display name>"
    
    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }
    
    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```     

Para más información sobre cómo asignar un usuario a un rol de aplicación, consulte la documentación de [AzureADUserAppRoleAssignment de nuevo](https://docs.microsoft.com/powershell/module/azuread/new-azureaduserapproleassignment?view=azureadps-2.0).

Para asignar un grupo a una aplicación empresarial, es necesario reemplazar `Get-AzureADUser` por `Get-AzureADGroup`.

### <a name="example"></a>Ejemplo

En este ejemplo se asigna el usuario Britta Simon a la aplicación [Microsoft Workplace Analytics](https://products.office.com/business/workplace-analytics) mediante PowerShell.

1. En PowerShell, asigne los valores correspondientes a las variables $username, app_name $ y $app_role_name. 

    ```powershell
    # Assign the values to the variables
    $username = "britta.simon@contoso.com"
    $app_name = "Workplace Analytics"
    ```

1. En este ejemplo, no se sabe cuál es el nombre exacto del rol de aplicación que se quiere asignar a Britta Simon. Ejecute los comandos siguientes para obtener el usuario ($user) y la entidad de servicio ($sp) mediante el UPN de usuario y los nombres para mostrar de la entidad de servicio.

    ```powershell
    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    ```
        
1. Ejecute el comando `$sp.AppRoles` para mostrar los roles disponibles para la aplicación Workplace Analytics. En este ejemplo, queremos asignar a Britta Simon el rol de Analyst (acceso limitado).
    
    ![Rol de Workplace Analytics](./media/assign-user-or-group-access-portal/workplace-analytics-role.png)

1. Asigne el nombre de rol a la variable `$app_role_name`.
        
    ```powershell
    # Assign the values to the variables
    $app_role_name = "Analyst (Limited access)"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }
    ```

1. Ejecute el comando siguiente para asignar el rol de aplicación al usuario:

    ```powershell
    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```

## <a name="next-steps"></a>Pasos siguientes
* [Ver todos mis grupos](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Eliminación de asignaciones de usuario o grupo de una aplicación empresarial](remove-user-or-group-access-portal.md)
* [Deshabilitar los inicios de sesión de los usuarios de una aplicación empresarial](disable-user-sign-in-portal.md)
* [Cambiar el nombre o el logotipo de una aplicación empresarial](change-name-or-logo-portal.md)
