---
title: Uso de la configuración dinámica en una aplicación de Spring Boot
titleSuffix: Azure App Configuration
description: Aprenda a actualizar dinámicamente los datos de configuración para aplicaciones de Spring Boot
services: azure-app-configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.topic: tutorial
ms.date: 3/5/2020
ms.custom: devx-track-java
ms.author: lcozzens
ms.openlocfilehash: a2864d610d71c6b3a86c131dabb3c0b9ed138bec
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327929"
---
# <a name="tutorial-use-dynamic-configuration-in-a-java-spring-app"></a>Tutorial: Uso de la configuración dinámica en una aplicación de Java Spring

La biblioteca cliente de Spring Boot de App Configuration admite la actualización de un conjunto de valores de configuración a petición sin provocar el reinicio de una aplicación. La biblioteca cliente almacena en caché cada configuración para evitar demasiadas llamadas al almacén de configuración. La operación de actualización no actualiza el valor hasta que haya expirado el valor almacenado en caché, incluso cuando el valor cambia en el almacén de configuración. El tiempo de expiración predeterminado de cada solicitud es de 30 segundos. Si es necesario, se puede invalidar.

Puede comprobar la configuración actualizada a petición llamando al método `refreshConfigurations()` de `AppConfigurationRefresh`.

Como alternativa, puede usar el paquete `spring-cloud-azure-appconfiguration-config-web`, que toma una dependencia de `spring-web` para administrar la actualización automatizada.

## <a name="use-automated-refresh"></a>Uso de la actualización automatizada

Para usar la actualización automatizada, comience con una aplicación de Spring Boot que usa App Configuration, como la aplicación que se crea siguiendo las indicaciones del [inicio rápido de Spring Boot con App Configuration](quickstart-java-spring-app.md).

Abra el archivo *pom.xml* en un editor de texto y agregue `<dependency>` para `spring-cloud-azure-appconfiguration-config-web`.

**Spring Cloud 1.1.x**

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>spring-cloud-azure-appconfiguration-config-web</artifactId>
    <version>1.1.2</version>
</dependency>
```

**Spring Cloud 1.2.x**

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>spring-cloud-azure-appconfiguration-config-web</artifactId>
    <version>1.2.2</version>
</dependency>
```

Guarde el archivo y, a continuación, compile y ejecute la aplicación como de costumbre.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado la aplicación de Spring Boot para actualizar dinámicamente la configuración a partir de App Configuration. Para obtener información sobre cómo usar una identidad administrada de Azure para simplificar el acceso a App Configuration, vaya al siguiente tutorial.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
