---
title: 'Creación de un miembro de Azure Blockchain Service: CLI de Azure'
description: Cree un miembro de Azure Blockchain Service para un consorcio de la cadena de bloques mediante la CLI de Azure.
ms.date: 07/23/2020
ms.topic: quickstart
ms.reviewer: ravastra
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: f97aab59d38e9b15838a78d0227bc2848615cd92
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504371"
---
# <a name="quickstart-create-an-azure-blockchain-service-blockchain-member-using-azure-cli"></a>Inicio rápido: Creación de un miembro de cadena de bloques de Azure Blockchain Service mediante la CLI de Azure

En esta guía de inicio rápido, implementará un nuevo miembro de la cadena de bloques y un consorcio en Azure Blockchain Service mediante la CLI de Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Ninguno.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/bash](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

Si prefiere instalar y usar la CLI de forma local, en este inicio rápido se requiere la versión 2.0.51 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

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

1. Cree un grupo de recursos.

    Azure Blockchain Service, como todos los recursos de Azure, se debe implementar en un grupo de recursos. Los grupos de recursos le permiten organizar y administrar los recursos relacionados de Azure.

    Para esta guía de inicio rápido, cree un grupo de recursos llamado _myResourceGroup_ en la ubicación _eastus_ con el comando [az group create](/cli/azure/group#az-group-create) siguiente:

    ```azurecli-interactive
    az group create \
                     --name "myResourceGroup" \
                     --location "eastus"
    ```

## <a name="create-a-blockchain-member"></a>Creación de un miembro de cadena de bloques

Un miembro de Azure Blockchain Service es un nodo de la cadena de bloques en una red de cadenas de bloques del consorcio privado. Al aprovisionar un miembro, puede crear una red de consorcio o unirse a ella. Necesita al menos un miembro para una red de consorcio. El número de miembros de la cadena de bloques que necesitan los participantes depende de su escenario. Los participantes del consorcio pueden tener uno o varios miembros de la cadena de bloques o pueden compartir miembros con otros participantes. Para más información sobre los consorcios, consulte [Consorcio de Azure Blockchain Service](consortium.md).

Hay varios parámetros y propiedades que se deben pasar. Reemplace los parámetros de ejemplo por los suyos propios.

```azurecli-interactive
az blockchain member create \
                            --resource-group "MyResourceGroup" \
                            --name "myblockchainmember" \
                            --location "eastus" \
                            --password "strongMemberAccountPassword@1" \
                            --protocol "Quorum" \
                            --consortium "myconsortium" \
                            --consortium-management-account-password "strongConsortiumManagementPassword@1" \
                            --sku "Basic"
```

| Parámetro | Descripción |
|---------|-------------|
| **resource-group** | Nombre del grupo de recursos donde se crean los recursos de Azure Blockchain Service. Use el grupo de recursos que creó en la sección anterior.
| **name** | Un nombre único que identifica al miembro de cadena de bloques de Azure Blockchain Service. El nombre se usa como dirección de punto de conexión público. Por ejemplo, `myblockchainmember.blockchain.azure.com`.
| **ubicación** | Región de Azure donde se crea el miembro de cadena de bloques. Por ejemplo, `westus2`. Elija la ubicación más cercana a los usuarios o a sus otras aplicaciones de Azure. Es posible que las características no estén disponibles en algunas regiones. Azure Blockchain Data Manager está disponible en las siguientes regiones de Azure: Este de EE. UU. y Oeste de Europa.
| **password** | La contraseña del nodo de transacción predeterminado del miembro. Use la contraseña para la autenticación básica al conectarse al punto de conexión público del nodo de transacción predeterminado del miembro de la cadena de bloques.
| **protocolo** | Protocolo de la cadena de bloques. Actualmente, se admite el protocolo *Quorum*.
| **consortium** | Nombre del consorcio al que unirse o que se va a crear. Para más información sobre los consorcios, consulte [Consorcio de Azure Blockchain Service](consortium.md).
| **consortium-management-account-password** | Este valor también se conoce como la contraseña de la cuenta del miembro, y se usa para cifrar la clave privada de la cuenta de Ethereum que se crea para el miembro. Usará la cuenta del miembro y la contraseña de la cuenta del miembro para la administración del consorcio.
| **sku** | Tipo de nivel. *Estándar* o *Básico*. Use el nivel *Básico* para desarrollo, pruebas y prueba de concepto. Use el nivel *Estándar* para las implementaciones de nivel de producción. También debe usar el nivel *Estándar* si va a utilizar Blockchain Data Manager o va a enviar un gran volumen de transacciones privadas. Después de la creación de un miembro, no se puede cambiar el plan de tarifa de Básico a Estándar.

La creación del miembro de cadena de bloques y de los recursos auxiliares tarda unos 10 minutos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede usar el miembro de cadena de bloques que ha creado en el siguiente inicio rápido o tutorial. Cuando ya no necesite los recursos, puede eliminarlos; para ello, elimine el grupo de recursos `myResourceGroup` que creó para el inicio rápido.

Ejecute el siguiente comando para quitar el grupo de recursos y todos los recursos relacionados.

```azurecli-interactive
az group delete \
                 --name myResourceGroup \
                 --yes
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha implementado un miembro de Azure Blockchain Service y un nuevo consorcio. Pruebe el siguiente inicio rápido para usar Azure Blockchain Development Kit para Ethereum para adjuntar a un miembro de Azure Blockchain Service.

> [!div class="nextstepaction"]
> [Uso de Visual Studio Code para conectarse a Azure Blockchain Service](connect-vscode.md)
