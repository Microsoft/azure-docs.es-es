---
title: Configuración del aprovisionamiento de usuarios en una aplicación de la galería de Azure AD | Microsoft Docs
description: Cómo configurar rápidamente el aprovisionamiento y desaprovisionamiento completos de cuentas de usuario para aplicaciones que ya aparecen en la galería de aplicaciones de Azure AD
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: be43f0e100bc96de2be916cbf52bca7d3ba51431
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784531"
---
# <a name="how-to-configure-user-provisioning-to-an-azure-ad-gallery-application"></a>Configuración del aprovisionamiento de usuarios en una aplicación de la galería de Azure AD

El *aprovisionamiento de cuentas de usuario* es el acto de crear, actualizar o deshabilitar registros de cuenta de usuario en el almacén de perfiles de usuario local de una aplicación. La mayoría de las aplicaciones SaaS y en la nube almacenan el rol y los permisos de los usuarios en su propio almacén local de perfiles de usuario, y la presencia de tal registro de usuario en su almacén local es *necesaria* para que funcionen el inicio de sesión único y el acceso.

En Azure Portal, en la pestaña **Aprovisionamiento** del panel de navegación izquierdo para una aplicación empresarial, se muestran los modos de aprovisionamiento admitidos para esa aplicación. Puede ser uno de dos valores:

## <a name="configuring-an-application-for-manual-provisioning"></a>Configuración de una aplicación para el aprovisionamiento manual

El aprovisionamiento *manual* quiere decir que las cuentas de usuario se deben crear manualmente mediante los métodos proporcionados por esa aplicación. Esto podría suponer iniciar sesión en un portal administrativo para esa aplicación y agregar usuarios mediante una interfaz de usuario basada en Web. O bien, podría tener que cargar una hoja de cálculo con los detalles de las cuentas de usuario, mediante un mecanismo proporcionado por esa aplicación. Consulte la documentación suministrada por la aplicación o póngase en contacto con su desarrollador para determinar qué mecanismos hay disponibles.

Si Manual es el único modo que se muestra para una aplicación determinada, significa que aún no se ha creado ningún conector de aprovisionamiento de Azure AD automático para la aplicación. O bien, significa que la aplicación no es compatible con la API de administración de usuarios obligatoria sobre la que se crea un conector de aprovisionamiento automatizado.

Si desea solicitar soporte técnico para el aprovisionamiento automático con una aplicación determinada, puede rellenar una solicitud en [Azure Active Directory Application Requests](https://aka.ms/aadapprequest) (Solicitudes de aplicación de Azure Active Directory).

## <a name="configuring-an-application-for-automatic-provisioning"></a>Configuración de una aplicación para el aprovisionamiento automático

*Automático* significa que se ha desarrollado un conector de aprovisionamiento de Azure AD para esta aplicación. Para más información sobre el servicio de aprovisionamiento de Azure AD y cómo funciona, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-app-provisioning).

Para más información sobre cómo aprovisionar usuarios y grupos específicos en una aplicación, consulte [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-apps-manage-provisioning).

Los pasos que son necesarios para habilitar y configurar el aprovisionamiento automático varían según la aplicación.

>[!NOTE]
>Para empezar, debería buscar el tutorial de configuración específico para configurar el aprovisionamiento para su aplicación y seguir esos pasos para configurar la aplicación y Azure AD para crear la conexión de aprovisionamiento. 
>
>

Puede encontrar tutoriales sobre aplicaciones en [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list).

Una cuestión importante que tener en cuenta al configurar el aprovisionamiento es revisar y configurar las asignaciones de atributos y los flujos de trabajo que definen qué propiedades de usuario (o de grupo) fluyen de Azure AD a la aplicación. Esto incluye la configuración de la "propiedad de coincidencia" que se usa para identificar de forma exclusiva y emparejar a usuarios y grupos entre ambos sistemas. Para más información acerca de este proceso importante.

## <a name="next-steps"></a>Pasos siguientes
[Personalización de asignaciones de atributos de aprovisionamiento de usuarios para aplicaciones SaaS en Azure Active Directory de usuarios](https://docs.microsoft.com/azure/active-directory/active-directory-saas-customizing-attribute-mappings)

