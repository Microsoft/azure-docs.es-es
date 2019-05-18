---
title: Error "Can't Access this Corporate Application" (No se puede acceder a esta aplicación corporativa) al usar una aplicación de proxy de aplicación | Microsoft Docs
description: Cómo resolver problemas comunes de acceso con aplicaciones de proxy de aplicación de Azure AD.
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
ms.date: 05/21/2018
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 290b548f31a967d514b09e4de775b12de7703b07
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65782958"
---
# <a name="cant-access-this-corporate-application-error-when-using-an-application-proxy-application"></a>Error "Can't Access this Corporate Application" al usar una aplicación de Proxy de aplicación

Este artículo le ayudará a solucionar problemas comunes cuando aparezca un error tipo "This corporate app can't be accessed" ("No se puede acceder a esta aplicación empresarial") en una aplicación de Azure AD Application Proxy.

## <a name="overview"></a>Información general
Cuando aparezca este error, busque el código de estado en la página de error. Probablemente, el código sea uno de los siguientes códigos de estado:

-   **Gateway Timeout** (Tiempo de espera agotado para la puerta de enlace): el servicio de Proxy de aplicación no puede alcanzar el conector. Este error suele indicar un problema con la asignación del conector, el propio conector o las reglas de red relativas al conector.

-   **Bad Gateway** (Puerta de enlace incorrecta): el conector no puede alcanzar la aplicación back-end. Esto podría indicar un error de configuración de la aplicación.

-   **Forbidden** (Prohibido): el usuario no está autorizado para acceder a la aplicación. Este error puede ocurrir cuando el usuario no está asignado a la aplicación en Azure Active Directory o, si en el back-end, el usuario no tiene permiso para acceder a la aplicación.

Para encontrar el código, busque el texto en la parte inferior izquierda del mensaje de error del campo "Código de estado". Busque también sugerencias adicionales en la parte inferior de la página.

   ![Error de tiempo de espera agotado para la puerta de enlace](./media/application-proxy-sign-in-bad-gateway-timeout-error/connection-problem.png)

Para más información sobre cómo solucionar la causa de estos errores y más detalles sobre las correcciones sugeridas, consulte la sección correspondiente a continuación.

## <a name="gateway-timeout-errors"></a>Errores Gateway Timeout (Tiempo de espera agotado para la puerta de enlace)

Estos errores se producen cuando el servicio intenta alcanzar el conector y no lo consigue durante el tiempo de espera. Normalmente, este error se debe a que una aplicación se ha asignado a un grupo de conectores con conectores que no funcionan o a que algunos puertos necesarios para el conector no están abiertos.


## <a name="bad-gateway-errors"></a>Errores Bad Gateway (Puerta de enlace incorrecta)

Estos errores indican que el conector no puede alcanzar la aplicación back-end. Asegúrese de que ha publicado la aplicación correcta. Errores comunes que provocan este error:

-   Un error de escritura o un error en la dirección URL interna

-   Que la raíz de la aplicación no se haya publicado. Por ejemplo, publicando <http://expenses/reimbursement>, pero tratando de acceder a <http://expenses>.

-   Problemas con la configuración de la delegación restringida de Kerberos (KCD)

-   Problemas con la aplicación de back-end

## <a name="forbidden-errors"></a>Errores Forbidden (Prohibido)

Este tipo de error indica que el usuario no se ha asignado a la aplicación, ya sea en Azure Active Directory o en la aplicación de back-end.

Para información sobre cómo asignar usuarios a la aplicación en Azure, consulte la [documentación de configuración](application-proxy-add-on-premises-application.md#test-the-application).

Si confirma que el usuario está asignado a la aplicación en Azure, compruebe la configuración del usuario en la aplicación de back-end. Si está utilizando la delegación limitada de kerberos o la autenticación integrada de Windows, consulte la página de solución de problemas de KCD para obtener directrices.

## <a name="check-the-applications-internal-url"></a>Comprobación de la dirección URL interna de la aplicación

Como primer paso rápido, vuelva a comprobar y corrija la dirección URL interna. Para ello, abra la aplicación a través de **Aplicaciones empresariales** y seleccione el menú **Proxy de aplicación**. Compruebe que la dirección URL interna es el utilizado en la red local para tener acceso a la aplicación.

## <a name="check-the-application-is-assigned-to-a-working-connector-group"></a>Comprobación de que la aplicación está asignada a un grupo de conectores en funcionamiento

Para comprobar que la aplicación está asignada a un grupo de conectores en funcionamiento:

1. Abra la aplicación en el portal; para ello, vaya a **Azure Active Directory**, haga clic en **Aplicaciones empresariales** y en **Todas las aplicaciones**. Abra la aplicación y seleccione **Proxy de aplicación** en el menú izquierdo.

2. Fíjese en el campo Grupo de conectores. Si no hay ningún conector activo en el grupo, verá una advertencia. Si no ve advertencias, pase a "verify all required ports are whitelisted" (comprobar que todos los puertos necesarios están en la lista de permitidos).

3. Si se muestra el grupo de conectores incorrecto, seleccione el correcto en la lista desplegable y confirme que ya no ve las advertencias. Si se muestra el grupo de conectores previsto, haga clic en el mensaje de advertencia para abrir la página de administración del conector.

4. Desde aquí, podemos profundizar de varias maneras:

   * Mueva un conector activo al grupo: si tiene un conector activo que debería pertenecer a este grupo y tiene línea de visión a la aplicación back-end de destino, puede moverlo al grupo asignado. Para ello, haga clic en el conector. En el campo "Grupo de conectores", seleccione el grupo correcto en la lista desplegable y haga clic en Guardar.

   * Descargue un conector nuevo para ese grupo: en esta página, puede obtener el vínculo para [descargar un conector nuevo](https://download.msappproxy.net/Subscription/d3c8b69d-6bf7-42be-a529-3fe9c2e70c90/Connector/Download). Instale el conector en una máquina con línea directa de visión a la aplicación de back-end. Normalmente, el conector está instalado en el mismo servidor que la aplicación. Utilice el vínculo de descarga del conector para descargarlo en la máquina de destino. A continuación, haga clic en el conector y compruebe en la lista desplegable "Grupo de conectores" que pertenece al grupo adecuado.

   * Investigue un conector inactivo: si un conector aparece como inactivo, no puede conectarse con el servicio. Este error suele deberse al bloqueo de algunos puertos necesarios. Para resolver este problema, pase a "verify all required ports are whitelisted" (comprobar que todos los puertos necesarios están en la lista de permitidos).

Después de seguir estos pasos para asegurarse de que la aplicación está asignada a un grupo de conectores que funcionan, vuelva a probar la aplicación. Si sigue sin funcionar, continúe con la siguiente sección.

## <a name="check-all-required-ports-are-whitelisted"></a>Comprobación de que todos los puertos necesarios están en la lista de permitidos

Para comprobar que todos los puertos necesarios están abiertos, consulte la documentación sobre cómo abrir los puertos. Si todos los puertos necesarios están abiertos, vaya a la sección siguiente.

## <a name="check-for-other-connector-errors"></a>Búsqueda de otros errores de los conectores

Si el problema no se resuelve con ninguno de los pasos anteriores, el paso siguiente consiste en buscar problemas o errores en el propio conector. En el [documento de solución de problemas](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-troubleshoot#connector-errors) encontrará algunos errores comunes. 

También puede buscar directamente en los registros del conector para identificar los errores. Muchos de los mensajes de error incluyen recomendaciones específicas para la corrección. Para ver los registros, consulte la [documentación sobre los conectores](application-proxy-connectors.md#under-the-hood).

## <a name="additional-resolutions"></a>Soluciones adicionales

Si los pasos anteriores no solucionan el problema, existen otras posibles causas. Para identificar el problema:

Si la aplicación está configurada para usar autenticación integrada de Windows (IWA), pruebe la aplicación sin el inicio de sesión único. En caso contrario, pase al párrafo siguiente. Para comprobar la aplicación sin inicio de sesión único, abra la aplicación a través de **Aplicaciones empresariales** y vaya al menú **Inicio de sesión único**. Cambie el menú desplegable de "Autenticación integrada de Windows" a "Se desactivó el inicio de sesión único de Azure AD". 

Ahora abra un explorador e intente volver a acceder a la aplicación. Se le debería solicitar autenticación para acceder a la aplicación. Si puede realizar la autenticación, el problema reside en la configuración de delegación limitada de kerberos (KCD) que permite el inicio de sesión único. Para más información, consulte la página de solución de problemas de KCD.

Si continúa viendo el error, vaya a la máquina donde está instalado el conector, abra un explorador e intente conectar con la dirección URL interna que se utiliza para la aplicación. El conector actúa como otro cliente desde la misma máquina. Si no se puede acceder a la aplicación, investigue por qué la máquina no puede acceder a la aplicación o use un conector de un servidor que pueda acceder a ella.

Si puede conectarse a la aplicación desde esa máquina, busque problemas o errores en el propio conector. En el [documento de solución de problemas](application-proxy-troubleshoot.md#connector-errors) encontrará algunos errores comunes. También puede buscar directamente en los registros del conector para identificar los errores. Muchos de los mensajes de error incluyen recomendaciones más específicas para la corrección. Para más información sobre cómo ver los registros, consulte [nuestra documentación sobre los conectores](application-proxy-connectors.md#under-the-hood).

## <a name="next-steps"></a>Pasos siguientes
[Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md)
