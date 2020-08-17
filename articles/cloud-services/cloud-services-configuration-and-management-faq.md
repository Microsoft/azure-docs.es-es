---
title: Preguntas más frecuentes sobre problemas de configuración y administración
titleSuffix: Azure Cloud Services
description: En este artículo se enumeran las preguntas frecuentes sobre la configuración y administración de Microsoft Azure Cloud Services.
services: cloud-services
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 84985660-2cfd-483a-8378-50eef6a0151d
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/23/2018
ms.author: genli
ms.openlocfilehash: 477f38e18f1bee881e4a9df23dd37048f39549be
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092752"
---
# <a name="configuration-and-management-issues-for-azure-cloud-services-frequently-asked-questions-faqs"></a>Configuración y problemas de administración de Microsoft Azure Cloud Services: Preguntas más frecuentes (P+F)

En este artículo se incluyen las preguntas frecuentes sobre la configuración y los problemas de administración de [Microsoft Azure Cloud Services](https://azure.microsoft.com/services/cloud-services). También puede consultar la [página Tamaños de Cloud Services](cloud-services-sizes-specs.md) para obtener información de tamaño.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

**Certificados**

- [¿Por qué está incompleta la cadena de certificados TLS/SSL de mi servicio en la nube?](#why-is-the-certificate-chain-of-my-cloud-service-tlsssl-certificate-incomplete)
- [¿Cuál es el propósito del "Windows Azure Tools Encryption Certificate for Extensions" (certificado de cifrado para extensiones de Windows Azure Tools)?](#what-is-the-purpose-of-the-windows-azure-tools-encryption-certificate-for-extensions)
- [¿Cómo se puede generar una solicitud de firma de certificado (CSR) sin ejecutar RDP en la instancia?](#how-can-i-generate-a-certificate-signing-request-csr-without-rdp-ing-in-to-the-instance)
- [Expira el certificado de administración del servicio en la nube. ¿Cómo se renueva?](#my-cloud-service-management-certificate-is-expiring-how-to-renew-it)
- [¿Cómo se automatiza la instalación de un certificado TLS/SSL principal (.pfx) y un certificado intermedio (.p7b)?](#how-to-automate-the-installation-of-main-tlsssl-certificatepfx-and-intermediate-certificatep7b)
- [¿Cuál es el propósito del certificado "Microsoft Azure Service Management for MachineKey"?](#what-is-the-purpose-of-the-microsoft-azure-service-management-for-machinekey-certificate)

**Supervisión y registro**

- [¿Cuáles son las próximas funcionalidades del servicio en la nube en Azure Portal que pueden ayudar en la administración y supervisión de las aplicaciones?](#what-are-the-upcoming-cloud-service-capabilities-in-the-azure-portal-which-can-help-manage-and-monitor-applications)
- [¿Por qué IIS deja de escribir en el directorio de registro?](#why-does-iis-stop-writing-to-the-log-directory)
- [¿Cómo se puede habilitar el registro de WAD para Cloud Services?](#how-do-i-enable-wad-logging-for-cloud-services)

**Configuración de la red**

- [Configuración del tiempo de expiración de inactividad de Azure Load Balancer?](#how-do-i-set-the-idle-timeout-for-azure-load-balancer)
- [Asociación de una dirección IP estática al servicio en la nube](#how-do-i-associate-a-static-ip-address-to-my-cloud-service)
- [¿Cuáles son las características y funcionalidades que proporcionan los IPS/IDS y DDoS básicos de Azure?](#what-are-the-features-and-capabilities-that-azure-basic-ipsids-and-ddos-provides)
- [¿Cómo se habilita HTTP/2 en máquinas virtuales de Cloud Services?](#how-to-enable-http2-on-cloud-services-vm)

**Permisos**

- [¿Pueden los ingenieros dentro de Microsoft ejecutar instancias de protocolo de escritorio remoto en el servicio en la nube sin permiso?](#can-microsoft-internal-engineers-remote-desktop-to-cloud-service-instances-without-permission)
- [No puedo usar el escritorio remoto con una máquina virtual del servicio en la nube mediante el archivo RDP. Obtengo el siguiente error: Error de autenticación (código: 0x80004005)](#i-cannot-remote-desktop-to-cloud-service-vm--by-using-the-rdp-file-i-get-following-error-an-authentication-error-has-occurred-code-0x80004005)

**Escalado**

- [No puedo escalar más de X instancias](#i-cannot-scale-beyond-x-instances)
- [¿Cómo se puede configurar el escalado automático en función de las métricas de memoria?](#how-can-i-configure-auto-scale-based-on-memory-metrics)

**Genérico**

- [¿Cómo se agrega "nosniff" a un sitio web?](#how-do-i-add-nosniff-to-my-website)
- [¿Cómo se personaliza IIS para un rol web?](#how-do-i-customize-iis-for-a-web-role)
- [¿Cuál es el límite de cuota del servicio en la nube?](#what-is-the-quota-limit-for-my-cloud-service)
- [¿Por qué la unidad de mi máquina virtual del servicio en la nube muestra muy poco espacio libre en disco?](#why-does-the-drive-on-my-cloud-service-vm-show-very-little-free-disk-space)
- [Adición de una extensión antimalware para Cloud Services de forma automática](#how-can-i-add-an-antimalware-extension-for-my-cloud-services-in-an-automated-way)
- [Habilitación de Indicación de nombre de servidor (SNI) para Cloud Services](#how-to-enable-server-name-indication-sni-for-cloud-services)
- [Adición de etiquetas a un servicio en la nube de Azure](#how-can-i-add-tags-to-my-azure-cloud-service)
- [Azure Portal no muestra la versión del SDK de un servicio en la nube. ¿Cómo se puede ver?](#the-azure-portal-doesnt-display-the-sdk-version-of-my-cloud-service-how-can-i-get-that)
- [Deseo apagar el servicio en la nube durante varios meses. ¿Cómo se reduce el costo de facturación del servicio en la nube sin perder la dirección IP?](#i-want-to-shut-down-the-cloud-service-for-several-months-how-to-reduce-the-billing-cost-of-cloud-service-without-losing-the-ip-address)


## <a name="certificates"></a>Certificados

### <a name="why-is-the-certificate-chain-of-my-cloud-service-tlsssl-certificate-incomplete"></a>¿Por qué está incompleta la cadena de certificados TLS/SSL de mi servicio en la nube?
    
Se recomienda que los clientes instalen la cadena de certificados completa (certificado de hoja, certificados intermedios y certificado raíz) en lugar de simplemente el certificado de hoja. Cuando se instala solamente el certificado de hoja, se confía en Windows para que genere la cadena de certificados recorriendo la CTL. Si se producen problemas intermitentes de la red o problemas de DNS en Azure o Windows Update cuando Windows está intentando validar el certificado, es posible que este se considere no válido. Al instalar la cadena de certificados completa se evita este problema. El blog en [How to install a chained SSL certificate](https://blogs.msdn.microsoft.com/azuredevsupport/2010/02/24/how-to-install-a-chained-ssl-certificate/) (cómo instalar una cadena de certificado SSL) muestra cómo hacerlo.

### <a name="what-is-the-purpose-of-the-windows-azure-tools-encryption-certificate-for-extensions"></a>¿Cuál es el propósito del "Windows Azure Tools Encryption Certificate for Extensions" (certificado de cifrado para extensiones de Windows Azure Tools)?

Estos certificados se crean automáticamente cada vez que se agrega una extensión al servicio en la nube. Normalmente, se trata de la extensión WAD o la extensión RDP, pero pueden ser otras, como la extensión del antimalware o la del recopilador de registros. Estos certificados solo se utilizan para cifrar y descifrar la configuración privada de la extensión. Nunca se comprueba la fecha de expiración, por lo que no importa si el certificado ha caducado. 

Puede omitir estos certificados. Si desea limpiar los certificados, puede intentar eliminaros todos. Azure generará un error si se intenta eliminar un certificado que está en uso.

### <a name="how-can-i-generate-a-certificate-signing-request-csr-without-rdp-ing-in-to-the-instance"></a>¿Cómo se puede generar una solicitud de firma de certificado (CSR) sin ejecutar RDP en la instancia?

Consulte la siguiente documentación de guía:

///[Obtaining a certificate for use with Windows Azure Web Sites (WAWS)](https://azure.microsoft.com/blog/obtaining-a-certificate-for-use-with-windows-azure-web-sites-waws/) (Obtención de un certificado para su uso con los sitios web de Windows Azure)

El CSR no es más que un archivo de texto. No tiene que crearse desde la máquina en la que en última instancia se utilizará el certificado. Aunque este documento está escrito para una instancia de App Service, la creación de CSR es genérica y se aplica también a Cloud Services.

### <a name="my-cloud-service-management-certificate-is-expiring-how-to-renew-it"></a>Expira el certificado de administración del servicio en la nube. ¿Cómo se renueva?

Puede usar los siguientes comandos de PowerShell para renovar sus certificados de administración:

```powershell
Add-AzureAccount
Select-AzureSubscription -Current -SubscriptionName <your subscription name>
Get-AzurePublishSettingsFile
```

**Get-AzurePublishSettingsFile** creará un certificado de administración en **Suscripción** > **Certificados de administración** en Azure Portal. El nombre del nuevo certificado es algo así como "YourSubscriptionNam]-[CurrentDate]-credentials".

### <a name="how-to-automate-the-installation-of-main-tlsssl-certificatepfx-and-intermediate-certificatep7b"></a>¿Cómo se automatiza la instalación de un certificado TLS/SSL principal (.pfx) y un certificado intermedio (.p7b)?

Puede automatizar esta tarea mediante un script de inicio (batch/cmd/PowerShell) y registrar dicho script en el archivo de definición de servicio. Agregue el script de inicio y el certificado (archivo. p7b) a la carpeta del proyecto del mismo directorio que el script de inicio.

### <a name="what-is-the-purpose-of-the-microsoft-azure-service-management-for-machinekey-certificate"></a>¿Cuál es el propósito del certificado "Microsoft Azure Service Management for MachineKey"?

Este certificado se usa para cifrar las claves de la máquina en los roles de web de Azure. Para más información, consulte [este aviso](https://docs.microsoft.com/security-updates/securityadvisories/2018/4092731).

Para más información, consulte los siguientes artículos.
- [Configuración y ejecución de tareas de inicio en un servicio en la nube](https://docs.microsoft.com/azure/cloud-services/cloud-services-startup-tasks)
- [Tareas de inicio comunes de un servicio en la nube](https://docs.microsoft.com/azure/cloud-services/cloud-services-startup-tasks-common)

## <a name="monitoring-and-logging"></a>Supervisión y registro

### <a name="what-are-the-upcoming-cloud-service-capabilities-in-the-azure-portal-which-can-help-manage-and-monitor-applications"></a>¿Cuáles son las próximas funcionalidades del servicio en la nube en Azure Portal que pueden ayudar a administrar y supervisar aplicaciones?

Capacidad de generar un certificado nuevo para el Protocolo de escritorio remoto (RDP) disponible próximamente. Como alternativa, puede ejecutar el script siguiente:

```powershell
$cert = New-SelfSignedCertificate -DnsName yourdomain.cloudapp.net -CertStoreLocation "cert:\LocalMachine\My" -KeyLength 20 48 -KeySpec "KeyExchange"
$password = ConvertTo-SecureString -String "your-password" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath ".\my-cert-file.pfx" -Password $password
```
Posibilidad de elegir blob o local para la ubicación de carga de csdef y cscfg, disponible próximamente. Si usa [New-AzureDeployment](/powershell/module/servicemanagement/azure.service/new-azuredeployment?view=azuresmps-4.0.0), puede establecer el valor de cada ubicación.

Capacidad para supervisar las métricas en el nivel de instancia. Hay funcionalidades de supervisión adicionales disponibles en [Supervisión de Cloud Services](cloud-services-how-to-monitor.md).

### <a name="why-does-iis-stop-writing-to-the-log-directory"></a>¿Por qué IIS deja de escribir en el directorio de registro?
Se ha agotado la cuota de almacenamiento local para escribir en el directorio de registro. Para corregir este problema, puede elegir entre tres cosas:
* Habilitar los diagnósticos para IIS y periódicamente mover los diagnósticos al almacenamiento de blobs.
* Quitar manualmente los archivos de registro del directorio de registro.
* Aumentar el límite de cuota para los recursos locales.

Para obtener más información, vea los documentos siguientes:
* [Almacenamiento y visualización de los datos de diagnóstico en Azure Storage](/azure/storage/common/storage-introduction)
* [IIS Logs stops writing in cloud service](https://blogs.msdn.microsoft.com/cie/2013/12/21/iis-logs-stops-writing-in-cloud-service/) (Los registros de IIS dejan de escribir en el servicio en la nube)

### <a name="how-do-i-enable-wad-logging-for-cloud-services"></a>¿Cómo se puede habilitar el registro de WAD para Cloud Services?
Puede habilitar el registro de Windows Azure Diagnostics (WAD) a través de las opciones siguientes:
1. [Habilitar desde Visual Studio](https://docs.microsoft.com/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines#turn-on-diagnostics-in-cloud-service-projects-before-you-deploy-them)
2. [Habilitación mediante código de .NET](https://docs.microsoft.com/azure/cloud-services/cloud-services-dotnet-diagnostics)
3. [Habilitar mediante Powershell](https://docs.microsoft.com/azure/cloud-services/cloud-services-diagnostics-powershell)

Para obtener la configuración actual de WAD en Cloud Services, puede usar el cmd [Get-AzureServiceDiagnosticsExtensions](https://docs.microsoft.com/azure/cloud-services/cloud-services-diagnostics-powershell#get-current-diagnostics-extension-configuration) de PS o puede verlo en el portal en la hoja "Cloud Services --> Extensiones".


## <a name="network-configuration"></a>Network configuration (Configuración de red)

### <a name="how-do-i-set-the-idle-timeout-for-azure-load-balancer"></a>¿Cómo se configura el tiempo de espera de inactividad para Azure Load Balancer?
Puede especificar el tiempo de espera en el archivo de definición se servicio (csdef) de la forma siguiente:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="mgVS2015Worker" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRole1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
    </ConfigurationSettings>
    <Imports>
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
    <Endpoints>
      <InputEndpoint name="Endpoint1" protocol="tcp" port="10100"   idleTimeoutInMinutes="30" />
    </Endpoints>
  </WorkerRole>
```
Consulte [Nuevo: Tiempo de espera de inactividad configurable para Azure Load Balancer](https://azure.microsoft.com/blog/new-configurable-idle-timeout-for-azure-load-balancer/) para más información.

### <a name="how-do-i-associate-a-static-ip-address-to-my-cloud-service"></a>¿Cómo se asocia una dirección IP estática a un servicio en la nube?
Para configurar una dirección IP estática, tiene que crear una dirección IP reservada. Esta dirección IP reservada se puede asociar a un nuevo servicio en la nube o a una implementación existente. Consulte los siguientes documentos para más información:
* [Cómo crear una dirección IP reservada](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#manage-reserved-vips)
* [Reserva de la dirección IP de un servicio en la nube existente](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#reserve-the-ip-address-of-an-existing-cloud-service)
* [Asociación de una dirección IP reservada a un nuevo servicio en la nube](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#associate-a-reserved-ip-to-a-new-cloud-service)
* [Asociación de una dirección IP reservada a una implementación en ejecución](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#associate-a-reserved-ip-to-a-running-deployment)
* [Asociación de una dirección IP reservada a un servicio en la nube mediante un archivo de configuración de servicio](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#associate-a-reserved-ip-to-a-cloud-service-by-using-a-service-configuration-file)

### <a name="what-are-the-features-and-capabilities-that-azure-basic-ipsids-and-ddos-provides"></a>¿Cuáles son las características y funcionalidades que proporcionan los IPS/IDS y DDoS básicos de Azure?
Azure tiene IPS/IDS básicos en los servidores físicos de los centros de datos para la defensa contra amenazas. Además, los clientes pueden implementar soluciones de seguridad de terceros, como firewalls de aplicación web, firewalls de red, antimalware, detección de intrusiones, sistemas de prevención (IDS/IPS) y mucho más. Para más información, consulte ///[Protect your data and assets and comply with global security standards](https://www.microsoft.com/en-us/trustcenter/Security/AzureSecurity) (Protección de datos y recursos y cumplimiento de los estándares de seguridad global).

Microsoft supervisa continuamente servidores, redes y aplicaciones para detectar amenazas. La administración de amenazas de Azure se realiza en varios frentes utilizando detección de intrusiones, prevención de ataques con denegación de servicio distribuido (DDoS), pruebas de penetración, análisis de comportamiento, detección de anomalías y aprendizaje automático para reforzar su defensa constantemente y reducir los riesgos. Microsoft Antimalware para Azure protege Azure Cloud Services y las máquinas virtuales. Además tiene la opción de implementar soluciones de seguridad de terceros, como firewalls de aplicación web, firewalls de red, antimalware, detección de intrusiones, además de sistemas de prevención (IDS/IPS) y mucho más.

### <a name="how-to-enable-http2-on-cloud-services-vm"></a>¿Cómo habilitar HTTP/2 en máquinas virtuales de Cloud Services?

Windows 10 y Windows Server 2016 incluyen compatibilidad con HTTP/2 en el lado servidor y cliente. El cliente (explorador) se conecta al servidor IIS a través de TLS que negocia HTTP/2 a través de extensiones TLS, no es necesario realizar ningún cambio en el lado servidor. Esto se debe a que, a través de TLS, el encabezado h2-14 que especifica el uso de HTTP/2 se envía de forma predeterminada. Si, por otro lado, el cliente envía un encabezado Upgrade para actualizar a HTTP/2, necesita realizar el cambio siguiente en el lado servidor para garantizar que Upgrade funciona y que acabará con una conexión HTTP/2. 

1. Ejecute regedit.exe.
2. Examine la clave del Registro: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HTTP\Parameters.
3. Cree un nuevo valor DWORD denominado **DuoEnabled**.
4. Establezca su valor en 1.
5. Reinicie el servidor.
6. Vaya a **Sitio web predeterminado** y, en **Enlaces**, cree un enlace de TLS con el certificado autofirmado que acaba de crear. 

Para más información, consulte:

- [HTTP/2 on IIS](https://blogs.iis.net/davidso/http2) (HTTP/2 en IIS)
- [Vídeo: HTTP/2 en Windows 10: explorador, aplicaciones y servidor web](https://channel9.msdn.com/Events/Build/2015/3-88)
         

Estos pasos se pueden automatizar mediante una tarea de inicio para que, cada vez que se cree una instancia de PaaS, pueda realizar los cambios anteriores en el registro del sistema. Para más información, consulte [Configuración y ejecución de tareas de inicio para un servicio en la nube](cloud-services-startup-tasks.md).

 
Una vez hecho esto, puede verificar si se ha habilitado HTTP/2 o no mediante alguno de los métodos siguientes:

- Habilite la versión del protocolo en los registros de IIS y busque en dichos registros. Mostrará HTTP/2 en los registros. 
- Habilite la herramienta de desarrollo F12 en Internet Explorer o Microsoft Edge y cambie a la pestaña Red para comprobar el protocolo. 

Para más información, vea [HTTP/2 on IIS](https://blogs.iis.net/davidso/http2) (HTTP/2 en IIS).

## <a name="permissions"></a>Permisos

### <a name="how-can-i-implement-role-based-access-for-cloud-services"></a>¿Cómo se puede implementar el acceso basado en rol para Cloud Services?
Cloud Services no es compatible con el modelo de control de acceso basado en rol (RBAC), ya que no es un servicio basado en Azure Resource Manager.

Consulte [Descripción de los distintos roles en Azure](../role-based-access-control/rbac-and-directory-admin-roles.md).

## <a name="remote-desktop"></a>Escritorio remoto

### <a name="can-microsoft-internal-engineers-remote-desktop-to-cloud-service-instances-without-permission"></a>¿Pueden los ingenieros dentro de Microsoft ejecutar instancias de protocolo de escritorio remoto en el servicio en la nube sin permiso?
Microsoft sigue un proceso estricto que no permite a los ingenieros internos ejecutar instancias de escritorio remoto en su servicio en la nube sin autorización por escrito (correo electrónico u otra comunicación escrita) del propietario o la persona designada a tal fin.

### <a name="i-cannot-remote-desktop-to-cloud-service-vm--by-using-the-rdp-file-i-get-following-error-an-authentication-error-has-occurred-code-0x80004005"></a>No puedo usar el escritorio remoto con una máquina virtual del servicio en la nube mediante el archivo RDP. Obtengo el siguiente error: Error de autenticación (código: 0x80004005)

Este error puede producirse si usa el archivo RDP desde un equipo replicado en Azure Active Directory. Para resolver el problema, siga estos pasos:

1. Haga clic en el archivo RDP que ha descargado y seleccione **Editar**.
2. Agregue "&#92;" delante del nombre de usuario. Por ejemplo, utilice **. \nombre de usuario**, en lugar de **nombre de usuario**.

## <a name="scaling"></a>Ampliación

### <a name="i-cannot-scale-beyond-x-instances"></a>No puedo escalar más allá de X instancias
La suscripción de Azure tiene un límite en el número de núcleos que se pueden usar. El escalado no funcionará si ha usado todos los núcleos disponibles. Por ejemplo, si tiene un límite de 100 núcleos, significa que puede tener 100 instancias de máquina virtual de tamaño A1 para su servicio en la nube, o bien 50 instancias de máquina virtual de tamaño A2.

### <a name="how-can-i-configure-auto-scale-based-on-memory-metrics"></a>¿Cómo se puede configurar el escalado automático en función de las métricas de memoria?

Actualmente no se admite el escalado automático en función de las métricas de memoria para Cloud Services. 

Para solucionar este problema, puede usar Application Insights. El escalado automático admite Application Insights como origen de métricas y puede escalar el número de instancias de rol en función de una métrica de invitado como "Memoria".  Tiene que configurar Application Insights en el archivo de paquete del proyecto del servicio en la nube (*.cspkg) y habilitar la extensión de Azure Diagnostics en el servicio para implementar esta operación.

Para más información acerca de cómo usar una métrica personalizada a través de Application Insights para configurar el escalado automático en Cloud Services, consulte [Introducción al escalado automático mediante métricas personalizadas en Azure](../azure-monitor/platform/autoscale-custom-metric.md)

Para más información acerca de cómo integrar Azure Diagnostics con Application Insights para Cloud Services, consulte [Envío de datos de diagnóstico de Cloud Services, Virtual Machines o Service Fabric a Application Insights](../azure-monitor/platform/diagnostics-extension-to-application-insights.md)

Para más información acerca de cómo habilitar Application Insights para Cloud Services, consulte [Application Insights para Azure Cloud Services](https://docs.microsoft.com/azure/application-insights/app-insights-cloudservices)

Para más información acerca de cómo habilitar Azure Diagnostics Logging para Cloud Services, consulte [Activación de diagnósticos en los proyectos de servicios en la nube antes de implementarlos](/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines#turn-on-diagnostics-in-cloud-service-projects-before-you-deploy-them)

## <a name="generic"></a>Genérico

### <a name="how-do-i-add-nosniff-to-my-website"></a>¿Cómo se agrega "nosniff" a mi sitio web?
Para evitar que los clientes curioseen en los tipos MIME, agregue un ajuste a su archivo *web.config*.

```xml
<configuration>
   <system.webServer>
      <httpProtocol>
         <customHeaders>
            <add name="X-Content-Type-Options" value="nosniff" />
         </customHeaders>
      </httpProtocol>
   </system.webServer>
</configuration>
```

También puede agregarlo como un ajuste en IIS. Use el comando siguiente con el artículo [common startup tasks](cloud-services-startup-tasks-common.md#configure-iis-startup-with-appcmdexe) (tareas comunes de inicio).

```cmd
%windir%\system32\inetsrv\appcmd set config /section:httpProtocol /+customHeaders.[name='X-Content-Type-Options',value='nosniff']
```

### <a name="how-do-i-customize-iis-for-a-web-role"></a>¿Cómo se personaliza IIS para un rol web?
Use el script de inicio de IIS del artículo [common startup tasks](cloud-services-startup-tasks-common.md#configure-iis-startup-with-appcmdexe) (tareas comunes de inicio).

### <a name="what-is-the-quota-limit-for-my-cloud-service"></a>¿Cuál es el límite de cuota para un servicio en la nube?
Consulte los [Límites específicos del servicio](../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits).

### <a name="why-does-the-drive-on-my-cloud-service-vm-show-very-little-free-disk-space"></a>¿Por qué la unidad de mi máquina virtual del servicio en la nube muestra muy poco espacio libre en disco?
Este es el comportamiento esperado y que no debería causar ningún problema para la aplicación. El registro en diario está activado para la unidad %approot% en las máquinas virtuales de Azure PaaS, que esencialmente consume el doble de espacio que suelen ocupar los archivos. De todas formas, hay varios aspectos a tener en cuenta que básicamente hacen que esto no sea un problema.

El tamaño de la unidad %approot% se calcula como \<size of .cspkg + max journal size + a margin of free space> o bien 1,5 GB, lo que sea mayor. El tamaño de la máquina virtual no influye en este cálculo. (El tamaño de la máquina virtual solo afecta al tamaño de la unidad C: temporal). 

No se admite para escribir en la unidad % approot %. Si va a escribir en la máquina virtual de Azure, tiene que hacerlo en un recurso de LocalStorage temporal (u otras opciones, como almacenamiento de blobs, Azure Files, etc.). Por lo que la cantidad de espacio libre en la carpeta % approot % no es algo significativo. Si no está seguro de si la aplicación está escribiendo en la unidad % approot %, siempre puede dejar que el servicio se ejecute durante unos días y, a continuación, comparar los tamaños del "antes" y el "después". 

Azure no escribirá nada en la unidad % approot %. Una vez que el disco duro virtual se crea desde su .cspkg y se monta en la máquina virtual de Azure, lo único que puede escribir en esta unidad es la aplicación. 

Los ajustes del diario no son configurables, por lo que no se pueden desactivar.

### <a name="how-can-i-add-an-antimalware-extension-for-my-cloud-services-in-an-automated-way"></a>¿Cómo puedo agregar una extensión antimalware para Cloud Services de forma automática?

Puede habilitar la extensión antimalware mediante el script de PowerShell en la tarea de inicio. Siga los pasos descritos en los siguientes artículos para implementarla: 
 
- [Creación de una tarea de inicio de PowerShell](cloud-services-startup-tasks-common.md#create-a-powershell-startup-task)
- [Set-AzureServiceAntimalwareExtension](/powershell/module/servicemanagement/azure.service/Set-AzureServiceAntimalwareExtension?view=azuresmps-4.0.0 )

Para obtener más información sobre los escenarios de implementación de antimalware y cómo habilitarlo desde el portal, vea [Escenarios de implementación de Antimalware](../security/fundamentals/antimalware.md#antimalware-deployment-scenarios).

### <a name="how-to-enable-server-name-indication-sni-for-cloud-services"></a>¿Cómo habilitar la Indicación de nombre de servidor (SNI) para Cloud Services?

Puede habilitar SNI en Cloud Services con alguno de los siguientes métodos:

**Método 1: Uso de PowerShell**

El enlace de SNI se puede configurar mediante el cmdlet de PowerShell **New-WebBinding** en una tarea de inicio de una instancia de rol de servicio en la nube como la siguiente:

```powershell
New-WebBinding -Name $WebsiteName -Protocol "https" -Port 443 -IPAddress $IPAddress -HostHeader $HostHeader -SslFlags $sslFlags
```

Como se describe [aquí](https://technet.microsoft.com/library/ee790567.aspx), $sslFlags puede ser uno de los valores del tipo:

|Value|Significado|
------|------
|0|Sin SNI|
|1|SNI habilitada|
|2|Sin enlace de SNI que use el almacén de certificados central|
|3|Enlace de SNI que usa el almacén de certificados central|
 
**Método 2: Uso de código**

El enlace de SNI también puede configurarse a través de código en el inicio del rol, tal y como se describe en esta [entrada de blog](https://blogs.msdn.microsoft.com/jianwu/2014/12/17/expose-ssl-service-to-multi-domains-from-the-same-cloud-service/):

```csharp
//<code snip> 
                var serverManager = new ServerManager(); 
                var site = serverManager.Sites[0]; 
                var binding = site.Bindings.Add(":443:www.test1.com", newCert.GetCertHash(), "My"); 
                binding.SetAttributeValue("sslFlags", 1); //enables the SNI 
                serverManager.CommitChanges(); 
    //</code snip>
```
    
Con cualquiera de los enfoques anteriores, los certificados correspondientes (*.pfx) para los nombres de host específicos deben instalarse primero en las instancias de rol mediante una tarea de inicio o a través de código para que el enlace de SNI sea efectivo.

### <a name="how-can-i-add-tags-to-my-azure-cloud-service"></a>¿Cómo puedo agregar etiquetas a mi servicio en la nube de Azure? 

El servicio en la nube es un recurso clásico. Solo los recursos creados a través de Azure Resource Manager son compatibles con las etiquetas. No puede aplicar etiquetas a los recursos clásicos, como el servicio en la nube. 

### <a name="the-azure-portal-doesnt-display-the-sdk-version-of-my-cloud-service-how-can-i-get-that"></a>Azure Portal no muestra la versión del SDK de un servicio en la nube. ¿Cómo se puede ver?

Estamos trabajando en la inclusión de esta característica en Azure Portal. Entretanto, puede usar los siguientes comandos de PowerShell para ver la versión del SDK:

```powershell
Get-AzureService -ServiceName "<Cloud Service name>" | Get-AzureDeployment | Where-Object -Property SdkVersion -NE -Value "" | select ServiceName,SdkVersion,OSVersion,Slot
```

### <a name="i-want-to-shut-down-the-cloud-service-for-several-months-how-to-reduce-the-billing-cost-of-cloud-service-without-losing-the-ip-address"></a>Deseo apagar el servicio en la nube durante varios meses. ¿Cómo se reduce el costo de facturación del servicio en la nube sin perder la dirección IP?

En un servicio en la nube ya implementado la facturación la dicta los servicios Compute y Storage que utilice. Por consiguiente, aunque apague la máquina virtual de Azure, se facturará el servicio Storage. 

Esto es lo que puede hacer para reducir la facturación sin perder la dirección IP de su servicio:

1. [Reserve la dirección IP](../virtual-network/virtual-networks-reserved-public-ip.md) antes de eliminar las implementaciones.  Solo se le cobrará esta dirección IP. Para más información acerca de la facturación de direcciones IP, consulte [Precios de las direcciones IP](https://azure.microsoft.com/pricing/details/ip-addresses/).
2. Elimine las implementaciones. No elimine xxx.cloudapp.net, para que pueda usarlo en el futuro.
3. Si desea volver a implementar el servicio en la nube con la misma dirección IP que reservó en su suscripción, consulte [Reserved IP addresses for Cloud Services & Virtual Machines](https://azure.microsoft.com/blog/reserved-ip-addresses/) (Direcciones IP reservadas para Cloud Services y Virtual Machines).

