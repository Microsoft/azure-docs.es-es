---
title: Protección de aplicaciones web y móviles PaaS
titleSuffix: Azure App Service
description: 'Aprenda sobre los procedimientos recomendados de seguridad de Azure App Service para proteger las aplicaciones web y móviles PaaS. '
services: security
documentationcenter: na
author: techlake
manager: barbkess
editor: ''
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/18/2019
ms.author: terrylan
ms.openlocfilehash: fc273c2e6e27c191b4407ffae86a4ba227ee642b
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131045"
---
# <a name="best-practices-for-securing-paas-web-and-mobile-applications-using-azure-app-service"></a>Procedimientos recomendados para proteger aplicaciones web y móviles PaaS con Azure App Service

En este artículo se expone una colección de procedimientos recomendados de seguridad de [Azure App Service](/azure/app-service/overview) para proteger las aplicaciones web y móviles PaaS. Estos procedimientos recomendados proceden de nuestra experiencia con Azure y las experiencias de clientes como usted.

Azure App Service es una oferta de plataforma como servicio (PaaS) que permite crear aplicaciones web y móviles para cualquier plataforma o dispositivo y conectarse a datos en cualquier lugar, en la nube o en un entorno local. App Service incluye las funcionalidades web y móviles que anteriormente se ofrecían por separado como Azure Websites y Azure Mobile Services. También incluye nuevas funcionalidades para automatizar procesos empresariales y hospedar las API en la nube. Como único servicio integrado, App Service ofrece un amplio conjunto de funcionalidades para escenarios web, móviles y de integración.

## <a name="authenticate-through-azure-active-directory-ad"></a>Autenticación mediante Azure Active Directory (AD)
App Service proporciona un servicio OAuth 2.0 para el proveedor de identidades. OAuth 2.0 se centra en la sencillez del desarrollador del cliente y ofrece flujos de autorización específicos de aplicaciones web, aplicaciones de escritorio y teléfonos móviles. Azure AD usa OAuth 2.0 para permitir la autorización del acceso a los dispositivos móviles y a las aplicaciones web. Para obtener más información, vea [Autenticación y autorización en Azure App Service](../../app-service/overview-authentication-authorization.md).

## <a name="restrict-access-based-on-role"></a>Restricción de acceso según el rol
La restricción del acceso es fundamental para las organizaciones que deseen aplicar directivas de seguridad para el acceso a los datos. Puede usar el control de acceso basado en rol (RBAC) para asignar permisos a usuarios, grupos y aplicaciones en un ámbito determinado, como el principio de necesidad de conocer y el principio de seguridad con privilegios mínimos. Si quiere saber más sobre cómo conceder a los usuarios acceso a las aplicaciones, consulte [¿Qué es el control de acceso basado en rol de Azure (Azure RBAC)?](/azure/role-based-access-control/overview)

## <a name="protect-your-keys"></a>Protección de las claves
No importa la calidad de la seguridad si pierde las claves de suscripción. Azure Key Vault ayuda a proteger claves criptográficas y secretos usados por servicios y aplicaciones en la nube. Con Key Vault, puede cifrar claves y secretos (por ejemplo claves de autenticación, claves de cuenta de almacenamiento, claves de cifrado de datos, archivos .PFX y contraseñas) a través del uso de claves que están protegidas por módulos de seguridad de hardware (HSM). Para tener mayor seguridad, puede importar o generar las claves en HSM. También puede utilizar Key Vault para administrar los certificados TLS con renovación automática. Vea [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md) para más información.

## <a name="restrict-incoming-source-ip-addresses"></a>Restricción de las direcciones IP de origen entrante
[App Service Environments](../../app-service/environment/intro.md) tiene una característica de integración de la red virtual con la que es más fácil restringir las direcciones IP de origen entrantes mediante grupos de seguridad de red (NSG). Si no conoce Azure Virtual Network, se trata de una funcionalidad que permite colocar muchos de los recursos de Azure en una red no enrutable sin conexión a Internet cuyo acceso controla. Consulte [Integración de su aplicación con una instancia de Azure Virtual Network](../../app-service/web-sites-integrate-with-vnet.md) para más información.

En Windows, App Service también permite restringir las direcciones IP dinámicamente mediante el archivo web.config. Para más información, consulte [Seguridad de direcciones IP dinámicas](/iis/configuration/system.webServer/security/dynamicIpSecurity/).


## <a name="next-steps"></a>Pasos siguientes
En este artículo se presenta una colección de procedimientos recomendados de seguridad de App Service para proteger las aplicaciones web y móviles PaaS. Para obtener más información sobre cómo proteger las implementaciones de PaaS, vea:

- [Protección de implementaciones de PaaS](paas-deployments.md)
- [Protección de bases de datos PaaS en Azure](paas-applications-using-sql.md)
