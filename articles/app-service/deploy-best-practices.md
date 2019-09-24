---
title: Procedimientos recomendados de implementación en Azure App Service | Microsoft Docs
description: Obtenga información sobre los componentes clave de la implementación en Azure App Service.
keywords: azure app service, web app, deploy, deployment, pipelines, build
services: app-service
documentationcenter: ''
author: jasonfreeberg
manager: ''
editor: ''
ms.assetid: bb51e565-e462-4c60-929a-2ff90121f41d
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/31/2019
ms.author: jafreebe
ms.custom: ''
ms.openlocfilehash: d1b6444b8512b1b55ac46370e805f8f662f5f555
ms.sourcegitcommit: 82499878a3d2a33a02a751d6e6e3800adbfa8c13
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2019
ms.locfileid: "70070681"
---
# <a name="deployment-best-practices"></a>Procedimientos recomendados de implementación

Cada equipo de desarrollo tiene requisitos únicos que pueden dificultar la implementación de una canalización de implementación eficaz en cualquier servicio en la nube. En este artículo se presentan los tres componentes principales de la implementación de App Service: orígenes de implementación, canalizaciones de compilación y mecanismos de implementación. En este artículo también se tratan algunos procedimientos recomendados y sugerencias para pilas de idiomas específicos.

## <a name="deployment-components"></a>Componentes de implementación

### <a name="deployment-source"></a>Origen de implementación

Un origen de implementación es la ubicación del código de la aplicación. En el caso de las aplicaciones de producción, el origen de implementación suele ser un repositorio hospedado por software de control de versiones como [GitHub, BitBucket o Azure Repos](deploy-continuous-deployment.md). En escenarios de desarrollo y prueba, el origen de implementación puede ser [un proyecto en la máquina local](deploy-local-git.md). App Service también admite las [ carpetas OneDrive y Dropbox](deploy-content-sync.md) como orígenes de implementación. A pesar de que las carpetas en la nube pueden facilitar la introducción a App Service, normalmente no se recomienda usar este origen para las aplicaciones de producción de nivel empresarial. 

### <a name="build-pipeline"></a>Canalización de compilación

Una vez que decida un origen de implementación, el paso siguiente consistirá en elegir una canalización de compilación. Una canalización de compilación lee el código fuente del origen de implementación y ejecuta una serie de pasos (por ejemplo, compilar el código, compactar el código HTML y JavaScript, ejecutar pruebas o empaquetar componentes) para hacer que la aplicación pueda ejecutarse. Los comandos específicos que la canalización de compilación ejecuta dependen de la pila de lenguajes. Estas operaciones se pueden ejecutar en un servidor de compilación como Azure Pipelines o de forma local.

### <a name="deployment-mechanism"></a>Mecanismo de implementación

El mecanismo de implementación es la acción que se usa para colocar la aplicación compilada en el directorio */home/site/wwwroot* de la aplicación Web. El directorio */wwwroot* es una ubicación de almacenamiento montada que todas las instancias de la aplicación Web comparten. Cuando el mecanismo de implementación coloca la aplicación en este directorio, las instancias reciben una notificación para sincronizar los nuevos archivos. App Service admite los siguientes mecanismos de implementación:

- Puntos de conexión de Kudu: [Kudu](https://github.com/projectkudu/kudu/wiki) es la herramienta de productividad para desarrolladores de código abierto que se ejecuta como un proceso independiente en Windows App Service y como un segundo contenedor en Linux App Service. Kudu controla las implementaciones continuas y proporciona puntos de conexión HTTP para la implementación, como zipdeploy.
- FTP y WebDeploy: Con las [credenciales de usuario o de sitio](deploy-configure-credentials.md), puede cargar archivos [a través de FTP ](deploy-ftp.md) o WebDeploy. Estos mecanismos no pasan por Kudu.  

Las herramientas de implementación como Azure Pipelines, Jenkins y los complementos de editor usan uno de estos mecanismos de implementación.

## <a name="language-specific-considerations"></a>Consideraciones específicas del idioma

### <a name="java"></a>Java

Use la API[ zipdeploy/](deploy-zip.md) de Kudu para implementar las aplicaciones JAR y [wardeploy/](deploy-zip.md#deploy-war-file) para las aplicaciones WAR. Si usa Jenkins, puede usar esas API directamente en la fase de implementación. Para obtener más información, consulte [este artículo](../jenkins/execute-cli-jenkins-pipeline.md).

### <a name="node"></a>Node

De forma predeterminada, Kudu ejecuta los pasos de compilación para la aplicación Node (`npm install`). Si usa un servicio de compilación como Azure DevOps, la compilación de Kudu es innecesaria. Para deshabilitarla, cree una opción de aplicación, `SCM_DO_BUILD_DURING_DEPLOYMENT`, con el valor `false`.

### <a name="net"></a>.NET 

De forma predeterminada, Kudu ejecuta los pasos de compilación para la aplicación .Net (`dotnet build`). Si usa un servicio de compilación como Azure DevOps, la compilación de Kudu es innecesaria. Para deshabilitarla, cree una opción de aplicación, `SCM_DO_BUILD_DURING_DEPLOYMENT`, con el valor `false`.

## <a name="other-deployment-considerations"></a>Otras consideraciones de implementación

### <a name="use-deployment-slots"></a>Uso de ranuras de implementación

Siempre que sea posible, use [ranuras de implementación](deploy-staging-slots.md) al implementar una nueva compilación de producción. Al usar un nivel de Plan de App Service estándar o superior, puede implementar la aplicación en un entorno de ensayo, validar los cambios y realizar pruebas de humo. Cuando esté preparado, puede intercambiar las ranuras de ensayo y de producción. La operación de intercambio prepara las instancias de trabajo necesarias para que coincidan con la escala de producción, lo que elimina el tiempo de inactividad. 

### <a name="local-cache"></a>caché local

El contenido de Azure App Service se almacena en Azure Storage y surge de manera duradera como un recurso compartido de contenido. Sin embargo, algunas aplicaciones solo necesitan un almacén de contenido de solo lectura y de gran rendimiento que puedan ejecutar con alta disponibilidad. Estas aplicaciones pueden beneficiarse del uso de la [memoria caché local](overview-local-cache.md). No se recomienda la memoria caché local para los sitios de administración de contenido como WordPress.

Use siempre la memoria caché local junto con [ranuras de implementación] (deploy-staging-slots md) para evitar tiempos de inactividad. Consulte [esta sección](overview-local-cache.md#best-practices-for-using-app-service-local-cache) para información sobre el uso conjunto de estas características.

### <a name="high-cpu-or-memory"></a>CPU o memoria elevadas

Si el Plan de App Service usa más del 90 % de la CPU o la memoria disponibles, es posible que la máquina virtual subyacente tenga problemas para procesar la implementación. Cuando esto suceda, escale de forma temporal el número de instancias para realizar la implementación. Una vez finalizada la implementación, puede devolver el número de instancias a su valor anterior.
