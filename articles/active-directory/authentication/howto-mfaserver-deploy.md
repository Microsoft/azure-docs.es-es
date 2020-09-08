---
title: 'Introducción al Servidor Azure MFA: Azure Active Directory'
description: Introducción paso a paso del Servidor Azure MFA local
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
ms.openlocfilehash: bbe8b4176e33ff04ff8033af3b50cbdd0d882ae2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919580"
---
# <a name="getting-started-with-the-azure-multi-factor-authentication-server"></a>Introducción a Servidor Azure Multi-Factor Authentication

<center>

![Introducción al Servidor MFA local](./media/howto-mfaserver-deploy/server2.png)</center>

Esta página describe una nueva instalación del servidor y su configuración con una instancia loal de Active Directory. Si ya tiene instalado el servidor MFA y desea actualizarlo, consulte [Upgrade to the latest Azure Multi-Factor Authentication Server](howto-mfaserver-deploy-upgrade.md) (Actualización a la versión más reciente del Servidor Microsoft Azure Multi-Factor Authentication). Para más información sobre cómo instalar solo el servicio web, consulte [Implementación del servicio web móvil de la aplicación móvil del Servidor Azure Multi-Factor Authentication](howto-mfaserver-deploy-mobileapp.md).

> [!IMPORTANT]
> A partir del 1 de julio de 2019, Microsoft ya no ofrecerá el servidor MFA para implementaciones nuevas. Los clientes nuevos que quieran exigir la autenticación multifactor (MFA) durante los eventos de inicio de sesión deben usar Azure Multi-Factor Authentication basado en la nube.
>
> Para empezar a trabajar con MFA basado en la nube, consulte [Tutorial: Protección de eventos de inicio de sesión de usuario con Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Los clientes existentes que hayan activado el Servidor MFA antes del 1 de julio de 2019 podrán descargar la versión más reciente y las actualizaciones futuras, así como generar credenciales de activación como de costumbre.

## <a name="plan-your-deployment"></a>Planeamiento de la implementación

Antes de descargar el servidor Azure Multi-Factor Authentication, piense cuáles son sus requisitos de alta disponibilidad y carga. Use esta información para decidir cómo y dónde realizar la implementación.

Una buena directriz para saber la cantidad de memoria que se necesita es el número de usuarios que se espera autenticar de forma regular.

| Usuarios | RAM |
| ----- | --- |
| 1-10,000 | 4 GB |
| 10,001-50,000 | 8 GB |
| 50,001-100,000 | 12 GB |
| 100,000-200,001 | 16 GB |
| 200,001+ | 32 GB |

¿Necesita configurar varios servidores para lograr alta disponibilidad o un equilibrio de carga? Hay varias maneras de definir esta configuración con el Servidor Azure MFA. Cuando se instala el primer Servidor Azure MFA, pasa a ser el maestro. Los servidores adicionales pasan a ser los subordinados y sincronizan automáticamente los usuarios y la configuración con el maestro. A continuación, puede configurar un servidor principal y hacer que el resto actúen como copia de seguridad, o bien puede configurar el equilibrio de carga entre todos los servidores.

Aunque se desconecte un Servidor Azure MFA maestro, los servidores subordinados pueden procesar solicitudes de comprobación de dos pasos. Sin embargo, no puede agregar usuarios nuevos y los existentes no pueden actualizar su configuración hasta que el servidor maestro vuelva a estar conectado o se promocione un servidor subordinado.

### <a name="prepare-your-environment"></a>Preparación del entorno

Asegúrese de que el servidor que usa para Azure Multi-Factor Authentication cumple los requisitos siguientes:

| Requisitos del Servidor Azure Multi-Factor Authentication | Descripción |
|:--- |:--- |
| Hardware |<li>200 MB de espacio de disco duro</li><li>Procesador compatible con x32 o x64</li><li>1 GB o más de RAM</li> |
| Software |<li>Windows Server 2016</li><li>Windows Server 2012 R2</li><li>Windows Server 2012</li><li>Windows Server 2008/R2 (solo con [ESU](https://docs.microsoft.com/lifecycle/faq/extended-security-updates))</li><li>Windows 10</li><li>Windows 8.1, todas las ediciones</li><li>Windows 8, todas las ediciones</li><li>Windows 7, todas las ediciones (solo con [ESU](https://docs.microsoft.com/lifecycle/faq/extended-security-updates))</li><li>Microsoft .NET 4.0 Framework</li><li>IIS 7.0 o superior si está instalado el SDK de servicio web o el portal de usuario</li> |
| Permisos | Cuenta de administrador de dominio o administrador de empresa para registrar con Active Directory |

### <a name="azure-mfa-server-components"></a>Componentes del servidor Azure MFA

Hay tres componentes web que componen el servidor Azure MFA:

* El SDK del servicio web: permite la comunicación con los demás componentes y se instala en el servidor de aplicaciones de Azure MFA
* El Portal de usuarios: un sitio web de IIS que permite a los usuarios inscribirse en Azure Multi-Factor Authentication (MFA) y mantener sus cuentas.
* El servicio web de aplicación móvil: permite el uso de una aplicación móvil como Microsoft Authenticator para la verificación en dos pasos.

Los tres componentes se pueden instalar en el mismo servidor si este tiene conexión a Internet. Si es necesario separar los componentes, el SDK del servicio web se puede instalar en el servidor de aplicaciones Azure MFA y el Portal de usuarios y el servicio web de aplicación móvil en un servidor con conexión a Internet.

### <a name="azure-multi-factor-authentication-server-firewall-requirements"></a>Requisitos de firewall del Servidor Azure Multi-Factor Authentication

Cada servidor MFA debe ser capaz de comunicarse en el puerto 443 de salida a las siguientes direcciones:

* https://pfd.phonefactor.net
* https://pfd2.phonefactor.net
* https://css.phonefactor.net

Si los firewalls de salida están restringidos en el puerto 443, deberán abrirse los siguientes intervalos de direcciones IP:

| Subred IP | Máscara de red | Rango de direcciones IP |
|:---: |:---: |:---: |
| 134.170.116.0/25 |255.255.255.128 |134.170.116.1 – 134.170.116.126 |
| 134.170.165.0/25 |255.255.255.128 |134.170.165.1 – 134.170.165.126 |
| 70.37.154.128/25 |255.255.255.128 |70.37.154.129 – 70.37.154.254   |
| 52.251.8.48/28   | 255.255.255.240 | 52.251.8.48 - 52.251.8.63     |
| 52.247.73.160/28 | 255.255.255.240 | 52.247.73.160 - 52.247.73.175 |
| 52.159.5.240/28  | 255.255.255.240 | 52.159.5.240 - 52.159.5.255   |
| 52.159.7.16/28   | 255.255.255.240 | 52.159.7.16 - 52.159.7.31     |
| 52.250.84.176/28 | 255.255.255.240 | 52.250.84.176 - 52.250.84.191 |
| 52.250.85.96/28  | 255.255.255.240 | 52.250.85.96 - 52.250.85.111  |

Si no está usando la característica de confirmación de eventos y los usuarios no usan aplicaciones móviles para comprobar desde dispositivos de la red corporativa, solo necesita los siguientes intervalos:

| Subred IP | Máscara de red | Rango de direcciones IP |
|:---: |:---: |:---: |
| 134.170.116.72/29 |255.255.255.248 |134.170.116.72 – 134.170.116.79|
| 134.170.165.72/29 |255.255.255.248 |134.170.165.72 – 134.170.165.79|
| 70.37.154.200/29 |255.255.255.248  |70.37.154.201 – 70.37.154.206  |
| 52.251.8.48/28   | 255.255.255.240 | 52.251.8.48 - 52.251.8.63     |
| 52.247.73.160/28 | 255.255.255.240 | 52.247.73.160 - 52.247.73.175 |
| 52.159.5.240/28  | 255.255.255.240 | 52.159.5.240 - 52.159.5.255   |
| 52.159.7.16/28   | 255.255.255.240 | 52.159.7.16 - 52.159.7.31     |
| 52.250.84.176/28 | 255.255.255.240 | 52.250.84.176 - 52.250.84.191 |
| 52.250.85.96/28  | 255.255.255.240 | 52.250.85.96 - 52.250.85.111  |

## <a name="download-the-mfa-server"></a>Descarga del servidor MFA

Para descargar el Servidor Azure Multi-Factor Authentication desde Azure Portal, siga estos pasos:

> [!IMPORTANT]
> A partir del 1 de julio de 2019, Microsoft ya no ofrecerá el servidor MFA para implementaciones nuevas. Los nuevos clientes que quieran exigir la autenticación multifactor (MFA) a sus usuarios deberán usar Azure Multi-Factor Authentication basado en la nube.
>
> Para empezar a trabajar con MFA basado en la nube, consulte [Tutorial: Protección de eventos de inicio de sesión de usuario con Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Los clientes existentes que hayan activado el Servidor MFA antes del 1 de julio de 2019 podrán descargar la versión más reciente y las actualizaciones futuras, así como generar credenciales de activación como de costumbre. Los siguientes pasos solo funcionan si se trata de un cliente de Servidor MFA existente.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador.
2. Busque y seleccione *Azure Active Directory*. Seleccione **Seguridad** > **MFA**.
3. En **Administrar servidor de MFA**, seleccione **Configuración del servidor**.
4. Seleccione **Descargar** y siga las instrucciones que aparecen en la página de descarga para guardar el programa de instalación. 

   ![Descargar el Servidor MFA desde Azure Portal](./media/howto-mfaserver-deploy/downloadportal.png)

5. Mantenga esta página abierta ya que habrá que consultarla después de ejecutar el programa de instalación.

## <a name="install-and-configure-the-mfa-server"></a>Instalación y configuración del servidor MFA

Una vez descargado el servidor, ya se puede instalar y configurar. Asegúrese de que el servidor en el que va a instalarlo cumple los requisitos que se enumeran en la sección de planificación.

1. Haga doble clic en el archivo ejecutable.
2. En la pantalla Seleccionar carpeta de instalación, asegúrese de que la carpeta sea correcta y haga clic en **Siguiente**.
3. Una vez completada la instalación, haga clic en **Finalizar**. Se inicia el asistente para configuración.
4. En la pantalla de bienvenida del asistente para configuración, active **Omitir el uso del Asistente para configuración de autenticación** y haga clic en **Siguiente**. El asistente se cierra y el servidor se inicia.

   ![Omitir el uso del Asistente para configuración de autenticación](./media/howto-mfaserver-deploy/skip2.png)

5. De vuelta en la página desde la que se ha descargado el servidor, haga clic en el botón **Generar credenciales de activación**. Copie esta información en Servidor Azure Multi-Factor Authentication en los cuadros correspondientes y haga clic en **Activar**.

> [!NOTE]
> Los administradores globales son los únicos que pueden generar credenciales de activación en Azure Portal.

## <a name="send-users-an-email"></a>Enviar un correo electrónico a los usuarios

Para facilitar la implementación, permita que el servidor MFA se comunique con otros usuarios. El servidor MFA puede enviar un correo electrónico para informarles de que se les ha inscrito en la verificación en dos pasos.

El correo electrónico que envíe estará determinado por cómo haya configurado la verificación en dos pasos para los usuarios. Por ejemplo, si puede importar los números de teléfono del directorio de la empresa, el mensaje de correo electrónico deberá incluir los números de teléfono predeterminados para que los usuarios sepan qué pueden esperar. Si no los ha importado o los usuarios van a usar la aplicación móvil, envíeles un mensaje de correo electrónico que les indique que deben completar la inscripción de su cuenta. Incluya un hipervínculo al portal de usuarios de Azure Multi-factor Authentication en el correo electrónico.

El contenido del correo electrónico variará según el método de autenticación que se haya establecido para el usuario (llamada de teléfono, SMS o aplicación móvil). Por ejemplo, si el usuario debe usar un PIN para autenticarse, el correo electrónico le indica el PIN inicial que se ha establecido. Los usuarios deben cambiar su código PIN durante su primera autenticación.

### <a name="configure-email-and-email-templates"></a>Configuración del correo electrónico y las plantillas de correo electrónico

Haga clic en el icono de correo electrónico situado a la izquierda para configurar las opciones de envío de estos correos electrónicos. En esta página puede especificar la información de SMTP del servidor de correo electrónico y seleccionar la casilla **Enviar correos electrónicos a los usuarios** para enviar correo electrónico.

![Configuración del correo electrónico del servidor MFA](./media/howto-mfaserver-deploy/email1.png)

En la pestaña Contenido del mensaje de correo electrónico, verá las plantillas de correo electrónico que hay disponibles. Elija la plantilla más adecuada según cómo haya configurado la verificación en dos pasos para los usuarios.

![Plantillas de correo electrónico del Servidor MFA en la consola](./media/howto-mfaserver-deploy/email2.png)

## <a name="import-users-from-active-directory"></a>Importación de usuarios desde Active Directory

Ahora que está instalado el servidor, probablemente querrá agregar usuarios. Puede decidir crearlos manualmente, importar los usuarios desde Active Directory o configurar la sincronización automática con Active Directory.

### <a name="manual-import-from-active-directory"></a>Importación manual desde Active Directory

1. En Servidor Azure Multi-Factor Authentication, a la izquierda, seleccione **Usuarios**.
2. En la parte inferior, seleccione **Importar desde Active Directory**.
3. Ahora puede buscar usuarios individuales o buscar en el directorio de AD las unidades organizativas con usuarios en ellas. En este caso, se especifican las unidades organizativas de los usuarios.
4. Resalte todos los usuarios de la derecha y haga clic en **Importar**. Debe aparecer una ventana emergente que le indica que la operación se realizó correctamente. Cierre la ventana de importación.

   ![Importación de usuarios del Servidor MFA desde Active Directory](./media/howto-mfaserver-deploy/import2.png)

### <a name="automated-synchronization-with-active-directory"></a>Sincronización automática con Active Directory

1. En el Servidor Azure MFA, a la izquierda, seleccione **Integración de directorios**.
2. Vaya a la pestaña **Sincronización**.
3. En la parte inferior, elija **Agregar**.
4. En el cuadro de diálogo **Agregar elemento de sincronización** que aparece, elija el dominio, la unidad organizativa **o** grupo de seguridad, la configuración, y los valores predeterminados de métodos e idiomas para esta tarea de sincronización, y haga clic en **Agregar**.
5. Active la casilla denominada **Habilitar sincronización con Active Directory** y elija un **intervalo de sincronización** entre un minuto y 24 horas.

## <a name="how-the-azure-multi-factor-authentication-server-handles-user-data"></a>¿Cómo controla el Servidor Azure Multi-Factor Authentication los datos de usuario?

Cuando se usa Servidor Multi-Factor Authentication (MFA) local, los datos de los usuarios se almacenan en los servidores locales. Los datos de usuario persistentes no se almacenan en la nube. Cuando el usuario realiza una verificación en dos pasos, Servidor MFA envía los datos al servicio en la nube Azure MFA para realizar la verificación. Cuando estas solicitudes de autenticación se envían al servicio en la nube, en la solicitud y los registros se incluyen los siguientes campos para que estén disponibles en los informes de uso/autenticación del cliente. Algunos de los campos son opcionales y se pueden habilitar o deshabilitar en Servidor Multi-Factor Authentication. La comunicación desde Servidor MFA al servicio en la nube MFA usa SSL/TLS en el puerto 443 de salida. Estos campos son:

* Id. exclusivo: nombre de usuario o Id. interno de Servidor MFA
* Nombre y apellidos (opcional)
* Dirección de correo electrónico (opcional)
* Número de teléfono: al realizar una llamada de voz o autenticación por SMS
* Token de dispositivo: al realizar la autenticación de una aplicación móvil
* Modo de autenticación
* Resultado de autenticación
* Nombre de Servidor MFA
* IP de Servidor MFA
* IP de cliente: si está disponible

Además de los campos anteriores, el resultado (éxito o denegación) de la verificación y el motivo de las denegaciones también se almacenan con los datos de autenticación y están disponibles en informes de uso y autenticación.

> [!IMPORTANT]
> A partir de marzo de 2019, las opciones de llamada de teléfono no estarán disponibles para los usuarios del Servidor MFA en inquilinos de Azure AD gratis o de evaluación. Los mensajes SMS no se ven afectados por este cambio. Las llamadas de teléfono seguirán estando disponibles para los usuarios de inquilinos de Azure AD de pago. Este cambio solo afecta a los inquilinos de Azure AD gratis o de evaluación.

## <a name="back-up-and-restore-azure-mfa-server"></a>Copia de seguridad y restauración del servidor Azure MFA

Asegurarse de que tiene una copia de seguridad actualizada es un paso importante que debe realizar con cualquier sistema.

Para hacer una copia de seguridad del servidor Azure MFA, asegúrese de que tiene una copia de la carpeta **C:\Archivos de programa\Multi-Factor Authentication Server\Data** incluido el archivo **PhoneFactor.pfdata**. 

En caso de que sea necesaria una restauración, realice los pasos siguientes:

1. Vuelva a instalar el servidor Azure MFA en un servidor nuevo.
2. Active el nuevo servidor Azure MFA.
3. Detenga el servicio **MultiFactorAuth**.
4. Sobrescriba el archivo **PhoneFactor.pfdata** con el de la copia de seguridad.
5. Inicie el servicio **MultiFactorAuth**.

El nuevo servidor está ahora en funcionamiento con los datos de usuario y la configuración originales procedentes de la copia de seguridad.

## <a name="managing-the-tlsssl-protocols-and-cipher-suites"></a>Administración de los protocolos TLS/SSL y conjuntos de cifrado

Una vez que ha actualizado a la versión 8.x o superior de Servidor de MFA, o la ha instalado, se recomienda deshabilitar o quitar los conjuntos de cifrado más antiguos o débiles a menos que los exija la organización. Se puede encontrar información sobre cómo completar esta tarea en el artículo sobre [Administración de conjuntos de cifrado y protocolos SSL/TLS de AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs).

## <a name="next-steps"></a>Pasos siguientes

- Instalación y configuración del [Portal de usuarios](howto-mfaserver-deploy-userportal.md) de autoservicio del usuario.
- Instale y configure el Servidor Azure MFA con [Servicios de federación de Active Directory](multi-factor-authentication-get-started-adfs.md), [Autenticación RADIUS](howto-mfaserver-dir-radius.md) o [Autenticación LDAP](howto-mfaserver-dir-ldap.md).
- Instalación y configuración de [Puerta de enlace de Escritorio remoto y Servidor Azure Multi-Factor Authentication con RADIUS](howto-mfaserver-nps-rdg.md).
- [Implementación del servicio web de aplicaciones móviles del Servidor Azure Multi-Factor Authentication](howto-mfaserver-deploy-mobileapp.md).
- [Escenarios avanzados con Azure Multi-Factor Authentication y VPN de terceros](howto-mfaserver-nps-vpn.md).