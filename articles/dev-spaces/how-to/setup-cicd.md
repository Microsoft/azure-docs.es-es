---
title: Uso de CI/CD con Azure Dev Spaces
services: azure-dev-spaces
author: DrEsteban
ms.author: stevenry
ms.date: 12/17/2018
ms.topic: conceptual
manager: gwallace
description: Aprenda a configurar la integración y la implementación continuas mediante Azure DevOps con Azure Dev Spaces
keywords: Docker, Kubernetes, Azure, AKS, Azure Container Service, contenedores
ms.openlocfilehash: c7b3eba0bea85082dbb4e39d108af9471d5dc45e
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88080273"
---
# <a name="use-cicd-with-azure-dev-spaces"></a>Uso de CI/CD con Azure Dev Spaces

En este artículo se le guía a través de la implementación continua y la integración continua (CI/CD) en Azure Kubernetes Service (AKS) con Dev Spaces habilitado. CI/CD en AKS permite que las actualizaciones de aplicaciones se implementen automáticamente cada vez que se inserta código confirmado en el repositorio de origen. El hecho de usar CI/CD junto con un clúster habilitado para Dev Spaces resulta útil porque puede mantener actualizada una línea base de la aplicación para que el equipo trabaje con ella.

![Diagrama de CI/CD de ejemplo](../media/common/ci-cd-simple.png)

Aunque en este artículo se le guiará por Azure DevOps, son de aplicación los mismos conceptos a sistemas de CI/CD como Jenkins, TeamCity, etc.

## <a name="prerequisites"></a>Requisitos previos
* Clúster de Azure Kubernetes Service (AKS) con Azure Dev Spaces habilitado
* [CLI de Azure Dev Spaces instalada](upgrade-tools.md)
* [Organización de Azure DevOps con un proyecto](/azure/devops/user-guide/sign-up-invite-teammates?view=vsts)
* [Azure Container Registry (ACR)](../../container-registry/container-registry-get-started-azure-cli.md)
    * Detalles disponibles de una [cuenta de administrador](../../container-registry/container-registry-authentication.md#admin-account) de Azure Container Registry
* [Autorización al clúster de AKS para extraer de Azure Container Registry](../../aks/cluster-container-registry-integration.md)

## <a name="download-sample-code"></a>Descarga de código de ejemplo
Por motivos de tiempo, vamos a crear una bifurcación de nuestro repositorio con código de ejemplo de GitHub. Vaya a https://github.com/Azure/dev-spaces y seleccione **Bifurcación**. Una vez completado el proceso de bifurcación, **clone** su versión local bifurcada del repositorio. De forma predeterminada, se extraerá del repositorio la rama _maestra_, pero hemos incluido para ahorrar tiempo algunos cambios en la rama _azds_updates_, que también debería haberse transferido durante la bifurcación. La rama _azds_updates_ contiene actualizaciones que le pedimos que realizara manualmente en las secciones del tutorial de Dev Spaces, así como algunos archivos JSON y YAML creados previamente para simplificar la implementación del sistema de CI/CD. Puede usar un comando como `git checkout -b azds_updates origin/azds_updates` para extraer la rama _azds_updates_ del repositorio local.

## <a name="dev-spaces-setup"></a>Configuración de Dev Spaces
Cree un espacio llamado _dev_ con el comando `azds space select`. El espacio _dev_ usará la canalización de CI/CD para insertar los cambios de código. También se usará para crear _espacios secundarios_ basados en _dev_.

```cmd
azds space select -n dev
```

Cuando se le pida que seleccione un espacio de desarrollo principal, elija _\<none\>_ .

Una vez creado el espacio de desarrollo, deberá determinar el sufijo del host. Use el comando `azds show-context` para mostrar el sufijo del host del controlador de entrada de Azure Dev Spaces.

```cmd
$ azds show-context
Name   ResourceGroup    DevSpace  HostSuffix
-----  ---------------  --------  ----------------------
MyAKS  MyResourceGroup  dev       fedcba098.eus.azds.io
```

En el ejemplo anterior, el sufijo del host es _fedcba098.eus.azds.io_. Este valor se usa más adelante al crear la definición de versión.

El espacio _dev_ siempre contendrá el estado más reciente del repositorio, una línea base, para que los desarrolladores puedan crear _espacios secundarios_ a partir de _dev_ para probar sus cambios aislados en el contexto de la aplicación de mayor tamaño. Este concepto se trata con más detalle en los tutoriales de Dev Spaces.

## <a name="creating-the-build-definition"></a>Creación de la definición de compilación
Abra el proyecto de equipo de Azure DevOps en un explorador web y vaya a la sección _Canalizaciones_. En primer lugar, haga clic en la foto de perfil de la esquina superior derecha del sitio de Azure DevOps, abra el panel de características en versión preliminar y deshabilite la _nueva experiencia de creación de canalizaciones de YAML_:

![Apertura del panel de características en versión preliminar](../media/common/preview-feature-open.png)

La opción para deshabilitar:

![Opción Nueva experiencia de creación de canalizaciones de YAML](../media/common/yaml-pipeline-preview-feature.png)

> [!Note]
> La característica en versión preliminar _Nueva experiencia de creación de canalizaciones de YAML_de Azure DevOps entra en conflicto con la creación de canalizaciones de compilación predefinidas en este momento. Debe deshabilitarla por ahora para implementar nuestra canalización de compilación predefinida.

En la rama _azds_updates_ hemos incluido una sencilla instancia de [YAML de canalización de Azure](/azure/devops/pipelines/yaml-schema?view=vsts&tabs=schema) que define los pasos de compilación necesarios para *mywebapi* y *webfrontend*.

Dependiendo del lenguaje que haya elegido, instancia de YAML de canalización se habrá insertado en una ruta de acceso similar a: `samples/dotnetcore/getting-started/azure-pipelines.dotnetcore.yml`

Para crear una canalización a partir de este archivo:
1. En la página principal del proyecto de DevOps, vaya a Canalizaciones > Compilaciones.
1. Seleccione la opción para crear una **nueva** canalización de compilación.
1. Seleccione **GitHub** como origen, autorice con su cuenta de GitHub si es necesario y, a continuación, seleccione la rama _azds_updates_ a partir de la versión bifurcada del repositorio de la aplicación de ejemplo de _dev-spaces_.
1. Seleccione **Configuración como código** o **YAML** como plantilla.
1. Ahora verá una página de configuración para la canalización de compilación. Como se mencionó anteriormente, vaya a la ruta de acceso específica del lenguaje para la **ruta del archivo YAML** mediante el botón **...** Por ejemplo, `samples/dotnetcore/getting-started/azure-pipelines.dotnet.yml`.
1. Vaya a la pestaña **Variables**.
1. Agregue manualmente _dockerId_ como variable, que es el nombre de usuario de su [cuenta de administrador de Azure Container Registry](../../container-registry/container-registry-authentication.md#admin-account). (Se menciona en los requisitos previos del artículo).
1. Agregue manualmente _dockerPassword_ como variable, que es la contraseña de su [cuenta de administrador de Azure Container Registry](../../container-registry/container-registry-authentication.md#admin-account). No olvide especificar _dockerPassword_ como secreto (seleccionando el icono de candado) por motivos de seguridad.
1. Seleccione **Guardar y poner en cola**.

Ahora tiene una solución de CI que compilará automáticamente *mywebapi* y *webfrontend* para cualquier actualización que se inserte en la rama _azds_updates_ de la bifurcación de GitHub. Para comprobar que las imágenes de Docker se han insertado, vaya a Azure Portal, seleccione su instancia de Azure Container Registry y explore la pestaña **Repositorios**. Pueden pasar varios minutos hasta que las imágenes se compilen y aparezcan en el registro de contenedor.

![Repositorios de Azure Container Registry](../media/common/ci-cd-images-verify.png)

## <a name="creating-the-release-definition"></a>Creación de la definición de versión

1. En la página principal del proyecto de DevOps, vaya a Canalizaciones > Versiones.
1. Si está trabajando en un nuevo proyecto de DevOps que aún no contiene una definición de versión, deberá crear primero una definición de versión vacía para continuar. La opción de importación no se muestra en la interfaz de usuario hasta que tenga una definición de versión existente.
1. En el lado izquierdo, haga clic en el botón **+ Nuevo** y, después, haga clic en **Importar una canalización**.
1. Haga clic en **Examinar** y seleccione `samples/release.json` en el proyecto.
1. Haga clic en **OK**. Observe que el panel de la canalización se ha cargado con la página de edición de la definición de versión. Fíjese también en que hay algunos iconos de advertencia de color rojo que indican los detalles específicos del clúster que todavía se deben configurar.
1. A la izquierda del panel Canalización, haga clic en **Agregar un artefacto**.
1. En la lista desplegable **Origen**, seleccione la canalización de compilación que creó anteriormente.
1. Para la **Versión predeterminada**, elija **Más reciente desde la rama de predeterminada de canalización de compilación con etiquetas**.
1. Deje **Etiquetas** vacío.
1. Establezca el **Alias de origen** en `drop`. Las tareas de la versión predefinidas utilizan el **Alias de origen**, así que debe configurarlo.
1. Haga clic en **Agregar**.
1. Ahora haga clic en el icono de rayo en el origen del artefacto `drop` recién creado, como se muestra a continuación:

    ![Configuración de la implementación continua del artefacto de versión](../media/common/release-artifact-cd-setup.png)
1. Habilite el **desencadenador de implementación continua**.
1. Mantenga el puntero sobre la pestaña **Tareas** junto a **Canalización** y haga clic en _dev_ (desarrollo) para editar las tareas de la fase _dev_ (desarrollo).
1. Compruebe que está activado **Azure Resource Manager** en **Tipo de conexión** y verá los tres controles desplegables resaltados en rojo: ![Configuración de definición de versión](../media/common/release-setup-tasks.png)
1. Seleccione la suscripción de Azure que utiliza con Azure Dev Spaces. Es posible que también tenga que hacer clic en **Autorizar**.
1. Seleccione el grupo de recursos y el clúster que utiliza con Azure Dev Spaces.
1. Haga clic en **Trabajo de agente**.
1. Seleccione **Hosted Ubuntu 1604** en **Grupo de agentes**.
1. Pase el puntero sobre el selector **Tareas** en la parte superior, haga clic en _prod_ (producción) para editar las tareas de la fase _prod_ (producción).
1. Compruebe que está activado **Azure Resource Manager** en **Tipo de conexión** y seleccione la suscripción de Azure, el grupo de recursos y el clúster que usa con Azure Dev Spaces.
1. Haga clic en **Trabajo de agente**.
1. Seleccione **Hosted Ubuntu 1604** en **Grupo de agentes**.
1. Haga clic en la pestaña **Variables** para actualizar las variables para su versión.
1. Actualice el valor de **DevSpacesHostSuffix** en **UPDATE_ME** para su sufijo de host. El sufijo del host se mostró cuando ejecutó el comando `azds show-context` anteriormente.
1. Haga clic en **Guardar** en la esquina superior derecha y, luego, en **Aceptar**.
1. Haga clic en **+ Versión** (junto al botón Guardar) y en **Crear una versión**.
1. En **Artefactos**, compruebe que está seleccionada la última compilación de su canalización de compilación.
1. Haga clic en **Crear**.

Ahora se iniciará un proceso de versiones automatizado, que implementa los gráficos *mywebapi* y *webfrontend* en su clúster de Kubernetes en el espacio de nivel superior _dev_. Puede supervisar el progreso de la versión en el portal web de Azure DevOps:

1. Navegue hasta la sección **Versiones** en **Canalizaciones**.
1. Haga clic en la canalización de versión para la aplicación de ejemplo.
1. Haga clic en el nombre de la versión más reciente.
1. Mantenga el puntero sobre la casilla **dev** en **Fases** y haga clic en **Registros**.

La versión estará lista cuando se hayan completado todas las tareas.

> [!TIP]
> Si se produce un error en la versión con un mensaje de error como *UPGRADE FAILED: timed out waiting for the condition* (ERROR EN LA ACTUALIZACIÓN: se agotó el tiempo de espera para la condición), intente inspeccionar los pods del clúster [mediante el panel de Kubernetes](../../aks/kubernetes-dashboard.md). Si ve que se producen errores al iniciar los pods, con mensajes de error como *Failed to pull image "azdsexample.azurecr.io/mywebapi:122": rpc error: code = Unknown desc = Error response from daemon: Get https:\//azdsexample.azurecr.io/v2/mywebapi/manifests/122: unauthorized: authentication required*, significa que es posible que el clúster no tenga autorización para realizar la extracción de Azure Container Registry. Asegúrese de que ha completado el requisito previo [Autorización al clúster de AKS para extraer de Azure Container Registry](../../aks/cluster-container-registry-integration.md).

Ahora dispone de una canalización de CI/CD completamente automatizada para su bifurcación de GitHub de las aplicaciones de ejemplo de Dev Spaces. Cada vez que confirme e inserte el código, la canalización de compilación compilará e insertará las imágenes *mywebapi* y *webfrontend* en la instancia de ACR personalizada. A continuación, la canalización de versión implementará el gráfico de Helm para cada aplicación en el espacio _dev_ en el clúster habilitado para Dev Spaces.

## <a name="accessing-your-_dev_-services"></a>Acceso a sus servicios de _dev_
Después de la implementación, la versión _dev_ de *webfrontend* puede obtenerse con una dirección URL pública, como: `http://dev.webfrontend.fedcba098.eus.azds.io`. Puede encontrar la dirección URL mediante la ejecución del comando `azds list-uri`: 

```cmd
$ azds list-uris

Uri                                           Status
--------------------------------------------  ---------
http://dev.webfrontend.fedcba098.eus.azds.io  Available
```

## <a name="deploying-to-production"></a>Implementación en producción

Para promover manualmente una versión particular a _prod_ con el sistema de CI/CD creado en este tutorial:
1. Navegue hasta la sección **Versiones** en **Canalizaciones**.
1. Haga clic en la canalización de versión para la aplicación de ejemplo.
1. Haga clic en el nombre de la versión más reciente.
1. Mantenga el puntero sobre la casilla **prod** (producción) en **Fases** y haga clic en **Implementar**.
    ![Promoción a producción](../media/common/prod-promote.png)
1. Mantenga el puntero sobre la casilla **prod** (producción) de nuevo en **Fases** y haga clic en **Registros**.

La versión estará lista cuando se hayan completado todas las tareas.

La fase de _producción_ de la canalización de CI/CD utiliza un equilibrador de carga en lugar del controlador de entrada de Dev Spaces para proporcionar acceso a los servicios de _producción_. Los servicios implementados en la fase de _producción_ son accesibles como direcciones IP en lugar de nombres DNS. En un entorno de producción, puede optar por crear su propio controlador de entrada para hospedar los servicios según su propia configuración de DNS.

Para determinar la dirección IP del servicio webfrontend, haga clic en el paso de **impresión de la dirección IP pública de webfrontend** para expandir la salida del registro. Use la dirección IP que se muestra en el registro de salida para tener acceso a la aplicación **webfrontend**.

```cmd
...
2019-02-25T22:53:02.3237187Z webfrontend can be accessed at http://52.170.231.44
2019-02-25T22:53:02.3320366Z ##[section]Finishing: Print webfrontend public IP
...
```

## <a name="dev-spaces-instrumentation-in-production"></a>Instrumentación de Dev Spaces en producción
Aunque la instrumentación de Dev Spaces se ha desarrollado para _no_ interponerse en el funcionamiento normal de la aplicación, se recomienda ejecutar las cargas de trabajo de producción en un espacio de nombres de Kubernetes que no esté habilitado con Dev Spaces. El uso de este tipo de espacio de nombres de Kubernetes implica que debe crear el espacio de nombres de producción mediante la CLI de `kubectl`, o permitir que el sistema de CI/CD lo cree durante la primera implementación de Helm. _Seleccionar_ o bien crear un espacio con las herramientas de Dev Spaces agregará instrumentación de Dev Spaces a ese espacio de nombres.

Este es un ejemplo de estructura de espacio de nombres que admite el desarrollo de características, el entorno "dev" _y_ producción, todo en un único clúster de Kubernetes:

![Ejemplo de estructura de espacio de nombres](../media/common/cicd-namespaces.png)

> [!Tip]
> Si ya ha creado un espacio de `prod`, y simplemente desea excluirlo de la instrumentación de Dev Spaces (¡sin eliminarlo!), puede hacerlo con el siguiente comando de la CLI de Dev Spaces:
>
> `azds space remove -n prod --no-delete`
>
> Es posible que deba eliminar todos los pods del espacio de nombres de `prod` una vez hecho esto para que puedan volver a crearse sin instrumentación de Dev Spaces.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo funciona Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Funcionamiento de Azure Dev Spaces](../how-dev-spaces-works.md)
