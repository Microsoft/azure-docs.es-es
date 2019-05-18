---
title: Consideraciones sobre la topología de red al utilizar el Proxy de aplicación de Azure Active Directory | Microsoft Docs
description: Se tratan las consideraciones de la topología de red al utilizar el Proxy de aplicación de Azure AD.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/28/2017
ms.author: mimart
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: a9ee8800d84d1090f97fb72980123f59d5aa0375
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783474"
---
# <a name="network-topology-considerations-when-using-azure-active-directory-application-proxy"></a>Consideraciones sobre la topología de red al utilizar el Proxy de aplicación de Azure Active Directory

En este artículo se explican las consideraciones de la topología de red al utilizar el Proxy de aplicación de Azure Active Directory (Azure AD) para la publicación y el acceso a las aplicaciones de forma remota.

## <a name="traffic-flow"></a>Flujo de tráfico

Cuando se publica una aplicación a través del proxy de aplicación de Azure AD, el tráfico de los usuarios a las aplicaciones fluye a través de tres conexiones:

1. El usuario conecta con el punto de conexión público del servicio Azure AD Application Proxy en Azure
2. El servicio Application Proxy se conecta al conector del proxy de aplicación
3. El conector del proxy de aplicación se conecta a la aplicación de destino

![Diagrama que muestra el flujo de tráfico del usuario a la aplicación de destino](./media/application-proxy-network-topology/application-proxy-three-hops.png)

## <a name="tenant-location-and-application-proxy-service"></a>Ubicación del inquilino y el servicio de Proxy de aplicación

Al suscribirse a un inquilino de Azure AD, la región de su inquilino viene determinada por el país o región que especifique. Al habilitar el proxy de aplicación, se muestran las instancias del servicio Application Proxy para el inquilino de la misma región del inquilino de Azure AD o la región más cercana a este.

Por ejemplo, si el país o la región del inquilino de Azure AD es el Reino Unido, todos los conectores del proxy de la aplicación usan las instancias de servicio de los centros de datos en la Unión Europea. Cuando los usuarios acceden a aplicaciones publicadas, el tráfico pasa por las instancias del servicio de proxy de la aplicación de esta ubicación.

## <a name="considerations-for-reducing-latency"></a>Consideraciones para reducir la latencia

Todas las soluciones de proxy introducirán latencia en la conexión de red. Independientemente del proxy o solución VPN que elija como solución de acceso remoto, siempre se incluirá un conjunto de servidores que habilita la conexión hacia su la red corporativa.

Las organizaciones suelen incluir puntos de conexión de servidor en su red perimetral. Sin embargo, con el proxy de la aplicación de Azure AD, el tráfico fluye a través del servicio de proxy en la nube, mientras que los conectores residen en la red corporativa. No se requiere red perimetral.

Las secciones siguientes contienen sugerencias adicionales para ayudar a reducir la latencia aún más. 

### <a name="connector-placement"></a>Colocación del conector

Proxy de aplicación elige automáticamente la ubicación de las instancias en función de la ubicación del inquilino. Sin embargo, corresponde al usuario decidir dónde instalar el conector, con lo que tiene la posibilidad de definir las características de la latencia del tráfico de su red.

Al configurar el servicio de proxy de aplicación, formule las siguientes preguntas:

* ¿Dónde se encuentra la aplicación?
* ¿Dónde se encuentran la mayoría de los usuarios que acceden a la aplicación?
* ¿Dónde se encuentra la instancia del proxy de aplicación?
* ¿Ya tiene configurada una conexión de red dedicada a centros de datos de Azure (como ExpressRoute o una VPN similar)?

El conector tiene que comunicarse con Azure y con sus aplicaciones (pasos 2 y 3 del diagrama de flujo del tráfico); por eso, la ubicación del conector afecta a la latencia de esas dos conexiones. Al evaluar la posición del conector, tenga en cuenta lo siguiente:

* Si desea utilizar la delegación limitada de Kerberos (KCD) para el inicio de sesión único, el conector necesita una línea de visión a un centro de datos. Además, el servidor de conector debe estar unido al dominio.  
* Si no está seguro, instale el conector más cerca de la aplicación.

### <a name="general-approach-to-minimize-latency"></a>Método general para minimizar la latencia

Puede probar a minimizar la latencia del tráfico de extremo a extremo optimizando cada conexión de red. Cada conexión se puede optimizar mediante uno de estos modos:

* Reducir la distancia entre los dos extremos del salto.
* Elegir la red adecuada por la que pasar. Por ejemplo, puede ser más rápido pasar por una red privada en lugar de la red pública de Internet debido a los vínculos dedicados.

Si tiene un vínculo de VPN o ExpressRoute dedicado entre Azure y la red corporativa, puede que desee usarlo.

## <a name="focus-your-optimization-strategy"></a>Centrar la estrategia de optimización

Se puede hacer poco para controlar la conexión entre los usuarios y el servicio Aplication Proxy. Los usuarios pueden tener acceso a las aplicaciones de una red doméstica, una cafetería o un país o región diferente. En cambio, puede optimizar las conexiones del servicio Application Proxy a los conectores del proxy de aplicación con las aplicaciones. Es aconsejable incorporar los siguientes patrones a su entorno.

### <a name="pattern-1-put-the-connector-close-to-the-application"></a>Patrón 1: colocar el conector cerca de la aplicación

Coloque el conector cerca de la aplicación de destino en la red del cliente. Esta configuración reduce al mínimo el paso 3 en el diagrama de la topología, porque el conector y la aplicación son similares. 

Si el conector necesita una línea de visión al controlador de dominio, este modelo es ventajoso. La mayoría de nuestros clientes lo usa, puesto que funciona bien para la mayoría de los escenarios. Este modelo se puede combinar con el 2 para optimizar el tráfico entre el servicio y el conector.

### <a name="pattern-2-take-advantage-of-expressroute-with-microsoft-peering"></a>Patrón 2: sacar partido de ExpressRoute con el emparejamiento de Microsoft

Si tiene una configuración de ExpressRoute con emparejamiento de Microsoft, puede usar la conexión de ExpressRoute más rápida para el tráfico entre el proxy de la aplicación y el conector. El conector sigue en su red; cierre la aplicación.

### <a name="pattern-3-take-advantage-of-expressroute-with-private-peering"></a>Patrón 3: sacar partido de ExpressRoute con el emparejamiento privado

Si tiene una configuración de VPN o ExpressRoute dedicada con emparejamiento privado entre Azure y la red corporativa, tiene otra opción. En esta configuración, la red virtual de Azure normalmente se considera una extensión de la red corporativa. Así pues, puede instalar el conector en el centro de datos de Azure y aun así cumplir los requisitos de latencia baja de la conexión del conector a la aplicación.

La latencia no se ve comprometida porque el tráfico está fluyendo por una red dedicada. También obtiene una mejor latencia del conector al servicio de proxy de la aplicación, ya que el conector está instalado en un centro de datos Azure cerca de la ubicación del inquilino de Azure AD.

![Diagrama que muestra el conector instalado en un centro de datos de Azure](./media/application-proxy-network-topology/application-proxy-expressroute-private.png)

### <a name="other-approaches"></a>Otros enfoques

Aunque el centro de atención de este artículo es la selección de la ubicación del conector, también puede cambiar la ubicación de la aplicación para obtener mejores características de latencia.

Las organizaciones están cambiando sus redes a entornos hospedados cada vez con más frecuencia. Esto les permite colocar sus aplicaciones en un entorno hospedado que también forma parte de su red corporativa y seguir estando dentro del dominio. En este caso, los patrones analizados en las secciones anteriores se pueden aplicar a la nueva ubicación de la aplicación. Si se plantea esta opción, consulte [Servicios de dominio de Azure Active Directory](../../active-directory-domain-services/active-directory-ds-overview.md).

Además, considere organizar los conectores con [grupos de conectores](application-proxy-connector-groups.md) para las aplicaciones de destino que se encuentren en ubicaciones y redes distintas. 

## <a name="common-use-cases"></a>Casos de uso comunes

En esta sección, se abordan algunos de los escenarios habituales. Suponga que el inquilino de Azure AD (y, por tanto, el punto de conexión de servicio del proxy) se encuentra en Estados Unidos. Los aspectos descritos en estos casos de uso también se aplican a otras regiones de todo el mundo.

Para estos escenarios, se llama "salto" a cada una de las conexiones y les asignamos un número para facilitar el análisis:

- **Salto 1**: usuario al servicio Application Proxy
- **Salto 2**: el servicio Application Proxy se conecta al conector del proxy de aplicación
- **Salto 3**: el conector del proxy de aplicación se conecta a la aplicación de destino 

### <a name="use-case-1"></a>Caso de uso 1

**Escenario:** La aplicación se encuentra en la red de una organización en los Estados Unidos con usuarios en la misma región. No hay ExpressRoute ni VPN entre el centro de datos de Azure y la red corporativa.

**Recomendación:** Siga el patrón 1, tal como se explica en la sección anterior. Para mejorar la latencia, considere la posibilidad de usar ExpressRoute, si es necesario.

Se trata de un modelo simple. Puede optimizar el salto 3 colocando el conector cerca de la aplicación. Se trata además de una opción natural, ya que el conector se instala normalmente con línea de visión hacia la aplicación y el centro de datos para realizar operaciones de KCD.

![Diagrama que muestra que los usuarios, el proxy, el conector y la aplicación están en EE. UU.](./media/application-proxy-network-topology/application-proxy-pattern1.png)

### <a name="use-case-2"></a>Caso de uso 2

**Escenario:** La aplicación se encuentra en la red de una organización en los Estados Unidos con usuarios dispersos por todo el mundo. No hay ExpressRoute ni VPN entre el centro de datos de Azure y la red corporativa.

**Recomendación:** Siga el patrón 1, tal como se explica en la sección anterior. 

Nuevamente, el patrón más común es optimizar el salto 3, donde el conector se coloca cerca de la aplicación. El salto 3 no es costoso por lo general si todo se encuentra dentro de la misma región. Sin embargo, el salto 1 puede resultar más costoso dependiendo de dónde esté el usuario, ya que los usuarios de todo el mundo tendrán que acceder a la instancia del proxy de aplicación en Estados Unidos. Merece la pena mencionar que cualquier solución de proxy tendrá similares características con respecto a los usuarios que se reparten por todo el mundo.

![Diagrama que muestra que los usuarios están repartidos por todo el mundo, mientras que el proxy, el conector y la aplicación están en EE. UU.](./media/application-proxy-network-topology/application-proxy-pattern2.png)

### <a name="use-case-3"></a>Caso de uso 3

**Escenario:** La aplicación se encuentra en una red de una organización en los Estados Unidos. Hay una instancia de ExpressRoute con emparejamiento de Microsoft entre Azure y la red corporativa.

**Recomendación:** Siga los modelos 1 y 2 explicados en la sección anterior.

En primer lugar, coloque el conector lo más próximo posible a la aplicación. Entonces, el sistema utilizará automáticamente ExpressRoute para el salto 2. 

Si el vínculo de ExpressRoute usa emparejamiento de Microsoft, el tráfico entre el proxy y el conector pasará por ese vínculo. El salto 2 tiene la latencia optimizada.

![Diagrama que muestra ExpressRoute entre el proxy y el conector](./media/application-proxy-network-topology/application-proxy-pattern3.png)

### <a name="use-case-4"></a>Caso de uso 4

**Escenario:** La aplicación se encuentra en una red de una organización en los Estados Unidos. Hay una instancia de ExpressRoute con emparejamiento privado entre Azure y la red corporativa.

**Recomendación:** Siga el patrón 3, tal como se explica en la sección anterior.

Coloque el conector del centro de datos de Azure que esté conectado a la red corporativa a través del emparejamiento privado de ExpressRoute. 

El conector puede colocarse en el centro de datos de Azure. Puesto que el conector todavía tiene una línea de visión hacia la aplicación y el centro de datos a través de la red privada, el salto 3 se mantiene optimizado. Además, el salto 2 se optimiza aún más.

![Diagrama que muestra el conector en un centro de datos de Azure y ExpressRoute entre el conector y la aplicación](./media/application-proxy-network-topology/application-proxy-pattern4.png)

### <a name="use-case-5"></a>Caso de uso 5

**Escenario:** La aplicación está en una red de la organización en la Unión Europea, mientras que la instancia del proxy de la aplicación y la mayoría de los usuarios se encuentran en EE. UU.

**Recomendación:** Coloque el conector cerca de la aplicación. Puesto que los usuarios de Estados Unidos tienen acceso a una instancia del Proxy de aplicación que resulta estar en la misma región, el salto 1 no es demasiado costoso. El salto 3 se optimiza. Es aconsejable usar ExpressRoute para optimizar el salto 2. 

![Diagrama que muestra los usuarios y el proxy en EE. UU. y el conector la aplicación en la Unión Europea](./media/application-proxy-network-topology/application-proxy-pattern5b.png)

También puede considerar la utilización de alguna otra variante en esta situación. Si la mayoría de los usuarios de la organización están en Estados Unidos, lo más probable es que la red también se extienda a Estados Unidos. Coloque el conector en Estados Unidos y use la línea de red corporativa interna dedicada a la aplicación en la Unión Europea. De esta forma se optimizan los saltos 2 y 3.

![Diagrama que muestra los usuarios, el proxy y el conector en EE. UU. y la aplicación en la Unión Europea](./media/application-proxy-network-topology/application-proxy-pattern5c.png)

## <a name="next-steps"></a>Pasos siguientes

- [Habilitación del proxy de la aplicación](application-proxy-add-on-premises-application.md)
- [Habilitar el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md)
- [Habilitar el acceso condicional](application-proxy-integrate-with-sharepoint-server.md)
- [Solucionar los problemas que tiene con el Proxy de aplicación](application-proxy-troubleshoot.md)
