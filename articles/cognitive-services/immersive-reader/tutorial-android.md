---
title: 'Tutorial: Inicio del Lector inmersivo con los ejemplos de código de Android'
titleSuffix: Azure Cognitive Services
description: En este tutorial, configurará y ejecutará una aplicación Android de ejemplo que inicia el Lector inmersivo.
services: cognitive-services
author: dylankil
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: tutorial
ms.date: 06/10/2020
ms.author: dylankil
ms.custom: devx-track-java
ms.openlocfilehash: f8420a42b87481b5e51b383ace1ba8a510d2a6a7
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87366745"
---
# <a name="tutorial-start-the-immersive-reader-using-the-android-java-code-sample"></a>Tutorial: Inicio del Lector inmersivo mediante el ejemplo de código Java de Android

En la [introducción](./overview.md), se indica lo que es el Lector inmersivo y cómo implementa técnicas demostradas para mejorar la comprensión lectora de nuevos lectores, alumnos de idiomas y alumnos con diferencias de aprendizaje. En este tutorial se explica cómo crear una aplicación Android que inicie el Lector inmersivo. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar y ejecutar una aplicación para Android con un proyecto de ejemplo.
> * Obtener un token de acceso.
> * Iniciar el Lector inmersivo con contenido de ejemplo.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* Un recurso del Lector inmersivo configurado para la autenticación de Azure Active Directory. Siga [estas instrucciones](./how-to-create-immersive-reader.md) para realizar la configuración. Necesitará algunos de los valores que se crean aquí cuando configure las propiedades del entorno. Guarde la salida de la sesión en un archivo de texto para futuras referencias.
* [Git](https://git-scm.com/).
* [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk).
* [Android Studio](https://developer.android.com/studio).

## <a name="configure-authentication-credentials"></a>Configuración de las credenciales de autenticación

1. Inicie Android Studio y abra el proyecto desde el directorio **immersive-reader-sdk/js/samples/quickstart-java-android** (Java) o **immersive-reader-sdk/js/samples/quickstart-kotlin** (Kotlin).

1. Cree un archivo denominado **env** dentro de la carpeta **/assets**. Agregue los siguientes nombres y valores y proporcione los valores según corresponda. No confirme este archivo en el control de código fuente, ya que contiene secretos que no deben hacerse públicos.
    
    ```text
    TENANT_ID=<YOUR_TENANT_ID>
    CLIENT_ID=<YOUR_CLIENT_ID>
    CLIENT_SECRET=<YOUR_CLIENT_SECRET>
    SUBDOMAIN=<YOUR_SUBDOMAIN>
    ```

## <a name="start-the-immersive-reader-with-sample-content"></a>Inicio del Lector inmersivo con contenido de ejemplo

Elija un emulador de dispositivos de AVD Manager y ejecute el proyecto.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y [Referencia del SDK del Lector inmersivo](./reference.md).
* Puede ver ejemplos de código en [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/).