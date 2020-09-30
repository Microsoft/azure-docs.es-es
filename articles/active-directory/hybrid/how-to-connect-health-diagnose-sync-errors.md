---
title: 'Azure AD Connect Health: diagnóstico de errores de sincronización de atributos duplicados | Microsoft Docs'
description: En este documento se describe el proceso de diagnóstico de los errores de sincronización de atributos duplicados y una posible solución de los objetos huérfanos directamente desde Azure Portal.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: maheshu
editor: billmath
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/11/2018
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: c2bd2e72b05cc01b1a351880d565323662635364
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89278690"
---
# <a name="diagnose-and-remediate-duplicated-attribute-sync-errors"></a>Diagnóstico y solución de errores de sincronización de atributos duplicados

## <a name="overview"></a>Información general
Azure Active Directory (Azure AD) Connect Health da un paso más en el resalte de errores de sincronización y presenta la corrección de autoservicio. Soluciona los errores de sincronización de atributos duplicados y repara los objetos huérfanos desde Azure AD.
La característica de diagnóstico ofrece estas ventajas:
- Proporciona un procedimiento de diagnóstico que reduce los errores de sincronización de atributos duplicados. Y aporta soluciones específicas.
- Aplica una corrección para los escenarios dedicados desde Azure AD para resolver el error en un solo paso.
- No es necesaria ninguna actualización o configuración para habilitar la característica.
Para más información acerca de Azure AD, consulte [Sincronización de identidades y resistencia de atributos duplicados](how-to-connect-syncservice-duplicate-attribute-resiliency.md).

## <a name="problems"></a>Problemas
### <a name="a-common-scenario"></a>Escenario común
Cuando se producen los errores de sincronización **QuarantinedAttributeValueMustBeUnique** y **AttributeValueMustBeUnique**, es habitual ver un conflicto de **UserPrincipalName** o **Proxy Addresses** en Azure AD. Para resolver los errores de sincronización, puede actualizar el objeto de origen en conflicto desde el lado local. El error de sincronización se resolverá después de la siguiente sincronización. Por ejemplo, esta imagen indica que dos usuarios tienen un conflicto en **UserPrincipalName**. Ambos son **Joe.J\@contoso.com**. Los objetos en conflicto se ponen en cuarentena en Azure AD.

![Diagnosticar un escenario común de diagnóstico de errores de sincronización](./media/how-to-connect-health-diagnose-sync-errors/IIdFixCommonCase.png)

### <a name="orphaned-object-scenario"></a>Escenario del objeto huérfano
A veces puede darse el caso de que un usuario existente pierde el **delimitador de origen**. La eliminación del objeto de origen se ha producido en la instancia local de Active Directory. Pero el cambio de señal de eliminación nunca se sincronizó con Azure AD. Esta pérdida se produce por motivos tales como problemas del motor de sincronización o la migración del dominio. Cuando se restaura o recrea el mismo objeto, lo lógico es que un usuario existente sea el que se sincronice desde el **delimitador de origen**. 

Cuando un usuario existente es un objeto solo en la nube, también puede ver que el usuario en conflicto está sincronizado con Azure AD. En la sincronización, el usuario no se puede hacer coincidir con el objeto existente. No hay ninguna forma directa de reasignar el **delimitador de origen**. Más información acerca de la [base de conocimientos existente](https://support.microsoft.com/help/2647098). 

Por ejemplo, el objeto existente en Azure AD conserva la licencia de Joe. Un objeto recién sincronizado con otro **delimitador de origen** se produce en un estado de atributo duplicado en Azure AD. Los cambios de Joe en una instancia local de Active Directory no se aplicarán al usuario original de Joe (objeto existente) en Azure AD.  

![Escenario de diagnóstico de errores de sincronización de objeto huérfano](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)

## <a name="diagnostic-and-troubleshooting-steps-in-connect-health"></a>Pasos de diagnóstico y de solución de problemas en Connect Health 
La característica de diagnóstico admite objetos de usuario con los siguientes atributos duplicados:

| Nombre del atributo | Tipos de error de sincronización|
| ------------------ | -----------------|
| UserPrincipalName | QuarantinedAttributeValueMustBeUnique o AttributeValueMustBeUnique | 
| ProxyAddresses | QuarantinedAttributeValueMustBeUnique o AttributeValueMustBeUnique | 
| SipProxyAddress | AttributeValueMustBeUnique | 
| OnPremiseSecurityIdentifier |  AttributeValueMustBeUnique |

>[!IMPORTANT]
> Para acceder a esta característica, se requieren los permisos de **administrador global** o de **colaborador** en Azure RBAC.
>

Siga los pasos de Azure Portal para limitar los datos de los errores de sincronización y proporcionar soluciones más específicas:

![Pasos de diagnóstico de errores de sincronización](./media/how-to-connect-health-diagnose-sync-errors/IIdFixSteps.png)

En Azure Portal, dé algunos pasos para identificar los escenarios específicos que pueden corregirse:  
1.  Consulte la columna **Estado de diagnóstico**. El estado muestra si hay una forma posible de corregir un error de sincronización directamente desde Azure Active Directory. En otras palabras, existe un flujo de solución de problemas que puede reducir el error y corregirlo potencialmente.

| Status | ¿Qué significa? |
| ------------------ | -----------------|
| No iniciado | No ha visitado este proceso de diagnóstico. En función del resultado del diagnóstico, hay una posible forma de corregir el error de sincronización directamente desde el portal. |
| Corrección manual necesaria | El error no se ajusta a los criterios de las soluciones disponibles en el portal. Los tipos de objeto en conflicto no son usuarios, o bien ya ha realizado los pasos de diagnóstico y no hay una solución disponible en el portal. En el último caso, una de las posibles soluciones es una revisión desde el lado local. [Más información sobre las correcciones locales](https://support.microsoft.com/help/2647098). | 
| Sincronización pendiente | Se aplicó la solución. El portal espera al siguiente ciclo de sincronización para borrar el error. |

  >[!IMPORTANT]
  > La columna de estado de diagnóstico se restablecerá después de cada ciclo de sincronización. 
  >

1. Seleccione el botón **Diagnosticar** en los detalles del error. Responderá a varias preguntas e identificará los detalles del error de sincronización. Dichas respuestas ayudan a identificar un caso de objeto huérfano.

1. Si aparece el botón **Cerrar** al final del diagnóstico, se debe a que no hay ninguna solución rápida disponible en el portal en función de sus respuestas. Consulte la solución que se muestra en el último paso. Las correcciones del entorno local siguen siendo las soluciones. Seleccione el botón **Cerrar**. El estado del error de sincronización actual cambia a  **Corrección manual necesaria**. El estado permanece durante el ciclo de sincronización actual.

1. Después de que se identifique el caso del objeto huérfano, puede solucionar los errores de sincronización de atributos duplicados directamente desde el portal. Para desencadenar el proceso, seleccione el botón **Aplicar corrección**. El estado del error de sincronización actual se actualiza a **Sincronización pendiente**.

1. Después del siguiente ciclo de sincronización, el error debería quitarse de la lista.

## <a name="how-to-answer-the-diagnosis-questions"></a>Cómo responder a las preguntas de diagnóstico 
### <a name="does-the-user-exist-in-your-on-premises-active-directory"></a>¿Existe el usuario en la instancia local de Active Directory?

La pregunta intenta identificar el objeto de origen del usuario existente en la instancia local de Active Directory.  
1. Compruebe si Azure Active Directory tiene un objeto con el valor **UserPrincipalName** especificado. Si no lo tiene, la respuesta es **No**.
2. Si lo tiene, compruebe si el objeto sigue en el ámbito de sincronización.  
   - Busque en el espacio conector de Azure AD mediante el DN.
   - Si el objeto se encuentra en estado **Pendiente de agregación**, responda **No**. Azure AD Connect no puede conectar el objeto al objeto de AD correcto.
   - Si no se encuentra el objeto, responda **Sí**.

En estos ejemplos, la pregunta intenta identificar si **Joe Jackson** sigue existiendo en la instancia local de Active Directory.
En el **escenario común**, ambos usuarios, **Joe Johnson** y **Joe Jackson**, están presentes en la instancia local de Active Directory. Los objetos en cuarentena son dos usuarios distintos.

![Diagnosticar un escenario común de diagnóstico de errores de sincronización](./media/how-to-connect-health-diagnose-sync-errors/IIdFixCommonCase.png)

Para el **escenario de objeto huérfano**, el usuario individual **Joe Johnson** es el único que está presente en la instancia local de Active Directory:

![Escenario de diagnóstico de errores de sincronización del objeto huérfano *no existe*](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)

### <a name="do-both-of-these-accounts-belong-to-the-same-user"></a>¿Pertenecen ambas cuentas al mismo usuario?
Esta pregunta comprueba el usuario entrante en conflicto y el objeto de usuario existente en Azure AD para ver si pertenecen al mismo usuario.  
1. El objeto en conflicto acaba de sincronizarse con Azure Active Directory. Comparar los atributos de los objetos:  
   - Display Name (Nombre para mostrar)
   - Nombre principal del usuario
   - Id. de objeto
2. Si Azure AD no puede compararlos, compruebe si Active Directory contiene objetos con los valores de **UserPrincipalNames** proporcionados. Respuesta **No** si encuentra ambos.

En el ejemplo siguiente, los dos objetos pertenecen al mismo usuario, **Joe Johnson**.

![Escenario de diagnóstico de errores de sincronización del objeto huérfano *mismo usuario*](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)


## <a name="what-happens-after-the-fix-is-applied-in-the-orphaned-object-scenario"></a>Qué ocurre después de aplicar la corrección en el escenario de objeto huérfano
En función de las respuestas a las preguntas anteriores, podrá ver el botón **Aplicar corrección** cuando haya una corrección disponible en Azure AD. En este caso, el objeto local se sincroniza con un objeto de Azure AD inesperado. Ambos objetos se asignan mediante el **delimitador de origen**. El cambio de **Aplicar corrección** realiza estos cambios, u otros similares:
1. Actualiza el **delimitador de origen** al objeto correcto en Azure AD.
2. Elimina el objeto en conflicto de Azure AD, en caso de que esté presente.

![Diagnostico de errores de sincronización después de la corrección](./media/how-to-connect-health-diagnose-sync-errors/IIdFixAfterFix.png)

>[!IMPORTANT]
> El cambio de **Aplicar corrección** solo se aplica a los casos de objetos huérfanos.
>

Después de los pasos anteriores, el usuario puede acceder al recurso original, que es un vínculo a un objeto existente. El valor **Diagnose status** (Estado de diagnóstico) de la vista de lista se actualizará a **Sincronización pendiente**. El error de sincronización se resolverá después de la siguiente sincronización. Connect Health dejará de mostrar el error de sincronización resuelto en la vista de lista.

## <a name="failures-and-error-messages"></a>Errores y mensajes de error
**El usuario con el atributo en conflicto es eliminado temporalmente en Azure Active Directory. Asegúrese de que el usuario se elimina de forma definitiva antes de volver a intentarlo.**  
El usuario con el atributo en conflicto en Azure AD debe limpiarse para poder aplicar la corrección. Revise el [procedimiento para eliminar de forma permanente el usuario en Azure AD](../fundamentals/active-directory-users-restore.md) antes de volver a intentar la corrección. El usuario también se eliminará automáticamente de forma permanente después de 30 días en el estado de eliminación temporal. 

**No se admite la actualización del delimitador de origen al usuario basado en la nube en el inquilino.**  
El usuario basado en la nube de Azure AD no debe tener el delimitador de origen. En este caso no se admite la actualización del delimitador de origen. Se requiere una corrección manual en el entorno local. 

## <a name="faq"></a>Preguntas más frecuentes
**P.** ¿Qué ocurre si se produce un error al ejecutarse **Aplicar corrección**?  
**R.** Si se produce un error en la ejecución, es posible que Azure AD Connect muestre un error de exportación. Actualice la página del portal y vuelva a intentar la operación después de la siguiente sincronización. El ciclo de sincronización predeterminado dura 30 minutos. 


**P.** ¿Qué ocurre si el **objeto existente** debe ser el objeto que se debe eliminar?  
**R.** Si el **objeto existente** debe eliminarse, el proceso no supone un cambio en el **delimitador de origen**. Por lo general, se puede corregir desde la instancia local de Active Directory. 


**P.** ¿Qué permiso necesita un usuario para aplicar la revisión?  
**R.** Un **administrador global** o un **colaborador** de Azure RBAC tienen permiso para acceder al proceso de diagnóstico y solución de problemas.


**P.** ¿Tengo que configurar Azure AD Connect o actualizar el agente de Azure AD Connect Health para esta característica?  
**R.** No, el proceso de diagnóstico es una característica completa basada en la nube.


**P.** Si el objeto existente se elimina temporalmente, ¿volverá a activarlo el proceso de diagnóstico?  
**R.** No, la corrección no actualizará los atributos del objeto, a excepción del **delimitador de origen**.