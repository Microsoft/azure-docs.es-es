---
title: Solucionar problemas de las configuraciones de delegación restringida de Kerberos para el proxy de aplicación | Microsoft Docs
description: Solucionar problemas de las configuraciones de delegación restringida de Kerberos para el proxy de aplicación
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
ms.date: 04/23/2019
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: c95eb45d75a54af75fdc8004316bc8bfeeddbd7e
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783933"
---
# <a name="troubleshoot-kerberos-constrained-delegation-configurations-for-application-proxy"></a>Solucionar problemas de las configuraciones de delegación restringida de Kerberos para el proxy de aplicación

Los métodos disponibles para lograr el SSO en aplicaciones publicadas pueden variar de una aplicación a otra. Una de las opciones que ofrece Azure Active Directory (Azure AD) Application Proxy es la delegación restringida de Kerberos (KCD). Puede configurar un conector para que los usuarios ejecuten la autenticación de Kerberos restringida en aplicaciones de back-end.

El procedimiento para habilitar KCD es sencillo. Solo es necesario conocer de manera general los diversos componentes y flujos de autenticación que admiten el SSO. Pero a veces, el SSO de KCD no funciona como estaba previsto. Necesita obtener buenas fuentes de información para solucionar los problemas de estos escenarios.

En este artículo se proporciona un único punto de referencia que le ayudará a solucionar y corregir algunos de los problemas más habituales. También le ofrece un diagnóstico de los problemas de implementación más complejos.

En este artículo se da por supuesto lo siguiente:

-   La implementación de Azure AD Application Proxy (según el artículo[Empezar a trabajar con el proxy de aplicación e instalar el conector](application-proxy-add-on-premises-application.md)) y el acceso general a aplicaciones sin KCD funcionan según lo previsto.

-   La aplicación de destino publicada se basa en Internet Information Services (IIS) y la implementación de Kerberos de Microsoft.

-   Los hosts de servidor y aplicación residen en un único dominio de Azure Active Directory. Encontrará información detallada acerca de los escenarios de bosque y entre dominios en las [notas del producto para KCD](https://aka.ms/KCDPaper).

-   La aplicación en cuestión se publica en un inquilino de Azure con la autenticación previa habilitada. Se espera que los usuarios se autentiquen en Azure a través de la autenticación basada en formularios. Los escenarios de autenticación de cliente enriquecidos no se detallan en este artículo. Los agregaremos en el futuro.

## <a name="prerequisites"></a>Requisitos previos

Azure AD Application Proxy se puede implementar en varios tipos de infraestructuras o entornos. Las arquitecturas varían de una organización a otra. Las causas más comunes de los problemas relacionados con KCD no son los entornos. Las configuraciones erróneas más simples o los errores generales son los que causan la mayoría de los problemas.

Por esta razón, es mejor asegurarse de cumplir con todos los requisitos previos en [Using KCD SSO with the Application Proxy](application-proxy-configure-single-sign-on-with-kcd.md) (Usar el SSO de KCD con Application Proxy) antes de iniciar la resolución de problemas. Le recomendamos que lea la sección sobre la configuración de la delegación restringida de Kerberos en 2012R2. En este proceso se emplea un enfoque diferente para configurar KCD en versiones anteriores de Windows. Además, tenga en cuenta lo siguiente:

-   No es raro que un servidor miembro del dominio abra un cuadro de diálogo de canal seguro con un controlador de dominio específico (DC). Entonces, el servidor podría moverse a otro diálogo en cualquier momento. Por lo tanto, los hosts de conector no tienen restringida la comunicación solo con los controladores de dominio de sitios locales específicos.

-   Asimismo, los escenarios entre dominios se basan en referencias que dirigen a un host de conector hacia controladores de dominio que pueden residir fuera del perímetro de la red local. En estos casos, es igualmente importante enviar tráfico hacia los controladores de dominio que representan otros dominios respectivos. De lo contrario, se produce un error en la delegación.

-   Donde sea posible, evite colocar cualquier dispositivo IPS o IDS activo entre los hosts del conector y los controladores de dominio. Estos dispositivos a veces son excesivamente invasivos e interfieren con el tráfico principal de RPC.

Pruebe la delegación en escenarios simples. Cuantas más variables introduzca, con más tendrá que trabajar. Para ahorrar tiempo, limite las pruebas a un solo conector. Agregue conectores adicionales después de resolver el problema.

Algunos factores del entorno también podrían provocar un problema. Para evitar estos factores, minimice arquitectura tanto como sea posible durante las pruebas. Por ejemplo, es común encontrar ACL mal configuradas en el firewall interno. Si es posible, envíe todo el tráfico de un conector directamente a los controladores de dominio y a la aplicación de back-end.

El mejor lugar para colocar los conectores es lo más cerca posible de sus destinos. Insertar un firewall al realizar pruebas, agrega complejidad innecesaria y puede prolongar sus investigaciones.

¿Qué muestra un problema de KCD? Hay varias indicaciones comunes de que el SSO de KCD está fallando. Los primeros signos de un problema aparecen en el navegador.

   ![Error de configuración de KCD incorrecta](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic1.png)

   ![No se pudo autorizar por falta de permisos](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic2.png)

Ambas imágenes muestran el mismo síntoma: un error de SSO. Se denegó el acceso de usuario a la aplicación.

## <a name="troubleshooting"></a>solución de problemas

El modo de solucionar cada problema depende del mismo problema y de los síntomas observados. Antes de seguir adelante, lea los siguientes artículos. Le proporcionarán información útil para solucionar problemas:

-   [Solución de problemas y mensajes de error de Proxy de aplicación](application-proxy-troubleshoot.md)

-   [Errores y síntomas de Kerberos](application-proxy-troubleshoot.md#kerberos-errors)

-   [Trabajar con SSO cuando las identidades locales y en la nube no son idénticas](application-proxy-configure-single-sign-on-with-kcd.md#working-with-different-on-premises-and-cloud-identities)

Si llegó a este punto, entonces el problema principal existe. Para comenzar, separe el flujo en tres fases distintas cuyos problemas pueda solucionar.

### <a name="client-pre-authentication"></a>Autenticación previa del cliente 
El usuario externo se autentica en Azure mediante un explorador. Es fundamental poder realizar la autenticación previa en Azure para que el SSO de KCD funcione. Debe probar y abordar este asunto si hay algún problema. La fase de autenticación previa no guarda relación con KCD ni con la aplicación publicada. Debería ser bastante fácil corregir cualquier discrepancia con una comprobación de que la cuenta en cuestión existe en Azure. Compruebe también que no esté deshabilitada o bloqueada. La respuesta de error en el explorador suele ser lo bastante descriptiva como para entender la causa. Si no está seguro de lo que pasa, consulte otros artículos de solución de problemas de Microsoft para obtener más información.

### <a name="delegation-service"></a>Servicio de delegación 
Es el conector Azure Proxy que obtiene un vale de servicio de Kerberos para los usuarios del Centro de distribución de claves de Kerberos (KCD).

Las comunicaciones externas entre el cliente y el front-end de Azure no tienen ninguna relación con KCD. Estas comunicaciones solo sirven para comprobar que KCD funciona. Por lo tanto, se proporciona al servicio de Azure Proxy un identificador de usuario válido que se usará para obtener un vale de Kerberos. Sin este id., KCD no funciona y se produce un error.

Tal como se mencionó antes, los mensajes de error del explorador suelen proporcionar buenos indicios de por qué se producen errores. Asegúrese de anotar el id. de actividad y la marca de tiempo en la respuesta. Esta información le ayudará a poner en correlación el comportamiento de eventos reales en el registro de eventos de Azure Proxy.

   ![Error de configuración de KCD incorrecta](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic3.png)

Las entradas correspondientes en el registro de eventos se muestran como los eventos 13019 o 12027. Puede encontrar los registros de eventos del conector en **Registros de aplicaciones y servicios** &gt; **Microsoft** &gt; **AadApplicationProxy** &gt; **Conector**&gt;**Administrador**.

   ![Evento 13019 del registro de eventos del proxy de aplicación](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic4.png)

   ![Evento 12027 del registro de eventos del proxy de aplicación](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic5.png)

1.   Use un registro **A** en el servicio DNS interno de la dirección de la aplicación, en lugar de un registro **CName**.

2.   Vuelva a confirmar que se han concedido al host de conector los derechos para delegar en el SPN de la cuenta de destino designada. Vuelva a confirmar que seleccionó **Use any authentication protocol** (Usar cualquier protocolo de autenticación). Para obtener más información, consulte el [artículo de configuración del SSO](application-proxy-configure-single-sign-on-with-kcd.md).

3.   Compruebe que existe solo una instancia de SPN en Azure AD. Emita `setspn -x` desde un símbolo del sistema en cualquier host de miembro de dominio.

4.   Compruebe que se aplica una política de dominio que limite el [tamaño máximo de los tokens de Kerberos emitidos](https://blogs.technet.microsoft.com/askds/2012/09/12/maxtokensize-and-windows-8-and-windows-server-2012/). Esta política impide que el conector obtenga un token si se determina que es excesivo.

El siguiente paso que se recomienda para obtener más información de bajo nivel sobre los problemas, es realizar un seguimiento de red que capture los intercambios entre el host de conector y un KDC de dominio. Para obtener más información, consulte [la información que se proporciona acerca de la solución de problemas](https://aka.ms/proxytshootpaper).

Si la generación de vales no presenta problemas, verá un evento en los registros que indica que se produjo un error en la autenticación porque la aplicación devolvió un error 401. Este evento indica que la aplicación de destino rechazó su vale. Vaya al paso siguiente.

### <a name="target-application"></a>Aplicación de destino 
Es el consumidor del vale de Kerberos que proporciona el conector. En esta fase, espere a que el conector haya enviado un vale de servicio de Kerberos al back-end. Este vale es un encabezado en la primera solicitud de la aplicación.

1.   Mediante la dirección URL interna de la aplicación definida en el portal, compruebe que la aplicación sea accesible directamente desde el explorador en el host de conector. Entonces podrá iniciar sesión correctamente. Encontrará detalles sobre el conector en la página de **solución de problemas**.

2.   Todavía en el host de conector, confirme que la autenticación entre el explorador y la aplicación esté utilizando Kerberos. Para ello, realice las siguientes acciones:

3.  Ejecute DevTools (**F12**) en Internet Explorer o use [Fiddler](https://blogs.msdn.microsoft.com/crminthefield/2012/10/10/using-fiddler-to-check-for-kerberos-auth/) desde el host de conector. Use la dirección URL interna para ir a la aplicación. Inspeccione los encabezados de autorización WWW ofrecidos que se devuelven en la respuesta de la aplicación, para asegurarse de que Negotiate o Kerberos esté presente. 

    - El siguiente blob de Kerberos que devuelve el explorador en respuesta a la aplicación, comienza con **YII**. Estas letras indican que Kerberos se está ejecutando. Por otra parte, Microsoft NT LAN Manager (NTLM) siempre empieza por **TlRMTVNTUAAB**, que se convierte en el Proveedor de compatibilidad para seguridad de NTLM (NTLMSSP) cuando se descodifica desde Base64. Si ve **TlRMTVNTUAAB** al principio del blob, significa que Kerberos no está disponible. Si no ve **TlRMTVNTUAAB**, es probable que Kerberos esté disponible.
   
       > [!NOTE]
       > Si usa Fiddler, este método requiere deshabilitar temporalmente la protección ampliada en la configuración de la aplicación en IIS.
      
      ![Ventana de inspección de red del explorador](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic6.png)
   
    - El blob en esta figura no comienza con **TIRMTVNTUAAB**. Por lo tanto, Kerberos está disponible en este ejemplo, y el blob de Kerberos no comienza con **YII**.

4.  Elimine temporalmente NTLM de la lista de proveedores en el sitio de IIS. Acceda a la aplicación directamente desde Internet Explorer en el host del conector. NTLM ya no se encuentra en la lista de proveedores. Puede acceder a la aplicación usando solo Kerberos. Si el acceso falla, puede haber un problema con la configuración de la aplicación. La autenticación Kerberos no funciona.

    - Si Kerberos no está disponible, compruebe la configuración de autenticación de la aplicación en IIS. Asegúrese de que **Negotiate** aparece en la parte superior, con NTLM justo debajo de él. Si ve **No negociar**, **Kerberos o Negotiate**, o **PKU2U**, continúe solo si Kerberos funciona.

       ![Proveedores de autenticación de Windows](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic7.png)
   
    - Con Kerberos y NTLM listos, debe deshabilitar temporalmente la autenticación previa de la aplicación en el portal. Pruebe a acceder a ella desde Internet mediante la dirección URL externa. Se le pedirá que se autentique. Puede hacerlo con la misma cuenta que usó en el paso anterior. Si no, es que hay un problema con la aplicación de back-end, y no con KCD.

    - Vuelva a habilitar la autenticación previa en el portal. A continuación, autentíquese con Azure; para ello, intente conectarse a la aplicación mediante su dirección URL externa. Si se produce un error de SSO, verá un mensaje de error en el explorador, además del evento 13022 en el registro:

       *El conector de proxy de aplicación de Microsoft AAD no puede autenticar al usuario porque el servidor back-end responde a los intentos de autenticación Kerberos con un error HTTP 401.*

       ![Error HTTTP 401: Prohibido](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic8.png)
   
    - Comprobar la aplicación de IIS. Asegúrese de que el grupo de aplicaciones y el SPN estén configurados para que puedan usar la misma cuenta en Azure AD. Navegue por IIS, tal como se muestra en la siguiente ilustración:
      
       ![Ventana de configuración de la aplicación IIS](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic9.png)
      
       Después de conocer la identidad, asegúrese de que esta cuenta esté configurada con el SPN en cuestión. Un ejemplo es `setspn –q http/spn.wacketywack.com`. Escriba lo siguiente en una ventana del símbolo del sistema:
      
       ![Ventana de comando SetSPN](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic10.png)
      
    - Compruebe el SPN que se definió según la configuración de la aplicación en el portal. Asegúrese de que usa el mismo SPN configurado con el destino de la cuenta de Azure AD por grupo de aplicaciones de la aplicación.

       ![Configuración del SPN en Azure Portal](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic11.png)
   
    - Vaya a IIS y seleccione la opción **Editor de configuración** de la aplicación. Vaya a **system.webServer/security/authentication/windowsAuthentication**. Asegúrese de que el valor **UseAppPoolCredentials** está establecido en **True**.

       ![Opción de credencial de grupos de aplicaciones en la configuración de IIS](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic12.png)

       Cambie este valor a **True**. Para quitar todos los vales de Kerberos en la caché desde el servidor de back-end, ejecute el siguiente comando:

       ```powershell
       Get-WmiObject Win32_LogonSession | Where-Object {$_.AuthenticationPackage -ne 'NTLM'} | ForEach-Object {klist.exe purge -li ([Convert]::ToString($_.LogonId, 16))}
       ``` 

Para obtener más información, consulte [Purge the Kerberos client ticket cache for all sessions](https://gallery.technet.microsoft.com/scriptcenter/Purge-the-Kerberos-client-b56987bf) (Purgar la memoria caché de vales de clientes de Kerberos para todas las sesiones).



Si deja el modo Kernel habilitado, mejora el rendimiento de las operaciones de Kerberos. Asimismo, también provoca que el vale del servicio solicitado se descifre mediante la cuenta de la máquina. Esta cuenta también se llama Sistema Local. Establezca este valor en **True**  para interrumpir el proceso de KCD cuando la aplicación esté alojada en más de un servidor de una granja de servidores.

-   Como comprobación adicional, deshabilite también la protección **extendida**. En algunos casos, la protección **extendida** interrumpe el proceso de KCD cuando se habilita en configuraciones específicas. En esos casos, una aplicación se publica como una subcarpeta del sitio web predeterminado. Esta aplicación está configurada para que solo pueda realizarse una autenticación anónima. Además, todos los cuadros de diálogo están atenuados, lo que sugiere que los objetos secundarios no heredarán ninguna configuración activa. Es recomendable que haga pruebas con esto, pero no olvide restaurar este valor a **habilitado**, cuando le sea posible.

    Este control adicional le permite usar la aplicación publicada. Puede poner en marcha conectores adicionales que también están configurados para realizar procesos de delegado. Para obtener más información, lea la guía técnica detallada, [Troubleshooting the Azure AD Application Proxy](https://aka.ms/proxytshootpaper) (Solución de problemas del Proxy de aplicación Azure AD).

Si aún no puede realizar ningún progreso, póngase en contacto con el servicio de soporte técnico de Microsoft para obtener ayuda. Cree una incidencia de soporte técnico directamente en el portal. Un ingeniero se pondrá en contacto con usted.

## <a name="other-scenarios"></a>Otros escenarios

- El proxy de aplicación de Azure solicita un vale de Kerberos antes de enviar su solicitud a una aplicación. Algunas aplicaciones de terceros no le gusta de este método de autenticación. Estas aplicaciones esperan que se lleven a cabo negociaciones más convencionales. La primera solicitud es anónima, lo que permite que la aplicación responda con los tipos de autenticación que admite mediante un error 401.

- La autenticación de varios saltos se usa normalmente en escenarios con una aplicación en niveles y con un back-end y un front-end, los cuales requieren realizar una autenticación, como SQL Server Reporting Services. Para configurar el escenario de múltiples saltos, consulte el artículo de soporte técnico [Kerberos Constrained Delegation May Require Protocol Transition in Multi-hop Scenarios](https://support.microsoft.com/help/2005838/kerberos-constrained-delegation-may-require-protocol-transition-in-mul) (La delegación restringida de Kerberos puede requerir una transición de protocolo en escenarios de varios saltos).

## <a name="next-steps"></a>Pasos siguientes
[Configurar KCD en un dominio administrado](../../active-directory-domain-services/active-directory-ds-enable-kcd.md).
