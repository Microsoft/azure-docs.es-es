---
title: Informe de Azure AD Connect Health con direcciones IP de riesgo de AD FS | Microsoft Docs
description: Describe el informe de Azure AD Connect Health con direcciones IP de riesgo.
services: active-directory
documentationcenter: ''
ms.reviewer: zhiweiwangmsft
author: billmath
manager: daveba
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/26/2019
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: 37489e2f2df063c6a974eaf5d401e6192cbf027c
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89296369"
---
# <a name="risky-ip-report-public-preview"></a>Informe de direcciones IP de riesgo (versión preliminar pública)
Los clientes de AD FS pueden exponer puntos de conexión de autenticación de contraseña en Internet para proporcionar servicios de autenticación para que los usuarios finales tengan acceso a aplicaciones de SaaS, como Office 365. En este caso, es posible que un actor no deseado intente inicios de sesión en el sistema de AD FS para adivinar la contraseña de usuario final y acceder a los recursos de la aplicación. AD FS proporciona la funcionalidad de bloqueo de cuenta de extranet para evitar estos tipos de ataques desde AD FS en Windows Server 2012 R2. Si se encuentra en una versión anterior, se recomienda que actualice el sistema de AD FS a Windows Server 2016. <br />

Además, es posible que una única dirección IP intente varios inicios de sesión en varios usuarios. En estos casos, puede que el número de intentos por usuario esté por debajo del umbral de la protección de bloqueo de cuenta de AD FS. Azure AD Connect Health ahora proporciona el "informe de direcciones IP de riesgo", que detecta esta condición y notifica a los administradores cuando esto ocurre. Estas son las ventajas principales de este informe: 
- Detección de direcciones IP que superan un umbral de inicios de sesión basada en contraseña erróneos
- Admite inicios de sesión con error debidos a una contraseña incorrecta o debidos a un estado de bloqueo de la extranet
- Notificación por correo electrónico a los administradores de la alerta en cuanto esto sucede con configuración de correo electrónico personalizable
- Configuración de umbrales personalizables ajustados a la directiva de seguridad de una organización
- Informes descargables para su análisis sin conexión e integración con otros sistemas mediante automatización

> [!NOTE]
> Para usar este informe, debe asegurarse de que está habilitada la auditoría de AD FS. Para obtener más información, consulte [Habilitación de la auditoría para AD FS](how-to-connect-health-agent-install.md#enable-auditing-for-ad-fs). <br />
> Para acceder a la versión preliminar, se requiere el permiso de Administrador global o [Lector de seguridad](../../role-based-access-control/built-in-roles.md#security-reader).  
> 

## <a name="what-is-in-the-report"></a>Contenido del informe
Las direcciones IP del cliente de la actividad de inicio de sesión con error se agregan mediante los servidores proxy de aplicación web. Cada elemento del informe de direcciones IP de riesgo muestra información agregada sobre las actividades de inicio de sesión de AD FS con errores que sobrepasan el umbral designado. Este ofrece la siguiente información: ![Portal de Azure AD Connect Health](./media/how-to-connect-health-adfs/report4a.png)

| Elemento de informe | Descripción |
| ------- | ----------- |
| Marca de tiempo | Muestra la marca de tiempo según la hora local de Azure Portal cuando se inicia la ventana de tiempo de detección.<br /> Todos los eventos diarios se generan a media noche hora UTC. <br />Los eventos por hora tienen la marca de tiempo redondeada al principio de la hora. Puede encontrar la primera hora de inicio de actividad en "firstAuditTimestamp" en el archivo exportado. |
| Tipo de desencadenador | Muestra el tipo de ventana de tiempo de detección. Los tipos de desencadenador de agregación son por hora o por día. Esto resulta útil para detecciones de ataques por fuerza bruta de alta frecuencia frente a un ataque lento en el que el número de intentos se distribuye a lo largo del día. |
| Dirección IP | La dirección IP de riesgo individual con actividades de inicio de sesión con contraseña incorrecta o de bloqueo de extranet. Podría tratarse de una dirección IPv4 o IPv6. |
| Número de errores de contraseña incorrecta | El número de errores de contraseña incorrecta producidos desde la dirección IP durante la ventana de tiempo de detección. Los errores de contraseña incorrecta pueden producirse varias veces con determinados usuarios. Tenga en cuenta que no se incluyen los intentos con error debidos a contraseñas que han expirado. |
| Número de errores de bloqueo de extranet | El número de errores de bloqueo de extranet que se han producido desde la dirección IP durante la ventana de tiempo de detección. Los errores de bloqueo de extranet pueden producirse varias veces con determinados usuarios. Esto solo es visible si se configura el bloqueo de extranet de AD FS (versiones 2012R2 o superior). <b>Nota:</b> Se recomienda activar esta característica si se permiten inicios de sesión de extranet mediante contraseñas. |
| Usuarios únicos con intentos | El número de cuentas de usuario únicas con intentos desde la dirección IP durante la ventana de tiempo de detección. Esto proporciona un mecanismo para diferenciar un patrón de ataque de un solo usuario en comparación con un patrón de ataque de varios usuarios.  |

Por ejemplo, el siguiente elemento del informe indica que en la ventana de las 6 p.m. a las 7 p.m. el 28/02/2018, la dirección IP <i>104.2XX.2XX.9</i> no tenía errores de contraseña incorrecta y tenía 284 errores de bloqueo de extranet. 14 usuarios únicos resultaron afectados dentro de los criterios. El evento de actividad había superado el umbral horario del informe designado. 

![portal de Azure AD Connect Health](./media/how-to-connect-health-adfs/report4b.png)

> [!NOTE]
> - En la lista del informe se mostrarán únicamente las actividades que superan el umbral designado. 
> - Este informe se puede seguir hacia atrás hasta 30 días como máximo.
> - Este informe de alertas no muestra las direcciones IP de Exchange ni las direcciones IP privadas. En cambio, se incluyen en la lista de exportación. 
>

![portal de Azure AD Connect Health](./media/how-to-connect-health-adfs/report4c.png)

## <a name="load-balancer-ip-addresses-in-the-list"></a>Direcciones IP del equilibrador de carga de la lista
La instancia agregada de Load Balancer no pudo iniciar la sesión de las actividades y alcanzó el umbral de alerta. Si ve direcciones IP del equilibrador de carga, es muy probable que el equilibrador de carga externo no esté enviando la dirección IP del cliente cuando pasa la solicitud al servidor proxy de la aplicación web. Configure el equilibrador de carga correctamente para que pase la dirección IP de reenvío del cliente. 

## <a name="download-risky-ip-report"></a>Descarga del informe de direcciones IP de riesgo 
Mediante la funcionalidad **Descargar**, se puede exportar la lista de direcciones IP de riesgo completa de los últimos 30 días desde el portal de Connect Health. El resultado de la exportación incluirá todas las actividades de inicio de sesión de AD FS con error en cada ventana de tiempo de detección para que pueda personalizar el filtrado después de la exportación. Además de las agregaciones resaltadas en el portal, el resultado de la exportación también muestra más detalles sobre las actividades de inicio de sesión con error por dirección IP:

|  Elemento de informe  |  Descripción  | 
| ------- | ----------- | 
| firstAuditTimestamp | Muestra la primera marca de tiempo en la que se iniciaron las actividades con errores durante la ventana de tiempo de detección.  | 
| lastAuditTimestamp | Muestra la última marca de tiempo en la que finalizaron las actividades con errores durante la ventana de tiempo de detección.  | 
| attemptCountThresholdIsExceeded | La marca indica si las actividades actuales superan el umbral de alerta.  | 
| isWhitelistedIpAddress | La marca indica si se filtra la dirección IP de las alertas e informes. Las direcciones IP privadas (<i>10.x.x.x, 172.x.x.x y 192.168.x.x</i>) y las direcciones IP de Exchange se filtran y se marcan como True. Si ve intervalos de direcciones IP privadas, es muy probable que el equilibrador de carga externo no esté enviando la dirección IP del cliente cuando pasa la solicitud al servidor proxy de la aplicación web.  | 

## <a name="configure-notification-settings"></a>Configuración de las opciones de notificación
Los contactos administrativos del informe se pueden actualizar en la **Configuración de notificaciones**. De forma predeterminada, la notificación por correo electrónico de alertas de IP de riesgo está desactivada. Puede habilitar las notificaciones mediante el botón conmutador "Obtener notificaciones por correo electrónico de las direcciones IP que superen el umbral del informe de actividades con error". Al igual que las opciones de notificación de alertas de Connect Health, le permite personalizar la lista de destinatarios de las notificaciones desde aquí. También puede notificar a todos los administradores globales al realizar el cambio. 

## <a name="configure-threshold-settings"></a>Configuración de las opciones del umbral
El umbral de alerta se puede actualizar mediante las Opciones del umbral. De forma inicial, el sistema tiene un umbral establecido de forma predeterminada. Hay cuatro categorías en las opciones del umbral del informe de direcciones IP de riesgo:

![portal de Azure AD Connect Health](./media/how-to-connect-health-adfs/report4d.png)

| Elemento del umbral | Descripción |
| --- | --- |
| Usuario y contraseña incorrectos y bloqueos de extranet por día  | Opción del umbral para informar de la actividad y desencadenar la notificación de alertas cuando el número de contraseñas erróneas más el número de bloqueos de extranet se ha superado por **día**. |
| Usuario y contraseña incorrectos y bloqueos de extranet por hora | Opción del umbral para informar de la actividad y desencadenar la notificación de alertas cuando el número de contraseñas erróneas más el número de bloqueos de extranet se ha superado por **hora**. |
| Bloqueos de extranet por día | Opción del umbral para informar de la actividad y desencadenar la notificación de alertas cuando el número de bloqueos de extranet se ha superado por **día**. |
| Bloqueos de extranet por hora| Opción del umbral para informar de la actividad y desencadenar la notificación de alertas cuando el número de bloqueos de extranet se ha superado por **hora**. |

> [!NOTE]
> - El cambio del umbral del informe se aplicará una hora después del cambio de la opción. 
> - El cambio del umbral no afectará a los elementos notificados existentes. 
> - Se recomienda que analice el número de eventos observados en su entorno y ajuste el umbral de forma adecuada. 
>
>

## <a name="faq"></a>Preguntas más frecuentes
**¿Por qué veo intervalos de direcciones IP privadas en el informe?**  <br />
Las direcciones IP privadas (<i>10.x.x.x, 172.x.x.x y 192.168.x.x</i>) y las direcciones IP de Exchange se filtran y se marcan como True en la lista de direcciones IP aprobadas. Si ve intervalos de direcciones IP privadas, es muy probable que el equilibrador de carga externo no esté enviando la dirección IP del cliente cuando pasa la solicitud al servidor proxy de la aplicación web.

**¿Por qué veo direcciones IP del equilibrador de carga en el informe?**  <br />
Si ve direcciones IP del equilibrador de carga, es muy probable que el equilibrador de carga externo no esté enviando la dirección IP del cliente cuando pasa la solicitud al servidor proxy de la aplicación web. Configure el equilibrador de carga correctamente para que pase la dirección IP de reenvío del cliente. 

**¿Qué hacer para bloquear la dirección IP?**  <br />
Debe agregar la dirección IP identificada como malintencionada al firewall o bloquearla en Exchange.   <br />

**¿Por qué no veo ningún elemento de este informe?** <br />
- Las actividades de inicio de sesión con errores no superan los valores del umbral.
- Asegúrese de que no haya ninguna alerta del tipo "Health Service no está actualizado" en la lista de servidores de AD FS.  Más información sobre [cómo solucionar problemas de esta alerta](how-to-connect-health-data-freshness.md).
- Las auditorías no están habilitadas en las granjas de servidores de AD FS.

**¿Por qué no veo ningún acceso al informe?**  <br />
Se requiere el permiso de administrador global o de [lector de seguridad](../../role-based-access-control/built-in-roles.md#security-reader). Póngase en contacto con el administrador global para obtener acceso.


## <a name="next-steps"></a>Pasos siguientes
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Instalación del agente de Azure AD Connect Health](how-to-connect-health-agent-install.md)