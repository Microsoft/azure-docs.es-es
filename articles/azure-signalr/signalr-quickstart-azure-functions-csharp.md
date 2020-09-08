---
title: 'Inicio rápido de Azure SignalR Service sin servidor: C#'
description: Una guía de inicio rápido para usar el servicio Azure SignalR y Azure Functions para crear un salón de chat.
author: sffamily
ms.service: signalr
ms.devlang: dotnet
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.date: 03/04/2019
ms.author: zhshang
ms.openlocfilehash: 54a4c9374c064073620f2679e7c7511e30c45de6
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050648"
---
# <a name="quickstart-create-a-chat-room-with-azure-functions-and-signalr-service-using-c"></a>Inicio rápido: Creación de un salón de chat con Azure Functions y SignalR Service mediante C\#

El servicio Azure SignalR le permite agregar fácilmente funcionalidad en tiempo real a la aplicación. Azure Functions es una plataforma sin servidor que le permite ejecutar el código sin tener que administrar ninguna infraestructura. En esta guía de inicio rápido, obtenga información sobre cómo usar el servicio SignalR y Functions para crear una aplicación de chat sin servidor en tiempo real.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene Visual Studio 2019 instalado, puede descargar y usar la **versión gratuita** de [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.

También puede ejecutar este tutorial en la línea de comandos (macOS, Windows o Linux) mediante [Azure Functions Core Tools (v2)](https://github.com/Azure/azure-functions-core-tools#installing), el [SDK de .NET Core](https://dotnet.microsoft.com/download) y su editor de código favorito.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qscsharp)

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en <https://portal.azure.com/> con su cuenta de Azure.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qscsharp)

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qscsharp)

[!INCLUDE [Clone application](includes/signalr-quickstart-clone-application.md)]

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qscsharp)

## <a name="configure-and-run-the-azure-function-app"></a>Configuración y ejecución de la aplicación Azure Function

1. Inicie Visual Studio u otro editor de código y abra la solución en la carpeta *src/chat/csharp* del repositorio clonado.

1. En el explorador que se muestra al abrir Azure Portal, confirme que la instancia del servicio SignalR que implementó anteriormente se ha creado correctamente buscando su nombre en el cuadro de búsqueda de la parte superior del portal. Seleccione la instancia para abrirla.

    ![Búsqueda del nombre de la instancia del servicio SignalR](media/signalr-quickstart-azure-functions-csharp/signalr-quickstart-search-instance.png)

1. Seleccione **Claves** para ver las cadenas de conexión para la instancia del servicio SignalR.

1. Seleccione y copie la cadena de conexión principal.

1. En Visual Studio, en el Explorador de soluciones, cambie el nombre de *local.settings.sample.json* a *local.settings.json*.

1. En **local.settings.json**, pegue la cadena de conexión en el valor de la configuración **AzureSignalRConnectionString**. Guarde el archivo.

1. Abra **Functions.cs**. Hay dos funciones desencadenadas por HTTP en esta aplicación de función:

    - **GetSignalRInfo**: usa el enlace de entrada *SignalRConnectionInfo* para generar y devolver información de conexión válida.
    - **SendMessage**: recibe un mensaje de chat en el cuerpo de la solicitud y usa el enlace de salida *SignalR* para difundir el mensaje a todas las aplicaciones cliente conectadas.

1. Use una de las opciones siguientes para iniciar localmente la aplicación de función de Azure.

    - **Visual Studio**: En el menú *Depurar*, seleccione *Iniciar depuración* para ejecutar la aplicación.

        ![Depurar la aplicación](media/signalr-quickstart-azure-functions-csharp/signalr-quickstart-debug-vs.png)

    - **Línea de comandos**: Ejecute el siguiente comando para iniciar el host de la función.

        ```bash
        func start
        ```
[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qscsharp)

[!INCLUDE [Run web application](includes/signalr-quickstart-run-web-application.md)]

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qscsharp)

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qscsharp)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha compilado y ejecutado una aplicación sin servidor en tiempo real en Visual Studio. A continuación, obtenga más información acerca de cómo desarrollar e implementar Azure Functions con Visual Studio.

> [!div class="nextstepaction"]
> [Desarrollo de Azure Functions con Visual Studio](../azure-functions/functions-develop-vs.md)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qscsharp)
