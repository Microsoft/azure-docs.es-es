---
title: Creación de una aplicación contenedor de Linux de Service Fabric en Azure
description: En este inicio rápido, compilará una imagen de Docker con la aplicación, insertará la imagen en un registro de contenedor y luego implementar el contenedor en un clúster de Service Fabric.
ms.topic: quickstart
ms.date: 07/22/2019
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: 652c5b7c2df4e1f8bd379390451886914628505d
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88854982"
---
# <a name="quickstart-deploy-linux-containers-to-service-fabric"></a>Inicio rápido: Implementar contenedores Linux en Service Fabric

Azure Service Fabric es una plataforma de sistemas distribuidos para implementar y administrar microservicios y contenedores escalables y confiables.

Este inicio rápido muestra cómo implementar contenedores Linux en un clúster de Service Fabric en Azure. Una vez completado, tendrá una aplicación de votación que consta de un front-end web de Python y un back-end de Redis en ejecución en un clúster de Service Fabric. También aprenderá cómo conmutar por error una aplicación y cómo escalar una aplicación en el clúster.

![Página web de la aplicación de votación][quickstartpic]

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía de inicio rápido:

1. Cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar, si no tiene una suscripción a Azure.

2. Instale la [CLI de Azure](/cli/azure/install-azure-cli-apt?view=azure-cli-latest).

3. Instale el [SDK de Service Fabric y la CLI](service-fabric-get-started-linux.md#installation-methods).

4. Instalación de [Git](https://git-scm.com/)


## <a name="get-the-application-package"></a>Obtención del paquete de la aplicación

Para implementar contenedores en Service Fabric, necesita un conjunto de archivos de manifiesto (la definición de la aplicación), que describen los contenedores individuales y la aplicación.

En una consola, utilice git para clonar una copia de la definición de aplicación; a continuación, cambie los directorios al directorio `Voting` en el clon.

```bash
git clone https://github.com/Azure-Samples/service-fabric-containers.git

cd service-fabric-containers/Linux/container-tutorial/Voting
```

## <a name="create-a-service-fabric-cluster"></a>Creación de un clúster de Service Fabric

Para implementar la aplicación en Azure, se necesita un clúster de Service Fabric que ejecute la aplicación. Los siguientes comandos crean un clúster de cinco nodos en Azure.  Los comandos también crean un certificado autofirmado, lo agregan a un almacén de claves y los descargan en el sitio local. El nuevo certificado se usa para proteger el clúster al implementarlo y también para autenticar a los clientes.

```azurecli
#!/bin/bash

# Variables
ResourceGroupName='containertestcluster' 
ClusterName='containertestcluster' 
Location='eastus' 
Password='q6D7nN%6ck@6' 
Subject='containertestcluster.eastus.cloudapp.azure.com' 
VaultName='containertestvault' 
VmPassword='Mypa$$word!321'
VmUserName='sfadminuser'

# Login to Azure and set the subscription
az login

az account set --subscription <mySubscriptionID>

# Create resource group
az group create --name $ResourceGroupName --location $Location 

# Create secure five node Linux cluster. Creates a key vault in a resource group
# and creates a certficate in the key vault. The certificate's subject name must match 
# the domain that you use to access the Service Fabric cluster.  The certificate is downloaded locally.
az sf cluster create --resource-group $ResourceGroupName --location $Location --certificate-output-folder . --certificate-password $Password --certificate-subject-name $Subject --cluster-name $ClusterName --cluster-size 5 --os UbuntuServer1604 --vault-name $VaultName --vault-resource-group $ResourceGroupName --vm-password $VmPassword --vm-user-name $VmUserName
```

> [!Note]
> El servicio front-end web está configurado para escuchar en el puerto 80 el tráfico entrante. De forma predeterminada, el puerto 80 está abierto en las máquinas virtuales del el clúster y en el equilibrador de carga de Azure.
>

## <a name="configure-your-environment"></a>Configurar su entorno

Service Fabric proporciona varias herramientas que puede usar para administrar un clúster y sus aplicaciones:

- Service Fabric Explorer, una herramienta basada en explorador.
- Interfaz de la línea de comandos (CLI) de Service Fabric, que se ejecuta sobre la CLI de Azure. 
- Comandos de PowerShell.

En este inicio rápido, se usan la CLI de Service Fabric y Service Fabric Explorer (una herramienta basada en web). Para usar Service Fabric Explorer, deberá importar el archivo PFX del certificado en el explorador. De forma predeterminada, el archivo PFX no tiene contraseña.

Mozilla Firefox es el explorador predeterminado en Ubuntu 16.04. Para importar el certificado en Firefox, haga clic en el botón de menú en la esquina superior derecha del explorador y, a continuación, haga clic en **Opciones**. En la página **Preferencias**, utilice el cuadro de búsqueda para buscar "certificados". Haga clic en **Ver certificados**, seleccione la pestaña **Sus certificados**, haga clic en **Importar** y siga las indicaciones para importar el certificado.

   ![Instalación del certificado en Firefox](./media/service-fabric-quickstart-containers-linux/install-cert-firefox.png)

## <a name="deploy-the-service-fabric-application"></a>Implementación de la aplicación de Service Fabric

1. Conéctese al clúster de Service Fabric en Azure mediante la CLI. El punto de conexión es el punto de conexión de administración del clúster. Ha creado el archivo PEM en la sección anterior. 

    ```bash
    sfctl cluster select --endpoint https://containertestcluster.eastus.cloudapp.azure.com:19080 --pem containertestcluster22019013100.pem --no-verify
    ```

2. Use el script de instalación para copiar la definición de la aplicación de votación en el clúster, registrar el tipo de aplicación y crear una instancia de la aplicación.  El archivo de certificado PEM debe encontrarse en el mismo directorio que el archivo *install.sh*.

    ```bash
    ./install.sh
    ```

3. Abra un explorador web y navegue hasta el punto de conexión de Service Fabric Explorer del clúster. El punto de conexión tiene el siguiente formato: **https://\<my-azure-service-fabric-cluster-url>: 19080/Explorer**; por ejemplo, `https://containertestcluster.eastus.cloudapp.azure.com:19080/Explorer`. </br>

4. Expanda el nodo **Aplicaciones** para comprobar que ahora hay una entrada para el tipo de aplicación de votación y la instancia que ha creado.

    ![Service Fabric Explorer][sfx]

5. Para conectar con el contenedor en ejecución, abra un explorador web y navegue a la dirección URL del clúster; por ejemplo, `http://containertestcluster.eastus.cloudapp.azure.com:80`. Debería ver la aplicación de votación en el explorador.

    ![Página web de la aplicación de votación][quickstartpic]

> [!NOTE]
> También puede implementar aplicaciones de Service Fabric con Docker Compose. Por ejemplo, puede usar el siguiente comando para implementar e instalar la aplicación en el clúster mediante Docker Compose.
>  ```bash
> sfctl compose create --deployment-name TestApp --file-path ../docker-compose.yml
> ```

## <a name="fail-over-a-container-in-a-cluster"></a>Conmutación por error de un contenedor en un clúster

Service Fabric se asegura de que las instancias de contenedor se mueven automáticamente a otros nodos del clúster en caso de error. También puede descargar manualmente un nodo de contenedores y moverlos correctamente a otros nodos del clúster. Service Fabric proporciona varias formas de escalar los servicios. En los pasos siguientes, se utiliza Service Fabric Explorer.

Para conmutar por error el contenedor de front-end, siga estos pasos:

1. Abra Service Fabric Explorer en el clúster, por ejemplo, `https://containertestcluster.eastus.cloudapp.azure.com:19080/Explorer`.
2. Haga clic en el nodo **fabric:/Voting/azurevotefront** en la vista de árbol y expanda el nodo de partición (representado por un GUID). Observe el nombre de nodo de la vista de árbol, que le muestra los nodos en los que el contenedor se ejecuta actualmente; por ejemplo, `_nodetype_1`.
3. Expanda el nodo **Nodos** en la vista de árbol. Haga clic en el botón de puntos suspensivos (...) situado junto al nodo que ejecuta el contenedor.
4. Elija **Reiniciar** para reiniciar ese nodo y confirmar la acción de reinicio. El reinicio hace que el contenedor conmute por error a otro nodo del clúster.

    ![Vista de nodo en Service Fabric Explorer][sfxquickstartshownodetype]

## <a name="scale-applications-and-services-in-a-cluster"></a>Escalar aplicaciones y servicios en un clúster

Es fácil escalar servicios de Service Fabric en un clúster para adaptarse a la carga de los servicios. Para escalar un servicio, cambie el número de instancias que se ejecutan en el clúster.

Para escalar el servicio front-end web, siga estos pasos:

1. Abra Service Fabric Explorer en el clúster, por ejemplo, `https://containertestcluster.eastus.cloudapp.azure.com:19080`.
2. Haga clic en el botón de puntos suspensivos (...) situado junto al nodo **fabric:/Voting/azurevotefront** en la vista de árbol y seleccione **Scale Service** (Escalar servicio).

    ![Inicio del servicio de escalado de Service Fabric Explorer][containersquickstartscale]

    Ahora puede escalar el número de instancias del servicio front-end web.

3. Cambie el número a **2** y haga clic en **Scale Service** (Escalar servicio).
4. Haga clic en el nodo **fabric:/Voting/azurevotefront** en la vista de árbol y expanda el nodo de partición (representado por un GUID).

    ![Finalización del servicio de escalado de Service Fabric Explorer][containersquickstartscaledone]

    Ahora puede ver que el servicio tiene dos instancias. En la vista de árbol, puede ver en qué nodos se ejecutan las instancias.

Mediante esta sencilla tarea de administración, hemos duplicado los recursos disponibles para el servicio de front-end para procesar la carga de usuarios. Es importante entender que no hacen falta varias instancias de un servicio para que se ejecute de forma confiable. Si se produce un error en un servicio, Service Fabric se asegurará de que se ejecute una nueva instancia de servicio en el clúster.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar la instancia de la aplicación del clúster y anular el registro del tipo de aplicación, utilice el script de desinstalación (uninstall.sh) proporcionado en la plantilla. Este script tarda algún tiempo en limpiar la instancia, por lo que no se debe ejecutar el script de instalación inmediatamente después de este script. Puede usar Service Fabric Explorer para determinar cuándo se han eliminado la instancia y el registro del tipo de aplicación.

```bash
./uninstall.sh
```

La manera más sencilla de eliminar el clúster y todos los recursos que consume es eliminar el grupo de recursos.

Inicie sesión en Azure y seleccione el identificador de suscripción con el que quiere quitar el clúster. Para encontrar el identificador de suscripción, inicie sesión en Azure Portal. Elimine el grupo de recursos y todos los recursos de clúster mediante el comando [az group delete](/cli/azure/group?view=azure-cli-latest).

```azurecli
az login
az account set --subscription <guid>
ResourceGroupName="containertestcluster"
az group delete --name $ResourceGroupName
```

Si ya ha terminado de trabajar con el clúster, puede eliminar el certificado del almacén de certificados. Por ejemplo:
- En Windows: use el [complemento MMC de certificados](/dotnet/framework/wcf/feature-details/how-to-view-certificates-with-the-mmc-snap-in). No olvide seleccionar **Mi cuenta de usuario** al agregar el complemento. Vaya a `Certificates - Current User\Personal\Certificates` y elimine el certificado.
- En Mac: use la aplicación Keychain.
- En Ubuntu: siga los pasos utilizados para ver los certificados y elimine el certificado.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado una aplicación de contenedor de Linux en un clúster de Service Fabric en Azure, ha realizado una conmutación por error de la aplicación y ha escalado la aplicación en el clúster. Para más información sobre cómo trabajar con contenedores de Linux en Service Fabric, siga el tutorial para aplicaciones de contenedor de Linux.

> [!div class="nextstepaction"]
> [Creación de una aplicación contenedor de Linux](./service-fabric-tutorial-create-container-images.md)

[sfx]: ./media/service-fabric-quickstart-containers-linux/containersquickstartappinstance.png
[quickstartpic]: ./media/service-fabric-quickstart-containers-linux/votingapp.png
[sfxquickstartshownodetype]:  ./media/service-fabric-quickstart-containers-linux/containersquickstartrestart.png
[containersquickstartscale]: ./media/service-fabric-quickstart-containers-linux/containersquickstartscale.png
[containersquickstartscaledone]: ./media/service-fabric-quickstart-containers-linux/containersquickstartscaledone.png
