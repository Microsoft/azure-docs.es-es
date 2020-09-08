---
title: Inicio rápido para aprender a usar Azure App Configuration
description: En este inicio rápido se crea una aplicación de Java Spring con Azure App Configuration para centralizar el almacenamiento y la administración de la configuración de la aplicación de forma independiente del código.
services: azure-app-configuration
documentationcenter: ''
author: lisaguthrie
manager: maiye
editor: ''
ms.service: azure-app-configuration
ms.topic: quickstart
ms.date: 04/18/2020
ms.custom: devx-track-java
ms.author: lcozzens
ms.openlocfilehash: 10b38d5c5802ec4f6bc33b15b3ddef12e8256a78
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88586460"
---
# <a name="quickstart-create-a-java-spring-app-with-azure-app-configuration"></a>Inicio rápido: Creación de una aplicación de Java Spring con Azure App Configuration

En este inicio rápido incorporará Azure App Configuration a una aplicación de Java Spring para centralizar el almacenamiento y la administración de la configuración de la aplicación de forma independiente del código.

## <a name="prerequisites"></a>Prerrequisitos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- Un [kit de desarrollo de Java (JDK)](https://docs.microsoft.com/java/azure/jdk) admitido, versión 8.
- [Apache Maven](https://maven.apache.org/download.cgi), versión 3.0 o posterior.

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. Seleccione **Explorador de configuración** >  **+ Crear** > **Clave-valor** para agregar los siguientes pares clave-valor:

    | Clave | Value |
    |---|---|
    | /application/config.message | Hola |

    Deje **Etiqueta** y **Tipo de contenido** en blanco, por ahora.

7. Seleccione **Aplicar**.

## <a name="create-a-spring-boot-app"></a>Creación de una aplicación Spring Boot

Para crear un proyecto de Spring Boot, use [Spring Initializr](https://start.spring.io/).

1. Vaya a <https://start.spring.io/>.

1. Especifique las opciones siguientes:

   - Genere un proyecto de **Maven** con **Java**.
   - Especifique una versión de **Spring Boot** igual o superior a la 2.0.
   - Especifique los nombres de **Group** (Grupo) y **Artifact** (Artefacto) de la aplicación.
   - Adición de la dependencia de **Spring Web**

1. Después de especificar las opciones anteriores, seleccione **Generar proyecto**. Cuando se le solicite, descargue el proyecto en una ruta de acceso del equipo local.

## <a name="connect-to-an-app-configuration-store"></a>Conexión a un almacén de App Configuration

1. Después de extraer los archivos en el sistema local, la aplicación sencilla de Spring Boot estará lista para editarla. Busque el archivo *pom.xml* en el directorio raíz de la aplicación.

1. Abra el archivo *pom.xml* en un editor de texto y agregue el iniciador Spring Cloud Azure Config a la lista `<dependencies>`:

    **Spring Cloud 1.1.x**

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-azure-appconfiguration-config</artifactId>
        <version>1.1.5</version>
    </dependency>
    ```

    **Spring Cloud 1.2.x**

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-azure-appconfiguration-config</artifactId>
        <version>1.2.7</version>
    </dependency>
    ```

1. Cree un archivo Java llamado *MessageProperties.javaº* en el directorio del paquete de la aplicación. Agregue las siguientes líneas:

    ```java
    package com.example.demo;

    import org.springframework.boot.context.properties.ConfigurationProperties;

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

1. Cree un archivo Java nuevo llamado *HelloController.java* en el directorio del paquete de la aplicación. Agregue las siguientes líneas:

    ```java
    package com.example.demo;

    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;

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

1. Abra el archivo de Java de la aplicación principal y agregue `@EnableConfigurationProperties` para habilitar esta característica.

    ```java
    import org.springframework.boot.context.properties.EnableConfigurationProperties;

    @SpringBootApplication
    @EnableConfigurationProperties(MessageProperties.class)
    public class DemoApplication {
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
    ```

1. Cree un nuevo archivo denominado `bootstrap.properties` bajo el directorio de recursos de la aplicación y agréguele las siguientes líneas. Reemplace los valores de ejemplo por las propiedades adecuadas para su almacén de App Configuration.

    ```CLI
    spring.cloud.azure.appconfiguration.stores[0].connection-string= ${APP_CONFIGURATION_CONNECTION_STRING}
    ```

1. Establezca una variable de entorno llamada **APP_CONFIGURATION_CONNECTION_STRING** y defínala como la clave de acceso a su almacén de App Configuration. En la línea de comandos, ejecute el siguiente comando y reinicie el símbolo del sistema para que se aplique el cambio:

    ```cmd
    setx APP_CONFIGURATION_CONNECTION_STRING "connection-string-of-your-app-configuration-store"
    ```

    Si usa Windows PowerShell, ejecute el siguiente comando:

    ```azurepowershell
    $Env:APP_CONFIGURATION_CONNECTION_STRING = "connection-string-of-your-app-configuration-store"
    ```

    Si usa macOS o Linux, ejecute el siguiente comando:

    ```cmd
    export APP_CONFIGURATION_CONNECTION_STRING='connection-string-of-your-app-configuration-store'
    ```

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Compile la aplicación de Spring Boot con Maven y ejecútela; por ejemplo:

    ```cmd
    mvn clean package
    mvn spring-boot:run
    ```

2. Una vez que se está ejecutando la aplicación, puede usar *curl* para probarla, por ejemplo:

      ```cmd
      curl -X GET http://localhost:8080/
      ```

    Verá el mensaje que escribió en el almacén de App Configuration.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de App Configuration y lo ha usado con una aplicación de Java Spring. Para más información, consulte [Spring en Azure](https://docs.microsoft.com/java/azure/spring-framework/). Para aprender a habilitar la aplicación de Java Spring y actualizar dinámicamente la configuración, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Habilitación de la configuración dinámica](./enable-dynamic-configuration-java-spring-app.md)
