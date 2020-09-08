---
title: Funcionalidad del sistema operativo
description: Obtenga información sobre la funcionalidad del sistema operativo en Azure App Service en Windows. Averigüe qué tipos de acceso de archivo, red y registro obtiene su aplicación.
ms.assetid: 39d5514f-0139-453a-b52e-4a1c06d8d914
ms.topic: article
ms.date: 10/30/2018
ms.custom: seodec18
ms.openlocfilehash: 11798db483f0ba370f73340489c17f38c87ede41
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88080205"
---
# <a name="operating-system-functionality-on-azure-app-service"></a>Funcionalidad del sistema operativo en Azure App Service
En este artículo se describe la funcionalidad del sistema operativo de línea base común que está disponible para todas las aplicaciones Windows que se ejecutan en el [Azure App Service](https://go.microsoft.com/fwlink/?LinkId=529714). Esta funcionalidad incluye archivo, red, acceso de registro, registros de diagnóstico y eventos. 

> [!NOTE] 
> Las [aplicaciones Linux](overview.md#app-service-on-linux) en App Service se ejecutan en sus propios contenedores. No se permite ningún acceso al sistema operativo host, pero sí tiene acceso a la raíz del contenedor. Del mismo modo, para las [aplicaciones que se ejecutan en contenedores de Windows](quickstart-custom-container.md?pivots=container-windows), tiene acceso administrativo al contenedor, pero no al sistema operativo host. 
>

<a id="tiers"></a>

## <a name="app-service-plan-tiers"></a>Niveles de plan de App Service
App Service ejecuta aplicaciones cliente en un entorno de hospedaje multiempresa. Las aplicaciones implementadas en los niveles **Gratis** y **Compartido** se ejecutan en procesos de trabajo en máquinas virtuales compartidas, mientras que las aplicaciones implementadas en los niveles **Estándar** y **Premium** se ejecutan en máquinas virtuales dedicadas específicamente a las aplicaciones asociadas con un único cliente.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

Como App Service permite una experiencia de escalado sin fisuras entre distintos niveles, la configuración de seguridad aplicada a App Service permanece sin cambios. Esto garantiza que las aplicaciones no se comportan de manera diferente de forma repentina, provocando errores de manera inesperada, cuando un plan de App Service cambia entre un nivel y otro.

<a id="developmentframeworks"></a>

## <a name="development-frameworks"></a>Marcos de desarrollo
Los planes de tarifa de App Service controlan la cantidad de recursos de proceso (CPU, almacenamiento en disco, memoria y entrada de red) disponibles para aplicaciones. Sin embargo, la amplitud de la funcionalidad del marco disponible para las aplicaciones sigue siendo la misma independientemente de los planes de escalado.

App Service es compatible con una gran variedad de marcos de desarrollo, incluidos ASP.NET, ASP clásico, node.js, PHP y Python (todos se ejecutan como extensiones en IIS). Para simplificar y normalizar la configuración de seguridad, las aplicaciones de App Service ejecutan normalmente los distintos marcos de desarrollo con su configuración predeterminada. Un enfoque para configurar aplicaciones podría ser personalizar la funcionalidad y el área expuesta de la API para cada marco de desarrollo individual. Por el contrario, App Service adopta un enfoque más genérico mediante la habilitación de una línea base común de la funcionalidad del sistema operativo independientemente de un marco de desarrollo de la aplicación.

Las siguientes secciones resumen los tipos generales de funcionalidad del sistema operativo disponibles para aplicaciones de App Service.

<a id="FileAccess"></a>

## <a name="file-access"></a>Acceso a archivos
Existen varias unidades en App Service, incluidas las unidades locales y las unidades de red.

<a id="LocalDrives"></a>

### <a name="local-drives"></a>Unidades locales
Principalmente, App Service es un servicio que se ejecuta en la parte superior de la infraestructura PaaS (plataforma como servicio) de Azure. Como resultado, las unidades locales "conectadas" a una máquina virtual son los mismos tipos de unidades disponibles para cualquier rol de trabajo que se ejecute en Azure. Esto incluye:

- Una unidad del sistema operativo (la unidad D:\)
- Una unidad de aplicación que contiene archivos cspkg del paquete de Azure que App Service usa de forma exclusiva (y que no son accesibles para los clientes)
- Una unidad de "usuario" (la unidad C:\), cuyo tamaño varía según el tamaño de la máquina virtual. 

Es importante supervisar el uso del disco a medida que la aplicación crece. Si se alcanza la cuota de disco, puede tener efectos negativos para la aplicación. Por ejemplo: 

- Se puede producir un error en la aplicación que indica que no hay suficiente espacio en disco.
- Es posible que vea errores de disco al ir a la consola de Kudu.
- La implementación desde Azure DevOps o Visual Studio puede producir el siguiente error `ERROR_NOT_ENOUGH_DISK_SPACE: Web deployment task failed. (Web Deploy detected insufficient space on disk)`.
- La aplicación puede experimentar un rendimiento lento.

<a id="NetworkDrives"></a>

### <a name="network-drives-unc-shares"></a>Unidades de red (recursos compartidos UNC)
Uno de los aspectos exclusivos de App Service que hace que el mantenimiento y la implementación de aplicaciones sean sencillos es que todo el contenido del usuario se almacena en un conjunto de recursos compartidos UNC. Este modelo se asigna perfectamente al patrón común de almacenamiento de contenido usado en entornos locales de hospedaje web que disponen de varios servidores con equilibrio de carga. 

En App Service existen varios recursos compartidos UNC creados en cada centro de datos. Un porcentaje del contenido de usuario para todos los clientes en cada centro de datos se asigna a cada recurso compartido UNC. Además, todo el contenido del archivo para una suscripción de cliente única se coloca siempre en el mismo recurso compartido UNC. 

Debido al funcionamiento de los servicios de Azure, la máquina virtual específica responsable del hospedaje de un recurso compartido UNC cambiará con el tiempo. Se garantiza que distintas máquinas virtuales realizarán el montaje de los recursos compartidos UNC, ya que estos aumentan y disminuyen durante el curso normal de operaciones de Azure. Por este motivo, las aplicaciones nunca realizan suposiciones de forma rígida sobre que la información de la máquina en una ruta de acceso del UNC permanezca estable con el tiempo. En su lugar, deben usar la ruta de acceso absoluta *faux* adecuada **D:\home\site** que proporciona App Service. Esta ruta de acceso absoluta faux proporciona un método portátil independiente del usuario y la aplicación para hacer referencia a la propia aplicación de alguien. Si usa **D:\home\site**, puede transferir archivos compartidos de una aplicación a otra sin tener que configurar una nueva ruta de acceso absoluta para cada transferencia.

<a id="TypesOfFileAccess"></a>

### <a name="types-of-file-access-granted-to-an-app"></a>Tipos de acceso a archivo concedidos a una aplicación
Cada suscripción de cliente tiene una estructura de directorio reservada en un recurso compartido UNC específico dentro de un centro de datos. Un cliente puede disponer de varias aplicaciones creadas en un centro de datos determinado, por lo que todos los directorios que pertenecen a una única suscripción de cliente se crean en el mismo recurso compartido UNC. El recurso compartido puede incluir directorios como los de contenido, registros de diagnóstico y error, y versiones anteriores de la aplicación creadas por el control de origen. Lo esperado es que los directorios de la aplicación del cliente dispongan de acceso de lectura y escritura en el tiempo de ejecución del código de aplicación de la aplicación.

En las unidades locales conectadas a una máquina virtual que ejecute una aplicación, App Service reserva un fragmento de espacio en la unidad C:\ para el almacenamiento local temporal específico de la aplicación. Aunque la aplicación dispone de un acceso de lectura/escritura completo a su propio almacenamiento local temporal, el almacenamiento no se ha creado realmente para que lo use directamente el código de aplicación. El objetivo real es proporcionar un almacenamiento temporal de archivos para marcos de aplicaciones web e IIS. App Service también limita la cantidad de almacenamiento local temporal disponible para cada aplicación a fin de evitar que las aplicaciones consuman cantidades excesivas de almacenamiento de archivos locales.

Los dos ejemplos de cómo usa App Service el almacenamiento local temporal son el directorio para archivos temporales ASP.NET y el directorio para archivos comprimidos IIS. El sistema de compilación ASP.NET usa el directorio de archivos temporales ASP.NET como ubicación temporal de la memoria caché de compilación. IIS usa el directorio de archivos temporales comprimidos IIS para almacenar los resultados comprimidos de la respuesta. Ambos tipos de uso de archivo (al igual que otros) se reasignan en App Service según el almacenamiento local temporal de la aplicación. Esta reasignación garantiza que la funcionalidad continúa según lo esperado.

Cada aplicación de App Service se ejecuta como una identidad de proceso de trabajo con privilegios reducidos, exclusiva y aleatoria denominada "identidad de grupo de aplicaciones", que se describe a continuación: [https://www.iis.net/learn/manage/configuring-security/application-pool-identities](https://www.iis.net/learn/manage/configuring-security/application-pool-identities). El código de la aplicación usa esta identidad para el acceso básico de solo lectura para la unidad del sistema operativo (la unidad D:\). Esto significa que el código de aplicación puede enumerar estructuras de directorio comunes y leer archivos comunes en la unidad del sistema operativo. Aunque esto puede parecer ser un tipo de nivel amplio de acceso, puede obtenerse acceso a los mismos directorios y archivos cuando realiza el aprovisionamiento de un rol de trabajo en un servicio hospedado de Azure y lee el contenido de la unidad. 

<a name="multipleinstances"></a>

### <a name="file-access-across-multiple-instances"></a>Acceso al archivo en varias instancias
El directorio particular incluye el contenido de una aplicación y el código de una aplicación puede escribir en él. Si una aplicación se ejecuta en varias instancias, el directorio particular se comparte entre todas las instancias de modo que todas ellas ven el mismo directorio. Así, por ejemplo, si una aplicación guarda los archivos cargados en el directorio particular, esos archivos se encuentran inmediatamente disponibles para todas las instancias. 

<a id="NetworkAccess"></a>

## <a name="network-access"></a>Acceso de red
El código de aplicación puede usar protocolos basados en TCP/IPyd UDP para convertir las conexiones de red salientes en extremos accesibles a Internet que expongan servicios externos. Las aplicaciones pueden usar los mismos protocolos para conectarse a los servicios en Azure, por ejemplo, mediante el establecimiento de conexiones HTTPS a SQL Database.

Existe también una funcionalidad limitada para las aplicaciones a la hora de establecer una conexión de bucle invertido local y disponer de una escucha de la aplicación en ese socket de bucle invertido local. Esta característica existe principalmente para habilitar aplicaciones que escuchan en sockets de bucle invertido local como parte de su funcionalidad. Cada aplicación ve una conexión de bucle invertido "privada". La aplicación "A" no puede escuchar en un socket de bucle invertido local establecido por la aplicación "B".

Las canalizaciones con nombre también son compatibles como un mecanismo de comunicación entre procesos (IPC) entre los distintos procesos que ejecutan conjuntamente una aplicación. Por ejemplo, el módulo FastCGI de IIS se basa en conductos con nombre para coordinar los procesos individuales que ejecutan páginas PHP.

<a id="Code"></a>

## <a name="code-execution-processes-and-memory"></a>Memoria, procesos y ejecución de código
Como se indicó anteriormente, las aplicaciones se ejecutan dentro de los procesos de trabajo con privilegios reducidos mediante una identidad aleatoria de grupo de aplicaciones. El código de aplicación dispone de acceso al espacio de memoria asociado al proceso de trabajo, así como a procesos secundarios que pueden dividirse en procesos CGI u otras aplicaciones. Sin embargo, una aplicación no puede obtener acceso a la memoria o los datos de otra aplicación incluso si se encuentra en la misma máquina virtual.

Las aplicaciones pueden ejecutar scripts o páginas escritas con marcos de desarrollo web compatibles. App Service no configura los ajustes del marco web en modos más restringidos. Por ejemplo, las aplicaciones de ASP.NET que se ejecutan en App Service con una confiabilidad "completa" en oposición a un modo de confiabilidad más restringido. Los marcos web, incluidos ASP clásico y ASP.NET, pueden llamar a componentes COM en proceso (pero no componentes COM fuera del proceso) como ADO (objetos de datos ActiveX) que se registran de manera predeterminada en el sistema operativo Windows.

Las aplicaciones pueden dividir y ejecutar código arbitrario. Una aplicación puede realizar acciones como la división de un shell de comandos o ejecutar un script PowerShell. Sin embargo, aunque los procesos y el código arbitrario pueden dividirse desde una aplicación, los programas ejecutables y los scripts siguen restringidos a los privilegios concedidos en el grupo de aplicaciones primario. Por ejemplo, una aplicación puede dividir un ejecutable que realice una llamada HTTP saliente, pero el mismo ejecutable no puede intentar desvincular la dirección IP de una máquina virtual desde la NIC. Se permite la realización de una llamada de red saliente al código de privilegios reducidos, pero el intento de volver a configurar los ajustes de red en una máquina virtual requiere privilegios administrativos.

<a id="Diagnostics"></a>

## <a name="diagnostics-logs-and-events"></a>Eventos y registros de diagnóstico
La información de registro es otro conjunto de datos a los que las aplicaciones intentan obtener acceso. Los tipos de información de registro disponibles para el código que se ejecuta en App Service incluyen la información de registro y diagnóstico que genera una aplicación, que también es de fácil acceso para la aplicación. 

Por ejemplo, los registros HTTP de W3C generados por una aplicación activa se encuentran disponibles en un directorio de registro en la ubicación de recursos compartidos de red para la aplicación o están disponibles en el almacenamiento de blobs si un cliente ha configurado el registro de W3C en el almacenamiento. La última opción permite que se recopile una gran cantidad de registros sin que exista el riesgo de exceder los límites de almacenamiento de archivos asociados a un recurso compartido de red.

De forma similar, la información de diagnóstico en tiempo real de aplicaciones de .NET también puede registrarse mediante la infraestructura de diagnóstico y seguimiento de .NET con opciones para escribir la información de seguimiento en el recurso compartido de red de la aplicación o, de manera alternativa, en una ubicación del almacenamiento de blobs.

Las áreas de seguimiento y registro de diagnóstico que no están disponibles para las aplicaciones son eventos ETW de Windows y registros de eventos comunes de Windows (por ejemplo, registros de eventos del sistema, de la aplicación y de seguridad). Puesto que es posible que la información de seguimiento ETW se visualice en toda la máquina (con ACL correctas), el acceso de lectura y escritura a los eventos ETW se encuentra bloqueado. Los desarrolladores pueden experimentar que las llamadas a la API para leer y escribir eventos ETW y registros de eventos comunes de Windows parezcan funcionar, pero eso se debe a que App Service "emula" las llamadas de manera que parezcan ser correctas. En realidad, el código de la aplicación no dispone de acceso a estos datos de eventos.

<a id="RegistryAccess"></a>

## <a name="registry-access"></a>Acceso al registro
Las aplicaciones disponen de acceso de solo lectura a la mayor parte del registro (no a todo él) de la máquina virtual que están ejecutando. En la práctica, esto significa que las aplicaciones pueden disponer de acceso a las claves de registro que permiten acceso de solo lectura al grupo de usuarios local. Un área de registro no compatible actualmente para el acceso de lectura y escritura es el subárbol HKEY\_CURRENT\_USER.

El acceso de escritura al registro está bloqueado, incluido el acceso a las claves de registro por usuario. Desde una perspectiva de la aplicación, el acceso de escritura al registro no debe basarse nunca en el entorno de Azure, puesto que se pueden migrar aplicaciones entre distintas máquinas virtuales y, de hecho, se hace. El único almacenamiento persistente en el que se puede escribir y que puede depender de la aplicación es la estructura de directorios de contenido por aplicación almacenada en recursos compartidos UNC de App Service. 

## <a name="remote-desktop-access"></a>Acceso de Escritorio remoto

App Service no proporciona acceso mediante Escritorio remoto a las instancias de la máquina virtual.

## <a name="more-information"></a>Más información

[Espacio aislado de Azure App Service](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox): la información más actualizada sobre el entorno de ejecución de App Service. El equipo de desarrollo de App Service mantiene directamente esta página.

