---
title: Sistema operativo y cadencia de aplicación de revisiones en el entorno de ejecución
description: Conozca cómo Azure App Service actualiza el sistema operativo y los entornos de ejecución, qué entornos de ejecución y nivel de aplicación de revisiones tiene la aplicación, y cómo se pueden obtener anuncios de actualización.
ms.topic: article
ms.date: 02/02/2018
ms.custom: seodec18
ms.openlocfilehash: 831ba5f055b70e2f46cb8c6a941c0401df347dd5
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961523"
---
# <a name="os-and-runtime-patching-in-azure-app-service"></a>Aplicación de revisiones al sistema operativo y al entorno de tiempo de ejecución en Azure App Service

En este artículo se explica cómo obtener información de versión determinada sobre el sistema operativo o el software en [App Service](overview.md). 

App Service es una plataforma como servicio, lo que significa que Azure administra en nombre del usuario el sistema operativo y la pila de aplicaciones; el usuario solo se encarga de administrar la aplicación y sus datos. Se puede encontrar más información sobre la administración del sistema operativo y de la pila de aplicaciones en [Azure Virtual Machines](../virtual-machines/index.yml). Teniendo en cuenta esto, no obstante, es importante, como usuario de App Service, que conozca más información, como:

-   ¿Cómo y cuándo se aplican las actualizaciones del sistema operativo?
-   ¿Cómo se aplican las revisiones de App Service frente a vulnerabilidades importantes (por ejemplo, día cero)?
-   ¿Qué versiones del sistema operativo y de tiempo de ejecución ejecutan las aplicaciones?

Por motivos de seguridad, no se publican determinados detalles específicos sobre seguridad. Sin embargo, el artículo pretende mitigar los problemas al maximizar la transparencia del proceso y, además, se aborda cómo estar al tanto de las actualizaciones de tiempo de ejecución o los anuncios sobre seguridad.

## <a name="how-and-when-are-os-updates-applied"></a>¿Cómo y cuándo se aplican las actualizaciones del sistema operativo?

Azure administra la aplicación de revisiones del sistema operativo a dos niveles, es decir, los servidores físicos y las máquinas virtuales invitadas que ejecutan los recursos de App Service. Ambos se actualizan mensualmente, de tal forma que se produce una alineación con el programa [Patch Tuesday](/security-updates/) mensual. Estas actualizaciones se aplican automáticamente, de una forma que garantiza el SLA de Servicios de Azure de alta disponibilidad. 

Para obtener información detallada sobre cómo se aplican las actualizaciones, consulte [Demystifying the magic behind App Service OS updates](https://azure.github.io/AppService/2018/01/18/Demystifying-the-magic-behind-App-Service-OS-updates.html) (Desmitificar la magia que hay detrás de las actualizaciones del sistema operativo de App Service).

## <a name="how-does-azure-deal-with-significant-vulnerabilities"></a>¿Cómo trata Azure las vulnerabilidades importantes?

Si las vulnerabilidades del servidor precisan de una aplicación de revisiones inmediata, como las [vulnerabilidades de día cero](https://wikipedia.org/wiki/Zero-day_(computing)), las actualizaciones de alta prioridad se administran caso por caso.

Manténgase al día con anuncios de seguridad críticos en Azure en el [blog de seguridad de Azure](https://azure.microsoft.com/blog/topics/security/). 

## <a name="when-are-supported-language-runtimes-updated-added-or-deprecated"></a>¿Cuándo se actualizan, agregan o dejan de usar los tiempos de ejecución de los lenguajes admitidos?

Las nuevas versiones estables de los tiempos de ejecución de lenguajes admitidos (principales, secundarios o aplicación de revisiones) se agregan periódicamente a las instancias de App Service. Algunas actualizaciones sobrescriben la instalación existente, mientras que otras se instalan en paralelo a las versiones existentes. Una instalación de sobrescritura significa que la aplicación se ejecuta automáticamente en el tiempo de ejecución actualizado. Una instalación en paralelo significa que debe migrar manualmente la aplicación para beneficiarse de una versión en tiempo de ejecución nueva. Para más información, vea una de las subsecciones.

Las actualizaciones y el desuso de tiempo de ejecución se anuncian aquí:

- https://azure.microsoft.com/updates/?product=app-service 
- https://github.com/Azure/app-service-announcements/issues

> [!NOTE] 
> Esta información se aplica a tiempos de ejecución de lenguaje integrados en una aplicación de App Service. Un tiempo de ejecución personalizado cargado en App Service, por ejemplo, no varía, a menos que lo actualice manualmente.
>
>

### <a name="new-patch-updates"></a>Nuevas actualizaciones de revisiones

Las actualizaciones de .NET, PHP, el SDK de Java o la versión de Tomcat/Jetty se aplican automáticamente al sobrescribir la instalación existente con la versión nueva. Las actualizaciones de revisión de Node.js se instalan en paralelo con las versiones existentes (de forma similar a las versiones principales y secundarias en la siguiente sección). Las nuevas versiones de revisión de Python pueden instalarse manualmente mediante [extensiones de sitio](https://azure.microsoft.com/blog/azure-web-sites-extensions/), en paralelo con las instalaciones de Python integradas.

### <a name="new-major-and-minor-versions"></a>Nuevas versiones principales y secundarias

Cuando se agrega una nueva versión principal o secundaria, se instala en paralelo con las versiones existentes. Puede actualizar manualmente la aplicación a la nueva versión. Si configuró la versión en tiempo de ejecución en un archivo de configuración (como `web.config` y `package.json`), debe actualizarla con el mismo método. Si usó algún ajuste de App Service para configurar la versión en tiempo de ejecución, puede cambiarla en [Azure Portal](https://portal.azure.com) o mediante la ejecución de un comando de la [CLI de Azure](/cli/azure/get-started-with-azure-cli) en [Cloud Shell](../cloud-shell/overview.md), como se muestra en los ejemplos siguientes:

```azurecli-interactive
az webapp config set --net-framework-version v4.7 --resource-group <groupname> --name <appname>
az webapp config set --php-version 7.0 --resource-group <groupname> --name <appname>
az webapp config appsettings set --settings WEBSITE_NODE_DEFAULT_VERSION=8.9.3 --resource-group <groupname> --name <appname>
az webapp config set --python-version 3.4 --resource-group <groupname> --name <appname>
az webapp config set --java-version 1.8 --java-container Tomcat --java-container-version 9.0 --resource-group <groupname> --name <appname>
```

### <a name="deprecated-versions"></a>Versiones en desuso  

Cuando está en desuso una versión anterior, la fecha de eliminación se anuncia para que pueda planear la actualización de la versión en tiempo de ejecución. 

## <a name="how-can-i-query-os-and-runtime-update-status-on-my-instances"></a>¿Cómo puedo consultar el estado de actualización del sistema operativo y en tiempo de ejecución en mis instancias?  

Aunque el acceso a información crítica sobre el sistema operativo está bloqueada (vea [Funcionalidad del sistema operativo en Azure App Service](operating-system-functionality.md)), la [consola de Kudu](https://github.com/projectkudu/kudu/wiki/Kudu-console) le permite consultar las versiones del sistema operativo y en tiempo de ejecución en la instancia de App Service. 

En la tabla siguiente se muestra cómo consultar las versiones de Windows y del tiempo de ejecución de lenguaje que ejecutan las aplicaciones:

| Information | Dónde encontrarla | 
|-|-|
| Versión de Windows | Consulte `https://<appname>.scm.azurewebsites.net/Env.cshtml` (en Información del sistema) |
| Versión de .NET | En `https://<appname>.scm.azurewebsites.net/DebugConsole`, ejecute el siguiente comando en el símbolo del sistema: <br>`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full"` |
| Versión de .NET Core | En `https://<appname>.scm.azurewebsites.net/DebugConsole`, ejecute el siguiente comando en el símbolo del sistema: <br> `dotnet --version` |
| Versión de PHP | En `https://<appname>.scm.azurewebsites.net/DebugConsole`, ejecute el siguiente comando en el símbolo del sistema: <br> `php --version` |
| Versión de Node.js predeterminada | En [Cloud Shell](../cloud-shell/overview.md), ejecute el comando siguiente: <br> `az webapp config appsettings list --resource-group <groupname> --name <appname> --query "[?name=='WEBSITE_NODE_DEFAULT_VERSION']"` |
| Versión de Python | En `https://<appname>.scm.azurewebsites.net/DebugConsole`, ejecute el siguiente comando en el símbolo del sistema: <br> `python --version` |  
| Versión de Java | En `https://<appname>.scm.azurewebsites.net/DebugConsole`, ejecute el siguiente comando en el símbolo del sistema: <br> `java -version` |  

> [!NOTE]  
> El acceso a la ubicación del registro `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages`, donde se almacena la información sobre las [revisiones "KB"](/security-updates/SecurityBulletins/securitybulletins), está bloqueado.
>
>

## <a name="more-resources"></a>Más recursos

[Centro de confianza: seguridad](https://www.microsoft.com/en-us/trustcenter/security)  
[ASP.NET Core de 64 bits en Azure App Service](https://gist.github.com/glennc/e705cd85c9680d6a8f1bdb62099c7ac7)