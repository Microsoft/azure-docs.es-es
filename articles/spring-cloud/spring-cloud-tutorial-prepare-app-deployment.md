---
title: Preparación de una aplicación Java Spring para su implementación en Azure Spring Cloud
description: Obtenga información sobre cómo preparar una aplicación Java Spring para su implementación en Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 02/03/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: b2ae94da3d9b2dee62bc031c4a32d17b43be00a6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87021280"
---
# <a name="prepare-a-java-spring-application-for-deployment-in-azure-spring-cloud"></a>Preparación de una aplicación Java Spring para su implementación en Azure Spring Cloud

En este tema se muestra cómo preparar una aplicación Java Spring existente para su implementación en Azure Spring Cloud. Si se ha configurado correctamente, Azure Spring Cloud proporciona servicios sólidos para supervisar, escalar y actualizar cualquier aplicación Java Spring Cloud.

En otros ejemplos se explica cómo implementar una aplicación en Azure Spring Cloud cuando se configura el archivo POM. 
* [Iniciar la aplicación mediante Azure Portal](spring-cloud-quickstart-launch-app-portal.md)
* [Iniciar la aplicación mediante la CLI de Azure](spring-cloud-quickstart-launch-app-cli.md)

En este artículo se explican las dependencias necesarias y cómo agregarlas al archivo POM.

## <a name="java-runtime-version"></a>Versión del entorno de ejecución de Java

Las aplicaciones de Spring/Java son las únicas que se pueden ejecutar en Azure Spring Cloud.

Azure Spring Cloud es compatible con Java 8 y Java 11. El entorno de hospedaje contiene la versión más reciente de Azul Zulu OpenJDK para Azure. Para más información sobre Azul Zulu OpenJDK para Azure, consulte [Instalación del JDK para Azure y Azure Stack](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-install).

## <a name="spring-boot-and-spring-cloud-versions"></a>Versiones de Spring Boot y Spring Cloud

Para preparar una aplicación de Spring Boot existente para la implementación en Azure Spring Cloud, incluya las dependencias de Spring Boot y Spring Cloud en el archivo POM de la aplicación, como se muestra en las siguientes secciones.

Azure Spring Cloud solo admite aplicaciones de Spring Boot, ya sea la versión 2.1 o la 2.2. En la tabla siguiente se enumeran las combinaciones admitidas de Spring Boot y Spring Cloud:

Versión de Spring Boot | Versión de Spring Cloud
---|---
2.1 | Greenwich.RELEASE
2.2 | Hoxton.RELEASE
2.3 | Hoxton.SR5

### <a name="dependencies-for-spring-boot-version-21"></a>Dependencias de Spring Boot, versión 2.1

En Spring Boot 2.1, agregue las siguientes dependencias al archivo POM de la aplicación.

```xml
    <!-- Spring Boot dependencies -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.12.RELEASE</version>
    </parent>

    <!-- Spring Cloud dependencies -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.SR4</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

### <a name="dependencies-for-spring-boot-version-22"></a>Dependencias de Spring Boot, versión 2.2

En Spring Boot 2.2, agregue las siguientes dependencias al archivo POM de la aplicación.

```xml
    <!-- Spring Boot dependencies -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
    </parent>

    <!-- Spring Cloud dependencies -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
### <a name="dependencies-for-spring-boot-version-23"></a>Dependencias de Spring Boot, versión 2.3

En Spring Boot 2.3, agregue las siguientes dependencias al archivo POM de la aplicación.

```xml
    <!-- Spring Boot dependencies -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
    </parent>

    <!-- Spring Cloud dependencies -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR5</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
## <a name="azure-spring-cloud-client-dependency"></a>Dependencia de cliente de Azure Spring Cloud

Azure Spring Cloud hospeda y administra componentes de Spring Cloud. Entre los componentes se incluyen Spring Cloud Service Registry y Spring Cloud Config Server. Incluya la biblioteca cliente de Azure Spring Cloud en sus dependencias para permitir la comunicación con su instancia de servicio de Azure Spring Cloud.

En la tabla siguiente se enumeran las versiones correctas de Azure Spring Cloud para la aplicación que usa Spring Boot y Spring Cloud.

Versión de Spring Boot | Versión de Spring Cloud | Versión de Azure Spring Cloud
---|---|---
2.1 | Greenwich.RELEASE | 2.1
2.2 | Hoxton.RELEASE | 2.2
2.3 | Hoxton.SR5 | 2.3

Incluya una de las dependencias siguientes en el archivo pom.xml. Seleccione la dependencia cuya versión de Azure Spring Cloud coincida con la suya.

### <a name="dependency-for-azure-spring-cloud-version-21"></a>Dependencia de Azure Spring Cloud, versión 2.1

En Spring Boot 2.1, agregue la siguiente dependencia al archivo POM de la aplicación.

```xml
<dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-starter-azure-spring-cloud-client</artifactId>
        <version>2.1.2</version>
</dependency>
```

### <a name="dependency-for-azure-spring-cloud-version-22"></a>Dependencia de Azure Spring Cloud, versión 2.2

En Spring Boot 2.2, agregue la siguiente dependencia al archivo POM de la aplicación.

```xml
<dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-starter-azure-spring-cloud-client</artifactId>
        <version>2.2.1</version>
</dependency>
```

En Spring Boot 2.3, agregue la siguiente dependencia al archivo POM de la aplicación.

```xml
<dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-starter-azure-spring-cloud-client</artifactId>
        <version>2.3.0</version>
</dependency>
```

## <a name="other-recommended-dependencies-to-enable-azure-spring-cloud-features"></a>Otras dependencias recomendadas para habilitar las características de Azure Spring Cloud

Para habilitar las características integradas de Azure Spring Cloud desde el registro de servicio a la traza distribuida, debe incluir también las dependencias siguientes en la aplicación. Puede quitar algunas de estas dependencias si no necesita las características correspondientes para las aplicaciones específicas.

### <a name="service-registry"></a>Registro del servicio

Para usar el servicio Azure Service Registry administrado, incluya la dependencia `spring-cloud-starter-netflix-eureka-client` en el archivo pom.xml, como se muestra aquí:

```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
```

El punto de conexión del servidor de Service Registry se inserta automáticamente como variables de entorno con la aplicación. Así, las aplicaciones se pueden registrar por sí solas en el servidor de Service Registry y detectar otros microservicios dependientes.

#### <a name="enablediscoveryclient-annotation"></a>Anotación de EnableDiscoveryClient

Agregue la anotación siguiente al código fuente de la aplicación.
```java
@EnableDiscoveryClient
```
Por ejemplo, vea la aplicación piggymetrics de ejemplos anteriores:
```java
package com.piggymetrics.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableDiscoveryClient
@EnableZuulProxy

public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

### <a name="distributed-configuration"></a>Configuración distribuida

Para habilitar la configuración distribuida, incluya la siguiente dependencia `spring-cloud-config-client` en la sección de dependencias del archivo pom.xml:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```

> [!WARNING]
> No especifique `spring.cloud.config.enabled=false` en la configuración de arranque. De lo contrario, la aplicación dejará de funcionar con Config Server.

### <a name="metrics"></a>Métricas

Incluya la dependencia `spring-boot-starter-actuator` en la sección de dependencias del archivo pom.xml, como se muestra aquí:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

 Se extraen métricas periódicamente de los puntos de conexión JMX. Estas métricas se pueden visualizar mediante Azure Portal.

 > [!WARNING]
 > Especifique `spring.jmx.enabled=true` en la propiedad de configuración. De lo contrario, las métricas no se pueden visualizar en Azure Portal.

### <a name="distributed-tracing"></a>Seguimiento distribuido

Incluya las dependencias `spring-cloud-starter-sleuth` y `spring-cloud-starter-zipkin` en la sección de dependencias del archivo pom.xml:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

 También debe habilitar una instancia de Azure Application Insights para que funcione con su instancia de servicio de Azure Spring Cloud. Lea la [documentación sobre el seguimiento distribuido](spring-cloud-tutorial-distributed-tracing.md) para aprender a usar Application Insights con Azure Spring Cloud.

## <a name="see-also"></a>Consulte también
* [Análisis de los registros y las métricas de la aplicación](https://docs.microsoft.com/azure/spring-cloud/diagnostic-services)
* [Configuración de un servidor de configuración](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-tutorial-config-server)
* [Uso del seguimiento distribuido con Azure Spring Cloud](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-tutorial-distributed-tracing)
* [Inicio rápido de Spring](https://spring.io/quickstart)
* [Documentación de Spring Boot](https://spring.io/projects/spring-boot)

## <a name="next-steps"></a>Pasos siguientes

En este tema ha aprendido a configurar una aplicación Java Spring para su implementación en Azure Spring Cloud. Para aprender a configurar una instancia de Config Server, consulte el artículo [Configuración de una instancia de Config Server](spring-cloud-tutorial-config-server.md).

Hay más ejemplos disponibles en GitHub: [Ejemplos de Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples).
