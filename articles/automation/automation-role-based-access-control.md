---
title: Administración de seguridad y permisos de roles en Azure Automation
description: En este artículo se describe el uso del control de acceso basado en rol (RBAC), que permite la administración del acceso en los recursos de Azure.
keywords: automatización de rbac, control de acceso basado en roles, rbac de azure
services: automation
ms.subservice: shared-capabilities
ms.date: 07/21/2020
ms.topic: conceptual
ms.openlocfilehash: 21742d2db6a7fde69568e5fd1e5eda98542faa47
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87528675"
---
# <a name="manage-role-permissions-and-security"></a>Administración de seguridad y permisos de roles

El control de acceso basado en rol (RBAC) permite la administración del acceso en los recursos de Azure. Con [RBAC](../role-based-access-control/overview.md), se pueden separar los deberes del equipo y conceder a los usuarios, grupos y aplicaciones únicamente el acceso que necesiten para su trabajo. El acceso basado en rol se puede conceder a los usuarios mediante Azure Portal, las herramientas de la línea de comandos de Azure o las API de administración de Azure.

## <a name="roles-in-automation-accounts"></a>Roles en cuentas de Automation

En Azure Automation, el acceso se concede mediante la asignación del rol de Azure adecuado a los usuarios, grupos y aplicaciones en el ámbito de las cuentas de Automation. Estos son los roles integrados compatibles que admiten las cuentas de Automation:

| **Rol** | **Descripción** |
|:--- |:--- |
| Propietario |El rol Propietario permite el acceso a todos los recursos y las acciones de una cuenta de Automation, lo que incluye proporcionar acceso a otros usuarios, grupos y aplicaciones para que administren la cuenta de Automation. |
| Colaborador |El rol Colaborador permite administrar todo, excepto la modificación de los permisos de acceso de otros usuarios a una cuenta de Automation. |
| Lector |El rol Lector permite ver todos los recursos de una cuenta de Automation, pero no realizar cambios. |
| Operador de Automation |El rol de función de Operador de Automation permite ver las propiedades y el nombre del runbook, y crear y administrar trabajos para todos los runbooks de una cuenta de Automation. Este rol es útil si se desea proteger los recursos de la cuenta de Automation, como los recursos de credenciales y los runbooks, para que no puedan verse ni modificarse, pero permitir a los miembros de la organización ejecutar los runbooks. |
|Operador de trabajos de Automation|El rol de Operador de trabajos de Automation permite crear y administrar trabajos para todos los runbooks de una cuenta de Automation.|
|Operador de runbooks de Automation|El rol de Operador de runbooks de Automation permite leer el nombre y las propiedades de los runbooks.|
| Colaborador de Log Analytics | El rol de colaborador de Log Analytics permite leer todos los datos de supervisión y editar la configuración de supervisión. La edición de la configuración de supervisión incluye la posibilidad de agregar la extensión de máquina virtual a las máquinas virtuales, leer las claves de las cuentas de almacenamiento para poder configurar la recopilación de registros de Azure Storage, crear y configurar cuentas de Automation, agregar características de Azure Automation y configurar Azure Diagnostics en todos los recursos de Azure.|
| Lector de Log Analytics | El rol de lector de Log Analytics permite ver y buscar todos los datos de supervisión, así como ver la configuración de supervisión. Esto incluye la visualización de la configuración de Azure Diagnostics en todos los recursos de Azure. |
| Colaborador de supervisión | El rol de colaborador de supervisión permite leer todos los datos de supervisión y actualizar la configuración de supervisión.|
| Lector de supervisión | El rol de lector de supervisión permite leer todos los datos de supervisión. |
| Administrador de acceso de usuario |El rol Administrador de acceso de usuario permite administrar el acceso de los usuarios a las cuentas de Azure Automation. |

## <a name="role-permissions"></a>Permisos de los roles

Las tablas siguientes describen los permisos específicos concedidos a cada rol. En ellas, se incluyen elementos correspondientes a las categorías Actions (para conceder permisos) y NotActions (para restringirlos).

### <a name="owner"></a>Propietario

Un propietario puede administrar todo, incluido el acceso. La siguiente tabla muestra los permisos concedidos para el rol:

|Acciones|Descripción|
|---|---|
|Microsoft.Automation/automationAccounts/|Crear y administrar recursos de todos los tipos.|

### <a name="contributor"></a>Colaborador

Un colaborador puede administrar todo, excepto el acceso. La siguiente tabla muestra los permisos concedidos y denegados para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/|Crear y administrar recursos de todos los tipos|
|**No acciones**||
|Microsoft.Authorization/*/Delete| Eliminar roles y asignaciones de roles.       |
|Microsoft.Authorization/*/Write     |  Crear roles y asignaciones de roles.       |
|Microsoft.Authorization/elevateAccess/Action    | Deniega la capacidad de crear un administrador de acceso de usuario.       |

### <a name="reader"></a>Lector

Un lector puede ver todos los recursos de una cuenta de Automation, pero no realizar cambios.

|**Acciones**  |**Descripción**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/read|Ver todos los recursos en una cuenta de Automation. |

### <a name="automation-operator"></a>Operador de Automation

El rol de Operador de Automation permite crear y administrar trabajos, y leer las propiedades y el nombre del runbook de todos los runbooks de una cuenta de Automation.

>[!NOTE]
>Si desea controlar el acceso de operador a runbooks individuales, no establezca este rol. En su lugar, use los roles **operador de trabajo de Automation** y el **operador de runbook de Automation** en combinación.

La siguiente tabla muestra los permisos concedidos para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|Microsoft.Authorization/*/read|Autorización de lectura.|
|Microsoft.Automation/automationAccounts/hybridRunbookWorkerGroups/read|Lee los recursos de Hybrid Runbook Worker.|
|Microsoft.Automation/automationAccounts/jobs/read|Enumerar trabajos del runbook.|
|Microsoft.Automation/automationAccounts/jobs/resume/action|Reanudar un trabajo que está en pausa.|
|Microsoft.Automation/automationAccounts/jobs/stop/action|Cancelar un trabajo en curso.|
|Microsoft.Automation/automationAccounts/jobs/streams/read|Leer los flujos de trabajo y los resultados.|
|Microsoft.Automation/automationAccounts/jobs/output/read|Obtiene la salida de un trabajo.|
|Microsoft.Automation/automationAccounts/jobs/suspend/action|Pausar un trabajo en curso.|
|Microsoft.Automation/automationAccounts/jobs/write|Crear trabajos.|
|Microsoft.Automation/automationAccounts/jobSchedules/read|Obtiene una programación de trabajos de Azure Automation.|
|Microsoft.Automation/automationAccounts/jobSchedules/write|Crea una programación de trabajos de Azure Automation.|
|Microsoft.Automation/automationAccounts/linkedWorkspace/read|Obtiene el área de trabajo vinculada a la cuenta de Automation.|
|Microsoft.Automation/automationAccounts/read|Obtiene una cuenta de Azure Automation.|
|Microsoft.Automation/automationAccounts/runbooks/read|Obtiene un runbook de Azure Automation.|
|Microsoft.Automation/automationAccounts/schedules/read|Obtiene un recurso de programación de Azure Automation.|
|Microsoft.Automation/automationAccounts/schedules/write|Crea o actualiza un recurso de programación de Azure Automation.|
|Microsoft.Resources/subscriptions/resourceGroups/read      |Roles de lectura y asignaciones de roles.         |
|Microsoft.Resources/deployments/*      |Crear y administrar implementaciones de grupos de recursos.         |
|Microsoft.Insights/alertRules/*      | Crear y administrar reglas de alertas.        |
|Microsoft.Support/* |Crear y administrar incidencias de soporte técnico.|

### <a name="automation-job-operator"></a>Operador de trabajos de Automation

El rol de Operador de trabajos de Automation se asigna en el ámbito de la cuenta de Automation. Esto permite a los roles con permiso de Operador crear y administrar trabajos para todos los runbooks de la cuenta. Si a la función de operador de trabajo se le conceden permisos de lectura en el grupo de recursos que contiene la cuenta de Automation, los miembros del rol tendrán la capacidad de iniciar runbooks. Sin embargo, no tienen la capacidad de crearlos, modificarlos ni eliminarlos.

La siguiente tabla muestra los permisos concedidos para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|Microsoft.Authorization/*/read|Autorización de lectura.|
|Microsoft.Automation/automationAccounts/jobs/read|Enumerar trabajos del runbook.|
|Microsoft.Automation/automationAccounts/jobs/resume/action|Reanudar un trabajo que está en pausa.|
|Microsoft.Automation/automationAccounts/jobs/stop/action|Cancelar un trabajo en curso.|
|Microsoft.Automation/automationAccounts/jobs/streams/read|Leer los flujos de trabajo y los resultados.|
|Microsoft.Automation/automationAccounts/jobs/suspend/action|Pausar un trabajo en curso.|
|Microsoft.Automation/automationAccounts/jobs/write|Crear trabajos.|
|Microsoft.Resources/subscriptions/resourceGroups/read      |  Roles de lectura y asignaciones de roles.       |
|Microsoft.Resources/deployments/*      |Crear y administrar implementaciones de grupos de recursos.         |
|Microsoft.Insights/alertRules/*      | Crear y administrar reglas de alertas.        |
|Microsoft.Support/* |Crear y administrar incidencias de soporte técnico.|

### <a name="automation-runbook-operator"></a>Operador de runbooks de Automation

El rol de operador de runbooks de Automation se concede en el ámbito del runbook. Un Operador de runbooks de Automation puede ver las propiedades y el nombre del runbook. Este rol, combinado con el de **operador de trabajos de Automation**, permite también al operador crear y administrar trabajos para el runbook. La siguiente tabla muestra los permisos concedidos para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/runbooks/read     | Enumerar los runbooks.        |
|Microsoft.Authorization/*/read      | Autorización de lectura.        |
|Microsoft.Resources/subscriptions/resourceGroups/read      |Roles de lectura y asignaciones de roles.         |
|Microsoft.Resources/deployments/*      | Crear y administrar implementaciones de grupos de recursos.         |
|Microsoft.Insights/alertRules/*      | Crear y administrar reglas de alertas.        |
|Microsoft.Support/*      | Crear y administrar incidencias de soporte técnico.        |

### <a name="log-analytics-contributor"></a>Colaborador de Log Analytics

Un colaborador de Log Analytics puede leer todos los datos de supervisión y editar la configuración de supervisión. La edición de la configuración de supervisión incluye la posibilidad de agregar la extensión de máquina virtual a las máquinas virtuales, leer las claves de las cuentas de almacenamiento para poder configurar la recopilación de registros de Azure Storage, crear y configurar cuentas de Automation, agregar características y configurar Azure Diagnostics en todos los recursos de Azure. La siguiente tabla muestra los permisos concedidos para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|*/read|Leer recursos de todos los tipos, excepto secretos.|
|Microsoft.Automation/automationAccounts/*|Administrar cuentas de Automation.|
|Microsoft.ClassicCompute/virtualMachines/extensions/*|Crear y administrar extensiones de máquinas virtuales.|
|Microsoft.ClassicStorage/storageAccounts/listKeys/action|Enumerar claves de cuentas de almacenamiento clásicas.|
|Microsoft.Compute/virtualMachines/extensions/*|Crear y administrar extensiones de máquinas virtuales clásicas.|
|Microsoft.Insights/alertRules/*|Reglas de alerta de lectura, escritura y eliminación.|
|Microsoft.Insights/diagnosticSettings/*|Configuración de diagnóstico de lectura, escritura y eliminación.|
|Microsoft.OperationalInsights/*|Administrar registros de Azure Monitor.|
|Microsoft.OperationsManagement/*|Administración de características de Azure Automation en áreas de trabajo.|
|Microsoft.Resources/deployments/*|Crear y administrar implementaciones de grupos de recursos.|
|Microsoft.Resources/subscriptions/resourcegroups/deployments/*|Crear y administrar implementaciones de grupos de recursos.|
|Microsoft.Storage/storageAccounts/listKeys/action|Enumerar claves de cuentas de almacenamiento.|
|Microsoft.Support/*|Crear y administrar incidencias de soporte técnico.|

### <a name="log-analytics-reader"></a>Lector de Log Analytics

Un lector de Log Analytics puede ver y buscar los datos de supervisión, así como consultar la configuración, incluida la de Azure Diagnostics en todos los recursos de Azure. La siguiente tabla muestra los permisos concedidos o denegados para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|*/read|Leer recursos de todos los tipos, excepto secretos.|
|Microsoft.OperationalInsights/workspaces/analytics/query/action|Administrar consultas de búsqueda en registros de Azure Monitor.|
|Microsoft.OperationalInsights/workspaces/search/action|Buscar datos de registros de Azure Monitor.|
|Microsoft.Support/*|Crear y administrar incidencias de soporte técnico.|
|**No acciones**| |
|Microsoft.OperationalInsights/workspaces/sharedKeys/read|No se pueden leer las claves de acceso compartido.|

### <a name="monitoring-contributor"></a>Colaborador de supervisión

Un colaborador de supervisión puede leer todos los datos de supervisión y actualizar la configuración de supervisión. La siguiente tabla muestra los permisos concedidos para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|*/read|Leer recursos de todos los tipos, excepto secretos.|
|Microsoft.AlertsManagement/alerts/*|Administrar alertas.|
|Microsoft.AlertsManagement/alertsSummary/*|Administrar el panel de alertas.|
|Microsoft.Insights/AlertRules/*|Administrar reglas de alertas.|
|Microsoft.Insights/components/*|Administrar componentes de Application Insights.|
|Microsoft.Insights/DiagnosticSettings/*|Administrar la configuración de diagnóstico.|
|Microsoft.Insights/eventtypes/*|Enumerar eventos del registro de actividades (eventos de administración) de una suscripción. Este permiso es aplicable para el acceso mediante programación y mediante el portal al registro de actividades.|
|Microsoft.Insights/LogDefinitions/*|Este permiso es necesario para los usuarios que necesitan acceder a registros de actividades a través del portal. Enumere las categorías de registro del registro de actividad.|
|Microsoft.Insights/MetricDefinitions/*|Leer definiciones de métrica (lista de tipos de métricas disponibles para un recurso).|
|Microsoft.Insights/Metrics/*|Leer las métricas de un recurso.|
|Microsoft.Insights/Register/Action|Registrar el proveedor de Microsoft.Insights.|
|Microsoft.Insights/webtests/*|Administrar pruebas web de Application Insights.|
|Microsoft.OperationalInsights/workspaces/intelligencepacks/*|Administrar paquetes de soluciones de registros de Azure Monitor.|
|Microsoft.OperationalInsights/workspaces/savedSearches/*|Administrar búsquedas guardadas de registros de Azure Monitor.|
|Microsoft.OperationalInsights/workspaces/search/action|Buscar áreas de trabajo de Log Analytics.|
|Microsoft.OperationalInsights/workspaces/sharedKeys/action|Enumerar las claves de un área de trabajo de Log Analytics.|
|Microsoft.OperationalInsights/workspaces/storageinsightconfigs/*|Administrar configuraciones de visión de almacenamiento de registros de Azure Monitor.|
|Microsoft.Support/*|Crear y administrar incidencias de soporte técnico.|
|Microsoft.WorkloadMonitor/workloads/*|Administrar cargas de trabajo.|

### <a name="monitoring-reader"></a>Lector de supervisión

Un lector de supervisión puede leer todos los datos de supervisión. La siguiente tabla muestra los permisos concedidos para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|*/read|Leer recursos de todos los tipos, excepto secretos.|
|Microsoft.OperationalInsights/workspaces/search/action|Buscar áreas de trabajo de Log Analytics.|
|Microsoft.Support/*|Crear y administrar incidencias de soporte técnico|

### <a name="user-access-administrator"></a>Administrador de acceso de usuario

Un administrador de acceso de usuario puede administrar el acceso de los usuarios a los recursos de Azure. La siguiente tabla muestra los permisos concedidos para el rol:

|**Acciones**  |**Descripción**  |
|---------|---------|
|*/read|Leer todos los recursos.|
|Microsoft.Authorization/*|Administrar la autorización|
|Microsoft.Support/*|Crear y administrar incidencias de soporte técnico|

## <a name="feature-setup-permissions"></a>Permisos de configuración de características

En las secciones siguientes se describen los permisos mínimos necesarios para habilitar las características Update Management y Change Tracking e Inventario.

### <a name="permissions-for-enabling-update-management-and-change-tracking-and-inventory-from-a-vm"></a>Permisos para habilitar Update Management y Change Tracking e Inventario en una máquina virtual

|**Acción**  |**Permiso**  |**Ámbito mínimo**  |
|---------|---------|---------|
|Escribir nueva implementación      | Microsoft.Resources/deployments/*          |Subscription          |
|Escribir nuevo grupo de recursos      | Microsoft.Resources/subscriptions/resourceGroups/write        | Suscripción          |
|Crear nueva área de trabajo predeterminada      | Microsoft.OperationalInsights/workspaces/write         | Resource group         |
|Crear nueva cuenta      |  Microsoft.Automation/automationAccounts/write        |Resource group         |
|Vincular área de trabajo y cuenta      |Microsoft.OperationalInsights/workspaces/write</br>Microsoft.Automation/automationAccounts/read|Área de trabajo</br>Cuenta de Automation
|Crear extensión MMA      | Microsoft.Compute/virtualMachines/write         | Máquina virtual         |
|Crear búsqueda guardada      | Microsoft.OperationalInsights/workspaces/write          | Área de trabajo         |
|Crear configuración de ámbito      | Microsoft.OperationalInsights/workspaces/write          | Área de trabajo         |
|Comprobación de estado de incorporación: leer área trabajo      | Microsoft.OperationalInsights/workspaces/read         | Área de trabajo         |
|Comprobación de estado de incorporación: leer propiedad de área de trabajo vinculada de la cuenta     | Microsoft.Automation/automationAccounts/read      | Cuenta de Automation        |
|Comprobación de estado de incorporación: leer solución      | Microsoft.OperationalInsights/workspaces/intelligencepacks/read          | Solución         |
|Comprobación de estado de incorporación: leer máquina virtual      | Microsoft.Compute/virtualMachines/read         | Máquina virtual         |
|Comprobación de estado de incorporación: leer cuenta      | Microsoft.Automation/automationAccounts/read  |  Cuenta de Automation   |
| Incorporación de comprobación de área de trabajo para VM<sup>1</sup>       | Microsoft.OperationalInsights/workspaces/read         | Subscription         |
| Registro del proveedor de Log Analytics |Microsoft.Insights/register/action | Subscription|

<sup>1</sup> Este permiso es necesario para habilitar características a través de la experiencia de máquina virtual del portal.

### <a name="permissions-for-enabling-update-management-and-change-tracking-and-inventory-from-an-automation-account"></a>Permisos para habilitar Update Management y Change Tracking e inventario en una cuenta de Automation

|**Acción**  |**Permiso** |**Ámbito mínimo**  |
|---------|---------|---------|
|Crear una nueva implementación     | Microsoft.Resources/deployments/*        | Subscription         |
|Crear un grupo de recursos     | Microsoft.Resources/subscriptions/resourceGroups/write         | Suscripción        |
|Hoja AutomationOnboarding: crear nueva área de trabajo     |Microsoft.OperationalInsights/workspaces/write           | Resource group        |
|Hoja AutomationOnboarding: leer área de trabajo vinculada     | Microsoft.Automation/automationAccounts/read        | Cuenta de Automation       |
|Hoja AutomationOnboarding: leer solución     | Microsoft.OperationalInsights/workspaces/intelligencepacks/read         | Solución        |
|Hoja AutomationOnboarding: leer área de trabajo     | Microsoft.OperationalInsights/workspaces/intelligencepacks/read        | Área de trabajo        |
|Crear vínculo para área de trabajo y cuenta     | Microsoft.OperationalInsights/workspaces/write        | Área de trabajo        |
|Escribir cuenta para Shoebox      | Microsoft.Automation/automationAccounts/write        | Cuenta        |
|Crear o editar búsqueda guardada     | Microsoft.OperationalInsights/workspaces/write        | Área de trabajo        |
|Crear o editar la configuración de ámbito     | Microsoft.OperationalInsights/workspaces/write        | Área de trabajo        |
| Registro del proveedor de Log Analytics |Microsoft.Insights/register/action | Subscription|
|**Paso 2: Habilitación de varias máquinas virtuales**     |         |         |
|Hoja VMOnboarding: crear extensión MMA     | Microsoft.Compute/virtualMachines/write           | Máquina virtual        |
|Crear o editar búsqueda guardada     | Microsoft.OperationalInsights/workspaces/write           | Área de trabajo        |
|Crear o editar la configuración de ámbito  | Microsoft.OperationalInsights/workspaces/write   | Área de trabajo|

## <a name="update-management-permissions"></a>Permisos de administración de actualizaciones

La administración de actualizaciones cubre varios servicios para proporcionar su servicio. La siguiente tabla muestra los permisos necesarios para administrar las implementaciones de administración de actualizaciones:

|**Recurso**  |**Rol**  |**Ámbito**  |
|---------|---------|---------|
|Cuenta de Automation     | Colaborador de Log Analytics       | Cuenta de Automation        |
|Cuenta de Automation    | Colaborador de la máquina virtual        | Grupo de recursos para la cuenta        |
|Área de trabajo de Log Analytics     | Colaborador de Log Analytics| Área de trabajo de Log Analytics        |
|Área de trabajo de Log Analytics |Lector de Log Analytics| Subscription|
|Solución     |Colaborador de Log Analytics         | Solución|
|Máquina virtual     | Colaborador de la máquina virtual        | Máquina virtual        |

## <a name="configure-rbac-for-your-automation-account"></a>Configuración de RBAC para una cuenta de Automation

En la sección siguiente se muestra cómo configurar RBAC en la cuenta de Automation mediante [Azure Portal](#configure-rbac-using-the-azure-portal) y [PowerShell](#configure-rbac-using-powershell).

### <a name="configure-rbac-using-the-azure-portal"></a>Configuración de RBAC mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y abra su cuenta de Automation en la página Cuentas de Automation.
2. Haga clic en **Control de acceso (IAM)** para abrir la página Control de acceso (IAM). Puede usar esta página para agregar nuevos usuarios, grupos y aplicaciones para administrar la cuenta de Automation y ver los roles existentes que se pueden configurar para la cuenta de Automation.
3. Haga clic en la pestaña **Asignaciones de roles**.

   ![Botón de acceso](media/automation-role-based-access-control/automation-01-access-button.png)

#### <a name="add-a-new-user-and-assign-a-role"></a>Adición de usuarios nuevos y asignación de roles

1. En la página Control de acceso (IAM), haga clic en **+ Agregar asignación de roles**. Esta acción abre la página Agregar asignación de roles, donde puede agregar un usuario, un grupo o una aplicación y asignar el rol correspondiente.

2. Seleccione en rol en la lista de roles disponibles. Puede elegir cualquiera de los roles integrados disponibles que admitan las cuentas de Automation o cualquier rol personalizado que haya definido.

3. Escriba el nombre del usuario al que desee conceder permisos en el campo **Seleccionar**. Elija el usuario de la lista y haga clic en **Guardar**.

   ![Agregar usuarios](media/automation-role-based-access-control/automation-04-add-users.png)

   Ahora debería ver el usuario agregado en la página Usuarios con el rol seleccionado asignado.

   ![Enumerar usuarios](media/automation-role-based-access-control/automation-05-list-users.png)

   También se puede asignar un rol al usuario desde la página Roles.

4. Haga clic en **Roles** en la página Control de acceso (IAM) para abrir la página Roles. Puede ver el nombre del rol y el número de usuarios y grupos asignados a dicho rol.

    ![Asignar rol en la página de usuarios](media/automation-role-based-access-control/automation-06-assign-role-from-users-blade.png)

   > [!NOTE]
   > El control de acceso basado en roles solo se puede establecer en el ámbito de la cuenta de Automation, no en los recursos por debajo de la cuenta de Automation.

#### <a name="remove-a-user"></a>Eliminación de un usuario

Puede quitar el permiso de usuario de cualquier usuario que no administre la cuenta de Automation o que haya dejado de trabajar en la organización. Estos son los pasos que deben seguirse para eliminar un usuario:

1. En la página Control de acceso (IAM), seleccione el usuario que desea eliminar y haga clic en **Eliminar**.
2. Haga clic en el botón **Quitar** del panel de detalles de asignación.
3. Haga clic en **Sí** para confirmar la eliminación.

   ![Quitar usuarios](media/automation-role-based-access-control/automation-08-remove-users.png)

### <a name="configure-rbac-using-powershell"></a>Configuración de RBAC mediante PowerShell

El acceso basado en rol también se puede configurar en una cuenta de Automation mediante los siguientes [cmdlets de Azure PowerShell](../role-based-access-control/role-assignments-powershell.md):

[Get-AzRoleDefinition](/powershell/module/Az.Resources/Get-AzRoleDefinition?view=azps-3.7.0) enumera todos los roles de Azure disponibles en Azure Active Directory. Puede usar este cmdlet con el parámetro `Name` para enumerar todas las acciones que puede realizar un rol específico.

```azurepowershell-interactive
Get-AzRoleDefinition -Name 'Automation Operator'
```

A continuación se incluye el resultado de ejemplo:

```azurepowershell
Name             : Automation Operator
Id               : d3881f73-407a-4167-8283-e981cbba0404
IsCustom         : False
Description      : Automation Operators are able to start, stop, suspend, and resume jobs
Actions          : {Microsoft.Authorization/*/read, Microsoft.Automation/automationAccounts/jobs/read, Microsoft.Automation/automationAccounts/jobs/resume/action,
                   Microsoft.Automation/automationAccounts/jobs/stop/action...}
NotActions       : {}
AssignableScopes : {/}
```

[Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment?view=azps-3.7.0) enumera las asignaciones de roles de Azure en el ámbito especificado. Sin parámetros, este cmdlet devuelve todas las asignaciones de roles realizadas en la suscripción. Use el parámetro `ExpandPrincipalGroups` para enumerar las asignaciones de acceso del usuario especificado, así como los grupos a los que pertenezca.

**Ejemplo**: use el siguiente cmdlet para enumerar todos los usuarios de una cuenta de Automation y sus roles.

```azurepowershell-interactive
Get-AzRoleAssignment -Scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

A continuación se incluye el resultado de ejemplo:

```powershell
RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Automation/automationAccounts/myAutomationAccount/provid
                     ers/Microsoft.Authorization/roleAssignments/cc594d39-ac10-46c4-9505-f182a355c41f
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Automation/automationAccounts/myAutomationAccount
DisplayName        : admin@contoso.com
SignInName         : admin@contoso.com
RoleDefinitionName : Automation Operator
RoleDefinitionId   : d3881f73-407a-4167-8283-e981cbba0404
ObjectId           : 15f26a47-812d-489a-8197-3d4853558347
ObjectType         : User
```

Use [New-AzRoleAssignment](/powershell/module/Az.Resources/New-AzRoleAssignment?view=azps-3.7.0) para asignar acceso a usuarios, grupos y aplicaciones en un ámbito determinado.

**Ejemplo**: use el siguiente comando para asignar el rol "Operador de Automation" para un usuario en el ámbito de la cuenta de Automation.

```azurepowershell-interactive
New-AzRoleAssignment -SignInName <sign-in Id of a user you wish to grant access> -RoleDefinitionName 'Automation operator' -Scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

A continuación se incluye el resultado de ejemplo:

```azurepowershell
RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/myResourceGroup/Providers/Microsoft.Automation/automationAccounts/myAutomationAccount/provid
                     ers/Microsoft.Authorization/roleAssignments/25377770-561e-4496-8b4f-7cba1d6fa346
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/myResourceGroup/Providers/Microsoft.Automation/automationAccounts/myAutomationAccount
DisplayName        : admin@contoso.com
SignInName         : admin@contoso.com
RoleDefinitionName : Automation Operator
RoleDefinitionId   : d3881f73-407a-4167-8283-e981cbba0404
ObjectId           : f5ecbe87-1181-43d2-88d5-a8f5e9d8014e
ObjectType         : User
```

Use [Remove-AzRoleAssignment](/powershell/module/Az.Resources/Remove-AzRoleAssignment?view=azps-3.7.0) para quitar el acceso a un usuario, grupo o aplicación concretos de un ámbito determinado.

**Ejemplo**: Use el siguiente comando para quitar el usuario del rol Operador de Automation en el ámbito de la cuenta de Automation.

```azurepowershell-interactive
Remove-AzRoleAssignment -SignInName <sign-in Id of a user you wish to remove> -RoleDefinitionName 'Automation Operator' -Scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

En el ejemplo anterior, reemplace `sign-in ID of a user you wish to remove`, `SubscriptionID`, `Resource Group Name` y `Automation account name` por los detalles de su cuenta. Elija **Sí** cuando se le pida que confirme antes de proceder a la eliminación de las asignaciones de roles de usuario.

### <a name="user-experience-for-automation-operator-role---automation-account"></a>Experiencia del usuario en el rol Operador de Automation: cuenta de Automation

Cuando un usuario asignado al rol Operador de Automation en el ámbito de la cuenta de Automation vea la cuenta de Automation a la que está asignado, solo verá la lista de runbooks, los trabajos de runbook y las programaciones creados en la cuenta de Automation. Este usuario no puede ver las definiciones de estos elementos. El usuario puede iniciar, detener, suspender, reanudar o programar el trabajo de runbook. Sin embargo, el usuario no tiene acceso a otros recursos de Automation como configuraciones, grupos de Hybrid Worker o nodos de DSC.

![Sin acceso a los recursos](media/automation-role-based-access-control/automation-10-no-access-to-resources.png)

## <a name="configure-rbac-for-runbooks"></a>Configuración de RBAC para runbooks

Azure Automation permite asignar RBAC a runbooks específicos. Para ello, ejecute el siguiente script para agregar un usuario a un runbook específico. Un administrador de la cuenta de Automation o un administrador de inquilinos pueden ejecutar este script.

```azurepowershell-interactive
$rgName = "<Resource Group Name>" # Resource Group name for the Automation account
$automationAccountName ="<Automation account name>" # Name of the Automation account
$rbName = "<Name of Runbook>" # Name of the runbook
$userId = "<User ObjectId>" # Azure Active Directory (AAD) user's ObjectId from the directory

# Gets the Automation account resource
$aa = Get-AzResource -ResourceGroupName $rgName -ResourceType "Microsoft.Automation/automationAccounts" -ResourceName $automationAccountName

# Get the Runbook resource
$rb = Get-AzResource -ResourceGroupName $rgName -ResourceType "Microsoft.Automation/automationAccounts/runbooks" -ResourceName "$rbName"

# The Automation Job Operator role only needs to be run once per user.
New-AzRoleAssignment -ObjectId $userId -RoleDefinitionName "Automation Job Operator" -Scope $aa.ResourceId

# Adds the user to the Automation Runbook Operator role to the Runbook scope
New-AzRoleAssignment -ObjectId $userId -RoleDefinitionName "Automation Runbook Operator" -Scope $rb.ResourceId
```

Una vez ejecutado el script, los usuarios deberán iniciar sesión en Azure Portal y seleccionar **Todos los recursos**. En la lista, el usuario puede ver el runbook para el que se ha agregado como Operador de runbook de Automation.

![RBAC de runbook en el portal](./media/automation-role-based-access-control/runbook-rbac.png)

### <a name="user-experience-for-automation-operator-role---runbook"></a>Experiencia del usuario en el rol Operador de Automation: runbook

Cuando un usuario asignado al rol Operador de Automation en el ámbito del runbook vea un runbook al que esté asignado, solo podrá iniciar el runbook y ver los trabajos de runbook.

![Acceso solo a la página de inicio](media/automation-role-based-access-control/automation-only-start.png)

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre RBAC y PowerShell, consulte [Incorporación o eliminación de asignaciones de roles de Azure con Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).
* Para más información sobre los tipos de runbooks, consulte [Tipos de runbooks de Azure Automation](automation-runbook-types.md).
* Para iniciar un runbook, consulte [Inicio de un runbook en Azure Automation](start-runbooks.md).
