---
title: Administrar una instancia de Azure Blockchain Service con la CLI de Azure
description: Administración de Azure Blockchain Service con la CLI de Azure
ms.date: 07/23/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: 36b012c486c0c7d3303a81998e88f1605999c899
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87170858"
---
# <a name="manage-azure-blockchain-service-using-azure-cli"></a>Administrar una instancia de Azure Blockchain Service con la CLI de Azure

Además de Azure Portal, puede usar la CLI de Azure para administrar los miembros de la cadena de bloques y los nodos de transacción para Azure Blockchain Service.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/bash](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

Para instalar y usar la CLI de forma local, consulte el artículo sobre [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="prepare-your-environment"></a>Preparación del entorno

1. Inicie sesión.

    Si está usando una instalación local de la CLI, inicie sesión con el comando [az login](/cli/azure/reference-index#az-login).

    ```azurecli
    az login
    ```

    Siga los pasos que se muestran en el terminal para completar el proceso de autenticación.

1. Instale la extensión de la CLI de Azure.

    Para trabajar con referencias de extensión de la CLI de Azure, primero debe instalar la extensión.  Las extensiones de la CLI de Azure le proporcionan acceso a comandos experimentales y en versión preliminar que todavía no se han enviado como parte de la CLI principal.  Para más información acerca de las extensiones, incluida la actualización y la desinstalación, consulte [Uso de extensiones con la CLI de Azure](/cli/azure/azure-cli-extensions-overview).

    Ejecute el siguiente comando para instalar la [extensión para Azure Blockchain Service](/cli/azure/ext/blockchain/blockchain):

    ```azurecli-interactive
    az extension add --name blockchain
    ```

## <a name="create-blockchain-member"></a>Creación de un miembro de cadena de bloques

En el ejemplo [se crea un miembro de cadena de bloques](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-create) en Azure Blockchain Service que ejecuta el protocolo de libro de contabilidad de Quorum en un nuevo consorcio.

```azurecli
az blockchain member create \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName> \
                            --location <myBlockchainLocation> \
                            --password <strongMemberAccountPassword> \
                            --protocol "Quorum" \
                            --consortium <myConsortiumName> \
                            --consortium-management-account-password <strongConsortiumManagementPassword> \
                            --sku <skuName>
```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos donde se crean los recursos de Azure Blockchain Service. |
| **name** | Un nombre único que identifica al miembro de cadena de bloques de Azure Blockchain Service. El nombre se usa como dirección de punto de conexión público. Por ejemplo, `myblockchainmember.blockchain.azure.com`. |
| **ubicación** | Región de Azure donde se crea el miembro de cadena de bloques. Por ejemplo, `eastus`. Elija la ubicación más cercana a los usuarios o a sus otras aplicaciones de Azure. Es posible que las características no estén disponibles en algunas regiones. |
| **password** | La contraseña del nodo de transacción predeterminado del miembro. Use la contraseña para la autenticación básica al conectarse al punto de conexión público del nodo de transacción predeterminado del miembro de la cadena de bloques. La contraseña debe cumplir tres de los cuatro requisitos siguientes: debe tener una longitud de entre 12 y 72 caracteres, un carácter en minúscula, un carácter en mayúscula, un número y un carácter especial que no sea el signo de número (#), porcentaje (%), coma (,), asterisco (*), comilla inversa (\`), comilla doble ("), comilla simple ('), guion (-) y punto y coma (;).|
| **protocolo** | Protocolo de la cadena de bloques. Actualmente, se admite el protocolo *Quorum*. |
| **consortium** | Nombre del consorcio al que unirse o que se va a crear. Para más información sobre los consorcios, consulte [Consorcio de Azure Blockchain Service](consortium.md). |
| **consortium-management-account-password** | Este valor también se conoce como la contraseña de la cuenta del miembro, y se usa para cifrar la clave privada de la cuenta de Ethereum que se crea para el miembro. Usará la cuenta del miembro y la contraseña de la cuenta del miembro para la administración del consorcio. |
| **sku** | Tipo de nivel. *Estándar* o *Básico*. Use el nivel *Básico* para desarrollo, pruebas y prueba de concepto. Use el nivel *Estándar* para las implementaciones de nivel de producción. También debe usar el nivel *Estándar* si va a utilizar Blockchain Data Manager o va a enviar un gran volumen de transacciones privadas. Después de la creación de un miembro, no se puede cambiar el plan de tarifa de Básico a Estándar. |

## <a name="change-blockchain-member-passwords-or-firewall-rules"></a>Cambio de las contraseñas o reglas de firewall del miembro de la cadena de bloques

En el ejemplo [se actualiza la contraseña de un miembro de la cadena de bloques](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-update), la contraseña de administración del consorcio y la regla de firewall.

```azurecli
az blockchain member update \
                     --resource-group <myResourceGroup> \
                     --name <myMemberName> \
                     --password <strongMemberAccountPassword> \
                     --consortium-management-account-password <strongConsortiumManagementPassword> \
                     --firewall-rules <firewallRules>
```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos donde se crean los recursos de Azure Blockchain Service. |
| **name** | Nombre que identifica al miembro de Azure Blockchain Service. |
| **password** | La contraseña del nodo de transacción predeterminado del miembro. Use la contraseña para la autenticación básica al conectarse al punto de conexión público del nodo de transacción predeterminado del miembro de la cadena de bloques. La contraseña debe cumplir tres de los cuatro requisitos siguientes: debe tener una longitud de entre 12 y 72 caracteres, un carácter en minúscula, un carácter en mayúscula, un número y un carácter especial que no sea el signo de número (#), porcentaje (%), coma (,), asterisco (*), comilla inversa (\`), comilla doble ("), comilla simple ('), guion (-) y punto y coma (;).|
| **consortium-management-account-password** | Este valor también se conoce como la contraseña de la cuenta del miembro, y se usa para cifrar la clave privada de la cuenta de Ethereum que se crea para el miembro. Usará la cuenta del miembro y la contraseña de la cuenta del miembro para la administración del consorcio. |
| **firewall-rules** | Dirección IP inicial y final de la lista de direcciones IP permitidas. |

## <a name="create-transaction-node"></a>Creación del nodo de transacción

[Cree un nodo de transacción](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-create) en un miembro de la cadena de bloques existente. Al agregar nodos de transacción, puede aumentar el aislamiento de la seguridad y distribuir la carga. Por ejemplo, podría tener un punto de conexión del nodo de transacción para diferentes aplicaciones cliente.

```azurecli
az blockchain transaction-node create \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --password <strongTransactionNodePassword> \
                     --name <myTransactionNodeName>
```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos donde se crean los recursos de Azure Blockchain Service. |
| **ubicación** | Región de Azure de un miembro de la cadena de bloques. |
| **member-name** | Nombre que identifica al miembro de Azure Blockchain Service. |
| **password** | Contraseña para el nodo de transacción. Use la contraseña para la autenticación básica al conectarse al punto de conexión público del nodo de transacción. La contraseña debe cumplir tres de los cuatro requisitos siguientes: debe tener una longitud de entre 12 y 72 caracteres, un carácter en minúscula, un carácter en mayúscula, un número y un carácter especial que no sea el signo de número (#), porcentaje (%), coma (,), asterisco (*), comilla inversa (\`), comilla doble ("), comilla simple ('), guion (-) y punto y coma (;).|
| **name** | Nombre del nodo de transacción. |

## <a name="change-transaction-node-password"></a>Cambio de la contraseña del nodo de transacción

En el ejemplo [se cambia la contraseña de un nodo de transacción](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-update).

```azurecli
az blockchain transaction-node update \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --password <strongTransactionNodePassword> \
                     --name <myTransactionNodeName>
```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos en el que se encuentran los recursos de Azure Blockchain Service. |
| **member-name** | Nombre que identifica al miembro de Azure Blockchain Service. |
| **password** | Contraseña para el nodo de transacción. Use la contraseña para la autenticación básica al conectarse al punto de conexión público del nodo de transacción. La contraseña debe cumplir tres de los cuatro requisitos siguientes: debe tener una longitud de entre 12 y 72 caracteres, un carácter en minúscula, un carácter en mayúscula, un número y un carácter especial que no sea el signo de número (#), porcentaje (%), coma (,), asterisco (*), comilla inversa (\`), comilla doble ("), comilla simple ('), guion (-) y punto y coma (;).|
| **name** | Nombre del nodo de transacción. |

## <a name="list-api-keys"></a>Enumeración de las claves de API

Las claves de API se pueden usar para acceder al nodo de forma similar al nombre de usuario y la contraseña. Hay dos claves de API que admiten la rotación de claves. Para [enumerar las claves de API](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-transaction-node-list-api-key), use el comando siguiente.

```azurecli
az blockchain member list-api-key \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName>
```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos en el que se encuentran los recursos de Azure Blockchain Service. |
| **name** | Nombre del miembro de la cadena de bloques de Azure Blockchain Service. |

## <a name="regenerate-api-keys"></a>Regeneración de claves de API

Para [volver a generar claves de API](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-transaction-node-regenerate-api-key), use el comando siguiente.

```azurecli
az blockchain member regenerate-api-key \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName> \
                            [--key-name {<keyValue1>, <keyValue2>}]
```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos en el que se encuentran los recursos de Azure Blockchain Service. |
| **name** | Nombre del miembro de la cadena de bloques de Azure Blockchain Service. |
| **keyName** | Reemplace \<keyValue\> por key1, key2 o ambas. |

## <a name="delete-a-transaction-node"></a>Eliminar un nodo de transacción

En el ejemplo [se elimina un nodo de transacción del miembro de la cadena de bloques](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-delete).

```azurecli
az blockchain transaction-node delete \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --name <myTransactionNode>
```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos en el que se encuentran los recursos de Azure Blockchain Service. |
| **member-name** | Nombre del miembro de la cadena de bloques de Azure Blockchain Service que también incluye el nombre del nodo de transacción que se va a eliminar. |
| **name** | Nombre del nodo de transacción que se va a eliminar. |

## <a name="delete-a-blockchain-member"></a>Eliminación de un miembro de cadena de bloques

En el ejemplo [se elimina un miembro de la cadena de bloques](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-delete).

```azurecli
az blockchain member delete \
                     --resource-group <myResourceGroup> \
                     --name <myMemberName>

```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos en el que se encuentran los recursos de Azure Blockchain Service. |
| **name** | Nombre del miembro de cadena de bloques de Azure Blockchain Service que se va a eliminar. |

## <a name="azure-active-directory"></a>Azure Active Directory

### <a name="grant-access-for-azure-ad-user"></a>Concesión de acceso a un usuario de Azure AD

```azurecli
az role assignment create \
                            --role <role> \
                            --assignee <assignee> \
                            --scope /subscriptions/<subId>/resourceGroups/<groupName>/providers/Microsoft.Blockchain/blockchainMembers/<myMemberName>
```

| Parámetro | Descripción |
|---------|-------------|
| **role** | Nombre del rol de Azure AD. |
| **assignee** | Identificador de usuario de Azure AD. Por ejemplo: `user@contoso.com` |
| **scope** | Ámbito de la asignación de roles. Puede ser un miembro de cadena de bloques o de nodo de transacción. |

**Ejemplo**:

Conceda acceso al nodo al usuario de Azure AD para un **miembro** de la cadena de bloques:

```azurecli
az role assignment create \
                            --role 'myRole' \
                            --assignee user@contoso.com \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1
```

**Ejemplo**:

Conceda acceso al nodo al usuario de Azure AD para un **nodo de transacción** de la cadena de bloques:

```azurecli
az role assignment create \
                            --role 'MyRole' \
                            --assignee user@contoso.com \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1/transactionNodes/contosoTransactionNode1
```

### <a name="grant-node-access-for-azure-ad-group-or-application-role"></a>Concesión de acceso al nodo a un rol de aplicación o grupo de Azure AD

```azurecli
az role assignment create \
                            --role <role> \
                            --assignee-object-id <assignee_object_id>
```

| Parámetro | Descripción |
|---------|-------------|
| **role** | Nombre del rol de Azure AD. |
| **assignee-object-id** | Identificador de la aplicación o del grupo de Azure AD. |
| **scope** | Ámbito de la asignación de roles. Puede ser un miembro de cadena de bloques o de nodo de transacción. |

**Ejemplo**:

Conceda acceso al nodo para el **rol de aplicación**.

```azurecli
az role assignment create \
                            --role 'myRole' \
                            --assignee-object-id 22222222-2222-2222-2222-222222222222 \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1
```

### <a name="remove-azure-ad-node-access"></a>Retirada del acceso al nodo de Azure AD

```azurecli
az role assignment delete \
                            --role <myRole> \
                            --assignee <assignee> \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/<myResourceGroup>/providers/Microsoft.Blockchain/blockchainMembers/<myMemberName>/transactionNodes/<myTransactionNode>
```

| Parámetro | Descripción |
|---------|-------------|
| **role** | Nombre del rol de Azure AD. |
| **assignee** | Identificador de usuario de Azure AD. Por ejemplo: `user@contoso.com` |
| **scope** | Ámbito de la asignación de roles. Puede ser un miembro de cadena de bloques o de nodo de transacción. |

## <a name="next-steps"></a>Pasos siguientes

Aprenda cómo [configurar los nodos de transacción de Azure Blockchain Service con Azure Portal](configure-transaction-nodes.md).
