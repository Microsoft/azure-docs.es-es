---
title: 'Requerir MFA desde redes que no son de confianza: Azure Active Directory'
description: Aprenda a configurar una directiva de acceso condicional en Azure Active Directory (Azure AD) para los intentos de acceso desde redes que no son de confianza.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 11/21/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6b2f9a6e13fdc39ab18a2056fab15982aa9fb0e4
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88948171"
---
# <a name="how-to-require-mfa-for-access-from-untrusted-networks-with-conditional-access"></a>Procedimientos: Uso obligatorio de MFA para el acceso desde redes que no son de confianza con el acceso condicional   

Azure Active Directory (Azure AD) habilita el inicio de sesión único en dispositivos, aplicaciones y servicios desde cualquier ubicación. Los usuarios pueden acceder a las aplicaciones en la nube no solo desde la red de la organización, sino también desde cualquier ubicación de Internet que no sea de confianza. Un procedimiento recomendado para el acceso desde redes que no son de confianza es requerir la autenticación multifactor (MFA).

En este artículo se proporciona la información necesaria para configurar una directiva de acceso condicional que exija MFA para el acceso desde redes que no son de confianza. 

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se asume que está familiarizado con: 

- Los [conceptos básicos](overview.md) del acceso condicional de Azure AD 
- Los [procedimientos recomendados](best-practices.md) para configurar directivas de acceso condicional en Azure Portal

## <a name="scenario-description"></a>Descripción del escenario

Para dominar el equilibrio entre seguridad y productividad, podría ser suficiente con requerir una contraseña para iniciar sesión desde la red de la organización. Sin embargo, con el acceso desde una ubicación de red que no es de confianza, hay un mayor riesgo de que los inicios de sesión no procedan de usuarios legítimos. Para solucionar este problema, puede bloquear el acceso desde redes que no son de confianza. También puede requerir la autenticación multifactor (MFA) para obtener más garantías de que el intento ha sido realizado por el propietario legítimo de la cuenta. 

Con el acceso condicional de Azure AD, puede abordar este requisito con una única directiva que concede acceso: 

- A determinadas aplicaciones en la nube
- A los usuarios y grupos seleccionados  
- Con autenticación multifactor 
- Cuando el acceso se origina desde: 
   - Una ubicación que no es de confianza

## <a name="implementation"></a>Implementación

La dificultad de este escenario consiste en convertir el *acceso desde una ubicación de red que no es de confianza* en una condición de acceso condicional. En una directiva de acceso condicional, puede configurar la [condición de ubicación](location-condition.md) para abordar los escenarios relacionados con las ubicaciones de red. La condición de ubicación permite seleccionar ubicaciones con nombre, que son agrupaciones lógicas de intervalos de direcciones IP, países y regiones.  

Normalmente, su organización posee uno o más intervalos de direcciones, por ejemplo, 199.30.16.0 - 199.30.16.15.
Para configurar una ubicación con nombre, puede:

- Especificar este intervalo (199.30.16.0/28) 
- Asignar un nombre descriptivo como **Red corporativa** 

En lugar de intentar definir cuáles son todas las ubicaciones que no son de confianza, hacer lo siguiente:

- Incluir todas las ubicaciones 

   ![Acceso condicional](./media/untrusted-networks/02.png)

- Excluir todas las ubicaciones de confianza 

   ![Acceso condicional](./media/untrusted-networks/01.png)

## <a name="policy-deployment"></a>Implementación de directivas

Con el enfoque descrito en este artículo, ahora puede configurar una directiva de acceso condicional para las ubicaciones que no son de confianza. Para asegurarse de que la directiva funciona según lo esperado, el procedimiento recomendado es probarla antes de implementarla en producción. Lo ideal es usar un inquilino de prueba para comprobar si la nueva directiva funciona según lo previsto. Para más información, consulte [Implementación de una nueva directiva](best-practices.md#how-should-you-deploy-a-new-policy). 

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el acceso condicional, consulte [Acceso condicional de Azure Active Directory](./overview.md).