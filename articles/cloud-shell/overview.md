---
title: Introducción a Azure Cloud Shell | Microsoft Docs
description: Introducción a Azure Cloud Shell.
services: ''
documentationcenter: ''
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/20/2020
ms.author: damaerte
ms.openlocfilehash: 766245dcfe42c16106df6beed938fe2f04af05cc
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286685"
---
# <a name="overview-of-azure-cloud-shell"></a>Introducción a Azure Cloud Shell

Azure Cloud Shell es un shell interactivo, autenticado y al que se puede acceder desde un explorador para administrar recursos de Azure. Ofrece la flexibilidad de poder elegir la experiencia de shell que mejor se adapte a la forma de trabajar de cada uno, Bash o PowerShell.

Puede acceder a Cloud Shell de tres maneras:

- **Vínculo directo**: Abra un explorador en [https://shell.azure.com](https://shell.azure.com).

- **Portal de Azure**: Seleccione el icono de Cloud Shell en [Azure Portal](https://portal.azure.com):

    ![Icono para iniciar Cloud Shell desde Azure Portal](media/overview/portal-launch-icon.png)

- **Fragmentos de código**: En [docs.microsoft.com](https://docs.microsoft.com) y [Microsoft Learn](https://docs.microsoft.com/learn/), seleccione el botón **Probar** que aparece con los fragmentos de código de la CLI de Azure y Azure PowerShell:

    ```azurecli-interactive
    az account show
    ```

    ```azurepowershell-interactive
    Get-AzSubscription
    ```

    El botón **Probar** abre Cloud Shell directamente junto con la documentación mediante Bash (para fragmentos de código de la CLI de Azure) o PowerShell (para los fragmentos de código de Azure PowerShell).

    Para ejecutar el comando, use **Copiar** en el fragmento de código, use **Ctrl**+**Mayús**+**V** (Windows/Linux) o **Cmd**+**Mayús**+**V** (macOS) para pegar el comando y, a continuación, presione **Entrar**.

## <a name="features"></a>Características

### <a name="browser-based-shell-experience"></a>Experiencia de shell basada en explorador

Cloud Shell permite el acceso a una experiencia de línea de comandos basada en explorador compilada con las tareas de administración de Azure en mente. Aproveche Cloud Shell para trabajar sin restricción de un equipo local que una forma que solo puede proporcionar la nube.

### <a name="choice-of-preferred-shell-experience"></a>Elección de la experiencia de shell de su preferencia

Los usuarios pueden elegir entre Bash o PowerShell.

1. Seleccione **Cloud Shell**.

    ![Icono de Cloud Shell](media/overview/overview-cloudshell-icon.png)

2. Seleccione **Bash** o **PowerShell**.

    ![Elección de Bash o PowerShell](media/overview/overview-choices.png)

    Después del primer inicio, puede usar el control desplegable de tipo de shell para cambiar entre Bash y PowerShell:

    ![Control desplegable para seleccionar Bash o PowerShell](media/overview/select-shell-drop-down.png)

### <a name="authenticated-and-configured-azure-workstation"></a>Estación de trabajo de Azure autenticada y configurada

Microsoft administra Cloud Shell, por lo que incluye herramientas de línea de comandos populares y compatibilidad para distintos lenguajes. Cloud Shell también se autentica de forma segura y automática para obtener acceso inmediato a los recursos mediante la CLI de Azure o los cmdlets de Azure PowerShell.

Vea toda la [lista de herramientas instaladas en Cloud Shell.](features.md#tools)

### <a name="integrated-cloud-shell-editor"></a>Editor de Cloud Shell integrado

Cloud Shell ofrece un editor de texto gráfico integrado basado en el Editor Monaco de código abierto. El mero hecho de ejecutar `code .` para crear y editar los archivos de configuración permite una implementación fluida mediante la CLI de Azure o Azure PowerShell.

[Más información acerca del editor de Cloud Shell](using-cloud-shell-editor.md).

### <a name="multiple-access-points"></a>Varios puntos de acceso

Cloud Shell es una herramienta flexible que puede utilizarse desde:

* [portal.azure.com](https://portal.azure.com)
* [shell.azure.com](https://shell.azure.com)
* [Documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure)
* [Documentación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/)
* [Aplicación móvil de Azure](https://azure.microsoft.com/features/azure-portal/mobile-app/)
* [La extensión de la cuenta de Azure de Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)

### <a name="connect-your-microsoft-azure-files-storage"></a>Conexión con el almacenamiento de Microsoft Azure Files

Las máquinas de Cloud Shell son temporales, pero los archivos se conservan de dos maneras: a través de una imagen de disco y a través de un recurso compartido de archivos montado denominado `clouddrive`. Al iniciarse por primera vez, Cloud Shell le indica que va a crear un grupo de recursos, una cuenta de almacenamiento y un recurso compartido de Azure Files en su nombre. Esto es un paso único y se adjuntará automáticamente en todas las sesiones. Se puede asignar un recurso de archivos único que Bash y PowerShell usarán en Cloud Shell.

Obtenga más información sobre cómo montar una [cuenta de almacenamiento nueva o existente](persisting-shell-storage.md) o sobre los [mecanismos de persistencia usados en Cloud Shell](persisting-shell-storage.md#how-cloud-shell-storage-works).

> [!NOTE]
> El firewall de Azure Storage no es compatible con las cuentas de almacenamiento de Cloud Shell.

## <a name="concepts"></a>Conceptos

* Cloud Shell se ejecuta en un host temporal que se proporciona por cada sesión y usuario.
* Cloud Shell agota el tiempo de espera tras 20 minutos sin actividad interactiva.
* Cloud Shell requiere montar un recurso compartido de archivos de Azure.
* Cloud Shell usa el mismo recurso compartido de archivos de Azure para Bash y para PowerShell.
* Se asigna a Cloud Shell una máquina por cuenta de usuario.
* Cloud Shell conserva $HOME con una imagen de 5 GB mantenida en el recurso compartido de archivos
* Los permisos se establecen como usuario de Linux normal en Bash.

Obtenga más información sobre las características en [Bash en Cloud Shell](features.md) y [PowerShell en Cloud Shell](features-powershell.md).

## <a name="pricing"></a>Precios

La máquina que hospeda Cloud Shell es gratis, con un requisito previo de un recurso compartido de Azure Files montado. Se aplican costos por almacenamiento normal.

## <a name="next-steps"></a>Pasos siguientes

[Guía de inicio rápido de Bash en Cloud Shell](quickstart.md) <br>
[Guía de inicio rápido de PowerShell en Cloud Shell](quickstart-powershell.md)
