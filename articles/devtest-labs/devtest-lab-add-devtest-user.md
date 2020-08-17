---
title: Adición de propietarios y usuarios en Azure DevTest Labs | Microsoft Docs
description: Agregar propietarios y usuarios en Azure DevTest Labs mediante Azure Portal o PowerShell
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 40173589c4798a8b00d940bca2a71f4d61f2c199
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533332"
---
# <a name="add-owners-and-users-in-azure-devtest-labs"></a>Adición de propietarios y usuarios en Azure DevTest Labs
> [!VIDEO https://channel9.msdn.com/Blogs/Azure/How-to-set-security-in-your-DevTest-Lab/player]
> 
> 

El acceso a Azure DevTest Labs se controla mediante el [control de acceso basado en rol de Azure (RBAC de Azure)](../role-based-access-control/overview.md). Mediante RBAC, puede repartir las tareas del equipo en *roles* y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo. Tres de estos roles de Azure son *Propietario*, *Usuario de DevTest Labs* y *Colaborador*. En este artículo, aprenderá qué acciones se pueden realizar en cada uno de los tres roles principales de Azure. A partir de ahí, aprenderá cómo agregar usuarios a un laboratorio a través del portal o a través de un script de PowerShell y cómo agregar los usuarios en el nivel de suscripción.

## <a name="actions-that-can-be-performed-in-each-role"></a>Acciones que se pueden realizar en cada rol
Hay tres roles principales que puede asignar a un usuario:

* Propietario
* Usuario de DevTest Labs
* Colaborador

En la tabla siguiente se muestran las acciones que pueden realizar los usuarios pertenecientes a cada uno de estos roles:

| **Acciones que pueden realizar los usuarios de este rol** | **Usuario de DevTest Labs** | **Propietario** | **Colaborador** |
| --- | --- | --- | --- |
| **Tareas de laboratorio** | | | |
| Agregar usuarios a un laboratorio |No |Sí |No |
| Actualizar la configuración de costo |No |Sí |Sí |
| **Tareas base de máquina virtual** | | | |
| Agregar y quitar imágenes personalizadas |No |Sí |Sí |
| Agregar, actualizar y eliminar las fórmulas |Sí |Sí |Sí |
| Incluir en la lista de permitidos imágenes de Azure Marketplace |No |Sí |Sí |
| **Tareas de la máquina virtual** | | | |
| Creación de máquinas virtuales |Sí |Sí |Sí |
| Iniciar, detener y eliminar máquinas virtuales |Solo las máquinas virtuales creadas por el usuario |Sí |Sí |
| Actualizar directivas de máquinas virtuales |No |Sí |Sí |
| Agregar discos de datos o quitarlos en máquinas virtuales |Solo las máquinas virtuales creadas por el usuario |Sí |Sí |
| **Tareas de artefacto** | | | |
| Agregar y quitar repositorios de artefacto |No |Sí |Sí |
| Aplicar artefactos |Sí |Sí |Sí |

> [!NOTE]
> Cuando un usuario crea una máquina virtual, a ese usuario se le asigna automáticamente el rol **Propietario** de la máquina virtual creada.
> 
> 

## <a name="add-an-owner-or-user-at-the-lab-level"></a>Agregar un propietario o usuario en el nivel de laboratorio
Los propietarios y los usuarios se pueden agregar en el nivel de laboratorio a través de Azure Portal. Un usuario puede ser un usuario externo con una [cuenta Microsoft (MSA)](devtest-lab-faq.md#what-is-a-microsoft-account)válida.
Los siguientes pasos le guiarán a través del proceso de agregación de un propietario o usuario a un laboratorio de Azure DevTest Labs:

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
2. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
3. En la lista de laboratorios, seleccione el laboratorio que desee.
4. En la hoja del laboratorio, seleccione **Directivas y configuración**. 
5. En la página **Configuración y directivas**, seleccione **Control de acceso (IAM)** en el menú de la izquierda. 
6. Seleccione **Agregar asignación de roles** en la barra de herramientas para agregar un usuario a un rol.
1. En la ventana **Agregar permisos**, realice las siguientes acciones: 
    1. Seleccione un rol (por ejemplo: Usuario de DevTest Labs). En la sección [Acciones que se pueden realizar en cada rol](#actions-that-can-be-performed-in-each-role) se enumeran las distintas acciones que pueden realizar los usuarios de los roles de Propietario, Usuario de DevTest y Colaborador.
    2. Seleccione el usuario que se va a agregar al rol. 
    3. Seleccione **Guardar**. 
11. Cuando vuelva a la hoja **Usuarios** , el usuario ya se habrá agregado.  

## <a name="add-an-external-user-to-a-lab-using-powershell"></a>Incorporación de un usuario externo a un laboratorio mediante PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Además de agregar usuarios en Azure Portal, puede agregar un usuario externo al laboratorio mediante un script de PowerShell. En el ejemplo siguiente, basta con modificar los valores de parámetro en el comentario **Values to change** (Valores para cambiar).
Puede recuperar los valores `subscriptionId`, `labResourceGroup` y `labName` de la hoja de laboratorio en Azure Portal.

> [!NOTE]
> El script de ejemplo supone que el usuario especificado se ha agregado como invitado a Active Directory y se producirá un error si este no es el caso. Para agregar un usuario que no está en Active Directory a un laboratorio, use Azure Portal para asignar el usuario a un rol, tal como se muestra en la sección [Agregar un propietario o usuario en el nivel de laboratorio](#add-an-owner-or-user-at-the-lab-level).   
> 
> 

```azurepowershell
# Add an external user in DevTest Labs user role to a lab
# Ensure that guest users can be added to the Azure Active directory:
# https://azure.microsoft.com/documentation/articles/active-directory-create-users/#set-guest-user-access-policies

# Values to change
$subscriptionId = "<Enter Azure subscription ID here>"
$labResourceGroup = "<Enter lab's resource name here>"
$labName = "<Enter lab name here>"
$userDisplayName = "<Enter user's display name here>"

# Log into your Azure account
Connect-AzAccount

# Select the Azure subscription that contains the lab. 
# This step is optional if you have only one subscription.
Select-AzSubscription -SubscriptionId $subscriptionId

# Retrieve the user object
$adObject = Get-AzADUser -SearchString $userDisplayName

# Create the role assignment. 
$labId = ('subscriptions/' + $subscriptionId + '/resourceGroups/' + $labResourceGroup + '/providers/Microsoft.DevTestLab/labs/' + $labName)
New-AzRoleAssignment -ObjectId $adObject.Id -RoleDefinitionName 'DevTest Labs User' -Scope $labId
```

## <a name="add-an-owner-or-user-at-the-subscription-level"></a>Agregar un propietario o usuario en el nivel de suscripción
Los permisos de Azure se propagan desde el ámbito primario al secundario en Azure. Por lo tanto, los propietarios de una suscripción de Azure que contiene laboratorios, automáticamente son propietarios de estos. También serán propietarios de las máquinas virtuales y de los demás recursos creados por los usuarios del laboratorio y el servicio Azure DevTest Labs. 

Puede agregar propietarios adicionales a un laboratorio a través de la hoja del laboratorio en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040). Sin embargo, el ámbito de administración del propietario agregado será más limitado que el ámbito del propietario de la suscripción. Por ejemplo, los propietarios agregados no tienen acceso completo a algunos de los recursos que el servicio DevTest Labs crea en la suscripción. 

Para agregar un propietario a una suscripción de Azure, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
2. Seleccione **Todos los servicios** y, en la lista, **Suscripciones**.
3. Seleccione la suscripción deseada.
4. Seleccione el icono **Acceder** . 
   
    ![Usuarios de acceso](./media/devtest-lab-add-devtest-user/access-users.png)
5. En la hoja **Usuarios**, seleccione **Agregar**.
   
    ![Agregar usuario](./media/devtest-lab-add-devtest-user/devtest-users-blade.png)
6. En la hoja **Seleccionar rol**, seleccione **Propietario**.
7. En la hoja **Agregar usuarios** , escriba la dirección de correo electrónico o el nombre del usuario que desea agregar como propietario. Si no se encuentra el usuario, aparecerá un mensaje de error que explica el problema. Si se encuentra el usuario, dicho usuario se mostrará en el cuadro de texto **Usuario** .
8. Seleccione el nombre de usuario encontrado.
9. Elija **Seleccionar**.
10. Seleccione **Aceptar** para cerrar la hoja **Agregar acceso**.
11. Cuando vuelva a la hoja **Usuarios** , el usuario ya se habrá agregado como propietario. Este usuario será ya propietario de cualquier laboratorio creado con esta suscripción y, por lo tanto, podrá realizar tareas de propietario. 

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]
