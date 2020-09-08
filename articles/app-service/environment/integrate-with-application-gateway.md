---
title: Integración con Application Gateway
description: Obtenga más información sobre la integración de una aplicación de una instancia de App Service Environment con ILB con Azure Application Gateway.
author: ccompy
ms.assetid: a6a74f17-bb57-40dd-8113-a20b50ba3050
ms.topic: article
ms.date: 03/03/2018
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 3b73d528802a8aa33c6122eaf5edfa9d046b6753
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962084"
---
# <a name="integrate-your-ilb-app-service-environment-with-the-azure-application-gateway"></a>Integración de ILB App Service Environment con Azure Application Gateway #

[App Service Environment](./intro.md) es una implementación de Azure App Service en la subred de una red virtual de Azure de cliente. Se puede implementar con un punto de conexión público o privado para el acceso a las aplicaciones. La implementación de App Service Environment con un punto de conexión privado (es decir, un equilibrador de carga interno) se denomina ILB App Service Environment.  

Los firewalls de aplicaciones web ayudan a proteger las aplicaciones web, ya que inspeccionan el tráfico web entrante y bloquean las inyecciones de código SQL, un posible ataque de scripts de sitios, cargas de malware y DDoS de aplicaciones, y otros ataques. También examina las respuestas de los servidores web back-end para prevención de pérdida de datos (DLP). Puede obtener un dispositivo WAF en Azure Marketplace o puede usar [Azure Application Gateway][appgw].

Azure Application Gateway es una aplicación virtual que ofrece un equilibrio de carga de nivel 7, descarga de TLS/SSL y protección mediante firewall de aplicaciones web (WAF). Puede escuchar en una dirección IP pública y enrutar el tráfico hacia el punto de conexión de la aplicación. La siguiente información describe cómo integrar una puerta de enlace de aplicaciones configurada con firewall de aplicaciones web con una aplicación en una instancia de ILB App Service Environment.  

La integración de la puerta de enlace de aplicaciones con ILB App Service Environment se produce en el nivel de la aplicación. Al configurar la puerta de enlace de aplicaciones con ILB App Service Environment, lo hace para aplicaciones específicas de este último. Esta técnica permite hospedar aplicaciones para varios inquilinos seguras en una única instancia de ILB App Service Environment.  

![Puerta de enlace de aplicaciones que apunta a la aplicación de una instancia de ILB App Service Environment][1]

En este tutorial realizará lo siguiente:

* Crear una instancia de Azure Application Gateway.
* Configurar la instancia de Application Gateway para que apunte a una aplicación de la instancia de ILB App Service Environment.
* Configurar la aplicación para que admita el nombre de dominio personalizado.
* Editar el nombre de host público del DNS que apunta a la puerta de enlace de aplicaciones.

## <a name="prerequisites"></a>Prerrequisitos

Para integrar la instancia de Application Gateway con ILB App Service Environment, necesita:

* Una instancia de ILB App Service Environment.
* Una aplicación que se ejecute en la instancia de ILB App Service Environment.
* Un nombre de dominio enrutable de Internet para su uso con la aplicación de la instancia de ILB App Service Environment.
* La dirección del equilibrador de carga interno que usa la instancia de ILB App Service Environment. Esta información se encuentra en el portal de App Service Environment en **Configuración** > **Direcciones IP**:

    ![Ejemplo de lista de direcciones IP que usa ILB App Service Environment][9]
    
* Un nombre DNS público que se utilizará posteriormente para que apunte a la instancia de Application Gateway. 

Para más información sobre la creación de una instancia de App Service Environment de ILB, consulte [Creación y uso de un equilibrador de carga interno con una instancia de App Service Environment de ILB][ilbase].

En este artículo se da por supuesto que desea una instancia de Application Gateway en la misma red virtual de Azure donde se implementa la instancia de App Service Environment. Antes de empezar a crear la instancia de Application Gateway, elija o cree la subred donde la hospedará. 

Debe usar una subred distinta a la denominada GatewaySubnet. Si coloca la instancia de Application Gateway en GatewaySubnet, no podrá crear ninguna puerta de enlace de red virtual posteriormente. 

Tampoco se puede colocar la puerta de enlace en la subred que usa la instancia de ILB App Service Environment. App Service Environment es lo único que puede estar en esta subred.

## <a name="configuration-steps"></a>Pasos de configuración ##

1. En Azure Portal, vaya a **Nuevo** > **Red** > **Application Gateway**.

2. En la zona **Básico**:

   a. Como **Nombre**, escriba el de Azure Application Gateway.

   b. Como **Nivel**, seleccione **WAF**.

   c. Como **Suscripción**, seleccione la misma que usa la red virtual de App Service Environment.

   d. Como **Grupo de recursos**, cree o seleccione el grupo de recursos.

   e. Como **Ubicación**, seleccione la de la red virtual de App Service Environment.

   ![Conceptos básicos nuevos de la creación de instancias de Application Gateway][2]

3. En el área **Configuración**:

   a. Como **Red virtual**, seleccione la de App Service Environment.

   b. Como **Subred**, seleccione donde debe implementarse la instancia de Azure Application Gateway. No utilice GatewaySubnet, ya que esto impediría la creación de puertas de enlace de VPN.

   c. Como **Tipo de dirección IP**, seleccione **Pública**.

   d. Como **Dirección IP pública**, seleccione una dirección IP pública. Si no tiene una, es el momento de crearla.

   e. Como **Protocolo**, seleccione **HTTP** o **HTTPS**. Si configura para HTTPS, deberá proporcionar un certificado PFX.

   f. Como **Firewall de aplicaciones web**, puede habilitar el firewall y también establecerlo en **Detección** o en **Prevención**, según estime oportuno.

   ![Nueva configuración de la creación de Application Gateway][3]
    
4. En la sección **Resumen**, revise la configuración y seleccione **Aceptar**. La instancia de Application Gateway puede tardar algo más de 30 minutos en instalarse.  

5. Una vez instalada, vaya a su portal. Seleccione **Grupo back-end**. Agregue la dirección del equilibrador de carga interno que usa la instancia de ILB App Service Environment.

   ![Configuración del grupo back-end][4]

6. Una vez completado el proceso de configuración del grupo de back-end, seleccione **Sondeos de estado**. Cree un sondeo de estado para el nombre de dominio que desea usar con la aplicación. 

   ![Configuración de sondeos de mantenimiento][5]
    
7. Una vez completado el proceso de configuración de los sondeos de estado, seleccione **Configuración HTTP**. Edite la configuración existente, seleccione **Usar sondeo personalizado** y elija el que ha configurado.

   ![Configuración de HTTP][6]
    
8. Vaya a la sección **Información general** de la instancia de Application Gateway y copie la dirección IP pública que esta usa. Establezca esa dirección IP como registro A del nombre de dominio de la aplicación, o utilice el nombre DNS para esa dirección en un registro CNAME. Es más fácil seleccionar la dirección IP pública y copiarla desde la interfaz de usuario de dirección IP pública que desde el vínculo de la sección **Información general** de Application Gateway. 

   ![Portal de Application Gateway][7]

9. Establezca el nombre de dominio personalizado para la aplicación de la instancia de ILB App Service Environment. Vaya a la aplicación en el portal y, en **Configuración**, seleccione **Dominios personalizados**.

   ![Establecer el nombre de dominio personalizado en la aplicación][8]

En el artículo [Configuración de nombres de dominio personalizados para la aplicación web][custom-domain] encontrará información sobre el tema. Pero para una aplicación de una instancia de ILB App Service Environment, no hay validación del nombre de dominio. Como usted es el propietario del DNS que administra los puntos de conexión de la aplicación, puede colocar todo lo que desee allí. No es necesario que el nombre de dominio personalizado que agregue en este caso esté en el DNS, pero deberá configurarlo con la aplicación. 

Después de completar la instalación y de haber permitido un breve período de tiempo para que los cambios de DNS se propaguen, puede acceder a la aplicación mediante el nombre de dominio personalizado que creó. 


<!--IMAGES-->
[1]: ./media/integrate-with-application-gateway/appgw-highlevel.png
[2]: ./media/integrate-with-application-gateway/appgw-createbasics.png
[3]: ./media/integrate-with-application-gateway/appgw-createsettings.png
[4]: ./media/integrate-with-application-gateway/appgw-backendpool.png
[5]: ./media/integrate-with-application-gateway/appgw-healthprobe.png
[6]: ./media/integrate-with-application-gateway/appgw-httpsettings.png
[7]: ./media/integrate-with-application-gateway/appgw-publicip.png
[8]: ./media/integrate-with-application-gateway/appgw-customdomainname.png
[9]: ./media/integrate-with-application-gateway/appgw-iplist.png

<!--LINKS-->
[appgw]: ../../application-gateway/overview.md
[custom-domain]: ../app-service-web-tutorial-custom-domain.md
[ilbase]: ./create-ilb-ase.md