---
title: Guía para la solución de problemas de Azure Spring Cloud | Microsoft Docs
description: Guía para la solución de problemas de Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: troubleshooting
ms.date: 11/04/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 5a67ebbf0f83f2dc3a340f52cab7437bbfaa350e
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299174"
---
# <a name="troubleshoot-common-azure-spring-cloud-issues"></a>Solución de problemas comunes de Azure Spring Cloud

En este artículo se proporcionan instrucciones para solucionar problemas de desarrollo de Azure Spring Cloud. Para obtener más información, consulte las [preguntas más frecuentes sobre Azure Spring Cloud](spring-cloud-faq.md).

## <a name="availability-performance-and-application-issues"></a>Incidencias relacionadas con la disponibilidad, el rendimiento y las aplicaciones

### <a name="my-application-cant-start-for-example-the-endpoint-cant-be-connected-or-it-returns-a-502-after-a-few-retries"></a>No se puede iniciar la aplicación (por ejemplo, el punto de conexión no se puede conectar o devuelve el error 502 a los pocos reintentos)

Exporte los registros a Azure Log Analytics. La tabla de los registros de aplicaciones de Spring se denomina *AppPlatformLogsforSpring*. Para obtener más información, consulte [Análisis de registros y métricas con la configuración de diagnóstico](diagnostic-services.md).

Puede aparecer el siguiente mensaje de error en los registros:

> "org.springframework.context.ApplicationContextException: Unable to start web server" (No se puede iniciar el servidor web).

El mensaje indica uno de dos problemas probables: 
* Falta uno de los beans o una de sus dependencias.
* Falta una de las propiedades de bean o no es válida. En este caso, es probable que se muestre "java.lang.IllegalArgumentException".

Es posible que los enlaces de servicios también provoquen errores de inicio de la aplicación. Para consultar los registros, use palabras clave relacionadas con los servicios enlazados. Por ejemplo, supongamos que la aplicación tiene un enlace a una instancia de MySQL establecida en la hora del sistema local. Si la aplicación no se inicia, puede aparecer el siguiente error en el registro:

> "java.sql.SQLException: The server time zone value 'Coordinated Universal Time' is unrecognized or represents more than one time zone" (El valor de zona horaria del servidor 'Hora universal coordinada' no se reconoce o representa más de una zona horaria).

Para solucionar este error, vaya a `server parameters` en la instancia de MySQL y cambie el valor `time_zone` de *SYSTEM* a *+0:00*.


### <a name="my-application-crashes-or-throws-an-unexpected-error"></a>La aplicación se bloquea o genera un error inesperado

Al depurar los bloqueos de la aplicación, empiece por comprobar el estado de la ejecución y de detección de la aplicación. Para ello, vaya a _Administración de aplicaciones_ en Azure Portal para asegurarse de que los estados de todas las aplicaciones sean _En ejecución_ y _ACTIVO_.

* Si el estado es _En ejecución_, pero el estado de detección no es _ACTIVO_, vaya a la sección [No se puede registrar mi aplicación](#my-application-cant-be-registered).

* Si el estado de detección es _ACTIVO_, vaya a Métricas para comprobar el estado de la aplicación. Compruebe las siguientes métricas:


  - `TomcatErrorCount` (_tomcat.global.error_): Aquí se cuentan todas las excepciones de la aplicación Spring. Si el número es grande, vaya a Azure Log Analytics para comprobar los registros de aplicaciones.

  - `AppMemoryMax` (_jvm.memory.max_): La cantidad máxima de memoria disponible para la aplicación. Es posible que la cantidad no esté definida o que cambie con el tiempo si se define. Si se define, la cantidad de memoria usada y confirmada siempre es menor o igual que el máximo. Sin embargo, es posible que se produzca un error en una asignación de memoria con un mensaje `OutOfMemoryError` si la asignación intenta aumentar la memoria usada de forma que *usada > confirmada*, incluso si se sigue cumpliendo *usada <= máx.* . En tal caso, intente aumentar el tamaño máximo del montón mediante el parámetro `-Xmx`.

  - `AppMemoryUsed` (_jvm.memory.used_): La cantidad de memoria, en bytes, que usa actualmente la aplicación. En el caso de una aplicación Java de carga normal, esta serie de métricas forma un patrón de *sierra*, donde el uso de memoria aumenta y disminuye continuamente en pequeños incrementos y caídas repentinas y, a continuación, este patrón se repite. Esta serie de métricas se debe a la recolección de elementos no utilizados dentro de la máquina virtual Java, donde las acciones de recolección representan caídas en el patrón de sierra.
    
    Esta métrica es importante para ayudar a identificar problemas de memoria, por ejemplo:
    * Una explosión de memoria al principio mismo.
    * La asignación de memoria de sobrecarga para una ruta de acceso lógica específica.
    * Fugas de memoria graduales.

  Para obtener más información, consulte [Métricas](spring-cloud-concept-metrics.md).

Para obtener más información sobre Azure Log Analytics, consulte [Introducción a los análisis de registros de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal).

### <a name="my-application-experiences-high-cpu-usage-or-high-memory-usage"></a>Mi aplicación experimenta un uso elevado de la CPU o la memoria

Si la aplicación experimenta un uso elevado de la CPU o de la memoria, sucede una de estas dos cosas:
* Todas las instancias de la aplicación experimentan un uso elevado de la CPU o de la memoria.
* Algunas de las instancias de la aplicación experimentan un uso elevado de la CPU o de la memoria.

Para averiguar qué situación se aplica, haga lo siguiente:

1. Vaya a **Métricas** y seleccione **Service CPU Usage Percentage** (Porcentaje de uso de CPU del servicio) o **Service Memory Used** (Memoria de servicio usada).
2. Agregue un filtro **App=** para especificar qué aplicación quiere supervisar.
3. Divida las métricas por **Instancia**.

Si *todas las instancias* experimentan un uso elevado de la CPU o memoria, debe escalar horizontalmente la aplicación o escalar verticalmente e uso de CPU o memoria. Para más información, consulte el [Tutorial: Escalado de una aplicación en Azure Spring Cloud](spring-cloud-tutorial-scale-manual.md).

Si solo *algunas instancias* experimentan un uso elevado de la CPU o de la memoria, compruebe el estado de las instancias y su estado de detección.

Para obtener más información, consulte [Métricas en Azure Spring Cloud](spring-cloud-concept-metrics.md).

Si todas las instancias están en funcionamiento, vaya a Azure Log Analytics para consultar los registros de la aplicación y examine la lógica del código. Esto le ayudará a ver si alguna puede afectar a la creación de particiones de escalado. Para obtener más información, consulte [Análisis de registros y métricas con la configuración de diagnóstico](diagnostic-services.md).

Para obtener más información sobre Azure Log Analytics, consulte [Introducción a los análisis de registros de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal). Consulte los registros con el [lenguaje de consulta Kusto](https://docs.microsoft.com/azure/kusto/query/).

### <a name="checklist-for-deploying-your-spring-application-to-azure-spring-cloud"></a>Lista de comprobación para la implementación de la aplicación Spring en Azure Spring Cloud

Antes de incorporar la aplicación, asegúrese de que cumple los siguientes criterios:

* La aplicación se puede ejecutar localmente con la versión del runtime de Java especificada.
* La configuración del entorno (CPU/RAM/instancias) cumple los requisitos mínimos que establece el proveedor de aplicaciones.
* Los elementos de configuración tienen los valores esperados. Para más información, consulte el artículo acerca de [Config Server](spring-cloud-tutorial-config-server.md).
* Las variables de entorno tienen los valores esperados.
* Los parámetros de JVM tienen los valores esperados.
* Se recomienda deshabilitar o quitar los servicios _Config Server_ y _Spring Service Registry_ insertados del paquete de aplicación.
* Si algún recurso de Azure se debe enlazar mediante _Enlaces de servicio_, asegúrese de que los recursos de destino estén en funcionamiento.

## <a name="configuration-and-management"></a>Configuración y administración

### <a name="i-encountered-a-problem-with-creating-an-azure-spring-cloud-service-instance"></a>Ha aparecido un problema al crear una instancia de servicio de Azure Spring Cloud

Al configurar una instancia de servicio de Azure Spring Cloud desde Azure Portal, Azure Spring Cloud realiza la validación automáticamente.

Pero si intenta configurar la instancia de servicio de Azure Spring Cloud mediante la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) o una [plantilla de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/), verifique lo siguiente:

* Que la suscripción está activa.
* Que Azure Spring Cloud [admite](spring-cloud-faq.md) la ubicación.
* Que se ha creado el grupo de recursos de la instancia.
* Que el nombre del recurso se ajusta a la regla de nomenclatura Solo debe contener minúsculas, números y guiones. El primer carácter debe ser una letra. El último carácter debe ser una letra o un número. El valor tiene que contener entre 2 y 32 caracteres.

Si desea configurar la instancia de servicio de Azure Spring Cloud mediante la plantilla de Resource Manager, consulte primero [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates).

El nombre de la instancia de servicio de Azure Spring Cloud se usará para solicitar un nombre de subdominio en `azureapps.io`, por lo que se producirá un error en la configuración si el nombre entra en conflicto con uno existente. Puede encontrar más detalles en los registros de actividad.

### <a name="i-cant-deploy-a-jar-package"></a>No puedo implementar un paquete JAR

No puede cargar el archivo de Java Archive (JAR) o el paquete de origen mediante Azure Portal o la plantilla de Resource Manager.

Al implementar el paquete de aplicación mediante la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli), esta sondea periódicamente el progreso de la implementación y, al final, muestra el resultado de la implementación.

Si se interrumpe el sondeo, se puede seguir usando el siguiente comando para capturar los registros de implementación:

`az spring-cloud app show-deploy-log -n <app-name>`

Asegúrese de que la aplicación esté empaquetada en el [formato jar ejecutable](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html) correcto. Si no está empaquetada correctamente, recibirá un mensaje de error similar al siguiente:

> "Error: Invalid or corrupt jarfile /jar/38bc8ea1-a6bb-4736-8e93-e8f3b52c8714" (Archivo JAR no válido o dañado /jar/38bc8ea1-a6bb-4736-8e93-e8f3b52c8714)

### <a name="i-cant-deploy-a-source-package"></a>No puedo implementar un paquete de origen

No puede cargar el paquete de origen o JAR mediante Azure Portal o la plantilla de Resource Manager.

Al implementar el paquete de aplicación mediante la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli), esta sondea periódicamente el progreso de la implementación y, al final, muestra el resultado de la implementación.

Si se interrumpe el sondeo, se puede seguir usando el siguiente comando para capturar los registros de creación e implementación:

`az spring-cloud app show-deploy-log -n <app-name>`

Sin embargo, tenga en cuenta que una instancia de servicio de Azure Spring Cloud solo puede desencadenar un trabajo de compilación para un paquete de origen a la vez. Para obtener más información, vea [Implementación de una aplicación](spring-cloud-quickstart.md) y [Configuración de un entorno de ensayo en Azure Spring Cloud](spring-cloud-howto-staging-environment.md).

### <a name="my-application-cant-be-registered"></a>No se puede registrar mi aplicación

En la mayoría de los casos, esta situación se produce cuando *Dependencias requeridas* y *Detección de servicios* no están configuradas correctamente en el archivo de Project Object Model (POM). Una vez configuradas, el punto de conexión del servidor de Service Registry integrado se inserta como variable de entorno con la aplicación. A continuación, las aplicaciones se registran automáticamente en el servidor de Service Registry y detectan otros microservicios dependientes.

Espere al menos dos minutos para que una instancia recién registrada empiece a recibir tráfico.

Si va a migrar a Azure una solución existente de Spring Cloud, asegúrese de que las instancias ad-hoc de _Service Registry_ y _Config Server_ se hayan quitado (o deshabilitado) para evitar conflictos con las instancias administradas proporcionadas por Azure Spring Cloud.

También puede consultar los registros de cliente de _Service Registry_ en Azure Log Analytics. Para obtener más información, consulte [Análisis de registros y métricas con la configuración de diagnóstico](diagnostic-services.md).

Para obtener más información sobre Azure Log Analytics, consulte [Introducción a los análisis de registros de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal). Consulte los registros con el [lenguaje de consulta Kusto](https://docs.microsoft.com/azure/kusto/query/).

### <a name="i-want-to-inspect-my-applications-environment-variables"></a>Deseo inspeccionar las variables de entorno de mi aplicación

Las variables de entorno informan al marco de Azure Spring Cloud, lo que garantiza que Azure sabe dónde y cómo configurar los servicios que componen la aplicación. Asegurarse de que las variables de entorno son correctas es un primer paso necesario para solucionar los posibles problemas.  Puede usar el punto de conexión de Spring Boot Actuator para examinar las variables de entorno.  

> [!WARNING]
> Este procedimiento expone las variables de entorno mediante un punto de conexión de prueba.  No continúe si se puede acceder de forma pública al punto de conexión de prueba o si ha asignado un nombre de dominio a la aplicación.

1. Ir a `https://<your application test endpoint>/actuator/health`.  
    - Una respuesta similar a `{"status":"UP"}` indica que se ha habilitado el punto de conexión.
    - Si la respuesta es negativa, incluya la siguiente dependencia en el archivo *POM.xml*:

        ```xml
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
        ```

1. Con el punto de conexión de Spring Boot Actuator habilitado, vaya a Azure Portal y busque la página de configuración de la aplicación.  Agregue una variable de entorno con el nombre `MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE` y el valor `*` . 

1. Reinicie la aplicación.

1. Vaya a `https://<your application test endpoint>/actuator/env` e inspeccione la respuesta.  Debería ser parecido a este:

    ```json
    {
        "activeProfiles": [],
        "propertySources": {,
            "name": "server.ports",
            "properties": {
                "local.server.port": {
                    "value": 1025
                }
            }
        }
    }
    ```

Busque el nodo secundario denominado `systemEnvironment`.  Este nodo contiene las variables de entorno de la aplicación.

> [!IMPORTANT]
> No olvide invertir la exposición de las variables de entorno antes de que la aplicación esté disponible para el público.  Vaya a Azure Portal, busque la página de configuración de la aplicación y elimine esta variable de entorno: `MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE`.

### <a name="i-cant-find-metrics-or-logs-for-my-application"></a>No encuentro las métricas ni los registros de la aplicación

Vaya a **Administración de la aplicación** para asegurarse de que los estados de la aplicación sean _En ejecución_ y _ACTIVO_.

Active esta opción para ver si _JMX_ está habilitado en el paquete de aplicación. Esta característica se puede habilitar con la propiedad de configuración `spring.jmx.enabled=true`.  

Compruebe si la dependencia de `spring-boot-actuator` está habilitada en el paquete de aplicación y si arranca correctamente.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Si los registros de la aplicación se pueden archivar en una cuenta de almacenamiento, pero no se envían a Azure Log Analytics, compruebe si [configuró el área de trabajo correctamente](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace). Si usa un nivel gratuito de Azure Log Analytics, tenga en cuenta que [el nivel gratuito no proporciona ningún contrato de nivel de servicio (SLA)](https://azure.microsoft.com/support/legal/sla/log-analytics/v1_3/).
