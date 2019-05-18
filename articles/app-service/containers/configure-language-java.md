---
title: 'Configurar aplicaciones de Java para Linux: Azure App Service | Microsoft Docs'
description: Aprenda a configurar aplicaciones Java que se ejecutan en Azure App Service en Linux.
keywords: servicio de aplicación de Azure, aplicación web, linux, oss, java y java ee, jee, javaee
services: app-service
author: rloutlaw
manager: angerobe
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: java
ms.topic: article
ms.date: 03/28/2019
ms.author: routlaw
ms.custom: seodec18
ms.openlocfilehash: 883042e7c8abb43338c55a76bba3d64844ce1c56
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604346"
---
# <a name="configure-a-linux-java-app-for-azure-app-service"></a>Configurar una aplicación de Java para Linux para Azure App Service

Azure App Service en Linux permite a los desarrolladores de Java compilar, implementar y escalar rápidamente sus aplicaciones web empaquetadas de Tomcat o Java Standard Edition (SE) en un servicio basado en Linux totalmente administrado. Implemente aplicaciones con complementos de Maven desde la línea de comandos o en editores, como IntelliJ, Eclipse o Visual Studio Code.

Esta guía proporciona los conceptos clave e instrucciones para desarrolladores de Java que usa un contenedor de Linux integrado en App Service. Si nunca ha usado Azure App Service, siga el [inicio rápido de Java](quickstart-java.md) y [Java con PostgreSQL tutorial](tutorial-java-enterprise-postgresql-app.md) primero.

## <a name="deploying-your-app"></a>Implementación de la aplicación

Puede usar [complemento Maven de Azure App Service](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme) para implementar los archivos .jar y .war. También se admite la implementación con IDE populares con [Azure Toolkit for IntelliJ](/java/azure/intellij/azure-toolkit-for-intellij) o [Azure Toolkit for Eclipse](/java/azure/eclipse/azure-toolkit-for-eclipse).

En caso contrario, el método de implementación dependerá de su tipo de archivo:

- Para implementar archivos .war en Tomcat, utilice el punto de conexión `/api/wardeploy/` para realizar el conjunto de rutinas POST en el archivo. Para obtener más información sobre esta API, consulte [este documento](https://docs.microsoft.com/azure/app-service/deploy-zip#deploy-war-file).
- Para implementar archivos .jar en las imágenes de Java SE, use el punto de conexión `/api/zipdeploy/` del sitio de Kudu. Para obtener más información sobre la API, consulte [este documento](https://docs.microsoft.com/azure/app-service/deploy-zip#rest).

No implemente el archivo .war o .jar con el FTP. La herramienta FTP está diseñada para cargar los scripts de inicio, dependencias u otros archivos en tiempo de ejecución. Tenga en cuenta que no es la opción óptima para realizar la implementación de aplicaciones web.

## <a name="logging-and-debugging-apps"></a>Registro y depuración de aplicaciones

Encontrará informes de rendimiento, visualizaciones de tráfico y comprobaciones de mantenimiento de cada aplicación a través de Azure Portal. Para obtener más información, consulte [Introducción a diagnósticos de Azure App Service](../overview-diagnostics.md).

### <a name="ssh-console-access"></a>Acceso a la consola SSH

[!INCLUDE [Open SSH session in browser](../../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

### <a name="stream-diagnostic-logs"></a>Transmisión de registros de diagnóstico

[!INCLUDE [Access diagnostic logs](../../../includes/app-service-web-logs-access-no-h.md)]

Para obtener más información, consulte el apartado sobre el [streaming de registros con la CLI de Azure](../troubleshoot-diagnostic-logs.md#streaming-with-azure-cli).

### <a name="app-logging"></a>Registro de aplicaciones

Habilite el [registro de aplicaciones](../troubleshoot-diagnostic-logs.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#enablediag) a través de Azure Portal o la [CLI de Azure](/cli/azure/webapp/log#az-webapp-log-config) para configurar App Service para que escriba los flujos de salida y de error de la consola estándar en el sistema de archivos local o en Azure Blob Storage. El registro en el sistema de archivos local de App Service se deshabilita 12 horas después de su configuración. Si necesita una retención más prolongada, configure la aplicación para escribir la salida en un contenedor de Blob Storage. Los registros de aplicación de Java y Tomcat pueden encontrarse en el `/home/LogFiles/Application/` directory.

Si la aplicación usa [Logback](https://logback.qos.ch/) o [Log4j](https://logging.apache.org/log4j) para el seguimiento, puede reenviar estos seguimientos para su revisión en Azure Application Insights mediante las instrucciones de configuración del marco de registro en [Exploración de los registros de seguimiento de Java en Application Insights](/azure/application-insights/app-insights-java-trace-logs).

### <a name="troubleshooting-tools"></a>Herramientas de solución de problemas

Las imágenes integradas de Java se basan en el [Alpine Linux](https://alpine-linux.readthedocs.io/en/latest/getting_started.html) del sistema operativo. Use la `apk` Administrador de paquetes para instalar cualquier solución de problemas de las herramientas o comandos.

## <a name="customization-and-tuning"></a>Personalización y optimización

Azure App Service para Linux admite fuera del cuadro optimización y la personalización mediante la CLI y Azure portal. Revise los artículos siguientes para la configuración de aplicación web de Java específicos:

- [Configuración de App Service](../web-sites-configure.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- [Configuración de un dominio personalizado](../app-service-web-tutorial-custom-domain.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- [Habilitación de SSL](../app-service-web-tutorial-custom-ssl.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- [Adición de una red CDN](../../cdn/cdn-add-to-web-app.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- [Configurar el sitio de Kudu](https://github.com/projectkudu/kudu/wiki/Configurable-settings#linux-on-app-service-settings)

### <a name="set-java-runtime-options"></a>Definición de las opciones de Java Runtime

Para establecer la memoria asignada u otras opciones de tiempo de ejecución JVM en entornos de Tomcat y de Java SE, cree un [configuración de la aplicación](../web-sites-configure.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#app-settings) denominado `JAVA_OPTS` con las opciones. App Service Linux pasa esta configuración como una variable de entorno para Java Runtime cuando se inicia.

En Azure Portal, en **Configuración de la aplicación** para la aplicación web, cree un nuevo valor de la aplicación denominado `JAVA_OPTS` que incluya valores de configuración adicionales, como `-Xms512m -Xmx1204m`.

Para configurar la configuración de la aplicación desde el complemento Maven, agregue las etiquetas de configuración y valor en la sección de complemento de Azure. El ejemplo siguiente establece un tamaño de montón de Java mínimo y máximo específico:

```xml
<appSettings>
    <property>
        <name>JAVA_OPTS</name>
        <value>-Xms512m -Xmx1204m</value>
    </property>
</appSettings>
```

Los desarrolladores que ejecutan una sola aplicación con una ranura de implementación en su plan de App Service pueden usar las siguientes opciones:

- Instancias de S1 y B1: `-Xms1024m -Xmx1024m`
- Instancias de B2 y S2: `-Xms3072m -Xmx3072m`
- Instancias B3 y S3: `-Xms6144m -Xmx6144m`

Cuando optimice la configuración del montón de la aplicación, revise los detalles de su plan de App Service y tenga en cuenta distintas necesidades de aplicaciones y ranuras de implementación para encontrar la asignación óptima de memoria.

Si va a implementar una aplicación de JAR, debe denominarse `app.jar` para que la imagen integrada puede identificar correctamente la aplicación. (El complemento Maven realiza automáticamente este cambio de nombre). Si no desea cambiar el nombre de archivo JAR para `app.jar`, puede cargar un script de shell con el comando para ejecutar el archivo JAR. A continuación, pegue la ruta de acceso completa a esta secuencia de comandos en el [archivo de inicio](https://docs.microsoft.com/azure/app-service/containers/app-service-linux-faq#startup-file) cuadro de texto en la sección de configuración del portal.

### <a name="turn-on-web-sockets"></a>Activación de sockets web

Active la compatibilidad con sockets web en Azure Portal en **Configuración de la aplicación** para la aplicación. Deberá reiniciar la aplicación para que la configuración surta efecto.

Active la compatibilidad con los sockets web mediante la CLI de Azure con el comando siguiente:

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --web-sockets-enabled true
```

A continuación, reinicie su aplicación:

```azurecli-interactive
az webapp stop --name <app-name> --resource-group <resource-group-name>
az webapp start --name <app-name> --resource-group <resource-group-name>
```

### <a name="set-default-character-encoding"></a>Definición de la codificación de caracteres predeterminada

En Azure Portal, en **Configuración de la aplicación** para la aplicación web, cree un nuevo valor de la aplicación denominado `JAVA_OPTS` con el valor `-Dfile.encoding=UTF-8`.

También puede configurar el valor de la aplicación mediante el complemento de Maven de App Service. Agregue las etiquetas setting y value del valor en la configuración del complemento:

```xml
<appSettings>
    <property>
        <name>JAVA_OPTS</name>
        <value>-Dfile.encoding=UTF-8</value>
    </property>
</appSettings>
```

### <a name="adjust-startup-timeout"></a>Ajustar el tiempo de espera de inicio

Si la aplicación de Java es especialmente grande, debe aumentar el límite de tiempo de inicio. Para ello, cree una configuración de aplicación, `WEBSITES_CONTAINER_START_TIME_LIMIT` y establézcalo como el número de segundos que App Service debe esperar antes de agotar el tiempo. El valor máximo es `1800` segundos.

## <a name="secure-applications"></a>Aplicaciones seguras

Las aplicaciones de Java que se ejecutan en App Service para Linux presentan el mismo conjunto de [procedimientos recomendados de seguridad](/azure/security/security-paas-applications-using-app-services) que otras aplicaciones. 

### <a name="authenticate-users"></a>Autenticación de usuarios

Configurar la autenticación de la aplicación en el portal de Azure con el **autenticación y autorización** opción. Desde allí, puede habilitar la autenticación con Azure Active Directory o con inicios de sesión en redes sociales como Facebook, Google o GitHub. La configuración de Azure Portal solo funciona al configurar un proveedor de autenticación único. Para obtener más información, consulte [Configuración de una aplicación de App Service para usar el inicio de sesión de Azure Active Directory](../configure-authentication-provider-aad.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json) y los artículos relacionados de otros proveedores de identidades.

Si tiene que habilitar varios proveedores de inicio de sesión, siga las instrucciones del artículo sobre la [personalización de la autenticación en App Service](../app-service-authentication-how-to.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json).

 Los desarrolladores de Spring Boot pueden usar el [iniciador de Spring Boot para Azure Active Directory](/java/azure/spring-framework/configure-spring-boot-starter-java-app-with-azure-active-directory?view=azure-java-stable) para proteger las aplicaciones mediante las anotaciones y las API conocidas de Spring Security. Asegúrese de aumentar el tamaño máximo del encabezado en el archivo `application.properties`. Se recomienda un valor de `16384`.

### <a name="configure-tlsssl"></a>Configuración de TLS/SSL

Siga las instrucciones de [Enlace de un certificado SSL personalizado existente](../app-service-web-tutorial-custom-ssl.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json) para cargar un certificado SSL existente y enlazarlo al nombre de dominio de la aplicación. De forma predeterminada, la aplicación seguirá permitiendo que las conexiones HTTP sigan los pasos específicos del tutorial para aplicar SSL y TLS.

### <a name="use-keyvault-references"></a>Usar referencias de almacén de claves

[Azure Key Vault](../../key-vault/key-vault-overview.md) proporciona administración centralizada de secreto con el historial de auditoría y de las directivas de acceso. Puede almacenar información confidencial (como contraseñas o las cadenas de conexión) en Key Vault y tener acceso a estos secretos en la aplicación a través de las variables de entorno.

En primer lugar, siga las instrucciones para [conceder su aplicación acceso a Key Vault](../app-service-key-vault-references.md#granting-your-app-access-to-key-vault) y [que hace una referencia de almacén de claves a su secreto en una configuración de aplicación](../app-service-key-vault-references.md#reference-syntax). Puede validar que la referencia se resuelve como el secreto mediante la impresión de la variable de entorno al acceder remotamente a la terminal de App Service.

Para insertar estos secretos en el archivo de configuración Spring o Tomcat, use la sintaxis de inserción de variable de entorno (`${MY_ENV_VAR}`). Para archivos de configuración de la primavera, consulte esta documentación sobre [externalizado configuraciones](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html).

## <a name="configure-apm-platforms"></a>Configurar plataformas APM

En esta sección se muestra cómo conectar las aplicaciones Java implementadas en Azure App Service en Linux con el rendimiento de la aplicación NewRelic y AppDynamics supervisión plataformas (APM).

[Configurar New Relic](#configure-new-relic)
[configurar AppDynamics](#configure-appdynamics)

### <a name="configure-new-relic"></a>Configuración de New Relic

1. Cree una cuenta de NewRelic en [NewRelic.com](https://newrelic.com/signup).
2. Descargue el agente de Java desde NewRelic; tendrá un nombre de archivo similar a `newrelic-java-x.x.x.zip`.
3. Copie la clave de licencia, ya que la necesitará más tarde para configurar el agente.
4. [SSH en la instancia de App Service](app-service-linux-ssh-support.md) y cree un nuevo directorio `/home/site/wwwroot/apm`.
5. Cargue los archivos desempaquetados del agente de Java de NewRelic en un directorio en `/home/site/wwwroot/apm`. Los archivos del agente deben estar en `/home/site/wwwroot/apm/newrelic`.
6. Modifique el archivo YAML en `/home/site/wwwroot/apm/newrelic/newrelic.yml` y reemplace el marcador de posición de valor de la licencia por su propia clave de licencia.
7. En Azure Portal, vaya a la aplicación en App Service y cree una nueva configuración de la aplicación.
    - Si la aplicación usa **Java SE**, cree una variable de entorno llamada `JAVA_OPTS` con el valor `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar`.
    - Si usa **Tomcat**, cree una variable de entorno llamada `CATALINA_OPTS` con el valor `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar`.
    - Si usas **WildFly**, consulte la documentación de New Relic [aquí](https://docs.newrelic.com/docs/agents/java-agent/additional-installation/wildfly-version-11-installation-java) para obtener instrucciones acerca de cómo instalar el agente de Java y la configuración de JBoss.
    - Si ya tiene una variable de entorno para `JAVA_OPTS` o `CATALINA_OPTS`, anexe la opción `javaagent` al final del valor actual.

### <a name="configure-appdynamics"></a>Configuración de AppDynamics

1. Cree una cuenta de AppDynamics en [AppDynamics.com](https://www.appdynamics.com/community/register/).
1. Descargue el agente de Java desde el sitio web de AppDynamics; el nombre de archivo será similar a `AppServerAgent-x.x.x.xxxxx.zip`.
1. [SSH en la instancia de App Service](app-service-linux-ssh-support.md) y cree un nuevo directorio `/home/site/wwwroot/apm`.
1. Cargue los archivos del agente de Java en un directorio en `/home/site/wwwroot/apm`. Los archivos del agente deben estar en `/home/site/wwwroot/apm/appdynamics`.
1. En Azure Portal, vaya a la aplicación en App Service y cree una nueva configuración de la aplicación.
    - Si usa **Java SE**, cree una variable de entorno llamada `JAVA_OPTS` con el valor `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>` donde `<app-name>` es el nombre de su instancia de App Service.
    - Si usa **Tomcat**, cree una variable de entorno llamada `CATALINA_OPTS` con el valor `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>` donde `<app-name>` es el nombre de su instancia de App Service.
    - Si usas **WildFly**, consulte la documentación de AppDynamics [aquí](https://docs.appdynamics.com/display/PRO45/JBoss+and+Wildfly+Startup+Settings) para obtener instrucciones acerca de cómo instalar el agente de Java y la configuración de JBoss.
    
## <a name="configure-jar-applications"></a>Configurar aplicaciones JAR

### <a name="starting-jar-apps"></a>A partir de las aplicaciones del archivo JAR

De forma predeterminada, App Service, espera que su aplicación JAR denominarse `app.jar`. Si tiene este nombre, se ejecutará automáticamente. Para los usuarios de Maven, puede establecer el nombre de archivo JAR incluyendo `<finalName>app</finalName>` en el `<build>` sección de su `pom.xml`. [Puede hacer lo mismo en Gradle](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.bundling.Jar.html#org.gradle.api.tasks.bundling.Jar:archiveFileName) estableciendo el `archiveFileName` propiedad.

Si desea usar un nombre diferente para el archivo JAR, también debe proporcionar el [comando de inicio](app-service-linux-faq.md#built-in-images) que ejecuta el archivo JAR. Por ejemplo, `java -jar my-jar-app.jar`. Puede establecer el valor para el comando de inicio en el Portal, en Configuración > configuración General, o con una configuración de aplicación denominada `STARTUP_COMMAND`.

### <a name="server-port"></a>Puerto del servidor

App Service Linux enruta las solicitudes entrantes al puerto 80, por lo que la aplicación debe escuchar en el puerto 80. Puede hacerlo en configuración de la aplicación (como del muelle `application.properties` archivo), o en el comando de inicio (por ejemplo, `java -jar spring-app.jar --server.port=80`). Consulte la siguiente documentación para marcos de trabajo comunes de Java:

- [Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-properties-and-configuration.html#howto-use-short-command-line-arguments)
- [SparkJava](http://sparkjava.com/documentation#embedded-web-server)
- [Micronaut](https://docs.micronaut.io/latest/guide/index.html#runningSpecificPort)
- [Reproducir el marco de trabajo](https://www.playframework.com/documentation/2.6.x/ConfiguringHttps#Configuring-HTTPS)
- [Vertx](https://vertx.io/docs/vertx-core/java/#_start_the_server_listening)
- [Quarkus](https://quarkus.io/guides/application-configuration-guide)

## <a name="data-sources"></a>Orígenes de datos

### <a name="tomcat"></a>Tomcat

Estas instrucciones se aplican a todas las conexiones de base de datos. Deberá rellenar los marcadores de posición con el nombre de clase de controlador de la base de datos elegido y con el archivo JAR. Se proporciona una tabla con los nombres de clase y las descargas de controladores de las bases de datos más habituales.

| Base de datos   | Nombre de clase de controlador                             | Controlador JDBC                                                                      |
|------------|-----------------------------------------------|------------------------------------------------------------------------------------------|
| PostgreSQL | `org.postgresql.Driver`                        | [Descargar](https://jdbc.postgresql.org/download.html)                                    |
| MySQL      | `com.mysql.jdbc.Driver`                        | [Descargar](https://dev.mysql.com/downloads/connector/j/) (seleccione "Platform Independent") |
| SQL Server | `com.microsoft.sqlserver.jdbc.SQLServerDriver` | [Descargar](https://docs.microsoft.com/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-2017#available-downloads-of-jdbc-driver-for-sql-server)                                                           |

Para configurar Tomcat para usar Java Database Connectivity (JDBC) o la API de persistencia de Java (JPA), personalizar el `CATALINA_OPTS` variable de entorno que se lee en Tomcat durante el inicio. Establezca estos valores a través de un valor de la aplicación en el [complemento de Maven de App Service](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md):

```xml
<appSettings>
    <property>
        <name>CATALINA_OPTS</name>
        <value>"$CATALINA_OPTS -Ddbuser=${DBUSER} -Ddbpassword=${DBPASSWORD} -DconnURL=${CONNURL}"</value>
    </property>
</appSettings>
```

O bien establecer las variables de entorno en la hoja "Configuración de la aplicación" de Azure Portal.

A continuación, determine si el origen de datos debe estar disponible para una aplicación o para todas las aplicaciones que se ejecutan en el servlet de Tomcat.

#### <a name="application-level-data-sources"></a>Orígenes de datos de nivel de aplicación

1. Cree un archivo `context.xml` en el directorio `META-INF/` del proyecto. Cree el directorio `META-INF/`, en caso de que no exista.

2. En `context.xml`, agregue un elemento `Context` para vincular el origen de datos a una dirección JNDI. Reemplace el marcador de posición `driverClassName` por el nombre de clase de su controlador de la tabla anterior.

    ```xml
    <Context>
        <Resource
            name="jdbc/dbconnection" 
            type="javax.sql.DataSource"
            url="${dbuser}"
            driverClassName="<insert your driver class name>"
            username="${dbpassword}" 
            password="${connURL}"
        />
    </Context>
    ```

3. Actualice el archivo `web.xml` de la aplicación para usar en esta el origen de datos.

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/dbconnection</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

#### <a name="shared-server-level-resources"></a>Recursos compartidos de nivel de servidor

1. Copie el contenido de `/usr/local/tomcat/conf` en `/home/tomcat/conf` en su instancia de App Service Linux mediante SSH si no tiene ninguna configuración allí todavía.
    ```
    mkdir -p /home/tomcat
    cp -a /usr/local/tomcat/conf /home/tomcat/conf
    ```

2. Agregue un elemento Context a `server.xml` dentro del elemento `<Server>`.

    ```xml
    <Server>
    ...
    <Context>
        <Resource
            name="jdbc/dbconnection" 
            type="javax.sql.DataSource"
            url="${dbuser}"
            driverClassName="<insert your driver class name>"
            username="${dbpassword}" 
            password="${connURL}"
        />
    </Context>
    ...
    </Server>
    ```

3. Actualice el archivo `web.xml` de la aplicación para usar en esta el origen de datos.

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/dbconnection</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

#### <a name="finalize-configuration"></a>Finalización de la configuración

Por último, coloque los archivos JAR de controlador en la classpath de Tomcat y reinicie el servicio de aplicaciones.

1. Asegúrese de que los archivos del controlador JDBC estén disponibles para el cargador de clases de Tomcat. Para ello, colóquelos en el directorio `/home/tomcat/lib`. (cree el directorio si no existe). Para cargar estos archivos en su instancia de App Service, realice los pasos siguientes:
    1. En el [Cloud Shell](https://shell.azure.com), instale la extensión de aplicación Web:

      ```azurecli-interactive
      az extension add -–name webapp
      ```

    2. Ejecute el siguiente comando CLI para crear un túnel SSH desde el sistema local a App Service:

      ```azurecli-interactive
      az webapp remote-connection create --resource-group <resource-group-name> --name <app-name> --port <port-on-local-machine>
      ```

    3. Establezca la conexión al puerto de tunelización local con el cliente SFTP y cargue los archivos en la carpeta `/home/tomcat/lib`.

    Como alternativa, puede usar un cliente FTP para cargar el controlador JDBC. Siga estas [instrucciones para obtener las credenciales de FTP](../deploy-configure-credentials.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json).

2. Si ha creado un origen de datos de nivel de servidor, reinicie la aplicación App Service de Linux. Tomcat restablecerá `CATALINA_HOME` a `/home/tomcat/conf` y usará la configuración actualizadas.

### <a name="spring-boot"></a>Spring Boot

Para conectarse a orígenes de datos en aplicaciones de Spring Boot, se recomienda crear cadenas de conexión e insertarlas en su `application.properties` archivo.

1. En la sección "Configuración de la aplicación" de la hoja de App Service, establezca un nombre para la cadena, pegue la cadena de conexión de JDBC en el campo de valor y establezca el tipo en "Custom". Opcionalmente, puede establecer esta cadena de conexión como configuración de ranura.

    ! [Crear una cadena de conexión en el Portal.]
    

    Esta cadena de conexión es accesible a nuestra aplicación como una variable de entorno denominada `CUSTOMCONNSTR_<your-string-name>`. Por ejemplo, la cadena de conexión creados anteriormente se denominarán `CUSTOMCONNSTR_exampledb`.

2. En su `application.properties` de archivos, hacer referencia a esta cadena de conexión con el nombre de variable de entorno. En nuestro ejemplo, se usaría la siguiente.

    ```yml
    app.datasource.url=${CUSTOMCONNSTR_exampledb}
    ```

Consulte la [documentación de Spring Boot en acceso a datos](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-data-access.html) y [externalizado configuraciones](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html) para obtener más información sobre este tema.

## <a name="configure-java-ee-wildfly"></a>Configuración de Java EE (WildFly)

> [!NOTE]
> Java Enterprise Edition en App Service Linux está actualmente en versión preliminar. Esta pila está **no** recomendado para el trabajo de producción. información sobre nuestras pilas de Java SE y Tomcat.

Azure App Service en Linux permite a los desarrolladores de Java para crear, implementar y escalar aplicaciones empresariales de Java (Java EE) en un servicio totalmente administrado basado en Linux.  El entorno de tiempo de ejecución subyacente de Java Enterprise es el servidor de aplicaciones [WildFly](https://wildfly.org/) de código abierto.

[Escala con App Service](#scale-with-app-service)
[Personalizar configuración de servidor de aplicaciones](#customize-application-server-configuration)
[módulos y dependencias](#modules-and-dependencies)
[datos orígenes](#data-sources)
[permiten a los proveedores de mensajes](#enable-messaging-providers)
[configurar almacenamiento en caché de administración de sesión](#configure-session-management-caching)

### <a name="scale-with-app-service"></a>Escalado con App Service

El servidor de aplicaciones WildFly, que se ejecuta en App Service en Linux, se ejecuta en modo independiente, no en una configuración de dominio. Al escalar horizontalmente el plan de App Service, cada instancia de WildFly se configura como un servidor independiente.

 Escale la aplicación vertical u horizontalmente con [reglas de escalado](../../monitoring-and-diagnostics/monitoring-autoscale-get-started.md) y [aumentando el número de instancias](../web-sites-scale.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json).

### <a name="customize-application-server-configuration"></a>Personalización de la configuración del servidor de aplicaciones

Las instancias de la aplicación web no tienen estado, por lo que cada instancia nueva iniciada debe configurarse al inicio para admitir la configuración de Wildfly que necesite la aplicación.
Puede escribir un script de Bash de inicio para llamar a la CLI de WildFly para:

- Configurar orígenes de datos
- Configurar proveedores de mensajería
- Incorporar otros módulos y dependencias a la configuración del servidor WildFly.

 El script se ejecuta cuando WildFly está en funcionamiento, pero antes de que se inicie la aplicación. El script debe usar la [CLI de JBOSS](https://docs.jboss.org/author/display/WFLY/Command+Line+Interface) llamada desde `/opt/jboss/wildfly/bin/jboss-cli.sh` para configurar el servidor de aplicaciones con cualquier configuración o cambios necesarios una vez que se inicie el servidor.

No use el modo interactivo de la CLI para configurar WildFly. En su lugar, puede proporcionar un script de comandos a la CLI de JBoss mediante el comando `--file`, por ejemplo:

```bash
/opt/jboss/wildfly/bin/jboss-cli.sh -c --file=/path/to/your/jboss_commands.cli
```

Cargue el script de inicio `/home/site/deployments/tools` en la instancia de App Service. Consulte [este documento](../deploy-configure-credentials.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#userscope) para obtener instrucciones sobre cómo obtener las credenciales de FTP.

Establezca el campo **Script de inicio** en Azure Portal en la ubicación del script de shell de inicio, por ejemplo `/home/site/deployments/tools/your-startup-script.sh`.

Proporcionar [configuración de la aplicación](../web-sites-configure.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#app-settings) en la configuración de aplicación para pasar variables de entorno para su uso en la secuencia de comandos. La configuración de la aplicación mantiene las cadenas de conexión y otros secretos necesarios para configurar la aplicación fuera del control de versiones.

### <a name="modules-and-dependencies"></a>Módulos y dependencias

Para instalar los módulos y sus dependencias en la ruta de clase WildFly a través de la CLI de JBoss, deberá crear los siguientes archivos en su propio directorio. Algunos módulos y dependencias pueden necesitar una configuración adicional, como la nomenclatura de JNDI u otra configuración específica de la API, por lo que esta lista es un conjunto mínimo de lo que tendrá que configurar de una dependencia en la mayoría de los casos.

- Un [descriptor del módulo XML](https://jboss-modules.github.io/jboss-modules/manual/#descriptors). Este archivo XML define el nombre, los atributos y las dependencias del módulo. En este [archivo module.xml de ejemplo](https://access.redhat.com/documentation/jboss_enterprise_application_platform/6/html/administration_and_configuration_guide/example_postgresql_xa_datasource) se definen un módulo de Postgres, su dependencia JDBC del archivo JAR y otras dependencias del módulo necesarias.
- Las dependencias del archivo JAR necesarias para el módulo.
- Un script con los comandos de la CLI de JBoss para configurar el módulo nuevo. Este archivo contendrá los comandos que ejecutará la CLI de JBoss para configurar el servidor para usar la dependencia. Para obtener documentación sobre los comandos para agregar módulos, orígenes de datos y proveedores de mensajería, consulte [este documento](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.0/html-single/management_cli_guide/#how_to_cli).
- Un script de inicio de Bash para llamar a la CLI de JBoss y ejecutar el script del paso anterior. Este archivo se ejecutará cuando se reinicie la instancia de App Service o cuando se aprovisionen nuevas instancias durante un escalado horizontal. Este script de inicio es donde puede realizar otras configuraciones para la aplicación, ya que los comandos de JBoss se pasan a la CLI de JBoss. Como mínimo, este archivo puede ser un único comando para pasar el script de comandos de la CLI de JBoss a la CLI de JBoss:

```bash
`/opt/jboss/wildfly/bin/jboss-cli.sh -c --file=/path/to/your/jboss_commands.cli`
```

Una vez que tenga los archivos y el contenido para el módulo, siga estos pasos para agregar el módulo al servidor de aplicaciones de WildFly.

1. Cargue en FTP los archivos en `/home/site/deployments/tools` en la instancia de App Service. Consulte este documento para obtener instrucciones sobre cómo obtener las credenciales de FTP.
2. En la hoja Configuración de la aplicación de Azure Portal, establezca el campo "Script de inicio" en Azure Portal en la ubicación del script de shell de inicio, por ejemplo `/home/site/deployments/tools/your-startup-script.sh`.
3. Reinicie la instancia del servicio de la aplicación presionando el **reiniciar** situado en la **información general sobre** sección del portal o mediante la CLI de Azure.

### <a name="configure-data-source-connections"></a>Configurar conexiones a orígenes de datos

Para configurar WildFly para una conexión de orígenes de datos, siga el mismo proceso descrito anteriormente en la sección Módulos y dependencias. Puede seguir los mismos pasos para cualquier servicio de Azure Database.

1. Descargue el controlador JDBC para el tipo de base de datos. Para mayor comodidad, estos son los controladores para [Postgres](https://jdbc.postgresql.org/download.html) y [MySQL](https://dev.mysql.com/downloads/connector/j/). Desempaquete la descarga para obtener el archivo .jar.
2. Siga los pasos detallados en "Módulos y dependencias" para crear y cargar el descriptor del módulo XML, el script de la CLI de JBoss, el script de inicio y la dependencia JDBC de .jar.

Hay disponible más información sobre cómo configurar WildFly con [PostgreSQL](https://developer.jboss.org/blogs/amartin-blog/2012/02/08/how-to-set-up-a-postgresql-jdbc-driver-on-jboss-7), [MySQL](https://docs.jboss.org/jbossas/docs/Installation_And_Getting_Started_Guide/5/html/Using_other_Databases.html#Using_other_Databases-Using_MySQL_as_the_Default_DataSource) y [SQL Database](https://docs.jboss.org/jbossas/docs/Installation_And_Getting_Started_Guide/5/html/Using_other_Databases.html#d0e3898). Puede usar estas instrucciones personalizadas junto con el enfoque generalizado anterior para agregar definiciones de orígenes de datos al servidor.

### <a name="enable-messaging-providers"></a>Habilitar a los proveedores de mensajes

Para habilitar Beans controlados por mensajes usando Service Bus como mecanismo de mensajería:

1. Use la [biblioteca de mensajería de Apache QPId JMS](https://qpid.apache.org/proton/index.html). Incluya esta dependencia en pom.xml (u otro archivo de compilación) de la aplicación.

2. Cree [Recursos de Service Bus](/azure/service-bus-messaging/service-bus-java-how-to-use-jms-api-amqp). Cree un espacio de nombres de Azure Service Bus, una cola dentro de ese espacio de nombres y una directiva de acceso compartido con funcionalidades de envío y recepción.

3. Pase la clave de la directiva de acceso compartido al código codificando la dirección URL de la clave principal de la directiva o [usando el SDK de Service Bus](/azure/service-bus-messaging/service-bus-java-how-to-use-jms-api-amqp#setup-jndi-context-and-configure-the-connectionfactory).

4. Siga los pasos descritos en la sección Módulos y dependencias con su descriptor XML de módulo, las dependencias de .jar, los comandos de la CLI de JBoss y el script de inicio para el proveedor de JMS. Además de los cuatro archivos, también deberá crear un archivo XML que defina el nombre JNDI de la cola de JMS y el tema. Consulte [este repositorio](https://github.com/JasonFreeberg/widlfly-server-configs/tree/master/appconfig) para obtener los archivos de configuración de referencia.

### <a name="configure-session-management-caching"></a>Configuración del almacenamiento en caché de administración de sesiones

De forma predeterminada, App Service en Linux usará las cookies de afinidad de sesión para asegurarse de que las solicitudes de los clientes con sesiones existentes se enrutan a la misma instancia de la aplicación. Este comportamiento predeterminado no requiere configuración, pero tiene algunas limitaciones:

- Si una instancia de aplicación se reinicia o se reduce verticalmente, se perderá el estado de sesión de usuario en el servidor de aplicaciones.
- Si las aplicaciones tienen una configuración de tiempo de espera de sesión es demasiado largo o un número fijo de usuarios, las nuevas instancias escaladas automáticamente pueden tardar en recibir la carga, puesto que solo las sesiones nuevas se enrutarán a las instancias recién iniciadas.

Puede configurar WildFly para usar un almacén de sesión externo, como [Azure Cache for Redis](/azure/azure-cache-for-redis/). Deberá [deshabilitar la configuración de afinidad de instancia existente de ARR](https://azure.microsoft.com/blog/disabling-arrs-instance-affinity-in-windows-azure-web-sites/) para desactivar el enrutamiento basado en cookies de sesión y permitir que el almacén de sesiones de WildFly configurado funcione sin interferencias.

## <a name="docker-containers"></a>Contenedores de Docker

Para utilizar el JDK de Zulu compatible con Azure en los contenedores, asegúrese de que extraer y usar las imágenes pregeneradas como se documenta en la [página de descarga de Azul Zulu Enterprise for Azure admitida](https://www.azul.com/downloads/azure-only/zulu/) o usar los ejemplos `Dockerfile` del [repositorio de GitHub de Java de Microsoft](https://github.com/Microsoft/java/tree/master/docker).

## <a name="statement-of-support"></a>Instrucción de compatibilidad

### <a name="runtime-availability"></a>Disponibilidad en tiempo de ejecución

App Service para Linux es compatible con dos entornos de ejecución para el hospedaje administrado de aplicaciones web Java:

- El [contenedor de servlets Tomcat](https://tomcat.apache.org/) para ejecutar aplicaciones empaquetadas como archivos WAR (archivo web). Las versiones compatibles son 8.5 y 9.0.
- Java SE Runtime Environment para ejecutar aplicaciones como archivos JAR (archivo Java). Versiones compatibles son Java 8 y 11.

### <a name="jdk-versions-and-maintenance"></a>Mantenimiento y versiones de JDK

El JDK (Java Development Kit) compatible con Azure es [Zulu](https://www.azul.com/downloads/azure-only/zulu/) y se distribuye a través de [Azul Systems](https://www.azul.com/).

Las actualizaciones de versión principal se proporcionarán a través de nuevas opciones de entorno de ejecución nuevas en Azure App Service para Linux. Para actualizar a estas versiones más recientes, los clientes deben configurar su implementación de App Service. Asimismo, son responsables de probar y garantizar que la actualización principal satisface sus necesidades.

Los JDK compatibles se revisarán trimestralmente de manera automática en enero, abril, julio y octubre de cada año.

### <a name="security-updates"></a>Actualizaciones de seguridad

Las revisiones y correcciones de las vulnerabilidades de seguridad principales se publicarán en cuanto estén disponibles a través de Azul Systems. Una vulnerabilidad "principal" se define con una puntuación total de 9.0 o superior en el [CVSS (Common Vulnerability Scoring System), versión 2 del NIST](https://nvd.nist.gov/cvss.cfm).

### <a name="deprecation-and-retirement"></a>Desuso y retirada

Si un entorno compatible de Java Runtime se va a retirar, los desarrolladores de Azure que lo usen recibirán un aviso de desuso al menos seis meses antes de su retirada.

### <a name="local-development"></a>Desarrollo local

Los desarrolladores pueden descargar el JDK de Azul Zulu Enterprise Production Edition para el desarrollo local desde el [sitio de descarga de Azul](https://www.azul.com/downloads/azure-only/zulu/).

### <a name="development-support"></a>Soporte para el desarrollo

El soporte técnico para el [JDK de Azul Zulu compatible con Azure](https://www.azul.com/downloads/azure-only/zulu/) está disponible al desarrollar para Azure o [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) con un [plan de soporte técnico de Azure cualificado](https://azure.microsoft.com/support/plans/).

### <a name="runtime-support"></a>Compatibilidad con el tiempo de ejecución

Los desarrolladores pueden [abrir una incidencia](/azure/azure-supportability/how-to-create-azure-support-request) con los JDK de Azul Zulu a través de soporte técnico de Azure si tienen un [plan de soporte técnico completo](https://azure.microsoft.com/support/plans/).

## <a name="next-steps"></a>Pasos siguientes

Visite el centro de [Azure para desarrolladores de Java](/java/azure/) para encontrar guías de inicio rápido de Azure, tutoriales y documentación de referencia de Java.

Encontrará respuestas a las preguntas generales sobre el uso de App Service para Linux que no son específicas del desarrollo de Java en las [Preguntas más frecuentes sobre Azure App Service en Linux](app-service-linux-faq.md).

