---
title: 'Inicio rápido: Compilación e inserción de imágenes de contenedor de Java en Azure Container Registry mediante Maven y Jib'
description: Compile una aplicación de Java en contenedor e insértela en Azure Container Registry mediante el complemento Jib de Maven.
author: KarlErickson
ms.custom: devx-track-java, devx-track-azurecli
ms.author: karler
ms.topic: quickstart
ms.date: 02/26/2020
ms.openlocfilehash: 42200fe61b0ac73a63b9f8a3317687fac60c5c02
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88078587"
---
# <a name="quickstart-build-and-push-java-container-images-to-azure-container-registry"></a>Inicio rápido: Compilación e inserción de imágenes de contenedor de Java en Azure Container Registry

En este inicio rápido se muestra cómo compilar una aplicación de Java en contenedor y cómo insertarla en Azure Container Registry mediante el complemento Jib de Maven. El uso de Maven y Jib es un ejemplo de uso de las herramientas de desarrollador para interactuar con un registro de contenedor de Azure.

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción de Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta de Azure gratuita](https://azure.microsoft.com/pricing/free-trial).
* La [Interfaz de la línea de comandos (CLI) de Azure](/cli/azure/overview).
* Un kit de desarrollo de Java (JDK) admitido Para más información sobre los JDK disponibles para desarrollar en Azure, consulte <https://aka.ms/azure-jdks>.
* La herramienta de compilación [Maven](http://maven.apache.org) de Apache (versión 3 o posterior).
* Un cliente [Git](https://git-scm.com).
* Un cliente de [Docker](https://www.docker.com).
* El [asistente de credenciales de Docker de ACR](https://github.com/Azure/acr-docker-credential-helper).

## <a name="create-the-spring-boot-on-docker-getting-started-web-app"></a>Creación de la aplicación web Spring Boot on Docker Getting Started

Los siguientes pasos le guían a través de la creación de una aplicación web Spring Boot y la realización de pruebas de la misma de forma local.

1. En el símbolo del sistema, use el siguiente comando para clonar el proyecto de ejemplo [Spring Boot on Docker Getting Started](https://github.com/spring-guides/gs-spring-boot-docker).

   ```bash
   git clone https://github.com/spring-guides/gs-spring-boot-docker.git
   ```

1. Cambie de directorio al del proyecto completado.

   ```bash
   cd gs-spring-boot-docker/complete
   ```

1. Utilice Maven para compilar y ejecutar la aplicación de ejemplo.

   ```bash
   mvn package spring-boot:run
   ```

1. Para probar la aplicación web, vaya a `curl` o use el siguiente comando `http://localhost:8080`:

   ```bash
   curl http://localhost:8080
   ```

Debería ver el mensaje siguiente mostrado: **Hello Docker World**

## <a name="create-an-azure-container-registry-using-the-azure-cli"></a>Creación de una instancia de Azure Container Registry mediante la CLI de Azure

A continuación, creará un grupo de recursos de Azure y el ACR mediante los pasos siguientes:

1. Inicie sesión en la cuenta de Azure mediante el siguiente comando.

   ```azurecli
   az login
   ```

1. Especifique la suscripción de Azure que se usará:

   ```azurecli
   az account set -s <subscription ID>
   ```

1. Cree un grupo de recursos para los recursos de Azure que se usan en este tutorial. En el siguiente comando, asegúrese de reemplazar los marcadores de posición por su propio nombre de recurso y una ubicación como `eastus`.

   ```azurecli
   az group create \
       --name=<your resource group name> \
       --location=<location>
   ```

1. Cree una instancia privada de Azure Container Registry en el grupo de recursos con el siguiente comando: Asegúrese de reemplazar los marcadores de posición por valores reales. El tutorial inserta la aplicación de ejemplo en forma de imagen Docker en este registro en los pasos posteriores.

   ```azurecli
   az acr create \
       --resource-group <your resource group name> \
       --location <location> \
       --name <your registry name> \
       --sku Basic
   ```

## <a name="push-your-app-to-the-container-registry-via-jib"></a>Inserción de una aplicación en el registro de contenedor mediante Jib

Por último, actualizará la configuración del proyecto y usará el símbolo del sistema para compilar e implementar la imagen.

> [!NOTE]
> Para iniciar sesión en el registro de contenedor de Azure que acaba de crear, es necesario que se ejecute el demonio de Docker. Si desea instalar Docker en la máquina, [esta es la documentación oficial de Docker](https://docs.docker.com/install/).

1. Inicie sesión en su instancia de Azure Container Registry desde la CLI de Azure con el siguiente comando. Asegúrese de reemplazar el marcador de posición por su propio nombre de registro.

   ```azurecli
   az configure --defaults acr=<your registry name>
   az acr login
   ```

   El comando `az configure` establece el nombre del registro predeterminado que se usará con los comandos `az acr`.

1. Navegue hasta el directorio de proyecto completado de la aplicación Spring Boot (por ejemplo, "*C:\SpringBoot\gs-spring-boot-docker\complete*"o" */users/robert/SpringBoot/gs-spring-boot-docker/complete*") y abra el archivo *pom.xml* con un editor de texto.

1. Actualice la colección `<properties>` del archivo *pom.xml* con el siguiente XML. Reemplace el marcador de posición por el nombre del registro y agregue una propiedad `<jib-maven-plugin.version>` con el valor `2.2.0`, o una versión más reciente de [jib-maven-plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin).

   ```xml
   <properties>
      <docker.image.prefix><your registry name>.azurecr.io</docker.image.prefix>
      <java.version>1.8</java.version>
      <jib-maven-plugin.version>2.2.0</jib-maven-plugin.version>
   </properties>
   ```

1. Actualice la colección `<plugins>` del archivo *pom.xml* para que el elemento `<plugin>` contenga una entrada de `jib-maven-plugin`, como se muestra en el ejemplo siguiente. Tenga en cuenta que se va a usar una imagen base de Microsoft Container Registry (MCR): `mcr.microsoft.com/java/jdk:8-zulu-alpine`, que contiene un JDK admitido oficialmente para Azure. En el caso de otras imágenes base de MCR con JDK admitidos oficialmente, consulte [JDK de Java SE](https://hub.docker.com/_/microsoft-java-jdk), [JRE de Java SE](https://hub.docker.com/_/microsoft-java-jre), [JRE sin periféricos de Java SE](https://hub.docker.com/_/microsoft-java-jre-headless) y [JDK de Java SE y Maven](https://hub.docker.com/_/microsoft-java-maven).

   ```xml
   <plugin>
     <artifactId>jib-maven-plugin</artifactId>
     <groupId>com.google.cloud.tools</groupId>
     <version>${jib-maven-plugin.version}</version>
     <configuration>
        <from>
            <image>mcr.microsoft.com/java/jdk:8-zulu-alpine</image>
        </from>
        <to>
            <image>${docker.image.prefix}/${project.artifactId}</image>
        </to>
     </configuration>
   </plugin>
   ```

1. Vaya al directorio de proyecto completado de la aplicación de Spring Boot y ejecute el siguiente comando para crear la imagen e insertarla en el registro:

   ```bash
   az acr login && mvn compile jib:build
   ```

> [!NOTE]
>
> Por motivos de seguridad, la credencial creada por `az acr login` es válida solo durante 1 hora. Si recibe un error *401 No autorizado*, puede volver a ejecutar el comando `az acr login -n <your registry name>` para autenticarse de nuevo.

## <a name="verify-your-container-image"></a>Comprobación de la imagen de contenedor

Felicidades. Ahora tiene la aplicación de Java en contenedor compilada en un JDK admitido por Azure insertado en el ACR. Ya puede probar la imagen; para ello, impleméntela en Azure App Service o extráigala en el entorno local con un comando (reemplace el marcador de posición):

```bash
docker pull <your registry name>.azurecr.io/gs-spring-boot-docker
```

## <a name="next-steps"></a>Pasos siguientes

Para ver otras versiones de las imágenes base de Java oficiales admitidas por Microsoft, consulte:

* [JDK de Java SE](https://hub.docker.com/_/microsoft-java-jdk)
* [JRE de Java SE](https://hub.docker.com/_/microsoft-java-jre)
* [JRE sin periféricos de Java SE](https://hub.docker.com/_/microsoft-java-jre-headless)
* [JDK de Java SE y Maven](https://hub.docker.com/_/microsoft-java-maven)

Para más información acerca de Spring y Azure, vaya al centro de documentación de Azure.

> [!div class="nextstepaction"]
> [Spring en Azure](/azure/developer/java/spring-framework)

### <a name="additional-resources"></a>Recursos adicionales

Para obtener más información, consulte los siguientes recursos:

* [Azure para desarrolladores de Java](/azure/java)
* [Working with Azure DevOps and Java](/azure/devops/java) (Trabajo con Azure DevOps y Java)
* [Spring Boot on Docker Getting Started](https://spring.io/guides/gs/spring-boot-docker)
* [Spring Initializr](https://start.spring.io)
* [Implementación de una aplicación de Spring Boot en Azure App Service](/azure/developer/java/spring-framework/deploy-spring-boot-java-app-from-container-registry-using-maven-plugin)
* [Uso de una imagen personalizada de Docker para Web App on Linux de Azure](../app-service/tutorial-custom-container.md)
