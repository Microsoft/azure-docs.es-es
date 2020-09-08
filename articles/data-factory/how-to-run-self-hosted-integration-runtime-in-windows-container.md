---
title: Ejecución de un entorno de ejecución de integración autohospedado en un contenedor de Windows
description: Aprenda a ejecutar un entorno de ejecución de integración autohospedado en un contenedor de Windows.
services: data-factory
ms.author: abnarain
author: nabhishek
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/05/2020
ms.openlocfilehash: d6f292ff89a70de90e6b86f19f73de26963d997f
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927538"
---
# <a name="how-to-run-self-hosted-integration-runtime-in-windows-container"></a>Ejecución de un entorno de ejecución de integración autohospedado en un contenedor de Windows

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se explica cómo ejecutar un entorno de ejecución de integración autohospedado en un contenedor de Windows.
Azure Data Factory proporciona la compatibilidad de contenedor oficial de Windows con los entornos de ejecución de integración autohospedados. Puede descargar el código fuente de compilación de Docker y combinar el proceso de compilación y ejecución en la canalización de entrega continua propia. 

## <a name="prerequisites"></a>Prerrequisitos 
- [Requisitos de los contenedores de Windows](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/system-requirements)
- Docker, versión 2.3 y posterior 
- Entorno de ejecución de integración autohospedado, versión 4.11.7512.1 y posterior 
## <a name="get-started"></a>Primeros pasos 
1.  Instale Docker y habilite el contenedor de Windows 
2.  Descargue el código fuente desde https://github.com/Azure/Azure-Data-Factory-Integration-Runtime-in-Windows-Container
3.  Descargue la versión más reciente de SHIR en la carpeta "SHIR" 
4.  Abra la carpeta en el shell: 
```console
cd "yourFolderPath"
```

5.  Compile la imagen de Docker de Windows: 
```console
docker build . -t "yourDockerImageName" 
```
6.  Ejecute el contenedor de Docker: 
```console
docker run -d -e NODE_NAME="irNodeName" -e AUTH_KEY="IR_AUTHENTICATION_KEY" -e ENABLE_HA=true HA_PORT=8060 "yourDockerImageName"    
```
> [!NOTE]
> AUTH_KEY es obligatorio para este comando. NODE_NAME, ENABLE_HA y HA_PORT son opcionales. Si no establece el valor, el comando usa los valores predeterminados. El valor predeterminado de ENABLE_HA es false y el de HA_PORT es 8060.

## <a name="container-health-check"></a>Comprobación de estado del contenedor 
Después del período de inicio de 120 segundos, el comprobador de estado se ejecuta periódicamente cada 30 segundos. Proporciona el estado de mantenimiento de IR al motor del contenedor. 

## <a name="limitations"></a>Limitaciones
Actualmente no se admiten las características siguientes cuando se ejecuta el entorno de ejecución de integración autohospedado en el contenedor de Windows:
- Proxy HTTP 
- Comunicación de nodo a nodo cifrada con certificado TLS/SSL 
- Generación e importación de copias de seguridad 
- Servicio de demonio 
- Actualización automática 

### <a name="next-steps"></a>Pasos siguientes
- Revise los [conceptos del entorno de ejecución de integración en Azure Data Factory](https://docs.microsoft.com/azure/data-factory/concepts-integration-runtime).
- Aprenda a [crear un entorno de ejecución de integración autohospedado en Azure Portal](https://docs.microsoft.com/azure/data-factory/create-self-hosted-integration-runtime).


