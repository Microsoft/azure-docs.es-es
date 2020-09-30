---
title: Preguntas frecuentes acerca de Azure Spring Cloud | Microsoft Docs
description: En este artículo se responden las preguntas más frecuentes sobre Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 1947e57a5f200521fe57c6aaf41e9d57c1085592
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90888717"
---
# <a name="azure-spring-cloud-faq"></a>Preguntas frecuentes de Azure Spring Cloud

En este artículo se responden las preguntas más frecuentes sobre Azure Spring Cloud.

## <a name="general"></a>General

### <a name="why-azure-spring-cloud"></a>¿Por qué Azure Spring Cloud?

Azure Spring Cloud proporciona una plataforma como servicio (PaaS) para los desarrolladores de Spring Cloud. Azure Spring Cloud administra la infraestructura de la aplicación, de modo que el usuario puede centrarse en el código de la aplicación y en la lógica de negocios. Las características principales integradas en Azure Spring Cloud incluyen Eureka, Config Server, Service Registry Server, Pivotal Build Service e implementación Blue-Green, entre otras. Este servicio también permite a los desarrolladores enlazar sus aplicaciones con otros servicios de Azure, como Azure Cosmos DB, Azure Database for MySQL y Azure Cache for Redis.

Azure Spring Cloud mejora la experiencia de diagnóstico de aplicaciones para los desarrolladores y operadores mediante la integración de Azure Monitor, Application Insights y Log Analytics.

### <a name="how-secure-is-azure-spring-cloud"></a>¿Es seguro Azure Spring Cloud?

La seguridad y la privacidad se encuentran entre las principales prioridades para los clientes de Azure y de Azure Spring Cloud. Azure permite garantizar que solo los clientes tienen acceso a los datos, registros o configuraciones de la aplicación mediante el cifrado seguro de todos estos datos. 

* Las instancias de servicio de Azure Spring Cloud están aisladas unas de otras.
* Azure Spring Cloud proporciona administración completa de certificados y TLS/SSL.
* Las revisiones de seguridad críticas de los runtimes de OpenJDK y Spring Cloud se aplicarán a Azure Spring Cloud lo antes posible.

### <a name="in-which-regions-is-azure-spring-cloud-available"></a>¿En qué regiones está disponible Azure Spring Cloud?

Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Centro-sur de EE. UU., Oeste de EE. UU.2, Oeste de Europa, Norte de Europa, Sur de Reino Unido, Sudeste Asiático y Este de Australia.


### <a name="what-are-the-known-limitations-of-azure-spring-cloud"></a>¿Cuáles son las limitaciones conocidas de Azure Spring Cloud?

Azure Spring Cloud tiene las limitaciones conocidas siguientes:
    
* `spring.application.name` se sustituirá por el nombre de la aplicación que se usó para crear cada aplicación.
* El valor predeterminado de `server.port` es el puerto 1025. Si se aplica cualquier otro valor, se reemplazará por 1025.
* Azure Portal y las plantillas de Azure Resource Manager no admiten la carga de paquetes de aplicación. Solo puede cargar paquetes de aplicación al implementar la aplicación a través de la CLI de Azure.

### <a name="what-pricing-tiers-are-available"></a>¿Qué planes de tarifa están disponibles? 
¿Cuál debo usar y cuáles son los límites dentro de cada plan?
* Azure Spring Cloud ofrece dos planes de tarifa: Básico y Estándar. El plan Básico está destinado a desarrollo y pruebas y a probar Azure Spring Cloud. El plan Estándar está optimizado para ejecutar el tráfico de producción de uso general. Consulte [Detalle de precios de Azure Spring Cloud](https://azure.microsoft.com/pricing/details/spring-cloud/) para obtener información sobre los límites y la comparación de niveles de las características.

### <a name="how-can-i-provide-feedback-and-report-issues"></a>¿Cómo puedo realizar comentarios y notificar incidencias?

Si encuentra algún problema con Azure Spring Cloud, cree una [solicitud de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request). Para enviar una solicitud de característica o proporcionar comentarios, vaya a [Comentarios de Azure](https://feedback.azure.com/forums/34192--general-feedback).

## <a name="development"></a>Desarrollo

### <a name="i-am-a-spring-cloud-developer-but-new-to-azure-what-is-the-quickest-way-for-me-to-learn-how-to-develop-an-azure-spring-cloud-application"></a>Soy un desarrollador de Spring Cloud, pero nunca he usado Azure. ¿Cuál es la forma más rápida de aprender a desarrollar una aplicación de Azure Spring Cloud?

Para conocer la forma más rápida de empezar a trabajar con Azure Spring Cloud, siga las instrucciones de [Inicio rápido: inicio de una aplicación de Azure Spring Cloud mediante Azure Portal](spring-cloud-quickstart.md).

::: zone pivot="programming-language-java"
### <a name="what-java-runtime-does-azure-spring-cloud-support"></a>¿Qué runtime de Java es compatible con Azure Spring Cloud?

Azure Spring Cloud es compatible con Java 8 y 11. Consulte las [versiones del sistema operativo y el runtime de Java](#java-runtime-and-os-versions).
::: zone-end

### <a name="where-can-i-view-my-spring-cloud-application-logs-and-metrics"></a>¿Dónde puedo ver mis métricas y registros de aplicaciones de Spring Cloud?

Busque las métricas en la pestaña Información general de la aplicación y en la pestaña [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/data-platform-metrics#interacting-with-azure-monitor-metrics).

Azure Spring Cloud admite la exportación de las métricas y los registros de aplicaciones de Spring Cloud a Azure Storage, EventHub y [Log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/data-platform-logs#log-queries). El nombre de la tabla en Log Analytics es *AppPlatformLogsforSpring*. Para obtener más información sobre cómo habilitarlo, consulte [Servicios de diagnóstico](diagnostic-services.md).

### <a name="does-azure-spring-cloud-support-distributed-tracing"></a>¿Admite Azure Spring Cloud el seguimiento distribuido?

Sí. Para más información, consulte el [Tutorial: uso del seguimiento distribuido con Azure Spring Cloud](spring-cloud-tutorial-distributed-tracing.md).

::: zone pivot="programming-language-java"
### <a name="what-resource-types-does-service-binding-support"></a>¿Qué tipos de recursos admiten el enlace de servicios?

Actualmente se admiten tres servicios:
* Azure Cosmos DB
* Azure Database for MySQL
* Azure Cache for Redis.
::: zone-end

### <a name="can-i-view-add-or-move-persistent-volumes-from-inside-my-applications"></a>¿Puedo ver, agregar o mover volúmenes persistentes desde dentro de mis aplicaciones?

Sí.

### <a name="when-i-deletemove-an-azure-spring-cloud-service-instance-will-its-extension-resources-be-deletedmoved-as-well"></a>Cuando elimino o traslado una instancia del servicio Azure Spring Cloud, ¿también se eliminarán o trasladarán sus recursos de extensión?

Depende de la lógica de los proveedores de recursos a los que pertenecen los recursos de extensión. Los recursos de extensión de una instancia de `Microsoft.AppPlatform` no pertenecen al mismo espacio de nombres, por lo que el comportamiento varía en función del proveedor de recursos. Por ejemplo, la operación de eliminación o traslado no se aplicará en cascada a los recursos de la **configuración de diagnósticos**. Si se aprovisiona una nueva instancia de Azure Spring Cloud con el mismo id. de recurso que la eliminada o si la instancia de Azure Spring Cloud anterior se vuelve a trasladar, los recursos de la **configuración de diagnósticos** anteriores siguen extendiéndola.

Puede eliminar la configuración de diagnóstico de Spring Cloud mediante la CLI de Azure:

```azurecli
 az monitor diagnostic-settings delete --name $diagnosticSettingName --resource $azureSpringCloudResourceId
```

::: zone pivot="programming-language-java"
## <a name="java-runtime-and-os-versions"></a>Runtime de Java y versiones del sistema operativo

### <a name="which-versions-of-java-runtime-are-supported-in-azure-spring-cloud"></a>¿Qué versiones del runtime de Java se admiten en Azure Spring Cloud?

Azure Spring Cloud es compatible con las versiones LTS de Java con las compilaciones más recientes; actualmente, en junio de 2020, se admiten la compilación 252 de Java 8 y la compilación 7 de Java 11. Consulte [Instalación del JDK para Azure y Azure Stack](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-install).

### <a name="who-built-these-java-runtimes"></a>¿Quién ha compilado estos runtimes de Java?

Azul Systems. Las compilaciones del JDK Azul Zulu for Azure - Enterprise Edition son una distribución gratuita, multiplataforma y lista para producción del OpenJDK para Azure y Azure Stack, con la tecnología de Microsoft y Azul Systems. Contienen todos los componentes para la creación y ejecución de aplicaciones de Java SE.

### <a name="how-often-will-java-runtimes-get-updated"></a>¿Con qué frecuencia se actualizan los runtimes de Java?

Las versiones LTS y MTS del JDK tienen actualizaciones de seguridad trimestrales, correcciones de errores, así como actualizaciones críticas y revisiones puntuales según sea necesario. Este soporte técnico incluye entregas de actualizaciones de seguridad y correcciones de errores para Java 7 y 8 que figuran en las versiones más recientes de Java, como Java 11.

### <a name="how-long-will-java-8-and-java-11-lts-versions-be-supported"></a>¿Cuánto tiempo se admitirán las versiones LTS de Java 8 y Java 11?

Consulte [Soporte técnico de Java a largo plazo para Azure y Azure Stack](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-long-term-support).

* Java 8 LTS tiene soporte técnico hasta diciembre de 2030.
* Java 11 LTS tiene soporte técnico hasta septiembre de 2027.

### <a name="how-can-i-download-a-supported-java-runtime-for-local-development"></a>¿Cómo puedo descargar un runtime de Java compatible para el desarrollo local?

Consulte [Instalación del JDK para Azure y Azure Stack](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-install).

### <a name="what-is-the-retire-policy-for-older-java-runtimes"></a>¿Cuál es la directiva de retirada de los runtimes de Java más antiguos?

Se realizará un anuncio público 12 meses antes de retirar cualquier versión anterior del runtime. Tendrá 12 meses para migrar a una versión posterior.

* Los administradores de la suscripción recibirán una notificación por correo electrónico acerca de cuándo se retirará una versión de Java.
* La información de retirada se publicará en la documentación.

### <a name="how-can-i-get-support-for-issues-at-the-java-runtime-level"></a>¿Cómo puedo obtener soporte técnico para problemas en el nivel de runtime de Java?

Puede abrir una incidencia de soporte técnico con el departamento de Soporte técnico de Azure.  Consulte [Creación de una solicitud de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).

### <a name="what-is-the-operation-system-to-run-my-apps"></a>¿En qué sistema operativo se ejecutan mis aplicaciones?

Se utiliza la versión LTS de Ubuntu más reciente; actualmente, el sistema operativo predeterminado es [Ubuntu 20.04 LTS (Focal Fossa)](https://releases.ubuntu.com/focal/).

### <a name="how-often-are-os-security-patches-applied"></a>¿Con qué frecuencia se aplican las actualizaciones de seguridad del sistema operativo?

Las actualizaciones de seguridad aplicables a Azure Spring Cloud se implementarán en producción mensualmente.
Las actualizaciones de seguridad críticas (puntuación CVE >= 9) aplicables a Azure Spring Cloud se implementarán lo antes posible.
::: zone-end

## <a name="deployment"></a>Implementación

### <a name="does-azure-spring-cloud-support-blue-green-deployment"></a>¿Admite Azure Spring Cloud la implementación blue-green?
Sí. Para obtener más información, consulte [Configuración de un entorno de ensayo](spring-cloud-howto-staging-environment.md).

### <a name="can-i-access-kubernetes-to-manipulate-my-application-containers"></a>¿Puedo acceder a Kubernetes para manipular mis contenedores de aplicaciones?

No.  Azure Spring Cloud ayuda a desviar la atención del desarrollador de la arquitectura subyacente, lo que le permite concentrarse en el código de la aplicación y en la lógica de negocios.

### <a name="does-azure-spring-cloud-support-building-containers-from-source"></a>¿Admite Azure Spring Cloud la creación de contenedores desde el origen?

Sí. Para obtener más información, consulte [Inicio de la aplicación Spring Cloud desde el código fuente](spring-cloud-quickstart.md).

### <a name="does-azure-spring-cloud-support-autoscaling-in-app-instances"></a>¿Admite Azure Spring Cloud la escalabilidad automática en instancias de aplicaciones?

No.

::: zone pivot="programming-language-java"
### <a name="what-are-the-best-practices-for-migrating-existing-spring-cloud-microservices-to-azure-spring-cloud"></a>¿Cuáles son los procedimientos recomendados para migrar los microservicios de Spring Cloud existentes a Azure Spring Cloud?

Al migrar los microservicios de Spring Cloud existentes a Azure Spring Cloud, es una buena idea realizar los siguientes procedimientos recomendados:
* Todas las dependencias de la aplicación deben resolverse.
* Es preciso preparar las entradas de configuración, las variables de entorno y los parámetros de JVM para que pueda compararlos con los de la implementación en Azure Spring Cloud.
* Si quiere usar el enlace de servicios, examine los servicios de Azure y asegúrese de que ha establecido los permisos de acceso adecuados.
* Se recomienda quitar o deshabilitar todos los servicios insertados que podrían entrar en conflicto con los servicios administrados por Azure Spring Cloud, como nuestro servicio Service Discovery y Config Server, entre otros.
* Se recomienda usar las bibliotecas de Pivotal Spring oficiales estables. Las versiones no oficiales, beta o bifurcadas de las bibliotecas de Pivotal Spring no tienen el soporte de un Acuerdo de Nivel de Servicio (SLA).

Después de la migración, supervise las métricas de la CPU y RAM, y el tráfico de red para garantizar que las instancias de la aplicación se escalan adecuadamente.
::: zone-end

::: zone pivot="programming-language-csharp"
## <a name="net-core-versions"></a>Versiones de .NET Core

### <a name="which-net-core-versions-are-supported"></a>¿Qué versiones de .NET Core son compatibles?

.NET Core 3.1 y versiones posteriores.

### <a name="how-long-will-net-core-31-be-supported"></a>¿Cuánto tiempo se admitirá .NET Core 3.1?

Hasta el 3 de diciembre de 2022. Consulte [Visualización de la directiva de compatibilidad de .NET Core](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).
::: zone-end


## <a name="troubleshooting"></a>Solución de problemas

### <a name="what-are-the-impacts-of-service-registry-rarely-unavailable"></a>¿Cuáles son las repercusiones del registro de servicio que raramente no está disponible?

En un escenario que rara vez sucede, puede ver algunos errores como 
```
RetryableEurekaHttpClient: Request execution failure with status code 401; retrying on another server if available
```
desde los registros de aplicaciones. Este es un problema introducido por Spring Framework con una tasa muy baja debido a la inestabilidad de la red o a otros problemas de red. 

No debe haber ninguna repercusión en la experiencia del usuario, ya que el cliente Eureka tiene una directiva de reintentos y latido para poder encargarse de ello. Puede considerarlo como un error transitorio y omitirlo de forma segura.

Mejoraremos esta parte y evitaremos este error de las aplicaciones de los usuarios en breve.


## <a name="next-steps"></a>Pasos siguientes

Si tiene más preguntas, consulte la [guía de solución de problemas de Azure Spring Cloud](spring-cloud-troubleshoot.md).
