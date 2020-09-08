---
title: 'Tutorial de creación, compilación e implementación de contratos inteligentes: Azure Blockchain Service'
description: Tutorial sobre cómo usar el kit de desarrollo de Azure Blockchain para la extensión Ethereum en Visual Studio Code a fin de crear, compilar e implementar un contrato inteligente en Azure Blockchain Service.
ms.date: 04/22/2020
ms.topic: tutorial
ms.reviewer: caleteet
ms.openlocfilehash: dc23c680dfb2ed33cae2a251af16e1b1f25c6ac7
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "82086664"
---
# <a name="tutorial-create-buildanddeploysmartcontracts-on-azure-blockchain-service"></a>Tutorial: Creación, compilación e implementación de contratos inteligentes en Azure Blockchain Service

En este tutorial, usará el kit de desarrollo de Azure Blockchain para la extensión Ethereum en Visual Studio Code a fin de crear, compilar e implementar un contrato inteligente en Azure Blockchain Service. También puede usar el kit de desarrollo para ejecutar una función de contrato inteligente mediante una transacción.

Use el kit de desarrollo de Azure Blockchain para Ethereum para:

> [!div class="checklist"]
> * Crear un contrato inteligente
> * Implementar un contrato inteligente
> * Ejecutar una función de contrato inteligente mediante una transacción

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

* Realizar el tutorial [Quickstart: Uso de Visual Studio Code para conectarse a una red del consorcio de Azure Blockchain Service](connect-vscode.md)
* [Visual Studio Code](https://code.visualstudio.com/Download)
* [Kit de desarrollo de Azure Blockchain para la extensión de Ethereum](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain)
* [Node.js 10.15.x o superior](https://nodejs.org/download)
* [Git 2.10.x o superior](https://git-scm.com)
* [Python 2.7.15](https://www.python.org/downloads/release/python-2715/) Agregue python.exe a la ruta de acceso. Se necesita la versión 2.7.15 de Python en la ruta de acceso de Azure Blockchain Development Kit.
* [Truffle 5.0.0](https://www.trufflesuite.com/docs/truffle/getting-started/installation)
* [CLI de Ganache 6.0.0](https://github.com/trufflesuite/ganache-cli)

En Windows, se requiere un compilador de C++ instalado para el módulo node-gyp. Puede usar las herramientas de MSBuild:

* Si está instalado Visual Studio 2017, configure npm para que use las herramientas de MSBuild con el comando `npm config set msvs_version 2017 -g`
* Si está instalado Visual Studio 2019, establezca la ruta de acceso de las herramientas de MSBuild para npm. Por ejemplo: `npm config set msbuild_path "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin\MSBuild.exe"`
* En caso contrario, instale las herramientas independientes de VS Build mediante `npm install --global windows-build-tools` en un shell de comandos *Ejecutar como administrador* con privilegios elevados.

Para más información sobre node-gyp, consulte el [repositorio de node-gyp en GitHub](https://github.com/nodejs/node-gyp).

## <a name="create-a-smart-contract"></a>Crear un contrato inteligente

El kit de desarrollo de Azure Blockchain para Ethereum usa plantillas de proyecto y herramientas de Truffle para ayudar a preparar, compilar e implementar contratos. Antes de empezar, debe cumplir los siguientes requisitos previos [Inicio rápido: Uso de Visual Studio Code para conectarse a una red del consorcio de Azure Blockchain Service](connect-vscode.md). El inicio rápido le guía en la instalación y configuración del Kit de desarrollo de Azure Blockchain para Ethereum.

1. En la paleta de comandos de VS Code, elija **Azure Blockchain: New Solidity Project** (Azure Blockchain: nuevo proyecto de Solidity).
1. Elija **Create basic project** (Crear proyecto básico).
1. Cree una nueva carpeta denominada `HelloBlockchain` y seleccione **New project path** (Nueva ruta de acceso de proyecto).

El kit de desarrollo de Azure Blockchain crea e inicializa un nuevo proyecto de Solidity automáticamente. El proyecto básico incluye un contrato inteligente de ejemplo **HelloBlockchain** y todos los archivos necesarios para compilar e implementar en el miembro del consorcio en Azure Blockchain Service. La creación del proyecto puede tardar unos minutos. Puede supervisar el progreso en el panel del terminal de VS Code; para ello, seleccione la salida de Azure Blockchain.

La estructura del proyecto debe ser similar al siguiente ejemplo:

   ![Proyecto de Solidity](./media/send-transaction/solidity-project.png)

## <a name="build-a-smart-contract"></a>Creación de un contrato inteligente

Los contratos inteligentes se encuentran en el directorio de **contratos** del proyecto. Primero se crean los contratos inteligentes y después se implementan en una cadena de bloques. Use el comando **Build Contracts** para crear todos los contratos inteligentes del proyecto.

1. En la barra lateral del explorador de VS Code, expanda la carpeta de **contratos** del proyecto.
1. Haga clic con el botón derecho en **HelloBlockchain.sol** y elija **Build Contracts** (Crear contratos) en el menú.

    ![Selección del menú Build contracts (Crear contratos) ](./media/send-transaction/build-contracts.png)

El kit de desarrollo de Azure Blockchain usa Truffle para compilar los contratos inteligentes.

![Salida del compilador Truffle](./media/send-transaction/compile-output.png)

## <a name="deploy-a-smart-contract"></a>Implementar un contrato inteligente

Truffle usa scripts de migración para implementar los contratos en una red de Ethereum. Las migraciones son archivos JavaScript ubicados en el directorio de **migraciones** del proyecto.

1. Para implementar el contrato inteligente, haga clic con el botón derecho en **HelloBlockchain.sol** y elija **Deploy Contracts** (Implementar contratos) en el menú.
1. Elija su red de consorcio de Azure Blockchain en la paleta de comandos. La red de consorcio de la red de la cadena de bloques se agregó al archivo de configuración Truffle del proyecto al crearlo.
1. Elija **Generate mnemonic** (Generar mnemotécnico). Elija un nombre de archivo y guarde el archivo mnemotécnico en la carpeta del proyecto. Por ejemplo, `myblockchainmember.env`. El archivo mnemotécnico se usa para generar una clave privada de Ethereum para el miembro de la cadena de bloques.

El kit de desarrollo de Azure Blockchain usa Truffle para ejecutar el script de migración con el fin de implementar los contratos en la cadena de bloques.

![Contrato implementado correctamente](./media/send-transaction/deploy-contract.png)

## <a name="call-a-contract-function"></a>Llamada a una función de contrato

La función **SendRequest** del contrato **HelloBlockchain** cambia la variable de estado **RequestMessage**. El cambio del estado de una red de la cadena de bloques se realiza con una transacción. Puede usar la página de interacción del contrato inteligente del kit de desarrollo de Azure Blockchain para llamar a la función **SendRequest** mediante una transacción.

1. Para interactuar con el contrato inteligente, haga clic con el botón derecho en **HelloBlockchain.sol** y elija **Show Smart Contract Interaction Page** (Mostrar página de interacción del contrato inteligente) en el menú.

    ![Selección de Show Smart Contract Interaction Page (Mostrar página de interacción del contrato inteligente) en el menú](./media/send-transaction/contract-interaction.png)

1. La página de interacción permite elegir una versión del contrato implementada, llamar a funciones, ver el estado actual y ver los metadatos.

    ![Página de interacción del contrato inteligente de ejemplo](./media/send-transaction/interaction-page.png)

1. Para llamar a una función del contrato inteligente, seleccione la acción del contrato y pase los argumentos. Elija la acción de contrato **SendRequest** y escriba **Hello, Blockchain!** para el parámetro **requestMessage**. Seleccione **Execute** (Ejecutar) para llamar a la función **SendRequest** mediante una transacción.

    ![Ejecutar la acción SendRequest](./media/send-transaction/sendrequest-action.png)

Una vez que se procesa la transacción, la sección de interacción refleja los cambios de estado.

![Cambios de estado del contrato](./media/send-transaction/contract-state.png)

La función SendRequest establece los campos **RequestMessage** y **State**. El estado actual de **RequestMessage** es el argumento que pasó, **Hello, blockchain**. El valor del campo **State** permanece en **Request**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos, puede eliminarlos; para ello, elimine el grupo de recursos `myResourceGroup` que creó en el inicio rápido de requisitos previos *Creación de un miembro de cadena de bloques*.

Para eliminar el grupo de recursos:

1. En Azure Portal, vaya a **Grupo de recursos** en el panel de navegación izquierdo y seleccione el grupo de recursos que desea eliminar.
1. Seleccione **Eliminar grupo de recursos**. Compruebe la eliminación escribiendo el nombre del grupo de recursos y seleccionando **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado un proyecto de Solidity de ejemplo mediante el kit de desarrollo de Azure Blockchain. Ha creado e implementado un contrato inteligente y ha llamado a una función con una transacción en una red de cadena de bloques de consorcio hospedada Azure Blockchain Service.

> [!div class="nextstepaction"]
> [Desarrollo de aplicaciones de cadena de bloques con Azure Blockchain Service](develop.md)
