---
title: ¿Qué es Azure App Configuration?
description: Lea la información general del servicio Azure App Configuration. Sepa por qué le conviene usar App Configuration y aprenda a utilizarlo.
author: lisaguthrie
ms.author: lcozzens
ms.service: azure-app-configuration
ms.topic: overview
ms.date: 02/19/2020
ms.openlocfilehash: f38b8de37e84cd8a65c5a5caadf8d3b6b71ce455
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88587079"
---
# <a name="what-is-azure-app-configuration"></a>¿Qué es Azure App Configuration?

Azure App Configuration proporciona un servicio para administrar la configuración de la aplicación y las marcas de características de forma centralizada. Los programas actuales, especialmente los que se ejecutan en una nube, tienen por lo general muchos componentes que están distribuidos por naturaleza. La propagación de valores de configuración entre estos componentes puede conducir a errores difíciles de solucionar durante la implementación de una aplicación. Use App Configuration para almacenar toda la configuración de la aplicación y proteger sus accesos en un solo lugar.

## <a name="why-use-app-configuration"></a>Razones para usar App Configuration

Las aplicaciones basadas en la nube a menudo se ejecutan en varias máquinas virtuales o contenedores de varias regiones y emplean varios servicios externos. La creación de una aplicación sólida y escalable en un entorno distribuido supone un desafío importante.

Varias metodologías de programación ayudan a los desarrolladores a abordar el aumento en la complejidad de la creación de estas aplicaciones. Por ejemplo, en [The Twelve-Factor App](https://12factor.net/) se describen muchos patrones arquitectónicos bien comprobados y procedimientos recomendados para usarlos con aplicaciones en la nube. Una de las recomendaciones principales de esta guía es separar la configuración del código. La configuración de una aplicación debería mantenerse de forma externa a su archivo ejecutable y leerse desde su entorno de ejecución o desde un origen externo.

Aunque todas las aplicaciones pueden usar App Configuration, los siguientes ejemplos son los tipos de aplicaciones que mejor aprovechan su uso:

* Microservicios basados en Azure Kubernetes Service, Azure Service Fabric u otras aplicaciones en contenedores implementadas en una o varias ubicaciones geográficas
* Aplicaciones sin servidor, incluida Azure Functions u otras aplicaciones de proceso sin estado controladas por eventos
* Canalización de implementación continua

App Configuration ofrece las siguientes ventajas:

* Un servicio totalmente administrado que puede configurar en cuestión de minutos
* Representaciones y asignaciones de claves flexibles
* Etiquetado con etiquetas
* Reproducción de la configuración en un momento dado
* Interfaz de usuario dedicada para la administración de marcas de características
* Comparación de dos conjuntos de configuraciones en dimensiones definidas de forma personalizada
* Seguridad mejorada mediante las identidades administradas de Azure
* Cifrado de información confidencial en reposo y en tránsito
* Integración nativa con plataformas conocidas

App Configuration complementa [Azure Key Vault](https://azure.microsoft.com/services/key-vault/), que se usa para almacenar secretos de aplicación. App Configuration facilita la implementación de los siguientes escenarios:

* La administración centralizada y la distribución de datos de configuración jerárquicos para diferentes entornos y regiones geográficas
* Los cambios de configuración dinámica sin necesidad de volver a implementar o reiniciar una aplicación
* El control de la disponibilidad de las características en tiempo real

## <a name="use-app-configuration"></a>Uso de App Configuration

La forma más fácil de agregar un almacén de App Configuration a la aplicación es mediante una biblioteca de cliente que Microsoft proporciona. Los siguientes métodos están disponibles para conectarse con la aplicación según el lenguaje y el marco elegidos

| Lenguaje y plataforma de programación | Cómo conectarse |
|---|---|
| .NET Core y ASP.NET Core | Proveedor de App Configuration para .NET Core |
| .NET Framework y ASP.NET | Generador de App Configuration para .NET |
| Java Spring | Cliente de App Configuration para Spring Cloud |
| Otros | API REST de App Configuration |

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido de ASP.NET Core](./quickstart-aspnet-core-app.md)
* [Inicio rápido de .NET Core](./quickstart-dotnet-core-app.md)
* [Inicio rápido de .NET Framework](./quickstart-dotnet-app.md)
* [Inicio rápido de Azure Functions](./quickstart-azure-functions-csharp.md)
* [Inicio rápido de Java Spring](./quickstart-java-spring-app.md)
* [Inicio rápido sobre las marcas de características de ASP.NET Core](./quickstart-feature-flag-aspnet-core.md)
* [Inicio rápido de la de marca de características de Spring Boot](./quickstart-feature-flag-spring-boot.md)
