---
title: Solucionar problemas de afinidad de sesión de puerta de enlace de aplicación de Azure
description: En este artículo se proporciona información sobre cómo solucionar problemas de afinidad de sesión en Azure Application Gateway
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: article
ms.date: 02/22/2019
ms.author: absha
ms.openlocfilehash: 07165a497e75934a65719e48a9af7d8d6906ee7b
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65538337"
---
# <a name="troubleshoot-azure-application-gateway-session-affinity-issues"></a>Solucionar problemas de afinidad de sesión de puerta de enlace de aplicación de Azure

Obtenga información sobre cómo diagnosticar y resolver problemas de afinidad de sesión con Azure Application Gateway.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general

La característica de afinidad de sesión basada en cookies es útil cuando se quiere mantener una sesión de usuario en el mismo servidor. Mediante el uso de cookies administradas por la puerta de enlace, Application Gateway puede dirigir el tráfico posterior de una sesión de usuario hasta el mismo servidor para su procesamiento. Esto es importante en aquellos casos en los que se guarda el estado de la sesión de forma local en el servidor para una sesión de usuario.

## <a name="possible-problem-causes"></a>Causas posibles problemas

El problema en el mantenimiento de la afinidad de sesión basada en cookies puede ocurrir debido a los siguientes motivos principales:

- Configuración de "afinidad basada en cookies" no está habilitada
- La aplicación no puede controlar la afinidad basada en cookies
- Aplicación está usando la afinidad basada en cookies, pero las solicitudes de rebote todavía entre los servidores back-end

### <a name="check-whether-the-cookie-based-affinity-setting-is-enabled"></a>Comprobar si está habilitada la configuración de "afinidad basada en cookies"

A veces pueden producirse los problemas de afinidad de sesión cuando se olvida de habilitar la configuración de "Afinidad basada en cookies". Para determinar si se ha habilitado la configuración de "Afinidad basada en cookies" en la ficha Configuración de HTTP en el portal de Azure, siga las instrucciones:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. En el **barra de navegación izquierda** panel, haga clic en **todos los recursos**. Haga clic en el nombre de la puerta de enlace de aplicaciones en la hoja todos los recursos. Si la suscripción que seleccionó ya tiene varios recursos en ella, puede escribir el nombre de la puerta de enlace de aplicaciones en la **filtrar por nombre...** para acceder fácilmente a la puerta de enlace de la aplicación.

3. Seleccione **configuración HTTP** pestaña **configuración**.

   ![solución de problemas-sesión-afinidad-problemas de-1](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-1.png)

4. Haga clic en **appGatewayBackendHttpSettings** en el lado derecho para comprobar si ha seleccionado **habilitado** para afinidad basada en cookies.

   ![solución de problemas-sesión-afinidad-problemas-2](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-2.jpg)



También puede comprobar el valor de la "**CookieBasedAffinity**" se establece en *habilitado*bajo "**backendHttpSettingsCollection**" mediante uno de los métodos siguientes:

- Ejecute [Get AzApplicationGatewayBackendHttpSetting](https://docs.microsoft.com/powershell/module/az.network/get-azapplicationgatewaybackendhttpsetting) en PowerShell
- Examine el archivo JSON con la plantilla de Azure Resource Manager

```
"cookieBasedAffinity": "Enabled", 
```

### <a name="the-application-cannot-handle-cookie-based-affinity"></a>La aplicación no puede controlar la afinidad basada en cookies

#### <a name="cause"></a>Causa

La puerta de enlace de la aplicación solo puede realizar la afinidad basada en sesión mediante el uso de una cookie.

#### <a name="workaround"></a>Solución alternativa

Si la aplicación no puede controlar la afinidad basada en cookies, debe usar un equilibrador de carga interno o externo u otra solución de terceros.

### <a name="application-is-using-cookie-based-affinity-but-requests-still-bouncing-between-back-end-servers"></a>Aplicación está usando la afinidad basada en cookies, pero las solicitudes de rebote todavía entre los servidores back-end

#### <a name="symptom"></a>Síntoma

Ha habilitado la configuración de afinidad basada en cookies, cuando tiene acceso a la puerta de enlace de aplicaciones mediante una dirección URL de nombre corto en Internet Explorer, por ejemplo: [ http://website ](http://website/) , la solicitud aún está rebote entre los servidores back-end.

Para identificar este problema, siga las instrucciones:

1. Realizar un seguimiento del depurador de web en el "cliente" que se conecta a la aplicación detrás de la aplicación de Gateway(We are using Fiddler in this example).
    **Sugerencia** si no sabe cómo usar el Fiddler, active la opción "**desea recopilar el tráfico de red y analícelos con el depurador web**" en la parte inferior.

2. Compruebe y analizar los registros de sesión, para determinar si las cookies proporcionadas por el cliente tienen los detalles de ARRAffinity. Si no encuentra los detalles de ARRAffinity, como "**ARRAffinity =** *ARRAffinityValue*" dentro del conjunto de cookie, que significa que el cliente no está respondiendo con la cookie TRIZ, que se proporciona mediante el Puerta de enlace de aplicaciones.
    Por ejemplo:

    ![solución de problemas-sesión-afinidad-problemas-3](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-3.png)

        ![troubleshoot-session-affinity-issues-4](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-4.png)

La aplicación continúa intentar establecer la cookie en cada solicitud hasta que obtiene respuesta.

#### <a name="cause"></a>Causa

Este problema se produce porque Internet Explorer y otros exploradores no pueden almacenar o usar la cookie con una dirección URL de nombre corto.

#### <a name="resolution"></a>Resolución

Para corregir este problema, debe tener acceso a la puerta de enlace de aplicaciones mediante el uso de un FQDN. Por ejemplo, usar [ http://website.com ](https://website.com/) o [ http://appgw.website.com ](http://appgw.website.com/) .

## <a name="additional-logs-to-troubleshoot"></a>Otros registros para solucionar problemas

Puede recopilar registros adicionales y analizarlos para solucionar problemas de la afinidad de sesión basada en cookies relacionado de problemas

### <a name="analyze-application-gateway-logs"></a>Análisis de registros de Application Gateway

Para recopilar los registros de Application Gateway, siga las instrucciones:

Habilitación del registro mediante Azure Portal

1. En el [portal Azure](https://portal.azure.com/), busque el recurso y, a continuación, haga clic en **registros de diagnóstico**.

   Hay tres registros de auditoría disponibles para Application Gateway: Registro de acceso, el registro de rendimiento, el registro de Firewall

2. Para empezar a recopilar datos, haga clic en **Activar diagnósticos**.

   ![solución de problemas-sesión-afinidad-problemas-5](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-5.png)

3. En la hoja **Configuración de diagnóstico**, se encuentran las opciones de configuración de los registros de diagnóstico. En este ejemplo, se utiliza Log Analytics para almacenar los registros. Haga clic en **configurar** en **Log Analytics** para establecer el área de trabajo. Se pueden utilizar también Events Hubs y la cuenta de almacenamiento para guardar los registros de diagnóstico.

   ![solución de problemas-sesión-afinidad-problemas-6](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-6.png)

4. Confirme la configuración y, a continuación, haga clic en **guardar**.

   ![solución de problemas-sesión-afinidad-problemas-7](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-7.png)

#### <a name="view-and-analyze-the-application-gateway-access-logs"></a>Ver y analizar los registros de acceso de Application Gateway

1. En el portal de Azure en la vista de recursos de puerta de enlace de aplicaciones, seleccione **los registros de diagnóstico** en el **supervisión** sección.

   ![solución de problemas-sesión-afinidad-problemas-8](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-8.png)

2. En el lado derecho, seleccione "**ApplicationGatewayAccessLog**" en la lista desplegable bajo **categorías de registro.**  

   ![solución de problemas-sesión-afinidad-problemas-9](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-9.png)

3. En la lista de registro de acceso de puerta de enlace de aplicaciones, haga clic en el registro que desee analizar y exportar y, a continuación, exporte el archivo JSON.

4. Convertir el archivo JSON que exportó en el paso 3 para el archivo CSV y verlos en Excel, Power BI o cualquier otra herramienta de visualización de datos.

5. Compruebe los siguientes datos:

- **ClientIP**: se trata de la dirección IP del cliente del cliente de conexión.
- **ClientPort** -éste es el puerto de origen del cliente de conexión para la solicitud.
- **RequestQuery** : Esto indica que el servidor de destino que se recibe la solicitud.
- **Server-Routed**: Instancia del grupo de back-end que se recibe la solicitud.
- **X-AzureApplicationGateway-LOG-ID**: identificador de correlación que se ha usado para la solicitud. Se puede utilizar para solucionar problemas de tráfico en los servidores back-end. Por ejemplo: X-AzureApplicationGateway-CACHE-HIT=0&SERVER-ROUTED=10.0.2.4.

  - **SERVER-STATUS**: código de respuesta HTTP que Application Gateway ha recibido del back-end.

  ![solución de problemas-sesión-afinidad-problemas-11](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-11.png)

Si ve dos elementos proceden de la misma IP de cliente y el puerto de cliente y se envían al mismo servidor back-end, que significa que la puerta de enlace de aplicaciones configurado correctamente.

Si ve dos elementos proceden de la misma IP de cliente y el puerto de cliente y se envían a los distintos servidores de back-end, que significa que la solicitud es rebote entre los servidores back-end, seleccionados "**aplicación está usando la afinidad basada en cookies, pero las solicitudes todavía rebote entre los servidores back-end**"en la parte inferior para solucionar el problema.

### <a name="use-web-debugger-to-capture-and-analyze-the-http-or-https-traffics"></a>Usar el depurador web para capturar y analizar el tráfico HTTP o HTTPS

Herramientas de depuración, como Fiddler Web, puede ayudar a depurar aplicaciones web mediante la captura de tráfico de red entre los equipos de Internet y de prueba. Estas herramientas permiten inspeccionar los datos entrantes y salientes según el explorador recibe o envía ellos. Fiddler, en este ejemplo, tiene la opción de reproducción HTTP que puede ayudarle a solucionar problemas del lado cliente con aplicaciones web, especialmente para el tipo de autenticación del problema.

Usar al depurador web de su elección. En este ejemplo usaremos Fiddler para capturar y analizar el tráfico http o https, siga las instrucciones:

1. Descargue la herramienta Fiddler en <https://www.telerik.com/download/fiddler>.

    > [!NOTE]
    > Elija Fiddler4 si el equipo de captura tiene instalado .NET 4. En caso contrario, elija Fiddler2.

2. Haga clic con el botón secundario del mouse en el ejecutable del programa de instalación y ejecutar como administrador para instalar.

            ![troubleshoot-session-affinity-issues-12](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-12.png)

3. Cuando abra Fiddler, debe iniciar automáticamente captura del tráfico (Observe la captura en la esquina inferior izquierda). Presione F12 para iniciar o detener la captura de tráfico.

        ![troubleshoot-session-affinity-issues-13](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-13.png)

4. Probablemente, estará interesado en descifrar tráfico HTTPS, y puede habilitar el descifrado de HTTPS mediante la selección **herramientas** > **opciones de Fiddler**y Active la casilla " **descifrar El tráfico HTTPS**".

        ![troubleshoot-session-affinity-issues-14](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-14.png)

5. Puede quitar las sesiones no relacionadas anteriores antes de reproducir el problema haciendo **X** (icono) > **quitar todo** que sigan la captura de pantalla: 

        ![troubleshoot-session-affinity-issues-15](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-15.png)

6. Una vez que se ha reproducido el problema, guarde el archivo para su revisión seleccionando **archivo** > **guardar** > **todas las sesiones...** . 

        ![troubleshoot-session-affinity-issues-16](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-16.png)

7. Compruebe y analizar los registros de sesiones para determinar cuál es el problema.

    Por ejemplo:

- **Ejemplo A:** Buscar un registro de sesión que la solicitud se envía desde el cliente y lo pasa a la dirección IP pública de la puerta de enlace de aplicaciones, haga clic en este registro para ver los detalles.  En el lado derecho, los datos en el cuadro inferior están lo que la puerta de enlace de la aplicación devuelve al cliente. Seleccione la pestaña "RAW" y determinar si el cliente recibe un "**Set-Cookie: ARRAffinity=** *ARRAffinityValue*." Si no hay ninguna cookie, no se establece la afinidad de sesiones o la puerta de enlace de la aplicación no está aplicando la cookie al cliente.

   > [!NOTE]
   > Este valor ARRAffinity es el identificador de cookie, que establece la puerta de enlace de aplicaciones para que el cliente se envíen a un servidor back-end determinado.

    ![solución de problemas-sesión-afinidad-problemas-17](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-17.png)

- **Ejemplo B:** El siguiente registro de sesiones seguido de uno es el cliente responda a la puerta de enlace de aplicación, que estableció el ARRAAFFINITY anterior. Si coincide con el identificador de cookie ARRAffinity, se debe enviar el paquete en el mismo servidor back-end que se usó anteriormente. Compruebe las siguientes líneas de comunicación de http para ver si está cambiando la cookie de ARRAffinity del cliente.

    ![solución de problemas-sesión-afinidad-problemas-18](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-18.png)

> [!NOTE]
> Para la misma sesión de comunicación, la cookie no se debe para cambiar. Active la casilla superior en el lado derecho, seleccione la pestaña "Cookies" para ver si el cliente es mediante la cookie y enviarlo a la puerta de enlace de la aplicación. De lo contrario, el explorador del cliente no es mantener y mediante la cookie para las conversaciones. En ocasiones, podría quedar el cliente.

 

## <a name="next-steps"></a>Pasos siguientes

Si los pasos anteriores no resuelven el problema, abra una [incidencia de soporte técnico](https://azure.microsoft.com/support/options/).
