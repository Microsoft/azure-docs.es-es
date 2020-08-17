---
title: Herramientas de desarrollo
titleSuffix: Azure Data Science Virtual Machine
description: Obtenga información sobre las herramientas y los entornos de desarrollo integrado disponibles en Data Science Virtual Machine.
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: 42a595246f45b2c7423aaf77c95f17243cf8813c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87012390"
---
# <a name="development-tools-on-the-azure-data-science-virtual-machine"></a>Herramientas de desarrollo en Azure Data Science Virtual Machine

Data Science Virtual Machine (DSVM) agrupa varias herramientas populares en un entorno de desarrollo integrado (IDE) altamente productivo. A continuación presentamos algunas herramientas que se proporcionan en DSVM.

## <a name="visual-studio-community-edition"></a>Visual Studio Community Edition

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | IDE de uso general      |
| Versiones de DSVM compatibles      | Windows: Visual Studio 2017, Windows 2019: Visual Studio 2019      |
| Usos típicos      | Desarrollo de software    |
| ¿Cómo se configura e instala en DSVM?      | Carga de trabajo de ciencia de datos (herramientas de Python y R), carga de trabajo de Azure (Hadoop, Data Lake), Node.js, herramientas de SQL Server, [Azure Machine Learning para Visual Studio Code](https://github.com/Microsoft/vs-tools-for-ai)    |
| Cómo usarla y ejecutarla      | Acceso directo de escritorio (`C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE\devenv.exe`). De forma gráfica, abra Visual Studio mediante el icono de escritorio o el menú **Inicio**. Busque programas (logotipo de Windows + S), seguido de **Visual Studio**. Desde allí, puede crear proyectos en lenguajes como C#, Python, R o Node.js.   |
| Herramientas relacionadas en DSVM      |     Visual Studio Code, RStudio y Juno  |

> [!NOTE]
> Quizás vea un mensaje que indica que el período de evaluación ha expirado. Escriba las credenciales de su cuenta Microsoft. O cree una nueva cuenta gratuita para acceder a Visual Studio Community.

## <a name="visual-studio-code"></a>Visual Studio Code 

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | IDE de uso general      |
| Versiones de DSVM compatibles      | Windows, Linux     |
| Usos típicos      | Editor de código e integración de Git   |
| Cómo usarla y ejecutarla      | Acceso directo de escritorio (`C:\Program Files (x86)\Microsoft VS Code\Code.exe`) en Windows o acceso directo de escritorio o terminal (`code`) en Linux    |
| Herramientas relacionadas en DSVM      |     Visual Studio Code, RStudio, Juno  |

## <a name="rstudio-desktop"></a>RStudio Desktop

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | IDE de cliente para el lenguaje R   |
| Versiones de DSVM compatibles      | Windows, Linux      |
| Usos típicos      |  Desarrollo de R     |
| Cómo usarla y ejecutarla      | Acceso directo de escritorio (`C:\Program Files\RStudio\bin\rstudio.exe`) en Windows y acceso directo de escritorio (`/usr/bin/rstudio`) en Linux      |
| Herramientas relacionadas en DSVM      |   Visual Studio, Visual Studio Code, Juno      |

## <a name="rstudio-server"></a>RStudio Server

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | IDE de cliente para el lenguaje R   |
| ¿Qué es?   | IDE basada en web para R    |
| Versiones de DSVM compatibles      | Linux      |
| Usos típicos      |  Desarrollo de R     |
| Cómo usarla y ejecutarla      | Habilite el servicio con _systemctl enable rstudio-server_ y, a continuación, inicie el servicio con _systemctl start rstudio-server_. A continuación, inicie sesión en RStudio Server en http:\//your-vm-ip:8787.       |
| Herramientas relacionadas en DSVM      |   Visual Studio 2019, Visual Studio Code, RStudio Desktop      |

## <a name="juno"></a>Juno 

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | IDE de cliente para el lenguaje Julia   |
| Versiones de DSVM compatibles      | Windows, Linux      |
| Usos típicos      |  Desarrollo de Julia     |
| Cómo usarla y ejecutarla      | Acceso directo de escritorio (`C:\JuliaPro-0.5.1.1\Juno.bat`) en Windows y acceso directo de escritorio (`/opt/JuliaPro-VERSION/Juno`) en Linux      |
| Herramientas relacionadas en DSVM      |   Visual Studio 2019, Visual Studio Code, RStudio      |

## <a name="pycharm"></a>Pycharm

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | IDE de cliente para el lenguaje Python    |
| Versiones de DSVM compatibles      | Windows 2019, Linux      |
| Usos típicos      |  Desarrollo de Python     |
| Cómo usarla y ejecutarla      | Acceso directo de escritorio (`C:\Program Files\tk`) en Windows. Acceso directo de escritorio (`/usr/bin/pycharm`) en Linux      |
| Herramientas relacionadas en DSVM      |   Visual Studio 2019, Visual Studio Code, RStudio      |
