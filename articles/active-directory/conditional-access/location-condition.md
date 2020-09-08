---
title: Condición de ubicación del acceso condicional de Azure Active Directory
description: Obtenga información sobre el uso de la condición de ubicación para controlar el acceso a las aplicaciones en la nube en función de la ubicación de la red del usuario.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 06/15/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.custom: contperfq4
ms.openlocfilehash: 08e236d798f700a3c48dd41ba61941bc0037d613
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88055384"
---
# <a name="using-the-location-condition-in-a-conditional-access-policy"></a>Uso la condición de ubicación en una directiva de acceso condicional 

Como se explica en el [artículo de información general](overview.md), las directivas de acceso condicional se encuentran en la instrucción if-then más básica, que combina señales para tomar decisiones y aplicar directivas de la organización. Una de esas señales que se puede incorporar en el proceso de toma de decisiones es la ubicación de red.

![Señal condicional conceptual más la decisión de obtener el cumplimiento](./media/location-condition/conditional-access-signal-decision-enforcement.png)

Las organizaciones pueden usar esta ubicación de red para tareas comunes como: 

- Solicitar la autenticación multifactor a los usuarios que obtengan acceso a un servicio desde fuera de la red corporativa.
- Bloquear el acceso a los usuarios que accedan a un servicio desde determinados países o regiones.

La ubicación de red viene determinada por la dirección IP pública que proporciona un cliente a Azure Active Directory. De forma predeterminada, las directivas de acceso condicional se aplican a todas las direcciones IPv4 e IPv6. 

> [!TIP]
> Los intervalos de IPv6 solo se admiten en la interfaz de **[Ubicación con nombre (versión preliminar)](#preview-features)** . 

## <a name="named-locations"></a>Ubicaciones con nombre

Las ubicaciones se designan en Azure Portal en **Azure Active Directory** > **Seguridad** > **Acceso condicional** > **Ubicaciones con nombre**. Estas ubicaciones de red con nombre pueden incluir ubicaciones, como intervalos de redes de la sede central de una organización, intervalos de redes VPN o intervalos que desea bloquear. 

![Ubicación con nombre en Azure Portal](./media/location-condition/new-named-location.png)

Para configurar una ubicación, debe rellenar al menos el campo **Nombre** y especificar el intervalo IP. 

El número de ubicaciones con nombre que se pueden configurar está restringido por el tamaño del objeto relacionado en Azure AD. Puede configurar las ubicaciones en función de las limitaciones siguientes:

- Una ubicación con nombre de hasta 1200 intervalos IPv4.
- Un máximo de 90 ubicaciones con nombre con un intervalo IP asignado a cada una.

> [!TIP]
> Los intervalos de IPv6 solo se admiten en la interfaz de **[Ubicación con nombre (versión preliminar)](#preview-features)** . 

### <a name="trusted-locations"></a>Ubicaciones de confianza

Al crear una ubicación de red, un administrador tiene la opción de marcar una ubicación como una ubicación de confianza. 

![Ubicaciones de confianza en Azure Portal](./media/location-condition/new-trusted-location.png)

Esta opción puede incluirse en las directivas de acceso condicional, donde, por ejemplo, se puede requerir el registro de la autenticación multifactor desde una ubicación de red de confianza. También se incluye en el cálculo del riesgo de Azure AD Identity Protection, lo que reduce el riesgo de inicio de sesión de los usuarios desde una ubicación marcada como de confianza.

### <a name="countries-and-regions"></a>Países y regiones

Algunas organizaciones pueden optar por definir los límites de IP de países o regiones completos como ubicaciones con nombre para las directivas de acceso condicional. Estas ubicaciones pueden usarse al bloquear tráfico innecesario cuando se sabe que los usuarios válidos nunca provienen de una ubicación como Corea del Norte. Estas asignaciones de dirección IP a países se actualizan periódicamente. 

> [!NOTE]
> Los países no incluyen intervalos de direcciones IPv6, solo intervalos de direcciones IPv4 conocidos, por lo que no se pueden marcar como de confianza.

![Crear una nueva ubicación basada en el país o la región en Azure Portal](./media/location-condition/new-named-location-country-region.png)

#### <a name="include-unknown-areas"></a>Inclusión de áreas desconocidas

Algunas direcciones IP no están asignadas a ningún país o región específicos. Para capturar estas ubicaciones IP, active la casilla **Incluir áreas desconocidas** al definir una ubicación. Esta opción le permite elegir si estas direcciones IP deberían estar incluidas en la ubicación con nombre. Use esta configuración cuando la directiva que usa la ubicación con nombre deba aplicarse en ubicaciones desconocidas.

### <a name="configure-mfa-trusted-ips"></a>Configurar IP de confianza de MFA

También puede configurar los intervalos de direcciones IP que representen a la intranet local de su organización en la [configuración del servicio de la autenticación multifactor](https://account.activedirectory.windowsazure.com/usermanagement/mfasettings.aspx). Esta función le permite configurar hasta 50 intervalos de direcciones IP. Los intervalos de direcciones IP están en formato CIDR. Para obtener más información, vea [IP de confianza](../authentication/howto-mfa-mfasettings.md#trusted-ips).  

Si se han configurado IP de confianza, se mostrarán como **IP de confianza de MFA** en la lista de ubicaciones de la condición de ubicación.

### <a name="skipping-multi-factor-authentication"></a>Omisión de la autenticación multifactor

En la página de configuración del servicio de la autenticación multifactor, podrá identificar a los usuarios de la intranet corporativa seleccionando **Omitir la autenticación multifactor para solicitudes de usuarios federados en mi intranet**. Esta configuración indica que la notificación interna de la red corporativa, la cual emiten los Servicios de federación de Active Directory (AD FS), es de confianza y se utiliza para identificar al usuario como si estuviera en la red corporativa. Para obtener más información, vea [Habilitar la función de direcciones IP de confianza mediante el acceso condicional](../authentication/howto-mfa-mfasettings.md#enable-the-trusted-ips-feature-by-using-conditional-access).

Tras activar esta opción, incluida la ubicación con nombre, las **IP de confianza de MFA** se aplicarán a todas las directivas que tengan esta opción seleccionada.

Para las aplicaciones móviles y de escritorio que hayan tenido sesiones de larga duración, el acceso condicional se vuelve a evaluar periódicamente. El valor predeterminado es de una hora. Cuando la notificación dentro de la red corporativa solo se emite en el momento de la autenticación inicial, puede que Azure AD no tenga ninguna lista de intervalos de IP de confianza. En este caso, es más difícil determinar si el usuario sigue conectado a la red corporativa:

1. Compruebe si la dirección IP del usuario está en uno de los intervalos IP de confianza.
1. Compruebe si los tres primeros octetos de la dirección IP del usuario coinciden con los tres primeros octetos de la dirección IP de la autenticación inicial. La dirección IP se compara con la autenticación inicial cuando se emitió la notificación interna de red corporativa y se validó la ubicación del usuario.

Si se produce un error en los dos pasos, se considera que el usuario ya no está conectado a ninguna dirección IP de confianza.

## <a name="preview-features"></a>Características en vista previa

Además de la característica de ubicación con nombre disponible con carácter general, también existe una ubicación con nombre (versión preliminar). Puede tener acceso a la versión preliminar de ubicación con nombre mediante el banner situado en la parte superior de la hoja de ubicación con nombre actual.

![Probar la versión preliminar de ubicaciones con nombre](./media/location-condition/preview-features.png)

Con la versión preliminar de las ubicaciones con nombre, puede hacer lo siguiente:

- Configurar hasta 195 ubicaciones con nombre.
- Configurar hasta 2000 intervalos IP por ubicación con nombre.
- Configurar direcciones IPv6 junto con direcciones IPv4

También hemos agregado algunas comprobaciones adicionales para ayudar a reducir el cambio de configuración incorrecta.

- Los intervalos de IP privados ya no se pueden configurar.
- El número de direcciones IP que se pueden incluir en un intervalo es limitado. Solo se permitirán las máscaras CIDR mayores que/8 al configurar un intervalo de direcciones IP.

Con la versión preliminar, ahora hay dos opciones de creación: 

- **Ubicación de los países**
- **Ubicación de los intervalos de direcciones IP**

> [!NOTE]
> Los países no incluyen intervalos de direcciones IPv6, solo intervalos de direcciones IPv4 conocidos, por lo que no se pueden marcar como de confianza.

![Interfaz de la versión preliminar de ubicaciones con nombre](./media/location-condition/named-location-preview.png)

## <a name="location-condition-in-policy"></a>Condición de ubicación de la directiva

Al configurar la condición de ubicación, puede distinguir entre:

- Cualquier ubicación
- Todas las ubicaciones de confianza
- Ubicaciones seleccionadas

### <a name="any-location"></a>Cualquier ubicación

De manera predeterminada, al seleccionar **Cualquier ubicación**, se aplicará una directiva a todas las direcciones IP, lo que incluye cualquier dirección de Internet. Esta configuración no está limitada a las direcciones IP que haya configurado como ubicación con nombre. Al seleccionar **Cualquier ubicación**, todavía puede excluir determinadas ubicaciones de una directiva. Por ejemplo, puede aplicar una directiva en todas las ubicaciones (excepto en aquellas que sean de confianza) para establecer el ámbito en todas las ubicaciones menos en la red corporativa.

### <a name="all-trusted-locations"></a>Todas las ubicaciones de confianza

Esta opción se aplica a:

- Todas las ubicaciones que se hayan marcado como ubicación de confianza
- **IP de confianza de MFA** (si están configuradas)

### <a name="selected-locations"></a>Ubicaciones seleccionadas

Con esta opción, puede seleccionar una o varias ubicaciones con nombre. Para una directiva a la que se aplicará esta configuración, el usuario debe conectarse desde cualquiera de las ubicaciones seleccionadas. Al hacer clic en **Seleccionar**, se abrirá el control de selección de red con nombre que muestra la lista de redes con nombre. En la lista también se muestra si la ubicación de red se ha marcado como de confianza. La ubicación con nombre llamada **IP de confianza de MFA** se utiliza para incluir los valores de IP que pueden configurarse en la página de configuración del servicio de la autenticación multifactor.

## <a name="ipv6-traffic"></a>Tráfico de IPv6

De forma predeterminada, las directivas de acceso condicional se aplicarán a todo el tráfico de IPv6. Con la [versión preliminar de la ubicación con nombre](#preview-features), puede excluir intervalos de direcciones IPv6 específicos de una directiva de acceso condicional. Esta opción es útil en los casos en los que no se quiere aplicar la directiva en determinados intervalos IPv6. Por ejemplo, si no quiere aplicar una directiva en la red corporativa, y la red corporativa está hospedada en intervalos IPv6 públicos.  

### <a name="when-will-my-tenant-have-ipv6-traffic"></a>¿Cuándo tendrá mi inquilino tráfico IPv6?

Azure Active Directory (Azure AD) no admite actualmente conexiones de red directas que usan IPv6. No obstante, hay algunos casos en los que el tráfico de autenticación se realiza mediante proxy a través de otro servicio. En estos casos, la dirección IPv6 se usará durante la evaluación de la directiva.

La mayor parte del tráfico IPv6 que se dirige mediante proxy a Azure AD procede de Microsoft Exchange Online. Cuando estén disponibles, Exchange preferirá conexiones IPv6. **Por lo tanto, si tiene directivas de acceso condicional para Exchange que se han configurado para intervalos IPv4 específicos, querrá asegurarse de que también ha agregado los intervalos IPv6 de las organizaciones.** Si no se incluyen intervalos IPv6, se producirá un comportamiento inesperado en los dos casos siguientes:

- Cuando se usa un cliente de correo para conectarse a Exchange Online con autenticación heredada, Azure AD puede recibir una dirección IPv6. La solicitud de autenticación inicial va a Exchange y, después, se dirige mediante proxy a Azure AD.
- Si se usa Outlook Web Access (OWA) en el explorador, esta aplicación comprueba periódicamente que se están cumpliendo todas las directivas de acceso condicional. Esta comprobación sirve para detectar los casos en los que un usuario puede haber pasado de una dirección IP permitida a una nueva ubicación, como una cafetería de la calle. En este caso, si se usa una dirección IPv6 que no está en un intervalo configurado, es posible que el usuario vea su sesión interrumpida y se le redirija a Azure AD para que se vuelva a autenticar. 

Estos son los motivos más comunes por los que es posible que necesite configurar intervalos IPv6 en sus ubicaciones con nombre. Además, si usa redes virtuales de Azure, tendrá tráfico procedente de una dirección IPv6. Si el tráfico de red virtual está bloqueado por una directiva de acceso condicional, consulte el registro de inicio de sesión de Azure AD. Una vez que haya identificado el tráfico, puede obtener la dirección IPv6 que se está usando y excluirla de la directiva. 

> [!NOTE]
> Si quiere especificar un intervalo CIDR de IP para una sola dirección, aplique la máscara de /32 bits. Por ejemplo, si queremos excluir la dirección IPv6 2607:fb90:b27a:6f69:f8d5:dea0:fb39:74a como un intervalo, se usará 2607:fb90:b27a:6f69:f8d5:dea0:fb39:74a/32.

### <a name="identifying-ipv6-traffic-in-the-azure-ad-sign-in-activity-reports"></a>Identificación del tráfico IPv6 en los informes de actividad de inicio de sesión de Azure AD

Para detectar el tráfico IPv6 en el inquilino, vaya a [Informes de actividad de inicio de sesión en Azure AD](../reports-monitoring/concept-sign-ins.md). Una vez que haya abierto el informe de actividad, agregue la columna "dirección IP". Esta columna le permitirá identificar el tráfico IPv6.

También puede ver cuál es la dirección IP del cliente; para ello, haga clic en una fila del informe y, después, vaya a la pestaña "Ubicación" en los detalles de la actividad de inicio de sesión. 

## <a name="what-you-should-know"></a>Qué debería saber

### <a name="when-is-a-location-evaluated"></a>¿Cuando se evalúa una ubicación?

Las directivas de acceso condicional se evalúan cuando:

- Un usuario inicia sesión por primera vez en una aplicación web, móvil o de escritorio.
- Una aplicación móvil o de escritorio que usa la autenticación moderna, con un token de actualización para adquirir un nuevo token de acceso. De forma predeterminada, esta comprobación se realiza una vez por hora.

Esta comprobación significa que, para que las aplicaciones móviles y de escritorio usen la autenticación moderna, se detecta un cambio en la ubicación dentro de la hora siguiente al cambio de la ubicación de red. Para las aplicaciones de escritorio y móviles que no usan la autenticación moderna, la directiva se aplica para cada solicitud de token. La frecuencia de la solicitud varía en función de la aplicación. De forma similar, para las aplicaciones web, la directiva se aplica en el primer inicio de sesión y es adecuada para la duración de la sesión en la aplicación web. Debido a las diferencias en la duración de la sesión de las aplicaciones, el tiempo de una evaluación de directiva a otra también varía. Cada vez que la aplicación solicita un nuevo token de inicio de sesión, la directiva se aplica.

De manera predeterminada, Azure AD emite un token cada hora. Tras salir de la red corporativa, la directiva se aplica a las aplicaciones que usan la autenticación moderna en un plazo de una hora.

### <a name="user-ip-address"></a>Dirección IP del usuario

La dirección IP que se utiliza en la evaluación de directivas es la dirección IP pública del usuario. Para los dispositivos conectados a una red privada, esta no es la dirección IP de cliente del dispositivo del usuario conectado a la intranet, sino la dirección que utiliza la red para conectarse a la red pública de Internet.

### <a name="bulk-uploading-and-downloading-of-named-locations"></a>Carga y descarga masiva de ubicaciones con nombre

Al crear o actualizar ubicaciones con nombre, en el caso de las actualizaciones masivas, puede cargar o descargar un archivo CSV con los intervalos IP. Una carga reemplaza los intervalos IP que aparecen en la lista por los del archivo. Cada fila del archivo contiene un intervalo de direcciones IP en formato CIDR.

### <a name="cloud-proxies-and-vpns"></a>Servidores proxy en la nube y soluciones VPN

Cuando utiliza un proxy hospedado en la nube o una solución VPN, la dirección IP que utiliza Azure AD al evaluar una directiva es la dirección IP del proxy. No se usa el encabezado X-Forwarded-For (XFF) que contiene la dirección IP pública del usuario, ya que no hay ninguna validación de que provenga de un origen de confianza y podría ser un método de falsificación de la dirección IP.

Cuando hay en funcionamiento un servidor proxy en la nube, se puede utilizar una directiva que se usa para requerir un dispositivo unido a Azure AD híbrido o la notificación interna de la red corporativa desde AD FS.

### <a name="api-support-and-powershell"></a>Compatibilidad con la API y PowerShell

Una versión preliminar de Graph API para ubicaciones con nombre está disponible. Para obtener más información, consulte [namedLocation API](/graph/api/resources/namedlocation?view=graph-rest-beta).

## <a name="next-steps"></a>Pasos siguientes

- Si desea saber cómo configurar una directiva de acceso condicional, consulte [Creación de una directiva de acceso condicional](concept-conditional-access-policies.md).
- ¿Busca una directiva de ejemplo mediante la condición de ubicación? Consulte el artículo [Acceso condicional: Bloqueo del acceso por ubicación](howto-conditional-access-policy-location.md)
