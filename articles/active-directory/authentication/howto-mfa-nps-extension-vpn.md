---
title: 'VPN con Azure MFA con la extensión de NPS: Azure Active Directory'
description: Integración de la infraestructura de VPN con Azure MFA utilizando la extensión Servidor de directivas de redes para Microsoft Azure.
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
ms.openlocfilehash: 13ed87903845d9f8295e56f187b643d73fbfb04e
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717887"
---
# <a name="integrate-your-vpn-infrastructure-with-azure-mfa-by-using-the-network-policy-server-extension-for-azure"></a>Integración de la infraestructura de VPN con Azure MFA utilizando la extensión Servidor de directivas de redes para Azure

La extensión Servidor de directivas de redes (NPS) para Azure permite a las organizaciones proteger la autenticación de cliente del Servicio de autenticación remota telefónica de usuario (RADIUS) utilizando la autenticación basada en la nube [Azure Multi-Factor Authentication (MFA)](howto-mfaserver-nps-rdg.md), que proporciona verificación en dos pasos.

En este artículo se proporcionan instrucciones para la integración de la infraestructura NPS con Azure MFA con la extensión NPS para Azure. Este proceso permite la verificación en dos pasos segura para los usuarios que intentan conectarse a la red mediante una VPN.

El rol Servicios de acceso y directivas de redes da a las organizaciones la posibilidad de hacer lo siguiente:

* Asignar una ubicación central para la administración y el control de las solicitudes de red para especificar:

  * Quién puede conectarse

  * A qué horas del día se permiten las conexiones

  * La duración de las conexiones

  * El nivel de seguridad que los clientes deben usar para conectarse

    En lugar de especificar las directivas en cada servidor VPN o de puerta de enlace de Escritorio remoto, debe hacerlo una vez que se encuentren en una ubicación central. El protocolo RADIUS se usa para proporcionar autenticación, autorización y contabilización de cuentas (AAA) centralizadas.

* Establecer y aplicar directivas de mantenimiento de cliente de Protección de acceso a redes (NAP) que determinan si los dispositivos tienen acceso restringido o sin restricciones a los recursos de la red.

* Proporcionar una forma de aplicar la autenticación y autorización para el acceso a puntos de acceso inalámbricos 802.1x y conmutadores Ethernet.
  Vea [Servidor de directivas de redes](/windows-server/networking/technologies/nps/nps-top) para más información.

Para mejorar la seguridad y proporcionar un nivel elevado de cumplimiento, las organizaciones pueden integrar NPS con Azure Multi-Factor Authentication para asegurarse de que los usuarios utilizan la verificación en dos pasos para conectarse al puerto virtual en el servidor VPN. Para tener acceso, los usuarios deben proporcionar su combinación de nombre de usuario y contraseña y otra información que tengan bajo su control. Esta información debe ser de confianza y difícil de duplicar. Puede incluir un número de teléfono móvil, un número de teléfono fijo o una aplicación en un dispositivo móvil.

Antes de la disponibilidad de la extensión NPS para Azure, los clientes que querían implementar la verificación en dos pasos para entornos integrados de NPS y MFA tenían que configurar y mantener un servidor independiente de MFA en un entorno local tal. Este tipo de autenticación la ofrecen la pPuerta de enlace de Escritorio remoto y el Servidor Azure Multi-Factor Authentication con RADIUS.

Con la extensión NPS para Azure, las organizaciones pueden proteger la autenticación de clientes RADIUS con la implementación de una solución MFA basada en el entorno local o de una solución MFA basada en la nube.

## <a name="authentication-flow"></a>Flujo de autenticación

Cuando los usuarios se conectan a un puerto virtual en un servidor VPN, primero deben autenticarse mediante una variedad de protocolos. Los protocolos permiten usar una combinación de nombre de usuario y contraseña y métodos de autenticación basada en certificados.

Además de la autenticación y verificación de identidad, los usuarios deben tener los permisos adecuados de acceso telefónico. En implementaciones sencillas, los permisos de acceso telefónico que permiten tener acceso se establecen directamente en los objetos de usuario de Active Directory.

![Pestaña Marcado en las propiedades de usuario para equipos y usuarios de Active Directory](./media/howto-mfa-nps-extension-vpn/image1.png)

En implementaciones sencillas, cada servidor VPN concede o deniega el acceso en función de las directivas definidas en cada servidor VPN local.

En implementaciones más grandes y escalables, las directivas que conceden o deniegan el acceso a la VPN están centralizadas en servidores RADIUS. En estos casos, el servidor VPN actúa como un servidor de acceso (cliente RADIUS) que reenvía las solicitudes de conexión y mensajes de la cuenta a un servidor RADIUS. Para conectarse al puerto virtual en el servidor VPN, los usuarios deben autenticarse y cumplir las condiciones definidas de forma centralizada en servidores RADIUS.

Cuando la extensión NPS para Azure está integrada con NPS, el flujo de una autenticación correcta es el siguiente:

1. El servidor VPN recibe una solicitud de autenticación de un usuario de VPN que incluye el nombre de usuario y la contraseña para conectarse a un recurso, como una sesión de Escritorio remoto.
2. Actuando como un cliente RADIUS, el servidor VPN convierte la solicitud en un mensaje de *solicitud de acceso* RADIUS y lo envía (con una contraseña cifrada) al servidor RADIUS donde está instalada la extensión NPS.
3. La combinación de nombre de usuario y contraseña se verifica en Active Directory. Si el nombre de usuario o la contraseña son incorrectos, el servidor RADIUS envía un mensaje de *rechazo de acceso*.
4. Si se cumplen todas las condiciones especificadas en la solicitud de conexión NPS y las directivas de red (por ejemplo, la hora del día o restricciones por pertenencia a un grupo), la extensión NPS desencadena una solicitud de autenticación secundaria con Azure Multi-Factor Authentication.
5. Azure Multi-Factor Authentication se comunica con Azure Active Directory, recupera los detalles del usuario y realiza la autenticación secundaria con el método configurado por el usuario (llamada al móvil, mensaje de texto o aplicación móvil).
6. Cuando el desafío de MFA es correcto, Azure Multi-Factor Authentication comunica el resultado a la extensión NPS.
7. Después de que el intento de conexión se autentica y autoriza, el servidor NPS donde está instalada la extensión envía un mensaje de *aceptación de acceso* RADIUS al servidor VPN (cliente RADIUS).
8. Se concede acceso al usuario al puerto virtual en el servidor VPN y se establece un túnel VPN cifrado.

## <a name="prerequisites"></a>Requisitos previos

En esta sección se detallan los requisitos previos que hay que cumplir antes de integrar MFA con la VPN. Antes de comenzar, debe cumplir los siguientes requisitos previos:

* Infraestructura de VPN
* Rol Servicios de acceso y directivas de redes
* Licencia de Azure Multi-Factor Authentication
* Software de Windows Server
* Bibliotecas
* Azure Active Directory (Azure AD) sincronizado con Active Directory local
* Identificador de GUID de Azure Active Directory

### <a name="vpn-infrastructure"></a>Infraestructura de VPN

En este artículo se da por supuesto que tiene una infraestructura de VPN en funcionamiento con Microsoft Windows Server 2016 y que el servidor VPN actualmente no está configurado para reenviar solicitudes de conexión a un servidor RADIUS. En este artículo, se va a configurar la infraestructura de VPN para usar un servidor RADIUS central.

Si no tiene una infraestructura de VPN en funcionamiento, puede crear una rápidamente siguiendo las instrucciones indicadas en numerosos tutoriales de instalación de VPN que puede encontrar en sitios de terceros y de Microsoft.

### <a name="the-network-policy-and-access-services-role"></a>El rol Servicios de acceso y directivas de redes

El rol Servicios de acceso y directivas de redes proporciona la funcionalidad de cliente y servidor RADIUS. En este artículo se da por supuesto que ha instalado el rol Servicios de acceso y directivas de redes en un servidor miembro o en un controlador de dominio en su entorno. En esta guía configurará RADIUS para una configuración de VPN. Instale el rol Servicios de acceso y directivas de redes en un servidor *distinto del* servidor VPN.

Para obtener información sobre cómo instalar el servicio de rol Servicios de acceso y directivas de redes en Windows Server 2012 o posterior, vea [Install a NAP Health Policy Server](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd296890(v=ws.10)) (Instalación de un servidor de directivas de mantenimiento de NAP). NAP está en desuso en Windows Server 2016. Para obtener una descripción de las prácticas recomendadas para NPS, incluida la recomendación de instalar NPS en un controlador de dominio, vea [Best practices for NPS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771746(v=ws.10)) (Procedimientos recomendados para NPS).

### <a name="azure-mfa-license"></a>Licencia de Azure MFA

Se requiere una licencia para Azure Multi-Factor Authentication, que está disponible a través de una licencia independiente de Azure AD Premium, Enterprise Mobility + Security o Multi-Factor Authentication. Las licencias basadas en el consumo para Azure MFA, como las licencias por usuario o por autenticación no son compatibles con la extensión de NPS. Para más información, consulte [Cómo obtener Azure Multi-Factor Authentication](concept-mfa-licensing.md). Para realizar pruebas, puede usar una suscripción de evaluación.

### <a name="windows-server-software"></a>Software de Windows Server

La extensión NPS requiere Windows Server 2008 R2 SP1 o posterior, con el rol Servicios de acceso y directivas de redes instalado. Todos los pasos de esta guía se han realizado con Windows Server 2016.

### <a name="libraries"></a>Bibliotecas

Las siguientes bibliotecas se instalan automáticamente con la extensión NPS:

-    [Paquetes redistribuibles de Visual C++ para Visual Studio 2013 (X64)](https://www.microsoft.com/download/details.aspx?id=40784)
-    [Módulo Microsoft Azure Active Directory para Windows PowerShell versión1.1.166.0](https://connect.microsoft.com/site1164/Downloads/DownloadDetails.aspx?DownloadID=59185)

El módulo Microsoft Azure Active Directory para PowerShell todavía no está presente, pero se instala a través de un script de configuración que se ejecuta como parte del proceso de instalación. No es necesario instalar el módulo con antelación si aún no está instalado.

### <a name="azure-active-directory-synced-with-on-premises-active-directory"></a>Azure Active Directory sincronizado con Active Directory local

Para usar la extensión NPS, los usuarios locales deben estar sincronizados con Azure Active Directory y habilitados para MFA. En esta guía se da por supuesto que los usuarios locales están sincronizados con Azure Active Directory mediante Azure AD Connect. A continuación, se proporcionan instrucciones para habilitar a los usuarios para usar MFA.

Para obtener información sobre Azure AD Connect, vea [Integración de los directorios locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

### <a name="azure-active-directory-guid-id"></a>Identificador de GUID de Azure Active Directory

Para instalar la extensión NPS, debe conocer el GUID de Azure Active Directory. En la siguiente sección se proporcionan instrucciones para buscar el GUID de Azure Active Directory.

## <a name="configure-radius-for-vpn-connections"></a>Configuración de RADIUS para conexiones VPN

Si ha instalado el rol NPS en un servidor miembro, debe configurarlo para autenticar y autorizar el cliente VPN que solicite conexiones VPN. 

En esta sección se supone que ha instalado el rol Servicios de acceso y directivas de redes pero no lo ha configurado para usarlo en la infraestructura.

> [!NOTE]
> Si ya tiene un servidor VPN en funcionamiento que usa un servidor RADIUS centralizado para la autenticación, puede omitir esta sección.
>

### <a name="register-server-in-active-directory"></a>Registro del servidor en Active Directory

Para que funcione correctamente en este escenario, el servidor NPS debe estar registrado en Active Directory.

1. Abra el Administrador del servidor.

2. En el Administrador del servidor, seleccione **Herramientas** y después **Servidor de directivas de redes**.

3. En la consola del Servidor de directivas de redes, haga clic con el botón derecho en **NPS (Local)** y después seleccione **Registrar servidor en Active Directory**. Seleccione **Aceptar** dos veces.

    ![Opción de menú Registro del servidor en Active Directory](./media/howto-mfa-nps-extension-vpn/image2.png)

4. Deje la consola abierta para el siguiente procedimiento.

### <a name="use-wizard-to-configure-the-radius-server"></a>Uso del asistente para configurar el servidor RADIUS

Puede usar la opción de configuración estándar (basada en asistente) o la opción avanzada para configurar el servidor RADIUS. En esta sección se supone que usa la opción de configuración estándar con el asistente.

1. En la consola del Servidor de directivas de redes, seleccione **NPS (Local)** .

2. En **Configuración estándar**, seleccione **Servidor RADIUS para conexiones VPN o de acceso telefónico** y después seleccione **Configurar VPN o acceso telefónico**.

    ![Configuración del servidor RADIUS para conexiones de acceso telefónico o VPN](./media/howto-mfa-nps-extension-vpn/image3.png)

3. En la ventana **Seleccionar tipo de conexiones de red privada virtual o de acceso telefónico**, seleccione **Virtual Private Network Connections** (Conexiones de red privada virtual) y **Siguiente**.

    ![Configuración de las conexiones de la red privada virtual](./media/howto-mfa-nps-extension-vpn/image4.png)

4. En la página **Especificar servidor VPN o de acceso telefónico**, haga clic en **Agregar**.

5. En la ventana **Nuevo cliente RADIUS**, proporcione un nombre descriptivo, escriba el nombre para resolver o la dirección IP del servidor VPN y escriba una contraseña secreta compartida. Elija una contraseña secreta compartida larga y compleja. Anótela, porque la necesitará en esta sección.

    ![Creación de una nueva ventana de conmutador compatible con 802.1X](./media/howto-mfa-nps-extension-vpn/image5.png)

6. Seleccione **Aceptar** y después **Siguiente**.

7. En la ventana **Configurar métodos de autenticación**, acepte la selección predeterminada **Autenticación cifrada de Microsoft versión 2 (MS-CHAP v2)** o elija otra opción y seleccione **Siguiente**.

    > [!NOTE]
    > Si configura el protocolo de autenticación extensible (EAP), debe utilizar el protocolo de autenticación por desafío mutuo de Microsoft (CHAPv2) o el protocolo de autenticación extensible protegido (PEAP). No se admite ningún otro tipo de EAP.

8. En la ventana **Especificar grupos de usuarios**, seleccione **Agregar** y después elija un grupo apropiado. Si no existe ningún grupo, deje la selección en blanco para conceder acceso a todos los usuarios.

    ![Especificación en la ventana Grupos de usuarios para permitir o denegar el acceso](./media/howto-mfa-nps-extension-vpn/image7.png)

9. Seleccione **Next** (Siguiente).

10. En la ventana **Especificar filtros IP**, seleccione **Siguiente**.

11. En la ventana **Especificar configuración de cifrado**, acepte la configuración predeterminada y después seleccione **Siguiente**.

    ![Ventana Especificar configuración de cifrado](./media/howto-mfa-nps-extension-vpn/image8.png)

12. En la ventana **Especificar un nombre de dominio kerberos**, deje el nombre en blanco, acepte la configuración predeterminada y después seleccione **Siguiente**.

    ![Ventana Especificar un nombre de dominio kerberos](./media/howto-mfa-nps-extension-vpn/image9.png)

13. En la ventana **Finalización de nuevas conexiones de acceso telefónico o red privada virtual y clientes RADIUS**, seleccione **Finalizar**.

    ![Ventana Configuración completa](./media/howto-mfa-nps-extension-vpn/image10.png)

### <a name="verify-the-radius-configuration"></a>Verificación de la configuración de RADIUS

En esta sección se detalla la configuración creada con el asistente.

1. En el Servidor de directivas de redes, en la consola NPS (Local), expanda **Clientes RADIUS** y seleccione **Clientes RADIUS**.

2. En el panel de detalles, haga clic con el botón derecho en el cliente RADIUS que ha creado y después seleccione **Propiedades**. Las propiedades del cliente RADIUS (el servidor VPN) deben ser similares a las que se muestran a continuación:

    ![Compruebe las propiedades VPN y la configuración](./media/howto-mfa-nps-extension-vpn/image11.png)

3. Seleccione **Cancelar**.

4. En el Servidor de directivas de redes, en la consola NPS (Local), expanda **Directivas** y después seleccione **Directivas de solicitud de conexión**. La directiva Conexiones VPN aparece tal como se muestra en la siguiente imagen:

    ![Directiva de solicitud de conexión que muestra la directiva de conexión de VPN](./media/howto-mfa-nps-extension-vpn/image12.png)

5. En **Directivas**, seleccione **Directivas de red**. Debería ver una directiva de conexiones de red privada virtual (VPN), parecida a la directiva que aparece en la imagen siguiente:

    ![Directivas de red que muestran la directiva de conexiones de la red privada virtual](./media/howto-mfa-nps-extension-vpn/image13.png)

## <a name="configure-your-vpn-server-to-use-radius-authentication"></a>Configuración del servidor VPN para que utilice la autenticación RADIUS

En esta sección, configurará el servidor VPN para utilizar la autenticación RADIUS. En estas instrucciones se da por supuesto que tiene una configuración de servidor VPN en funcionamiento, pero no se ha configurado para utilizar la autenticación RADIUS. Después de configurar el servidor VPN, confirme que la configuración funciona según lo previsto.

> [!NOTE]
> Si ya tiene una configuración de servidor VPN en funcionamiento que usa autenticación RADIUS, puede omitir esta sección.
>

### <a name="configure-authentication-provider"></a>Configuración del proveedor de autenticación

1. En el servidor VPN, abra el Administrador del servidor.

2. En el Administrador del servidor, seleccione **Herramientas** y después **Enrutamiento y acceso remoto**.

3. En la ventana **Enrutamiento y acceso remoto**, haga clic con el botón derecho en **\<server name> (local)** y después seleccione **Propiedades**.

4. En la ventana de propiedades de **\<server name> (local)** , seleccione la pestaña **Seguridad**.

5. En la pestaña **Seguridad**, en **Proveedor de autenticación**, seleccione **Autenticación RADIUS** y después **Configurar**.

    ![Configuración del proveedor de autenticación de RADIUS](./media/howto-mfa-nps-extension-vpn/image15.png)

6. En la ventana **Autenticación RADIUS**, seleccione **Agregar**.

7. En la ventana **Agregar servidor RADIUS**, haga lo siguiente:

    a. En el cuadro **Nombre del servidor**, escriba el nombre o la dirección IP del servidor RADIUS configurado en la sección anterior.

    b. En **Secreto compartido**, seleccione **Cambiar** y agregue la contraseña secreta compartida que creó y anotó anteriormente.

    c. En el cuadro **Tiempo de espera (segundos)** , escriba el valor **30**.  
    El valor de tiempo es necesario para dar tiempo suficiente para completar el segundo factor de autenticación. Algunas VPN o regiones requieren una configuración de tiempo de espera superior a 30 segundos para impedir que los usuarios reciban varias llamadas telefónicas. Si los usuarios experimentan este problema, aumente el valor de **Tiempo de espera (segundos)** en incrementos de 30 segundos hasta que no se reproduzca el problema.

    ![Ventana Agregar servidor de RADIUS que configura el tiempo de espera](./media/howto-mfa-nps-extension-vpn/image16.png) 

8. Seleccione **Aceptar**.

### <a name="test-vpn-connectivity"></a>Probar la conectividad VPN

En esta sección, confirmará que el cliente VPN se autentica y es autorizado por el servidor RADIUS cuando intenta conectarse al puerto virtual de VPN. En estas instrucciones se supone que usa Windows 10 como un cliente VPN.

> [!NOTE]
> Si ya tiene configurado un cliente VPN para conectarse al servidor VPN y ha guardado la configuración, puede omitir los pasos descritos para configurar y guardar un objeto de conexión VPN.
>

1. En el equipo del cliente VPN, seleccione el botón **Inicio** y después seleccione el botón **Configuración**.

2. En la ventana **Configuración de Windows**, seleccione **Red e Internet**.

3. Seleccione **VPN**.

4. Seleccione **Agregar una conexión VPN**.

5. En la ventana **Agregar una conexión VPN**, en el cuadro **Proveedor de VPN**, seleccione **Windows (integrado)** , complete los campos restantes según corresponda y después seleccione **Guardar**.

    ![Ventana "Agregar una conexión VPN"](./media/howto-mfa-nps-extension-vpn/image17.png)

6. Vaya a **Panel de control** y después seleccione **Centro de redes y recursos compartidos**.

7. Seleccione **Cambiar configuración del adaptador**.

    ![Centro de redes y recursos compartidos: Cambiar configuración del adaptador](./media/howto-mfa-nps-extension-vpn/image18.png)

8. Haga clic con el botón derecho en la conexión de red VPN y después seleccione **Propiedades**.

9. En la ventana de propiedades de VPN, seleccione la pestaña **Seguridad**.

10. En la pestaña **Seguridad**, asegúrese de que solo está seleccionado **Microsoft CHAP versión 2 (MS-CHAP v2)** y después seleccione **Aceptar**.

    ![Opción "Permitir estos protocolos"](./media/howto-mfa-nps-extension-vpn/image20.png)

11. Haga clic con el botón derecho en la conexión VPN y después seleccione **Conectar**.

12. En la ventana **Configuración**, seleccione **Conectar**.  
    Aparece una conexión correcta en el registro de seguridad en el servidor RADIUS con el identificador de evento 6272, como se muestra a continuación:

    ![Ventana Propiedades de evento que muestra una conexión correcta](./media/howto-mfa-nps-extension-vpn/image21.png)

## <a name="troubleshooting-radius"></a>Solución de problemas de RADIUS

Se supone que la configuración de VPN funcionaba correctamente antes de configurar el servidor VPN para que utilice un servidor RADIUS centralizado para la autenticación y autorización. Si la configuración funcionaba, es probable que el problema se deba a una configuración incorrecta del servidor RADIUS o al uso de un nombre de usuario o contraseña no válidos. Por ejemplo, si usa el sufijo UPN alternativo en el nombre de usuario, el intento de inicio de sesión podría producir errores. Utilice el mismo nombre de cuenta para obtener los mejores resultados.

Para solucionar estos problemas, un lugar ideal para comenzar es examinar los registros de eventos de seguridad en el servidor RADIUS. Para ahorrar tiempo buscando eventos, puede utilizar la vista personalizada basada en roles Servidor de directivas de red y acceso en el Visor de eventos, como se muestra a continuación. El "identificador de evento 6273" indica eventos en los que el servidor NPS deniega el acceso a un usuario.

![Visor de eventos que muestra eventos de NPAS](./media/howto-mfa-nps-extension-vpn/image22.png)

## <a name="configure-multi-factor-authentication"></a>Configuración de Multi-Factor Authentication

Para obtener ayuda para configurar los usuarios para Multi-Factor Authentication, consulte los artículos [Planear una implementación de Azure Multi-Factor Authentication basada en la nube](howto-mfa-getstarted.md#create-conditional-access-policy) y [Configuración de mi cuenta para la verificación en dos pasos](../user-help/multi-factor-authentication-end-user-first-time.md).

## <a name="install-and-configure-the-nps-extension"></a>Instalación y configuración de la extensión NPS

Esta sección proporciona instrucciones para configurar la VPN para usar MFA para la autenticación de cliente con el servidor VPN.

> [!NOTE]
> La clave del registro REQUIRE_USER_MATCH distingue mayúsculas de minúsculas. Todos los valores se deben expresar en mayúsculas.
>

Una vez que instale y configure la extensión NPS, toda la autenticación de cliente basada en RADIUS procesada por este servidor debe usar MFA. Si todos los usuarios VPN no están inscritos en Azure Multi-Factor Authentication, puede realizar una de las siguientes acciones:

* Configurar otro servidor RADIUS para autenticar a usuarios que no están configurados para usar MFA.

* Crear una entrada del Registro que permite a los usuarios proporcionar un segundo factor de autenticación si están inscritos en Azure Multi-Factor Authentication.

Cree un nuevo valor de cadena denominado _REQUIRE_USER_MATCH in HKLM\SOFTWARE\Microsoft\AzureMfa_ y establezca el valor como *TRUE* o *FALSE*.

![Configuración Requerir coincidencia de usuario](./media/howto-mfa-nps-extension-vpn/image34.png)

Si el valor se establece en *TRUE* o está en blanco, todas las solicitudes de autenticación están sujetas a un desafío MFA. Si el valor se establece en *FALSE*, se emiten desafíos MFA únicamente a los usuarios que están inscritos en Azure Multi-Factor Autenticación. Use solo el valor *FALSE* en las pruebas o en entornos de producción durante un período de incorporación.



### <a name="obtain-the-azure-active-directory-tenant-id"></a>Obtención del identificador de inquilino de Azure Active Directory

Como parte de la configuración de la extensión NPS, debe proporcionar las credenciales de administrador y el identificador del inquilino de Azure AD. Para obtener el identificador de inquilino, complete los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global del inquilino de Azure.
1. En el menú de Azure Portal, seleccione **Azure Active Directory** o busque y seleccione **Azure Active Directory** desde cualquier página.
1. En la página **Información general**, se muestra la *información del inquilino*. Junto al *identificador de inquilino*, seleccione el icono **Copiar**, tal y como se muestra en la siguiente captura de pantalla de ejemplo:

   ![Obtención del identificador de inquilino en Azure Portal](./media/howto-mfa-nps-extension-vpn/azure-active-directory-tenant-id-portal.png)

### <a name="install-the-nps-extension"></a>Instalación de la extensión de NPS

La extensión NPS debe instalarse en un servidor que tenga el rol Servicios de acceso y directivas de redes instalado y que funcione como servidor RADIUS en el diseño. *No* instale la extensión NPS en el servidor VPN.

1. Descargue la extensión NPS desde el [Centro de descarga de Microsoft](https://aka.ms/npsmfa).

2. Copie el archivo ejecutable de instalación (*NpsExtnForAzureMfaInstaller.exe*) en el servidor NPS.

3. En el servidor NPS, haga doble clic en **NpsExtnForAzureMfaInstaller.exe** y, si se le solicita, seleccione **Ejecutar**.

4. En la ventana **Extensión NPS para Azure MFA**, revise los términos de licencia de software, seleccione la casilla **Acepto los términos y condiciones de licencia** y después seleccione **Instalar**.

    ![Ventana "Extensión NPS para Azure MFA"](./media/howto-mfa-nps-extension-vpn/image36.png)

5. En la ventana de instalación de **Extensión NPS para Azure MFA**, seleccione **Cerrar**.  

    ![Ventana de confirmación "La instalación se realizó correctamente"](./media/howto-mfa-nps-extension-vpn/image37.png)

### <a name="configure-certificates-for-use-with-the-nps-extension-by-using-a-powershell-script"></a>Configuración de los certificados para su uso con la extensión NPS mediante un script de PowerShell

Configure los certificados para su uso con la extensión NPS para garantizar la seguridad de las comunicaciones. Los componentes de NPS incluyen un script de Windows PowerShell que configura un certificado autofirmado para su uso con NPS.

Este script realiza las acciones siguientes:

* Crea un certificado autofirmado.
* Asocia la clave pública del certificado a la entidad de servicio en Azure AD.
* Almacena el certificado en el almacén del equipo local.
* Concede acceso a la clave privada del certificado al usuario de red.
* Reinicia el servicio NPS.

Si desea utilizar sus propios certificados, debe asociar la clave pública de su certificado con la entidad de servicio en Azure AD, etc.

Para usar el script, indique a la extensión sus credenciales de administrador de Azure Active Directory y el identificador del inquilino de Azure Active Directory que copió anteriormente. La cuenta debe estar en el mismo inquilino de Azure AD para el que desea habilitar la extensión. Ejecute el script en cada servidor NPS donde instaló la extensión NPS.

1. Ejecute Windows PowerShell como administrador.

2. En el símbolo del sistema de PowerShell, escriba **cd "c:\Program Files\Microsoft\AzureMfa\Config"** y presione ENTRAR.

3. En la siguiente ventana del símbolo del sistema, escriba **.\AzureMfaNpsExtnConfigSetup.ps1** y presione ENTRAR. El script comprueba si está instalado el módulo de PowerShell de Azure AD. Si no está instalado, el script instala el módulo.

    ![Ejecutar el script de configuración AzureMfsNpsExtnConfigSetup.ps1](./media/howto-mfa-nps-extension-vpn/image38.png)

    Si recibe un error de seguridad debido a TLS, habilite TLS 1.2 con el comando `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12` desde el símbolo del sistema de PowerShell.
    
    Una vez que el script verifica la instalación del módulo de PowerShell, muestra la ventana del módulo de PowerShell de Azure Active Directory.

4. Escriba sus credenciales de administrador de Azure AD y después seleccione **Iniciar sesión**.

    ![Autenticación en Azure AD PowerShell](./media/howto-mfa-nps-extension-vpn/image39.png)

5. En el símbolo del sistema, pegue el identificador del inquilino que copió anteriormente y después presione ENTRAR.

    ![Escriba el identificador de inquilino de Azure AD copiado antes](./media/howto-mfa-nps-extension-vpn/image40.png)

    El script crea un certificado autofirmado y realiza otros cambios en la configuración. La salida debe ser similar a la que se muestra en la imagen siguiente:

    ![Ventana de PowerShell que muestra un certificado autofirmado](./media/howto-mfa-nps-extension-vpn/image41.png)

6. Reinicie el servidor.

### <a name="verify-the-configuration"></a>Comprobar la configuración

Para verificar la configuración, debe establecer una conexión VPN nueva con el servidor VPN. Después de escribir correctamente las credenciales para la autenticación principal, la conexión VPN espera a que la autenticación secundaria sea correcta para establecer la conexión, como se muestra a continuación.

![Ventana de VPN en la configuración de Windows](./media/howto-mfa-nps-extension-vpn/image42.png)

Si se autentica correctamente con el método de verificación secundario configurado previamente en Azure MFA, se conecta al recurso. Sin embargo, si la autenticación secundaria no se realiza correctamente, se deniega el acceso al recurso.

En el ejemplo siguiente, la aplicación Microsoft Authenticator en un dispositivo Windows Phone proporciona la autenticación secundaria:

![Aviso de MFA de ejemplo en Windows Phone](./media/howto-mfa-nps-extension-vpn/image43.png)

Una vez que se haya autenticado correctamente con el método secundario, se le concede acceso al puerto virtual en el servidor VPN. Como ha sido necesario usar un método de autenticación secundario con una aplicación móvil en un dispositivo de confianza, el proceso de inicio de sesión es más seguro que utilizando solo una combinación de nombre de usuario y contraseña.

### <a name="view-event-viewer-logs-for-successful-sign-in-events"></a>Visualización de los registros del Visor de eventos para eventos de inicio de sesión correcto

Para ver los eventos de inicio de sesión correctos en los registros del Visor de eventos de Windows, escriba el siguiente comando de PowerShell para consultar el registro de seguridad de Windows en el servidor NPS:

```powershell
Get-WinEvent -Logname Security | where {$_.ID -eq '6272'} | FL
```

![Visor de eventos de seguridad de PowerShell](./media/howto-mfa-nps-extension-vpn/image44.png)

También puede ver el registro de seguridad o la vista personalizada de Servicios de acceso y directivas de redes, tal y como se muestra a continuación:

![Ejemplo de registro del Servidor de directivas de redes](./media/howto-mfa-nps-extension-vpn/image45.png)

En el servidor donde instaló la extensión NPS para Azure Multi-Factor Authentication, puede encontrar registros de aplicación del Visor de eventos específicos de la extensión en *Application and Services Logs\Microsoft\AzureMfa*.

```powershell
Get-WinEvent -Logname Security | where {$_.ID -eq '6272'} | FL
```

![Ejemplo de panel de registros de AuthZ del Visor de eventos](./media/howto-mfa-nps-extension-vpn/image46.png)

## <a name="troubleshooting-guide"></a>Guía de solución de problemas

Si la configuración no funciona según lo esperado, verifique que el usuario está configurado para usar MFA para empezar a solucionar problemas. Haga que el usuario se conecte a [Azure Portal](https://portal.azure.com). Si al usuario se le solicita la autenticación secundaria y se puede autenticar correctamente, puede descartar una configuración incorrecta de MFA.

Si MFA funciona para el usuario, debe revisar los registros pertinentes del Visor de eventos. Se incluyen los registros de eventos de seguridad, los registros operativos de la puerta de enlace y los registros de Azure Multi-Factor Authentication descritos en la sección anterior.

A continuación, se muestra un ejemplo de un registro de seguridad que muestra un evento de inicio de sesión con errores (identificador de evento 6273):

![Registro de seguridad que muestra un evento de inicio de sesión con error](./media/howto-mfa-nps-extension-vpn/image47.png)

A continuación, se muestra un evento relacionado del registro de Azure Multi-Factor Authentication:

![Registros de Azure Multi-Factor Authentication](./media/howto-mfa-nps-extension-vpn/image48.png)

Para realizar opciones avanzadas de solución de problemas, consulte los archivos de registro de formato de la base de datos de NPS donde está instalado el servicio NPS. Estos archivos de registro se crean en la carpeta _%SystemRoot%\System32\Logs_ como archivos de texto delimitado por comas. Para obtener una descripción de estos archivos de registro, vea [Interpret NPS Database Format Log Files](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771748(v=ws.10)) (Interpretación de los archivos de registro de formato de la base de datos de NPS).

Las entradas de estos archivos de registro son difíciles de interpretar, a menos que los exporte en una hoja de cálculo o una base de datos. Puede encontrar muchas herramientas de análisis del Servicio de autenticación de Internet en línea para facilitar la interpretación de los archivos de registro. A continuación, se muestra la salida de una [aplicación shareware](https://www.deepsoftware.com/iasviewer) que se puede descargar:

![Ejemplo del analizador de IAS de la aplicación Shareware](./media/howto-mfa-nps-extension-vpn/image49.png)

Para más opciones de solución de problemas, puede utilizar un analizador de protocolos, como Wireshark o el [Analizador de mensajes de Microsoft](/message-analyzer/microsoft-message-analyzer-operating-guide). La siguiente imagen de Wireshark muestra los mensajes RADIUS entre el servidor VPN y el servidor NPS.

![Analizador de mensajes de Microsoft que muestra el tráfico filtrado](./media/howto-mfa-nps-extension-vpn/image50.png)

Para más información, consulte [Integración de la infraestructura NPS existente con Azure Multi-Factor Authentication](howto-mfa-nps-extension.md).

## <a name="next-steps"></a>Pasos siguientes

[Obtener Azure Multi-Factor Authentication](concept-mfa-licensing.md)

[Puerta de enlace de Escritorio remoto y Servidor Azure Multi-Factor Authentication con RADIUS](howto-mfaserver-nps-rdg.md)

[Integración de los directorios locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md)