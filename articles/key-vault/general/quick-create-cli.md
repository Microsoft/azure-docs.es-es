---
title: 'Inicio rápido: Creación de un almacén de claves de Azure Key Vault con la CLI de Azure'
description: Inicio rápido que muestra cómo crear un almacén de claves de Azure Key Vault mediante la CLI de Azure
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: quickstart
ms.date: 07/20/2020
ms.author: mbaldwin
ms.openlocfilehash: f3b9758d52c4be8e4d85c398f5ef1d0b3fae7e86
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541811"
---
# <a name="quickstart-create-a-key-vault-using-the-azure-cli"></a>Inicio rápido: Creación de un almacén de claves mediante la CLI de Azure

Azure Key Vault es un servicio de almacenamiento seguro en la nube para [claves](../keys/index.yml), [secretos](../secrets/index.yml) y [certificados](../certificates/index.yml). Para obtener más información sobre Key Vault, consulte el artículo [Acerca de Azure Key Vault](overview.md); para obtener más información sobre lo que se puede almacenar en un almacén de claves, consulte el artículo [Acerca de las claves, secretos y certificados](about-keys-secrets-certificates.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

En este inicio rápido, creará un almacén de claves con la [CLI de Azure](/cli/azure/). La CLI de Azure se usa para crear y administrar recursos de Azure mediante comandos o scripts.  Si decide instalar y usar la CLI en un entorno local, para esta guía de inicio rápido se necesita la versión 2.0.4 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Para iniciar sesión en Azure mediante la CLI puede escribir:

```azurecli
az login
```

Para más información sobre las opciones de inicio de sesión mediante la CLI, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli?view=azure-cli-latest)

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *ContosoResourceGroup* en la ubicación *eastus*.

```azurecli
az group create --name "myResourceGroup" -l "EastUS"
```

## <a name="create-a-key-vault"></a>Creación de un Almacén de claves

Cree un almacén de claves en el grupo de recursos del paso anterior. Tendrá que proporcionar algo de información:

- Nombre del almacén de claves: cadena de entre 3 y 24 caracteres que puede contener solo números (0-9), letras (a-z, A-Z) y guiones (-).

  > [!Important]
  > Cada almacén de claves debe tener un nombre único. Reemplace <nombre-almacén de claves-único> por el nombre del almacén de claves en los ejemplos siguientes.

- Nombre del grupo de recursos: **myResourceGroup**.
- Ubicación: **EastUS**.

```azurecli
az keyvault create --name "<your-unique-keyvault-name>" --resource-group "myResourceGroup" --location "EastUS"
```

La salida de este cmdlet muestra las propiedades del almacén de claves que acaba de crear. Tome nota de las dos propiedades siguientes:

- **Nombre del almacén**: nombre que proporcionó al parámetro --name anterior.
- **URI de almacén**: en el ejemplo, es https://&lt;su-nombre-de-almacén-de-claves-único&gt;.vault.azure.net/. Las aplicaciones que utilizan el almacén a través de su API de REST deben usar este identificador URI.

En este momento, su cuenta de Azure es la única autorizada para realizar operaciones en este nuevo almacén.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de esta colección se basan en los valores de esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group) de la CLI de Azure para quitar el grupo de recursos y todos los recursos relacionados.

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado un almacén de claves y lo ha eliminado. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](overview.md).
- Consulte la referencia de los [comandos az keyvault de la CLI de Azure](/cli/azure/keyvault?view=azure-cli-latest).
- Consulte los [procedimientos recomendados de Azure Key Vault](best-practices.md).
