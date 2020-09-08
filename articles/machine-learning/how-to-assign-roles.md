---
title: Administración de roles del área de trabajo
titleSuffix: Azure Machine Learning
description: Conozca cómo acceder a un área de trabajo de Azure Machine Learning mediante el control de acceso basado en rol (RBAC).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: Blackmist
ms.author: nigup
author: nishankgu
ms.date: 07/24/2020
ms.custom: how-to, seodec18
ms.openlocfilehash: afffdd0267cde8ffc841587748e51dd27e021369
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079593"
---
# <a name="manage-access-to-an-azure-machine-learning-workspace"></a>Administración del acceso a un área de trabajo de Azure Machine Learning
[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, obtendrá información sobre cómo administrar el acceso a un área de trabajo de Azure Machine Learning. El [control de acceso basado en rol de Azure (RBAC de Azure)](/azure/role-based-access-control/overview) se utiliza para administrar el acceso a los recursos de Azure. Se asignan roles específicos a los usuarios de Azure Active Directory que les conceden acceso a los recursos. Azure proporciona tanto roles integrados como la capacidad de crear roles personalizados.

## <a name="default-roles"></a>Roles predeterminados

Un área de trabajo de Azure Machine Learning es un recurso de Azure. Al igual que otros recursos de Azure, cuando se crea un área de trabajo de Azure Machine Learning nueva, viene con tres roles predeterminados. Puede agregar usuarios al área de trabajo y asignarlos a uno de estos roles integrados.

| Role | Nivel de acceso |
| --- | --- |
| **Lector** | Acciones de solo lectura en el área de trabajo. Los lectores pueden enumerar y ver los recursos (incluidas las credenciales del [almacén de datos](how-to-access-data.md)) de un área de trabajo, pero no pueden crear ni actualizar estos recursos. |
| **Colaborador** | Ver, crear, editar o eliminar (si procede) los recursos de un área de trabajo. Por ejemplo, los colaboradores pueden crear un experimento, crear o conectar un clúster de proceso, enviar una ejecución e implementar un servicio web. |
| **Propietario** | Acceso total al área de trabajo, incluida la posibilidad de ver, crear, editar o eliminar (si procede) los recursos de un área de trabajo. Además, puede cambiar las asignaciones de roles. |
| **Rol personalizado** | Permite personalizar el acceso a las operaciones de control o de plano de datos específicas dentro de un área de trabajo. Por ejemplo, el envío de una ejecución, la creación de un proceso, la implementación de un modelo o el registro de un conjunto de registros. |

> [!IMPORTANT]
> El acceso de los roles puede tener un ámbito de varios niveles en Azure. Por ejemplo, un usuario con acceso de propietario a un área de trabajo podría no tener acceso de propietario al grupo de recursos que contiene el área de trabajo. Para más información, consulte [Cómo funciona RBAC](/azure/role-based-access-control/overview#how-rbac-works).

Para más información sobre roles integrados específicos, consulte [Roles integrados de Azure](/azure/role-based-access-control/built-in-roles).

## <a name="manage-workspace-access"></a>Administración del acceso al área de trabajo

Si es propietario de un área de trabajo, puede agregar y eliminar roles del área de trabajo. También puede asignar roles a los usuarios. Use los siguientes vínculos para descubrir cómo administrar el acceso:
- [Interfaz de usuario de Azure Portal](/azure/role-based-access-control/role-assignments-portal)
- [PowerShell](/azure/role-based-access-control/role-assignments-powershell)
- [CLI de Azure](/azure/role-based-access-control/role-assignments-cli)
- [REST API](/azure/role-based-access-control/role-assignments-rest)
- [Plantillas del Administrador de recursos de Azure](/azure/role-based-access-control/role-assignments-template)

Si ha instalado la [CLI de Azure Machine Learning](reference-azure-machine-learning-cli.md), puede usar comandos de la CLI para asignar roles a los usuarios.

```azurecli-interactive 
az ml workspace share -w <workspace_name> -g <resource_group_name> --role <role_name> --user <user_corp_email_address>
```

El campo `user` es la dirección de correo electrónico de un usuario existente en la instancia de Azure Active Directory donde se encuentra la suscripción principal del área de trabajo. Este es un ejemplo de cómo utilizar este comando:

```azurecli-interactive 
az ml workspace share -w my_workspace -g my_resource_group --role Contributor --user jdoe@contoson.com
```

> [!NOTE]
> El comando "az ml workspace share" no funciona para la cuenta federada de Azure Active Directory B2B. Use el portal de la interfaz de usuario de Azure en lugar del comando.


## <a name="azure-machine-learning-operations"></a>Operaciones de Azure Machine Learning

Las acciones integradas de Azure Machine Learning se utilizan para muchas operaciones y tareas. Para obtener una lista completa, consulte [Operaciones del proveedor de recursos de Azure](/azure/role-based-access-control/resource-provider-operations#microsoftmachinelearningservices).

## <a name="create-custom-role"></a>Creación de un rol personalizado

Si los roles integrados no son suficientes, puede crear roles personalizados. Los roles personalizados pueden tener permisos para leer, escribir, eliminar y procesar recursos de ese área de trabajo. Puede hacer que el rol esté disponible en un nivel de área de trabajo específico, un nivel de grupo de recursos específico o un nivel de suscripción específico.

> [!NOTE]
> Debe ser un propietario del recurso en ese nivel para crear roles personalizados en ese recurso.

Para crear un rol personalizado, primero debe construir un archivo JSON de definición de rol que especifica el permiso y el ámbito del rol. En el ejemplo siguiente se define un rol personalizado llamado "Data Scientist Custom" (Personalizado para científico de datos) con ámbito en un nivel de área de trabajo específica:

`data_scientist_custom_role.json` :
```json
{
    "Name": "Data Scientist Custom",
    "IsCustom": true,
    "Description": "Can run experiment but can't create or delete compute.",
    "Actions": ["*"],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/*/delete",
        "Microsoft.MachineLearningServices/workspaces/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/delete", 
        "Microsoft.Authorization/*/write"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>/resourceGroups/<resource_group_name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace_name>"
    ]
}
```

> [!TIP]
> Puede cambiar el campo `AssignableScopes` para establecer el ámbito de este rol personalizado en el nivel de suscripción, el nivel de grupo de recursos o un nivel de área de trabajo específica.
> El rol personalizado anterior es solo un ejemplo, consulte algunos [roles personalizados sugeridos para el servicio Azure Machine Learning Service](#customroles).

Este rol personalizado puede hacer todo en el área de trabajo, excepto las siguientes acciones:

- No puede crear o actualizar un recurso de proceso.
- No puede eliminar un recurso de proceso.
- No puede agregar, eliminar o modificar asignaciones de roles.
- No puede eliminar el área de trabajo.

Para implementar este rol personalizado, use el siguiente comando de la CLI de Azure:

```azurecli-interactive 
az role definition create --role-definition data_scientist_role.json
```

Después de la implementación, este rol está disponible en el área de trabajo especificada. Ahora puede agregar y asignar este rol en Azure Portal. O bien, puede asignar este rol a un usuario mediante el comando de la CLI `az ml workspace share`:

```azurecli-interactive
az ml workspace share -w my_workspace -g my_resource_group --role "Data Scientist" --user jdoe@contoson.com
```

Para más información sobre los roles personalizados, consulte [Roles personalizados de Azure](/azure/role-based-access-control/custom-roles). Para más información sobre las operaciones (acciones y no acciones) que se pueden usar con roles personalizados, consulte [Operaciones del proveedor de recursos](/azure/role-based-access-control/resource-provider-operations#microsoftmachinelearningservices).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes


### <a name="q-what-are-the-permissions-needed-to-perform-some-common-scenarios-in-the-azure-machine-learning-service"></a>Q. ¿Cuáles son los permisos necesarios para realizar algunos escenarios comunes en Azure Machine Learning Service?

En la tabla siguiente se proporciona un resumen de las actividades de Azure Machine Learning y los permisos necesarios para realizarlas en el menor ámbito. Por ejemplo, si una actividad se puede realizar con un ámbito de área de trabajo (columna 4), cualquier el ámbito superior con ese permiso también funcionará automáticamente.

> [!IMPORTANT]
> Todas las rutas de acceso de esta tabla que comienzan por `/` son **rutas de acceso relativas** a `Microsoft.MachineLearningServices/`:

| Actividad | Ámbito a nivel de supervisión | Ámbito a nivel de grupo de recursos | Ámbito a nivel de área de trabajo |
| ----- | ----- | ----- | ----- |
| Crear área de trabajo nueva | No se requiere | Propietario o colaborador | N/A (se convierte en Propietario o hereda el rol del ámbito superior después de la creación) |
| Actualización de la edición del área de trabajo | No se requiere | No se requiere | Propietario, colaborador o rol personalizado que permita: `/workspaces/write` |
| Solicitud de cuota de Amlcompute de nivel de suscripción o establecimiento de cuota de nivel de área de trabajo | Propietario, colaborador o rol personalizado </br>que permite `/locations/updateQuotas/action`</br> en el ámbito de suscripción | No autorizado | No autorizado |
| Crear un clúster de proceso | No se requiere | No se requiere | Propietario, colaborador o rol personalizado que permita: `/workspaces/computes/write` |
| Crear una instancia de proceso | No se requiere | No se requiere | Propietario, colaborador o rol personalizado que permita: `/workspaces/computes/write` |
| Envío de cualquier tipo de ejecución | No se requiere | No se requiere | Propietario, colaborador o rol personalizado que permita: `"/workspaces/*/read", "/workspaces/environments/write", "/workspaces/experiments/runs/write", "/workspaces/metadata/artifacts/write", "/workspaces/metadata/snapshots/write", "/workspaces/environments/build/action", "/workspaces/experiments/runs/submit/action", "/workspaces/environments/readSecrets/action"` |
| Publicación de un punto de conexión de canalización | No se requiere | No se requiere | Propietario, colaborador o rol personalizado que permita: `"/workspaces/pipelines/write", "/workspaces/endpoints/pipelines/*", "/workspaces/pipelinedrafts/*", "/workspaces/modules/*"` |
| Implementación de un modelo registrado en un recurso AKS/ACI | No se requiere | No se requiere | Propietario, colaborador o rol personalizado que permita: `"/workspaces/services/aks/write", "/workspaces/services/aci/write"` |
| Puntuación con un punto de conexión de AKS implementado | No se requiere | No se requiere | Propietario, colaborador o rol personalizado que permite: `"/workspaces/services/aks/score/action", "/workspaces/services/aks/listkeys/action"` (cuando no se usa la autenticación de Azure Active Directory) o `"/workspaces/read"` (cuando se usa la autenticación por tokens) |
| Acceso al almacenamiento mediante cuadernos interactivos | No se requiere | No se requiere | Propietario, colaborador o rol personalizado que permita: `"/workspaces/computes/read", "/workspaces/notebooks/samples/read", "/workspaces/notebooks/storage/*"` |
| Creación de un nuevo rol personalizado | Propietario, colaborador o rol personalizado que permita `Microsoft.Authorization/roleDefinitions/write`. | No se requiere | Propietario, colaborador o rol personalizado que permita: `/workspaces/computes/write` |

> [!TIP]
> Si recibe un error al intentar crear un área de trabajo por primera vez, asegúrese de que su rol permita `Microsoft.MachineLearningServices/register/action`. Esta acción le permite registrar el proveedor de recursos de Azure Machine Learning con su suscripción de Azure.

### <a name="q-are-we-publishing-azure-built-in-roles-for-the-machine-learning-service"></a>Q. ¿Se publican roles integrados de Azure para el servicio Machine Learning?

Actualmente, no estamos publicando [roles integrados de Azure](/azure/role-based-access-control/built-in-roles) para el servicio Machine Learning Service. Una vez publicado el rol integrado, no se puede actualizar y todavía se están confirmando las definiciones de roles en función de los escenarios y comentarios del cliente. 

<a id="customroles"></a>

### <a name="q-are-there-some-custom-role-templates-for-the-most-common-scenarios-in-machine-learning-service"></a>Q. ¿Hay plantillas de roles personalizados para los escenarios más comunes en Machine Learning Service?

Sí, estos son algunos escenarios comunes con definiciones de roles propuestos personalizados que puede usar como base para definir sus propios roles personalizados:

* __Data Scientist Custom__ (Personalizado para científico de datos): Permite que un científico de datos realice todas las operaciones dentro de un área de trabajo **excepto**:

    * Creación de proceso
    * Implementación de modelos en un clúster de AKS de producción
    * Implementación de un punto de conexión de canalización de producción

    `data_scientist_custom_role.json` :
    ```json
    {
        "Name": "Data Scientist Custom",
        "IsCustom": true,
        "Description": "Can run experiment but can't create or delete compute or deploy production endpoints.",
        "Actions": [
            "Microsoft.MachineLearningServices/workspaces/*/read",
            "Microsoft.MachineLearningServices/workspaces/*/action",
            "Microsoft.MachineLearningServices/workspaces/*/delete",
            "Microsoft.MachineLearningServices/workspaces/*/write"
        ],
        "NotActions": [
            "Microsoft.MachineLearningServices/workspaces/delete",
            "Microsoft.MachineLearningServices/workspaces/write",
            "Microsoft.MachineLearningServices/workspaces/computes/*/write",
            "Microsoft.MachineLearningServices/workspaces/computes/*/delete", 
            "Microsoft.Authorization/*",
            "Microsoft.MachineLearningServices/workspaces/computes/listKeys/action",
            "Microsoft.MachineLearningServices/workspaces/listKeys/action",
            "Microsoft.MachineLearningServices/workspaces/services/aks/write",
            "Microsoft.MachineLearningServices/workspaces/services/aks/delete",
            "Microsoft.MachineLearningServices/workspaces/endpoints/pipelines/write"
        ],
        "AssignableScopes": [
            "/subscriptions/<subscription_id>"
        ]
    }
    ```

* __Data Scientist Restricted Custom__ (Personalizado restringido para el científico de datos): Una definición de roles más restringida sin caracteres comodín en las acciones permitidas. Puede realizar todas las operaciones dentro de un área de trabajo **excepto**:

    * Creación de proceso
    * Implementación de modelos en un clúster de AKS de producción
    * Implementación de un punto de conexión de canalización de producción

    `data_scientist_restricted_custom_role.json` :
    ```json
    {
        "Name": "Data Scientist Restricted Custom",
        "IsCustom": true,
        "Description": "Can run experiment but can't create or delete compute or deploy production endpoints",
        "Actions": [
            "Microsoft.MachineLearningServices/workspaces/*/read",
            "Microsoft.MachineLearningServices/workspaces/computes/start/action",
            "Microsoft.MachineLearningServices/workspaces/computes/stop/action",
            "Microsoft.MachineLearningServices/workspaces/computes/restart/action",
            "Microsoft.MachineLearningServices/workspaces/computes/applicationaccess/action",
            "Microsoft.MachineLearningServices/workspaces/notebooks/storage/read",
            "Microsoft.MachineLearningServices/workspaces/notebooks/storage/write",
            "Microsoft.MachineLearningServices/workspaces/notebooks/storage/delete",
            "Microsoft.MachineLearningServices/workspaces/notebooks/samples/read",
            "Microsoft.MachineLearningServices/workspaces/experiments/runs/write",
            "Microsoft.MachineLearningServices/workspaces/experiments/write",
            "Microsoft.MachineLearningServices/workspaces/experiments/runs/submit/action",
            "Microsoft.MachineLearningServices/workspaces/pipelinedrafts/write",
            "Microsoft.MachineLearningServices/workspaces/metadata/snapshots/write",
            "Microsoft.MachineLearningServices/workspaces/metadata/artifacts/write",
            "Microsoft.MachineLearningServices/workspaces/environments/write",
            "Microsoft.MachineLearningServices/workspaces/models/write",
            "Microsoft.MachineLearningServices/workspaces/modules/write",
            "Microsoft.MachineLearningServices/workspaces/datasets/registered/write", 
            "Microsoft.MachineLearningServices/workspaces/datasets/registered/delete",
            "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/write",
            "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/delete",
            "Microsoft.MachineLearningServices/workspaces/computes/listNodes/action",
            "Microsoft.MachineLearningServices/workspaces/environments/build/action"
        ],
        "NotActions": [
            "Microsoft.MachineLearningServices/workspaces/computes/write",
            "Microsoft.MachineLearningServices/workspaces/write",
            "Microsoft.MachineLearningServices/workspaces/computes/delete",
            "Microsoft.MachineLearningServices/workspaces/delete",
            "Microsoft.MachineLearningServices/workspaces/computes/listKeys/action",
            "Microsoft.MachineLearningServices/workspaces/listKeys/action",
            "Microsoft.Authorization/*",
            "Microsoft.MachineLearningServices/workspaces/datasets/registered/profile/read",
            "Microsoft.MachineLearningServices/workspaces/datasets/registered/preview/read",
            "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/profile/read",
            "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/preview/read",
            "Microsoft.MachineLearningServices/workspaces/datasets/registered/schema/read",    
            "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/schema/read",
            "Microsoft.MachineLearningServices/workspaces/datastores/write",
            "Microsoft.MachineLearningServices/workspaces/datastores/delete"
        ],
        "AssignableScopes": [
            "/subscriptions/<subscription_id>"
        ]
    }
    ```

* __MLOps Custom__ (Personalizado para MLOps): permite asignar un rol a una entidad de servicio y usarlo para automatizar las canalizaciones de MLOps. Por ejemplo, para enviar ejecuciones en una canalización ya publicada:

    `mlops_custom_role.json` :
    ```json
    {
        "Name": "MLOps Custom",
        "IsCustom": true,
        "Description": "Can run pipelines against a published pipeline endpoint",
        "Actions": [
            "Microsoft.MachineLearningServices/workspaces/read",
            "Microsoft.MachineLearningServices/workspaces/endpoints/pipelines/read",
            "Microsoft.MachineLearningServices/workspaces/metadata/artifacts/read",
            "Microsoft.MachineLearningServices/workspaces/metadata/snapshots/read",
            "Microsoft.MachineLearningServices/workspaces/environments/read",    
            "Microsoft.MachineLearningServices/workspaces/metadata/secrets/read",
            "Microsoft.MachineLearningServices/workspaces/modules/read",
            "Microsoft.MachineLearningServices/workspaces/experiments/runs/read",
            "Microsoft.MachineLearningServices/workspaces/datasets/registered/read",
            "Microsoft.MachineLearningServices/workspaces/datastores/read",
            "Microsoft.MachineLearningServices/workspaces/environments/write",
            "Microsoft.MachineLearningServices/workspaces/experiments/runs/write",
            "Microsoft.MachineLearningServices/workspaces/metadata/artifacts/write",
            "Microsoft.MachineLearningServices/workspaces/metadata/snapshots/write",
            "Microsoft.MachineLearningServices/workspaces/environments/build/action",
            "Microsoft.MachineLearningServices/workspaces/experiments/runs/submit/action"
        ],
        "NotActions": [
            "Microsoft.MachineLearningServices/workspaces/computes/write",
            "Microsoft.MachineLearningServices/workspaces/write",
            "Microsoft.MachineLearningServices/workspaces/computes/delete",
            "Microsoft.MachineLearningServices/workspaces/delete",
            "Microsoft.MachineLearningServices/workspaces/computes/listKeys/action",
            "Microsoft.MachineLearningServices/workspaces/listKeys/action",
            "Microsoft.Authorization/*"
        ],
        "AssignableScopes": [
            "/subscriptions/<subscription_id>"
        ]
    }
    ```

* __Workspace Admin__ (Administrador del área de trabajo): permite realizar todas las operaciones dentro del ámbito de un área de trabajo, **excepto**:

    * Crear una nueva área de trabajo
    * Asignación de cuotas de nivel de suscripción o de área de trabajo
    * Actualización de la edición del área de trabajo

    El administrador del área de trabajo tampoco puede crear un nuevo rol. Solo puede asignar roles integrados o personalizados existentes dentro del ámbito de su área de trabajo:

    `workspace_admin_custom_role.json` :
    ```json
    {
        "Name": "Workspace Admin Custom",
        "IsCustom": true,
        "Description": "Can perform all operations except quota management and upgrades",
        "Actions": [
            "Microsoft.MachineLearningServices/workspaces/*/read",
            "Microsoft.MachineLearningServices/workspaces/*/action",
            "Microsoft.MachineLearningServices/workspaces/*/write",
            "Microsoft.MachineLearningServices/workspaces/*/delete",
            "Microsoft.Authorization/roleAssignments/*"
        ],
        "NotActions": [
            "Microsoft.MachineLearningServices/workspaces/write"
        ],
        "AssignableScopes": [
            "/subscriptions/<subscription_id>"
        ]
    }
    ```

<a name="labeler"></a>
* __Labeler Custom__ (Personalizado para etiquetador): Permite definir un rol con ámbito únicamente para etiquetar los datos:

    `labeler_custom_role.json` :
    ```json
    {
        "Name": "Labeler Custom",
        "IsCustom": true,
        "Description": "Can label data for Labeling",
        "Actions": [
            "Microsoft.MachineLearningServices/workspaces/read",
            "Microsoft.MachineLearningServices/workspaces/labeling/projects/read",
            "Microsoft.MachineLearningServices/workspaces/labeling/labels/write"
        ],
        "NotActions": [
            "Microsoft.MachineLearningServices/workspaces/labeling/projects/summary/read"
        ],
        "AssignableScopes": [
            "/subscriptions/<subscription_id>"
        ]
    }
    ```

### <a name="q-how-do-i-list-all-the-custom-roles-in-my-subscription"></a>Q. ¿Cómo obtengo una lista de todos los roles personalizados en mi suscripción?

Ejecute el siguiente comando en la CLI de Azure.

```azurecli-interactive
az role definition list --subscription <sub-id> --custom-role-only true
```

### <a name="q-how-do-i-find-the-operations-supported-by-the-machine-learning-service"></a>Q. Cómo encontrar las operaciones admitidas por Machine Learning Service?

Ejecute el siguiente comando en la CLI de Azure.

```azurecli-interactive
az provider operation show –n Microsoft.MachineLearningServices
```

También pueden encontrarse en la lista de [operaciones del proveedor de recursos](/azure/role-based-access-control/resource-provider-operations#microsoftmachinelearningservices).


### <a name="q-what-are-some-common-gotchas-when-using-azure-rbac"></a>Q. ¿Cuáles son algunos problemas comunes al usar RBAC de Azure?

Estos son algunos de los aspectos que debe tener en cuenta al usar el control de acceso basado en rol de Azure (RBAC de Azure):

- Al crear un recurso en Azure, por ejemplo, un área de trabajo, no es directamente el propietario del área de trabajo. El rol se hereda del rol de ámbito más alto con el que está autorizado en esa suscripción. Por ejemplo, si es un administrador de red y tiene permisos para crear un área de trabajo de Machine Learning, se le asignará el rol de administrador de red en esa área de trabajo y no el rol de propietario.
- Cuando hay dos asignaciones de roles al mismo usuario de Azure Active Directory con secciones en conflicto de Actions/NotActions, las operaciones enumeradas en NotActions de un rol podrían no surtir efecto si también aparecen como acciones en otro rol. Para más información sobre cómo Azure analiza las asignaciones de roles, consulte [Cómo determina Azure RBAC si un usuario tiene acceso a un recurso](/azure/role-based-access-control/overview#how-azure-rbac-determines-if-a-user-has-access-to-a-resource).
- Para implementar los recursos de proceso dentro de una red virtual, debe tener explícitamente permiso para las siguientes acciones:
    - "Microsoft.Network/virtualNetworks/join/action" en el recurso de red virtual.
    - "Microsoft.Network/virtualNetworks/subnet/join/action" en el recurso de subred.
    
    Para más información sobre RBAC con redes, consulte los [roles integrados de redes](/azure/role-based-access-control/built-in-roles#networking).

- A veces puede tardar hasta una hora para que las nuevas asignaciones de roles surtan efecto sobre los permisos almacenados en caché en la pila.

### <a name="q-what-permissions-do-i-need-to-use-a-user-assigned-managed-identity-with-my-amlcompute-clusters"></a>Q. ¿Qué permisos necesito para usar una identidad administrada asignada por el usuario con mis clústeres de Amlcompute?

Para asignar una identidad asignada por el usuario en los clústeres de Amlcompute, debe tener permisos de escritura para crear el proceso y el [rol de operador de identidad administrada](/azure/role-based-access-control/built-in-roles#managed-identity-operator). Para más información sobre RBAC con identidades administradas, consulte [Creación, enumeración, eliminación o asignación de un rol a una identidad administrada asignada por el usuario mediante Azure Portal](/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal).


### <a name="q-do-we-support-role-based-access-control-on-the-studio-portal"></a>Q. ¿Se admite el control de acceso basado en rol en el portal de Studio?

Azure Machine Learning Studio admite el control de acceso basado en rol de Azure (RBAC de Azure). 

> [!IMPORTANT]
> Cuando haya asignado un rol personalizado con permisos específicos a un científico de datos en el área de trabajo, las acciones correspondientes (por ejemplo, agregar un botón de proceso) se ocultan automáticamente para los usuarios. Al ocultar estos elementos, se evita cualquier confusión al ver los controles que devuelven una notificación de acceso no autorizado del servicio cuando se utiliza.

### <a name="q-how-do-i-find-the-role-definition-for-a-role-in-my-subscription"></a>Q. ¿Cómo encuentro la definición de roles de un rol en mi suscripción?

Ejecute el siguiente comando en la CLI de Azure. Tenga en cuenta que `<role-name>` debe tener el mismo formato devuelto por el comando anterior.

```azurecli-interactive
az role definition list -n <role-name> --subscription <sub-id>
```

### <a name="q-how-do-i-update-a-role-definition"></a>Q. ¿Cómo actualizo una definición de roles?

Ejecute el siguiente comando en la CLI de Azure.

```azurecli-interactive
az role definition update --role-definition update_def.json --subscription <sub-id>
```

Debe tener permisos en todo el ámbito de la nueva definición de roles. Por ejemplo, si este nuevo rol tiene un ámbito entre tres suscripciones, debe tener permisos en las tres suscripciones. 

> [!NOTE]
> Las actualizaciones de roles pueden tardar entre 15 minutos y una hora en aplicarse a todas las asignaciones de roles de ese ámbito.
### <a name="q-can-i-define-a-role-that-prevents-updating-the-workspace-edition"></a>Q. ¿Puedo definir un rol que impida la actualización de la edición del área de trabajo? 

Sí, puede definir un rol que impida la actualización de la edición del área de trabajo. Dado que la actualización del área de trabajo es una llamada PATCH en el objeto del área de trabajo, puede hacerlo colocando la siguiente acción en la matriz `"NotActions"` en la definición de JSON: 

`"Microsoft.MachineLearningServices/workspaces/write"`

### <a name="q-what-permissions-are-needed-to-perform-quota-operations-in-a-workspace"></a>Q. ¿Qué permisos son necesarios para realizar operaciones de cuota en un área de trabajo? 

Necesita permisos de nivel de suscripción para realizar cualquier operación relacionada con la cuota en el área de trabajo. Esto significa que la configuración de la cuota de nivel de suscripción o de nivel de área de trabajo para los recursos de proceso administrados solo puede producirse si tiene permisos de escritura en el ámbito de la suscripción. 


## <a name="next-steps"></a>Pasos siguientes

- [Introducción a la seguridad en la empresa](concept-enterprise-security.md)
- [Ejecución de experimentos y realización de inferencias/puntuaciones de forma segura en una red virtual](how-to-enable-virtual-network.md)
- [Tutorial: Entrenamiento de modelos](tutorial-train-models-with-aml.md)
- [Operaciones del proveedor de recursos](/azure/role-based-access-control/resource-provider-operations#microsoftmachinelearningservices)
