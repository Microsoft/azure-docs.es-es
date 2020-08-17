---
title: Creación y recuperación de los atributos de una clave en Azure Key Vault mediante la CLI de Azure
description: Inicio rápido en el que se muestra cómo establecer y recuperar una clave de Azure Key Vault mediante la CLI de Azure
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.date: 03/30/2020
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: e59482c3dbc4bd0abb20a5e9094aad5f31c358cb
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497416"
---
# <a name="quickstart-set-and-retrieve-a-key-from-azure-key-vault-using-azure-cli"></a>Inicio rápido: Establecimiento y recuperación de una clave desde Azure Key Vault mediante la CLI de Azure

En este inicio rápido, creará un almacén de claves en Azure Key Vault con la CLI de Azure. Azure Key Vault es un servicio en la nube que funciona como un almacén de secretos seguro. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Para más información sobre Key Vault, puede consultar esta [introducción](../general/overview.md). La CLI de Azure se usa para crear y administrar recursos de Azure mediante comandos o scripts. Una vez que haya terminado, almacenará una clave.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.


[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI en un entorno local, para esta guía de inicio rápido se necesita la versión 2.0.4 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

Para iniciar sesión en Azure mediante la CLI puede escribir:

```azurecli
az login
```

Para más información sobre las opciones de inicio de sesión mediante la CLI, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli?view=azure-cli-latest)

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *ContosoResourceGroup* en la ubicación *eastus*.

```azurecli
az group create --name "ContosoResourceGroup" --location eastus
```

## <a name="create-a-key-vault"></a>Creación de un almacén de claves

A continuación, creará una instancia de Key Vault en el grupo de recursos creado en el paso anterior. Tendrá que proporcionar algo de información:

- En esta guía de inicio rápido se usará **Contoso-vault2**. Debe proporcionar un nombre exclusivo en las pruebas.
- Nombre del grupo de recursos: **ContosoResourceGroup**.
- Ubicación: **Este de EE. UU**.

```azurecli
az keyvault create --name "Contoso-Vault2" --resource-group "ContosoResourceGroup" --location eastus
```

La salida de este cmdlet muestra las propiedades del almacén de claves que acaba de crear. Tome nota de las dos propiedades siguientes:

- **Nombre del almacén**: en este ejemplo es **Contoso-Vault2**. Utilizará este nombre para otros comandos de Key Vault.
- **URI de almacén**: en el ejemplo es https://contoso-vault2.vault.azure.net/. Las aplicaciones que utilizan el almacén a través de su API de REST deben usar este identificador URI.

En este momento, su cuenta de Azure es la única autorizada para realizar operaciones en este nuevo almacén.

## <a name="add-a-key-to-key-vault"></a>Incorporación de una clave a Key Vault

Para agregar una clave al almacén, no tiene más que realizar un par de pasos adicionales. Esta clave la podría usar una aplicación. 

Escriba los siguientes comandos para crear un **ExampleKey**:

```azurecli
az keyvault key create --vault-name "Contoso-Vault2" -n ExampleKey --protection software
```

Ahora puede hacer referencia a esta clave que agregó a Azure Key Vault utilizando su URI. Use **https://Contoso-Vault2.vault.azure.net/keys/ExampleKey** para obtener la versión actual. 

Para ver la clave previamente almacenada:

```azurecli

az keyvault key show --name "ExampleKey" --vault-name "Contoso-Vault2"
```

Ya ha creado una instancia de Key Vault, ha almacenado una clave y, posteriormente, la ha recuperado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de esta colección se basan en los valores de esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.
Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos y todos los recursos relacionados. Puede eliminar los recursos como se indica a continuación:

```azurecli
az group delete --name ContosoResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado una instancia de Key Vault y ha almacenado una clave en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la referencia de los [comandos az keyvault de la CLI de Azure](/cli/azure/keyvault?view=azure-cli-latest).
- Consulte los [procedimientos recomendados de Azure Key Vault](../general/best-practices.md).
