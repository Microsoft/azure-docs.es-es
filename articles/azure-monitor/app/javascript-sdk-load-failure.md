---
title: 'Solución de problemas de errores de carga del SDK para aplicaciones web de JavaScript: Azure Application Insights'
description: Cómo solucionar problemas de errores de carga del SDK para aplicaciones web de JavaScript
ms.topic: conceptual
author: MSNev
ms.author: newylie
ms.date: 06/05/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 83f518326b38e9832c46997a0a2791eb9f35705e
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371641"
---
# <a name="troubleshooting-sdk-load-failure-for-javascript-web-apps"></a>Solución de problemas de errores de carga del SDK para aplicaciones web de JavaScript

<!-- 
Editor Note: This link name above "SDK Load Failure" has a direct references by https://go.microsoft.com/fwlink/?linkid=2128109 which is embedded in the snippet and from the JavaScript Page. If you change this text you MUST also update these references.
-->

El fragmento de código de JavaScript (v3 o posterior) crea y notifica la excepción del error de carga del SDK cuando detecta que el script del SDK no se pudo descargar o inicializar. Simplificando, el cliente (explorador) de los usuarios finales no pudo descargar el SDK de Application Insights o inicializar desde la página de hospedaje identificada y, por tanto, no se informará ningún evento ni datos de telemetría.

![Información general del error del explorador en Azure Portal](./media/javascript-sdk-load-failure/overview.png)

> [!NOTE]
> Esta excepción se admite en todos los principales exploradores que admiten la API fetch() o XMLHttpRequest. Esto excluye IE 8 y versiones anteriores, por lo que no obtendrá este tipo de excepción desde dichos exploradores (a menos que su entorno incluya el polyfill fetch).

![Detalle de la excepción del explorador](./media/javascript-sdk-load-failure/exception.png)

Los detalles de la pila incluyen la información básica con las direcciones URL que usa el usuario final.

| Nombre                      | Descripción                                                                                                  |
|---------------------------|--------------------------------------------------------------------------------------------------------------|
| &lt;CDN&nbsp;Endpoint&gt; | Dirección URL que se utilizó (y produjo un error) para descargar el SDK.                                                      |
| &lt;Help&nbsp;Link&gt;    | Dirección URL que enlaza a la documentación de solución de problemas (esta página).                                              |
| &lt;Host&nbsp;URL&gt;     | Dirección URL completa de la página que utilizaba el usuario final.                                                    |
| &lt;Endpoint&nbsp;URL&gt; | Dirección URL que se utilizó para informar de la excepción. Este valor puede ser útil para identificar si se accedió a la página de hospedaje desde la red pública de Internet o desde una nube privada.

Los motivos más comunes para que se produzca esta excepción son:

- Errores intermitentes de conectividad de red.
- Interrupción en la red CDN de Application Insights.
- No se pudo inicializar el SDK después de cargar el script.
- Se ha bloqueado la red CDN de Application Insights para JavaScript.

Los [errores intermitentes de conectividad de red](#intermittent-network-connectivity-failure) son el motivo más común de esta excepción, especialmente en escenarios móviles en itinerancia en los que los usuarios pierden la conectividad de red de forma intermitente.

En las secciones siguientes se describe cómo solucionar cada posible causa principal de este error.

> [!NOTE]
> Algunos de los pasos de solución de problemas suponen que la aplicación tiene control directo de la etiqueta &lt;script /&gt; del fragmento de código y su configuración, que se devuelven como parte de la página HTML de hospedaje. Si no es así, los pasos identificados no se aplicarán a su escenario.

## <a name="intermittent-network-connectivity-failure"></a>Errores intermitentes de conectividad de red

**Si el usuario experimenta errores intermitentes de conectividad de red, hay menos soluciones posibles y, por lo general, se resolverán por sí mismos transcurrido un breve período de tiempo.** Por ejemplo, si el usuario vuelve a cargar el sitio (actualiza la página), los archivos (con el tiempo) se descargarán y se almacenarán en caché localmente hasta que se publique una versión actualizada.

> [!NOTE]
> Si esta excepción es persistente y le ocurre a muchos de los usuarios (diagnosticado mediante un nivel rápido y sostenido de notificaciones de esta excepción) junto con una reducción en los datos de telemetría habituales del cliente, los problemas intermitentes de conectividad de red _no serán probablemente_ la verdadera causa del problema y debe seguir con el diagnóstico de otros problemas conocidos.

Cuando se da esta situación, es poco probable que el hospedaje del SDK en su propia red CDN reduzca las repeticiones de esta excepción, ya que su propia red CDN se verá afectada por el mismo problema.

Lo mismo se aplica también cuando se usa el SDK mediante la solución de paquetes npm. Sin embargo, desde la perspectiva de los usuarios finales, cuando esto ocurre, es toda la aplicación lo que no se puede cargar o inicializar (en lugar de _simplemente_ el SDK de telemetría, algo que no ven de forma visual), por lo que lo más probable es que actualicen el sitio hasta que se cargue completamente.

También puede intentar usar [paquetes npm](#use-npm-packages-to-embed-the-application-insight-sdk) para insertar el SDK de Application Insights.

Para minimizar los errores intermitentes de conectividad de red, hemos implementado encabezados de control de la memoria caché en todos los archivos de la red CDN, de modo que, una vez que el explorador del usuario final haya descargado la versión actual del SDK, no tendrá que volver a descargarlo y el explorador volverá a usar la copia obtenida anteriormente (consulte [cómo funciona el almacenamiento en caché](../../cdn/cdn-how-caching-works.md)). Si se produce un error en la comprobación del almacenamiento en caché o si se ha publicado una nueva versión, el explorador del usuario final tendrá que descargar la versión actualizada. Por lo tanto, es posible que observe un nivel de _"ruido"_ de fondo en el escenario de comprobación de errores o un pico temporal cuando se publica una nueva versión y está disponible con carácter general (se implementa en la red CDN).
 
## <a name="application-insights-cdn-outage"></a>Interrupción en la red CDN de Application Insights

Puede confirmar si hay una interrupción en la red CDN de Application Insights; para ello, intente acceder al punto de conexión de la red CDN directamente desde el explorador (por ejemplo, https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js) ) desde una ubicación diferente a la de los usuarios finales, probablemente desde su propia máquina de desarrollo (suponiendo que la organización no haya bloqueado este dominio).

Si confirma que hay una interrupción, puede [crear una nueva incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/) o intentar cambiar la dirección URL que se usa para descargar el SDK.

### <a name="change-the-cdn-endpoint"></a>Cambio del punto de conexión de la red CDN
  
Como la aplicación devuelve el fragmento de código y su configuración como parte de cada página generada, puede cambiar la configuración `src` del fragmento de código para que utilice una dirección URL diferente para el SDK. Con este enfoque, puede evitar el problema de la red CDN bloqueada, ya que la nueva dirección URL no debería estar bloqueada.

Puntos de conexión actuales de la red CDN del SDK para JavaScript de Application Insights
- `https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js`
- `https://js.monitor.azure.com/scripts/b/ai.2.min.js`

> [!NOTE]
> El punto de conexión `https://js.monitor.azure.com/` es un alias que nos permite cambiar entre proveedores de la red CDN en unos 5 minutos, sin necesidad de que el usuario cambie la configuración. Esto nos permite corregir los problemas detectados relacionados con la red CDN más rápidamente si un proveedor de CDN tiene problemas regionales o globales sin necesidad de que todos los usuarios ajusten su configuración.

## <a name="sdk-failed-to-initialize-after-loading-the-script"></a>No se pudo inicializar el SDK después de cargar el script

Cuando no se puede inicializar el SDK, el elemento &lt;script /&gt; se descargó correctamente desde la red CDN, pero se produce un error durante la inicialización. Esto se puede deber a que faltan dependencias o que no son válidas o bien a algún tipo de excepción de JavaScript.

Lo primero que hay que comprobar es si el SDK se descargó correctamente. Si el script no se descargó, este escenario __no__ es la causa de la excepción del error de carga del SDK.

Comprobación rápida: con un explorador que admita herramientas de desarrollo (F12), valide en la pestaña Red que el script definido en la configuración ```src``` del fragmento de código se descargó con un código de respuesta 200 (correcto) o 304 (no modificado). También puede usar una herramienta como Fiddler para revisar el tráfico de red.

En las secciones siguientes se incluyen diferentes opciones de informes y se recomienda crear una incidencia de soporte técnico o generar un problema en GitHub.

 Reglas básicas para las notificaciones:

- Si el problema solo afecta a un número pequeño de usuarios y a un determinado subconjunto de versiones del explorador (consulte los detalles de la excepción que se ha comunicado), es probable que se trate de un problema con el entorno o el usuario final, lo que probablemente requerirá que la aplicación proporcione implementaciones de `polyfill` adicionales. En estos casos, [notifique un problema en GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues).
- Si este problema afecta a toda la aplicación y todos los usuarios se ven afectados, es probable que se trate de un problema relacionado con la versión y, por lo tanto, debe [crear una nueva incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/).

### <a name="javascript-exceptions"></a>Excepciones de JavaScript

En primer lugar, compruebe si hay excepciones de JavaScript. Cargue la página con un explorador que admita las herramientas de desarrollo (F12) y revise si se produjeron excepciones.

Si se notifican excepciones en el script del SDK (por ejemplo, ai.2.min.js), esto puede indicar que la configuración que se ha pasado al SDK contiene una configuración inesperada o que falta o que se ha implementado una versión defectuosa en la red CDN.

Para comprobar si hay una configuración defectuosa, cambie la configuración que se pasa al fragmento de código (si no lo ha hecho ya), de modo que solo incluya la clave de instrumentación como un valor de cadena.

> src: "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js ",<br />
> cfg:{<br />
> instrumentationKey: "CLAVE_DE_INSTRUMENTACIÓN"<br />
> }});<br />

Si al usar esta configuración mínima todavía aparece una excepción de JavaScript en el script del SDK, [cree una nueva incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/), ya que esto requerirá que se revierta la compilación defectuosa, ya que probablemente sea un problema con una versión implementada recientemente.

Si la excepción desaparece, es probable que el problema se deba a un error de coincidencia de tipos o a un valor inesperado. Empiece a agregar de nuevo las opciones de configuración de una en una y pruebe hasta que se produzca la excepción. A continuación, consulte la documentación del elemento que causa el problema. Si la documentación no es clara o necesita ayuda, [notifique un problema en GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues).

Si la configuración se ha implementado previamente y funcionaba, pero ha comenzado a notificar esta excepción, puede tratarse de un problema con una versión recién implementada. Compruebe si solo afecta a un pequeño conjunto de usuarios o exploradores y [notifique un problema en GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues) o [cree una nueva incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/).

### <a name="enable-console-debugging"></a>Habilitación de la depuración en la consola

Si no se produce ninguna excepción, el paso siguiente consiste en habilitar la depuración en la consola; para ello, agregue la opción `loggingLevelConsole` a la configuración y se enviarán todos los errores y advertencias de inicialización a la consola de los exploradores (disponible normalmente mediante las herramientas de desarrollo (F12)). Los errores detectados deberían ser autodescriptivos y puede [notificar un problema en GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues) si necesita ayuda adicional.

> cfg:{<br />
> instrumentationKey: "CLAVE_DE_INSTRUMENTACIÓN",<br />
> loggingLevelConsole: 2<br />
> }});<br />

> [!NOTE]
> Durante la inicialización, el SDK realiza algunas comprobaciones básicas de las principales dependencias conocidas. Si el runtime actual no las proporciona, informará de los errores como mensajes de advertencia en la consola, pero solo si el valor de `loggingLevelConsole` es mayor que cero.

Si todavía no se puede inicializar, intente habilitar la opción de configuración ```enableDebug```. Esto hará que se generen todos los errores internos como una excepción (lo que hará que se pierdan los datos de telemetría). Dado que se trata de una configuración solo del desarrollador, es probable que se produzca ruido con las excepciones que se inician como parte de algunas comprobaciones internas, por lo que deberá revisar cada excepción para determinar qué problema está causando el error en el SDK. Use la versión no reducida del script (observe que la extensión del fragmento siguiente es ".js" y no ".min.js"); de lo contrario, las excepciones no serán legibles.

> [!WARNING]
> Se trata de una configuración solo para desarrolladores y nunca se debe habilitar en un entorno de producción completo, ya que se perderán los datos de telemetría.

> src: "https://az416426.vo.msecnd.net/scripts/b/ai.2.js ",<br />
> cfg:{<br />
> instrumentationKey: "CLAVE_DE_INSTRUMENTACIÓN",<br />
> enableDebug: true<br />
> }});<br />

Si esto tampoco proporciona información, debe [notificar un problema en GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues) con los detalles y un sitio de ejemplo, si dispone de uno. Incluya la versión del explorador, el sistema operativo y los detalles del marco de JS para ayudar a identificar el problema.

## <a name="the-application-insights-javascript-cdn-has-been-blocked"></a>Se ha bloqueado la red CDN de Application Insights para JavaScript

Es posible que se bloquee la red CDN cuando se ha notificado o identificado un punto de conexión de la red CDN del SDK de Application Insights para JavaScript como no seguro. Cuando esto sucede, el punto de conexión se incluye en una lista pública de bloqueos y los consumidores de estas listas comenzarán a bloquear todo el acceso.

Para resolverlo, es necesario que el propietario del punto de conexión de la red CDN colabore con la entidad de la lista de bloqueos que marcó el punto de conexión como no seguro para que este se elimine de la lista correspondiente.

Compruebe si el punto de conexión de la red CDN se ha identificado como no seguro.
- [Informe de transparencia de Google](https://transparencyreport.google.com/safe-browsing/search)
- [VirusTotal](https://www.virustotal.com/gui/home/url)
- [Sucuri SiteCheck](https://sitecheck.sucuri.net/)

En función de la frecuencia con la que la aplicación, el firewall o el entorno actualicen sus copias locales de estas listas, puede tardar una cantidad considerable de tiempo y requerir la intervención manual de los usuarios finales o los departamentos de TI corporativos para forzar una actualización o permitir explícitamente que los puntos de conexión de la red CDN resuelvan el problema.

Si el punto de conexión de la red CDN se identifica como no seguro, [cree una incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/) para asegurarse de que el problema se resuelve lo antes posible.

Para evitar *potencialmente* este problema con más rapidez, puede [cambiar el punto de conexión de la red CDN del SDK](#change-the-cdn-endpoint).

### <a name="application-insights-javascript-cdn-is-blocked-by-end-user---blocked-by-browser-installed-blocker-personal-firewall"></a>La red CDN de Application Insights para JavaScript está bloqueada (por el usuario final: bloqueado por el explorador, por un bloqueador instalado o por el firewall personal)

Compruebe si los usuarios finales han:

- Instalado un complemento del explorador (normalmente algún tipo de bloqueador de anuncios, malware o elementos emergentes).
- Bloqueado (o no permitido) los puntos de conexión de la red CDN de Application Insights en el explorador o servidor proxy.
- Configurado una regla de firewall que provoca el bloqueo del dominio de la red CDN del SDK (o que no se resuelva entrada de DNS).

Si han configurado alguna de estas opciones, tendrá que colaborar con ellos (o proporcionar documentación) para permitir el acceso a los puntos de conexión de la red CDN.

Es posible que el complemento que han instalado utilice listas públicas de bloqueos. Si no es así, lo más probable es que exista alguna otra solución configurada manualmente o que se usen listas de bloqueo de dominio privado.

#### <a name="add-exceptions-for-cdn-endpoints"></a>Adición de excepciones para los puntos de conexión de la red CDN

Colabore con los usuarios finales o proporcione documentación que les informe de que deben permitir que los scripts de los puntos de conexión de la red CDN de Application Insights se descarguen mediante su inclusión en el complemento del explorador o en la lista de excepciones de las reglas de firewall (esto variará en función del entorno del usuario final).

Este es un ejemplo de cómo configurar [Chrome para permitir o bloquear el acceso a sitios web](https://support.google.com/chrome/a/answer/7532419?hl=en).

### <a name="application-insights-cdn-is-blocked-by-corporate-firewall"></a>La red CDN de Application Insights está bloqueada (por el firewall corporativo)

Si los usuarios finales se encuentran en una red corporativa, es más probable que se trate de su solución de firewall y que su departamento de TI haya implementado algún tipo de sistema de filtrado de Internet. En este caso, tendrá que colaborar con ellos para permitir las reglas necesarias para los usuarios finales.

#### <a name="add-exceptions-for-cdn-endpoints-for-corporations"></a>Adición de excepciones para los puntos de conexión de la red CDN para entidades corporativas

  Es similar a la [adición de excepciones para los usuarios finales](#add-exceptions-for-cdn-endpoints), pero debe colaborar con el departamento de TI de la empresa para que configuren los puntos de conexión de la red CDN de Application Insights desde los que se va a descargar mediante la inclusión (o eliminación) de los mismos en todos los servicios de lista de bloqueos o de lista de permitidos del dominio.

  > [!WARNING]
  > Si el usuario corporativo usa una [nube privada](https://azure.microsoft.com/overview/what-is-a-private-cloud/) y no puede habilitar ninguna forma de excepción que proporcione a sus usuarios internos acceso público a los puntos de conexión de la red CDN, tendrá que usar los [paquetes npm de Application Insights](https://www.npmjs.com/package/applicationinsights) u hospedar el SDK de Application Insights en su propia red CDN.  

### <a name="additional-troubleshooting-for-blocked-cdn"></a>Solución de problemas adicional para casos de red CDN bloqueada

> [!NOTE]
> Si los usuarios usan una [nube privada](https://azure.microsoft.com/overview/what-is-a-private-cloud/) y no tienen acceso a la red pública de Internet, deberá hospedar el SDK en su propia red CDN o usar paquetes NPM.

#### <a name="host-the-sdk-on-your-own-cdn"></a>Hospedaje del SDK en su propia red CDN

 En lugar de que los usuarios finales descarguen el SDK de Application Insights desde la red CDN pública, puede hospedar el SDK de Application Insights en su propio punto de conexión de la red CDN. Se recomienda que use una versión específica (ai.2.#.#.min.js) para que sea más fácil identificar la versión que se usa. También puede actualizarlo de forma periódica a la versión actual (ai.2.min.js) para que pueda aprovechar las correcciones de errores y las nuevas características que estén disponibles.

#### <a name="use-npm-packages-to-embed-the-application-insight-sdk"></a>Uso de paquetes npm para insertar el SDK de Application Insights

En lugar de usar el fragmento de código y los puntos de conexión de la red CDN pública, podría usar [paquetes npm](https://www.npmjs.com/package/applicationinsights) para incluir el SDK como parte de sus propios archivos JavaScript. El SDK se convertirá en otro paquete dentro de sus propios scripts.

> [!NOTE]
> Al usar los paquetes NPM, se recomienda que también use algún tipo de [agrupador de JavaScript](https://www.bing.com/search?q=javascript+bundler) para ayudar con la división y la minificación del código.

Al igual que con el fragmento de código, también es posible que sus propios scripts (con o sin usar los paquetes NPM del SDK) se vean afectados por los mismos problemas de bloqueo que se enumeran aquí, por lo que, en función de la aplicación, los usuarios y el marco de trabajo, puede que desee considerar la posibilidad de implementar algo similar a la lógica del fragmento de código para detectar estos problemas.


## <a name="next-steps"></a>Pasos siguientes 
- [Obtención de ayuda adicional mediante la notificación de un problema en GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues)
- [Supervisar el uso de las páginas web](javascript.md)
