---
title: Problema al guardar las credenciales de administrador durante la configuración del aprovisionamiento de usuarios para una aplicación de la galería de Azure AD | Microsoft Docs
description: Cómo solucionar los problemas habituales que surgen al configurar el aprovisionamiento de usuarios para una aplicación que ya aparece en la galería de aplicaciones de Azure AD
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
ms.date: 02/21/2018
ms.author: mimart
ms.reviewer: asmalser
ms.collection: M365-identity-device-management
ms.openlocfilehash: c2175a23b922a21fe5b6e7ca3ed316e1e22644d2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784518"
---
# <a name="problem-saving-administrator-credentials-while-configuring-user-provisioning-to-an-azure-active-directory-gallery-application"></a>Problema al guardar las credenciales de administrador durante la configuración del aprovisionamiento de usuarios para una aplicación de la galería de Azure Active Directory 

Al usar Azure Portal para configurar el [aprovisionamiento de usuarios automático](user-provisioning.md) para una aplicación empresarial, puede hallarse en una situación en la que:

* Las **credenciales de administrador** especificadas para la aplicación son válidas y el botón **Probar conexión** funciona. Sin embargo, las credenciales no se pueden guardar y Azure Portal devuelve un mensaje de error genérico.

Si el inicio de sesión único basado en SAML también está configurado para la misma aplicación, la causa más probable del error será que se ha superado el límite de almacenamiento por aplicación interno de Azure AD en el caso de los certificados y las credenciales.

En la actualidad, Azure AD cuenta con una capacidad de almacenamiento máximo de un 1024 bytes para todos los certificados, tokens secretos, credenciales y datos de configuración relacionados asociados a una sola instancia de una aplicación (también conocida como registro de entidades de servicio en Azure AD).

Al configurarse el inicio de sesión único basado en SAML, el certificado usado para firmar los tokens SAML se almacena aquí. Además, suele consumir más del 50 % del espacio.

Los tokens secretos, URI, direcciones de correo electrónico de notificación, nombres de usuario y contraseñas que se escriben durante la configuración del aprovisionamiento de usuarios pueden hacer que se supere el límite de almacenamiento.

## <a name="how-to-work-around-this-issue"></a>Cómo evitar este problema 

Hay dos formas posibles de evitar este problema hoy:

1. **Usar dos instancias de aplicación de galería, una para el inicio de sesión único y la otra para el aprovisionamiento de usuarios**: tomando la aplicación de galería [LinkedIn Elevate](../saas-apps/linkedinelevate-tutorial.md) como ejemplo, puede agregar LinkedIn Elevate desde la galería y configurarla para el inicio de sesión único. Para el aprovisionamiento, agregue otra instancia de LinkedIn Elevate de la galería de aplicaciones y asígnele el nombre "LinkedIn Elevate (aprovisionamiento)". Para esta segunda instancia, configure el [aprovisionamiento](../saas-apps/linkedinelevate-provisioning-tutorial.md), pero no el inicio de sesión único. Al usar esta solución alternativa, los mismos usuarios y grupos deben [asignarse](assign-user-or-group-access-portal.md) a ambas aplicaciones. 

2. **Reducir la cantidad de datos de configuración almacenados**: todos los datos escritos en la sección [Credenciales de administrador](user-provisioning.md#how-do-i-set-up-automatic-provisioning-to-an-application) de la pestaña de aprovisionamiento se almacenan en el mismo lugar que el certificado SAML. Mientras que puede que no sea posible reducir la longitud de todos estos datos, algunos campos de configuración opcionales como el **Correo electrónico de notificación** se pueden quitar.

## <a name="next-steps"></a>Pasos siguientes
[Configuración del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS](user-provisioning.md)
