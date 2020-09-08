---
title: Funcionamiento de la depuración remota del código con Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Describe los procesos de depuración remota en Azure Kubernetes Service con Azure Dev Spaces.
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores
ms.openlocfilehash: fd984ff6a8ebe336f76643406c0957769dbfd3da
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213384"
---
# <a name="how-remote-debugging-your-code-with-azure-dev-spaces-works"></a>Funcionamiento de la depuración remota del código con Azure Dev Spaces

Azure Dev Spaces le ofrece varias formas de iterar y depurar rápidamente las aplicaciones Kubernetes y colaborar con su equipo en un clúster de Azure Kubernetes Service (AKS). Una vez que el proyecto se ejecuta en un espacio de desarrollo, Azure Dev Spaces proporciona una manera de asociar y depurar una aplicación en ejecución en AKS.

En este artículo se describe cómo funciona la depuración remota con Dev Spaces.

## <a name="debug-your-code"></a>Depuración del código

En el caso de las aplicaciones de Java, .NET Core y Node.js, puede usar Visual Studio Code o Visual Studio para depurar la aplicación que se ejecuta directamente en el espacio de desarrollo. Visual Studio Code y Visual Studio proporcionan herramientas para conectarse al espacio de desarrollo, iniciar la aplicación y asociar un depurador. Después de ejecutar `azds prep`, puede abrir el proyecto en Visual Studio Code o Visual Studio. Visual Studio Code o Visual Studio generarán sus propios archivos de configuración para la conexión, que es independiente de la ejecución de `azds prep`. Desde Visual Studio Code o Visual Studio, puede establecer puntos de interrupción e iniciar la aplicación en el espacio de desarrollo.

![Depuración del código](media/get-started-node/debug-configuration-nodejs2.png)

Al iniciar la aplicación con Visual Studio Code o Visual Studio para la depuración, estos se encargan de iniciarla y conectarla al espacio de desarrollo de la misma manera que la ejecución de `azds up`. Así mismo, las herramientas del lado cliente de Visual Studio Code y Visual Studio proporcionan un parámetro adicional con información específica para la depuración. El parámetro contiene el nombre de la imagen del depurador, la ubicación del depurador en la imagen y la ubicación de destino en el contenedor de la aplicación para montar la carpeta del depurador.

Las herramientas de cliente determinan automáticamente la imagen del depurador mediante un método similar al que se usa durante la generación del archivo Dockerfile y el gráfico de Helm cuando se ejecuta `azds prep`. Una vez que se ha montado el depurador en la imagen de la aplicación, se ejecuta con `azds exec`.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo funciona Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Funcionamiento de Azure Dev Spaces](how-dev-spaces-works.md)
