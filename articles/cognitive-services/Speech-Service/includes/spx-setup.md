---
author: v-demjoh
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 05/15/2020
ms.author: v-demjoh
ms.openlocfilehash: abfb4f6ba9452581811db1f462089cbafc771266
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86545121"
---
## <a name="prerequisites"></a>Requisitos previos

El único requisito previo es una suscripción a Azure Speech. Consulte la [guía](../get-started.md#new-resource) sobre cómo crear una suscripción si aún no tiene una.

## <a name="download-and-install"></a>Descargar e instalar

#### <a name="windows-install"></a>[Instalación de Windows](#tab/windowsinstall)

Siga estos pasos para instalar la CLI de Voz en Windows:

1. Instale [.NET Framework 4.7](https://dotnet.microsoft.com/download/dotnet-framework/net471) o [.NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0).
2. Descargue el [archivo ZIP](https://aka.ms/speech/spx-zips.zip) de la CLI de Voz y extráigalo.
3. Vaya al directorio raíz `spx-zips` que extrajo de la descarga y extraiga el subdirectorio que necesita (`spx-net471` para .NET Framework 4.7 o `spx-netcore-win-x64` para .NET Core 3.0 en una CPU x64).

En el símbolo del sistema, cambie el directorio a esta ubicación y, luego, escriba `spx` para ver la ayuda de la CLI de Voz.

> [!NOTE]
> En Windows, la CLI de Voz solo puede mostrar las fuentes disponibles para el símbolo del sistema en el equipo local.
> El [terminal de Windows](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701) admite todas las fuentes que genera de forma interactiva la CLI de Voz.
> Si se genera la salida a un archivo, un editor de texto como el Bloc de notas o un explorador Web como Microsoft Edge también pueden mostrar todas las fuentes.

> [!NOTE]
> PowerShell no comprueba el directorio local al buscar un comando. En PowerShell, cambie el directorio a la ubicación de `spx` y llame a la herramienta escribiendo `.\spx`.
> Si agrega este directorio a su ruta de acceso, PowerShell y el símbolo del sistema de Windows buscarán `spx` en cualquier directorio sin incluir el prefijo `.\`.

#### <a name="linux-install"></a>[Instalación de Linux](#tab/linuxinstall)

Siga estos pasos para instalar la CLI de Voz en Linux en una CPU x64:

1. Instale [.NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0).
2. Descargue el [archivo ZIP](https://aka.ms/speech/spx-zips.zip) de la CLI de Voz y extráigalo.
3. Vaya al directorio raíz `spx-zips` que extrajo de la descarga y extraiga `spx-netcore-30-linux-x64` en un nuevo directorio `~/spx`.
4. En un terminal, escriba estos comandos:
   1. `cd ~/spx`
   2. `sudo chmod +r+x spx`
   3. `PATH=~/spx:$PATH`

Escriba `spx` para ver la ayuda de la CLI de Voz.

***

## <a name="create-subscription-config"></a>Creación de la configuración de la suscripción

Para empezar a usar la CLI de Voz, primero debe escribir la clave de la suscripción y la información de la región de Voz. Consulte la página de [soporte por regiones](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#speech-sdk) para buscar el identificador de región. Una vez que tenga la clave de suscripción y el identificador de región (p. ej., `eastus`, `westus`), ejecute los comandos siguientes.

```shell
spx config @key --set YOUR-SUBSCRIPTION-KEY
spx config @region --set YOUR-REGION-ID
```

La autenticación de la suscripción se almacena ahora para futuras solicitudes de SPX. Si tiene que quitar cualquiera de estos valores almacenados, ejecute `spx config @region --clear` o `spx config @key --clear`.
