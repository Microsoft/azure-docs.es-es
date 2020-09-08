---
title: 'Solución de problemas de acceso condicional con la herramienta What If: Azure Active Directory'
description: Dónde encontrar las directivas de acceso condicional que se han aplicado y por qué se han aplicado
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: troubleshooting
ms.date: 08/07/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: eb51e8590e2859a9fe8692f79c5b1238ed5a2098
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88948188"
---
# <a name="troubleshooting-conditional-access-using-the-what-if-tool"></a>Solución de problemas de acceso condicional con la herramienta What If

La [herramienta What If](what-if-tool.md) de Acceso condicional es eficaz para intentar comprender por qué una directiva se ha aplicado o no a un usuario en una circunstancia específica, o bien si una directiva se aplicaría en un estado conocido.

La herramienta What If se encuentra en **Azure Portal** > **Azure Active Directory** > **Acceso condicional** > **What If**.

![Herramienta What If de Acceso condicional en el estado predeterminado](./media/troubleshoot-conditional-access-what-if/conditional-access-what-if-tool.png)

> [!NOTE]
> Actualmente, la herramienta What If no evalúa las directivas en modo de solo informe.

## <a name="gathering-information"></a>Recopilación de información

La herramienta What If solo requiere un **usuario** para comenzar a usarla. 

La información adicional siguiente es opcional, pero le ayudará a reducir el ámbito para casos concretos.

* Aplicaciones o acciones en la nube
* Dirección IP 
* País/región
* Plataforma de dispositivo
* Aplicaciones cliente (versión preliminar)
* Estado del dispositivo (versión preliminar) 
* Riesgo de inicio de sesión

Esta información se puede recopilar del usuario, su dispositivo o el registro de inicio de sesión de Azure AD.

## <a name="generating-results"></a>Generación de resultados

Escriba los criterios recopilados en la sección anterior y seleccione **What If** para generar una lista de resultados. 

En cualquier momento, puede seleccionar **Restablecer** para borrar todos los criterios de entrada y volver al estado predeterminado.

## <a name="evaluating-results"></a>Evaluación de resultados

### <a name="policies-that-will-apply"></a>Directivas que se aplicarán

En esta lista se mostrarán las directivas de acceso condicional que se aplicarán según las condiciones. En la lista se incluirán los controles de concesión y de sesión que se aplican. En algunos ejemplos se requiere la autenticación multifactor para acceder a una aplicación específica.

### <a name="policies-that-will-not-apply"></a>Directivas que no se aplicarán

En esta lista se mostrarán las directivas de acceso condicional que no se aplicarían si se aplicaran las condiciones. En la lista se incluirán las directivas y el motivo por el que no se aplican. Entre los ejemplos se incluyen usuarios y grupos que se pueden excluir de una directiva.

## <a name="use-case"></a>Caso de uso

Muchas organizaciones crean directivas basadas en ubicaciones de red, lo que permite ubicaciones de confianza y bloquea aquellas donde no se debería producir el acceso.

Para validar que una configuración se ha realizado de forma correcta, un administrador podría usar la herramienta What If para imitar el acceso, desde una ubicación que se debería permitir y desde otra que se debería denegar.

[ ![Herramienta What If en la que se muestran resultados con Bloquear acceso](./media/troubleshoot-conditional-access-what-if/conditional-access-what-if-results.png)](./media/troubleshoot-conditional-access-what-if/conditional-access-what-if-results.png#lightbox)

En este ejemplo, se bloquearía el acceso del usuario a cualquier aplicación en la nube en su viaje a Corea del Norte, ya que se ha bloqueado el acceso a Contoso desde esa ubicación.

Esta prueba se podría expandir para incorporar otros puntos de datos con el fin de restringir el ámbito.

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es el acceso condicional?](overview.md)
* [¿Qué es Azure Active Directory Identity Protection?](../identity-protection/overview-identity-protection.md)
* [¿Qué es una identidad de dispositivo?](../devices/overview.md)
* [Funcionamiento: Azure Multi-Factor Authentication](../authentication/concept-mfa-howitworks.md)