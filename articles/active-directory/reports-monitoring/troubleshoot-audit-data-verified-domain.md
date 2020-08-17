---
title: Solución de problemas de auditoría de datos de un cambio de dominio comprobado | Microsoft Docs
description: Se proporciona información que aparecerá en los registros de actividad de Azure Active Directory cuando cambie el dominio comprobado de un usuario.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 07/22/2020
ms.author: markvi
ms.collection: M365-identity-device-management
ms.openlocfilehash: f3c9ec3b1e96e47dbf46c6acb2c81147b614d069
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87117419"
---
# <a name="troubleshoot-audit-data-on-verified-domain-change"></a>Solución de problemas: Auditoría de datos en un cambio de dominio comprobado 


## <a name="i-have-a-lot-of-changes-to-my-users-and-i-am-not-sure-what-the-cause-of-it-is"></a>Tengo muchos cambios en los usuarios y no estoy seguro de cuál es el motivo.

### <a name="symptoms"></a>Síntomas

He comprobado los registros de auditoría de Azure AD y he notado que se han producido varias actualizaciones de usuario en mi inquilino de Azure AD. Estos eventos **Actualizar usuario** no muestran información del **Actor**, lo que produce incertidumbre sobre quién o qué desencadenó los cambios masivos en los usuarios. 

### <a name="cause"></a>Causa

 Un motivo común tras los cambios masivos en los objetos es una operación de back-end no sincrónica denominada **ProxyCalc**.  **ProxyCalc** es la lógica que determina el **UserPrincipalName** adecuado y las **direcciones de proxy**, que se actualizan en los usuarios, grupos o contactos de Azure AD. El diseño detrás de **ProxyCalc** busca asegurarse de que todos los valores **UserPrincipalName** y las **direcciones de proxy** sean coherentes en Azure AD todo el tiempo. **ProxyCalc** debe desencadenarse debido a un cambio explícito, como un cambio de dominio comprobado, y no se ejecuta continuamente en segundo plano como una tarea. 

  

#### <a name="what-does-userprincipalname-consistency-mean"></a>¿Qué significa la coherencia de UserPrincipalName? 

En el caso de los usuarios que solo están en la nube, la coherencia implica que **UserPrincipalName** está establecido en un sufijo de dominio comprobado. Cuando se procesa un **UserPrincipalName** incoherente, **ProxyCalc** lo convierte al sufijo onmicrosoft.com predeterminado; por ejemplo: username@Contoso.onmicrosoft.com. 

En el caso de los usuarios sincronizados, la coherencia implica que **UserPrincipalName** está establecido en un sufijo de dominio comprobado y coincide con el valor de **UserPrincipalName** local (ShadowUserPrincipalName). Cuando se procesa un **UserPrincipalName** incoherente, **ProxyCalc** se revertirá al mismo valor de **ShadowUserPrincipalName** o, en caso de que se haya quitado el sufijo de dominio del inquilino, se convertirá al sufijo del dominio *.onmicrosoft.com predeterminado. 

  

#### <a name="what-does-proxy-address-consistency-mean"></a>¿Qué significa la coherencia de direcciones de proxy? 

En el caso de los usuarios que solo están en la nube, la coherencia implica que las direcciones de proxy coinciden con un sufijo de dominio comprobado. Cuando se procesa una dirección de proxy incoherente, **ProxyCalc** la convierte al sufijo del dominio onmicrosoft.com predeterminado; por ejemplo: SMTP:username@Contoso.onmicrosoft.com. 

En el caso de los usuarios sincronizados, la coherencia implica que las direcciones de proxy coinciden con los valores de las direcciones de proxy local (es decir, ShadowProxyAddresses). Se espera que las direcciones **ProxyAddresses** estén sincronizadas con **ShadowProxyAddresses**. Si el usuario sincronizado tiene asignada una licencia de Exchange, las direcciones de proxy deben coincidir con los valores de direcciones de proxy locales y con un sufijo de dominio comprobado. En este escenario, **ProxyCalc** corregirá la dirección de proxy incoherente con un sufijo de dominio sin comprobar y la quitará del objeto en Azure AD. Si el dominio sin comprobar se comprueba más adelante, **ProxyCalc** volverá a calcular y agregar la dirección de proxy de **ShadowProxyAddresses** al objeto en Azure AD.  

> [!NOTE]
> En el caso de los objetos sincronizados, para evitar que la lógica de **ProxyCalc** calcule resultados inesperados, se recomienda establecer las direcciones de proxy en un dominio comprobado de Azure AD en el objeto local.  

  
Una de las tareas de administración que podría desencadenar la lógica **ProxyCalc** es cada cambio de dominio comprobado. Esta tarea se produce cada vez que se agrega o quita un dominio comprobado en un inquilino de Azure AD, que desencadena internamente **ProxyCalc**.  

Por ejemplo, si agrega un dominio comprobado Fabrikam.com al inquilino Contoso.onmicrosoft.com, esta acción desencadenará una operación ProxyCalc en todos los objetos del inquilino. Este evento se capturará en los registros de auditoría de Azure AD como un evento **Actualizar usuario** precedidos por un evento **Agregar dominio comprobado**. Por otro lado, si Fabrikam.com se quitó del inquilino de Contoso.onmicrosoft.com, todos los eventos **Actualizar usuario** irán precedidos de un evento **Quitar dominio comprobado**.   

#### <a name="additional-notes"></a>Notas adicionales:

ProxyCalc no produce cambios en objetos determinados que: 

- no tienen una licencia de Exchange activa. 
- tienen **MSExchRemoteRecipientType** establecido en NULL. 
- no se consideran un recurso compartido. Un recurso compartido es cuando **CloudMSExchRecipientDisplayType** contiene alguno de los siguientes valores: **MailboxUser (compartido)** , **PublicFolder**, **ConferenceRoomMailbox**, **EquipmentMailbox**, **ArbitrationMailbox**, **RoomList**, **TeamMailboxUser**, **Buzón de grupo**, **Buzón de programación**, **ACLableMailboxUser** o **ACLableTeamMailboxUser**. 
  
 Con el fin de generar una mayor correlación entre estos dos eventos dispares, Microsoft está trabajando en la actualización de la información del **Actor** en los registros de auditoría para indicar que estos cambios los desencadenó un cambio de dominio comprobado. Esta acción le ayudará a comprobar cuándo tuvo lugar el evento de cambio de dominio comprobado y cuándo empezó a actualizar masivamente los objetos de su inquilino. 

Además, en la mayoría de los casos, no se realiza ningún cambio en los usuarios, ya que su **UserPrincipalName** y las **direcciones de proxy** son coherentes; por este motivo, estamos trabajando para mostrar en los registros de auditoría solo aquellas actualizaciones que causaron un cambio real en el objeto. Esta acción evitará la información innecesaria en los registros de auditoría y ayudará a que los administradores correlacionen los cambios de usuario restantes con el evento de cambio de dominio comprobado, tal como se explicó anteriormente. 

## <a name="next-steps"></a>Pasos siguientes

[Atributos paralelos del servicio de sincronización de Azure AD Connect](../hybrid/how-to-connect-syncservice-shadow-attributes.md)
