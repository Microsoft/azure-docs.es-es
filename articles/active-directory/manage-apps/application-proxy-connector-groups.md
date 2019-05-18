---
title: Publicación de aplicaciones en redes y ubicaciones independientes mediante grupos de conectores en el proxy de aplicación de Azure AD | Microsoft Docs
description: Explica cómo crear y administrar grupos de conectores en el Proxy de aplicación de Azure AD.
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 11/08/2018
ms.author: mimart
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: c22d44b02b3cc25c855361cab17132c46fa04794
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783695"
---
# <a name="publish-applications-on-separate-networks-and-locations-using-connector-groups"></a>Publicación de aplicaciones en redes independientes y ubicaciones mediante grupos de conectores

Los clientes usan el proxy de aplicación de Azure AD en cada vez más escenarios y aplicaciones. Por este motivo, hemos habilitado más topologías para que sea más flexible. Puede crear grupos de conectores del proxy de aplicación a fin de asignar conectores específicos para atender a aplicaciones concretas. Esta funcionalidad proporciona un mayor control y formas de optimizar la implementación del proxy de aplicación. 

Cada conector del proxy de aplicación se asigna a un grupo de conectores. Todos los conectores que pertenecen al mismo grupo funcionan como una unidad independiente de alta disponibilidad y equilibrio de carga. Todos los conectores pertenecen a un grupo de conectores. Si no crea grupos, todos sus conectores estarán en un grupo predeterminado. Su administrador puede crear nuevos grupos y asignar conectores a dichos grupos en Azure Portal. 

Todas las aplicaciones se asignan a un grupo de conectores. Si no crea grupos, todas sus aplicaciones se asignarán a un grupo predeterminado. Sin embargo, si organiza los conectores en grupos, puede establecer que cada aplicación funcione con un grupo de conectores concreto. En este caso, solamente los conectores de ese grupo atienden a la aplicación previa solicitud. Esta característica es útil si las aplicaciones se hospedan en diferentes ubicaciones. Puede crear grupos de conectores en función de la ubicación para que los conectores que se encuentren próximos físicamente a las aplicaciones atiendan siempre a estas.

>[!TIP] 
>Si cuenta con una implementación del proxy de aplicación grande, no asigne ninguna aplicación al grupo de conectores predeterminado. De ese modo, los nuevos conectores no recibirán tráfico directo hasta que los asigne a un grupo de conectores activo. Esta configuración también permite poner los conectores en modo inactivo moviéndolos de nuevo al grupo predeterminado para que se pueda realizar el mantenimiento sin que esto afecte a los usuarios.

## <a name="prerequisites"></a>Requisitos previos
Para agrupar los conectores, debe asegurarse de [tener instalados varios conectores](application-proxy-add-on-premises-application.md). Cuando se instala un nuevo conector, se une automáticamente al grupo de conectores **predeterminado** .

## <a name="create-connector-groups"></a>Creación de grupos de conectores
Siga estos pasos para crear tantos grupos de conectores como desee. 

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
1. Seleccione **Azure Active Directory** > **Aplicaciones empresariales** > **Proxy de aplicación**.
2. Seleccione **Nuevo grupo de conectores**. Aparece la hoja New Connector Group (Nuevo grupo de conectores).

   ![Seleccione Nuevo grupo de conectores](./media/application-proxy-connector-groups/new-group.png)

3. Asigne un nombre al nuevo grupo de conectores y use luego el menú deplegable para seleccionar qué conectores pertenecen a este grupo.
4. Seleccione **Guardar**.

## <a name="assign-applications-to-your-connector-groups"></a>Asignación de aplicaciones a los grupos de conectores
Siga estos pasos para cada una de las aplicaciones que haya publicado con el proxy de aplicación. Puede asignar una aplicación a un grupo de conectores la primera vez que la publique, o bien seguir estos pasos para cambiar la asignación siempre que lo desee.   

1. En el panel de administración del directorio, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > la aplicación que desea asignar a un grupo de conectores > **Proxy de la aplicación**.
2. Use el menú desplegable **Grupo de conectores** para seleccionar el grupo que quiere que use la aplicación.
3. Seleccione **Guardar** para aplicar el cambio.

## <a name="use-cases-for-connector-groups"></a>Casos de uso de grupos de conectores 

Los grupos de conectores son útiles para diversos escenarios, por ejemplo:

### <a name="sites-with-multiple-interconnected-datacenters"></a>Sitios con varios centros de datos interconectados.

Muchas organizaciones tienen muchos centros de datos interconectados. En este caso, desea mantener el mayor tráfico posible dentro del centro de datos, ya que los vínculos entre centros de datos son costosos y lentos. Puede implementar conectores en cada centro de datos para atender solamente las aplicaciones que se encuentran en él. Con este enfoque se minimizan los vínculos entre centros de datos y se proporciona una experiencia totalmente transparente a los usuarios.

### <a name="applications-installed-on-isolated-networks"></a>Aplicaciones instaladas en redes aisladas

Las aplicaciones pueden hospedarse en redes que no forman parte de la red corporativa principal. Puede usar grupos de conectores para instalar conectores específicos en redes aisladas y así permitir también el aislamiento de aplicaciones en la red. Esto suele suceder cuando un proveedor de terceros mantiene una aplicación concreta para su organización. 

Los grupos de conectores permiten instalar conectores específicos en aquellas redes que solo publican determinadas aplicaciones, con lo que resulta más sencillo y seguro externalizar la administración de aplicaciones a proveedores de terceros.

### <a name="applications-installed-on-iaas"></a>Aplicaciones instaladas en IaaS 

En lo que respecta a las aplicaciones instaladas en IaaS para el acceso a la nube, los grupos de conectores proporcionan un servicio común para proteger el acceso a todas las aplicaciones. Los grupos de conectores no crean dependencia adicional en la red corporativa ni fragmentan la experiencia de la aplicación. Los conectores se pueden instalar en todos los centros de datos en la nube y servir solo a las aplicaciones que residen en esa red. Puede instalar varios conectores para lograr alta disponibilidad.

Sirva como ejemplo una organización que tiene varias máquinas virtuales conectadas a su propia red virtual hospedada en IaaS. Para permitir que los empleados usen estas aplicaciones, estas redes privadas están conectadas a la red corporativa mediante VPN de sitio a sitio. De este modo, los empleados que trabajan en un entorno local podrán usarlas sin ningún problema. Sin embargo, puede que este enfoque no sea el ideal para los empleados remotos, ya que requiere contar con otra infraestructura local para enrutar el acceso, como puede ver en el diagrama siguiente:

![Red IaaS de Azure AD](./media/application-proxy-connector-groups/application-proxy-iaas-network.png)
  
Con los grupos de conectores de proxy de aplicación de Azure AD, puede habilitar un servicio común para proteger el acceso a todas las aplicaciones sin necesidad de crear otra dependencia de la red corporativa:

![Varios proveedores de nube (IaaS de Azure AD)](./media/application-proxy-connector-groups/application-proxy-multiple-cloud-vendors.png)

### <a name="multi-forest--different-connector-groups-for-each-forest"></a>Varios bosques: grupos de conectores diferentes para cada bosque

La mayoría de los clientes que han implementado el proxy de aplicación usan sus funcionalidades de inicio de sesión único (SSO) mediante la delegación restringida de kerberos (KCD). Para tal fin, las máquinas del conector deben estar unidas a un dominio que pueda delegar los usuarios a la aplicación. KCD es compatible con las funcionalidades de varios bosques. No obstante, en el caso de las empresas que tienen distintos entornos de varios bosques sin ninguna relación de confianza entre ellos, no se puede usar un solo conector para todos los bosques. 

Entonces, pueden implementarse conectores específicos por cada bosque y establecer que se atiendan a las aplicaciones que se publicaron para dar servicio solo a los usuarios de ese bosque concreto. Cada grupo de conectores representa un bosque diferente. Aunque el inquilino y la mayor parte de la experiencia queda unificada en todos los bosques, pueden asignarse usuarios a sus aplicaciones de bosque mediante los grupos de Azure AD.
 
### <a name="disaster-recovery-sites"></a>Sitios de recuperación ante desastres

Hay dos enfoques diferentes que puede adoptar con un sitio de recuperación ante desastres, dependiendo de cómo estén implementados sus sitios:

* Si el sitio de recuperación ante desastres se basa en el modo activo-activo, que es exactamente igual que el sitio principal, con la misma red y configuración de AD, puede crear los conectores del sitio de recuperación ante desastres en el mismo grupo de conectores que el sitio principal. De esta forma, Azure AD podrá detectar automáticamente las conmutaciones por error.
* Si el sitio de recuperación ante desastres es independiente del principal, puede crear un grupo de conectores diferente en el sitio de recuperación ante desastres, y 1) tener aplicaciones de la copia de seguridad y 2) desviar manualmente la aplicación existente al grupo de conectores de recuperación ante desastres según sea necesario.
 
### <a name="serve-multiple-companies-from-a-single-tenant"></a>Servicio a varias empresas desde un solo inquilino

Hay muchas maneras de implementar un modelo en el que un solo proveedor de servicio implemente y mantenga servicios relacionados con Azure AD para varias empresas. Los grupos de conectores ayudan al administrador a separar los conectores y las aplicaciones en grupos distintos. Un método, que es adecuado para las pequeñas empresas, consiste en contar con un solo inquilino de Azure AD mientras que las diferentes empresas tienen sus propias redes y su propio nombre de dominio. También es válido para los escenarios de fusión y adquisición, donde una sola división de TI presta servicios a varias empresas por motivos normativos o empresariales. 

## <a name="sample-configurations"></a>Configuraciones de ejemplo

Algunos ejemplos que puede implementar incluyen los siguientes grupos de conectores.
 
### <a name="default-configuration--no-use-for-connector-groups"></a>Configuración predeterminada: ningún uso para los grupos de conectores

Si no usa grupos de conectores, la configuración tendría un aspecto similar al siguiente:

![Sin grupos de conectores de Azure AD](./media/application-proxy-connector-groups/application-proxy-sample-config-1.png)
 
Esta configuración es suficiente para pruebas e implementaciones de menor envergadura. También funcionará bien si su organización tiene una topología de red plana.
 
### <a name="default-configuration-and-an-isolated-network"></a>Configuración predeterminada y red aislada

Esta configuración es una evolución de la predeterminada, en la que hay una aplicación específica que se ejecuta en una red aislada, como la red virtual IaaS: 

![Sin grupos de conectores de Azure AD](./media/application-proxy-connector-groups/application-proxy-sample-config-2.png)
 
### <a name="recommended-configuration--several-specific-groups-and-a-default-group-for-idle"></a>Configuración recomendada: varios grupos específicos y un grupo predeterminado para conectores inactivos

La configuración recomendada para organizaciones grandes y complejas consiste en que el grupo de conectores predeterminado actúe como un grupo que no atienda a ninguna aplicación y que se use para los conectores inactivos o recién instalados. Las aplicaciones se atienden con grupos de conectores personalizados. Así es como se genera toda la complejidad de los escenarios descritos anteriormente.

En el ejemplo siguiente, la empresa tiene dos centros de datos, A y B, con dos conectores que atienden a cada sitio. Cada uno de los sitios tienen distintas aplicaciones que se ejecutan en él. 

![Sin grupos de conectores de Azure AD](./media/application-proxy-connector-groups/application-proxy-sample-config-3.png)
 
## <a name="next-steps"></a>Pasos siguientes

* [Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md)
* [Habilitar el inicio de sesión único](what-is-single-sign-on.md)


