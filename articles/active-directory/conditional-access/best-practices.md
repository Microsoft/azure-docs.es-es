---
title: Procedimientos recomendados para el acceso condicional en Azure Active Directory | Microsoft Docs
description: Conozca lo que debe saber y lo que debe evitar al configurar directivas de acceso condicional.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 01/25/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: b5536c3c427e5b6225d81d649722d8af48c23091
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88948460"
---
# <a name="best-practices-for-conditional-access-in-azure-active-directory"></a>Procedimientos recomendados para el acceso condicional en Azure Active Directory

Con el [acceso condicional de Azure Active Directory (Azure AD)](./overview.md), puede controlar el modo en que los usuarios autorizados acceden a las aplicaciones en la nube. En este artículo se proporciona información acerca de lo siguiente:

- Qué debería saber 
- Qué debe evitar al configurar directivas de acceso condicional 

En este artículo se asume que está familiarizado con los conceptos y la terminología que se describen en [¿Qué es el acceso condicional en Azure Active Directory?](./overview.md)

## <a name="whats-required-to-make-a-policy-work"></a>¿Cuáles son los requisitos para que una directiva funcione?

Cuando crea una directiva nueva, no hay usuarios, grupos, aplicaciones ni controles de acceso seleccionados.

![Aplicaciones de nube](./media/best-practices/02.png)

Para realizar un trabajo de directiva, debe configurar:

| Qué           | Cómo                                  | Por qué |
| :--            | :--                                  | :-- |
| **Aplicaciones en la nube** |Seleccione una o varias aplicaciones.  | El objetivo de una directiva de acceso condicional es permitirle controlar cómo los usuarios autorizados pueden acceder a las aplicaciones en la nube.|
| **Usuarios y grupos** | Seleccione al menos un usuario o grupo que esté autorizado para acceder a las aplicaciones en la nube que haya seleccionado. | Una directiva de acceso condicional que no tiene usuarios ni grupos asignados nunca se desencadena. |
| **Controles de acceso** | Seleccione al menos un control de acceso. | Si se cumplen las condiciones, el procesador de directivas debe saber qué hacer. |

## <a name="what-you-should-know"></a>Qué debería saber

### <a name="how-are-conditional-access-policies-applied"></a>¿Cómo se aplican las directivas de acceso condicional?

Al acceder a una aplicación en la nube se puede aplicar más de una directiva de acceso condicional. En este caso, se tienen que satisfacer todas las directivas que se aplican. Por ejemplo, si una directiva exige la autenticación multifactor (MFA) y otra requiere un dispositivo compatible, tendrá que completar la MFA y usar un dispositivo compatible. 

Todas las directivas se aplican en dos fases:

- Fase 1: Recopilación de detalles de la sesión 
   - Recopile los detalles de la sesión, como la ubicación del usuario y la identidad del dispositivo, que serán necesarios para la evaluación de la directiva. 
   - Durante esta fase, es posible que se solicite un certificado a los usuarios si el cumplimiento del dispositivo forma parte de las directivas de acceso condicional. Esta solicitud puede producirse en aplicaciones de explorador cuando el sistema operativo del dispositivo no es Windows 10. 
   - La fase 1 de la evaluación de la directiva se produce para todas las directivas habilitadas, así como para las directivas en [modo de solo informe](concept-conditional-access-report-only.md).
- Fase 2: Cumplimiento 
   - Use los detalles de la sesión recopilados en la fase 1 para identificar los requisitos que no se han cumplido. 
   - Si hay una directiva que está configurada para bloquear el acceso, con el control de concesión de bloqueo, la aplicación se detendrá aquí y se bloqueará al usuario. 
   - Luego, se le pedirá al usuario que complete los requisitos de control de concesión adicionales que no se han satisfecho durante la fase 1 en el orden siguiente, hasta que se satisfaga la directiva:  
      - Multi-Factor Authentication 
      - Directiva de protección de aplicaciones/aplicaciones cliente aprobada 
      - Dispositivo administrado (combinación de Azure AD compatible o híbrido) 
      - Términos de uso 
      - Controles personalizados  
      - Una vez que se han satisfecho los controles de concesión, aplique controles de sesión (aplicación forzada, Microsoft Cloud App Security y duración del token). 
   - La fase 2 de la evaluación de directivas se realiza para todas las directivas habilitadas. 

### <a name="how-are-assignments-evaluated"></a>¿Cómo se evalúan las asignaciones?

A todas las asignaciones se les asigna **la operación lógica AND**. Si tiene más de una asignación configurada, se deben satisfacer todas las asignaciones para desencadenar una directiva.  

Si debe configurar una condición de ubicación que se aplique a todas las conexiones realizadas desde fuera de la red de la organización, puede:

- Incluir **todas las ubicaciones**.
- Excluir **todas las IP de confianza**.

### <a name="what-to-do-if-you-are-locked-out-of-the-azure-ad-admin-portal"></a>¿Qué se debe hacer si está bloqueado en el portal de administración de Azure AD?

Si está bloqueado en el portal de Azure AD debido a una configuración incorrecta de una directiva de acceso condicional:

- Compruebe si hay otros administradores en su organización que aún no estén bloqueados. Un administrador con acceso a Azure Portal puede deshabilitar la directiva que está afectando al inicio de sesión. 
- Si ninguno de los administradores de la organización puede actualizar la directiva, debe enviar una solicitud de soporte técnico. El servicio de soporte técnico de Microsoft puede revisar y actualizar las directivas de acceso condicional que impidan el acceso.

### <a name="what-happens-if-you-have-policies-in-the-azure-classic-portal-and-azure-portal-configured"></a>¿Qué ocurre si tiene directivas configuradas en el Portal de Azure clásico y en Azure Portal?  

Las dos directivas se aplican mediante Azure Active Directory y el usuario obtiene acceso únicamente cuando se cumplen todos los requisitos.

### <a name="what-happens-if-you-have-policies-in-the-intune-silverlight-portal-and-the-azure-portal"></a>¿Qué sucede si tiene directivas en el portal de Intune Silverlight y en Azure Portal?

Las dos directivas se aplican mediante Azure Active Directory y el usuario obtiene acceso únicamente cuando se cumplen todos los requisitos.

### <a name="what-happens-if-i-have-multiple-policies-for-the-same-user-configured"></a>¿Qué ocurre si tiene varias directivas configuradas para el mismo usuario?  

En cada inicio de sesión, Azure Active Directory evalúa todas las directivas y garantiza que se cumplan todos los requisitos antes de conceder acceso al usuario. El bloqueo del acceso altera todas las demás opciones de configuración. 

### <a name="does-conditional-access-work-with-exchange-activesync"></a>¿Funciona el acceso condicional con Exchange ActiveSync?

Sí, se puede usar Exchange ActiveSync en una directiva de acceso condicional.

Algunas aplicaciones en la nube, como SharePoint Online y Exchange Online, también admiten protocolos de autenticación heredados. Cuando una aplicación cliente puede utilizar un protocolo de autenticación heredado para tener acceso a una aplicación en la nube, Azure AD no puede aplicar una directiva de acceso condicional en este intento de acceso. Para evitar que una aplicación cliente omita el cumplimiento de directivas, debe comprobar si es posible habilitar solo la autenticación moderna en las aplicaciones en la nube afectadas.

### <a name="how-should-you-configure-conditional-access-with-office-365-apps"></a>¿Cómo se debe configurar el acceso condicional con aplicaciones de Office 365?

Dado que las aplicaciones de Office 365 están conectadas entre sí, se recomienda asignar juntas las aplicaciones usadas frecuentemente al crear directivas.

Las aplicaciones normales interconectadas incluyen Microsoft Flow, Microsoft Planner, Microsoft Teams, Office 365 Exchange Online, Office 365 SharePoint Online y Office 365 Yammer.

Para las directivas que requieren interacciones del usuario, como la autenticación multifactor, es importante conocer cuándo se controla el acceso al principio de una sesión o tarea. En caso contrario, los usuarios no podrán completar algunas tareas dentro de una aplicación. Por ejemplo, si necesita autenticación multifactor en dispositivos no administrados para tener acceso a SharePoint, pero no al correo electrónico, los usuarios que trabajen con el correo electrónico no podrán adjuntar archivos de SharePoint a un mensaje. Se puede encontrar más información en el artículo [¿Cuáles son las dependencias del servicio de acceso condicional de Azure Active Directory?](service-dependencies.md).

## <a name="what-you-should-avoid-doing"></a>¿Qué no debería hacer?

El marco de trabajo de acceso condicional le proporciona una gran flexibilidad de configuración. Sin embargo, una gran flexibilidad también implica revisar cuidadosamente cada directiva de configuración antes de liberarla para evitar resultados no deseados. En este contexto, debe prestar especial atención a las asignaciones que afectan a conjuntos completos, como **todos los usuarios / grupos / aplicaciones en la nube**.

En su entorno, debería evitar las siguientes configuraciones:

**Para todos los usuarios, todas las aplicaciones en la nube:**

- **Bloquear acceso**: esta configuración bloquea toda la organización, lo cual no es en absoluto una buena idea.
- **Requerir dispositivo compatible**: para usuarios que aún no han inscrito sus dispositivos, esta directiva bloquea todo el acceso, incluido el acceso al portal Intune. Si es un administrador y no tiene un dispositivo inscrito, esta directiva le impide volver a acceder Azure Portal para cambiar la directiva.
- **Requerir unión a un dominio** : este acceso al bloqueo de directivas también ofrece la posibilidad de bloquear el acceso a todos los usuarios de su organización si aún no tiene un dispositivo unido al dominio.
- **Requerir la directiva de protección de aplicaciones**: este acceso al bloqueo de directivas también ofrece la posibilidad de bloquear el acceso a todos los usuarios de su organización si no tiene una directiva de Intune. Si es un administrador sin una aplicación cliente que tenga una directiva de protección de aplicaciones de Intune, esta directiva le impedirá acceder a portales como el de Intune o el de Azure.

**Para todos los usuarios, todas las aplicaciones en la nube, todas las plataformas de dispositivos:**

- **Bloquear acceso**: esta configuración bloquea toda la organización, lo cual no es en absoluto una buena idea.

## <a name="how-should-you-deploy-a-new-policy"></a>¿Cómo debe implementar una directiva nueva?

Como primer paso, debe evaluar la directiva con la [herramienta What if](what-if-tool.md).

Cuando las nuevas directivas estén listas para su entorno, impleméntelas en fases:

1. Aplique una directiva a un pequeño conjunto de usuarios y compruebe que se comporta según lo esperado. 
1. Al expandir una directiva para incluir a más usuarios. Continúe excluyendo a todos los administradores de la directiva para asegurarse de que todavía tienen acceso y pueden actualizar una directiva si es necesario un cambio.
1. Aplique una directiva a todos los usuarios solo si es necesario. 

Como procedimiento recomendado, cree una cuenta de usuario que esté:

- Dedicada a la administración de directivas 
- Excluida de todas las directivas

## <a name="policy-migration"></a>Migración de directivas

Considere la posibilidad de migrar las directivas que no haya creado en Azure Portal porque:

- Ahora puede resolver situaciones que antes no podía controlar.
- Puede reducir el número de directivas que tiene que administrar mediante su consolidación.   
- Puede administrar todas las directivas de Acceso condicional en una ubicación central.
- Se ha retirado la versión clásica de Azure Portal.   

Para obtener más información, consulte [Migración de directivas clásicas en Azure Portal](policy-migration.md).

## <a name="next-steps"></a>Pasos siguientes

Si quiere saber más sobre:

- Cómo configurar una directiva de acceso condicional, consulte [Exigir MFA para aplicaciones específicas con acceso condicional de Azure Active Directory](../authentication/tutorial-enable-azure-mfa.md).
- Cómo planear las directivas de acceso condicional, consulte [Instrucciones: Planee la implementación del acceso condicional a Azure Active Directory](plan-conditional-access.md).