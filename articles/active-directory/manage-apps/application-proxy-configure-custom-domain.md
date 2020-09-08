---
title: Dominios personalizados en Azure AD Application Proxy
description: Configuración y administración de dominios personalizados en Azure AD Application Proxy.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 10/24/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 6688875385d34fcbece964d43827c6d62ae7ced4
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88587776"
---
# <a name="configure-custom-domains-with-azure-ad-application-proxy"></a>Configuración de dominios personalizados con Azure AD Application Proxy

Al publicar una aplicación a través de Azure Active Directory Application Proxy, se crea una dirección URL externa para los usuarios. Esta dirección URL obtiene el dominio predeterminado *yourtenant-msappproxy.net*. Por ejemplo, si publica una aplicación denominada *Expenses* y el inquilino se llama *Contoso*, la dirección URL externa es *https:\//expenses-contoso.msappproxy.net*. Si quiere usar su propio nombre de dominio en lugar de *msappproxy.net*, puede configurar un dominio personalizado para la aplicación. 

## <a name="benefits-of-custom-domains"></a>Ventajas de los dominios personalizadas

Es aconsejable configurar dominios personalizados para sus aplicaciones siempre que sea posible. Entre los motivos para usar dominios personalizados se incluyen los siguientes:

- Los vínculos entre las aplicaciones funcionan incluso fuera de la red corporativa. Sin un dominio personalizado, si la aplicación tiene vínculos internos codificados de forma rígida a destinos fuera de Application Proxy y los vínculos no se pueden resolver externamente, se interrumpirán. Cuando las direcciones URL internas y externas son las mismas, se evita este problema. Si no puede usar dominios personalizados, consulte [Redirección de los vínculos codificados de manera rígida para las aplicaciones publicadas con Azure AD Application Proxy](../application-proxy-link-translation.md) para conocer otras formas de abordar este problema. 
  
- Los usuarios tendrán una experiencia más sencilla, ya que pueden acceder a la aplicación con la misma dirección URL desde dentro o fuera de la red. No necesitan aprender diferentes direcciones URL internas y externas, o realizar un seguimiento de su ubicación actual. 

- Puede controlar la personalización de marca y crear las direcciones URL que quiera. Un dominio personalizado puede ayudar a dar confianza a los usuarios, ya que estos ven y usan un nombre conocido en lugar de *msappproxy.net*.

- Algunas configuraciones solo funcionarán con dominios personalizados. Por ejemplo, necesita dominios personalizados para aplicaciones que usan Lenguaje de marcado de aserción de seguridad (SAML), como cuando usa Servicios de federación de Active Directory (AD FS) pero no puede usar WS-Federation. Para más información, consulte [Trabajo con aplicaciones para notificaciones en Application Proxy](application-proxy-configure-for-claims-aware-applications.md). 

Si no puede hacer coincidir las direcciones URL internas y externas, no es tan importante usar dominios personalizados, pero puede seguir aprovechando las otras ventajas. 

## <a name="dns-configuration-options"></a>Opciones de configuración de DNS

Hay varias opciones para definir la configuración de DNS, en función de sus requisitos:

### <a name="same-internal-and-external-url-different-internal-and-external-behavior"></a>Misma dirección URL interna y externa, diferente comportamiento interno y externo 

Si no quiere que los usuarios internos se dirijan a través de Application Proxy, puede configurar un *DNS de cerebro dividido*. Una infraestructura de DNS dividido dirige los hosts internos a un servidor de nombres de dominio interno, y los hosts externos a un servidor de nombres de dominio externo, para la resolución de nombres. 

![DNS de cerebro dividido](./media/application-proxy-configure-custom-domain/split-brain-dns.png)

### <a name="different-internal-and-external-urls"></a>Direcciones URL internas y externas diferentes 

Si las direcciones URL internas y externas son diferentes, no es necesario configurar el comportamiento de cerebro dividido, ya que la dirección URL determina el enrutamiento del usuario. En este caso, solo cambiará el DNS externo y enrutará la dirección URL externa al punto de conexión de Application Proxy. 

Al seleccionar un dominio personalizado para una dirección URL externa, en una barra de información se muestra la entrada CNAME que se debe agregar al proveedor DNS externo. Siempre puede ver esta información yendo a la página **Proxy** de aplicación de la aplicación.

## <a name="set-up-and-use-custom-domains"></a>Configuración y uso de dominios personalizados

Para configurar una aplicación local para que use un dominio personalizado, necesita un dominio personalizado de Azure Active Directory comprobado, un certificado PFX para dicho dominio y una aplicación local para configurar. 

### <a name="create-and-verify-a-custom-domain"></a>Creación y comprobación de un dominio personalizado

Para crear y comprobar un dominio personalizado:

1. En Azure Active Directory, seleccione **Nombres de dominio personalizados** en el menú de navegación de la izquierda y luego seleccione **Agregar un dominio personalizado**. 
1. Escriba el nombre de dominio personalizado y seleccione **Agregar dominio**. 
1. En la página dominio, copie la información del registro TXT del dominio. 
1. Vaya al registrador de dominios y cree un nuevo registro TXT para el dominio, según la información de DNS copiada.
1. Después de registrar el dominio, en la página del dominio en Azure Active Directory, seleccione **Comprobar**. Una vez que el estado del dominio es **Comprobado**, puede usar el dominio en todas las configuraciones de Azure AD, incluida Application Proxy. 

Para instrucciones más detalladas, consulte [Incorporación del nombre de dominio personalizado mediante el portal de Azure Active Directory](../fundamentals/add-custom-domain.md).

### <a name="configure-an-app-to-use-a-custom-domain"></a>Configuración de una aplicación para que use un dominio personalizado

Para publicar la aplicación a través de Application Proxy con un dominio personalizado:

1. Para una nueva aplicación, desde Azure Active Directory, seleccione **Aplicaciones empresariales** en el menú izquierdo de navegación. Seleccione **Nueva aplicación**. En la sección **Aplicaciones locales**, seleccione **Incorporación de una aplicación local**. 
   
   En el caso de una aplicación que ya esté en **Aplicaciones empresariales**, selecciónela en la lista y luego seleccione **Proxy de la aplicación** en el panel de navegación de la izquierda. 

2. En la página de configuración de Proxy de aplicación, escriba un **Nombre** si va a agregar su propia aplicación local.

3.  En el campo **Dirección URL interna**, escriba la dirección URL interna de la aplicación.
   
4. En el campo **Dirección URL externa**, despliegue la lista y seleccione el dominio personalizado que desee usar.
   
5. Seleccione **Agregar**.
   
   ![Selección de un dominio personalizado](./media/application-proxy-configure-custom-domain/application-proxy.png)
   
6. Si el dominio ya tiene un certificado, el campo **Certificado** mostrará la información de dicho certificado. En caso contrario, seleccione el campo **Certificado**.
   
   ![Haga clic para cargar un certificado.](./media/application-proxy-configure-custom-domain/certificate.png)
   
7. En la página **Certificados SSL**, busque y seleccione el archivo de certificado PFX. Escriba la contraseña del certificado y seleccione **Cargar certificado**. Para más información sobre los certificados, consulte la sección [Certificados para dominios personalizados](#certificates-for-custom-domains). Si el certificado no es válido o hay un problema con la contraseña, verá un mensaje de error. Las [preguntas frecuentes acerca de Application Proxy](application-proxy-faq.md#application-configuration) contienen algunos pasos de solución de problemas que puede probar.
   
   ![Carga del certificado](./media/application-proxy-configure-custom-domain/ssl-certificate.png)
   
   > [!TIP] 
   > Un dominio personalizado solo necesita cargar su certificado una vez. Después, el certificado cargado se aplica automáticamente al usar el dominio personalizado para otras aplicaciones.
   
8. Si agregó un certificado, en la página **Proxy de aplicación**, seleccione **Guardar**. 
   
9. En la barra de información de la página **Proxy de aplicación**, tenga en cuenta la entrada CNAME que debe agregar a la zona DNS. 
   
   ![Incorporación de la entrada CNAME para DNS](./media/application-proxy-configure-custom-domain/dns-info.png)
   
10. Siga las instrucciones que se indican en [Administración de registros y conjuntos de registros DNS mediante Azure Portal](../../dns/dns-operations-recordsets-portal.md) para agregar un registro DNS que redirija la nueva dirección URL externa al dominio *msappproxy.net*.
   
11. Para comprobar que el registro DNS está configurado correctamente, use el comando [nslookup](https://social.technet.microsoft.com/wiki/contents/articles/29184.nslookup-for-beginners.aspx) para confirmar que la dirección URL externa es accesible y que el dominio *msapproxy.net* aparece como un alias.

Ahora la aplicación está configurada para usar el dominio personalizado. Asegúrese de asignar usuarios a la aplicación antes de probarla o liberarla. 

Para cambiar el dominio de una aplicación, seleccione otro dominio en la lista desplegable en **Dirección URL externa** en la página **Proxy de aplicación** de la aplicación. Cargue un certificado para el dominio actualizado, si es necesario, y actualice el registro de DNS. Si no ve el dominio personalizado que desea en la lista desplegable en **Dirección URL externa**, es posible que no se haya comprobado.

Para instrucciones más detalladas sobre Application Proxy, consulte [Tutorial: Adición de una aplicación local para el acceso remoto mediante Application Proxy en Azure Active Directory](application-proxy-add-on-premises-application.md).

## <a name="certificates-for-custom-domains"></a>Certificados para dominios personalizados

Un certificado crea la conexión TLS segura para el dominio personalizado. 

### <a name="certificate-formats"></a>Formatos de certificado

Debe usar un certificado PFX para asegurarse de que se incluyen todos los certificados intermedios necesarios. El certificado debe incluir la clave privada.

Se admiten los métodos de firma de certificado más comunes, como el nombre alternativo del firmante (SAN). 

Puede usar certificados comodín siempre y cuando el carácter comodín coincida con la dirección URL externa. Debe utilizar certificados comodín para [aplicaciones comodín](application-proxy-wildcard.md). Si desea utilizar el certificado para acceder también a los subdominios, debe agregar los caracteres comodín de subdominio como nombres alternativos del firmante en el mismo certificado. Por ejemplo, un certificado para *\*.adventure-works.com* no funcionará para *\*.apps.adventure-works.com* a menos que agregue *\*.apps.adventure-works.com* como un nombre alternativo del firmante. 

Puede usar los certificados emitidos por su propia infraestructura de clave pública (PKI) si la cadena de certificados está instalada en los dispositivos cliente. Intune puede implementar estos certificados en dispositivos administrados. En el caso de los dispositivos no administrados, debe instalar estos certificados manualmente. 

No se recomienda usar una entidad de certificación raíz privada, ya que la entidad de certificación raíz privada también tendría que insertarse en las máquinas cliente, lo que puede suponer numerosos desafíos.

### <a name="certificate-management"></a>Administración de certificados

Toda la administración de certificados se realiza a través de las páginas individuales de la aplicación. Vaya a la página **Proxy de aplicación** de la aplicación para acceder al campo **Certificado**.

Una vez que se carga un certificado para una aplicación, también se aplicará automáticamente a **nuevas** aplicaciones configuradas que usen el mismo certificado. Tendrá que volver a cargar el certificado para las aplicaciones del inquilino.

Cuando un certificado expire, recibirá una advertencia que le indicará que cargue otro certificado. Si se revoca el certificado, los usuarios pueden ver una advertencia de seguridad al acceder a la aplicación. Para actualizar el certificado para una aplicación, vaya a la página **Proxy de aplicación** de la aplicación, seleccione **Certificado** y cargue un certificado nuevo. Si otras aplicaciones no usan el certificado antiguo, se elimina automáticamente. 

## <a name="next-steps"></a>Pasos siguientes

* [Habilitar el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md) en las aplicaciones publicadas con la autenticación de Azure AD.
* [Acceso condicional](../conditional-access/concept-conditional-access-cloud-apps.md) a las aplicaciones en la nube publicadas.
