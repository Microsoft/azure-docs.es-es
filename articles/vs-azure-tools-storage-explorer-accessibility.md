---
title: Accesibilidad del Explorador de Azure Storage | Microsoft Docs
description: Conozca la accesibilidad en el Explorador de Azure Storage. Revise los lectores de pantalla disponibles, la funcionalidad de zoom, los temas de contraste alto y las teclas de método abreviado.
services: storage
documentationcenter: na
author: MrayermannMSFT
manager: jinglouMSFT
editor: ''
ms.assetid: 1ed0f096-494d-49c4-ab71-f4164ee19ec8
ms.service: storage
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/20/2018
ms.author: marayerm
ms.openlocfilehash: ca4a8d719277eaa1d853d53d282649f839256be9
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88035492"
---
# <a name="storage-explorer-accessibility"></a>Accesibilidad al Explorador de Storage

## <a name="screen-readers"></a>Lectores de pantalla

El Explorador de Storage admite el uso de un lector de pantalla en Windows y Mac. Se recomiendan los lectores de pantalla siguientes para cada plataforma:

Plataforma | Lector de pantalla
---------|--------------
Windows  | NVDA
Mac      | VoiceOver
Linux    | (no se admiten lectores de pantalla en Linux)

Si surge un problema de accesibilidad al utilizar el Explorador de almacenamiento, [abra una incidencia en GitHub](https://github.com/Microsoft/AzureStorageExplorer/issues).

## <a name="zoom"></a>Zoom

Puede ampliar el texto en el Explorador de Storage con el zoom. Para ello, haga clic en **Acercar** en el menú Ayuda. También puede utilizar el menú de ayuda para alejar y restablecer el nivel de zoom al nivel predeterminado.

![Opciones de zoom en el menú Ayuda][0]

La configuración del zoom aumenta el tamaño de la mayoría de los elementos de la interfaz de usuario. Se recomienda también habilitar un texto grande y la configuración del zoom para el sistema operativo para garantizar que todos los elementos de interfaz de usuario se escalen correctamente.

## <a name="high-contrast-themes"></a>Temas de contraste alto

El Explorador de Storage tiene dos temas de contraste alto: **Contraste alto (claro)** y **Contraste alto (oscuro)**. Para cambiar el tema, selecciónelo en el menú Ayuda > Temas.

![Submenú Temas][1]

La configuración de tema cambia el color de la mayoría de los elementos de la interfaz de usuario. Se recomienda también que habilite el tema correspondiente de contraste alto del SO para asegurarse de que todos los elementos de la IU se coloreen adecuadamente.

## <a name="shortcut-keys"></a>Teclas de método abreviado

### <a name="window-commands"></a>Comandos de Windows

Get-Help       | Métodos abreviados de teclado
--------------|--------------------
Nueva ventana    | **Control + Mayús + N**
Cerrar editor  | **Control + F4**
Salga.          | **Control + Mayús + W**

### <a name="navigation-commands"></a>Comandos de navegación

Get-Help                | Métodos abreviados de teclado
-----------------------|----------------------
Enfocar el panel siguiente       | **F6**
Enfocar el panel anterior   | **Mayús+F6**
Explorador               | **Control + Mayús + E**
Administración de cuentas     | **Control + Mayús + A**
Alternar barra lateral        | **Control + B**
Registro de actividad           | **Control + Mayús + L**
Acciones y propiedades | **Control + Mayús + P**
Editor actual         | **Control + Inicio**
Editor siguiente            | **Control + AvPág**
Editor anterior        | **Control + RePág**

### <a name="zoom-commands"></a>Comandos de zoom

Get-Help  | Métodos abreviados de teclado
---------|------------------
Acercar  | **Control + =**
Alejar | **Control + -**

### <a name="blob-and-file-share-editor-commands"></a>Comandos del editor de recursos compartidos de blob y archivo

Get-Help | Métodos abreviados de teclado
--------|--------------------
Atrás    | **Alt+Flecha izquierda**
Adelante | **Alt+Flecha derecha**
Arriba      | **Alt+Flecha arriba**

### <a name="editor-commands"></a>Comandos del Editor

Get-Help | Métodos abreviados de teclado
--------|------------------
Copiar    | **Control+C**
Cortar     | **Control+X**
Pegar   | **Control+V**
Actualizar  | **Control + R**

### <a name="other-commands"></a>Otros comandos

Get-Help                | Métodos abreviados de teclado
-----------------------|------------------
Alternar herramientas de desarrollo | **F12**
Recargar                 | **Alt+Control+R**

[0]: ./media/vs-azure-tools-storage-explorer-accessibility/Zoom.png
[1]: ./media/vs-azure-tools-storage-explorer-accessibility/HighContrast.png
