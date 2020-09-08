---
title: Prácticas recomendadas
description: Obtenga información sobre los procedimientos recomendados y los escenarios comunes de solución de problemas de las aplicaciones que se ejecutan en Azure App Service.
author: dariagrigoriu
ms.assetid: f3359464-fa44-4f4a-9ea6-7821060e8d0d
ms.topic: article
ms.date: 07/01/2016
ms.author: dariac
ms.custom: seodec18
ms.openlocfilehash: 0a25ae41a5f4ed73148f629799ca4865d756a769
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962458"
---
# <a name="best-practices-for-azure-app-service"></a>Procedimientos recomendados para Azure App Service
En este artículo se resumen los procedimientos recomendados para usar el [Azure App Service](https://go.microsoft.com/fwlink/?LinkId=529714). 

## <a name="colocation"></a><a name="colocation"></a>Ubicación compartida
Cuando los recursos de Azure que componen una solución como una aplicación web y una base de datos se encuentran en regiones distintas puede ocurrir lo siguiente:

* Mayor latencia en la comunicación entre recursos
* Cargos monetarios de transferencia de datos entre regiones como se indica en el [página de precios de Azure](https://azure.microsoft.com/pricing/details/data-transfers)

Para los recursos de Azure que componen una solución, como una aplicación web y una cuenta de almacenamiento o base de datos usada para mantener el contenido o los datos, se recomienda que la ubicación compartida se encuentre en la misma región. Al crear recursos, asegúrese de que estén de la misma región de Azure, salvo que haya motivos comerciales o de diseño concretos para que no sea así. Para mover una aplicación de App Service a la región de la base de datos, use la [característica de clonación de App Service](app-service-web-app-cloning.md) que actualmente está disponible para las aplicaciones del plan de App Service Premium.   

## <a name="when-apps-consume-more-memory-than-expected"></a><a name="memoryresources"></a>Cuando las aplicaciones consumen más memoria de lo esperado
Si observa que una aplicación consume más memoria de lo esperado, lo que se indica a través de la supervisión o de recomendaciones de servicio, considere la posibilidad de usar la [característica de recuperación automática de App Service](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites). Una de las opciones de esta característica es que realiza acciones personalizadas en función de un umbral de memoria. Las acciones abarcan todo el espectro, desde las notificaciones por correo electrónico a la investigación a través de volcado de memoria a la mitigación inmediata mediante el reciclado del proceso de trabajo. La recuperación automática se puede configurar tanto a través de web.config como a través de una interfaz de usuario, tal y como se describe en esta entrada del blog [App Service Support Site Extension](https://azure.microsoft.com/blog/additional-updates-to-support-site-extension-for-azure-app-service-web-apps)(Extensión del sitio de soporte del Servicio de aplicaciones).   

## <a name="when-apps-consume-more-cpu-than-expected"></a><a name="CPUresources"></a>Cuando las aplicaciones consumen más CPU de lo esperado
Si observa que una aplicación consume más CPU de lo esperado o experimenta repetidas puntas de actividad de la CPU como indican la supervisión o las recomendaciones de servicio, considere la posibilidad de escalar el plan de App Service vertical u horizontalmente. Si se trata de una aplicación con estado, el escalado vertical es la única opción, mientras que si es sin estado, el escalado horizontal le proporcionará mayor flexibilidad y mayor potencial de escalado. 

Para más información sobre las aplicaciones con y sin estado puede ver este vídeo: [Planning a Scalable End-to-End Multi-Tier Application on Azure App Service](https://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DEV-B414#fbid=?hashlink=fbid) (Planeamiento de una aplicación de varios niveles y de un extremo a otro en Azure App Service). Para más información sobre las opciones de escalado y escalado automático de App Service, consulte el artículo sobre el [escalado de una aplicación web en Azure App Service](manage-scale-up.md).  

## <a name="when-socket-resources-are-exhausted"></a><a name="socketresources"></a>Cuando se agotan los recursos del socket
Una razón habitual para agotar las conexiones TCP salientes es el uso de bibliotecas de cliente que no se han implementado para reutilizar las conexiones TCP o cuando no se usa un protocolo de nivel superior como HTTP de conexión persistente. Revise la documentación de las bibliotecas a las que hacen referencia las aplicaciones del plan de App Service para asegurarse de que se configuran o se tiene acceso a ellas en el código para una reutilización eficiente de las conexiones salientes. Siga también la guía de la documentación de la biblioteca para que la creación y liberación, o la limpieza sean correctas para evitar las conexiones con fugas. Aunque las investigaciones de estas bibliotecas de cliente están en curso, el impacto se pueden mitigar mediante el escalado horizontal a varias instancias.

### <a name="nodejs-and-outgoing-http-requests"></a>Node.js y solicitudes HTTP salientes
Cuando se trabaja con Node.js y muchas solicitudes HTTP salientes, es importante usar HTTP de conexión persistente. Puede usar el paquete [agentkeepalive](https://www.npmjs.com/package/agentkeepalive) `npm` para que le resulte más fácil en su código.

Controle siempre la respuesta `http`, aunque no haga nada en el controlador. Si no controla la respuesta correctamente, la aplicación se acaba bloqueando porque no hay más sockets disponibles.

Por ejemplo, si trabaja con el paquete `http` o `https`:

```javascript
const request = https.request(options, function(response) {
    response.on('data', function() { /* do nothing */ });
});
```

Si ejecuta App Service en Linux en una máquina con varios núcleos, otro procedimiento recomendado es usar PM2 para iniciar varios procesos de Node.js para ejecutar la aplicación. Para hacerlo, especifique un comando de inicio al contenedor.

Por ejemplo, para iniciar cuatro instancias:

```
pm2 start /home/site/wwwroot/app.js --no-daemon -i 4
```

## <a name="when-your-app-backup-starts-failing"></a><a name="appbackup"></a>Cuando la copia de seguridad de la aplicación comienza a fallar
Los dos principales motivos por los que comienza a fallar la copia de seguridad de una aplicación son una configuración de almacenamiento no válida y una configuración de base de datos no válida. Estos errores suelen ocurrir cuando se producen cambios en los recursos de almacenamiento o de base de datos o en el acceso a estos recursos (por ejemplo, la actualización de las credenciales de la base de datos seleccionada en la configuración de copia de seguridad). Las copias de seguridad suelen ejecutarse según una programación y requieren acceso al almacenamiento (para generar los archivos de copia de seguridad) y a las bases de datos (para copiar y leer el contenido que se incluirá en la copia de seguridad). Si no se tiene acceso a cualquiera de estos recursos, se produciría un error de copia de seguridad. 

Cuando aparezcan errores de copia de seguridad, revise los resultados más recientes para saber qué tipo de error se está produciendo. En el caso de errores de acceso de almacenamiento, revise y actualice la configuración de almacenamiento que se usa en la configuración de copia de seguridad. En el caso de errores de acceso de la base de datos, revise y actualice las cadenas de conexiones como parte de la configuración de la aplicación. Después, continúe para actualizar la configuración de copia de seguridad con el fin de incluir correctamente las bases de datos necesarias. Para más información sobre las copias de seguridad de aplicaciones, consulte el artículo sobre cómo [crear la copia de seguridad de una aplicación web en Azure App Service](manage-backup.md).

## <a name="when-new-nodejs-apps-are-deployed-to-azure-app-service"></a><a name="nodejs"></a>Cuando se implementan nuevas aplicaciones de Node.js en Azure App Service
La configuración predeterminada de Azure App Service para las aplicaciones Node.js tiene que como objetivo satisfacer mejor los requisitos de las aplicaciones más comunes. Si considera que la configuración de la aplicación Node.js va a beneficiarse del ajuste personalizado para mejorar el rendimiento o para optimizar el uso de recursos para los recursos de la CPU, memoria o red, consulte los [procedimientos recomendados y la guía para solucionar problemas de las aplicaciones Node en Azure App Service](app-service-web-nodejs-best-practices-and-troubleshoot-guide.md). En este artículo se describen las opciones de IISNode que pueda necesitar configurar para la aplicación de Node.js, así como también se describen los distintos escenarios o problemas que dicha aplicación podría encontrar y se muestra cómo abordarlos.


## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre los procedimientos recomendados, visite [Diagnósticos de App Service](./overview-diagnostics.md) para obtener información sobre los procedimientos recomendados viables específicos para el recurso.

- Vaya a la aplicación web en [Azure Portal](https://portal.azure.com).
- En el panel izquierdo, haga clic en **Diagnosticar y solucionar problemas** para abrir la página Diagnósticos de App Service.
- Elija el icono **Procedimientos recomendados** de la página principal.
- Haga clic en **Best Practices for Availability & Performance** (Prácticas recomendadas de disponibilidad y rendimiento) o en **Best Practices for Optimal Configuration** (Procedimientos recomendados para la configuración óptima) para ver el estado actual de la aplicación en lo que respecta a estas prácticas recomendadas.

También puede usar este vínculo para abrir directamente Diagnósticos de App Service para el recurso: `https://ms.portal.azure.com/?websitesextension_ext=asd.featurePath%3Ddetectors%2FParentAvailabilityAndPerformance#@microsoft.onmicrosoft.com/resource/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/sites/{siteName}/troubleshoot`.