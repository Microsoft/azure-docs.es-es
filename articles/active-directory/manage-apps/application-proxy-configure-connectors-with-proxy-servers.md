---
title: Trabajo con servidores proxy locales existentes y Azure AD | Microsoft Docs
description: Se explica cómo trabajar con servidores proxy locales existentes.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 04/07/2020
ms.author: kenwith
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: d177dce250d65b4f9d825c9d70916f70c4076d4b
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88077516"
---
# <a name="work-with-existing-on-premises-proxy-servers"></a>Trabajo con servidores proxy locales existentes

En este artículo se explica cómo configurar conectores del proxy de aplicación de Azure Active Directory (Azure AD) para que funcionen con servidores proxy de salida. Está destinado a clientes con entornos de red que tienen servidores proxy en la actualidad.

Empezaremos examinando estos escenarios de implementación principales:

* Configuración de conectores para omitir los proxy de salida locales.
* Configuración de conectores para usar un proxy de salida para obtener acceso al Proxy de aplicación de Azure AD.
* Configuración mediante un proxy entre el conector y la aplicación de back-end.

Para más información sobre cómo funcionan los conectores, consulte [Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md).

## <a name="bypass-outbound-proxies"></a>Omisión de servidores proxy de salida

Los conectores tienen componentes del sistema operativo subyacentes que realizan solicitudes salientes. Estos componentes intentan automáticamente encontrar un servidor proxy en la red mediante la detección automática de proxy web (WPAD).

Los componentes del sistema operativo intentan localizar un servidor proxy llevando a cabo una búsqueda de DNS para wpad.domainsuffix. Si la búsqueda se resuelve en DNS, se realiza una solicitud HTTP para la dirección IP para wpad.dat. Esta solicitud se convierte en el script de configuración de proxy en su entorno. El conector lo usará para seleccionar un servidor proxy de salida. Sin embargo, es posible que el tráfico del conector no pase aún porque se requiera una configuración adicional en el servidor proxy.

Puede configurar el conector para omitir el proxy local a fin de garantizar que utilice conectividad directa a los servicios de Azure. Se recomienda este método, siempre que lo permita la directiva de red, ya que significa que tendrá una configuración menos que mantener.

Para deshabilitar el uso del proxy de salida para el conector, edite el archivo C:\Archivos de programa\Microsoft AAD App Proxy Connector\ApplicationProxyConnectorService.exe.config y agregue la sección *system.net* que se muestra en este ejemplo de código:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.net>
    <defaultProxy enabled="false"></defaultProxy>
  </system.net>
  <runtime>
    <gcServer enabled="true"/>
  </runtime>
  <appSettings>
    <add key="TraceFilename" value="AadAppProxyConnector.log" />
  </appSettings>
</configuration>
```

Para asegurarse de que el servicio de actualización de los conectores también omite el proxy, realice un cambio similar en el archivo ApplicationProxyConnectorUpdaterService.exe.config. El archivo se encuentra en C:\Archivos de programa\Microsoft AAD App Proxy Connector Updater.

Asegúrese de realizar copias de los archivos originales por si fuera necesario revertir a los archivos .config predeterminados.

## <a name="use-the-outbound-proxy-server"></a>Uso del servidor proxy saliente

Algunos entornos requieren que todo el tráfico saliente atraviese sin excepciones un proxy de salida. Por lo tanto, no se contempla la opción de omitir el proxy.

Puede configurar el tráfico del conector de manera que vaya a través del proxy de salida, tal y como se muestra en el diagrama siguiente:

 ![Configuración del tráfico del conector para pasar a través de un proxy de salida para acceder al Proxy de aplicación de Azure AD](./media/application-proxy-configure-connectors-with-proxy-servers/configure-proxy-settings.png)

Como solo se tiene tráfico saliente, no es necesario configurar el acceso entrante a través de los firewalls.

> [!NOTE]
> El proxy de aplicación no admite la autenticación en otros proxies. Las cuentas de servicio de red del conector/actualizador deben ser capaces de conectarse con el proxy sin que se tengan que enfrentar a una autenticación.

### <a name="step-1-configure-the-connector-and-related-services-to-go-through-the-outbound-proxy"></a>Paso 1: Configurar el conector y los servicios relacionados para que vayan a través del proxy de salida

Si se ha habilitado WPAD en el entorno y se ha configurado correctamente, el conector detecta automáticamente el servidor del proxy de salida y trata de usarlo. Sin embargo, puede configurar explícitamente el conector para que vaya a través de un proxy de salida.

Para ello, edite el archivo C:\Archivos de programa\Microsoft AAD App Proxy Connector\ApplicationProxyConnectorService.exe.config y agregue la sección *system.net* que se muestra en este ejemplo de código. Cambie *proxyserver:8080* para que refleje el nombre o la dirección IP del servidor proxy local y el puerto en el que está escuchando. El valor debe tener el prefijo http://, incluso si usa una dirección IP.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.net>  
    <defaultProxy>   
      <proxy proxyaddress="http://proxyserver:8080" bypassonlocal="True" usesystemdefault="True"/>   
    </defaultProxy>  
  </system.net>
  <runtime>
    <gcServer enabled="true"/>
  </runtime>
  <appSettings>
    <add key="TraceFilename" value="AadAppProxyConnector.log" />
  </appSettings>
</configuration>
```

A continuación, configure el servicio de actualización de los conectores para que use el proxy; para ello, haga un cambio similar en el archivo C:\Archivos de programa\Microsoft AAD App Proxy Connector Updater\ApplicationProxyConnectorUpdaterService.exe.config.

### <a name="step-2-configure-the-proxy-to-allow-traffic-from-the-connector-and-related-services-to-flow-through"></a>Paso 2: Configurar el proxy para permitir que pase el tráfico desde el conector y los servicios relacionados

Hay cuatro aspectos que se deben tener en cuenta en el servidor proxy saliente:

* Las reglas de salida del proxy
* La autenticación del proxy
* Los puertos del proxy
* La inspección de TLS

#### <a name="proxy-outbound-rules"></a>Las reglas de salida del proxy

Permita el acceso a las siguientes direcciones URL:

| URL | Cómo se usa |
| --- | --- |
| \*.msappproxy.net<br>\*.servicebus.windows.net | Comunicación entre el conector y el servicio en la nube del proxy de aplicación |
| mscrl.microsoft.com:80<br>crl.microsoft.com:80<br>ocsp.msocsp.com:80<br>www.microsoft.com:80 | El conector utiliza estas direcciones URL para comprobar los certificados. |
| login.windows.net<br>secure.aadcdn.microsoftonline-p.com<br>*.microsoftonline.com<br>* .microsoftonline-p.com<br>*.msauth.net<br>* .msauthimages.net<br>*.msecnd.net<br>* .msftauth.net<br>*.msftauthimages.net<br>* .phonefactor.net<br>enterpriseregistration.windows.net<br>management.azure.com<br>policykeyservice.dc.ad.msft.net<br>ctldl.windowsupdate.com:80 | El conector utiliza estas direcciones URL durante el proceso de registro. |

Si el firewall o proxy le permite configurar listas de DNS permitidos, puede permitir conexiones a \*.msappproxy.net y \*.servicebus.windows.net. Si no, deberá permitir el acceso a los [intervalos IP del centro de datos de Azure](https://www.microsoft.com/download/details.aspx?id=41653). Los intervalos IP se actualizan cada semana.

Si no puede permitir la conectividad mediante el FQDN y debe especificar intervalos de direcciones IP en su lugar, use estas opciones:

* Permitir el acceso de salida del conector a todos los destinos.
* Permitir el acceso de salida del conector a todos los [rangos de direcciones IP del centro de datos de Azure](https://www.microsoft.com//download/details.aspx?id=41653). El desafío de usar la lista de intervalos IP del centro de datos de Azure es que se actualiza semanalmente. Es necesario colocar un proceso para garantizar que las reglas de acceso se actualizan en consecuencia. Si se utiliza solo un subconjunto de las direcciones IP, se puede producir una interrupción de la configuración.

#### <a name="proxy-authentication"></a>La autenticación del proxy

Actualmente no se admite la autenticación del proxy. Nuestra recomendación actual es permitir el acceso anónimo de conector a los destinos de Internet.

#### <a name="proxy-ports"></a>Los puertos del proxy

El conector realiza las conexiones salientes basadas en TLS con el método CONNECT. Básicamente, este método configura un túnel a través del proxy de salida. Configure el servidor proxy para permitir la tunelización a los puertos no estándar 443 y 80.

> [!NOTE]
> Service Bus, si se ejecuta a través de HTTPS, usa el puerto 443. Sin embargo, de forma predeterminada, Service Bus intenta crear conexiones TCP directas y recurrir a HTTPS solo si se produce un error en la conectividad directa.

#### <a name="tls-inspection"></a>La inspección de TLS

No utilice la inspección de TLS para el tráfico del conector, ya que le causa problemas. El conector usa un certificado para autenticarse en el servicio de proxy de aplicación y ese certificado se puede perder durante la inspección de TLS.

## <a name="configure-using-a-proxy-between-the-connector-and-backend-application"></a>Configuración mediante un proxy entre el conector y la aplicación de back-end
El uso de un proxy de reenvío para la comunicación hacia la aplicación de back-end puede ser un requisito especial en algunos entornos.
Para habilitarlo, siga estos pasos:

### <a name="step-1-add-the-required-registry-value-to-the-server"></a>Paso 1: Agregar el valor del Registro requerido al servidor
1. Para habilitar el uso del proxy predeterminado, agregue el valor del Registro (DWORD) siguiente `UseDefaultProxyForBackendRequests = 1` a la clave del Registro de configuración del conector que se encuentra en "HKEY_LOCAL_MACHINE\Software\Microsoft\Microsoft AAD App Proxy Connector".

### <a name="step-2-configure-the-proxy-server-manually-using-netsh-command"></a>Paso 2: Configurar el servidor proxy manualmente con el comando netsh
1.  Habilite la directiva de grupo Configuración de proxy por equipo. Se encuentra en: Configuración del equipo\Directivas\Plantillas administrativas\Componentes de Windows\Internet Explorer. Se debe establecer esto en lugar de tener esta directiva establecida en por usuario.
2.  Ejecute `gpupdate /force` en el servidor o reinicie el servidor para asegurarse de que usa la configuración de directiva de grupo actualizada.
3.  Inicie un símbolo del sistema con privilegios elevados con derechos de administrador y especifique `control inetcpl.cpl`.
4.  Configure los valores de proxy necesarios. 

Esta configuración hace que el conector use el mismo proxy de reenvío para la comunicación a Azure y a la aplicación de back-end. Si el conector para la comunicación de Azure no requiere ningún proxy de reenvío ni un proxy de reenvío distinto, puede configurarlo mediante la modificación del archivo ApplicationProxyConnectorService.exe.config, como se describe en las secciones Omisión de servidores proxy de salida o Uso del servidor proxy saliente.

> [!NOTE]
> Hay varias maneras de configurar el proxy de Internet en el sistema operativo. La configuración de proxy establecida a través de NETSH WINHTTP (ejecute `NETSH WINHTTP SHOW PROXY` para comprobarlo) invalida la configuración del proxy que estableció en el paso 2. 

El servicio de actualización del conector usará también el proxy del equipo. Este comportamiento se puede cambiar si modifica el archivo ApplicationProxyConnectorUpdaterService.exe.config.

## <a name="troubleshoot-connector-proxy-problems-and-service-connectivity-issues"></a>Solución de problemas del proxy del conector y de conectividad del servicio

Ahora debería ver todo el tráfico pasando a través del proxy. Si tiene problemas, la siguiente información debería ayudar.

La mejor manera de identificar y solucionar problemas de conectividad del conector es realizar una captura de red cuando se inicia el servicio de conector. Estas son algunas sugerencias rápidas sobre cómo capturar y filtrar seguimientos de red.

Puede usar la herramienta de supervisión que prefiera. En este artículo, hemos usado Analizador de mensajes de Microsoft.

Los ejemplos siguientes son específicos del Analizador de mensajes pero los principios se pueden aplicar a cualquier herramienta de análisis.

### <a name="take-a-capture-of-connector-traffic"></a>Toma de una captura del tráfico de conector

Para solucionar el problema inicial, realice los pasos siguientes:

1. Desde services.msc, detenga el servicio de conector del Proxy de aplicación de Azure AD.

   ![Servicio de conector del Proxy de aplicación de Azure AD en services.msc](./media/application-proxy-configure-connectors-with-proxy-servers/services-local.png)

1. Ejecute el Analizador de mensajes como administrador.
1. Seleccione **Start local trace** (Iniciar seguimiento local).
1. Inicie el servicio de conector del Proxy de aplicación de Azure AD.
1. Detenga la captura de red.

   ![Captura de pantalla que muestra el botón Detener captura de red](./media/application-proxy-configure-connectors-with-proxy-servers/stop-trace.png)

### <a name="check-if-the-connector-traffic-bypasses-outbound-proxies"></a>Comprobación de si el tráfico de conector omite proxies de salida

Si ha configurado el conector del proxy de aplicación para que omita los servidores proxy y se conecte directamente con el servicio del proxy de aplicación, busque en la captura de red para ver si hay intentos de conexión TCP con errores.

Use el filtro del Analizador de mensajes para identificar estos intentos. Escriba `property.TCPSynRetransmit` en el cuadro de filtro y seleccione **Apply** (Aplicar).

Un paquete SYN es el primer paquete enviado para establecer una conexión TCP. Si este paquete no devuelve ninguna respuesta, se intenta de nuevo el envío de SYN. Puede utilizar el filtro anterior para ver las solicitudes de SYN retransmitidas. A continuación, puede comprobar si estas solicitudes SYN corresponden a cualquier tráfico relacionado con el conector.

Si el conector debería estar realizando conexiones directas a los servicios de Azure, las respuestas de SynRetransmit en el puerto 443 son un claro indicio de que tiene un problema de red o firewall.

### <a name="check-if-the-connector-traffic-uses-outbound-proxies"></a>Comprobación de si el tráfico de conector utiliza proxies de salida

Si ha configurado el tráfico de conector del proxy de aplicación para pasar por los servidores proxy, busque en el proxy si hay conexiones https con errores.

Si desea filtrar la captura de red para ver estos intentos de conexión, escriba `(https.Request or https.Response) and tcp.port==8080` en el filtro de Analizador de mensajes reemplazando 8080 por el puerto de servicio de proxy. Seleccione **Apply** (Aplicar) para ver los resultados del filtro.

El filtro anterior solo muestra las solicitudes y respuestas HTTPS hacia o desde el puerto proxy. Ahora busque las solicitudes de CONNECT que muestren la comunicación con el servidor proxy. Si se realiza correctamente, obtiene una respuesta HTTP OK (200).

Si ve otros códigos de respuesta, como 407 o 502, el proxy requiere autenticación o no permite el tráfico por algún otro motivo. En este momento, póngase en contacto con el equipo de soporte técnico del servidor proxy.

## <a name="next-steps"></a>Pasos siguientes

* [Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md)
* Si tiene problemas de conectividad con los conectores, plantee una pregunta en la [Página de preguntas y respuestas de Microsoft sobre Azure Active Directory](https://docs.microsoft.com/answers/topics/azure-active-directory.html) o cree una incidencia para nuestro equipo de soporte técnico.
