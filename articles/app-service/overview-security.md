---
title: Seguridad
description: Obtenga información acerca de cómo App Service le ayuda a proteger su aplicación y aislarla de las amenazas.
keywords: azure app service, aplicación web, aplicación móvil, api app, aplicación de función, seguridad, seguro, protegido, cumplimiento, conforme, certificado, certificados, https, ftps, tls, confianza, cifrado, cifrar, restricción de ip, autenticación, autorización, authn, autho, msi, managed service identity, secretos, secreto, revisar, revisión, revisiones, versión, aislamiento, aislamiento de red, ddos, mitm
ms.topic: article
ms.date: 08/24/2018
ms.custom: seodec18
ms.openlocfilehash: a6c8aeeaded659fb7fd37409c3d9e495aa711fa6
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88080307"
---
# <a name="security-in-azure-app-service"></a>Seguridad en Azure App Service

En este artículo se muestra cómo [Azure App Service](overview.md) ayuda a proteger su aplicación web, el back-end de la aplicación móvil, la aplicación de API y la [aplicación de función](/azure/azure-functions/). También muestra cómo proteger aún más su aplicación con las características de App Service integradas.

[!INCLUDE [app-service-security-intro](../../includes/app-service-security-intro.md)]

Las secciones siguientes muestran cómo proteger aún más la aplicación de App Service frente a amenazas.

## <a name="https-and-certificates"></a>HTTPS y certificados

App Service permite proteger las aplicaciones con [HTTPS](https://wikipedia.org/wiki/HTTPS). Cuando se crea la aplicación, su nombre de dominio predeterminado (\<app_name>.azurewebsites.net) ya es accesible mediante HTTPS. Si [configura un dominio personalizado para la aplicación](app-service-web-tutorial-custom-domain.md), también debe [protegerla con un certificado TLS/SSL](configure-ssl-bindings.md) para que los exploradores de cliente pueden establecer conexiones HTTPS seguras con su dominio personalizado. App Service admite varios tipos de certificados:

- Certificado administrado de App Service gratuito
- Certificado de App Service
- Certificado de terceros
- Certificado importado de Azure Key Vault

Para más información, consulte [Adición de un certificado TLS/SSL en Azure App Service](configure-ssl-certificate.md).

## <a name="insecure-protocols-http-tls-10-ftp"></a>Protocolos no seguros (HTTP, TLS 1.0, FTP)

Para proteger su aplicación frente a todas las conexiones no cifradas (HTTP), App Service proporciona una configuración en un clic para aplicar HTTPS. Las solicitudes no seguras se descartan inmediatamente antes incluso de que lleguen al código de la aplicación. Para más información, consulte [Aplicación de HTTPS](configure-ssl-bindings.md#enforce-https).

[TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 ya no se considera seguro según las normas del sector, como [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard). App Service permite [exigir TLS 1.1/1.2](configure-ssl-bindings.md#enforce-tls-versions) para deshabilitar los protocolos no actualizados.

App Service admite FTP y FTPS para la implementación de los archivos. Sin embargo, debe utilizarse FTPS en lugar de FTP, si es posible. Si uno o los dos protocolos no están en uso, debe [deshabilitarlos](deploy-ftp.md#enforce-ftps).

## <a name="static-ip-restrictions"></a>Restricciones de IP estática

De forma predeterminada, la aplicación de App Service acepta solicitudes de todas las direcciones IP de Internet, pero puede limitar el acceso a un pequeño subconjunto de direcciones IP. En Windows, App Service permite definir una lista de direcciones IP que tienen permiso para acceder a su aplicación. La lista de direcciones permitidas puede incluir direcciones IP individuales o un intervalo de direcciones IP definido por una máscara de subred. Para más información, consulte [Restricciones de IP estáticas de Azure App Service](app-service-ip-restrictions.md).

En Windows, App Service también permite restringir las direcciones IP dinámicamente mediante el archivo _web.config_. Para más información, consulte [Seguridad de direcciones IP dinámicas\<dynamicIpSecurity>](https://docs.microsoft.com/iis/configuration/system.webServer/security/dynamicIpSecurity/).

## <a name="client-authentication-and-authorization"></a>Autenticación y autorización de clientes

Azure App Service incorpora autenticación y autorización de usuarios o aplicaciones cliente. Cuando están habilitadas, puede iniciar la sesión de usuarios y aplicaciones cliente con poco o ningún código de aplicación. Puede implementar su propia solución de autenticación y autorización o permitir que App Service lo controle automáticamente. El módulo de autenticación y autorización administra las solicitudes web antes de entregarlas al código de aplicación y deniega las solicitudes no autorizadas antes de que lleguen al código.

La funcionalidad de autorización y autenticación de App Service admite varios proveedores de autenticación, incluido Azure Active Directory, cuentas Microsoft, Facebook, Google y Twitter. Para obtener más información, consulte [Autenticación y autorización en Azure App Service](overview-authentication-authorization.md).

## <a name="service-to-service-authentication"></a>Autenticación entre servicios

Al autenticarse con un servicio back-end, App Service proporciona dos mecanismos diferentes dependiendo de sus necesidades:

- **Identidad de servicio**: inicie sesión en el recurso remoto con la identidad de la propia aplicación. App Service permite crear fácilmente una [identidad administrada](overview-managed-identity.md), que puede usar para autenticarse en otros servicios, como [Azure SQL Database](/azure/sql-database/) o [Azure Key Vault](/azure/key-vault/). Para ver un tutorial completo sobre este tema, consulte [Tutorial: protección de la conexión con Azure SQL Database desde App Service mediante una identidad administrada](app-service-web-tutorial-connect-msi.md).
- **En nombre de otra persona (OBO)** : cree un acceso delegado a los recursos remotos en nombre del usuario. Con Azure Active Directory como proveedor de autenticación, la aplicación de App Service puede realizar un inicio de sesión delegada a un servicio remoto, como [Microsoft Graph API](../active-directory/develop/microsoft-graph-intro.md) o una aplicación de API remota en App Service. Para ver un tutorial completo sobre este tema, consulte [Autenticación y autorización de usuarios de un extremo a otro en Azure App Service](tutorial-auth-aad.md).

## <a name="connectivity-to-remote-resources"></a>Conectividad a los recursos remotos

Hay tres tipos de recursos remotos a los que la aplicación puede necesitar acceder: 

- [Recursos de Azure](#azure-resources)
- [Recursos dentro de una red virtual de Azure](#resources-inside-an-azure-virtual-network)
- [Recursos locales](#on-premises-resources)

En cada uno de estos casos, App Service ofrece una manera de crear conexiones seguras; aún así, debe tener en cuenta los procedimientos recomendados de seguridad. Por ejemplo, use siempre conexiones cifradas aunque el recurso de back-end permita conexiones no cifradas. Además, asegúrese de que el servicio de Azure de back-end permite el conjunto mínimo de direcciones IP. Encontrará las direcciones IP de salida de su aplicación en [Direcciones IP de entrada y salida en Azure App Service](overview-inbound-outbound-ips.md).

### <a name="azure-resources"></a>Recursos de Azure

Cuando la aplicación se conecta a recursos de Azure tales como [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) y [Azure Storage](/azure/storage/), la conexión permanece dentro de Azure y no cruza los límites de la red. Sin embargo, la conexión va a través de la red compartida en Azure, por lo que siempre debe asegurarse de que la conexión esté cifrada. 

Si la aplicación se hospeda en un [entorno de App Service](environment/intro.md), deberá [conectarse a los servicios de Azure mediante puntos de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md).

### <a name="resources-inside-an-azure-virtual-network"></a>Recursos dentro de una red virtual de Azure

La aplicación puede acceder a los recursos de una [red virtual de Azure](/azure/virtual-network/) mediante la [integración de redes virtuales](web-sites-integrate-with-vnet.md). La integración se establece con una red virtual mediante una VPN de sitio a punto. Después, la aplicación puede acceder a los recursos de la red virtual con sus direcciones IP privadas. Sin embargo, la conexión de punto a sitio sigue atravesando las redes compartidas en Azure. 

Para aislar por completo la conectividad de los recursos de las redes compartidas en Azure, cree la aplicación en un [entorno de App Service](environment/intro.md). Como un entorno de App Service siempre se implementa en una red virtual dedicada, la conectividad entre la aplicación y los recursos dentro de la red virtual está totalmente aislada. Para otros aspectos de la seguridad de red en un entorno de App Service, consulte [Aislamiento de red](#network-isolation).

### <a name="on-premises-resources"></a>Recursos locales

Hay tres maneras de acceder de forma segura a los recursos locales, por ejemplo, las bases de datos: 

- [Conexiones híbridas](app-service-hybrid-connections.md): establece una conexión de punto a punto con el recurso remoto a través de un túnel TCP. El túnel TCP se establece con TLS 1.2 con las claves de firma de acceso compartido (SAS).
- [Integración de red virtual](web-sites-integrate-with-vnet.md) con VPN de sitio a sitio: tal y como se describe en [Recursos dentro de una red virtual de Azure](#resources-inside-an-azure-virtual-network), pero la red virtual se puede conectar a la red local mediante una [VPN de sitio a sitio](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md). En esta topología de red, la aplicación puede conectarse a los recursos locales igual que a los demás recursos de la red virtual.
- [Entorno de App Service](environment/intro.md) con VPN de sitio a sitio: tal y como se describe en [Recursos dentro de una red virtual de Azure](#resources-inside-an-azure-virtual-network), pero la red virtual se puede conectar a la red local mediante una [VPN de sitio a sitio](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md). En esta topología de red, la aplicación puede conectarse a los recursos locales igual que a los demás recursos de la red virtual.

## <a name="application-secrets"></a>Secretos de aplicación

No almacene los secretos de la aplicación, como las credenciales de la base de datos, los tokens de API y las claves privadas en el código o en archivos de configuración. El enfoque aceptado es acceder a ellos como [variables de entorno](https://wikipedia.org/wiki/Environment_variable), usando el patrón estándar en el lenguaje que prefiera. En App Service, las variables de entorno se definen mediante la [configuración de la aplicación](configure-common.md#configure-app-settings) y, especialmente para las aplicaciones .NET, [cadenas de conexión](configure-common.md#configure-connection-strings). La configuración de la aplicación y las cadenas de conexión se almacenan cifradas en Azure y se descifran solo antes de insertarlas en la memoria de proceso de la aplicación cuando se inicia la aplicación. Las claves de cifrado rotan con regularidad.

También puede integrar la aplicación de App Service con [Azure Key Vault](/azure/key-vault/) para la administración avanzada de secretos. Al [acceder a Key Vault con una identidad administrada](../key-vault/tutorial-web-application-keyvault.md), la aplicación App Service puede acceder de forma segura a los secretos que necesita.

## <a name="network-isolation"></a>Aislamiento de red

Excepto en el plan de tarifa **Aislado**, todos los niveles ejecutan las aplicaciones en la infraestructura de red compartida en App Service. Por ejemplo, las direcciones IP públicas y los equilibradores de carga de front-end se comparten con otros inquilinos. El plan **Aislado** ejecuta las aplicaciones dentro de un [entorno de App Service](environment/intro.md) dedicado para tener un aislamiento de red completo. Los entornos de App Service se ejecutan en su propia instancia de la [red virtual de Azure](/azure/virtual-network/). Le permite: 

- Servir las aplicaciones a través de un punto de conexión público dedicado, con servidores front-end dedicados.
- Servir la aplicación interna usando un equilibrador de carga interno (ILB), que permite el acceso únicamente desde dentro de la red virtual de Azure. El ILB tiene una dirección IP de la subred privada, que aísla completamente las aplicaciones de Internet.
- [Uso de un ILB detrás de un firewall de aplicaciones web (WAF)](environment/integrate-with-application-gateway.md). WAFS ofrece protección de nivel empresarial para las aplicaciones de acceso público, tales como protección frente a DDoS, filtrado de URI y prevención de inyección de SQL.

Para más información, consulte [Introducción a los entornos de Azure App Service](environment/intro.md). 
