---
title: 'Alta disponibilidad del servidor Azure MFA: Azure Active Directory'
description: Implemente varias instancias del Servidor Microsoft Azure Multi-Factor Authentication en configuraciones que proporcionan alta disponibilidad.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8f5f1e123447bfff53a603bad23b0444a93e4a83
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919631"
---
# <a name="configure-azure-multi-factor-authentication-server-for-high-availability"></a>Configuración del Servidor Microsoft Azure Multi-Factor Authentication para la alta disponibilidad

Para conseguir una alta disponibilidad con la implementación del Servidor Azure MFA, debe implementar varios servidores MFA. En esta sección se proporciona información sobre un diseño de carga equilibrada para conseguir objetivos de alta disponibilidad en la implementación del Servidor Azure MFA.

> [!IMPORTANT]
> A partir del 1 de julio de 2019, Microsoft ya no ofrecerá el servidor MFA para implementaciones nuevas. Los clientes nuevos que quieran exigir la autenticación multifactor (MFA) durante los eventos de inicio de sesión deben usar Azure Multi-Factor Authentication basado en la nube.
>
> Para empezar a trabajar con MFA basado en la nube, consulte [Tutorial: Protección de eventos de inicio de sesión de usuario con Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Los clientes existentes que hayan activado el Servidor MFA antes del 1 de julio de 2019 podrán descargar la versión más reciente y las actualizaciones futuras, así como generar credenciales de activación como de costumbre.

## <a name="mfa-server-overview"></a>Introducción al Servidor MFA

La arquitectura del servicio del Servidor Azure MFA consta de varios componentes, como se muestra en el siguiente diagrama:

 ![Componentes clave de la arquitectura del servidor MFA](./media/howto-mfaserver-deploy-ha/mfa-ha-architecture.png)

Un Servidor MFA es un servidor con Windows Server que tiene instalado el software Azure Multi-Factor Authentication. El servicio MFA debe activar la instancia del Servidor MFA en Azure para que dicho servidor funcione. Se puede instalar más de una instancia del Servidor MFA en el entorno local.

De forma predeterminada, la primera instancia del Servidor MFA instalada se corresponde con la instancia maestra del Servidor MFA tras la activación realizada por el servicio Azure MFA. La instancia maestra del Servidor MFA tiene una copia grabable de la base de datos PhoneFactor.pfdata. Las instalaciones posteriores de instancias del Servidor MFA se conocen como subordinadas. Las instancias subordinadas de MFA tienen una copia replicada de solo lectura de la base de datos PhoneFactor.pfdata. Las instancias del Servidor MFA replican información mediante la llamada a procedimiento remoto (RPC). Todas las instancias del Servidor MFA deben tener en común estar unidas a un dominio o ser independientes para replicar la información.

Las instancias maestras y subordinadas del Servidor MFA se comunican con el servicio MFA cuando se requiere la autenticación en dos fases. Por ejemplo, cuando un usuario intenta acceder a una aplicación que requiere autenticación en dos fases, al usuario lo autenticará primero un proveedor de identidades, como Active Directory (AD).

Tras la autenticación correcta con AD, la instancia del Servidor MFA se comunicará con el servicio MFA. La instancia del Servidor MFA espera a recibir una notificación del servicio MFA para permitir o denegar el acceso del usuario a la aplicación.

Si la instancia maestra del Servidor MFA se queda sin conexión, las autenticaciones todavía pueden procesarse, pero no las operaciones que requieren cambios en la base de datos de MFA. (Algunos ejemplos son la incorporación de usuarios, los cambios de PIN de autoservicio, las modificaciones de información del usuario o el acceso al portal de usuarios)

## <a name="deployment"></a>Implementación

Tenga en cuenta los siguientes puntos importantes para el equilibrio de carga del Servidor Azure MFA y los componentes relacionados.

* **Uso del protocolo RADIUS para conseguir alta disponibilidad**. Si usa instancias del Servidor Azure MFA como servidores RADIUS, puede configurar un Servidor MFA como un destino de autenticación de RADIUS principal y otras instancias del Servidor Azure MFA como destinos de autenticación secundarios. Sin embargo, este método para lograr alta disponibilidad puede no resultar práctico, porque debe transcurrir un tiempo de espera en caso de errores de autenticación en el destino de autenticación principal para poder realizar la autenticación en el destino de autenticación secundario. Resulta más eficaz equilibrar la carga del tráfico RADIUS entre el cliente RADIUS y los servidores RADIUS (en este caso, las instancias del Servidor Azure MFA actúan como servidores RADIUS), para que pueda configurar los clientes RADIUS con una sola dirección URL a la que puedan apuntar.
* **Necesidad de promover manualmente las instancias subordinadas de MFA**. Si la instancia maestra del Servidor Azure MFA se queda sin conexión, la instancia secundaria continúa con el procesamiento de solicitudes de MFA. Sin embargo, hasta que la instancia maestra del Servidor MFA no esté disponible, los administradores no pueden agregar usuarios ni modificar la configuración de MFA, y los usuarios no pueden realizar cambios a través del portal de usuarios. La promoción de una instancia subordinada de MFA al rol principal siempre se realiza con un proceso manual.
* **Separación de los componentes**. El Servidor Azure MFA consta de varios componentes que se pueden instalar en la misma instancia de Windows Server o en instancias diferentes. Estos componentes incluyen el Portal de usuarios, el servicio web de aplicación móvil y el adaptador de AD FS (agente). Esta separación permite usar el Proxy de aplicación web para publicar en el Portal de usuarios y en el servidor web de aplicación móvil desde la red perimetral. Esta configuración se agrega a la seguridad global del diseño, tal como se muestra en el diagrama siguiente. El Portal de usuarios de MFA y el servidor web de aplicación móvil también se pueden implementar en las configuraciones de carga equilibrada de alta disponibilidad.

   ![Servidor MFA con una red perimetral](./media/howto-mfaserver-deploy-ha/mfasecurity.png)

* **La contraseña de un solo uso (OTP) a través de SMS (conocido también como SMS unidireccional) requiere usar sesiones temporales si se trata de tráfico de carga equilibrada**. SMS unidireccional es una opción de autenticación que hace que la instancia del Servidor MFA envíe a los usuarios un mensaje de texto que contiene una OTP. El usuario escribe la OTP en una ventana del símbolo del sistema para completar el desafío de MFA. Si las instancias del Servidor Azure MFA son de carga equilibrada, el mismo servidor que atiende la solicitud de autenticación inicial debe ser el servidor que recibe el mensaje de la OTP del usuario; si otra instancia del Servidor MFA recibe la respuesta de la OTP, se produce un error en el desafío de autenticación. Para más información, vea [One Time Password over SMS Added to Azure MFA Server](https://blogs.technet.microsoft.com/enterprisemobility/2015/03/02/one-time-password-over-sms-added-to-azure-mfa-server) (Contraseña de un solo uso a través de SMS agregada a Servidor Azure MFA).
* **Las implementaciones de carga equilibrada del Portal de usuarios y del servicio web de aplicación móvil requieren sesiones temporales**. Si va a equilibrar la carga del Portal de usuarios de MFA y del servicio web de aplicación móvil, cada sesión debe estar en el mismo servidor.

## <a name="high-availability-deployment"></a>Implementación de alta disponibilidad

En el diagrama siguiente se muestra una implementación completa de carga equilibrada de alta disponibilidad de Azure MFA y sus componentes, junto con AD FS como referencia.

 ![Implementación de alta disponibilidad del Servidor Azure MFA](./media/howto-mfaserver-deploy-ha/mfa-ha-deployment.png)

Tenga en cuenta los siguientes elementos para el área numerada correspondiente del diagrama anterior.

1. Las dos instancias del Servidor Azure MFA (MFA1 y MFA2) tienen carga equilibrada (mfaapp.contoso.com) y están configuradas para usar un puerto estático (4443) para replicar la base de datos PhoneFactor.pfdata. El SDK del servicio web está instalado en cada una de las instancias del Servidor MFA para habilitar la comunicación a través del puerto TCP 443 con los servidores de AD FS. Las instancias del Servidor MFA se implementan en una configuración de carga equilibrada sin estado. Sin embargo, si desea utilizar la OTP a través de SMS, debe usar el equilibrio de carga con estado.
   ![Servidor Azure MFA: alta disponibilidad del servidor de aplicaciones](./media/howto-mfaserver-deploy-ha/mfaapp.png)

   > [!NOTE]
   > Dado que RPC usa puertos dinámicos, no se recomienda abrir los firewalls hasta el intervalo de puertos dinámicos que RPC puede usar. Si tiene un firewall **entre** los servidores de aplicaciones de MFA, debe configurar la instancia del Servidor MFA para comunicarse en un puerto estático para el tráfico de replicación entre servidores maestros y subordinados y abrir ese puerto en el firewall. Puede forzar el puerto estático mediante la creación de un valor del Registro DWORD en ```HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Positive Networks\PhoneFactor``` denominado ```Pfsvc_ncan_ip_tcp_port``` y establecer el valor en un puerto estático disponible. Las instancias subordinadas del Servidor MFA siempre inician las conexiones en la instancia maestra y el puerto estático solo es necesario en la instancia maestra, pero, como se puede promover una instancia secundaria para que actúe como maestra en cualquier momento, debe definir el puerto estático en todas las instancias de Servidor MFA.

2. Se equilibra la carga del servidor del Portal de usuarios y el servidor de aplicaciones móviles de MFA (MFA-UP-MAS1 y MFA-UP-MAS2) en una configuración **con estado** (mfa.contoso.com). Recuerde que las sesiones temporales son un requisito para el equilibrio de carga del servicio de aplicaciones móviles y el Portal de usuarios de MFA.
   ![Servidor Azure MFA: alta disponibilidad del servicio de aplicación móvil y del Portal de usuarios](./media/howto-mfaserver-deploy-ha/mfaportal.png)
3. Se equilibra la carga de la granja de servidores de AD FS y se publica en Internet a través de servidores proxy de AD FS de carga equilibrada en la red perimetral. Cada servidor de AD FS usa al agente de AD FS para comunicarse con las instancias del Servidor Azure MFA mediante una dirección URL única de carga equilibrada (mfaapp.contoso.com) a través del puerto TCP 443.

## <a name="next-steps"></a>Pasos siguientes

* [Instalación y configuración del Servidor Azure MFA](howto-mfaserver-deploy.md)
