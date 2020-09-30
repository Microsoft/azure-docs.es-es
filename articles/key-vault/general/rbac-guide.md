---
title: Concesión de permisos para que las aplicaciones accedan a una instancia de Azure Key Vault mediante Azure RBAC | Microsoft Docs
description: Aprenda a proporcionar acceso a claves, secretos y certificados mediante el control de acceso basado en rol de Azure.
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 8/30/2020
ms.author: mbaldwin
ms.openlocfilehash: ada966cb3c2a08a8a1ed81c3ba18ab9859774b44
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89394450"
---
# <a name="provide-access-to-key-vault-keys-certificates-and-secrets-with-an-azure-role-based-access-control-preview"></a>Acceso a las claves, los certificados y los secretos de Key Vault con un control de acceso basado en rol de Azure (versión preliminar)

El control de acceso basado en rol de Azure es un sistema de autorización basado en [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) que proporciona administración de acceso específico a los recursos de Azure.

Azure RBAC permite a los usuarios administrar los permisos de las claves, secretos y certificados. Proporciona un lugar para administrar todos los permisos en todos los almacenes de claves. 

El modelo de Azure RBAC brinda la posibilidad de establecer permisos en diferentes niveles de ámbito: grupo de administración, suscripción, grupo de recursos o recursos individuales.  Azure RBAC para Key Vault también brinda la posibilidad de tener permisos independientes en las claves, los secretos y los certificados independientes

Para más información, consulte [Control de acceso basado en rol de Azure (Azure RBAC)](https://docs.microsoft.com/azure/role-based-access-control/overview).

## <a name="best-practices-for-individual-keys-secrets-and-certificates"></a>Procedimientos recomendados para claves, secretos y certificados individuales

Nuestra recomendación es usar un almacén por aplicación y entorno (desarrollo, preproducción y producción).

Los permisos de claves, secretos y certificados individuales solo deben usarse para escenarios concretos:

-   Aplicaciones multicapa que necesitan un control de acceso independiente entre las distintas capas

-   Almacén de claves compartido con secretos comunes, cuando las aplicaciones necesitan acceso a los subconjuntos de secretos de ese almacén de claves

Para más información acerca de las directrices de administración de Azure Key Vault, consulte:

- [Procedimientos recomendados de Azure Key Vault](best-practices.md)
- [Límites de servicio Azure Key Vault](service-limits.md)

## <a name="azure-built-in-roles-for-key-vault-data-plane-operations-preview"></a>Roles integrados de Azure para operaciones del plano de datos de Key Vault (versión preliminar)

| Rol integrado | Descripción | ID |
| --- | --- | --- |
| Administrador de almacén de claves (versión preliminar) | Permite realizar todas las operaciones de plano de datos en un almacén de claves y en todos los objetos que contiene, incluidos los certificados, las claves y los secretos. No permite administrar los recursos del almacén de claves ni administrar las asignaciones de roles. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 00482a5a-887f-4fb3-b363-3b7fe8e74483 |
| Responsable de certificados de almacén de claves (versión preliminar) | Permite realizar cualquier acción en los certificados de un almacén de claves, excepto administrar permisos. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | a4417e6f-fecd-4de8-b567-7b0420556985 |
| Responsable criptográfico de almacén de claves (versión preliminar)| Permite realizar cualquier acción en las claves de un almacén de claves, excepto administrar permisos. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 14b46e9e-c2b7-41b4-b07b-48a6ebf60603 |
| Cifrado de servicio criptográfico de almacén de claves (versión preliminar) | Permite leer los metadatos de las claves y realizar operaciones de encapsulado/desencapsulado. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | e147488a-f6f5-4113-8e2d-b22465e65bf6 |
| Usuario criptográfico de almacén de claves (versión preliminar) | Permite realizar operaciones criptográficas mediante claves. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 12338af0-0e69-4776-bea7-57ae8d297424 |
| Lector de almacén de claves (versión preliminar)| Permite leer metadatos de almacenes de claves y sus certificados, claves y secretos. No se pueden leer valores confidenciales, como el contenido de los secretos o el material de las claves. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 21090545-7ca7-4776-b22c-e363652d74d2 |
| Responsable de secretos de almacén de claves (versión preliminar)| Permite realizar cualquier acción en los secretos de un almacén de claves, excepto administrar permisos. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | b86a8fe4-44ce-4948-aee5-eccb2c155cd7 |
| Usuario de secretos de almacén de claves (versión preliminar)| Permite leer el contenido de los secretos. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 4633458b-17de-408a-b874-0445c86b69e6 |

Para más información sobre las definiciones de roles de Azure, consulte [Roles integrados en Azure](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles).

## <a name="using-azure-rbac-secret-key-and-certificate-permissions-with-key-vault"></a>Uso de los permisos de secretos, claves y certificados de Azure RBAC con Key Vault

El nuevo modelo de permisos de Azure RBAC para el almacén de claves proporciona una alternativa al modelo de permisos de la directiva de acceso del almacén. 

### <a name="enable-azure-rbac-permissions-on-key-vault"></a>Habilitación de los permisos de Azure RBAC en Key Vault

> [!IMPORTANT]
> El establecimiento del modelo de permisos de Azure RBAC invalida todos los permisos de las directivas de acceso. Puede provocar interrupciones cuando los roles de Azure equivalentes no se asignen.

1.  Habilitación de los permisos de Azure RBAC en un almacén de claves nuevo:

    ![Habilitación de los permisos de RBAC: almacén nuevo](../media/rbac/image-1.png)

2.  Habilitación de los permisos de Azure RBAC en un almacén de claves existente:

    ![Habilitación de los permisos de RBAC: almacén existente](../media/rbac/image-2.png)

### <a name="assign-role"></a>Asignación de un rol

> [!Note]
> Se recomienda usar el identificador de rol único, en lugar del nombre de rol en los scripts. Por consiguiente, si se cambia el nombre de un rol, los scripts seguirían funcionando. En la versión preliminar, todos los roles tendrían el sufijo "(preview)", pero se eliminaría posteriormente. En este documento, el nombre de rol se usa solo para facilitar la legibilidad.

Comando de la CLI de Azure para crear una asignación de roles:

```azurecli
az role assignment create --role <role_name_or_id> --assignee <assignee> --scope <scope>
```

En Azure Portal, la pantalla de asignaciones de roles de Azure está disponible para todos los recursos en la pestaña Control de acceso (IAM).

![Asignación de roles (IAM)](../media/rbac/image-3.png)

### <a name="resource-group-scope-role-assignment"></a>Asignación de roles en el ámbito de un grupo de recursos

1.  Vaya al grupo de recursos del almacén de claves.
    ![Asignación de roles: grupo de recursos](../media/rbac/image-4.png)

2.  Haga clic en Control de acceso (IAM) \> Agregar asignación de roles\>Agregar

3.  Creación de un rol Lector de almacén de claves "Lector de almacén de claves (versión preliminar)" para el usuario actual

    ![Adición de rol: grupo de recursos](../media/rbac/image-5.png)

CLI de Azure:
```azurecli
az role assignment create --role "Key Vault Reader (preview)" --assignee {i.e user@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}
```

La asignación de roles anterior brinda la posibilidad de enumerar los objetos de almacén de claves que hay en el almacén de claves.

### <a name="key-vault-scope-role-assignment"></a>Asignación de roles del ámbito de Key Vault

1. Vaya a la pestaña Key Vault \> Control de acceso (IAM).

2. Haga clic en Agregar asignación de roles\>Agregar.

3. Cree el rol Responsable de secretos de claves "Responsable de secretos del almacén de claves (versión preliminar)" para el usuario actual.

    ![Asignación de roles: almacén de claves](../media/rbac/image-6.png)

 CLI de Azure:

```azurecli
az role assignment create --role "Key Vault Secrets Officer (preview)" --assignee {i.e jalichwa@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}
```

Después de crear la asignación de roles anterior puede crear, actualizar o eliminar los secretos.

4. Cree un secreto (Secrets \> +Generate/Import) [(Secretos > +Generar o importar)] para probar la asignación de roles de nivel de secreto.

    ![Adición de rol: almacén de claves](../media/rbac/image-7.png)

### <a name="secret-scope-role-assignment"></a>Asignación de roles del ámbito de secreto

1. Abra uno de los secretos creados previamente, observe Información general y control de acceso (IAM) (versión preliminar)

2. Haga clic en la pestaña Control de acceso (IAM) (versión preliminar)

    ![Asignación de roles: secretos](../media/rbac/image-8.png)

3. Cree el rol Responsable de secretos de claves "Responsable de secretos del almacén de claves (versión preliminar)" para el usuario actual de la misma forma que lo hizo antes para Key Vault.

CLI de Azure:

```azurecli
az role assignment create --role "Key Vault Secrets Officer (preview)" --assignee {i.e user@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}/secrets/RBACSecret
```

### <a name="test-and-verify"></a>Prueba y comprobación

> [!NOTE]
> Los exploradores usan el almacenamiento en caché y la actualización de página es necesaria después de quitar las asignaciones de roles.<br>
> Deje varios minutos para que se actualice la asignación de roles

1. Valide la adición de un secreto nuevo sin el rol "Responsable de secretos del almacén de claves" en el nivel del almacén de claves.

Vaya a la pestaña Control de acceso (IAM) del almacén de claves y elimine la asignación de roles "Responsable de secretos del almacén de claves (versión preliminar)" para este recurso.

![Eliminación de asignación: almacén de claves](../media/rbac/image-9.png)

Vaya al secreto creado antes. Puede ver todas las propiedades del secreto.

![Vista de secreto con acceso](../media/rbac/image-10.png)

Crear secreto (Secrets \> +Generate/Import) [(Secretos > +Generar o importar)] debe mostrar el siguiente error:

   ![Crear un secreto](../media/rbac/image-11.png)

2.  Valide la edición del secreto sin el rol "Responsable de secretos del almacén de claves" en el nivel del secreto.

-   Vaya a la pestaña Control de acceso (IAM) (versión preliminar) del secreto creado antes y elimine la asignación de roles "Responsable de secretos del almacén de claves (versión preliminar)" para este recurso.

-   Vaya al secreto que creó anteriormente. Puede ver las propiedades del secreto.

   ![Vista de secreto sin acceso](../media/rbac/image-12.png)

3. Valide la lectura de los secretos sin el rol de lectura en el nivel del almacén de claves.

-   Vaya a la pestaña Control de acceso (IAM) del grupo de recursos del almacén de claves y elimine la asignación de roles "Lector del almacén de claves (versión preliminar)".

-   Si va a la pestaña Secretos del almacén de claves, debería aparecer el siguiente error:

   ![Pestaña Secretos: error](../media/rbac/image-13.png)

### <a name="creating-custom-roles"></a>Creación de roles personalizados 

[comando az role definition create](https://docs.microsoft.com/cli/azure/role/definition#az-role-definition-create)

**(Script de Bash de CLI)</br>**
```azurecli
az role definition create --role-definition '{ \
   "Name": "Backup Keys Operator", \
   "Description": "Perform key backup/restore operations", \
    "Actions": [ 
    ], \
    "DataActions": [ \
        "Microsoft.KeyVault/vaults/keys/read ", \
        "Microsoft.KeyVault/vaults/keys/backup/action", \
         "Microsoft.KeyVault/vaults/keys/restore/action" \
    ], \
    "NotDataActions": [ 
   ], \
    "AssignableScopes": ["/subscriptions/{subscriptionId}"] \
}'
```

Para más información sobre cómo crear roles personalizados, consulte:

[Roles personalizados en los recursos de Azure](https://docs.microsoft.com/azure/role-based-access-control/custom-roles)

## <a name="known-limits-and-performance"></a>Límites conocidos y rendimiento

-   Dos mil asignaciones de roles de Azure por suscripción

-   Latencia de las asignación de roles: con el rendimiento actual esperado, pasarán 10 minutos (600 segundos) después de que la asignación de roles se cambia para que se aplique el rol

## <a name="learn-more"></a>Más información

- [Información general de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)
- [Tutorial de roles personalizados](https://docs.microsoft.com/azure/role-based-access-control/tutorial-custom-role-cli)
