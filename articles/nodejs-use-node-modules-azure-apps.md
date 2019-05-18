---
title: Trabajo con módulos Node.js
description: Obtenga información sobre cómo trabajar con módulos Node.js al usar Azure App Service o Cloud Services.
services: ''
documentationcenter: nodejs
author: rloutlaw
manager: rloutlaw
editor: ''
ms.assetid: c0e6cd3d-932d-433e-b72d-e513e23b4eb6
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 08/17/2016
ms.author: rloutlaw
ms.openlocfilehash: 67ae9eec4e6cd4fc7697e54dfd74c5e0c8f1f59c
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540946"
---
# <a name="using-nodejs-modules-with-azure-applications"></a>Uso de módulos Node.js con aplicaciones de Azure
Este documento le proporciona orientación sobre el uso de módulos Node.js con aplicaciones hospedadas en Azure. Mediante este documento podrá asegurarse de que su aplicación use una versión específica de un módulo y de que use módulos nativos con Azure.

Si ya está familiarizado con el uso de módulos Node.js y los archivos **package.json** y **npm-shrinkwrap.json**, la siguiente información muestra un resumen rápido del contenido de este artículo:

* Azure App Service entiende los archivos **package.json** y **npm-shrinkwrap.json** y puede instalar módulos basados en las entradas de estos archivos.

* Azure Cloud Services espera que todos los módulos se instalen en el entorno de desarrollo y que el directorio **node\_modules** se incluya como parte del paquete de implementación. Es posible ofrecer compatibilidad para la instalación de módulos con los archivos **package.json** o **npm-shrinkwrap.json** en Cloud Services. Sin embargo, esta configuración requiere la personalización de los scripts predeterminados que usan los proyectos de Cloud Services. Para obtener un ejemplo de la configuración de este entorno, consulte [Tarea de inicio de Azure para ejecutar la instalación de npm y evitar la implementación de módulos de nodos](https://github.com/woloski/nodeonazure-blog/blob/master/articles/startup-task-to-run-npm-in-azure.markdown)

> [!NOTE]
> No se tratan Azure Virtual Machines en este artículo, ya que la experiencia de implementación en una máquina virtual depende del sistema operativo hospedado por la máquina virtual.
> 
> 

## <a name="nodejs-modules"></a>Módulos Node.js
Los módulos son paquetes JavaScript que pueden cargarse y que proporcionan una funcionalidad específica para su aplicación. Los módulos se instalan normalmente con la herramienta de la línea de comandos **npm**. Sin embargo, algunos módulos (como el módulo http) se proporcionan como parte del paquete Node.js principal.

Cuando se instalan los módulos, se almacenan en el directorio **node\_modules** en la raíz de la estructura del directorio de la aplicación. Cada uno de los módulos dentro del directorio **node\_modules** mantiene su propio directorio que contiene los módulos de los que depende, y este comportamiento se repite para cada módulo hasta el final de la cadena de dependencia. Este entorno permite que los módulos instalados puedan disponer de los requisitos de la propia versión para los módulos de los que depende. Sin embargo, esto puede provocar una estructura de directorio bastante amplia.

Cuando se implementa el directorio **node\_modules** como parte de su aplicación, aumentará el tamaño de la implementación en comparación con el uso del archivo **package.json** o **npm-shrinkwrap.json**. Sin embargo, esto garantiza que la versión de los módulos usados en producción sea la misma que la de los usados en desarrollo.

### <a name="native-modules"></a>Módulos nativos
Mientras que la mayoría de los módulos son archivos JavaScript sin formato, algunos módulos son imágenes binarias específicas de plataforma. Estos módulos se compilan en el momento de la instalación, normalmente mediante Python y node-gyp. Puesto que Cloud Services de Azure se basan en la carpeta **node\_modules** que se implementa como parte de la aplicación, los módulos nativos incluidos como parte de los módulos instalados deben funcionar en un servicio en la nube siempre que se haya instalado y compilado en un sistema de desarrollo de Windows.

Azure App Service no es compatible con todos los módulos nativos y podría presentar errores al compilar módulos con requisitos previos específicos. Aunque algunos módulos conocidos, como MongoDB, tienen dependencias nativas opcionales y funcionan correctamente sin ellas, dos soluciones alternativas han demostrado funcionar correctamente con casi todos los módulos nativos actualmente disponibles:

* Ejecute **npm install** en una máquina con Windows que tenga instalados todos los requisitos previos del módulo nativo. A continuación, implemente la carpeta **node\_modules** creada como parte de la aplicación en Azure App Service.

  * Antes de compilar, compruebe que la instalación local de Node.js tiene arquitectura coincidente y la versión sea lo más cercana posible a la utilizada en Azure (los valores actuales se pueden comprobar en tiempo de ejecución de las propiedades **process.arch** y **process.version**).

* Azure App Service se puede configurar para ejecutar scripts de bash o shell personalizados durante la implementación, lo que le proporciona la oportunidad de ejecutar comandos personalizados y configurar de manera precisa la manera en que se ejecuta **npm install** . Para ver un vídeo que muestra cómo configurar dicho entorno, consulte [Scripts de implementación de sitios web personalizados con Kudu](https://azure.microsoft.com/resources/videos/custom-web-site-deployment-scripts-with-kudu/).

### <a name="using-a-packagejson-file"></a>Uso de un archivo package.json

El archivo **package.json** sirve para especificar las dependencias de nivel superior que requiere la aplicación de manera que la plataforma de hospedaje pueda instalar las dependencias evitando la necesidad de incluir la carpeta **node\_modules** como parte de la implementación. Una vez implementada la aplicación, el comando **npm install** se usa para analizar el archivo **package.json** e instalar todas las dependencias enumeradas.

Durante el desarrollo, puede usar los parámetros **--save**, **--save-dev** o **--save-optional** cuando instale los módulos para agregar una entrada para el módulo al archivo **package.json** automáticamente. Para obtener más información, consulte [npm-install](https://docs.npmjs.com/cli/install).

Un posible problema con el archivo **package.json** es que solo especifica la versión de dependencias de nivel superior. Cada módulo instalado puede o no especificar la versión de los módulos de los que depende y, por lo tanto, es posible que acabe con una cadena de dependencia distinta a la que usó en el desarrollo.

> [!NOTE]
> Cuando se implementa en Azure App Service, si el archivo <b>package.json</b> hace referencia a un módulo nativo, verá un error parecido al del siguiente ejemplo al publicar la aplicación con Git:
> 
> npm ERR! module-name@0.6.0 install: 'node-gyp configure build'
> 
> npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1
> 
> 

### <a name="using-a-npm-shrinkwrapjson-file"></a>Uso de un archivo npm-shrinkwrap.json
El archivo **npm-shrinkwrap.json** es un intento de solucionar las limitaciones de la versión del módulo del archivo **package.json**. Mientras que el archivo **package.json** solo incluye versiones para los módulos de nivel superior, el archivo **npm-shrinkwrap.json** contiene los requisitos de la versión para la cadena de dependencia del módulo completa.

Cuando la aplicación esté preparada para producción, puede bloquear los requisitos de la versión y crear un archivo **npm-shrinkwrap.json** mediante el comando **npm shrinkwrap**. Este comando usará las versiones instaladas actualmente en la carpeta **node\_modules** y registrará las versiones en el archivo **npm-shrinkwrap.json**. Una vez implementada la aplicación en el entorno de hospedaje, se usa el comando **npm install** para analizar el archivo **npm-shrinkwrap.json** e instalar todas las dependencias enumeradas. Para más información, consulte [npm-shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap).

> [!NOTE]
> Cuando implemente en Azure App Service, si el archivo <b>npm-shrinkwrap.json</b> hace referencia a un módulo nativo, verá un error parecido al del siguiente ejemplo al publicar la aplicación con Git:
> 
> npm ERR! module-name@0.6.0 install: 'node-gyp configure build'
> 
> npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1
> 
> 

## <a name="next-steps"></a>Pasos siguientes
Ahora que sabe cómo usar los módulos Node.js con Azure, obtenga información sobre cómo [especificar la versión de Node.js](https://github.com/squillace/staging/blob/master/articles/nodejs-specify-node-version-azure-apps.md), [crear e implementar una aplicación web de Node.js](app-service/app-service-web-get-started-nodejs.md) y [usar la interfaz de la línea de comandos de Azure para Mac y Linux](https://azure.microsoft.com/blog/using-windows-azure-with-the-command-line-tools-for-mac-and-linux/).

Para más información, vea el [Centro para desarrolladores de Node.js](/nodejs/azure/).

[specify the Node.js version]: nodejs-specify-node-version-azure-apps.md
[How to use the Azure Command-Line Interface for Mac and Linux]:cli-install-nodejs.md
[Custom Website Deployment Scripts with Kudu]: https://channel9.msdn.com/Shows/Azure-Friday/Custom-Web-Site-Deployment-Scripts-with-Kudu-with-David-Ebbo
