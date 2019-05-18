---
title: Inicio rápido para aprender a usar Azure App Configuration | Microsoft Docs
description: Una artículo de inicio rápido para usar Azure App Configuration con aplicaciones de Java Spring.
services: azure-app-configuration
documentationcenter: ''
author: yidon
manager: jeffya
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: java
ms.topic: quickstart
ms.tgt_pltfrm: Spring
ms.workload: tbd
ms.date: 01/08/2019
ms.author: yidon
ms.openlocfilehash: 9137262ffc7c172e6a99920a7abb31459a8703fe
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408577"
---
# <a name="quickstart-create-a-java-spring-app-with-app-configuration"></a>Inicio rápido: Creación de una aplicación de Java Spring con App Configuration

Azure App Configuration es un servicio de configuración administrado de Azure. Puede usarlo para almacenar y administrar fácilmente toda la configuración de una aplicación en un solo lugar que esté separado del código. En este manual de inicio rápido se muestra cómo incorporar el servicio en una aplicación de Java Spring.

Para realizar los pasos de este inicio rápido, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción disponible en las plataformas Windows, macOS y Linux.

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, instale una instancia de [Java Development Kit (JDK)](https://aka.ms/azure-jdks) compatible con la versión 8 y [Apache Maven](https://maven.apache.org/) con la versión 3.0 o posterior.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de configuración de aplicaciones

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. Seleccione **Explorador de configuración** > **+ Crear** para agregar los siguientes pares clave-valor:

    | Clave | Valor |
    |---|---|
    | /application/config.message | Hola |

    Deje **Etiqueta** y **Tipo de contenido** en blanco, por ahora.

## <a name="create-a-spring-boot-app"></a>Creación de una aplicación Spring Boot

Para crear un proyecto de Spring Boot, se usa [Spring Initializr](https://start.spring.io/).

1. Vaya a <https://start.spring.io/>.

2. Especifique las opciones siguientes:

   * Genere un proyecto de **Maven** con **Java**.
   * Especifique una versión de **Spring Boot** igual o superior a la 2.0.
   * Especifique los nombres de **Group** (Grupo) y **Artifact** (Artefacto) de la aplicación.
   * Agregue la dependencia **Web**.

3. Después de especificar las opciones anteriores, seleccione **Generar proyecto**. Cuando se le solicite, descargue el proyecto en una ruta de acceso del equipo local.

## <a name="connect-to-an-app-configuration-store"></a>Conexión a un almacén de configuración de aplicaciones

1. Después de extraer los archivos en el sistema local, la aplicación sencilla de Spring Boot estará lista para editarla. Busque el archivo *pom.xml* en el directorio raíz de la aplicación.

2. Abra el archivo *pom.xml* en un editor de texto y agregue el iniciador Spring Cloud Azure Config a la lista `<dependencies>`:

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-starter-azure-appconfiguration-config</artifactId>
        <version>1.1.0.M3</version>
    </dependency>
    ```

3. Cree un archivo Java llamado *MessageProperties.javaº* en el directorio del paquete de la aplicación. Agregue las siguientes líneas:

    ```java
    @ConfigurationProperties(prefix = "config")
    public class MessageProperties {
        private String message;

        public String getMessage() {
            return message;
        }

        public void setMessage(String message) {
            this.message = message;
        }
    }
    ```

4. Cree un archivo Java nuevo llamado *HelloController.java* en el directorio del paquete de la aplicación. Agregue las siguientes líneas:

    ```java
    @RestController
    public class HelloController {
        private final MessageProperties properties;

        public HelloController(MessageProperties properties) {
            this.properties = properties;
        }

        @GetMapping
        public String getMessage() {
            return "Message: " + properties.getMessage();
        }
    }
    ```

5. Abra el archivo de Java de la aplicación principal y agregue `@EnableConfigurationProperties` para habilitar esta característica.

    ```java
    @SpringBootApplication
    @EnableConfigurationProperties(MessageProperties.class)
    public class AzureConfigApplication {
        public static void main(String[] args) {
            SpringApplication.run(AzureConfigApplication.class, args);
        }
    }
    ```

6. Cree un nuevo archivo denominado `bootstrap.properties` bajo el directorio de recursos de la aplicación y agréguele las siguientes líneas. Reemplace los valores de muestra con las propiedades adecuadas para su almacén de configuración de la aplicación.

    ```properties
    spring.cloud.azure.appconfiguration.stores[0].connection-string=[your-connection-string]
    ```

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Compile la aplicación de Spring Boot con Maven y ejecútela; por ejemplo:

    ```shell
    mvn clean package
    mvn spring-boot:run
    ```
2. Una vez que se está ejecutando la aplicación, puede usar *curl* para probarla, por ejemplo:

      ```shell
      curl -X GET http://localhost:8080/
      ```
    Verá el mensaje que escribió en el almacén de configuración de aplicaciones.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de configuración de aplicaciones y lo ha usado con una aplicación de Java Spring. Para más información, consulte [Spring en Azure](https://docs.microsoft.com/java/azure/spring-framework/).

Para más información acerca del uso de App Configuration, continúe con el siguiente tutorial, ya que en él se muestra cómo realizar la autenticación.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
