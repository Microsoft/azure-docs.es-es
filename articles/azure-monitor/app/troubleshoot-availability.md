---
title: Solución de problemas de las pruebas de disponibilidad de Azure Application Insights
description: Solucione problemas de pruebas web en Azure Application Insights. Obtenga alertas si un sitio web deja de estar disponible o responde con lentitud.
ms.topic: conceptual
author: lgayhardt
ms.author: lagayhar
ms.date: 04/28/2020
ms.reviewer: sdash
ms.openlocfilehash: 0ac8dd189bee1c1d4f5a7a4d0f7de68b085fbc56
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318154"
---
# <a name="troubleshooting"></a>Solución de problemas

En este artículo se proporciona ayuda para solucionar problemas comunes que pueden producirse al usar la supervisión de disponibilidad.

## <a name="ssltls-errors"></a>Errores SSL o TLS

|Síntoma/mensaje de error| Causas posibles|
|--------|------|
|No se pudo crear el canal SSL o TLS seguro  | Versión de SSL. Solo se admiten TLS 1.0, 1.1 y 1.2. **SSLv3 no se admite.**
|Capa de registro de TLSv1.2: Alerta (Nivel: irrecuperable, Descripción: registro MAC incorrecto)| Consulte la conversación sobre StackExchange para [más información](https://security.stackexchange.com/questions/39844/getting-ssl-alert-write-fatal-bad-record-mac-during-openssl-handshake).
|La dirección URL en la que se produce un error se encuentra en una red CDN (Content Delivery Network) | Esto puede deberse a una configuración incorrecta en la red CDN |  

### <a name="possible-workaround"></a>Posible solución alternativa

* Si las direcciones URL que presentan el problema son siempre a recursos dependientes, se recomienda deshabilitar la opción **Analizar solicitudes dependientes** para la prueba web.

## <a name="test-fails-only-from-certain-locations"></a>La prueba muestra un error solo desde determinadas ubicaciones

|Síntoma/mensaje de error| Causas posibles|
|----|---------|
|Se produjo un error en el intento de conexión porque la parte conectada no respondió correctamente después de un período de tiempo  | Un firewall está bloqueando los agentes de prueba en determinadas ubicaciones.|
|    |Se están redirigiendo determinadas direcciones IP a través de equilibradores de carga, administradores de tráfico geográfico o Azure Express Route. 
|    |Si usa Azure ExpressRoute, hay escenarios en los que los paquetes se podrían descartar cuando se produce un [enrutamiento asimétrico](../../expressroute/expressroute-asymmetric-routing.md).|

## <a name="test-failure-with-a-protocol-violation-error"></a>Error de prueba con un error de infracción de protocolo.

|Síntoma/mensaje de error| Causas posibles| Posibles resoluciones |
|----|---------|-----|
|El servidor confirmó una infracción del protocolo. Section=ResponseHeader Detail=CR debe ir seguido de LF. | Esto sucede cuando se detectan encabezados mal formados. En concreto, algunos encabezados podrían no estar usando CRLF para indicar el final de línea, lo que provoca una infracción de la especificación HTTP. Application Insights aplica esta especificación HTTP y devuelve errores para las respuestas que tienen encabezados con un formato incorrecto.| a. Póngase en contacto con el proveedor del host del sitio web o el proveedor de CDN para corregir los servidores con errores. <br> b. En caso de que las solicitudes con error sean recursos (p. ej., archivos de estilo, imágenes o scripts), puede considerar la posibilidad de deshabilitar el análisis de las solicitudes dependientes. Tenga en cuenta que, si lo hace, perderá la capacidad de supervisar la disponibilidad de esos archivos.

> [!NOTE]
> La dirección URL podría no dar error en los exploradores que tienen una validación poco minuciosa de encabezados HTTP. Consulte esta entrada del blog para obtener una explicación detallada del problema: http://mehdi.me/a-tale-of-debugging-the-linkedin-api-net-and-http-protocol-violations/  

## <a name="common-troubleshooting-questions"></a>Preguntas habituales sobre la solución de problemas

### <a name="site-looks-okay-but-i-see-test-failures-why-is-application-insights-alerting-me"></a>Este sitio parece correcto, pero se ven errores de pruebas. ¿Por qué recibo alertas de Application Insights?

   * ¿La prueba tiene habilitada la opción **Analizar solicitudes dependientes**? Esta opción da como resultado una comprobación estricta de los recursos, como las secuencias de comandos y las imágenes, entre otros. Estos tipos de errores pueden no ser visibles en un explorador. Compruebe todas las imágenes, los scripts, las hojas de estilo y cualquier otro archivo cargado que haya cargado la página. Si se produce un error en cualquiera de ellos, se notifica que la prueba ha concluido con errores, incluso si la página HTML principal se carga correctamente. Para reducir la sensibilidad de la prueba para tales errores de recursos, simplemente desactive la opción "Analizar las solicitudes dependientes" de la configuración de pruebas.

   * Para reducir las probabilidades de ruido de señales de red transitorias etc., asegúrese de que se compruebe la configuración "Habilitar reintentos para errores de pruebas". También puede realizar pruebas desde más ubicaciones y administrar el umbral de la regla de alertas en consecuencia para evitar problemas específicos de ubicación que causan las alertas innecesarias.

   * Haga clic en cualquiera de los puntos rojos de la experiencia de disponibilidad o en cualquier error de disponibilidad desde el Explorador de búsqueda para ver los detalles de por qué se notificó el error. El resultado de la prueba, junto con la telemetría del lado servidor correlacionada (si la opción está habilitada) ayudará a comprender el motivo del error de la prueba. Los problemas de conexión o red son las causas más comunes de los problemas transitorios.

   * ¿Se agotó el tiempo de espera de la prueba? Las pruebas se anulan después de 2 minutos. Si la prueba de ping o de varios pasos tarda más de 2 minutos, se notificará como un error. Considere la posibilidad de dividir la prueba en varias partes más pequeñas que puedan completarse en lapsos más cortos.

   * ¿Todas las ubicaciones ha informado de un error, o solo algunas de ellas? Si solo algunas informan de errores, puede ser debido a problemas de red o CDN. De nuevo, hacer clic en los puntos rojos debe ayudar a entender por qué la ubicación notifica errores.

### <a name="i-did-not-get-an-email-when-the-alert-triggered-or-resolved-or-both"></a>No recibí ningún correo electrónico cuando la alerta se desencadenó, se resolvió, o ambos.

Compruebe la configuración de alertas clásicas para confirmar que su correo electrónico aparezca directamente en la lista, o que la lista de distribución en la que se encuentra esté configurada para recibir notificaciones. De ser así, compruebe la configuración de la lista de distribución para confirmar que puede recibir mensajes de correo electrónico externos. Compruebe también si el administrador de correo tiene configurada alguna directiva que pueda estar causando este problema.

### <a name="i-did-not-receive-the-webhook-notification"></a>No recibí la notificación de webhook.

Compruebe que la aplicación que recibe la notificación de webhook esté disponible y procese correctamente las solicitudes de webhook. Para obtener más información, consulte [este artículo](../platform/alerts-log-webhook.md).

### <a name="i-am-getting--403-forbidden-errors-what-does-this-mean"></a>Obtengo errores 403 Prohibido, ¿qué significa esto?

Este error indica que es necesario agregar excepciones de firewall para permitir que los agentes de disponibilidad prueben la dirección URL de destino. Para obtener una lista completa de las direcciones IP del agente admitidas, consulte el [artículo sobre excepciones de IP](./ip-addresses.md#availability-tests).

### <a name="intermittent-test-failure-with-a-protocol-violation-error"></a>Error de prueba intermitente con un error de infracción de protocolo.

El error ("infracción del protocolo... CR debe ir seguido de LF") indica un problema con el servidor (o las dependencias). Aparece cuando se establecen los encabezados con formato incorrecto en la respuesta. Puede deberse a equilibradores de carga o CDN. En concreto, algunos encabezados podrían no estar usando CRLF para indicar el final de línea, lo que provoca una infracción de la especificación del HTTP y, por tanto, no superan la validación en el nivel de WebRequest de .NET. Inspeccione la respuesta para detectar encabezados que podrían estar cometiendo una infracción.

> [!NOTE]
> La dirección URL podría no dar error en los exploradores que tienen una validación poco minuciosa de encabezados HTTP. Consulte esta entrada del blog para obtener una explicación detallada del problema: http://mehdi.me/a-tale-of-debugging-the-linkedin-api-net-and-http-protocol-violations/  

### <a name="i-dont-see-any-related-server-side-telemetry-to-diagnose-test-failures"></a>No veo telemetría de servidor relacionada para diagnosticar errores de pruebas.

Si tiene Application Insights configurado para la aplicación de servidor, esto puede deberse a que se está llevando a cabo un [muestreo](./sampling.md). Seleccione un resultado de disponibilidad diferente.

### <a name="can-i-call-code-from-my-web-test"></a>¿Puedo llamar a código desde mi prueba web?

No. Los pasos de la prueba deben encontrarse en el archivo .webtest. Y no se puede llamar a otras pruebas web ni utilizar bucles. Pero hay varios complementos que pueden resultarle útiles.


### <a name="is-there-a-difference-between-web-tests-and-availability-tests"></a>¿Existe alguna diferencia entre las "pruebas web" y las "pruebas de disponibilidad"?

Los dos términos se pueden usar indistintamente. El término "pruebas de disponibilidad" es más genérico, e incluye las pruebas de ping de dirección URL única y las pruebas web de varios pasos.

### <a name="id-like-to-use-availability-tests-on-our-internal-server-that-runs-behind-a-firewall"></a>Me gustaría usar pruebas de disponibilidad en nuestro servidor interno que se ejecuta detrás de un firewall.

   Hay dos soluciones posibles:

   * Configure el firewall para que permita las solicitudes entrantes de las [direcciones IP de los agentes de prueba web](./ip-addresses.md).
   * Escriba su propio código para comprobar periódicamente el servidor interno. Ejecute el código como un proceso en segundo plano en un servidor de prueba detrás del firewall. El proceso de prueba puede enviar sus resultados a Application Insights mediante la API [TrackAvailability()](/dotnet/api/microsoft.applicationinsights.telemetryclient.trackavailability) en el paquete de SDK principal. Para ello, es necesario que el servidor de prueba tenga acceso saliente al punto de conexión de ingesta de Application Insights, pero plantea un riesgo de seguridad mucho menor que la alternativa de permitir las solicitudes entrantes. Los resultados aparecerán en las hojas de pruebas web de disponibilidad, aunque la experiencia se verá ligeramente simplificada con respecto a lo que está disponible para las pruebas creadas mediante el portal. Las pruebas de disponibilidad personalizadas también aparecerán como resultados de disponibilidad en los análisis, las búsquedas y las métricas.

### <a name="uploading-a-multi-step-web-test-fails"></a>Al cargar una prueba web de varios pasos, se produce un error

Algunas razones por las que puede ocurrir:
   * Hay un límite de tamaño de 300 000.
   * No se admiten los bucles.
   * No se admiten referencias a otras pruebas web.
   * No se admiten orígenes de datos.

### <a name="my-multi-step-test-doesnt-complete"></a>No se completa la prueba de varios pasos

Hay un límite de 100 solicitudes por prueba. Además, la prueba se detiene si se ejecuta durante más de dos minutos.

### <a name="how-can-i-run-a-test-with-client-certificates"></a>¿Cómo se puede ejecutar una prueba con certificados de cliente?

Actualmente no se admite.

## <a name="who-receives-the-classic-alert-notifications"></a>¿Quién recibe las notificaciones de alerta (clásicas)?

Esta sección solo se aplica a las alertas clásicas y le ayudará a optimizar las notificaciones de alerta para asegurarse de que solo reciban las notificaciones los destinatarios que elija. Para más información sobre la diferencia entre [alertas clásicas](../platform/alerts-classic.overview.md) y la nueva experiencia de alertas, consulte el [artículo de información general de alertas](../platform/alerts-overview.md). Para controlar las notificaciones de alertas en la nueva experiencia de alertas, use [grupos de acciones](../platform/action-groups.md).

* Se recomienda el uso de destinatarios específicos para las notificaciones de alerta clásicas.

* Si está habilitada la opción de casilla de verificación **masiva o grupo**, las alertas sobre errores enviadas desde X de Y ubicaciones se envían a los usuarios con roles de administrador o coadministrador.  Básicamente _todos_ los administradores de la _suscripción_ recibirán notificaciones.

* Si está habilitada la opción de casilla **masiva o grupo**, las alertas sobre las métricas de disponibilidad se envían a los usuarios con roles de propietario, colaborador o lector de la suscripción. De hecho, _todos_ los usuarios con acceso a la suscripción del recurso de Application Insights están dentro del ámbito y recibirán notificaciones. 

> [!NOTE]
> Si actualmente usa la opción de casilla de verificación **masiva o grupo** y la deshabilita, no podrá revertir el cambio.

Use la nueva experiencia de alertas o las alertas prácticamente en tiempo real si tiene que enviar notificaciones a los usuarios según sus roles. Con los [grupos de acciones](../platform/action-groups.md), puede configurar el envío de notificaciones por correo electrónico a usuarios con cualquiera de los roles de propietario, colaborador o lector (no combinados conjuntamente como única opción).

## <a name="next-steps"></a>Pasos siguientes

* [Pruebas web de varios pasos](availability-multistep.md)
* [Pruebas de ping de dirección URL](monitor-web-app-availability.md)

