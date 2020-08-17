---
title: 'Supervisión de aplicaciones Java en ejecución en cualquier entorno: Application Insights de Azure Monitor'
description: Supervisión de rendimiento de aplicaciones para aplicaciones Java que se ejecutan en cualquier entorno con el agente independiente de Java sin instrumentar la aplicación. Seguimiento distribuido y mapa de aplicación.
ms.topic: conceptual
ms.date: 04/16/2020
ms.custom: devx-track-java
ms.openlocfilehash: b1a23c01fdf8a0145c26b709f36709a7b710103f
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374310"
---
# <a name="configuring-jvm-args-java-standalone-agent-for-azure-monitor-application-insights"></a>Configuración de los argumentos de JVM del agente independiente de Java para Application Insights de Azure Monitor



## <a name="azure-environments"></a>Entornos de Azure

Configure [App Services](../../app-service/configure-language-java.md#set-java-runtime-options).

## <a name="spring-boot"></a>Spring Boot

Agregue el argumento de JVM `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar` en alguna parte antes de `-jar`, por ejemplo:

```
java -javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar -jar <myapp.jar>
```

## <a name="spring-boot-via-docker-entry-point"></a>Spring Boot mediante el punto de entrada de Docker

Si usa el formato *exec*, agregue el parámetro `"-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar"` a la lista de parámetros en algún lugar antes del parámetro `"-jar"`, por ejemplo:

```
ENTRYPOINT ["java", "-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar", "-jar", "<myapp.jar>"]
```

Si usa el formato *shell*, agregue el argumento `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar` de JVM en algún lugar antes de `-jar`, por ejemplo:

```
ENTRYPOINT java -javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar -jar <myapp.jar>
```

## <a name="tomcat-8-linux"></a>Tomcat 8 (Linux)

### <a name="tomcat-installed-via-apt-get-or-yum"></a>Tomcat instalado mediante `apt-get` o `yum`

Si instaló Tomcat mediante `apt-get` o `yum`, debe tener un archivo `/etc/tomcat8/tomcat8.conf`.  Agregue esta línea al final del archivo:

```
JAVA_OPTS="$JAVA_OPTS -javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar"
```

### <a name="tomcat-installed-via-download-and-unzip"></a>Tomcat instalado mediante descarga y descompresión

Si instaló Tomcat mediante la descarga y descompresión de [https://tomcat.apache.org](https://tomcat.apache.org), debe tener un archivo denominado `<tomcat>/bin/catalina.sh`.  Cree un nuevo archivo en el mismo directorio denominado `<tomcat>/bin/setenv.sh` con el siguiente contenido:

```
CATALINA_OPTS="$CATALINA_OPTS -javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar"
```

Si el archivo `<tomcat>/bin/setenv.sh` ya existe, modifíquelo y agregue `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar` a `CATALINA_OPTS`.


## <a name="tomcat-8-windows"></a>Tomcat 8 (Windows)

### <a name="running-tomcat-from-the-command-line"></a>Ejecución de Tomcat desde la línea de comandos

Busque el archivo `<tomcat>/bin/catalina.bat`.  Cree un nuevo archivo en el mismo directorio denominado `<tomcat>/bin/setenv.bat` con el siguiente contenido:

```
set CATALINA_OPTS=%CATALINA_OPTS% -javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar
```

Las comillas no son necesarias, pero si quiere incluirlas, la ubicación adecuada es la siguiente:

```
set "CATALINA_OPTS=%CATALINA_OPTS% -javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar"
```

Si el archivo `<tomcat>/bin/setenv.bat` ya existe, simplemente modifíquelo y agregue `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar` a `CATALINA_OPTS`.

### <a name="running-tomcat-as-a-windows-service"></a>Ejecución de Tomcat como servicio de Windows

Busque el archivo `<tomcat>/bin/tomcat8w.exe`.  Ejecute el archivo ejecutable y agregue `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar` a la sección `Java Options` de la pestaña `Java`.


## <a name="jboss-eap-7"></a>JBoss EAP 7

### <a name="standalone-server"></a>Servidor independiente

Agregue `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar` a la variable de entorno `JAVA_OPTS` existente en el archivo `JBOSS_HOME/bin/standalone.conf` (Linux) o `JBOSS_HOME/bin/standalone.conf.bat` (Windows):

```java    ...
    JAVA_OPTS="<b>-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar</b> -Xms1303m -Xmx1303m ..."
    ...
```

### <a name="domain-server"></a>Servidor del dominio

Agregue `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar` al elemento `jvm-options` existente en el archivo `JBOSS_HOME/domain/configuration/host.xml`:

```xml
...
<jvms>
    <jvm name="default">
        <heap size="64m" max-size="256m"/>
        <jvm-options>
            <option value="-server"/>
            <!--Add Java agent jar file here-->
            <option value="-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar"/>
            <option value="-XX:MetaspaceSize=96m"/>
            <option value="-XX:MaxMetaspaceSize=256m"/>
        </jvm-options>
    </jvm>
</jvms>
...
```

Si ejecuta varios servidores administrados en un solo host, deberá agregar `applicationinsights.agent.id` al elemento `system-properties` de cada `server`:

```xml
...
<servers>
    <server name="server-one" group="main-server-group">
        <!--Edit system properties for server-one-->
        <system-properties> 
            <property name="applicationinsights.agent.id" value="..."/>
        </system-properties>
    </server>
    <server name="server-two" group="main-server-group">
        <socket-bindings port-offset="150"/>
        <!--Edit system properties for server-two-->
        <system-properties>
            <property name="applicationinsights.agent.id" value="..."/> 
        </system-properties>
    </server>
</servers>
...
```

El valor de `applicationinsights.agent.id` especificado debe ser único. Se usa para crear un subdirectorio en el directorio applicationinsights, ya que cada proceso de JVM necesita su propia configuración de applicationinsights y su propio archivo de registro applicationinsights local. Además, si se informa al recopilador central, el archivo `applicationinsights.properties` se comparte entre varios servidores administrados, por lo que el valor `applicationinsights.agent.id` especificado es necesario para invalidar la configuración `agent.id` en dicho archivo compartido. `applicationinsights.agent.rollup.id` se puede especificar de manera similar en el elemento `system-properties` del servidor si necesita invalidar la configuración `agent.rollup.id` de cada servidor administrado.


## <a name="jetty-9"></a>Jetty 9

Agregue estas líneas al archivo `start.ini`.

```
--exec
-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar
```


## <a name="payara-5"></a>Payara 5

Agregue `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar` al elemento `jvm-options` existente en el archivo `glassfish/domains/domain1/config/domain.xml`:

```xml
...
<java-config ...>
    <!--Edit the JVM options here-->
    <jvm-options>
        -javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar>
    </jvm-options>
        ...
</java-config>
...
```

## <a name="websphere-8"></a>WebSphere 8

Abra la consola de administración, vaya a **Servidores > Servidores de aplicaciones WebSphere > Servidores de aplicaciones**, elija los servidores de aplicaciones adecuados y haga clic en: 

```
Java and Process Management > Process definition >  Java Virtual Machine
```
En "Argumentos de la JVM genéricos", agregue lo siguiente:
```
-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar
```
Después, guarde y reinicie el servidor de aplicaciones.


## <a name="openliberty-18"></a>OpenLiberty 18

Cree un nuevo archivo `jvm.options` en el directorio del servidor (por ejemplo, `<openliberty>/usr/servers/defaultServer`) y agregue la siguiente línea:
```
-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.jar
```
