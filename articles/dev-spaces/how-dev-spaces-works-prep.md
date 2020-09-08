---
title: Cómo funciona la preparación de un proyecto para Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Describe cómo funciona la preparación del proyecto con Azure Dev Spaces
keywords: azds.yaml, Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores
ms.openlocfilehash: 61351072494b51d02a1d6c31399208b9e9b54fce
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213419"
---
# <a name="how-preparing-a-project-for-azure-dev-spaces-works"></a>Cómo funciona la preparación de un proyecto para Azure Dev Spaces

Azure Dev Spaces le ofrece varias formas de iterar y depurar rápidamente las aplicaciones Kubernetes y colaborar con su equipo en un clúster de Azure Kubernetes Service (AKS). Dev Spaces puede generar archivos Dockerfile y gráficos de Helm para su proyecto. Dev Spaces también crea y usa un archivo de configuración para implementar, ejecutar y depurar las aplicaciones de Kubernetes en AKS. Todos estos archivos residen con el código de la aplicación y se pueden agregar al sistema de control de versiones.

En este artículo se describe lo que sucede al preparar el proyecto para ejecutarlo en AKS con Dev Spaces.

## <a name="prepare-your-code"></a>Preparación del código

Para ejecutar la aplicación en un espacio de desarrollo, debe estar en un contenedor. Además, es necesario que defina cómo se implementará en Kubernetes. Para incluir la aplicación en un contenedor, necesita un archivo Dockerfile. Para definir cómo se implementará la aplicación en Kubernetes, es necesario un [gráfico de Helm](https://docs.helm.sh/). Con el fin de ayudar a crear el archivo Dockerfile y el gráfico de Helm para la aplicación, las herramientas de cliente proporcionan el comando `prep`:

```cmd
azds prep --enable-ingress
```

El comando `prep` consultará los archivos del proyecto e intentará crear el archivo Dockerfile y el gráfico de Helm para ejecutar la aplicación en Kubernetes. Actualmente, el comando `prep` generará un archivo Dockerfile y un gráfico de Helm con los lenguajes siguientes:

* Java
* Node.js
* .NET Core

Es *necesario* ejecutar el comando `prep` desde un directorio que contenga código fuente. Al ejecutar el comando `prep` desde el directorio correcto, las herramientas de cliente pueden identificar el lenguaje y crear un archivo Dockerfile adecuado para incluir la aplicación en un contenedor. También puede ejecutar el comando `prep` desde un directorio que contenga un archivo *pom.xml* para proyectos de Java.

Si ejecuta el comando `prep` desde un directorio que no contenga código fuente, las herramientas de cliente no generarán un archivo Dockerfile. Además, mostrarán el siguiente mensaje de error: *Dockerfile could not be generated due to unsupported language* (No se pudo generar un archivo Dockerfile debido a un lenguaje no admitido). Este error también se produce si las herramientas de cliente no reconocen el tipo de proyecto.

Al ejecutar el comando `prep`, tendrá la opción de especificar la marca `--enable-ingress`. Esta marca le indica al controlador que cree un punto de conexión accesible desde Internet para este servicio. Si no especifica esta marca, el servicio solo será accesible desde dentro del clúster o mediante el túnel de localhost que crean las herramientas de cliente. Para habilitar o deshabilitar este comportamiento después de ejecutar el comando `prep`, actualice el gráfico de Helm generado.

El comando `prep` no reemplazará ningún archivo Dockerfile o gráfico de Helm que tenga en el proyecto. Si un archivo Dockerfile o gráfico de Helm existente usa la misma convención de nomenclatura que los archivos generados por el comando `prep`, el comando `prep` omitirá la generación de esos archivos. En caso contrario, el comando `prep` generará su propio archivo Dockerfile o gráfico de Helm junto con los archivos existentes.

> [!IMPORTANT]
> Azure Dev Spaces usa Dockerfile y el gráfico de Helm del proyecto para compilar y ejecutar el código, pero el usuario puede modificar estos archivos si quiere cambiar la forma en que se compila y ejecuta el proyecto.

El comando `prep` también generará un archivo `azds.yaml` en la raíz del proyecto. Azure Dev Spaces usa este archivo para compilar, instalar, configurar y ejecutar la aplicación. Este archivo de configuración muestra la ubicación del archivo Dockerfile y del gráfico de Helm y, además, proporciona una configuración adicional encima de estos artefactos.

A continuación se muestra un archivo azds.yaml de ejemplo creado mediante la [aplicación de ejemplo de .NET Core](https://github.com/Azure/dev-spaces/tree/master/samples/dotnetcore/getting-started/webfrontend):

```yaml
kind: helm-release
apiVersion: 1.1
build:
  context: .
  dockerfile: Dockerfile
install:
  chart: charts/webfrontend
  values:
  - values.dev.yaml?
  - secrets.dev.yaml?
  set:
    replicaCount: 1
    image:
      repository: webfrontend
      tag: $(tag)
      pullPolicy: Never
    ingress:
      annotations:
        kubernetes.io/ingress.class: traefik-azds
      hosts:
        # This expands to [space.s.][rootSpace.]webfrontend.<random suffix>.<region>.azds.io
        # Customize the public URL by changing the 'webfrontend' text between the $(rootSpacePrefix) and $(hostSuffix) tokens
        # For more information see https://aka.ms/devspaces/routing
        - $(spacePrefix)$(rootSpacePrefix)webfrontend$(hostSuffix)
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
      useGitIgnore: true
      args:
        BUILD_CONFIGURATION: ${BUILD_CONFIGURATION:-Debug}
    container:
      sync:
      - "**/Pages/**"
      - "**/Views/**"
      - "**/wwwroot/**"
      - "!**/*.{sln,csproj}"
      command: [dotnet, run, --no-restore, --no-build, --no-launch-profile, -c, "${BUILD_CONFIGURATION:-Debug}"]
      iterate:
        processesToKill: [dotnet, vsdbg]
        buildCommands:
        - [dotnet, build, --no-restore, -c, "${BUILD_CONFIGURATION:-Debug}"]
```

El archivo `azds.yaml` generado por el comando `prep` debería funcionar en un escenario de desarrollo simple de un solo proyecto. Si la complejidad del proyecto ha aumentado, es posible que tenga que actualizar este archivo después de ejecutar el comando `prep`. Por ejemplo, el proyecto podría requerir algunos cambios en el proceso de compilación o de inicio, según las necesidades de desarrollo o depuración. También podría tener varias aplicaciones en el proyecto que requieren varios procesos de compilación o un contenido de una compilación diferente.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo ejecutar el código en su espacio de desarrollo, consulte [Cómo funciona la ejecución del código con Azure Dev Spaces][how-it-works-up].

[how-it-works-up]: how-dev-spaces-works-up.md