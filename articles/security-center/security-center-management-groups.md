---
title: Obtención de visibilidad de todos los inquilinos en Azure Security Center | Microsoft Docs
description: En este artículo se explica cómo administrar la posición de seguridad a escala mediante la aplicación de directivas a todas las suscripciones vinculadas al inquilino de Azure Active Directory.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: b85c0e93-9982-48ad-b23f-53b367f22b10
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/19/2018
ms.author: memildin
ms.openlocfilehash: 43a6c10c8c73e8fb5189b6f085a6969c0d776593
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534913"
---
# <a name="gain-tenant-wide-visibility-for-azure-security-center"></a>Obtención de visibilidad de todos los inquilinos en Azure Security Center
En este artículo se explica cómo administrar la posición de seguridad de la organización a escala mediante la aplicación de directivas de seguridad a todas las suscripciones vinculadas al inquilino de Azure Active Directory.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="management-groups"></a>Grupos de administración
Los grupos de administración de Azure permiten administrar de un modo eficaz el acceso, las directivas y los informes en grupos de suscripciones, además de administrar todas las inversiones en Azure mediante acciones en el grupo de administración raíz. Cada inquilino de Azure AD tiene un grupo de administración de nivel superior único llamado grupo de administración raíz. Este grupo de administración raíz está integrado en la jerarquía de manera que contiene todos los grupos de administración y suscripciones. Este grupo permite que las directivas globales y las asignaciones de roles de Azure se apliquen en el nivel de directorio. 

El grupo de administración raíz se crea de manera automática cuando realiza cualquiera de las siguientes acciones: 
1. Decide utilizar los grupos de administración de Azure en **Grupos de administración** en [Azure Portal](https://portal.azure.com).
2. Crea un grupo de administración mediante una llamada API.
3. Crea un grupo de administración con PowerShell.

Para más información sobre los grupos de administración, consulte el artículo [Organización de los recursos con grupos de administración de Azure](../azure-resource-manager/management-groups-overview.md).

## <a name="create-a-management-group-in-the-azure-portal"></a>Creación de un grupo de administración en Azure Portal
Puede organizar las suscripciones en grupos de administración y aplicar las directivas de gobernanza a los grupos de administración. Todas las suscripciones dentro de un grupo de administración heredan automáticamente las directivas que se aplican al grupo de administración. Aunque los grupos de administración no son necesarios para incorporar Security Center, se recomienda crear al menos un grupo de administración para que se cree el grupo de administración raíz. Una vez creado el grupo, todas las suscripciones del inquilino de Azure AD estarán vinculadas a él. Para las instrucciones de PowerShell y otra información, consulte [Creación de grupos de administración para la administración de los recursos y la organización](../azure-resource-manager/management-groups-create.md).

 
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** > **Grupos de administración**.
3. En la página principal, seleccione **Nuevo grupo de administración**. 

    ![Grupo principal](./media/security-center-management-groups/main.png) 
4.  Rellene el campo de identificador de grupo de administración. 
    - El **identificador de grupo de administración** es el identificador único de directorio que se usa para enviar comandos en este grupo de administración. Este identificador no es editable una vez creado, dado que se usa en todo el sistema de Azure para identificar este grupo. 
    - El campo de nombre para mostrar es el nombre que se muestra en Azure Portal. Un nombre para mostrar independiente es un campo opcional al crear el grupo de administración y se puede cambiar en cualquier momento.  

      ![Crear](./media/security-center-management-groups/create_context_menu.png)  
5.  Seleccione **Guardar**.

### <a name="view-management-groups-in-the-azure-portal"></a>Visualización de grupos de administración en Azure Portal
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Para ver los grupos de administración, seleccione **Todos los servicios** en el menú principal de Azure.
3. En **General**, seleccione **Grupos de administración**.

    ![Creación de un grupo de administración](./media/security-center-management-groups/all-services.png)

## <a name="grant-tenant-level-visibility-and-the-ability-to-assign-policies"></a>Concesión de visibilidad en el nivel de inquilino y la capacidad de asignar directivas

Para obtener visibilidad en la posición de seguridad de todas las suscripciones registradas en el inquilino de Azure AD, se necesita asignar un rol de Azure con permisos de lectura suficientes en el grupo de administración raíz.

### <a name="elevate-access-for-a-global-administrator-in-azure-active-directory"></a>Elevación de los privilegios de acceso de un administrador global en Azure Active Directory
Un administrador de inquilino de Azure Active Directory no tiene acceso directo a las suscripciones de Azure. No obstante, como administrador de directorio, tienen el derecho de elevarse a sí mismos a un rol que tiene derechos de acceso. Un administrador de inquilino de Azure AD debe elevarse a sí mismo a administrador de acceso de usuarios en el nivel de grupo de administración raíz para poder asignar roles de Azure. Para obtener instrucciones de PowerShell e información adicional, consulte [Elevación de los privilegios de acceso de un administrador global en Azure Active Directory](../role-based-access-control/elevate-access-global-admin.md). 


1. Inicie sesión en el [Azure Portal](https://portal.azure.com) o en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com).

2. En la lista de navegación, haga clic en **Azure Active Directory** y, luego, haga clic en **Propiedades**.

   ![Propiedades de Azure AD, captura de pantalla](./media/security-center-management-groups/aad-properties.png)

3. En **Access management for Azure resources** (Administración de acceso a recursos de Azure), establezca el modificador en **Sí**.

   ![Captura de pantalla de administración de acceso a recursos de Azure](./media/security-center-management-groups/aad-properties-global-admin-setting.png)

   - Al establecer el modificador en Sí, se le asigna el rol de administrador de acceso de usuario en Azure RBAC en el ámbito raíz (/). Esto le concede permiso para asignar roles en todas las suscripciones de Azure y los grupos de administración asociados a este directorio de Azure AD. Este modificador solo está disponible para los usuarios que tienen asignado el rol de administrador global de Azure AD.

   - Al establecer el modificador en No, se quita el rol de administrador de acceso de usuario en Azure RBAC de su cuenta de usuario. Ya no puede asignar roles en todas las suscripciones de Azure y los grupos de administración asociados a este directorio de Azure AD. Puede ver y administrar solo las suscripciones de Azure y los grupos de administración a los que se le ha concedido acceso.

4. Haga clic en **Guardar** para guardar la configuración.

    - Esta configuración no es una propiedad global y se aplica solo al usuario que tiene la sesión iniciada.

5. Realice las tareas que debe realizar al tener privilegios elevados de acceso. Cuando haya terminado, establezca el conmutador de nuevo en **No**.


### <a name="assign-azure-roles-to-users"></a>Asignación de roles de Azure a usuarios
Para ganar visibilidad en todas las suscripciones, los administradores de inquilinos deben asignar el rol de Azure adecuado a los usuarios a los que desea conceder visibilidad en todo el inquilino, incluidos ellos mismos, en el nivel de grupo de administración raíz. Los roles recomendados que se deben asignar son **Administrador de seguridad** o **Lector de seguridad**. Por lo general, el rol de Administrador de seguridad es necesario para aplicar directivas en el nivel raíz, mientras que el de Lector de seguridad será suficiente para proporcionar visibilidad en el nivel de inquilino. Para más información acerca de los permisos concedidos por estos roles, consulte la [descripción del rol integrado de Administrador de seguridad](../role-based-access-control/built-in-roles.md#security-admin) o la [descripción del rol integrado de Lector de seguridad](../role-based-access-control/built-in-roles.md#security-reader).


#### <a name="assign-azure-roles-to-users-through-the-azure-portal"></a>Asignación de roles de Azure a los usuarios mediante Azure Portal: 

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
1. Para ver los grupos de administración, seleccione **Todos los servicios** en el menú principal de Azure y, a continuación, seleccione **Grupo de administración**.
1.  Seleccione un grupo de administración y haga clic en **detalles**.

    ![Captura de pantalla con los detalles de Grupos de administración](./media/security-center-management-groups/management-group-details.PNG)
 
1. Haga clic en **Control de acceso (IAM)** y después en **Asignaciones de roles**.

1. Haga clic en **Agregar asignación de roles**.

1. Seleccione el rol para asignar y el usuario y, a continuación, haga clic en **Guardar**.  
   
   ![Captura de pantalla de incorporación del rol de Lector de seguridad](./media/security-center-management-groups/asc-security-reader.png)


#### <a name="assign-azure-roles-to-users-with-powershell"></a>Asignación de roles de Azure a los usuarios con PowerShell: 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

1. Instale [Azure PowerShell](/powershell/azure/install-az-ps).
2. Ejecute los comandos siguientes: 

    ```azurepowershell
    # Login to Azure as a Global Administrator user
    Connect-AzAccount
    ```

3. Cuando se le solicite, inicie sesión con credenciales de administrador global. 

    ![Captura de pantalla de la solicitud de inicio de sesión](./media/security-center-management-groups/azurerm-sign-in.PNG)

4. Conceda permisos de rol de lector mediante la ejecución del comando siguiente:

    ```azurepowershell
    # Add Reader role to the required user on the Root Management Group
    # Replace "user@domian.com” with the user to grant access to
    New-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Reader" -Scope "/"
    ```
5. Para eliminar el rol, use el siguiente comando: 

    ```azurepowershell
    Remove-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Reader" -Scope "/" 
    ```

### <a name="open-or-refresh-security-center"></a>Apertura o actualización de Security Center
Cuando tenga privilegios de acceso elevados, abra o actualice Azure Security Center para comprobar que tiene visibilidad en todas las suscripciones del inquilino de Azure AD. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. En el selector de suscripciones, asegúrese de seleccionar todas las suscripciones que quiere ver en Security Center.

    ![Captura de pantalla del selector de suscripciones](./media/security-center-management-groups/subscription-selector.png)

1. Seleccione **Todos los servicios** en el menú principal de Azure y, a continuación, seleccione **Security Center**.
2. En **Información general** hay un gráfico de cobertura de suscripciones.

    ![Captura de pantalla del gráfico de cobertura de suscripciones](./media/security-center-management-groups/security-center-subscription-coverage.png)

3. Haga clic en **Cobertura** para ver la lista de suscripciones incluidas. 

    ![Captura de pantalla de la lista de cobertura de suscripciones](./media/security-center-management-groups/security-center-coverage.png)

### <a name="remove-elevated-access"></a>Eliminación de privilegios de acceso elevados 
Después de asignar los roles de Azure a los usuarios, el administrador de inquilino debería eliminarse a sí mismo del rol de administrador de acceso de usuarios.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com) o en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com).

2. En la lista de navegación, haga clic en **Azure Active Directory** y, luego, haga clic en **Propiedades**.

3. En **Administración de acceso a recursos de Azure**, establezca el modificador en **No**.

4. Haga clic en **Guardar** para guardar la configuración.



## <a name="adding-subscriptions-to-a-management-group"></a>Incorporación de suscripciones a un grupo de administración
Puede agregar suscripciones al grupo de administración que ha creado. Estos pasos no son obligatorios para obtener visibilidad en todo el inquilino y administración de directivas globales y de acceso.

1. En **Grupos de administración**, seleccione el grupo de administración al que va a agregar la suscripción.

    ![Selección de un grupo de administración para agregar la suscripción](./media/security-center-management-groups/management-group-subscriptions.png)

2. Seleccione **Agregar existente**.

    ![Agregar existente](./media/security-center-management-groups/add-existing.png)

3. Escriba la suscripción en **Agregar recurso existente** y haga clic en **Guardar**.

4. Repita los pasos 1 a 3 hasta que haya agregado todas las suscripciones del ámbito.

   > [!NOTE]
   > Los grupos de administración pueden contener tanto suscripciones como grupos de administración secundarios. Cuando asigna un rol de Azure a un usuario en el grupo de administración primario, las suscripciones del grupo de administración secundario heredan el acceso. Las directivas establecidas en el grupo de administración primario también son heredadas por los secundarios. 

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha aprendido a obtener la visibilidad de todos los inquilinos en Azure Security Center. Para más información sobre Security Center, consulte los siguientes artículos:

> [!div class="nextstepaction"]
> [Supervisión del estado de seguridad en Azure Security Center](security-center-monitoring.md)

> [!div class="nextstepaction"]
> [Administración y respuesta a alertas de seguridad en Azure Security Center](security-center-managing-and-responding-alerts.md)
