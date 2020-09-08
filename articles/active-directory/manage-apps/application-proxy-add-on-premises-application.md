---
title: 'Tutorial: Adición de una aplicación local: proxy de aplicación en Azure AD'
description: Azure Active Directory (Azure AD) tiene un servicio de proxy de aplicación que permite a los usuarios tener acceso a aplicaciones locales, iniciando sesión con su cuenta de Azure AD. Este tutorial le muestra cómo preparar el entorno para su uso con el proxy de aplicación. Después, este usa Azure Portal para agregar una aplicación local al inquilino de Azure AD.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: tutorial
ms.date: 10/24/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 7d94b8604b2f947463dd760ca7baf25f19a15a26
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642153"
---
# <a name="tutorial-add-an-on-premises-application-for-remote-access-through-application-proxy-in-azure-active-directory"></a>Tutorial: Adición de una aplicación local para el acceso remoto mediante el proxy de aplicación en Azure Active Directory

Azure Active Directory (Azure AD) tiene un servicio de proxy de aplicación que permite a los usuarios tener acceso a aplicaciones locales, iniciando sesión con su cuenta de Azure AD. En este tutorial se prepara el entorno para su uso con el proxy de aplicación. Una vez que el entorno está preparado, usará Azure Portal para agregar una aplicación local a su inquilino de Azure AD.

Este tutorial:

> [!div class="checklist"]
> * Abre los puertos para el tráfico saliente y permite el acceso a direcciones URL específicas.
> * Instala el conector en Windows Server y lo registra con el proxy de aplicación.
> * Verifica si el conector está instalado y registrado correctamente.
> * Agrega una aplicación local al inquilino de Azure AD.
> * Verifica que un usuario de prueba puede iniciar sesión en la aplicación con una cuenta de Azure AD.

## <a name="prerequisites"></a>Requisitos previos

Para agregar una aplicación local a Azure AD, se necesita:

* Una [suscripción Premium a Microsoft Azure AD](https://azure.microsoft.com/pricing/details/active-directory).
* Una cuenta de administrador de aplicaciones.
* Las identidades de usuario se deben sincronizar desde un directorio local, o bien se deben crear directamente en los inquilinos de Azure AD. La sincronización de identidades permite a Azure AD realizar una autenticación previa de los usuarios antes de concederles acceso a aplicaciones publicadas en App Proxy y tener la información del identificador de usuario necesarios para realizar el inicio de sesión único (SSO).

### <a name="windows-server"></a>Windows Server

Para usar el proxy de aplicación, necesita un servidor de Windows que ejecute Windows Server 2012 R2 o posterior. Instalará el conector del proxy de aplicación en el servidor. Este servidor de conector necesita tener la capacidad de conectarse a los servicios del proxy de aplicación en Azure y a las aplicaciones locales que planea publicar.

Para conseguir una alta disponibilidad del entorno de producción, se recomienda disponer de más de un servidor Windows Server. Para este tutorial, es suficiente con un servidor Windows Server.

> [!IMPORTANT]
> Si va a instalar el conector en Windows Server 2019, debe deshabilitar la compatibilidad con el protocolo HTTP2 en el componente WinHttp para que la delegación restringida de Kerberos funcione correctamente. Esta opción está deshabilitada de forma predeterminada en versiones anteriores de los sistemas operativos compatibles. Al agregar la siguiente clave del registro y reiniciar el servidor, se deshabilita en Windows Server 2019. Tenga en cuenta que se trata de una clave del registro para toda la máquina.
>
> ```
> Windows Registry Editor Version 5.00
> 
> [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\WinHttp] "EnableDefaultHttp2"=dword:00000000
> ```
>

#### <a name="recommendations-for-the-connector-server"></a>Recomendaciones para el servidor de conector

1. Busque físicamente el servidor de conector cerca de los servidores de aplicación para optimizar el rendimiento entre el conector y la aplicación. Para más información, vea [Consideraciones sobre la topología de red](application-proxy-network-topology.md).
1. El servidor del conector y los servidores de aplicaciones web deben pertenecer al mismo dominio de Active Directory o abarcar dominios de confianza. Tener los servidores en el mismo dominio o en dominios de confianza es un requisito para usar el inicio de sesión único (SSO) con la autenticación integrada de Windows (IWA) y la delegación restringida de Kerberos (KCD). Si el servidor de conector y los servidores de aplicaciones web están en diferentes dominios de Active Directory, deberá utilizar la delegación basada en recursos para el inicio de sesión único. Para obtener más información, consulte el artículo sobre[la KCD para el inicio de sesión único con el proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md).

> [!WARNING]
> Si ha implementado Azure AD Password Protection Proxy, no instale Azure AD Application Proxy ni Azure AD Password Protection Proxy juntos en la misma máquina. Azure AD Application Proxy y Azure AD Password Protection Proxy instalan versiones diferentes del servicio Agent Updater de Azure AD Connect. Estas versiones diferentes son incompatibles cuando se instalan juntas en la misma máquina.

#### <a name="tls-requirements"></a>Requisitos de TLS

El servidor de conector de Windows debe tener habilitado TLS 1.2 antes de instalar el conector del proxy de aplicación.

Para habilitar TLS 1.2, siga estos pasos:

1. Establezca las siguientes claves del Registro:
    
   ```
   Windows Registry Editor Version 5.00

   [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
   [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
   "DisabledByDefault"=dword:00000000
   "Enabled"=dword:00000001
   [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server]
   "DisabledByDefault"=dword:00000000
   "Enabled"=dword:00000001
   [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
   "SchUseStrongCrypto"=dword:00000001
   ```

1. Reinicie el servidor.

## <a name="prepare-your-on-premises-environment"></a>Preparación del entorno local

Para preparar su entorno para Azure AD Application Proxy, primero debe habilitar la comunicación con los centros de datos de Azure. Si hay un firewall en la ruta de acceso, asegúrese de que está abierto. Un firewall abierto permite que el conector realice solicitudes HTTPS (TCP) al proxy de aplicación.

> [!IMPORTANT]
> Si va a instalar el conector para la nube de Azure Government, siga los [requisitos previos](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-government-cloud#allow-access-to-urls) y los [pasos de instalación](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-government-cloud#install-the-agent-for-the-azure-government-cloud). Esto requiere habilitar el acceso a un conjunto diferente de direcciones URL y que un parámetro adicional ejecute la instalación.

### <a name="open-ports"></a>Abrir puertos

Abra los siguientes puertos al tráfico **saliente**.

   | Número de puerto | Cómo se usa |
   | --- | --- |
   | 80 | Descarga de listas de revocación de certificados (CRL) al validar el certificado TLS/SSL |
   | 443 | Toda la comunicación saliente con el servicio del proxy de aplicación |

Si el firewall fuerza el tráfico según los usuarios de origen, abra también los puertos 80 y 443 para el tráfico de los servicios de Windows que se ejecutan como un servicio de red.

### <a name="allow-access-to-urls"></a>Permiso de acceso a direcciones URL

Permita el acceso a las siguientes direcciones URL:

| URL | Cómo se usa |
| --- | --- |
| \*.msappproxy.net<br>\*.servicebus.windows.net | Comunicación entre el conector y el servicio en la nube del proxy de aplicación |
| mscrl.microsoft.com:80<br>crl.microsoft.com:80<br>ocsp.msocsp.com:80<br>www.microsoft.com:80 | El conector utiliza estas direcciones URL para comprobar los certificados. |
| login.windows.net<br>secure.aadcdn.microsoftonline-p.com<br>\*.microsoftonline.com<br>\*.microsoftonline-p.com<br>\*.msauth.net<br>\*.msauthimages.net<br>\*.msecnd.net<br>\*.msftauth.net<br>\*.msftauthimages.net<br>\*.phonefactor.net<br>enterpriseregistration.windows.net<br>management.azure.com<br>policykeyservice.dc.ad.msft.net<br>ctldl.windowsupdate.com:80 | El conector utiliza estas direcciones URL durante el proceso de registro. |

Puede permitir conexiones a \*.msappproxy.net y \*.servicebus.windows.net si el firewall o el proxy le permiten configurar listas de DNS permitidos. Si no es así, deberá permitir acceso a [Intervalos IP de Azure y etiquetas de servicio: nube pública](https://www.microsoft.com/download/details.aspx?id=56519). Los intervalos IP se actualizan cada semana.

## <a name="install-and-register-a-connector"></a>Instalación y registro de un conector

Para usar Application Proxy, deberá instalar un conector en cada servidor Windows Server que use con el servicio Application Proxy. El conector es un agente que administra la conexión saliente desde los servidores de aplicación locales al proxy de aplicación en Azure AD. Puede instalar un conector en los servidores que también tienen otros agentes de autenticación instalados, como Azure AD Connect.


Para instalar el conector:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de aplicaciones del directorio que usa el proxy de aplicación. Por ejemplo, si el dominio del inquilino es contoso.com, el administrador debe ser admin@contoso.com o cualquier otro alias de administrador de ese dominio.
1. En la esquina superior derecha, seleccione su nombre de usuario. Compruebe que ha iniciado sesión en el directorio que usa Application Proxy. Si necesita cambiar directorios, seleccione **Cambiar directorio** y elija un directorio que use Application Proxy.
1. En el panel de navegación izquierdo, seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Application proxy**.
1. Seleccione **Descargar servicio de conector**.

    ![Descargue el servicio de conector para ver los términos del servicio.](./media/application-proxy-add-on-premises-application/application-proxy-download-connector-service.png)

1. Lea los términos del servicio. Cuando esté listo, seleccione **Aceptar las condiciones y descargar**.
1. En la parte inferior de la ventana, seleccione **Ejecutar** para instalar el conector. Se abre un asistente para la instalación.
1. Siga las instrucciones del asistente para instalar el servicio. Cuando se le pida que registre el conector en el proxy de aplicación para el inquilino de Azure AD, proporcione las credenciales del administrador de la aplicación.
    - En Internet Explorer (IE), si la opción **Configuración de seguridad mejorada de IE**  está **activada**, puede que no aparezca la pantalla de registro. Para acceder, siga las instrucciones del mensaje de error. Asegúrese de que la **configuración de seguridad mejorada de Internet Explorer** está **desactivada**.

### <a name="general-remarks"></a>Observaciones generales

Si anteriormente ha instalado un conector, vuelva a instalarlo para obtener la última versión. Para consultar información sobre las versiones publicadas anteriormente y qué cambios incluyen, consulte [Application Proxy: Version Release History](application-proxy-release-version-history.md) (Proxy de aplicación: Historial de lanzamiento de versiones).

Si elige tener más de un servidor Windows Server para las aplicaciones locales, deberá instalar y registrar el conector en cada servidor. Puede organizar los conectores en grupos. Para más información, vea [Grupos de conectores](application-proxy-connector-groups.md).

Si su organización usa servidores proxy para conectarse a Internet, deberá configurarlos para el proxy de aplicación.  Para obtener más información, consulte [Trabajo con servidores proxy locales existentes](application-proxy-configure-connectors-with-proxy-servers.md). 

Para obtener información sobre los conectores, el planeamiento de capacidad y cómo actualizarlos, vea [Descripción de los conectores de Azure AD Application Proxy](application-proxy-connectors.md).

## <a name="verify-the-connector-installed-and-registered-correctly"></a>Verificar si el conector se ha instalado y registrado correctamente

Puede usar Azure Portal o Windows Server para confirmar que el nuevo conector se ha instalado correctamente.

### <a name="verify-the-installation-through-azure-portal"></a>Comprobación de la instalación mediante Azure Portal

Para confirmar que el conector se ha instalado y registrado correctamente:

1. Inicie sesión en su directorio de inquilino en [Azure Portal](https://portal.azure.com).
1. En el panel de navegación izquierdo, seleccione **Azure Active Directory**y, a continuación, seleccione **Application Proxy** en la sección **Administración**. Todos los conectores y grupos de conectores aparecen en esta página.
1. Vea un conector para verificar sus detalles. Los conectores deben estar ampliados de forma predeterminada. Si el conector que desea ver no está expandido, expándalo para ver los detalles. Una etiqueta activa verde indica que el conector puede conectarse al servicio. Sin embargo, aun cuando la etiqueta es verde, un problema de red podría impedir que el conector reciba mensajes.

    ![Conectores de Azure AD Application Proxy](./media/application-proxy-add-on-premises-application/app-proxy-connectors.png)

Para obtener más ayuda con la instalación de un conector, consulte [Problema al instalar el conector de agente del proxy de aplicación](application-proxy-connector-installation-problem.md).

### <a name="verify-the-installation-through-your-windows-server"></a>Comprobación de la instalación mediante el servidor de Windows

Para confirmar que el conector se ha instalado y registrado correctamente:

1. Abra el Administrador de servicios de Windows; para ello, haga clic en la tecla del logotipo de **Windows** y escriba *services.msc*.
1. Compruebe si el estado de los dos servicios siguientes es **En ejecución**.
   - El **conector de Microsoft AAD Application Proxy** habilita la conectividad.
   - **Actualizador del conector del proxy de aplicación de Microsoft AAD** es un servicio de actualización automática. El actualizador busca nuevas versiones del conector y lo actualiza según sea necesario.

     ![Servicios de conector del proxy de la aplicación (captura de pantalla)](./media/application-proxy-add-on-premises-application/app_proxy_services.png)

1. Si el estado de los servicios no es **En ejecución**, haga clic con el botón derecho en cada servicio y elija **Iniciar**.

## <a name="add-an-on-premises-app-to-azure-ad"></a>Adición de una aplicación local a Azure AD

Ahora que ya ha preparado el entorno y ha instalado un conector, está listo para agregar las aplicaciones locales a Azure AD.  

1. Inicie sesión como administrador en [Azure Portal](https://portal.azure.com/).
2. En el panel de navegación izquierdo, seleccione **Azure Active Directory**.
3. Seleccione **Aplicaciones empresariales** y, después, **Nueva aplicación**.
4. En la sección **Aplicaciones locales**, seleccione **Incorporación de una aplicación local**.
5. En la sección **Agregar aplicación local propia**, proporcione la siguiente información sobre la aplicación:

    | Campo | Descripción |
    | :---- | :---------- |
    | **Nombre** | El nombre de la aplicación que va a aparecer en Aplicaciones y en Azure Portal. |
    | **Dirección URL interna** | La dirección URL para acceder a la aplicación desde la red privada. Puede especificar una ruta de acceso específica en el servidor back-end para publicar, mientras que el resto del servidor no se publica. De esta forma, puede publicar sitios diferentes en el mismo servidor como aplicaciones diferentes y dar a cada uno un nombre y unas reglas de acceso propios.<br><br>Si publica una ruta de acceso, asegúrese de que incluye todas las imágenes, los scripts y las hojas de estilos necesarias para la aplicación. Por ejemplo, si la aplicación está en https:\//yourapp/app y usa las imágenes que se encuentran en https:\//yourapp/media, debe publicar https:\//yourapp/ como la ruta de acceso. Esta dirección URL interna no tiene que ser la página de inicio que verán los usuarios. Para más información, consulte [Establecimiento de una página principal personalizada para aplicaciones publicadas mediante el proxy de aplicación de Azure AD](application-proxy-configure-custom-home-page.md). |
    | **Dirección URL externa** | La dirección para que los usuarios accedan a la aplicación desde fuera de la red. Si no desea usar el dominio del proxy de aplicación predeterminado, lea sobre el [uso de dominios personalizados en el proxy de aplicación de Azure AD](application-proxy-configure-custom-domain.md).|
    | **Autenticación previa** | La forma en que el proxy de aplicación verifica los usuarios antes de concederles acceso a la aplicación.<br><br>**Azure Active Directory**: el proxy de la aplicación redirige a los usuarios para que inicien sesión en Azure AD, que autentica sus permisos para el directorio y la aplicación. Se recomienda mantener esta opción como predeterminada, para que pueda aprovechar las características de seguridad de Azure AD como el acceso condicional y Multi-Factor Authentication. Se necesita **Azure Active Directory** para la supervisión de la aplicación con Microsoft Cloud Application Security.<br><br>**Acceso directo**: los usuarios no tienen que autenticarse en Azure AD para tener acceso a la aplicación. Esto no impide que pueda configurar los requisitos de autenticación en el back-end. |
    | **Grupo de conectores** | Los conectores procesan el acceso remoto a la aplicación, y los grupos de conectores le ayudan a organizar los conectores y las aplicaciones por región, red o finalidad. Si no tiene ningún grupo de conectores creado todavía, la aplicación se asigna al **predeterminado**.<br><br>Si la aplicación usa WebSockets para conectarse, todos los conectores del grupo deben tener la versión 1.5.612.0 o posterior.|

6. Si es necesario, realice otras configuraciones en **Configuración adicional**. En la mayoría de las aplicaciones, debe mantener esta configuración en su estado predeterminado. 

    | Campo | Descripción |
    | :---- | :---------- |
    | **Tiempo de espera de las aplicaciones de back-end** | Establezca este valor en **Largo** solo si la aplicación es lenta en autenticarse y conectarse. De forma predeterminada, el tiempo de espera de la aplicación back-end tiene una longitud de 85 segundos. Cuando se establece en Largo, el tiempo de espera de back-end se incrementa en 180 segundos. |
    | **Usar cookie solo HTTP** | Establezca este valor en **Sí** para que las cookies del proxy de aplicación incluyan la marca HTTPOnly en el encabezado de respuesta HTTP. Si se usan los Servicios de Escritorio remoto, establezca este valor en **No**.|
    | **Usar cookies seguras**| Establezca este valor en **Sí** para transmitir las cookies a través de un canal seguro como una solicitud HTTPS cifrada.
    | **Usar cookies persistentes**| Mantenga este valor establecido en **No**. Esta configuración solo debe usarse para las aplicaciones que no pueden compartir cookies entre procesos. Para más información sobre la configuración de las cookies, consulte [Cookie settings for accessing on-premises applications in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-cookie-settings) (Configuración de las cookies para el acceso a aplicaciones locales en Azure Active Directory).
    | **Traducir URL en encabezados** | Mantenga este valor como **Sí** a menos que la aplicación requiera el encabezado de host original en la solicitud de autenticación. |
    | **Traducir direcciones URL en el cuerpo de la aplicación** | Mantenga este valor como **No** a menos que tenga vínculos HTML codificados a otras aplicaciones locales y no use dominios personalizados. Para más información, consulte sobre la [traducción de vínculos con el proxy de aplicación](application-proxy-configure-hard-coded-link-translation.md).<br><br>Establezca este valor en **Sí** si tiene previsto supervisar esta aplicación con Microsoft Cloud App Security (MCAS). Para más información, consulte [Configure real-time application access monitoring with Microsoft Cloud App Security and Azure Active Directory](application-proxy-integrate-with-microsoft-cloud-application-security.md) (Configuración de la supervisión del acceso a las aplicaciones en tiempo real con Microsoft Cloud App Security y Azure Active Directory). |

7. Seleccione **Agregar**.

## <a name="test-the-application"></a>Prueba de la aplicación

Está listo para probar que la aplicación se agregó correctamente. En los pasos siguientes, agregará una cuenta de usuario a la aplicación y probará el inicio de sesión.

### <a name="add-a-user-for-testing"></a>Adición de un usuario de prueba

Antes de agregar un usuario a la aplicación, compruebe que la cuenta de usuario ya tiene permisos para acceder a la aplicación desde dentro de la red corporativa.

Para agregar un usuario de prueba:

1. Seleccione **Aplicaciones empresariales** y, después, seleccione la aplicación que desea probar.
2. Seleccione **Introducción** y, a continuación, seleccione **Assign a user for testing** (Asignar un usuario de prueba).
3. En **Usuarios y grupos**, seleccione **Agregar usuario**.
4. En **Agregar asignación**, seleccione **Usuarios y grupos**. Aparece la sección **Usuario y grupos**.
5. Elija la cuenta que desea agregar.
6. Elija **Seleccionar** y, a continuación, seleccione **Asignar**.

### <a name="test-the-sign-on"></a>Probar el inicio de sesión

Para probar el inicio de sesión en la aplicación:

1. En la aplicación que quiere probar, seleccione **Proxy de aplicación**.
2. En la parte superior de la página, seleccione **Aplicación de prueba** para ejecutar una prueba en la aplicación y comprobar posibles problemas de configuración.
3. Asegúrese de iniciar primero la aplicación para probar el inicio de sesión en la aplicación y, luego, descargue el informe de diagnóstico para revisar la guía de resolución de los problemas detectados.

Para solucionar problemas, vea [Solución de problemas y mensajes de error de Proxy de aplicación](application-proxy-troubleshoot.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos que creó en este tutorial.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se ha preparado el entorno local para que funcione con el proxy de aplicación y, después, se ha instalado y registrado el conector de proxy de aplicación. A continuación, agregó una aplicación a su inquilino de Azure AD. También comprobó que un usuario puede iniciar sesión en la aplicación con una cuenta de Azure AD.

Hizo todo esto:
> [!div class="checklist"]
> * Abrió los puertos para el tráfico saliente y permitió el acceso a direcciones URL específicas.
> * Instaló el conector en Windows Server y lo registró con el proxy de aplicación.
> * Verificó si el conector estaba instalado y registrado correctamente.
> * Agregó una aplicación local al inquilino de Azure AD.
> * Verificó que un usuario de prueba podía iniciar sesión en la aplicación con una cuenta de Azure AD.

Ya está preparado para configurar la aplicación para el inicio de sesión único. Use el siguiente vínculo para elegir un método de inicio de sesión único y buscar tutoriales sobre el inicio de sesión único.

> [!div class="nextstepaction"]
> [Configuración del inicio de sesión único](sso-options.md#choosing-a-single-sign-on-method)
