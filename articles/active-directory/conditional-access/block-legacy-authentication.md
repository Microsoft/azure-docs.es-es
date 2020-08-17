---
title: 'Bloqueo de la autenticación heredada: Azure Active Directory'
description: Aprenda a mejorar su seguridad mediante el bloqueo de la autenticación heredada con el acceso condicional de Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/13/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5d3df4eee14e5ce2f0638058efde0f80d0e5b051
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87275486"
---
# <a name="how-to-block-legacy-authentication-to-azure-ad-with-conditional-access"></a>Procedimientos: Bloqueo de la autenticación heredada en Azure AD con acceso condicional   

Para brindar a los usuarios un acceso sencillo a las aplicaciones en la nube, Azure Active Directory (Azure AD) admite una amplia variedad de protocolos de autenticación, incluida la autenticación heredada. Sin embargo, los protocolos heredados no admiten la autenticación multifactor (MFA). En muchos entornos, MFA es un requisito común para enfrentar el robo de identidad. 

En su entrada de blog del 12 de marzo de 2020, [New tools to block legacy authentication in your organization](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/new-tools-to-block-legacy-authentication-in-your-organization/ba-p/1225302#), Alex Weinert, Director de seguridad de identidades en Microsoft, destaca los motivos por los que las organizaciones deben bloquear la autenticación heredada, y señala otras herramientas que Microsoft proporciona para realizar esta tarea:

> Para que MFA sea efectivo, también es necesario bloquear la autenticación heredada. Esto se debe a que los protocolos de autenticación heredados como POP, SMTP, IMAP y MAPI no pueden aplicar MFA, lo que los convierte en los puntos de entrada preferidos de los adversarios que atacan su organización.
> 
>Las cifras relativas a la autenticación heredada que ha arrojado un análisis de tráfico de Azure Active Directory (Azure AD) son inapelables:
> 
> - Más del 99 por ciento de los ataques de difusión de contraseña usan protocolos de autenticación heredados.
> - Más del 97 por ciento de los ataques de obtención de credenciales usan la autenticación heredada.
> - Las cuentas de Azure AD de las organizaciones que han deshabilitado la autenticación heredada experimentaron un 67 % menos de riesgos que aquellas en las que la autenticación heredada está habilitada.
>

Si el entorno está listo para bloquear la autenticación heredada con el fin de mejorar la protección del inquilino, puede lograr este objetivo con el acceso condicional. En este artículo se explica cómo configurar las directivas de acceso condicional que bloquean la autenticación heredada para el inquilino.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se asume que está familiarizado con: 

- Los [conceptos básicos](overview.md) del acceso condicional de Azure AD 
- Los [procedimientos recomendados](best-practices.md) para configurar directivas de acceso condicional en Azure Portal

## <a name="scenario-description"></a>Descripción del escenario

Azure AD admite varios de los protocolos de autenticación y autorización usados más comúnmente, incluida la autenticación heredada. La autenticación heredada hace referencia a los protocolos que usan la autenticación básica. Por lo general, estos protocolos no pueden aplicar ningún tipo de autenticación de segundo factor. Estos son algunos de los ejemplos de las aplicaciones basadas en la autenticación heredada:

- Aplicaciones anteriores de Microsoft Office
- Aplicaciones que usan protocolos de correo como POP, IMAP y SMTP

La autenticación de un solo factor (por ejemplo, nombre de usuario y contraseña) ya no es suficiente. No se recomiendan las contraseñas, porque son fáciles de adivinar y porque los usuarios humanos no suelen elegir contraseñas seguras. Las contraseñas también son vulnerables ante una variedad de ataques, como suplantación de identidad (phishing) y difusión de contraseña. Una de las medidas más sencillas que puede tomar para protegerse contra las amenazas para las contraseñas es implementar la autenticación multifactor. Con MFA, incluso si el atacante cuenta con la contraseña del usuario, la contraseña por sí sola no es suficiente para autenticarse de manera correcta y acceder a los datos.

¿Cómo se puede evitar que las aplicaciones que usan la autenticación heredada accedan a los recursos del inquilino? La recomendación es simplemente bloquearlas con una directiva de acceso condicional. Si es necesario, puede permitir que solo ciertos usuarios y ubicaciones de red específicas usen aplicaciones basadas en la autenticación heredada.

Las directivas de acceso condicional se aplican una vez que se completa la autenticación en una fase. Por lo tanto, el acceso condicional no está pensado como primera línea de defensa en escenarios de tipo ataques por denegación de servicio (DoS), pero puede usar señales de estos eventos (por ejemplo, el nivel de riesgo de inicio de sesión, la ubicación de la solicitud, etc.) para determinar el acceso.

## <a name="implementation"></a>Implementación

En esta sección se explica cómo configurar una directiva de acceso condicional para bloquear la autenticación heredada. 

### <a name="legacy-authentication-protocols"></a>Protocolos de autenticación heredados

Las siguientes opciones se consideran protocolos de autenticación heredados.

- SMTP autenticado: usado por clientes POP e IMAP para enviar mensajes de correo electrónico.
- Detección automática: usada por clientes Outlook y EAS para buscar y conectarse a buzones en Exchange Online.
- Exchange ActiveSync (EAS): se usa para conectarse a los buzones en Exchange Online.
- Exchange Online PowerShell: se usa para conectarse a Exchange Online con PowerShell remoto. Si bloquea la autenticación básica para Exchange Online PowerShell, debe usar el módulo de Exchange Online PowerShell para conectarse. Para obtener instrucciones, consulte [Conexión a Exchange Online PowerShell con autenticación multifactor](/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/mfa-connect-to-exchange-online-powershell).
- Servicios web Exchange (EWS): una interfaz de programación que se usa en Outlook, Outlook para Mac y aplicaciones de terceros.
- IMAP4: usado por clientes de correo electrónico IMAP.
- MAPI sobre HTTP (MAPI/HTTP): usado por Outlook 2010 y versiones posteriores.
- Libreta de direcciones sin conexión (OAB): una copia de las colecciones de listas de direcciones que Outlook descarga y usa.
- Outlook en cualquier lugar (RPC a través de HTTP): usado por Outlook 2016 y versiones anteriores.
- Servicio Outlook: usado por la aplicación de correo electrónico y calendario de Windows 10.
- POP3: usado por clientes de correo electrónico POP.
- Servicios web de creación de informes: se usan para recuperar datos de informes en Exchange Online.
- Otros clientes: otros protocolos que usen autenticación heredada.

Para más información sobre estos protocolos y servicios de autenticación, vea [Informes de actividad de inicio de sesión en el portal de Azure Active Directory](../reports-monitoring/concept-sign-ins.md#filter-sign-in-activities).

### <a name="identify-legacy-authentication-use"></a>Identificación del uso de la autenticación heredada

Para poder bloquear la autenticación heredada en su directorio, primero debe entender si los usuarios tienen aplicaciones que la usen y cómo afecta a su directorio global. Se pueden usar los registros de inicio de sesión de Azure AD para saber si usa la autenticación heredada.

1. Vaya a **Azure Portal** > **Azure Active Directory** > **Inicios de sesión**.
1. Agregue la columna Aplicación cliente si no se muestra; para ello, haga clic en **Columnas** > **Aplicación cliente**.
1. **Agregar filtros** > **Aplicación cliente** > seleccione todos los protocolos de autenticación heredados. Seleccione fuera del cuadro de diálogo de filtrado para aplicar las selecciones y cierre el cuadro de diálogo.

Al filtrar solo se muestran los intentos de inicio de sesión que se realizaron con protocolos de autenticación heredada. Al hacer clic en cada intento de inicio de sesión individual se muestran detalles adicionales. El campo **Aplicación cliente** en la pestaña **Información básica** indicará qué protocolo de autenticación heredada se usó.

Estos registros indicarán qué usuarios dependen todavía de la autenticación heredada y qué aplicaciones usan protocolos heredados para realizar solicitudes de autenticación. Para los usuarios que no aparecen en estos registros y se confirme que no van a usar la autenticación heredada, implemente una directiva de acceso condicional solo para estos usuarios.

### <a name="block-legacy-authentication"></a>Bloquear la autenticación heredada 

En una directiva de acceso condicional, puede establecer una condición vinculada a las aplicaciones cliente que se usan para acceder a los recursos. La condición de aplicaciones cliente le permite restringir el ámbito a las aplicaciones que usan la autenticación heredada mediante la selección de **clientes de Exchange ActiveSync** y **otros clientes** en **Aplicaciones móviles y aplicaciones de escritorio**.

![Otros clientes](./media/block-legacy-authentication/01.png)

Para bloquear el acceso a estas aplicaciones, debe seleccionar **Bloquear acceso**.

![Bloquear acceso](./media/block-legacy-authentication/02.png)

### <a name="select-users-and-cloud-apps"></a>Selección de usuarios y aplicaciones en la nube

Si quiere bloquear la autenticación heredada en su organización, probablemente piense que puede hacerlo si selecciona:

- Todos los usuarios
- Todas las aplicaciones en la nube
- Bloquear acceso

![Assignments](./media/block-legacy-authentication/03.png)

Azure incluye una característica de seguridad que le impide crear una directiva similar, porque esta configuración infringe los [procedimientos recomendados](best-practices.md) para las directivas de acceso condicional.
 
![Configuración de directiva no compatible](./media/block-legacy-authentication/04.png)

La característica de seguridad es necesaria porque el *bloqueo de todos los usuarios y todas las aplicaciones en la nube* tiene el potencial de impedir que toda la organización inicie sesión en el inquilino. Se debe excluir al menos un usuario para cumplir con el requisito mínimo del procedimiento recomendado. También puede excluir un rol de directorio.

![Configuración de directiva no compatible](./media/block-legacy-authentication/05.png)

Para satisfacer esta característica de seguridad, excluya un usuario de la directiva. Idealmente, debe definir algunas [cuentas administrativas de acceso de emergencia en Azure AD](../users-groups-roles/directory-emergency-access.md) y excluirlas de la directiva.

El uso del [modo de solo informe](concept-conditional-access-report-only.md) al habilitar la directiva para bloquear la autenticación heredada brinda a la organización la oportunidad de ver cuál sería el impacto de la directiva.

## <a name="policy-deployment"></a>Implementación de directivas

Antes de implementar la directiva en el entorno de producción, encárguese de lo siguiente:
 
- **Cuentas de servicio**: identifique las cuentas de usuario que se utilizan como cuentas de servicio o a través de dispositivos, como teléfonos en salas de conferencias. Asegúrese de que estas cuentas tienen contraseñas seguras y agréguelas a un grupo de exclusión.
- **Informes de inicio de sesión**: revise el informe de inicio de sesión y busque el tráfico de **otro cliente**. Identifique el uso principal e investigué por qué está en uso. Por lo general, el tráfico lo generan clientes de Office más antiguos que no usan la autenticación moderna, o bien algunas aplicaciones de correo de terceros. Elabore un plan para quitar el uso de estas aplicaciones, o bien, si el impacto es bajo, notifique a los usuarios que ya no pueden usarlas.
 
Para más información, consulte [¿Cómo debe implementar una directiva nueva?](best-practices.md#how-should-you-deploy-a-new-policy)

## <a name="what-you-should-know"></a>Qué debería saber

Al bloquear el acceso mediante **Otros clientes** también se impide que PowerShell de Exchange Online y Dynamics 365 usen la autenticación básica.

La configuración de una directiva para **otros clientes** bloquea toda la organización ante determinados clientes como SPConnect. Este bloqueo se produce porque clientes más antiguos se autentican de formas inesperadas. Este problema no aplica a las aplicaciones principales de Office, como los clientes de Office anteriores.

La directiva puede tardar hasta 24 horas en surtir efecto.

Puede seleccionar todos los controles de concesión disponibles para la condición **Otros clientes**, pero la experiencia del usuario final siempre es la misma: el acceso bloqueado.

Si bloquea la autenticación heredada con la condición **Otros clientes**, también puede establecer la plataforma del dispositivo y la condición de la ubicación. Por ejemplo, si solo desea bloquear la autenticación heredada para dispositivos móviles, establezca la condición **plataformas de dispositivo** seleccionando:

- Android
- iOS
- Windows Phone

![Configuración de directiva no compatible](./media/block-legacy-authentication/06.png)

## <a name="next-steps"></a>Pasos siguientes

- [Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-report-only.md)
- Si todavía no sabe cómo configurar las directivas de acceso condicional, consulte [Exigir MFA para aplicaciones específicas con acceso condicional de Azure Active Directory](app-based-mfa.md) para ver un ejemplo.
- Para más información sobre la compatibilidad con la autenticación moderna, consulte [Cómo funciona la autenticación moderna para las aplicaciones de cliente de Office 2013 y Office 2016](/office365/enterprise/modern-auth-for-office-2013-and-2016). 
- [Cómo configurar una aplicación o dispositivo multifunción para enviar correos electrónicos mediante Office 365 y Microsoft 365](/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-office-3)
