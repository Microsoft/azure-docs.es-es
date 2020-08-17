---
title: Tutorial para utilizar marcas de características en la aplicación de Spring Boot en Azure App Configuration | Microsoft Docs
description: En este tutorial, aprenderá a implementar marcas de características en aplicaciones de Spring Boot.
services: azure-app-configuration
documentationcenter: ''
author: mrm9084
manager: zhenlan
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: java
ms.topic: tutorial
ms.date: 09/26/2019
ms.author: mametcal
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 83c437cb613e3dad04dee17f0f67040532066c3b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326603"
---
# <a name="tutorial-use-feature-flags-in-a-spring-boot-app"></a>Tutorial: Uso de marcas de características en una aplicación de Spring Boot

Las bibliotecas de administración de características de Spring Boot proporcionan soporte para implementar marcas de características en una aplicación de Spring Boot. Estas bibliotecas le permiten agregar mediante declaración marcas de características al código.

Las bibliotecas de administración de características también administran los ciclos de vida de las marcas de características en segundo plano. Por ejemplo, actualizan y almacenan en caché los estados de marca y garantizan que un estado de marca sea inmutable durante una llamada de solicitud. Además, la biblioteca de Spring Boot ofrece integraciones, como acciones de controlador de MVC, rutas y middleware.

[Add feature flags to a Spring Boot app Quickstart](./quickstart-feature-flag-spring-boot.md) (Inicio rápido: Agregar marcas de características a una aplicación de Spring Boot) muestra varias maneras de agregar las marcas de características en una aplicación de Spring Boot. Este tutorial explica estos métodos en profundidad.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar marcas de características en las partes clave de la aplicación para controlar la disponibilidad de características.
> * Integrar con App Configuration cuando se usa para administrar las marcas de características.

## <a name="set-up-feature-management"></a>Configuración de la administración de características

El administrador de características de Spring Boot `FeatureManager` obtiene las marcas de características del sistema de configuración nativo de la plataforma. Como resultado, puede definir marcas de características de la aplicación con cualquier origen de configuración que admita Spring Boot, incluido el archivo local *bootstrap.yml* o las variables de entorno. `FeatureManager` se basa en la inserción de dependencias. Puede registrar los servicios de administración de características mediante las convenciones estándar:

```java
private FeatureManager featureManager;

public HelloController(FeatureManager featureManager) {
    this.featureManager = featureManager;
}
```

Es recomendable que mantenga las marcas de características fuera de la aplicación y las administre por separado. De este modo, podrá modificar los estados de las marcas en cualquier momento y hacer que estos cambios surtan efecto en la aplicación de inmediato. App Configuration proporciona un lugar centralizado para organizar y controlar todas las marcas de características mediante la interfaz de usuario de un portal dedicado. App Configuration proporciona también las marcas a la aplicación directamente mediante sus bibliotecas cliente de Spring Boot.

La manera más fácil de conectar la aplicación de Spring Boot con App Configuration es mediante el proveedor de configuración:

### <a name="spring-cloud-11x"></a>Spring Cloud 1.1.x

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>spring-cloud-azure-feature-management-web</artifactId>
    <version>1.1.2</version>
</dependency>
```

### <a name="spring-cloud-12x"></a>Spring Cloud 1.2.x

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>spring-cloud-azure-feature-management-web</artifactId>
    <version>1.2.2</version>
</dependency>
```

## <a name="feature-flag-declaration"></a>Declaración de la marca de características

Cada marca de características tiene dos partes: un nombre y una lista de uno o varios filtros que se utilizan para evaluar si el estado de la característica está *activo* (es decir, cuando su valor es `True`). Un filtro define un caso de uso en el que debe activarse una característica.

Cuando una marca de características tiene varios filtros, la lista de filtros se recorre en orden hasta que uno de los filtros determina que se debe habilitar la característica. En ese momento, la marca de característica está *activa* y se omiten los resultados del filtro restantes. Si ningún filtro indica que se debe habilitar la característica, la marca de características está *desactivada*.

El administrador de características admite *application.yml* como origen de configuración para las marcas de características. El ejemplo siguiente muestra cómo establecer las marcas de características en un archivo YAML:

```yml
feature-management:
  feature-set:
    feature-a: true
    feature-b: false
    feature-c:
      enabled-for:
        -
          name: Percentage
          parameters:
            value: 50
```

Por convención, la sección `feature-management` de este documento YAML se usa para la configuración de la marca de características. El ejemplo anterior muestra tres marcas de características con los filtros definidos en la propiedad `EnabledFor`:

* `feature-a` está *activada*.
* `feature-b` está *desactivada*.
* `feature-c` especifica un filtro denominado `Percentage` con una propiedad `parameters`. `Percentage` es un filtro configurable. En este ejemplo, `Percentage` especifica una probabilidad del 50 por ciento de que la marca `feature-c` esté *activada*.

## <a name="feature-flag-checks"></a>Comprobaciones de las marcas de características

El patrón básico de administración de características consiste en comprobar primero si una marca de características está *activada*. Si es así, el administrador de características ejecutará las acciones que contiene la característica. Por ejemplo:

```java
private FeatureManager featureManager;
...
if (featureManager.isEnabledAsync("feature-a").block()) {
    // Run the following code
}
```

## <a name="dependency-injection"></a>Inserción de dependencia

En Spring Boot, puede acceder al administrador de características `FeatureManager` mediante la inserción de dependencias:

```java
@Controller
@ConfigurationProperties("controller")
public class HomeController {
    private FeatureManager featureManager;

    public HomeController(FeatureManager featureManager) {
        this.featureManager = featureManager;
    }
}
```

## <a name="controller-actions"></a>Acciones de controlador

En los controladores MVC, puede usar un atributo `@FeatureGate` para controlar si se habilita una acción específica. La siguiente acción `Index` requiere que `feature-a` esté *activada* para ejecutarse:

```java
@GetMapping("/")
@FeatureGate(feature = "feature-a")
public String index(Model model) {
    ...
}
```

Cuando una acción o controlador MVC está bloqueado porque la marca de característica de control está *desactivada*, se llama a una interfaz `IDisabledFeaturesHandler` registrada. La interfaz predeterminada `IDisabledFeaturesHandler` devuelve un código de estado 404 al cliente sin cuerpo de respuesta.

## <a name="mvc-filters"></a>Filtros MVC

Los filtros MVC se pueden configurar de manera que se activen en función del estado de una marca de características. El código siguiente agrega un filtro MVC denominado `FeatureFlagFilter`. Este filtro se desencadena en la canalización de MVC solo si está habilitada `feature-a`.

```java
@Component
public class FeatureFlagFilter implements Filter {

    @Autowired
    private FeatureManager featureManager;

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        if(!featureManager.isEnabledAsync("feature-a").block()) {
            chain.doFilter(request, response);
            return;
        }
        ...
        chain.doFilter(request, response);
    }
}
```

## <a name="routes"></a>Rutas

Puede usar marcas de características para redirigir rutas. El siguiente código que redirigirá a un usuario de `feature-a` está habilitado:

```java
@GetMapping("/redirect")
@FeatureGate(feature = "feature-a", fallback = "/getOldFeature")
public String getNewFeature() {
    // Some New Code
}

@GetMapping("/getOldFeature")
public String getOldFeature() {
    // Some New Code
}
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a implementar marcas de características en la aplicación de Spring Boot mediante el uso de bibliotecas `spring-cloud-azure-feature-management-web`. Consulte los siguientes recursos para más información sobre la compatibilidad de la administración de características en Spring Boot y App Configuration:

* [Spring Boot feature flag sample code](/azure/azure-app-configuration/quickstart-feature-flag-spring-boot) (Código de ejemplo de marca de características de Spring Boot)
* [Administración de marcas de características](./manage-feature-flags.md)
