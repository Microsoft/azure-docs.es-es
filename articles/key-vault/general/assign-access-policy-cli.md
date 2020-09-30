---
title: Asignación de una directiva de acceso de Azure Key Vault (CLI)
description: Cómo se usa la CLI de Azure para asignar una directiva de acceso de Key Vault a una identidad de aplicación o una entidad de servicio.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/27/2020
ms.author: mbaldwin
ms.openlocfilehash: 32a323e8cc56a8c45b18737bb55ebe19d4be27c0
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380562"
---
# <a name="assign-a-key-vault-access-policy"></a>Asignación de una directiva de acceso de Key Vault

Las directivas de acceso de Key Vault determinan si una entidad de servicio determinada, es decir, una aplicación o un grupo de usuarios, puede realizar distintas operaciones en los [secretos](../secrets/index.yml), las [claves](../keys/index.yml) y los [certificados](../certificates/index.yml) de Key Vault. Las directivas de acceso se pueden asignar mediante [Azure Portal](assign-access-policy-portal.md), la [CLI de Azure](assign-access-policy-powershell.md) (este artículo) o Azure PowerShell.

[!INCLUDE [key-vault-access-policy-limits.md](../../../includes/key-vault-access-policy-limits.md)]

Para más información sobre cómo crear grupos en Azure Active Directory mediante la CLI de Azure, consulte [az ad group create](/cli/azure/ad/group?view=azure-cli-latest#az-ad-group-create) y [az ad group member add](/cli/azure/ad/group/member?view=azure-cli-latest#az-ad-group-member-add).

## <a name="configure-the-azure-cli-and-sign-in"></a>Configuración de la CLI de Azure e inicio de sesión en ella

1. Para ejecutar los comandos de la CLI de Azure en un entorno local, instale la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).
 
    Para ejecutar comandos directamente en la nube, use [Azure Cloud Shell](/azure/cloud-shell/overview).

1. Solo la CLI local: inicie sesión en Azure mediante `az login`:

    ```bash
    az login
    ```

    El comando `az login` abre una ventana del explorador que recopila sus credenciales, en caso de que sea necesario.

## <a name="acquire-the-object-id"></a>Obtención del identificador del objeto

Determine el identificador del objeto de la aplicación, el grupo o el usuario al que desea asignar la directiva de acceso:

- Aplicaciones y otras entidades de servicio: use el comando [az ad sp list](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-list) para recuperar las entidades de servicio. Examine la salida del comando para determinar el identificador de objeto de la entidad de seguridad a la que desea asignar la directiva de acceso.

    ```azurecli-interactive
    az ad sp list --show-mine
    ```

- Grupos: use el comando [az ad group list](/cli/azure/ad/group?view=azure-cli-latest#az-ad-group-list) y filtre los resultados con el parámetro `--display-name`:

     ```azurecli-interactive
    az ad group list --display-name <search-string>
    ```

- Usuarios: use el comando [az ad user show](/cli/azure/ad/user?view=azure-cli-latest#az-ad-user-show) y pase la dirección de correo electrónico del usuario en el parámetro `--id` :

    ```azurecli-interactive
    az ad user show --id <email-address-of-user>
    ```

## <a name="assign-the-access-policy"></a>Asignación de una directiva de acceso
    
Use el comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) para asignar los permisos que desee:

```azurecli-interactive
az keyvault set-policy --name myKeyVault --object-id <object-id> --secret-permissions <secret-permissions> --key-permissions <key-permissions> --certificate-permissions <certificate-permissions>
```

Reemplace `<object-id>` por el identificador de objeto de la entidad de servicio.

Solo necesita incluir `--secret-permissions`, `--key-permissions` y `--certificate-permissions` al asignar permisos a esos tipos concretos. Los valores que se permiten para `<secret-permissions>`, `<key-permissions>`y `<certificate-permissions>` se especifican en la documentación de [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy).

## <a name="next-steps"></a>Pasos siguientes

- [Seguridad en Azure Key Vault: Administración de identidades y acceso](overview-security.md#identity-and-access-management)
- [Protección del almacén de claves](secure-your-key-vault.md)
- [Guía del desarrollador de Azure Key Vault](developers-guide.md)
- [Procedimientos recomendados de Azure Key Vault](best-practices.md)
