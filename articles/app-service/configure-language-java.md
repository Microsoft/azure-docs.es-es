---
title: Configuración de aplicaciones Java en Windows
description: Aprenda a configurar aplicaciones de Java para que se ejecuten en las instancias de máquina virtual de Windows en Azure App Service. En este artículo se muestran las tareas de configuración más comunes.
keywords: azure app service, aplicación web, windows, oss, java
author: jasonfreeberg
ms.devlang: java
ms.topic: article
ms.date: 04/12/2019
ms.author: jafreebe
ms.reviewer: cephalin
ms.custom: seodec18, devx-track-java
ms.openlocfilehash: d0850b2192df1392a245f27588ffcb8ddb337347
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319344"
---
# <a name="configure-a-windows-java-app-for-azure-app-service"></a>Configuración de una aplicación de Java en Windows para Azure App Service

Azure App Service permite a los desarrolladores de Java compilar, implementar y escalar rápidamente sus aplicaciones web de Tomcat en un servicio basado en Windows totalmente administrado. Implemente aplicaciones con complementos de Maven desde la línea de comandos o en editores, como IntelliJ, Eclipse o Visual Studio Code.

Esta guía incluye conceptos clave e instrucciones para los desarrolladores de Java que usan App Service. Si nunca ha usado Azure App Service, debe leer primero la [Guía de inicio rápido de Java](app-service-web-get-started-java.md). Encontrará respuestas a las preguntas generales sobre el uso de App Service que no son específicas del desarrollo de Java en las [Preguntas más frecuentes sobre Azure App Service en Windows](faq-configuration-and-management.md).

## <a name="deploying-your-app"></a>Implementación de la aplicación

Puede usar el [complemento de Azure Web App para Maven](/java/api/overview/azure/maven/azure-webapp-maven-plugin/readme) para implementar los archivos .war. La implementación con entornos de desarrollo integrado populares también se admite con [Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/) o [Azure Toolkit for Eclipse](/azure/developer/java/toolkit-for-eclipse).

De lo contrario, el método de implementación dependerá del tipo de archivo:

- Para implementar archivos .war en Tomcat, utilice el punto de conexión `/api/wardeploy/` para realizar el conjunto de rutinas POST en el archivo. Para obtener más información sobre esta API, consulte [este documento](https://docs.microsoft.com/azure/app-service/deploy-zip#deploy-war-file).
- Para implementar archivos .jar en Java SE, use el punto de conexión `/api/zipdeploy/` del sitio de Kudu. Para obtener más información sobre esta API, consulte [este documento](https://docs.microsoft.com/azure/app-service/deploy-zip#rest).

No implemente el archivo .war mediante FTP. La herramienta FTP está diseñada para cargar los scripts de inicio, dependencias u otros archivos en tiempo de ejecución. Tenga en cuenta que no es la opción óptima para realizar la implementación de aplicaciones web.

## <a name="logging-and-debugging-apps"></a>Registro y depuración de aplicaciones

Encontrará informes de rendimiento, visualizaciones de tráfico y comprobaciones de mantenimiento de cada aplicación a través de Azure Portal. Para más información, consulte [Introducción a los diagnósticos de Azure App Service](overview-diagnostics.md).

### <a name="use-flight-recorder"></a>Uso de Flight Recorder

Todos los entornos de ejecución de Java en App Service que usan JVM de Azul incluyen Zulu Flight Recorder. Puede usarlo para registrar eventos de JVM, del sistema y de nivel de Java para supervisar el comportamiento y solucionar problemas en sus aplicaciones Java.

Para realizar una grabación temporizada, necesitará el PID (identificador de proceso) de la aplicación Java. Para encontrar el PID, abra el sitio de SCM de su aplicación web en un explorador: https://<nombre-de-su-sitio>.scm.azurewebsites.net/ProcessExplorer/. En esta página se muestran los procesos en ejecución en la aplicación web. Busque el proceso denominado "Java" en la tabla y copie el PID (identificador de proceso) correspondiente.

A continuación, abra la **Consola de depuración** en la barra de herramientas superior del sitio de SCM y ejecute el siguiente comando. Reemplace `<pid>` por el identificador de proceso que ha copiado anteriormente. Este comando iniciará una grabación de 30 segundos del generador de perfiles de su aplicación Java y generará un archivo denominado `timed_recording_example.jfr` en el directorio `D:\home`.

```
jcmd <pid> JFR.start name=TimedRecording settings=profile duration=30s filename="D:\home\timed_recording_example.JFR"
```

Para más información, consulte la [referencia del comando Jcmd ](https://docs.oracle.com/javacomponents/jmc-5-5/jfr-runtime-guide/comline.htm#JFRRT190).

#### <a name="analyze-jfr-files"></a>Análisis de archivos `.jfr`

Use [FTPS](deploy-ftp.md) para descargar el archivo JFR en el equipo local. Para analizar el archivo JFR, descargue e instale [Zulu Mission Control](https://www.azul.com/products/zulu-mission-control/). Para obtener instrucciones acerca de Zulu Mission Control, consulte la [documentación de Azul](https://docs.azul.com/zmc/) y [las instrucciones de instalación](https://docs.microsoft.com/java/azure/jdk/java-jdk-flight-recorder-and-mission-control).

### <a name="stream-diagnostic-logs"></a>Transmisión de registros de diagnóstico

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

Para más información, consulte [Registros de secuencias en Cloud Shell](troubleshoot-diagnostic-logs.md#in-cloud-shell).

### <a name="app-logging"></a>Registro de aplicaciones

Habilite el [registro de aplicaciones](troubleshoot-diagnostic-logs.md#enable-application-logging-windows) a través de Azure Portal o la [CLI de Azure](/cli/azure/webapp/log#az-webapp-log-config) para configurar App Service para que escriba los flujos de salida y de error de la consola estándar en el sistema de archivos local o en Azure Blob Storage. El registro en el sistema de archivos local de App Service se deshabilita 12 horas después de su configuración. Si necesita una retención más prolongada, configure la aplicación para escribir la salida en un contenedor de Blob Storage. Los registros de aplicación de Java y Tomcat pueden encontrarse en el directorio */LogFiles/Application/* .

Si la aplicación usa [Logback](https://logback.qos.ch/) o [Log4j](https://logging.apache.org/log4j) para el seguimiento, puede reenviar estos seguimientos para su revisión en Azure Application Insights mediante las instrucciones de configuración del marco de registro en [Exploración de los registros de seguimiento de Java en Application Insights](/azure/application-insights/app-insights-java-trace-logs).


## <a name="customization-and-tuning"></a>Personalización y optimización

Azure App Service admite la optimización y la personalización de serie a través de Azure Portal y de la CLI de Azure. Consulte los artículos siguientes para conocer la configuración de aplicaciones web específicas que no son de Java:

- [Configuración de aplicaciones](configure-common.md#configure-app-settings)
- [Configuración de un dominio personalizado](app-service-web-tutorial-custom-domain.md)
- [Configuración de enlaces TLS](configure-ssl-bindings.md)
- [Adición de una red CDN](../cdn/cdn-add-to-web-app.md)
- [Configuración del sitio de Kudu](https://github.com/projectkudu/kudu/wiki/Configurable-settings)

### <a name="set-java-runtime-options"></a>Definición de las opciones de Java Runtime

Para establecer la memoria asignada u otras opciones de runtime de JVM, cree una [configuración de la aplicación](configure-common.md#configure-app-settings) denominada `JAVA_OPTS` con las opciones. App Service pasa esta configuración como variable de entorno para Java Runtime cuando se inicia.

En Azure Portal, en **Configuración de la aplicación** para la aplicación web, cree un nuevo valor de la aplicación denominado `JAVA_OPTS` que incluya valores de configuración adicionales, como `-Xms512m -Xmx1204m`.

Para definir la configuración de la aplicación desde el complemento MavenLinux, agregue las etiquetas setting/value a la sección de complementos de Azure. En el ejemplo siguiente se establece un tamaño del montón de Java mínimo y máximo específico:

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
- Instancias de S2 y B2: `-Xms3072m -Xmx3072m`
- Instancias de S3 y B3: `-Xms6144m -Xmx6144m`

Cuando optimice la configuración del montón de la aplicación, revise los detalles de su plan de App Service y tenga en cuenta distintas necesidades de aplicaciones y ranuras de implementación para encontrar la asignación óptima de memoria.

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

### <a name="pre-compile-jsp-files"></a>Precompilación de archivos JSP

Para mejorar el rendimiento de las aplicaciones Tomcat, puede compilar los archivos JSP antes de realizar la implementación en App Service. Puede usar el [complemento Maven](https://sling.apache.org/components/jspc-maven-plugin/plugin-info.html) proporcionado por Apache Sling o este [archivo de compilación Ant](https://tomcat.apache.org/tomcat-9.0-doc/jasper-howto.html#Web_Application_Compilation).

## <a name="secure-applications"></a>Aplicaciones seguras

Las aplicaciones de Java que se ejecutan en App Service presentan el mismo conjunto de [procedimientos recomendados de seguridad](/azure/security/security-paas-applications-using-app-services) que otras aplicaciones.

### <a name="authenticate-users-easy-auth"></a>Autenticación de usuarios (autenticación sencilla)

Configure la autenticación de la aplicación en Azure Portal con la opción **Autenticación y autorización**. Desde allí, puede habilitar la autenticación con Azure Active Directory o con inicios de sesión en redes sociales como Facebook, Google o GitHub. La configuración de Azure Portal solo funciona al configurar un proveedor de autenticación único. Para obtener más información, consulte [Configuración de una aplicación de App Service para usar el inicio de sesión de Azure Active Directory](configure-authentication-provider-aad.md) y los artículos relacionados de otros proveedores de identidades. Si tiene que habilitar varios proveedores de inicio de sesión, siga las instrucciones del artículo sobre la [personalización de la autenticación en App Service](app-service-authentication-how-to.md).

#### <a name="tomcat"></a>Tomcat

La aplicación Tomcat puede acceder a las reclamaciones del usuario directamente desde el servlet mediante la conversión del objeto Principal a un objeto Map. Este último objeto asignará ca tipo de reclamación a una colección de las notificaciones de dicho tipo. En el código siguiente, `request` es una instancia de `HttpServletRequest`.

```java
Map<String, Collection<String>> map = (Map<String, Collection<String>>) request.getUserPrincipal();
```

Ahora puede inspeccionar el objeto `Map` de cualquier notificación específica. Por ejemplo, el fragmento de código siguiente recorre en iteración todos los tipos de notificación e imprime el contenido de cada colección.

```java
for (Object key : map.keySet()) {
        Object value = map.get(key);
        if (value != null && value instanceof Collection {
            Collection claims = (Collection) value;
            for (Object claim : claims) {
                System.out.println(claims);
            }
        }
    }
```

Para cerrar la sesión de los usuarios, utilice la ruta de acceso `/.auth/ext/logout`. Para realizar otras acciones, consulte la documentación acerca del [uso de la autenticación y de la autorización de App Service](https://docs.microsoft.com/azure/app-service/app-service-authentication-how-to). También hay documentación oficial acerca de la [interfaz HttpServletRequest](https://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpServletRequest.html) y sus métodos. Los siguientes métodos de servlet también se hidrata métodos según la configuración de App Service:

```java
public boolean isSecure()
public String getRemoteAddr()
public String getRemoteHost()
public String getScheme()
public int getServerPort()
```

Para deshabilitar esta característica, cree una configuración de aplicación denominada `WEBSITE_AUTH_SKIP_PRINCIPAL` con un valor de `1`. Para deshabilitar todos los filtros de servlet que ha agregado App Service, cree una configuración denominada `WEBSITE_SKIP_FILTERS` con un valor de `1`.

### <a name="configure-tlsssl"></a>Configuración de TLS/SSL

Siga las instrucciones de [Protección de un nombre DNS personalizado con un enlace TLS en Azure App Service](configure-ssl-bindings.md) para cargar un certificado TLS/SSL y enlazarlo al nombre de dominio de la aplicación. De forma predeterminada, la aplicación seguirá permitiendo que las conexiones HTTP sigan los pasos específicos del tutorial para aplicar SSL y TLS.

### <a name="use-keyvault-references"></a>Uso de referencias de KeyVault

[Azure KeyVault](../key-vault/general/overview.md) proporciona administración de secretos centralizada con directivas de acceso un historial de directivas. En KeyVault puede almacenar secretos (como contraseñas o cadenas de conexión) y acceder a ellos en la aplicación través de las variables de entorno.

En primer lugar, siga las instrucciones para [conceder a su aplicación acceso a Key Vault](app-service-key-vault-references.md#granting-your-app-access-to-key-vault) y [hace una referencia de KeyVault a su secreto en una configuración de aplicación](app-service-key-vault-references.md#reference-syntax). Para validar que la referencia se resuelve en el secreto, imprima la variable de entorno mie3ntras acceder de forma remota al terminal de App Service.

Para insertar estos secretos en el archivo de configuración de Spring o Tomcat, use la sintaxis de inserción de variables de entorno (`${MY_ENV_VAR}`). En el caso de los archivos de configuración de Spring, consulte esta documentación acerca de [configuraciones externalizadas](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html).


## <a name="configure-apm-platforms"></a>Configuración de plataformas APM

En esta sección se muestra cómo conectar aplicaciones Java implementadas en Azure App Service en Linux con las plataformas de supervisión de aplicaciones (APM) NewRelic y AppDynamics.

### <a name="configure-new-relic"></a>Configuración de New Relic

1. Cree una cuenta de New Relic en [NewRelic.com](https://newrelic.com/signup).
2. Descargue el agente de Java desde NewRelic; tendrá un nombre de archivo similar a *newrelic-java-x.x.x.zip*.
3. Copie la clave de licencia, ya que la necesitará más tarde para configurar el agente.
4. Use la [consola de Kudu](https://github.com/projectkudu/kudu/wiki/Kudu-console) para crear un directorio */home/site/wwwroot/apm*.
5. Cargue los archivos desempaquetados del agente de Java de New Relic en un directorio de */home/site/wwwroot/apm*. Los archivos del agente deben estar en */home/site/wwwroot/apm/newrelic*.
6. Modifique el archivo YAML en */home/site/wwwroot/apm/newrelic/newrelic.yml* y reemplace el marcador de posición de valor de la licencia por su propia clave de licencia.
7. En Azure Portal, vaya a la aplicación en App Service y cree una nueva configuración de la aplicación.
    - Si la aplicación usa **Java SE**, cree una variable de entorno llamada `JAVA_OPTS` con el valor `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar`.
    - Si usa **Tomcat**, cree una variable de entorno llamada `CATALINA_OPTS` con el valor `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar`.

### <a name="configure-appdynamics"></a>Configuración de AppDynamics

1. Cree una cuenta de AppDynamics en [AppDynamics.com](https://www.appdynamics.com/community/register/).
2. Descargue el agente de Java desde el sitio web de AppDynamics; el nombre de archivo será similar a *AppServerAgent-x.x.x.xxxxx.zip*
3. Use la [consola de Kudu](https://github.com/projectkudu/kudu/wiki/Kudu-console) para crear un directorio */home/site/wwwroot/apm*.
4. Cargue los archivos del agente de Java en un directorio de */home/site/wwwroot/apm*. Los archivos del agente deben estar en */home/site/wwwroot/apm/appdynamics*.
5. En Azure Portal, vaya a la aplicación en App Service y cree una nueva configuración de la aplicación.
    - Si usa **Java SE**, cree una variable de entorno llamada `JAVA_OPTS` con el valor `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>` donde `<app-name>` es el nombre de su instancia de App Service.
    - Si usa **Tomcat**, cree una variable de entorno llamada `CATALINA_OPTS` con el valor `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>` donde `<app-name>` es el nombre de su instancia de App Service.

>  Si ya tiene una variable de entorno para `JAVA_OPTS` o `CATALINA_OPTS`, anexe la opción `-javaagent:/...` al final del valor actual.

## <a name="data-sources"></a>Orígenes de datos

### <a name="tomcat"></a>Tomcat

Estas instrucciones se aplican a todas las conexiones de base de datos. Deberá rellenar los marcadores de posición con el nombre de clase de controlador de la base de datos elegido y con el archivo JAR. Se proporciona una tabla con los nombres de clase y las descargas de controladores de las bases de datos más habituales.

| Base de datos   | Nombre de clase de controlador                             | Controlador JDBC                                                                      |
|------------|-----------------------------------------------|------------------------------------------------------------------------------------------|
| PostgreSQL | `org.postgresql.Driver`                        | [Descargar](https://jdbc.postgresql.org/download.html)                                    |
| MySQL      | `com.mysql.jdbc.Driver`                        | [Descargar](https://dev.mysql.com/downloads/connector/j/) (seleccione "Platform Independent") |
| SQL Server | `com.microsoft.sqlserver.jdbc.SQLServerDriver` | [Descargar](https://docs.microsoft.com/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-2017#download)                                                           |

Para configurar Tomcat para que use Java Database Connectivity (JDBC) o Java Persistence API (JPA), personalice primero la variable de entorno `CATALINA_OPTS` que lee Tomcat al iniciarse. Establezca estos valores a través de un valor de la aplicación en el [complemento de Maven de App Service](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md):

```xml
<appSettings>
    <property>
        <name>CATALINA_OPTS</name>
        <value>"$CATALINA_OPTS -Ddbuser=${DBUSER} -Ddbpassword=${DBPASSWORD} -DconnURL=${CONNURL}"</value>
    </property>
</appSettings>
```

O bien establezca las variables de entorno en la página **Configuración** > **Configuración de la aplicación** de Azure Portal.

A continuación, determine si el origen de datos debe estar disponible para una aplicación o para todas las aplicaciones que se ejecutan en el servlet de Tomcat.

#### <a name="application-level-data-sources"></a>Orígenes de datos de nivel de aplicación

1. Cree un archivo *context.xml* en el directorio *META-INF /* del proyecto. Si el directorio *META-INF/* no existe, créelo.

2. En *, context.xml*, agregue un elemento `Context` para vincular el origen de datos a una dirección JNDI. Reemplace el marcador de posición `driverClassName` por el nombre de clase de su controlador de la tabla anterior.

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

3. Actualice el archivo *web.xml* de la aplicación para que use el origen de datos de su aplicación.

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/dbconnection</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

#### <a name="finalize-configuration"></a>Finalización de la configuración

Por último, coloque los archivos JAR del controlador en la classpath de Tomcat y reinicie App Service. Asegúrese de que los archivos del controlador JDBC estén disponibles para el cargador de clases de Tomcat. Para ello, colóquelos en el directorio */home/tomcat/lib*. (cree el directorio si no existe). Para cargar estos archivos en su instancia de App Service, realice los pasos siguientes:

1. En el [Cloud Shell](https://shell.azure.com), instale la extensión webapp:

    ```azurecli-interactive
    az extension add -–name webapp
    ```

2. Ejecute el siguiente comando de la CLI para crear un túnel SSH desde el sistema local a App Service:

    ```azurecli-interactive
    az webapp remote-connection create --resource-group <resource-group-name> --name <app-name> --port <port-on-local-machine>
    ```

3. Establezca la conexión al puerto de tunelización local con el cliente SFTP y cargue los archivos en la carpeta */home/tomcat/lib*.

Como alternativa, puede usar un cliente FTP para cargar el controlador JDBC. Siga estas [instrucciones para obtener las credenciales de FTP](deploy-configure-credentials.md).

## <a name="configuring-tomcat"></a>Configuración de Tomcat

Para editar `server.xml` de Tomcat u otros archivos de configuración, en primer lugar anote su versión principal de Tomcat en el portal.

1. Encuentre el directorio principal de Tomcat de su versión ejecutando el comando `env`. Busque la variable de entorno que comienza por `AZURE_TOMCAT` y coincide con la versión principal. Por ejemplo, `AZURE_TOMCAT85_HOME` señala al directorio de Tomcat para Tomcat 8.5.
1. Una vez que haya identificado el directorio principal de Tomcat para su versión, copie el directorio de configuración para `D:\home`. Por ejemplo, si `AZURE_TOMCAT85_HOME` tuviera un valor de `D:\Program Files (x86)\apache-tomcat-8.5.37`, la nueva ruta de acceso del directorio de configuración copiado sería `D:\home\apache-tomcat-8.5.37`.

Finalmente, reinicie App Service. Las implementaciones deben ir a `D:\home\site\wwwroot\webapps` igual que antes.

## <a name="configure-java-se"></a>Configuración de Java SE

Al ejecutar una aplicación .JAR en Java SE en Windows, `server.port` se pasa como una opción de línea de comandos cuando se inicia la aplicación. Puede resolver manualmente el puerto HTTP desde la variable de entorno `HTTP_PLATFORM_PORT`. El valor de esta variable del entorno será el puerto HTTP en que debe escuchar la aplicación. 

## <a name="java-runtime-statement-of-support"></a>Instrucción de compatibilidad de Java Runtime

### <a name="jdk-versions-and-maintenance"></a>Mantenimiento y versiones de JDK

El JDK (Java Development Kit) compatible con Azure es [Zulu](https://www.azul.com/downloads/azure-only/zulu/) y se distribuye a través de [Azul Systems](https://www.azul.com/).

Las actualizaciones de versión principal se proporcionarán a través de nuevas opciones de entorno de ejecución en Azure App Service para Windows. Para actualizar a estas versiones más recientes, los clientes deben configurar su implementación de App Service. Asimismo, son responsables de probar y garantizar que la actualización principal satisface sus necesidades.

Los JDK compatibles se revisarán trimestralmente de manera automática en enero, abril, julio y octubre de cada año. Para obtener más información sobre Java en Azure, consulte [este documento de soporte técnico](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-long-term-support).

### <a name="security-updates"></a>Actualizaciones de seguridad

Las revisiones y correcciones de las vulnerabilidades de seguridad principales se publicarán en cuanto estén disponibles a través de Azul Systems. Una vulnerabilidad "principal" se define con una puntuación total de 9.0 o superior en el [CVSS (Common Vulnerability Scoring System), versión 2 del NIST](https://nvd.nist.gov/vuln-metrics/cvss).

Tomcat 8.0 alcanzó el [fin de la vida útil el 30 de septiembre de 2018](https://tomcat.apache.org/tomcat-80-eol.html). Aunque el entorno en tiempo de ejecución sigue disponible en Azure App Service, Azure no aplicará actualizaciones de seguridad a Tomcat 8.0. Si es posible, migre las aplicaciones a Tomcat 8.5 o 9.0. Tanto Tomcat 8.5 como 9.0 están disponibles en Azure App Service. Para más información, consulte el [sitio oficial de Tomcat](https://tomcat.apache.org/whichversion.html). 

### <a name="deprecation-and-retirement"></a>Desuso y retirada

Si un entorno compatible de Java Runtime se va a retirar, los desarrolladores de Azure que lo usen recibirán un aviso de desuso al menos seis meses antes de su retirada.

### <a name="local-development"></a>Desarrollo local

Los desarrolladores pueden descargar el JDK de Azul Zulu Enterprise Production Edition para el desarrollo local desde el [sitio de descarga de Azul](https://www.azul.com/downloads/azure-only/zulu/).

### <a name="development-support"></a>Soporte para el desarrollo

El soporte técnico para el JDK de [Azul Zulu compatible con Azure](https://www.azul.com/downloads/azure-only/zulu/) está disponible mediante Microsoft al desarrollar para Azure o [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) con un [plan de soporte técnico de Azure cualificado](https://azure.microsoft.com/support/plans/).

### <a name="runtime-support"></a>Compatibilidad con el tiempo de ejecución

Los desarrolladores pueden [abrir una incidencia](/azure/azure-portal/supportability/how-to-create-azure-support-request) con los JDK de Azul Zulu a través de soporte técnico de Azure si tienen un [plan de soporte técnico completo](https://azure.microsoft.com/support/plans/).

## <a name="next-steps"></a>Pasos siguientes

En este tema se proporciona la instrucción de compatibilidad de Java Runtime para Azure App Service en Windows.

- Para más información sobre el hospedaje de aplicaciones web con Azure App Service, consulte [Información general de App Service](overview.md).
- Para obtener información acerca del desarrollo de Java en Azure, visite el [centro para desarrolladores de Azure para Java](https://docs.microsoft.com/java/azure/?view=azure-java-stable).
