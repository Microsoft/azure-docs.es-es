---
title: 'Depuración interactiva: Instancias de proceso de VS Code y ML (versión preliminar)'
titleSuffix: Azure Machine Learning
description: Configure VS Code remoto para depurar de forma interactiva el código con Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: jmartens
author: j-martens
ms.date: 07/09/2020
ms.openlocfilehash: dfb8dac1b9027acd01b3c13c919d9c3cd8368819
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320126"
---
# <a name="debug-interactively-on-an-azure-machine-learning-compute-instance-with-vs-code-remote-preview"></a>Depuración interactiva en una instancia de proceso de Azure Machine Learning con VS Code remoto (versión preliminar)

En este artículo, aprenderá a configurar Visual Studio Code remoto en una instancia de proceso de Azure Machine Learning para que pueda **depurar el código de forma interactiva** desde VS Code. 

+ Una [instancia de proceso de Azure Machine Learning](concept-compute-instance.md) es una estación de trabajo basada en la nube totalmente administrada para científicos de datos que proporciona funciones de administración y preparación para empresas a los administradores de TI. 


+ El desarrollo de [Visual Studio Code remoto](https://code.visualstudio.com/docs/remote/remote-overview) le permite usar un contenedor, un equipo remoto o el subsistema de Windows para Linux (WSL) como un entorno de desarrollo completo. 

## <a name="prerequisite"></a>Requisito previo  

En las plataformas Windows, debe [instalar un cliente SSH compatible con OpenSSH](https://code.visualstudio.com/docs/remote/troubleshooting#_installing-a-supported-ssh-client) si todavía no lo hay. 

> [!Note]
> PuTTY no se admite en Windows, ya que el comando ssh debe estar en la ruta de acceso. 

## <a name="get-ip-and-ssh-port"></a>Obtención de la dirección IP y el puerto SSH 

1. Vaya a Azure Machine Learning Studio en https://ml.azure.com/.

2. Seleccione el [área de trabajo](concept-workspace.md).
1. Haga clic en la pestaña **Instancias de proceso**.
1. En la columna **URI de la aplicación**, haga clic en el vínculo **SSH** de la instancia de proceso que quiera usar como proceso remoto. 
1. En el cuadro de diálogo, anote la dirección IP y el puerto SSH. 
1. Guarde la clave privada en el directorio ~/.ssh/ del equipo local; por ejemplo, abra un editor para un archivo nuevo y pegue la clave: 

   **Linux**: 
   ```sh
   vi ~/.ssh/id_azmlcitest_rsa  
   ```

   **Windows**: 
   ```
   notepad C:\Users\<username>\.ssh\id_azmlcitest_rsa 
   ```

   La clave privada tendrá un aspecto similar al siguiente:
   ```
   -----BEGIN RSA PRIVATE KEY----- 

   MIIEpAIBAAKCAQEAr99EPm0P4CaTPT2KtBt+kpN3rmsNNE5dS0vmGWxIXq4vAWXD 
   ..... 
   ewMtLnDgXWYJo0IyQ91ynOdxbFoVOuuGNdDoBykUZPQfeHDONy2Raw== 

   -----END RSA PRIVATE KEY----- 
   ```

1. Cambie los permisos en el archivo para asegurarse de que solo usted puede leer el archivo.  
   ```sh
   chmod 600 ~/.ssh/id_azmlcitest_rsa   
   ```

## <a name="add-instance-as-a-host"></a>Adición de la instancia como un host 

Abra el archivo `~/.ssh/config` (Linux) o `C:\Users<username>.ssh\config` (Windows) en un editor y agregue una nueva entrada similar a la siguiente:

```
Host azmlci1 

    HostName 13.69.56.51 

    Port 50000 

    User azureuser 

    IdentityFile ~/.ssh/id_azmlcitest_rsa   
```

Estos son algunos detalles de los campos: 

|Campo|Descripción|
|----|---------|
|Host|Use la abreviatura que quiera para la instancia de proceso |
|HostName|Esta es la dirección IP de la instancia de proceso |
|Port|Este es el puerto que se muestra en el cuadro de diálogo SSH anterior |
|Usuario|Esto se debe establecer en  `azureuser` |
|IdentityFile|Debe apuntar al archivo en el que ha guardado la clave privada |

Ahora, debería poder acceder con ssh a la instancia de proceso con la abreviatura que ha usado antes, `ssh azmlci1`. 

## <a name="connect-vs-code-to-the-instance"></a>Conexión de VS Code a la instancia 

1. [Instalación de Visual Studio Code](https://code.visualstudio.com/).

1. [Instale la extensión Remote SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh). 

1. Haga clic en el icono de Remote-SSH de la izquierda para mostrar las configuraciones de SSH.

1. Haga clic con el botón derecho en la configuración del host SSH que acaba de crear.

1. Seleccione **Connect to Host in Current Window** (Conectar al host en la ventana actual). 

Desde aquí, trabajará por completo en la instancia de proceso y ahora puede editar, depurar, usar git, usar extensiones, etc., al igual que con la instancia local de Visual Studio Code. 

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado Visual Studio Code remoto, puede usar una instancia de proceso como proceso remoto desde Visual Studio Code para depurar el código de forma interactiva. 

[Tutorial: Formar su primer modelo de ML](tutorial-1st-experiment-sdk-train.md) muestra cómo usar una instancia de proceso con un cuaderno integrado.