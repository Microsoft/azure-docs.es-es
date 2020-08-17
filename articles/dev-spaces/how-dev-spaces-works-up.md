---
title: Cómo funciona la ejecución del código con Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Describe los procesos de ejecución del código en Azure Kubernetes Service con Azure Dev Spaces
keywords: azds.yaml, Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores
ms.openlocfilehash: c343c32f0817cc922784bb25283290dc9ed88d29
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072956"
---
# <a name="how-running-your-code-with-azure-dev-spaces-works"></a>Cómo funciona la ejecución del código con Azure Dev Spaces

Azure Dev Spaces proporciona varias maneras de iterar y depurar rápidamente las aplicaciones de Kubernetes y colaborar con su equipo en un clúster de Azure Kubernetes Service (AKS). Una vez que [el proyecto está preparado para ejecutarse en un espacio de desarrollo][how-it-works-prep], puede usar Dev Spaces para compilar y ejecutar el proyecto en el clúster de AKS.

En este artículo se describe lo que ocurre cuando se ejecuta el código en AKS con Dev Spaces.

## <a name="run-your-code"></a>Ejecución del código

Para ejecutar el código en un espacio de desarrollo, emita el comando `up` en el mismo directorio que el archivo `azds.yaml`:

```cmd
azds up
```

El comando `up` carga los archivos de origen de la aplicación y otros artefactos necesarios para compilar y ejecutar el proyecto en el espacio de desarrollo. Desde aquí, el controlador en el espacio de desarrollo hace lo siguiente:

1. Crea los objetos de Kubernetes para implementar la aplicación.
1. Compila el contenedor para la aplicación.
1. Implementa la aplicación en el espacio de desarrollo.
1. Crea un nombre DNS accesible públicamente para el punto de conexión de la aplicación, si se ha configurado.
1. Usa el *reenvío de puerto* para proporcionar acceso al punto de conexión de la aplicación mediante http://localhost.
1. Reenvía stdout y stderr a las herramientas de cliente.


## <a name="starting-a-service"></a>Inicio de un servicio

Cuando se inicia un servicio en un espacio de desarrollo, las herramientas de cliente y el controlador trabajan conjuntamente para sincronizar los archivos de origen, crear el contenedor y los objetos de Kubernetes y ejecutar la aplicación.

En un nivel más pormenorizado, esto es lo que sucede al ejecutar `azds up`:

1. [Los archivos se sincronizan][sync-section] desde el equipo del usuario en un almacenamiento de archivos de Azure exclusivo del clúster de AKS del usuario. Se carga el código fuente, el gráfico de Helm y los archivos de configuración.
1. El controlador crea una solicitud para iniciar una nueva sesión. Esta solicitud contiene varias propiedades, incluido un identificador único, el nombre del espacio, la ruta de acceso al código fuente y una marca de depuración.
1. El controlador reemplaza el marcador de posición *$(tag)* en el gráfico de Helm por el identificador de sesión único e instala el gráfico de Helm para el servicio. Si agrega una referencia al identificador de sesión único en el gráfico de Helm, el contenedor implementado en el clúster de AKS para esta sesión específica podrá asociarse a la solicitud de sesión y a la información relacionada.
1. Durante la instalación del gráfico de Helm, el servidor de admisión de webhook de Kubernetes agrega contenedores adicionales al pod de la aplicación para la instrumentación y el acceso al código fuente del proyecto. Los contenedores devspaces-proxy y devspaces-proxy-init se agregan para proporcionar seguimiento de HTTP y enrutamiento de espacio. El contenedor devspaces-build se agrega para proporcionar al pod acceso a la instancia de Docker y al código fuente del proyecto para crear el contenedor de la aplicación.
1. Cuando se inicia el pod de la aplicación, se usan los contenedores devspaces-build y devspaces-proxy-init para crear el contenedor de la aplicación. Después, se inician el contenedor de la aplicación y los contenedores devspaces-proxy.
1. Una vez que se ha iniciado el contenedor de la aplicación, la funcionalidad del lado cliente usa la funcionalidad de *reenvío de puerto* de Kubernetes para proporcionar acceso HTTP a la aplicación a través de http://localhost. El reenvío de puerto conecta el equipo de desarrollo con el servicio en el espacio de desarrollo.
1. Cuando se hayan iniciado todos los contenedores del pod, el servicio se ejecutará. En este momento, la funcionalidad del lado cliente empieza a transmitir en secuencias el seguimiento HTTP, stdout y stderr. Esta información se muestra al desarrollador a través de la funcionalidad del lado cliente.

## <a name="updating-a-running-service"></a>Actualización de un servicio en ejecución

Mientras se ejecuta un servicio, Azure Dev Spaces puede actualizarlo si se modifica alguno de los archivos de origen del proyecto. Además, Dev Spaces lleva a cabo la actualización del servicio de manera diferente en función del tipo de archivo que se haya modificado. Dev Spaces puede actualizar un servicio en ejecución de tres maneras:

* Mediante la actualización de un archivo directamente
* Mediante la recompilación y el reinicio del proceso de la aplicación dentro del contenedor de la aplicación en ejecución
* Mediante la recompilación y la reimplementación del contenedor de la aplicación

![Sincronización de archivos de Azure Dev Spaces](media/how-dev-spaces-works/file-sync.svg)

Algunos archivos del proyecto que son recursos estáticos (como los archivos html, css y cshtml) se pueden actualizar directamente en el contenedor de la aplicación sin necesidad de reiniciar nada. Si se modifica un recurso estático, el nuevo archivo se sincroniza con el espacio de desarrollo y, después, lo usa el contenedor en ejecución.

Para aplicar cambios en archivos como los de código fuente o los de configuración de la aplicación, reinicie el proceso de la aplicación dentro del contenedor en ejecución. Una vez que estos archivos se hayan sincronizado, el proceso de la aplicación se reiniciará en el contenedor en ejecución mediante el proceso *devhostagent*. Al crear inicialmente el contenedor de la aplicación, el controlador reemplaza el comando de inicio de la aplicación por un proceso diferente denominado *devhostagent*. Después, el proceso real de la aplicación se ejecuta como un proceso secundario bajo *devhostagent* y su salida se canaliza mediante la salida de *devhostagent*. El proceso *devhostagent* también forma parte de Dev Spaces y puede ejecutar comandos en el contenedor en ejecución en nombre de Dev Spaces. Al realizar un reinicio, *devhostagent*:

* Detiene el proceso actual o los procesos asociados a la aplicación.
* Recompila la aplicación.
* Reinicia los procesos asociados a la aplicación.

La manera en que *devhostagent* ejecuta los pasos anteriores [se establece en el archivo de configuración `azds.yaml`][azds-yaml-section].

Las actualizaciones de archivos del proyecto como los archivos Dockerfile, los archivos csproj o cualquier parte del gráfico de Helm requieren que se recompile y se reimplemente el contenedor de la aplicación. Cuando uno de estos archivos se sincroniza con el espacio de desarrollo, el controlador ejecuta el comando [helm upgrade][helm-upgrade] y el contenedor de la aplicación se recompila y reimplementa.

## <a name="file-synchronization"></a>Sincronización de archivos

La primera vez que se inicia una aplicación en un espacio de desarrollo, se cargan todos los archivos de código fuente de la aplicación. Durante la ejecución de la aplicación y en los reinicios posteriores, solo se cargan los archivos que se han modificado. Para coordinar este proceso se usan dos archivos: un archivo de cliente y un archivo de controlador.

El archivo de cliente se almacena en un directorio temporal y se le asigna un nombre en función de un hash del directorio del proyecto que se ejecuta en Dev Spaces. Por ejemplo, en Windows tendría un archivo como *Users\USERNAME\AppData\Local\Temp\1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef.synclog* para el proyecto. En Linux, el archivo de cliente se almacena en el directorio */tmp*. Para encontrar el directorio en macOS, ejecute el comando `echo $TMPDIR`.

Este archivo está en formato JSON y contiene lo siguiente:

* Una entrada para cada archivo de proyecto que se sincroniza con el espacio de desarrollo.
* Un identificador de sincronización.
* La marca de tiempo de la última operación de sincronización.

Cada entrada de archivo de proyecto contiene una ruta de acceso al archivo y su marca de tiempo.

El archivo de controlador se almacena en el clúster de AKS. Contiene el identificador de sincronización y la marca de tiempo de la última sincronización.

Se produce una sincronización cuando las marcas de tiempo de sincronización no coinciden entre los archivos de cliente y de controlador. Durante una sincronización, las herramientas de cliente iteran las entradas de archivos en el archivo de cliente. Si la marca de tiempo del archivo es posterior a la marca de tiempo de sincronización, el archivo se sincroniza con el espacio de desarrollo. Una vez completada la sincronización, las marcas de tiempo de sincronización se actualizan en los archivos de cliente y de controlador.

Todos los archivos del proyecto se sincronizan si el archivo de cliente no está presente. Este comportamiento le permite forzar una sincronización completa si elimina el archivo de cliente.

## <a name="how-running-your-code-is-configured"></a>Cómo se configura la ejecución del código

Azure Dev Spaces usa el archivo `azds.yaml` para instalar y configurar el servicio. El controlador usa la propiedad `install` en el archivo `azds.yaml` para instalar el gráfico de Helm y crear los objetos de Kubernetes:

```yaml
...
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
...
```

De forma predeterminada, el comando `prep` genera el gráfico de Helm. También establece la propiedad *install.chart* en el directorio del gráfico de Helm. Si quiere usar un gráfico de Helm en una ubicación diferente, puede actualizar esta propiedad para usar esa ubicación.

Al instalar los gráficos de Helm, Azure Dev Spaces proporciona una forma de reemplazar los valores del gráfico. Los valores predeterminados del gráfico de Helm se encuentran en `charts/APP_NAME/values.yaml`.

Con la propiedad *install.values*, puede enumerar uno o varios archivos que definen los valores que quiere que se reemplacen en el gráfico de Helm. Por ejemplo, si quiere un nombre de host o una configuración de base de datos en concreto al ejecutar la aplicación en un espacio de desarrollo, puede usar esta funcionalidad de reemplazo. También puede agregar el símbolo *?* al final de cualquiera de los nombres de archivo para establecerlo como opcional.

La propiedad *install.set* le permite configurar uno o varios valores que quiera reemplazar en el gráfico de Helm. Los valores configurados en *install.set* reemplazarán los valores configurados en los archivos que se enumeran en *install.values*. Las propiedades de *install.set* dependen de los valores del gráfico de Helm y pueden variar según el gráfico generado.

En el ejemplo anterior, la propiedad *install.set.replicaCount* indica al controlador cuántas instancias de la aplicación deben ejecutarse en el espacio de desarrollo. Según el escenario, puede aumentar este valor, pero esto tendrá unas consecuencias a la hora de asociar un depurador al pod de la aplicación. Para obtener más información, vea el [artículo sobre la solución de problemas][troubleshooting].

En el gráfico de Helm generado, la imagen de contenedor está establecida en *{{ .Values.image.repository }}:{{ .Values.image.tag }}* . El archivo `azds.yaml` define la propiedad *install.set.image.tag* como *$(tag)* de forma predeterminada, que se usa como valor para *{{ .Values.image.tag }}* . Al establecer la propiedad *install.set.image.tag* de este modo, permite que la imagen de contenedor de la aplicación se etiquete de manera diferente cuando se ejecuta Azure Dev Spaces. En este caso concreto, la imagen se etiqueta como *\<value from image.repository>:$(tag)* . Debe usar la variable *$(tag)* como valor de *install.set.image.tag* para que Dev Spaces reconozca y localice el contenedor en el clúster de AKS.

En el ejemplo anterior, `azds.yaml` define *install.set.ingress.hosts*. La propiedad *install.set.ingress.hosts* define un formato de nombre de host para los puntos de conexión públicos. Esta propiedad también usa *$(spacePrefix)* , *$(rootSpacePrefix)* y *$(hostSuffix)* , que son valores proporcionados por el controlador.

*$(spacePrefix)* es el nombre del espacio de desarrollo secundario, que adopta la forma *SPACENAME.s*. *$(rootSpacePrefix)* es el nombre del espacio primario. Por ejemplo, si *azureuser* es un espacio secundario de *default*, el valor de *$(rootSpacePrefix)* es *default* y el valor de *$(spacePrefix)* es *azureuser.s*. Si el espacio no es de tipo secundario, *$(spacePrefix)* está en blanco. Por ejemplo, si el espacio *default* no tiene ningún espacio primario, el valor de *$(rootSpacePrefix)* es *default* y el valor de *$(spacePrefix)* está en blanco. *$(hostSuffix)* es un sufijo DNS que apunta al controlador de entrada de Azure Dev Spaces que se ejecuta en el clúster de AKS. Este sufijo DNS corresponde a una entrada DNS comodín, por ejemplo, *\*.RANDOM_VALUE.eus.azds.IO*, que se creó al agregar el controlador de Azure Dev Spaces al clúster de AKS.

En el archivo `azds.yaml` anterior, también podría actualizar *install.set.ingress.hosts* para cambiar el nombre de host de la aplicación. Por ejemplo, si quiere simplificar el nombre de host de la aplicación de *$(spacePrefix)$(rootSpacePrefix)webfrontend$(hostSuffix)* a *$(spacePrefix)$(rootSpacePrefix)web$(hostSuffix)* .

Para crear el contenedor para la aplicación, el controlador usa las siguientes secciones del archivo de configuración `azds.yaml`:

```yaml
build:
  context: .
  dockerfile: Dockerfile
...
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
      useGitIgnore: true
      args:
        BUILD_CONFIGURATION: ${BUILD_CONFIGURATION:-Debug}
...
```

El controlador usa un archivo Dockerfile para compilar y ejecutar la aplicación.

La propiedad *build.context* indica el directorio donde se encuentran los archivos Dockerfile. La propiedad *build.dockerfile* define el nombre del archivo Dockerfile para compilar la versión de producción de la aplicación. La propiedad *configurations.develop.build.dockerfile* configura el nombre del archivo Dockerfile para la versión de desarrollo de la aplicación.

El hecho de tener diferentes archivos Dockerfile para el desarrollo y la producción permite habilitar ciertos elementos durante el desarrollo y deshabilitarlos para las implementaciones de producción. Por ejemplo, puede habilitar la depuración o el registro más detallado durante el desarrollo y deshabilitarlo en un entorno de producción. También puede actualizar estas propiedades si los archivos Dockerfile tienen nombres diferentes o se encuentran en una ubicación diferente.

Para ayudarle a iterar rápidamente durante el desarrollo, Azure Dev Spaces sincronizará los cambios del proyecto local y actualizará de forma incremental la aplicación. Para configurar la sincronización y la actualización, se usa la siguiente sección del archivo de configuración `azds.yaml`:

```yaml
...
configurations:
  develop:
    ...
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
...
```

Los archivos y los directorios que sincronizarán los cambios se indican en la propiedad *configurations.develop.container.sync*. Estos directorios se sincronizan inicialmente cuando se ejecuta el comando `up` y, además, cuando se detectan cambios. Si le interesa sincronizar otros directorios con su espacio de desarrollo, puede cambiar esta propiedad.

La propiedad *configurations.develop.container.iterate.buildCommands* especifica cómo se compila la aplicación en un escenario de desarrollo. La propiedad *configurations.develop.container.command* proporciona el comando para ejecutar la aplicación en un escenario de desarrollo. Tal vez le interese actualizar alguna de estas propiedades si quiere usar durante el desarrollo marcas o parámetros adicionales de compilación o tiempo de ejecución.

La propiedad *configurations.develop.container.iterate.processesToKill* enumera los procesos que se van a terminar para detener la aplicación. Es posible que quiera actualizar esta propiedad si le interesa cambiar el comportamiento de reinicio de la aplicación durante el desarrollo. Por ejemplo, si ha actualizado las propiedades *configurations.develop.container.iterate.buildCommands* o *configurations.develop.container.command* para cambiar cómo se compila o se inicia la aplicación, es posible que necesite cambiar qué procesos se detienen.

Cuando prepare el código con el comando `azds prep`, tendrá la opción de agregar la marca `--enable-ingress`. Al agregar la marca `--enable-ingress`, se crea una dirección URL accesible públicamente para la aplicación. Si omite esta marca, la aplicación solo será accesible dentro del clúster o mediante el túnel de localhost. Después de ejecutar el comando `azds prep`, puede cambiar esta opción si modifica la propiedad *ingress.enabled* en `charts/APPNAME/values.yaml`:

```yaml
ingress:
  enabled: true
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las redes y cómo se enrutan las solicitudes en Azure Dev Spaces, consulte [Funcionamiento del enrutamiento con Azure Dev Spaces][how-it-works-routing].

Para más información sobre el uso de Azure Dev Spaces para iterar y desarrollar rápidamente, consulte [Funcionamiento de Proceso local con Kubernetes][how-it-works-local-process-kubernetes] y [Funcionamiento de la depuración remota del código con Azure Dev Spaces][how-it-works-remote-debugging].

Para empezar a trabajar con Azure Dev Spaces ejecutando su proyecto, consulte los siguientes inicios rápidos:

* [Iteración y depuración rápidas con Visual Studio Code y Java][quickstart-java]
* [Iteración y depuración rápidas con Visual Studio Code y .NET][quickstart-netcore]
* [Iteración y depuración rápidas con Visual Studio Code y Node.js][quickstart-node]
* [Iteración y depuración rápidas con Visual Studio y .NET Core][quickstart-vs]
* [Uso de la CLI para desarrollar una aplicación en Kubernetes][quickstart-cli]


[azds-yaml-section]: #how-running-your-code-is-configured
[helm-upgrade]: https://helm.sh/docs/intro/using_helm/#helm-upgrade-and-helm-rollback-upgrading-a-release-and-recovering-on-failure
[how-it-works-local-process-kubernetes]: /visualstudio/containers/overview-local-process-kubernetes
[how-it-works-prep]: how-dev-spaces-works-prep.md
[how-it-works-remote-debugging]: how-dev-spaces-works-remote-debugging.md
[how-it-works-routing]: how-dev-spaces-works-routing.md
[quickstart-cli]: quickstart-cli.md
[quickstart-java]: quickstart-java.md
[quickstart-netcore]: quickstart-netcore.md
[quickstart-node]: quickstart-nodejs.md
[quickstart-vs]: quickstart-netcore-visualstudio.md
[sync-section]: #file-synchronization
[troubleshooting]: troubleshooting.md