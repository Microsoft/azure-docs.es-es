---
title: 'Inicio rápido: Configuración de la plataforma para usar Java en Windows, Linux o macOS con el SDK de Voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Use esta guía para configurar la plataforma para usar Java en Windows, Linux o macOS con el SDK del servicio de voz.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/11/2019
ms.custom: devx-track-java
ms.author: erhopf
ms.openlocfilehash: 2a6516ad5f0fc8d9faefd7b7f89ddb1eaa3fd7d6
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87375747"
---
En esta guía se muestra cómo instalar el [SDK de Voz](~/articles/cognitive-services/speech-service/speech-sdk.md) para Java 8 JRE de 64 bits. Si desea simplemente que el nombre del paquete comience por su cuenta, el SDK de Java no está disponible en el repositorio central de Maven. Si usa Gradle o un archivo de dependencia `pom.xml`, debe agregar un repositorio personalizado que apunte a `https://csspeechstorage.blob.core.windows.net/maven/` (consulte la siguiente información sobre el nombre del paquete).

> [!NOTE]
> Para el SDK de dispositivos de Voz y el dispositivo Roobo, consulte [SDK de dispositivos de voz](~/articles/cognitive-services/speech-service/speech-devices-sdk.md).

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

- El paquete del SDK de Voz para Java está disponible para estos sistemas operativos:
  - Windows: solo 64 bits
  - Mac: macOS X versión 10.13 o posterior
  - Linux: solo 64 bits en Ubuntu 16.04, Ubuntu 18.04, Debian 9, RHEL 7/8 y CentOS 7/8

## <a name="prerequisites"></a>Prerrequisitos

- [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) o [JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

- [IDE de Java para Eclipse](https://www.eclipse.org/downloads/) (es necesario tener ya instalado Java)
- Las plataformas Linux admitidas requerirán la instalación de determinadas bibliotecas (`libssl` para la compatibilidad con la capa de sockets seguros y `libasound2` para la compatibilidad con el audio). Consulte a continuación su distribución para saber cuáles son los comandos necesarios para instalar las versiones correctas de estas bibliotecas.

  - En Ubuntu, ejecute los siguientes comandos para instalar los paquetes necesarios:

    ```sh
    sudo apt-get update
    sudo apt-get install build-essential libssl1.0.0 libasound2
    ```

  - En Debian 9, ejecute los siguientes comandos para instalar los paquetes necesarios:

    ```sh
    sudo apt-get update
    sudo apt-get install build-essential libssl1.0.2 libasound2
    ```

  - En RHEL o CentOS, ejecute los siguientes comandos para instalar los paquetes necesarios:

    ```sh
    sudo yum update
    sudo yum install alsa-lib java-1.8.0-openjdk-devel openssl
    ```

> [!NOTE]
> - En RHEL/CentOS 7, siga las instrucciones sobre [cómo configurar RHEL/CentOS 7 para el SDK de voz](~/articles/cognitive-services/speech-service/how-to-configure-rhel-centos-7.md).
> - En RHEL/CentOS 8, siga las instrucciones sobre [cómo configurar OpenSSL para Linux](~/articles/cognitive-services/speech-service/how-to-configure-openssl-linux.md).

- En Windows, necesita [Microsoft Visual C++ Redistributable para Visual Studio 2019](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads) para su plataforma. Tenga en cuenta que si es la primera vez que instala este paquete, puede que deba reiniciar Windows antes de seguir con esta guía.

## <a name="create-an-eclipse-project-and-install-the-speech-sdk"></a>Creación de un proyecto de Eclipse e instalación del SDK de Voz

[!INCLUDE [](~/includes/cognitive-services-speech-service-quickstart-java-create-proj.md)]

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [windows](../quickstart-list.md)]
