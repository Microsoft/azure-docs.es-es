---
title: Transferencia de una suscripción de Azure a otro directorio de Azure AD (versión preliminar)
description: Aprenda a transferir una suscripción de Azure y recursos relacionados conocidos a otro directorio de Azure Active Directory (Azure AD).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.workload: identity
ms.date: 08/31/2020
ms.author: rolyon
ms.openlocfilehash: ab004c11b46428c5fad28177b0d94edc04b95654
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400551"
---
# <a name="transfer-an-azure-subscription-to-a-different-azure-ad-directory-preview"></a>Transferencia de una suscripción de Azure a otro directorio de Azure AD (versión preliminar)

> [!IMPORTANT]
> Actualmente, el proceso de seguir estos pasos para transferir una suscripción a otro directorio de Azure AD se encuentra en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Las organizaciones pueden tener varias suscripciones de Azure. Cada suscripción está asociada con un directorio concreto de Azure Active Directory (Azure AD). Para facilitar la administración, puede transferir una suscripción a otro directorio de Azure AD. Al transferir una suscripción a otro directorio de Azure AD, algunos recursos no se transfieren al directorio de destino. Por ejemplo, todas las asignaciones de roles y los roles personalizados en el control de acceso basado en roles de Azure (RBAC de Azure) se eliminan **permanentemente** del directorio de origen y no se transfieren al de destino.

En este artículo se describen los pasos básicos que puede seguir para transferir una suscripción a otro directorio de Azure AD diferente y volver a crear algunos recursos después de la transferencia.

> [!NOTE]
> En el caso de las suscripciones de proveedores de servicios en la nube (CSP) de Azure, no se admite el cambio del directorio de Azure AD de la suscripción.

## <a name="overview"></a>Información general

La transferencia de una suscripción de Azure a otro directorio de Azure AD es un proceso complejo que debe planearse y ejecutarse atentamente. Muchos servicios de Azure requieren entidades de seguridad (identidades) para funcionar con normalidad o incluso administrar otros recursos de Azure. En este artículo se intenta abarcar la mayoría de los servicios de Azure que dependen en gran medida de las entidades de seguridad, pero no es completo.

> [!IMPORTANT]
> En algunos escenarios, la transferencia de una suscripción puede requerir tiempo de inactividad para completar el proceso. Se requiere una planeación cuidadosa para evaluar si se requerirá tiempo de inactividad para la transferencia.

En el diagrama siguiente se muestran los pasos básicos que debe seguir al transferir una suscripción a otro directorio.

1. Preparación de la transferencia

1. Transferencia de una suscripción de Azure a un directorio diferente

1. Nueva creación de recursos en el directorio de destino, como asignaciones de roles, roles personalizados e identidades administradas

    ![Diagrama de transferencia de la suscripción](./media/transfer-subscription/transfer-subscription.png)

### <a name="deciding-whether-to-transfer-a-subscription-to-a-different-directory"></a>Decisión sobre la transferencia de una suscripción a otro directorio

A continuación se indican algunos de los motivos por los que es posible que quiera transferir una suscripción:

- Debido a una fusión o adquisición de la empresa, quiere administrar una suscripción adquirida en el directorio principal de Azure AD.
- Alguien de su organización creó una suscripción y quiere consolidar la administración en un directorio concreto de Azure AD.
- Tiene aplicaciones que dependen de un identificador de suscripción o una dirección URL determinados y no es fácil modificar el código o la configuración de la aplicación.
- Una parte de su negocio se ha dividido en una empresa independiente y debe trasladar algunos de sus recursos a otro directorio de Azure AD.
- Quiere administrar algunos de los recursos en otro directorio de Azure AD con fines de aislamiento de seguridad.

La transferencia de una suscripción requiere un tiempo de inactividad para completar el proceso. En función del escenario, podría ser mejor volver a crear los recursos y copiar los datos en el directorio y la suscripción de destino.

### <a name="understand-the-impact-of-transferring-a-subscription"></a>Comprensión del impacto de transferir una suscripción

Varios recursos de Azure tienen una dependencia de una suscripción o un directorio. En función de su situación, en la tabla siguiente se muestra el impacto conocido de la transferencia de una suscripción. Al seguir los pasos de este artículo, puede volver a crear algunos de los recursos que existían antes de la transferencia de la suscripción.

> [!IMPORTANT]
> En esta sección se enumeran los servicios o recursos de Azure conocidos que dependen de la suscripción. Dado que los tipos de recursos de Azure están evolucionando constantemente, puede haber dependencias adicionales que no aparezcan aquí y que puedan ocasionar un cambio importante en el entorno. 

| Servicio o recurso | Afectado | Recuperable | ¿Se ve afectado? | Qué puede hacer |
| --------- | --------- | --------- | --------- | --------- |
| Asignaciones de roles | Sí | Sí | [Lista de asignaciones de roles](#save-all-role-assignments) | Todas las asignaciones de roles se eliminan permanentemente. Debe asignar usuarios, grupos y entidades de seguridad de servicio a los objetos correspondientes del directorio de destino. Debe volver a crear asignaciones de roles. |
| Roles personalizados | Sí | Sí | [Lista de roles personalizados](#save-custom-roles) | Todos los roles personalizados se eliminan de forma permanente. Debe volver a crear los roles personalizados y las asignaciones de roles. |
| Identidades administradas asignadas por el sistema | Sí | Sí | [Lista de identidades administradas](#list-role-assignments-for-managed-identities) | Debe deshabilitar y volver a habilitar las identidades administradas. Debe volver a crear asignaciones de roles. |
| Identidades administradas asignadas por el usuario | Sí | Sí | [Lista de identidades administradas](#list-role-assignments-for-managed-identities) | Debe eliminar, volver a crear y adjuntar las identidades administradas al recurso adecuado. Debe volver a crear asignaciones de roles. |
| Azure Key Vault | Sí | Sí | [Lista de directivas de acceso de Key Vault](#list-key-vaults) | Debe actualizar el identificador de inquilino asociado con los almacenes de claves. Debe quitar y agregar nuevas directivas de acceso. |
| Instancias de Azure SQL Database con integración de la autenticación de Azure AD habilitada | Sí | No | [Consulta de instancias de Azure SQL Database con autenticación de Azure AD](#list-azure-sql-databases-with-azure-ad-authentication) |  |  |
| Azure Storage y Azure Data Lake Storage Gen2 | Sí | Sí |  | Debe volver a crear las ACL. |
| Azure Data Lake Storage Gen1 | Sí | Sí |  | Debe volver a crear las ACL. |
| Azure Files | Sí | Sí |  | Debe volver a crear las ACL. |
| Azure File Sync | Sí | Sí |  |  |
| Azure Managed Disks | Sí | N/D |  |  |
| Azure Container Service para Kubernetes | Sí | Sí |  |  |
| Azure Active Directory Domain Services | Sí | No |  |  |
| Registros de aplicaciones | Sí | Sí |  |  |

> [!WARNING]
> Si usa el cifrado en reposo para un recurso, como una cuenta de almacenamiento o una base de datos SQL, que tiene una dependencia de un almacén de claves que **no** está en la misma suscripción que se transfiere, puede provocar un escenario irrecuperable. Si tiene esta situación, debe seguir los pasos necesarios para usar un almacén de claves diferente o deshabilitar temporalmente las claves administradas por el cliente para evitar este escenario irrecuperable.

## <a name="prerequisites"></a>Requisitos previos

Para completar estos pasos, necesitará lo siguiente:

- [Bash en Azure Cloud Shell](/azure/cloud-shell/overview) o [CLI de Azure](https://docs.microsoft.com/cli/azure)
- Administrador de cuenta de la suscripción que quiere transferir en el directorio de origen
- Rol de [propietario](built-in-roles.md#owner) en el directorio de destino

## <a name="step-1-prepare-for-the-transfer"></a>Paso 1: Preparación de la transferencia

### <a name="sign-in-to-source-directory"></a>Inicio de sesión en el directorio de origen

1. Inicie sesión en Azure como administrador.

1. Obtenga la lista de suscripciones con el comando [az account list](/cli/azure/account#az-account-list).

    ```azurecli
    az account list --output table
    ```

1. Use [az account set](https://docs.microsoft.com/cli/azure/account#az-account-set) para definir la suscripción activa que quiere transferir.

    ```azurecli
    az account set --subscription "Marketing"
    ```

### <a name="install-the-resource-graph-extension"></a>Instalación de la extensión resource-graph

 La extensión resource-graph le permite usar el comando [az graph](https://docs.microsoft.com/cli/azure/ext/resource-graph/graph) para consultar los recursos que administra Azure Resource Manager. Usará este comando en pasos posteriores.

1. Use [az extension list](https://docs.microsoft.com/cli/azure/extension#az-extension-list) para ver si tiene instalada la extensión *resource-graph*.

    ```azurecli
    az extension list
    ```

1. En caso contrario, instálela.

    ```azurecli
    az extension add --name resource-graph
    ```

### <a name="save-all-role-assignments"></a>Guardar todas las asignaciones de roles

1. Use [az role assignment list](https://docs.microsoft.com/cli/azure/role/assignment#az-role-assignment-list) para mostrar todas las asignaciones de roles (incluidas las heredadas).

    Para facilitar la revisión de la lista, puede exportar la salida como JSON, TSV o tabla. Para obtener más información, consulte [Lista de asignaciones de roles con RBAC y la CLI de Azure](role-assignments-list-cli.md).

    ```azurecli
    az role assignment list --all --include-inherited --output json > roleassignments.json
    az role assignment list --all --include-inherited --output tsv > roleassignments.tsv
    az role assignment list --all --include-inherited --output table > roleassignments.txt
    ```

1. Guarde la lista de asignaciones de roles.

    Cuando se transfiere una suscripción, todas las asignaciones de roles se eliminan **permanentemente**, de modo que es importante guardar una copia.

1. Revise la lista de asignaciones de roles. Es posible que haya asignaciones de roles que no necesite en el directorio de destino.

### <a name="save-custom-roles"></a>Guardar roles personalizados

1. Use [az role definition list](https://docs.microsoft.com/cli/azure/role/definition#az-role-definition-list) para enumerar los roles personalizados. Para más información, consulte [Creación o actualización de roles personalizados de Azure mediante la CLI de Azure](custom-roles-cli.md).

    ```azurecli
    az role definition list --custom-role-only true --output json --query '[].{roleName:roleName, roleType:roleType}'
    ```

1. Guarde cada rol personalizado que necesitará en el directorio de destino como archivo JSON independiente.

    ```azurecli
    az role definition list --name <custom_role_name> > customrolename.json
    ```

1. Realice copias de los archivos de rol personalizados.

1. Modifique cada copia para usar el formato siguiente.

    Usará estos archivos posteriormente para volver a crear los roles personalizados en el directorio de destino.

    ```json
    {
      "Name": "",
      "Description": "",
      "Actions": [],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": []
    }
    ```

### <a name="determine-user-group-and-service-principal-mappings"></a>Determinación de las asignaciones de usuario, grupo y entidad de servicio

1. En función de la lista de asignaciones de roles, determine los usuarios, grupos y entidades de servicio con los que creará una asignación en el directorio de destino.

    Para identificar el tipo de entidad de seguridad, examine la propiedad `principalType` de cada asignación de roles.

1. Si es necesario, en el directorio de destino, cree los usuarios, grupos o entidades de servicio que necesitará.

### <a name="list-role-assignments-for-managed-identities"></a>Lista de asignaciones de roles para identidades administradas

Las identidades administradas no se actualizan cuando una suscripción se transfiere a otro directorio. Como resultado, se interrumpe cualquier identidad administrada asignada por el sistema o por el usuario. Después de la transferencia, puede volver a habilitar las identidades administradas asignadas por el sistema. En el caso de las identidades administradas asignadas por el usuario, tendrá que volver a crearlas y adjuntarlas en el directorio de destino.

1. Revise la [lista de servicios de Azure que admiten identidades administradas](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md) para anotar dónde podría estar usando identidades administradas.

1. Use [az ad sp list](/cli/azure/identity?view=azure-cli-latest#az-identity-list) para mostrar las identidades administradas asignadas por el sistema y por el usuario.

    ```azurecli
    az ad sp list --all --filter "servicePrincipalType eq 'ManagedIdentity'"
    ```

1. En la lista de identidades administradas, determine cuáles son las asignadas por el sistema y por el usuario. Puede usar los criterios siguientes para determinar el tipo.

    | Criterios | Tipo de identidad administrada |
    | --- | --- |
    | La propiedad `alternativeNames` incluye `isExplicit=False`. | Asignada por el sistema |
    | La propiedad `alternativeNames` no incluye `isExplicit`. | Asignada por el sistema |
    | La propiedad `alternativeNames` incluye `isExplicit=True`. | Asignada por el usuario |

    También puede usar [az identity list](https://docs.microsoft.com/cli/azure/identity#az-identity-list) para enumerar las identidades administradas asignadas por el usuario. Para obtener más información, consulte [Creación, enumeración o eliminación de una identidad administrada asignada por el usuario mediante la CLI de Azure](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md).

    ```azurecli
    az identity list
    ```

1. Obtenga una lista de los valores de `objectId` para las identidades administradas.

1. Compruebe si en la lista de asignaciones de roles hay alguna asignación de roles para las identidades administradas.

### <a name="list-key-vaults"></a>Enumeración de almacenes de claves

Cuando se crea un almacén de claves, se asocia automáticamente a un identificador de inquilino de Azure Active Directory para la suscripción en la que ha sido creado. Las entradas de la directiva de acceso también se asocian con este identificador de inquilino. Para obtener más información, consulte [Traslado de Azure Key Vault a otra suscripción](../key-vault/general/move-subscription.md).

> [!WARNING]
> Si usa el cifrado en reposo para un recurso, como una cuenta de almacenamiento o una base de datos SQL, que tiene una dependencia de un almacén de claves que **no** está en la misma suscripción que se transfiere, puede provocar un escenario irrecuperable. Si tiene esta situación, debe seguir los pasos necesarios para usar un almacén de claves diferente o deshabilitar temporalmente las claves administradas por el cliente para evitar este escenario irrecuperable.

- Si tiene un almacén de claves, use [az keyvault show](https://docs.microsoft.com/cli/azure/keyvault#az-keyvault-show) para mostrar las directivas de acceso. Para más información, consulte [Asignación de una directiva de acceso de Key Vault](../key-vault/general/assign-access-policy-cli.md).

    ```azurecli
    az keyvault show --name MyKeyVault
    ```

### <a name="list-azure-sql-databases-with-azure-ad-authentication"></a>Lista de instancias de Azure SQL Database con autenticación de Azure AD

- Use [az sql server ad-admin list](https://docs.microsoft.com/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-list) y la extensión [az graph](https://docs.microsoft.com/cli/azure/ext/resource-graph/graph) para ver si está usando instancias de Azure SQL Database con integración de la autenticación de Azure AD habilitada. Para obtener más información, consulte [Configuración y administración de la autenticación de Azure Active Directory con SQL](../azure-sql/database/authentication-aad-configure.md).

    ```azurecli
    az sql server ad-admin list --ids $(az graph query -q 'resources | where type == "microsoft.sql/servers" | project id' -o tsv | cut -f1)
    ```

### <a name="list-acls"></a>Lista de ACL

1. Si usa Azure Data Lake Storage Gen1, enumere las ACL que se aplican a cualquier archivo mediante Azure Portal o PowerShell.

1. Si usa Azure Data Lake Storage Gen2, enumere las ACL que se aplican a cualquier archivo mediante Azure Portal o PowerShell.

1. Si usa Azure Files, enumere las ACL que se aplican a cualquier archivo.

### <a name="list-other-known-resources"></a>Lista de otros recursos conocidos

1. Use [az account show](https://docs.microsoft.com/cli/azure/account#az-account-show) para obtener el identificador de la suscripción.

    ```azurecli
    subscriptionId=$(az account show --query id | sed -e 's/^"//' -e 's/"$//')
    ```

1. Use la extensión [az graph](https://docs.microsoft.com/cli/azure/ext/resource-graph/graph) para mostrar otros recursos de Azure con dependencias de directorio de Azure AD conocidas.

    ```azurecli
    az graph query -q \
    'resources | where type != "microsoft.azureactivedirectory/b2cdirectories" | where  identity <> "" or properties.tenantId <> "" or properties.encryptionSettingsCollection.enabled == true | project name, type, kind, identity, tenantId, properties.tenantId' \
    --subscriptions $subscriptionId --output table
    ```

## <a name="step-2-transfer-the-subscription"></a>Paso 2: Transferencia de la suscripción

En este paso, transferirá la suscripción del directorio de origen al de destino. Los pasos serán diferentes en función de si desea transferir también la propiedad de la facturación.

> [!WARNING]
> Al transferir la suscripción, todas las asignaciones de roles del directorio de origen se eliminan **permanentemente** y no se pueden restaurar. No puede volver atrás una vez transferida la suscripción. Asegúrese de completar los pasos anteriores antes de realizar este paso.

1. Determine si desea transferir también la propiedad de facturación a otra cuenta.

1. Transfiera la suscripción a un directorio diferente.

    - Si quiere mantener la propiedad de facturación actual, siga los pasos de [Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).
    - Si también quiere transferir la propiedad de facturación, siga los pasos de [Transferencia de la propiedad de facturación de una suscripción de Azure a otra cuenta](../cost-management-billing/manage/billing-subscription-transfer.md). Para transferir la suscripción a otro directorio, debe activar la casilla **Inquilino de la suscripción de Azure AD**.

1. Una vez que haya finalizado la transferencia de la suscripción, regrese a este artículo para volver a crear los recursos en el directorio de destino.

## <a name="step-3-re-create-resources"></a>Paso 3: Volver a crear los recursos

### <a name="sign-in-to-target-directory"></a>Inicio de sesión en el directorio de destino

1. En el directorio de destino, inicie sesión como el usuario que aceptó la solicitud de transferencia.

    Solo el usuario de la cuenta nueva que aceptó la solicitud de transferencia tendrá acceso para administrar los recursos.

1. Obtenga la lista de suscripciones con el comando [az account list](https://docs.microsoft.com/cli/azure/account#az-account-list).

    ```azurecli
    az account list --output table
    ```

1. Use [az account set](https://docs.microsoft.com/cli/azure/account#az-account-set) para definir la suscripción activa que quiere usar.

    ```azurecli
    az account set --subscription "Contoso"
    ```

### <a name="create-custom-roles"></a>Creación de roles personalizados
        
- Use [az role definition create](https://docs.microsoft.com/cli/azure/role/definition#az-role-definition-create) para crear cada rol personalizado a partir de los archivos que creó anteriormente. Para más información, consulte [Creación o actualización de roles personalizados de Azure mediante la CLI de Azure](custom-roles-cli.md).

    ```azurecli
    az role definition create --role-definition <role_definition>
    ```

### <a name="create-role-assignments"></a>Crear asignaciones de roles

- Use [az role assignment create](https://docs.microsoft.com/cli/azure/role/assignment#az-role-assignment-create) para crear las asignaciones de roles para usuarios, grupos y entidades de servicio. Para obtener más información, vea [Incorporación o eliminación de asignaciones de roles con RBAC y la CLI de Azure](role-assignments-cli.md).

    ```azurecli
    az role assignment create --role <role_name_or_id> --assignee <assignee> --resource-group <resource_group>
    ```

### <a name="update-system-assigned-managed-identities"></a>Actualización de identidades administradas asignadas por el sistema

1. Deshabilite y vuelva a habilitar las identidades administradas asignadas por el sistema.

    | Servicio de Azure | Más información | 
    | --- | --- |
    | Máquinas virtuales | [Configuración de identidades administradas para recursos de Azure en una VM de Azure mediante la CLI de Azure](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#system-assigned-managed-identity) |
    | Conjuntos de escalado de máquinas virtuales | [Configuración de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales mediante la CLI de Azure](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vmss.md#system-assigned-managed-identity) |
    | Otros servicios | [Servicios que admiten identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md) |

1. Use [az role assignment create](https://docs.microsoft.com/cli/azure/role/assignment#az-role-assignment-create) para crear asignaciones de roles para las identidades administradas asignadas por el sistema. Para más información, consulte [Asignación de un acceso de identidad administrada a un recurso mediante la CLI de Azure](../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md).

    ```azurecli
    az role assignment create --assignee <objectid> --role '<role_name_or_id>' --scope <scope>
    ```

### <a name="update-user-assigned-managed-identities"></a>Actualización de identidades administradas asignadas por el usuario

1. Elimine, vuelva a crear y adjunte identidades administradas asignadas por el usuario.

    | Servicio de Azure | Más información | 
    | --- | --- |
    | Máquinas virtuales | [Configuración de identidades administradas para recursos de Azure en una VM de Azure mediante la CLI de Azure](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity) |
    | Conjuntos de escalado de máquinas virtuales | [Configuración de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales mediante la CLI de Azure](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vmss.md#user-assigned-managed-identity) |
    | Otros servicios | [Servicios que admiten identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)<br/>[Creación, enumeración o eliminación de una identidad administrada asignada por el usuario mediante la CLI de Azure](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md) |

1. Use [az role assignment create](https://docs.microsoft.com/cli/azure/role/assignment#az-role-assignment-create) para crear asignaciones de roles para las identidades administradas asignadas por el usuario. Para más información, consulte [Asignación de un acceso de identidad administrada a un recurso mediante la CLI de Azure](../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md).

    ```azurecli
    az role assignment create --assignee <objectid> --role '<role_name_or_id>' --scope <scope>
    ```

### <a name="update-key-vaults"></a>Actualización de almacenes de claves

En esta sección se describen los pasos básicos para actualizar los almacenes de claves. Para obtener más información, consulte [Traslado de Azure Key Vault a otra suscripción](../key-vault/general/move-subscription.md).

1. Actualice el identificador de inquilino asociado a todas las instancias de Key Vault existentes en la suscripción al directorio de destino.

1. Quitar todas las entradas de la directiva de acceso existentes.

1. Agregue nuevas entradas de la directiva de acceso asociadas con el directorio de destino.

### <a name="update-acls"></a>Actualización de ACL

1. Si usa Azure Data Lake Storage Gen1, asigne las ACL adecuadas. Para más información, consulte [Protección de los datos almacenados en Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-secure-data.md).

1. Si usa Azure Data Lake Storage Gen2, asigne las ACL adecuadas. Para más información, consulte [Control de acceso en Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-access-control.md).

1. Si usa Azure Files, asigne las ACL adecuadas.

### <a name="review-other-security-methods"></a>Revisión de otros métodos de seguridad

Incluso si las asignaciones de roles se quitan durante la transferencia, es posible que los usuarios de la cuenta del propietario original sigan teniendo acceso a la suscripción mediante otros métodos seguridad, como los siguientes:

- Claves de acceso para servicios como Almacenamiento.
- [Certificados de administración](../cloud-services/cloud-services-certs-create.md) que conceden al administrador de usuarios acceso a los recursos de la suscripción.
- Credenciales de acceso remoto para servicios como Azure Virtual Machines.

Si su intención es quitar el acceso de los usuarios del directorio de origen para que no tengan acceso en el directorio de destino, considere la posibilidad de cambiar las credenciales. Hasta que se actualicen las credenciales, los usuarios seguirán teniendo acceso después de la transferencia.

1. Cambie las claves de acceso de la cuenta de almacenamiento. Para más información, consulte [Administración de las claves de acceso de la cuenta de almacenamiento](../storage/common/storage-account-keys-manage.md).

1. Si usa claves de acceso para otros servicios, como Azure SQL Database o la mensajería de Azure Service Bus, cambie las claves de acceso.

1. Para los recursos que usan secretos, abra la configuración del recurso y actualice el secreto.

1. En el caso de los recursos que usan certificados, actualice el certificado.

## <a name="next-steps"></a>Pasos siguientes

- [Transferencia de la propiedad de facturación de una suscripción de Azure a otra cuenta](../cost-management-billing/manage/billing-subscription-transfer.md)
- [Transferencia de suscripciones de Azure entre suscriptores y CSP](../cost-management-billing/manage/transfer-subscriptions-subscribers-csp.md)
- [Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)
