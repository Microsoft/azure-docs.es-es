---
title: Acceso Azure VMware Solution by CloudSimple desde un entorno local
titleSuffix: Azure VMware Solution by CloudSimple
description: Acceso a la Azure VMware Solution by CloudSimple desde la red local a través de un firewall
author: sharaths-cs
ms.author: dikamath
ms.date: 08/08/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 6ff057d99e29c7c6fe30e77f38a0bff265dbe7bf
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86998905"
---
# <a name="accessing-your-cloudsimple-private-cloud-environment-and-applications-from-on-premises"></a>Acceso al entorno y las aplicaciones de la nube privada de CloudSimple desde el entorno local

Se puede configurar una conexión desde la red local a CloudSimple con Azure ExpressRoute o VPN de sitio a sitio.  Acceda al vCenter de la nube privada de CloudSimple y a las cargas de trabajo que ejecute en la nube privada mediante la conexión.  Puede controlar qué puertos están abiertos en la conexión mediante un firewall en la red local.  En este artículo se abordan algunos de los requisitos habituales de los puertos para las aplicaciones.  En el caso de otras aplicaciones, consulte la documentación pertinente de esta para obtener información sobre los requisitos de los puertos.

## <a name="ports-required-for-accessing-vcenter"></a>Puertos necesarios para acceder a vCenter

Para acceder al vCenter de la nube privada y al administrador de NSX-T, los puertos definidos en la tabla siguiente deben estar abiertos en el firewall local.  

| Port       | Source                           | Destination                      | Propósito                                                                                                                |
|------------|----------------------------------|----------------------------------|------------------------------------------------------------------------------------------------------------------------|
| 53 (UDP)   | Servidores DNS locales          | Servidores DNS de la nube privada        | Se requiere para reenviar la búsqueda de DNS de *az.cloudsimple.io* a los servidores DNS de la nube privada desde la red local.       |
| 53 (UDP)   | Servidores DNS de la nube privada        | Servidores DNS locales          | Se requiere para reenviar la búsqueda de DNS de los nombres de dominio locales del vCenter de la nube privada a los servidores DNS en el entorno local. |
| 80 (TCP)   | Red local              | Red de administración de la nube privada | Se requiere para redirigir la dirección URL de vCenter de *http* a *https*.                                                           |
| 443 (TCP)  | Red local              | Red de administración de la nube privada | Se requiere para acceder al vCenter y al administrador de NSX-T desde la red local.                                             |
| 8000 (TCP) | Red local              | Red de administración de la nube privada | Se requiere para la migración de máquinas virtuales del entorno local a la nube privada con vMotion.                                            |
| 8000 (TCP) | Red de administración de la nube privada | Red local              | Se requiere para la migración de máquinas virtuales de la nube privada al entorno local con vMotion.                                            |

## <a name="ports-required-for-using-on-premises-active-directory-as-an-identity-source"></a>Puertos necesarios para usar Active Directory local como origen de identidades

Para configurar Active Directory local como un origen de identidades en vCenter de la nube privada, los puertos definidos en la tabla deben estar abiertos.  Para ver los pasos de configuración, consulte el artículo sobre el [uso de Azure AD como proveedor de identidades para vCenter en la nube privada de CloudSimple](./azure-ad.md).

| Port         | Source                           | Destination                                         | Propósito                                                                                                                                          |
|--------------|----------------------------------|-----------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| 53 (UDP)      | Servidores DNS de la nube privada        | Servidores DNS locales                             | Se requiere para reenviar la búsqueda de DNS de los nombres de dominio de Active Directory local del vCenter de la nube privada a los servidores DNS en el entorno local.          |
| 389 (TCP/UDP) | Red de administración de la nube privada | Controladores de dominio de Active Directory local     | Se requiere para la autenticación de usuario, para la comunicación LDAP desde el servidor vCenter de la nube privada con los controladores de dominio de Active Directory.                |
| 636 (TCP)     | Red de administración de la nube privada | Controladores de dominio de Active Directory local     | Se requiere para la autenticación de usuario, para la comunicación LDAP segura (LDAPS) desde el servidor vCenter de la nube privada con los controladores de dominio de Active Directory. |
| 3268 (TCP)    | Red de administración de la nube privada | Servidores de catálogo global de Active Directory local | Se requiere para la comunicación LDAP en las implementaciones de un controlador de varios dominios.                                                                        |
| 3269 (TCP)    | Red de administración de la nube privada | Servidores de catálogo global de Active Directory local | Se requiere para la comunicación LDAPS en las implementaciones de un controlador de varios dominios.                                                                       |                                           |

## <a name="common-ports-required-for-accessing-workload-virtual-machines"></a>Puertos comunes necesarios para acceder a las máquinas virtuales de carga de trabajo

El acceso a las máquinas virtuales de carga de trabajo que se ejecutan en la nube privada requiere que se abran los puertos en el firewall local.  En la tabla siguiente se muestran algunos de los puertos comunes necesarios y su finalidad.  En cuanto a los requisitos de los puertos específicos de una aplicación, consulte la documentación pertinente de esta.

| Port         | Source                         | Destination                          | Propósito                                                                              |
|--------------|--------------------------------|--------------------------------------|--------------------------------------------------------------------------------------|
| 22 (TCP)      | Red local            | Red de carga de trabajo de la nube privada       | Proteja el acceso del shell a las máquinas virtuales Linux que se ejecutan en la nube privada.              |
| 3389 (TCP)    | Red local            | Red de carga de trabajo de la nube privada       | Escritorio remoto para máquinas virtuales Windows que se ejecutan en la nube privada.                 |
| 80 (TCP)      | Red local            | Red de carga de trabajo de la nube privada       | Acceda a los servidores web implementados en las máquinas virtuales que se ejecutan en la nube privada.        |
| 443 (TCP)     | Red local            | Red de carga de trabajo de la nube privada       | Acceda a los servidores web seguros implementados en las máquinas virtuales que se ejecutan en la nube privada. |
| 389 (TCP/UDP) | Red de carga de trabajo de la nube privada | Red de Active Directory local | Conecte las máquinas virtuales Windows de carga de trabajo a un dominio de Active Directory local.       |
| 53 (UDP)      | Red de carga de trabajo de la nube privada | Red local                  | Acceso del servicio DNS para las máquinas virtuales de carga de trabajo a servidores DNS en el entorno local.         |

## <a name="next-steps"></a>Pasos siguientes

* [Creación y administración de VLAN y subredes](./create-vlan-subnet.md)
* [Conexión a la red local mediante Azure ExpressRoute](./on-premises-connection.md)
* [Configurar una VPN de sitio a sitio desde un entorno local](./vpn-gateway.md)
