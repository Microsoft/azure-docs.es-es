---
title: Preguntas frecuentes sobre la configuración
description: Obtenga respuestas a preguntas frecuentes acerca de los problemas de configuración y administración de Azure App Service.
author: genlin
manager: dcscontentpm
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.topic: article
ms.date: 10/30/2018
ms.author: genli
ms.openlocfilehash: 5545acbfd6bb239b9518fbe352b819f300dafaf0
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962356"
---
# <a name="configuration-and-management-faqs-for-web-apps-in-azure"></a>Preguntas más frecuentes sobre la configuración y administración de Web Apps en Azure

Este artículo contiene respuestas a las preguntas más frecuentes (P+F) sobre los problemas de configuración y administración de la [característica Web Apps de Azure App Service](https://azure.microsoft.com/services/app-service/web/).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="are-there-limitations-i-should-be-aware-of-if-i-want-to-move-app-service-resources"></a>¿Existen limitaciones que debo conocer si quiero mover recursos de App Service?

Si tiene previsto mover recursos de App Service a un nuevo grupo de recursos o suscripción, hay algunas limitaciones que debe conocer. Para más información, consulte [Limitaciones de App Service](../azure-resource-manager/management/move-limitations/app-service-move-limitations.md).

## <a name="how-do-i-use-a-custom-domain-name-for-my-web-app"></a>¿Cómo se usa un nombre de dominio personalizado para una aplicación web?

Para obtener respuestas a preguntas comunes sobre el uso de un nombre de dominio personalizado con su aplicación web de Azure, consulte nuestro vídeo de siete minutos sobre [cómo agregar un nombre de dominio personalizado](https://channel9.msdn.com/blogs/Azure-App-Service-Self-Help/Add-a-Custom-Domain-Name). El vídeo ofrece un tutorial sobre cómo agregar un nombre de dominio personalizado. Describe cómo usar su propia dirección URL en lugar de *.azurewebsites.net con la aplicación web de App Service. También puede ver un tutorial detallado de [cómo asignar un nombre de dominio personalizado](app-service-web-tutorial-custom-domain.md).


## <a name="how-do-i-purchase-a-new-custom-domain-for-my-web-app"></a>¿Cómo se compra un nuevo dominio personalizado para una aplicación web?

Para saber cómo comprar y configurar un dominio personalizado para su aplicación web de App Service, consulte [Comprar y configurar un nombre de dominio personalizado en Azure App Service](manage-custom-dns-buy-domain.md).


## <a name="how-do-i-upload-and-configure-an-existing-tlsssl-certificate-for-my-web-app"></a>¿Cómo se carga y configura un certificado TLS/SSL existente para una aplicación web?

Para obtener información sobre cómo cargar y configurar un certificado TLS/SSL personalizado, consulte [Adición de un certificado TLS/SSL a la aplicación App Service](configure-ssl-certificate.md).


## <a name="how-do-i-purchase-and-configure-a-new-tlsssl-certificate-in-azure-for-my-web-app"></a>¿Cómo se compra y configura un nuevo certificado TLS/SSL en Azure para una aplicación web?

Para saber cómo comprar y configurar un certificado TLS/SSL para la aplicación web App Service, consulte [Adición de un certificado TLS/SSL a la aplicación App Service](configure-ssl-certificate.md).


## <a name="how-do-i-move-application-insights-resources"></a>¿Cómo se mueven los recursos de Application Insights?

Actualmente, Azure Application Insights no admite la operación de movimiento. Si su grupo de recursos original incluye un recurso de Application Insights, no puede mover ese recurso. Si se incluye el recurso de Application Insights al intentar mover una aplicación de App Service, la operación entera de movimiento dará error. Sin embargo, no es necesario que el plan de App Service y Application Insights residan en el mismo grupo de recursos que la aplicación para que esta funcione correctamente.

Para más información, consulte [Limitaciones de App Service](../azure-resource-manager/management/move-limitations/app-service-move-limitations.md).

## <a name="where-can-i-find-a-guidance-checklist-and-learn-more-about-resource-move-operations"></a>¿Dónde puedo encontrar una lista de comprobación guía y aprender más sobre las operaciones de movimiento de recursos?

En [Limitaciones de App Service](../azure-resource-manager/management/move-limitations/app-service-move-limitations.md) se muestra cómo mover los recursos a una nueva suscripción o a un nuevo grupo de recursos de la misma suscripción. Podrá obtener información sobre la lista de comprobación de movimiento de recursos, aprender qué servicios admite la operación de movimiento y saber más sobre las limitaciones de App Service y otros temas.

## <a name="how-do-i-set-the-server-time-zone-for-my-web-app"></a>¿Cómo se establece la zona horaria del servidor para una aplicación web?

Para establecer la zona horaria de servidor para la aplicación web:

1. En Azure Portal, en la suscripción de App Service, vaya al menú **Configuración de la aplicación**.
2. En **Configuración de la aplicación**, agregue este valor:
    * Clave = WEBSITE_TIME_ZONE
    * Valor = *la zona horaria que desea*
3. Seleccione **Guardar**.

Para los servicios de aplicación que se ejecutan en Windows, consulte la columna **Zona horaria** en el artículo [Zonas horarias predeterminadas](/windows-hardware/manufacture/desktop/default-time-zones) para ver los valores aceptados. En el caso de los servicios de aplicaciones que se ejecutan en Linux, establezca el [nombre de la base de datos de TZ](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) como el valor de zona horaria. Este es un ejemplo de nombre de base de datos de TZ: América/Adak.

## <a name="why-do-my-continuous-webjobs-sometimes-fail"></a>¿Por qué mis trabajos web continuos en ocasiones dan error?

De forma predeterminada, las aplicaciones web se descargan si están inactivas durante algún tiempo. Esto permite que el sistema conserve recursos. En los planes Básico y Estándar, puede activar el valor **Siempre activado** para que la aplicación web se mantenga cargada en todo momento. Si la aplicación web ejecuta trabajos web continuos, debe seleccionar **Siempre activado** o dichos trabajos podrían no ejecutarse de manera confiable. Para más información, consulte [Creación de un trabajo web de ejecución continua](webjobs-create.md#CreateContinuous).

## <a name="how-do-i-get-the-outbound-ip-address-for-my-web-app"></a>¿Cómo se obtiene una dirección IP de salida para una aplicación web?

Para obtener la lista de direcciones IP de salida para la aplicación web:

1. En Azure Portal, en la hoja de la aplicación web, vaya al menú **Propiedades**.
2. Busque **direcciones ip de salida**.

Aparece la lista de direcciones IP de salida.

Para saber cómo obtener la dirección IP de salida si su sitio web está hospedado en App Service Environment, consulte [Direcciones de red de salida](environment/app-service-app-service-environment-network-architecture-overview.md#outbound-network-addresses).

## <a name="how-do-i-get-a-reserved-or-dedicated-inbound-ip-address-for-my-web-app"></a>¿Cómo se obtiene una dirección IP de entrada reservada o dedicada para una aplicación web?

Para configurar una dirección IP dedicada o reservada para las llamadas entrantes realizadas a su sitio web de Azure, instale y configure un certificado TLS/SSL basado en IP.

Tenga en cuenta que para usar una dirección IP dedicada o reservada para las llamadas entrantes, el plan de App Service debe estar en un plan de servicio Básico o superior.

## <a name="can-i-export-my-app-service-certificate-to-use-outside-azure-such-as-for-a-website-hosted-elsewhere"></a>¿Se puede exportar el certificado de App Service para usarlo fuera de Azure, por ejemplo, en un sitio web hospedado en otra parte? 

Sí, lo puede exportar para usarlo fuera de Azure. Para más información, consulte las [preguntas más frecuentes sobre los dominios personalizados y los certificados de App Service](https://social.msdn.microsoft.com/Forums/azure/f3e6faeb-5ed4-435a-adaa-987d5db43b80/faq-on-app-service-certificates-and-custom-domains?forum=windowsazurewebsitespreview).

## <a name="can-i-export-my-app-service-certificate-to-use-with-other-azure-cloud-services"></a>¿Se puede exportar un certificado de App Service para usarlo con otros servicios en la nube de Azure?

El portal proporciona una experiencia de primera clase para la implementación de un certificado de servicio en aplicaciones de App Service mediante Azure Key Vault. No obstante, se han recibido solicitudes de clientes para usar estos certificados fuera de la plataforma de App Service, por ejemplo, con Azure Virtual Machines. Para aprender a crear una copia PFX local de su certificado de App Service de forma que pueda usarlo con otros recursos de Azure, consulte la [entrada del blog sobre cómo crear una copia PFX local de un certificado de App Service](https://blogs.msdn.microsoft.com/appserviceteam/2017/02/24/creating-a-local-pfx-copy-of-app-service-certificate/).

Para más información, consulte las [preguntas más frecuentes sobre los dominios personalizados y los certificados de App Service](https://social.msdn.microsoft.com/Forums/azure/f3e6faeb-5ed4-435a-adaa-987d5db43b80/faq-on-app-service-certificates-and-custom-domains?forum=windowsazurewebsitespreview).


## <a name="why-do-i-see-the-message-partially-succeeded-when-i-try-to-back-up-my-web-app"></a>¿Por qué aparece un mensaje "Realizado parcialmente" cuando intento hacer una copia de seguridad de mi aplicación web?

Una causa común de errores en la copia de seguridad es que algunos archivos están en uso por la aplicación. Estos archivos se bloquean mientras se realiza la copia de seguridad. Como consecuencia, no se puede hacer una copia de seguridad de ellos y podría aparecer el estado "Realizado parcialmente". Existe una posibilidad de impedir que esto ocurra y es excluir los archivos del proceso de copia de seguridad. Puede elegir hacer copia de seguridad solo de lo necesario. Para más información, consulte [Backup just the important parts of your site with Azure web apps](https://zainrizvi.io/blog/creating-partial-backups-of-your-site-with-azure-web-apps/) (Copia de seguridad de solo las partes importantes de un sitio con aplicaciones web de Azure).

## <a name="how-do-i-remove-a-header-from-the-http-response"></a>¿Cómo se quita un encabezado de la respuesta HTTP?

Para quitar los encabezados de la respuesta HTTP, actualice el archivo web.config de su sitio. Para más información, consulte [Remove standard server headers on your Azure websites](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/) (Eliminación de encabezados de servidor estándar en los sitios web de Azure).

## <a name="is-app-service-compliant-with-pci-standard-30-and-31"></a>¿Es compatible App Service con el Estándar de PCI 3.0 y 3.1?

Actualmente, la característica Web Apps de Azure App Service es compatible con la versión 3.0, nivel 1 de PCI Data Security Standard (DSS). La versión 3.1 de PCI DSS está en nuestros planes. Ya está en marcha el plan de adopción del último estándar.

Para la certificación de la versión 3.1 de PCI DSS es necesario deshabilitar la versión 1.0 del protocolo Seguridad de la capa de transporte (TLS). En la actualidad, la deshabilitación de TLS 1.0 no es una opción en la mayoría de los planes de App Service. Sin embargo, si usa App Service Environment o está dispuesto a migrar su carga de trabajo a App Service Environment, puede obtener un mayor control de su entorno. Para ello, es necesario deshabilitar TLS 1.0 con la ayuda del soporte técnico de Azure. En un futuro próximo, tenemos pensado hacer que esos valores sean accesibles para los usuarios.

Para más información, consulte el artículo sobre el [cumplimiento de las aplicaciones web de Microsoft Azure App Service con el Estándar de PCI 3.0 y 3.1](https://support.microsoft.com/help/3124528).

## <a name="how-do-i-use-the-staging-environment-and-deployment-slots"></a>¿Cómo se usan el entorno de ensayo y las ranuras de implementación?

En los planes Premium y Standard de App Service, al implementar la aplicación web en App Service, puede hacerlo en una ranura de implementación independiente distinta a la de producción predeterminada. Los espacios de implementación son aplicaciones web activas que tienen sus propios nombres de host. Los elementos de contenido y configuración de aplicaciones web se pueden intercambiar entre dos espacios de implementación, incluida la ranura de producción.

Para más información sobre el uso de las ranuras de implementación, consulte [Configuración de entornos de ensayo en Azure App Service](deploy-staging-slots.md).

## <a name="how-do-i-access-and-review-webjob-logs"></a>¿Cómo se accede y se revisan registros de WebJob?

Para revisar los registros de WebJob:

1. Inicie sesión en su **sitio web de Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
2. Seleccione la instancia de WebJob.
3. Seleccione el botón **Alternar salida**.
4. Para descargar el archivo de salida, seleccione el vínculo **Descargar**.
5. Para ejecuciones individuales, seleccione **Individual Invoke** (Invocación individual).
6. Seleccione el botón **Alternar salida**.
7. Seleccione el vínculo de descarga.

## <a name="im-trying-to-use-hybrid-connections-with-sql-server-why-do-i-see-the-message-systemoverflowexception-arithmetic-operation-resulted-in-an-overflow"></a>Estoy intentando usar las conexiones híbridas con SQL Server. ¿Por qué aparece el mensaje "System.OverflowException: la operación aritmética ha provocado un desbordamiento"?

Si usa conexiones híbridas para acceder a SQL Server, una actualización de Microsoft .NET del 10 de mayo de 2016 podría provocar un error en las conexiones. Puede que aparezca este mensaje:

```
Exception: System.Data.Entity.Core.EntityException: The underlying provider failed on Open. —> System.OverflowException: Arithmetic operation resulted in an overflow. or (64 bit Web app) System.OverflowException: Array dimensions exceeded supported range, at System.Data.SqlClient.TdsParser.ConsumePreLoginHandshake
```

### <a name="resolution"></a>Solución

La excepción se produjo debido a un problema con el Administrador de conexiones híbridas que ya se ha corregido. Asegúrese de [actualizar el Administrador de conexiones híbridas](https://go.microsoft.com/fwlink/?LinkID=841308) para resolver este problema.

## <a name="how-do-i-add-a-url-rewrite-rule"></a>¿Cómo se agrega una regla de reescritura de direcciones URL?

Para agregar una regla de reescritura de URL, cree un archivo web.config con las entradas de configuración correspondientes en la carpeta **wwwroot**. Para más información, consulte [Azure App Services: Understanding URL rewrite](/archive/blogs/madhurabharadwaj/azure-app-services-understanding-url-re-write) (Azure App Services: descripción de la reescritura de URL).

## <a name="how-do-i-control-inbound-traffic-to-app-service"></a>¿Cómo se controla el tráfico de entrada a App Service?

En el nivel de sitio, tiene dos opciones para controlar el tráfico de entrada a App Service:

* Activar las restricciones de IP dinámicas. Para saber cómo activar las restricciones de IP dinámicas, consulte [IP and domain restrictions for Azure websites](https://azure.microsoft.com/blog/ip-and-domain-restrictions-for-windows-azure-web-sites/) (Restricciones de dominio e IP para sitios web de Azure).
* Active Module Security. Para saber cómo activar Module Security, consulte [ModSecurity web application firewall on Azure websites](https://azure.microsoft.com/blog/modsecurity-for-azure-websites/) (Firewall de aplicaciones web de ModSecurity en sitios web de Azure).

Si usa App Service Environment, puede emplear el [firewall de Barracuda](https://azure.microsoft.com/blog/configuring-barracuda-web-application-firewall-for-azure-app-service-environment/).

## <a name="how-do-i-block-ports-in-an-app-service-web-app"></a>¿Cómo se bloquean los puertos en una aplicación web de App Service?

En el entorno de inquilinos compartidos de App Service, no es posible bloquear puertos específicos debido a la naturaleza de la infraestructura. Los puertos TCP 4020, 4022 y 4024 podrían estar también abiertos para la depuración remota de Visual Studio.

En App Service Environment, dispone de control total sobre el tráfico de entrada y de salida. Puede usar grupos de seguridad de red para restringir o bloquear puertos específicos. Para más información sobre App Service Environment, consulte [Introducing App Service Environment](https://azure.microsoft.com/blog/introducing-app-service-environment/) (Introducción a App Service Environment).

## <a name="how-do-i-capture-an-f12-trace"></a>¿Cómo se captura un seguimiento F12?

Para capturar un seguimiento F12, tiene dos opciones:

* Seguimiento HTTP de F12
* Salida de consola de F12

### <a name="f12-http-trace"></a>Seguimiento HTTP de F12

1. En Internet Explorer, vaya a su sitio web. Es importante iniciar sesión antes de realizar los siguientes pasos. En caso contrario, el seguimiento F12 captura los datos de inicio de sesión confidenciales.
2. Presione F12.
3. Compruebe que esté seleccionada la pestaña **Red** y luego seleccione el botón verde **Reproducir**.
4. Realice los pasos que reproducen el problema.
5. Seleccione el botón rojo **Detener**.
6. Seleccione el botón **Guardar** (icono de disco) y guarde el archivo HAR (en Internet Explorer y Microsoft Edge) *o* haga clic con el botón derecho en él y seleccione **Save as HAR with content** (Guardar como HAR con contenido) (en Chrome).

### <a name="f12-console-output"></a>Salida de consola de F12

1. Seleccione la pestaña **Consola**.
2. Para cada pestaña que contenga más de cero elementos, seleccione la pestaña (**Error**, **Advertencia** o **Información**). Si la pestaña no está seleccionada, el icono de la pestaña está de color gris o negro al mover el cursor fuera de ella.
3. Haga clic con el botón derecho en el área de mensajes del panel y luego seleccione **Copiar todo**.
4. Pegue el texto copiado en un archivo y luego guarde el archivo.

Para ver un archivo HAR, puede usar el [visor de HAR](http://www.softwareishard.com/har/viewer/).

## <a name="why-do-i-get-an-error-when-i-try-to-connect-an-app-service-web-app-to-a-virtual-network-that-is-connected-to-expressroute"></a>¿Por qué recibo un error al intentar conectar una aplicación web de App Service a una red virtual que está conectada a ExpressRoute?

Si intenta conectar una aplicación web de Azure a una red virtual que está conectada a ExpressRoute de Azure, se produce un error. Aparece el mensaje siguiente: "La puerta de enlace no es una puerta de enlace de VPN".

Actualmente, no puede tener conexiones VPN de punto a sitio a una red virtual que esté conectada a ExpressRoute. Una VPN de punto a sitio y ExpressRoute no pueden coexistir en la misma red virtual. Para más información, consulte [Límites y limitaciones de ExpressRoute y las conexiones VPN de sitio a sitio](../expressroute/expressroute-howto-coexist-classic.md#limits-and-limitations).

## <a name="how-do-i-connect-an-app-service-web-app-to-a-virtual-network-that-has-a-static-routing-policy-based-gateway"></a>¿Cómo se conecta una aplicación web de App Service a una red virtual que tiene una puerta de enlace de enrutamiento estático (basado en directivas)?

Actualmente, no se admite la conexión de una aplicación web de App Service a una red virtual que tenga una puerta de enlace de enrutamiento estático (basado en directivas). Si su red virtual de destino ya existe, debe tener habilitada la VPN de punto a sitio, con una puerta de enlace de enrutamiento dinámico, para poder conectarse a una aplicación. Si la puerta de enlace está configurada para el enrutamiento estático, no podrá habilitar una VPN de punto a sitio. 

Para más información, consulte [Integración de su aplicación con una red virtual de Azure](web-sites-integrate-with-vnet.md).

## <a name="in-my-app-service-environment-why-can-i-create-only-one-app-service-plan-even-though-i-have-two-workers-available"></a>En App Service Environment, ¿por qué solo se puede crear un plan de App Service si tengo dos trabajos disponibles?

Para proporcionar tolerancia a errores, App Service Environment requiere que para cada grupo de trabajo haya al menos un recurso de proceso adicional. No se puede asignar una carga de trabajo al recurso de proceso adicional.

Para más información, consulte [Creación de App Service Environment](environment/app-service-web-how-to-create-an-app-service-environment.md).

## <a name="why-do-i-see-timeouts-when-i-try-to-create-an-app-service-environment"></a>¿Por qué veo los tiempos de espera al intentar crear una instancia de App Service Environment?

En ocasiones, la creación de una instancia de App Service Environment produce error. En ese caso, verá el siguiente error en los registros de actividad:
```
ResourceID: /subscriptions/{SubscriptionID}/resourceGroups/Default-Networking/providers/Microsoft.Web/hostingEnvironments/{ASEname}
Error:{"error":{"code":"ResourceDeploymentFailure","message":"The resource provision operation did not complete within the allowed timeout period."}}
```

Para resolver este problema, asegúrese de que ninguna de las siguientes condiciones sean ciertas:
* La subred sea demasiado pequeña.
* La subred no esté vacía.
* ExpressRoute impida los requisitos de conectividad de red de una instancia de App Service Environment.
* Un grupo de seguridad de red incorrecto impida los requisitos de conectividad de red de una instancia de App Service Environment.
* La tunelización forzada esté activada.

Para más información, consulte [Frequent issues when deploying (creating) a new Azure App Service Environment](/archive/blogs/waws/most-frequent-issues-when-deploying-creating-a-new-azure-app-service-environment-ase) (Problemas frecuentes al implementar [crear] una nueva instancia de Azure App Service Environment).

## <a name="why-cant-i-delete-my-app-service-plan"></a>¿Por qué no puedo eliminar mi plan de App Service?

No podrá eliminar un plan de App Service si alguna de las aplicaciones de App Service está asociada con dicho plan. Antes de eliminar un plan de App Service, quite todas las aplicaciones asociadas.

## <a name="how-do-i-schedule-a-webjob"></a>¿Cómo se programa un trabajo web?

Puede crear un trabajo web programado mediante expresiones Cron:

1. Cree un archivo settings.job.
2. En este archivo JSON, incluya una propiedad de programación mediante una expresión Cron: 
    ```json
    { "schedule": "{second}
    {minute} {hour} {day}
    {month} {day of the week}" }
    ```

Para más información sobre los trabajos web programados, consulte [Creación de un trabajo web programado utilizando una expresión CRON](webjobs-create.md#CreateScheduledCRON).

## <a name="how-do-i-perform-penetration-testing-for-my-app-service-app"></a>¿Cómo se realizan pruebas de penetración para una aplicación de App Service?

Para realizar pruebas de penetración, [envíe una solicitud](https://portal.msrc.microsoft.com/engage/pentest).

## <a name="how-do-i-configure-a-custom-domain-name-for-an-app-service-web-app-that-uses-traffic-manager"></a>¿Cómo se configura un nombre de dominio personalizado para una aplicación web de App Service que usa Traffic Manager?

Para aprender a usar un nombre de dominio común con una aplicación de App Service que emplea Azure Traffic Manager para el equilibrio de carga, consulte [Configuración de un nombre de dominio personalizado para una aplicación web de Azure con Traffic Manager](configure-domain-traffic-manager.md).

## <a name="my-app-service-certificate-is-flagged-for-fraud-how-do-i-resolve-this"></a>El certificado de App Service está marcado como fraudulento. ¿Cómo se resuelve este problema?

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Durante la comprobación del dominio de una compra de certificado de App Service, puede aparecer el siguiente mensaje:

"Su certificado se ha marcado por posible fraude. La solicitud está actualmente bajo revisión. Si el certificado no es utilizable en 24 horas, póngase en contacto con el soporte técnico de Azure".

Como indica el mensaje, este proceso de comprobación de fraude puede tardar hasta 24 horas en completarse. Durante este tiempo, seguirá apareciendo el mensaje.

Si el certificado de App Service sigue mostrando este mensaje transcurridas 24 horas, ejecute el siguiente script de PowerShell. El script establece contacto directamente con el [proveedor de certificados](https://www.godaddy.com/) para resolver este problema.

```powershell
Connect-AzAccount
Set-AzContext -SubscriptionId <subId>
$actionProperties = @{
    "Name"= "<Customer Email Address>"
    };
Invoke-AzResourceAction -ResourceGroupName "<App Service Certificate Resource Group Name>" -ResourceType Microsoft.CertificateRegistration/certificateOrders -ResourceName "<App Service Certificate Resource Name>" -Action resendRequestEmails -Parameters $actionProperties -ApiVersion 2015-08-01 -Force   
```

## <a name="how-do-authentication-and-authorization-work-in-app-service"></a>¿Cómo funciona la autenticación y la autorización en App Service?

Para obtener documentación detallada sobre la autenticación y la autorización en App Service, puede consultar la documentación de inicio de sesión de varios proveedores de identidad:
* [Azure Active Directory](configure-authentication-provider-aad.md)
* [Facebook](configure-authentication-provider-facebook.md)
* [Google](configure-authentication-provider-google.md)
* [Cuenta Microsoft](configure-authentication-provider-microsoft.md)
* [Twitter](configure-authentication-provider-twitter.md)

## <a name="how-do-i-redirect-the-default-azurewebsitesnet-domain-to-my-azure-web-apps-custom-domain"></a>¿Cómo se redirige el dominio *.azurewebsites.net predeterminado a un dominio personalizado de la aplicación web de Azure?

Cuando se crea un nuevo sitio web mediante Web Apps en Azure, se asigna un dominio *sitename*.azurewebsites.net predeterminado a su sitio. Si agrega un nombre de host personalizado al sitio y no desea que los usuarios puedan acceder a su dominio *.azurewebsites.net predeterminado, puede redirigir la dirección URL predeterminada. Para aprender a redirigir todo el tráfico desde el dominio predeterminado de su sitio web a su dominio personalizado, consulte [Redirect the default domain to your custom domain in Azure web apps](https://zainrizvi.io/blog/block-default-azure-websites-domain/) (Redirección del dominio predeterminado al dominio personalizado en aplicaciones web de Azure).

## <a name="how-do-i-determine-which-version-of-net-version-is-installed-in-app-service"></a>¿Cómo se determina qué versión de .NET está instalada en App Service?

La manera más rápida de encontrar la versión de Microsoft .NET que está instalada en App Service es usar la consola de Kudu. Puede acceder a la consola de Kudu desde el portal o mediante la dirección URL de su aplicación de App Service. Para obtener instrucciones detalladas, consulte [Determine the installed .NET version in App Service](/archive/blogs/waws/how-to-determine-the-installed-net-version-in-azure-app-services) (Determinación de la versión de .NET instalada en App Service).

## <a name="why-isnt-autoscale-working-as-expected"></a>¿Por qué no funciona el escalado automático según lo previsto?

Si con el escalado automático de Azure no se ha escalado o reducido la instancia de la aplicación web horizontalmente como se esperaba, puede que esté trabajando en un escenario en el que se ha decidido intencionadamente no escalar para evitar un bucle infinito debido a oscilaciones. Esto suele suceder cuando no hay un margen suficiente entre los umbrales de escalado y reducción horizontal. Para aprender a evitar las oscilaciones y leer sobre otros procedimientos recomendados del escalado automático, consulte [Procedimientos recomendados de escalado automático](../azure-monitor/platform/autoscale-best-practices.md#autoscale-best-practices).

## <a name="why-does-autoscale-sometimes-scale-only-partially"></a>¿Por qué a veces el escalado automático solo escala parcialmente?

El escalado automático se desencadena cuando las métricas superan los límites previamente configurados. En ocasiones, puede observar que la capacidad solo está parcialmente llena, en comparación con lo que se esperaba. Esto puede ocurrir cuando el número de instancias que desea no está disponible. En ese escenario, el escalado automático se llena parcialmente con el número de instancias disponibles. Luego, ejecuta la lógica de reequilibrio para obtener más capacidad y asigna el resto de instancias. Tenga en cuenta que esta operación puede tardar unos minutos.

Si, transcurridos unos minutos, no ve el número esperado de instancias, es posible que el relleno parcial fuera suficiente para llevar las métricas dentro de los límites. O bien, el escalado automático podría haber reducido verticalmente porque alcanzó el límite inferior de las métricas.

Si ninguna de estas condiciones se aplica y el problema continúa, envíe una solicitud de soporte técnico.

## <a name="how-do-i-turn-on-http-compression-for-my-content"></a>¿Cómo se activa la compresión HTTP para el contenido?

Para activar la compresión para los tipos de contenido estáticos y dinámicos, agregue el código siguiente al archivo web.config de nivel de aplicación:

```xml
<system.webServer>
    <urlCompression doStaticCompression="true" doDynamicCompression="true" />
</system.webServer>
```

También puede especificar los tipos MIME dinámicos y estáticos específicos que desea comprimir. Para más información, consulte nuestra respuesta a una pregunta del foro en [httpCompression settings on a simple Azure website](https://social.msdn.microsoft.com/Forums/azure/890b6d25-f7dd-4272-8970-da7798bcf25d/httpcompression-settings-on-a-simple-azure-website?forum=windowsazurewebsitespreview) (Configuración de httpCompression en un sitio web de Azure sencillo).

## <a name="how-do-i-migrate-from-an-on-premises-environment-to-app-service"></a>¿Cómo se migra desde un entorno local a App Service?

Para migrar sitios de servidores web de Windows y Linux a App Service, puede usar Azure App Service Migration Assistant. La herramienta de migración crea las aplicaciones web y las bases de datos en Azure que son necesarias y luego publica el contenido. Para más información, consulte [Azure App Service Migration Assistant](https://appmigration.microsoft.com/).