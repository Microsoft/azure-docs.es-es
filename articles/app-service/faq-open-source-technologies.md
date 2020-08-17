---
title: Preguntas más frecuentes sobre las tecnologías de código abierto
description: Conozca las respuestas a las preguntas más frecuentes sobre las tecnologías de código abierto de Azure App Service.
author: genlin
manager: dcscontentpm
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.topic: article
ms.date: 10/31/2018
ms.author: genli
ms.custom: seodec18, devx-track-python
ms.openlocfilehash: 89c317e6623a868fd75b09274c726720e726e470
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848645"
---
# <a name="open-source-technologies-faqs-for-web-apps-in-azure"></a>Preguntas más frecuentes sobre tecnologías de código abierto para Web Apps en Azure

En este artículo se proporcionan las respuestas a las preguntas más frecuentes sobre los problemas relacionados con las tecnologías de código abierto en la [característica Web Apps de Azure App Service](https://azure.microsoft.com/services/app-service/web/).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="how-do-i-turn-on-php-logging-to-troubleshoot-php-issues"></a>¿Cómo se activa el registro PHP para solucionar problemas de PHP?

Para activar el registro de PHP:

1. Inicie sesión en su **sitio web de Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
2. En el menú superior, elija **Consola de depuración** > **CMD**.
3. Seleccione la carpeta **Sitio**.
4. Seleccione la carpeta **wwwroot**.
5. Seleccione el icono **+** y, después, **Nuevo archivo**.
6. Establezca el nombre del archivo en **.user.ini**.
7. Seleccione el icono de lápiz situado junto a **.user.ini**.
8. En el archivo, agregue este código: `log_errors=on`
9. Seleccione **Guardar**.
10. Seleccione el icono de lápiz situado junto a **wp-config.php**.
11. Cambie el texto por el código siguiente:
    ```php
    //Enable WP_DEBUG modedefine('WP_DEBUG', true);//Enable debug logging to /wp-content/debug.logdefine('WP_DEBUG_LOG', true);
    //Suppress errors and warnings to screendefine('WP_DEBUG_DISPLAY', false);//Suppress PHP errors to screenini_set('display_errors', 0);
    ```
12. En Azure Portal, en el menú de la aplicación web, reinicie la aplicación.

Para obtener más información, vea [Enable WordPress error logs](https://blogs.msdn.microsoft.com/azureossds/2015/10/09/logging-php-errors-in-wordpress-2/) (Habilitar los registros de errores de WordPress).

## <a name="how-do-i-log-python-application-errors-in-apps-that-are-hosted-in-app-service"></a>¿Cómo se pueden registrar errores de aplicaciones de Python en las aplicaciones hospedadas en App Service?
[!INCLUDE [web-sites-python-troubleshooting-wsgi-error-log](../../includes/web-sites-python-troubleshooting-wsgi-error-log.md)]

## <a name="how-do-i-change-the-version-of-the-nodejs-application-that-is-hosted-in-app-service"></a>¿Cómo se cambia la versión de la aplicación Node.js que se hospeda en App Service?

Para cambiar la versión de la aplicación Node.js, puede usar una de las siguientes opciones:

* En Azure Portal, use **Configuración de la aplicación**.
  1. En Azure Portal, vaya a la aplicación web.
  2. En la hoja **Configuración**, seleccione **Configuración de la aplicación**.
  3. En **Configuración de la aplicación**, puede incluir WEBSITE_NODE_DEFAULT_VERSION como clave y la versión de Node.js que quiera como valor.
  4. Vaya a la **consola de Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
  5. Para comprobar la versión de Node.js, escriba el comando siguiente:  
     ```
     node -v
     ```
* Modifique el archivo iisnode.yml. Si cambia la versión de Node.js en el archivo iisnode.yml, solo establecerá el entorno en tiempo de ejecución que usa iisnode. El comando de Kudu y otros comandos seguirán usando la versión de Node.js que esté establecida en **Configuración de la aplicación** en Azure Portal.

  Para establecer manualmente iisnode.yml, cree un archivo iisnode.yml en la carpeta raíz de la aplicación. En el archivo, incluya la línea siguiente:
  ```yml
  nodeProcessCommandLine: "D:\Program Files (x86)\nodejs\5.9.1\node.exe"
  ```
   
* Establezca el archivo iisnode.yml mediante package.json durante la implementación de control de código fuente.
  El proceso de implementación de control de código fuente de Azure sigue estos pasos:
  1. Mueve el contenido a la aplicación web de Azure.
  2. Crea un script de implementación predeterminado, si todavía no hay ninguno (archivos deploy.cmd o .deployment) en la carpeta raíz de la aplicación web.
  3. Ejecuta un script de implementación en el que crea un archivo iisnode.yml si hace referencia a la versión de Node.js en el archivo package.json > motor `"engines": {"node": "5.9.1","npm": "3.7.3"}`
  4. El archivo iisnode.yml tiene la siguiente línea de código:
      ```yml
      nodeProcessCommandLine: "D:\Program Files (x86)\nodejs\5.9.1\node.exe"
      ```

## <a name="i-see-the-message-error-establishing-a-database-connection-in-my-wordpress-app-thats-hosted-in-app-service-how-do-i-troubleshoot-this"></a>Veo el mensaje "Error al establecer una conexión de base de datos" en mi aplicación de WordPress que está hospedada en App Service. ¿Cómo se soluciona este problema?

Si ve este error en la aplicación de WordPress de Azure, para habilitar php_errors.log y debug.log, complete los pasos que se indican en [Enable WordPress error logs](https://blogs.msdn.microsoft.com/azureossds/2015/10/09/logging-php-errors-in-wordpress-2/) (Habilitar los registros de errores de WordPress).

Cuando los registros estén habilitados, reproduzca el error y compruebe los registros para ver si se está quedando sin conexiones:
```
[09-Oct-2015 00:03:13 UTC] PHP Warning: mysqli_real_connect(): (HY000/1226): User ‘abcdefghijk79' has exceeded the ‘max_user_connections’ resource (current value: 4) in D:\home\site\wwwroot\wp-includes\wp-db.php on line 1454
```

Si ve este error en los archivos debug.log o php_errors.log, significa que la aplicación supera el número de conexiones. Si está hospedando en ClearDB, compruebe el número de conexiones que están disponibles en su [plan de servicio](https://www.cleardb.com/pricing.view).

## <a name="how-do-i-debug-a-nodejs-app-thats-hosted-in-app-service"></a>¿Cómo se puede depurar una aplicación Node.js hospedada en App Service?

1.  Vaya a la **consola de Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net/DebugConsole`).
2.  Vaya a la carpeta de registros de aplicación (D:\home\LogFiles\Application).
3.  En el archivo logging_errors.txt, busque el contenido.

## <a name="how-do-i-install-native-python-modules-in-an-app-service-web-app-or-api-app"></a>¿Cómo se instalan módulos nativos de Python en una aplicación web de App Service o una aplicación de API?

Algunos paquetes podrían no instalarse si se usa pip en Azure. El paquete podría no estar disponible en el índice de paquetes de Python o podría requerirse un compilador (no hay ningún compilador disponible en el equipo que ejecuta la aplicación web en App Service). Para obtener información sobre cómo instalar módulos nativos en aplicaciones web de App Service y aplicaciones de API, vea [Install Python modules in App Service](https://blogs.msdn.microsoft.com/azureossds/2015/06/29/install-native-python-modules-on-azure-web-apps-api-apps/) (Instalar módulos de Python en App Service).

## <a name="how-do-i-deploy-a-django-app-to-app-service-by-using-git-and-the-new-version-of-python"></a>¿Cómo se implementa una aplicación de Django en App Service mediante GIT y la nueva versión de Python?

Para obtener información sobre cómo instalar Django, vea [Deploying a Django app to App Service](https://blogs.msdn.microsoft.com/azureossds/2016/08/25/deploying-django-app-to-azure-app-services-using-git-and-new-version-of-python/) (Implementar una aplicación de Django en App Service).

## <a name="where-are-the-tomcat-log-files-located"></a>¿Dónde se encuentran los archivos de registro de Tomcat?

En Azure Marketplace e implementaciones personalizadas:

* Ubicación de la carpeta: D:\home\site\wwwroot\bin\apache-tomcat-8.0.33\logs
* Archivos de interés:
    * catalina.*aaaa-mm-dd*.log
    * host-manager.*aaaa-mm-dd*.log
    * localhost.*aaaa-mm-dd*.log
    * manager.*aaaa-mm-dd*.log
    * site_access_log.*aaaa-mm-dd*.log


Para implementaciones de **configuración de la aplicación** en el portal:

* Ubicación de la carpeta: D:\home\LogFiles
* Archivos de interés:
    * catalina.*aaaa-mm-dd*.log
    * host-manager.*aaaa-mm-dd*.log
    * localhost.*aaaa-mm-dd*.log
    * manager.*aaaa-mm-dd*.log
    * site_access_log.*aaaa-mm-dd*.log

## <a name="how-do-i-troubleshoot-jdbc-driver-connection-errors"></a>¿Cómo se solucionan los errores de conexión de JDBC Driver?

Es posible que vea el mensaje siguiente en los registros de Tomcat:

```
The web application[ROOT] registered the JDBC driver [com.mysql.jdbc.Driver] but failed to unregister it when the web application was stopped. To prevent a memory leak,the JDBC Driver has been forcibly unregistered
```

Para resolver el error:

1. Quite el archivo sqljdbc*.jar de la carpeta de app/lib.
2. Si está usando el servidor web de Tomcat personalizado o el servidor web de Tomcat de Azure Marketplace, copie este archivo .jar en la carpeta lib de Tomcat.
3. Si va a habilitar Java desde Azure Portal (seleccione **Java 1.8** > **Servidor de Tomcat**), copie el archivo sqljdbc.*jar en la carpeta que sea paralela a la aplicación. Después, agregue la siguiente configuración de Classpath en el archivo web.config:

    ```xml
    <httpPlatform>
    <environmentVariables>
    <environmentVariablename ="JAVA_OPTS" value=" -Djava.net.preferIPv4Stack=true
    -Xms128M -classpath %CLASSPATH%;[Path to the sqljdbc*.jarfile]" />
    </environmentVariables>
    </httpPlatform>
    ```

## <a name="why-do-i-see-errors-when-i-attempt-to-copy-live-log-files"></a>¿Por qué veo errores al intentar copiar archivos de registro activos?

Si intenta copiar archivos de registro activos para una aplicación de Java (por ejemplo, Tomcat), verá este error de FTP:

```
Error transferring file [filename] Copying files from remote side failed.
    
The process cannot access the file because it is being used by another process.
```

El mensaje de error podría variar, en función del cliente FTP.

Todas las aplicaciones de Java tienen este problema de bloqueo. Kudu solo permite descargar este archivo mientras se ejecuta la aplicación.

Si detiene la aplicación, permite el acceso FTP a estos archivos.

Otra solución alternativa consiste en escribir un trabajo web que se ejecute según una programación y que copie estos archivos en un directorio diferente. Para obtener un proyecto de ejemplo, vea el proyecto [CopyLogsJob](https://github.com/kamilsykora/CopyLogsJob).

## <a name="where-do-i-find-the-log-files-for-jetty"></a>¿Dónde se encuentran los archivos de registro de Jetty?

En el caso de Marketplace e implementaciones personalizadas, el archivo de registro está en la carpeta D:\home\site\wwwroot\bin\jetty-distribution-9.1.2.v20140210\logs. Tenga en cuenta que la ubicación de la carpeta depende de la versión de Jetty que use. Por ejemplo, la ruta de acceso que se proporciona aquí es para Jetty 9.1.2. Busque jetty_*AAAA_MM_DD*.stderrout.log.

En el caso de implementaciones de configuración de la aplicación en el portal, el archivo de registro está en D:\home\LogFiles. Busque jetty_*AAAA_MM_DD*.stderrout.log.

## <a name="can-i-send-email-from-my-azure-web-app"></a>¿Puedo enviar correo electrónico desde la aplicación web de Azure?

App Service no tiene una característica integrada de correo electrónico. Para conocer algunas alternativas adecuadas para enviar correo electrónico desde la aplicación, vea este [debate de Stack Overflow](https://stackoverflow.com/questions/17666161/sending-email-from-azure).

## <a name="why-does-my-wordpress-site-redirect-to-another-url"></a>¿Por qué mi sitio de WordPress redirige a otra dirección URL?

Si ha migrado recientemente a Azure, WordPress podría redirigir a la URL de dominio anterior. Esto se debe a una opción de configuración de la base de datos MySQL.

WordPress Buddy+ es una extensión de sitio de Azure que se puede usar para actualizar la dirección URL de redireccionamiento directamente en la base de datos. Para obtener más información sobre el uso de WordPress Buddy+, vea [WordPress tools and MySQL migration with WordPress Buddy+](https://sharepointforum.org/threads/wordpress-tools-and-mysql-migration-with-wordpress-buddy.82929/) (Herramientas de WordPress y migración de MySQL con WordPress Buddy+).

Como alternativa, si prefiere actualizar manualmente la URL de redireccionamiento mediante el uso de consultas SQL o PHPMyAdmin, vea [WordPress: Redirecting to wrong URL](https://blogs.msdn.microsoft.com/azureossds/2016/07/12/wordpress-redirecting-to-wrong-url/) (WordPress: redireccionamiento a una URL incorrecta).

## <a name="how-do-i-change-my-wordpress-sign-in-password"></a>¿Cómo se cambia la contraseña de inicio de sesión de WordPress?

Si ha olvidado su contraseña de inicio de sesión de WordPress, puede usar WordPress Buddy+ para actualizarla. Para restablecer la contraseña, instale la extensión de sitio de Azure WordPress Buddy+ y, después, lleve a cabo los pasos que se describen en [WordPress tools and MySQL migration with WordPress Buddy+](https://sharepointforum.org/threads/wordpress-tools-and-mysql-migration-with-wordpress-buddy.82929/) (Herramientas de WordPress y migración de MySQL con WordPress Buddy+).

## <a name="i-cant-sign-in-to-wordpress-how-do-i-resolve-this"></a>No puedo iniciar sesión en WordPress. ¿Cómo se resuelve este problema?

Si está bloqueado y no puede entrar en WordPress después de instalar recientemente un complemento, dicho complemento podría ser defectuoso. WordPress Buddy+ es una extensión de sitio de Azure que permite deshabilitar complementos en WordPress. Para obtener más información, vea [WordPress tools and MySQL migration with WordPress Buddy+](https://sharepointforum.org/threads/wordpress-tools-and-mysql-migration-with-wordpress-buddy.82929/) (Herramientas de WordPress y migración de MySQL con WordPress Buddy+).

## <a name="how-do-i-migrate-my-wordpress-database"></a>¿Cómo puedo migrar mi base de datos de WordPress?

Tiene varias opciones para migrar la base de datos MySQL conectada a su sitio web de WordPress:

* Desarrolladores: use el [símbolo del sistema o PHPMyAdmin](https://blogs.msdn.microsoft.com/azureossds/2016/03/02/migrating-data-between-mysql-databases-using-kudu-console-azure-app-service/).
* No desarrolladores: Use [WordPress Buddy +](https://sharepointforum.org/threads/wordpress-tools-and-mysql-migration-with-wordpress-buddy.82929/)

## <a name="how-do-i-help-make-wordpress-more-secure"></a>¿Cómo puedo conseguir que WordPress sea más seguro?

Para obtener información sobre los procedimientos de seguridad recomendados para WordPress, vea [Best practices for WordPress security in Azure](https://blogs.msdn.microsoft.com/azureossds/2016/12/26/best-practices-for-wordpress-security-on-azure/) (Procedimientos recomendados para la seguridad de WordPress en Azure).

## <a name="i-am-trying-to-use-phpmyadmin-and-i-see-the-message-access-denied-how-do-i-resolve-this"></a>Estoy intentando usar PHPMyAdmin y aparece el mensaje "Acceso denegado". ¿Cómo se resuelve este problema?

Puede experimentar este problema si la característica de MySQL en aplicación todavía no se está ejecutando en esta instancia de App Service. Para resolver el problema, intente obtener acceso a su sitio web. De este modo se inician los procesos necesarios, incluido el proceso de MySQL en aplicación. Para comprobar que MySQL en la aplicación se está ejecutando, asegúrese de que mysqld.exe aparece en los procesos que se muestran en el Explorador de procesos.

Después de asegurarse de que MySQL en la aplicación se está ejecutando, pruebe a usar PHPMyAdmin.

## <a name="i-get-an-http-403-error-when-i-try-to-import-or-export-my-mysql-in-app-database-by-using-phpmyadmin-how-do-i-resolve-this"></a>Obtengo un error HTTP 403 cuando intento importar o exportar mi base de datos de MySQL en la aplicación mediante el uso de PHPMyadmin. ¿Cómo se resuelve este problema?

Si usa una versión antigua de Chrome, podría tratarse de un problema conocido. Para resolver el problema, actualice a una versión más reciente de Chrome. También puede intentar usar un explorador diferente, como Internet Explorer o Microsoft Edge, en los que el problema no se produce.
