---
title: 'Azure AD Connect: Requisitos previos y hardware | Microsoft Docs'
description: En este artículo se describen los requisitos previos y los requisitos de hardware de Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 91b88fda-bca6-49a8-898f-8d906a661f07
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/25/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 17209919e9bcd93a9a0184d8d4df9c29fff62ef9
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89277330"
---
# <a name="prerequisites-for-azure-ad-connect"></a>Requisitos previos de Azure AD Connect
En este artículo se describen los requisitos previos y los requisitos de hardware de Azure Active Directory (Azure AD) Connect.

## <a name="before-you-install-azure-ad-connect"></a>Antes de instalar Azure AD Connect
Antes de instalar Azure AD Connect, hay algunas cosas que necesita.

### <a name="azure-ad"></a>Azure AD
* Necesita un inquilino de Azure AD. Obtiene uno con [una prueba gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/). Puede usar uno de los siguientes portales para administrar Azure AD Connect:
  * [Azure Portal](https://portal.azure.com)
  * [Portal de Office](https://portal.office.com)
* [Agregue y compruebe el dominio](../fundamentals/add-custom-domain.md) que pretende usar en Azure AD. Por ejemplo, si tiene previsto usar contoso.com para los usuarios, asegúrese de que este dominio se ha comprobado y de que no usa solamente el dominio predeterminado contoso.onmicrosoft.com.
* Un inquilino de Azure AD admite 50 000 objetos de forma predeterminada. Al comprobar el dominio, el límite se incrementa a 300 000 objetos. Si todavía necesita más objetos en Azure AD, abra una incidencia de soporte técnico para aumentar el límite aún más. Si necesita más de 500 000 objetos, necesitará una licencia, como Office 365, Azure AD Premium o Enterprise Mobility + Security.

### <a name="prepare-your-on-premises-data"></a>Preparación de los datos locales
* Use [IdFix](https://support.office.com/article/Install-and-run-the-Office-365-IdFix-tool-f4bd2439-3e41-4169-99f6-3fabdfa326ac) para identificar errores tales como duplicados y problemas de formato en el directorio antes de sincronizar con Azure AD y Office 365.
* Revise [las características de sincronización opcionales que se pueden habilitar en Azure AD](how-to-connect-syncservice-features.md) y valore cuáles son las que debe habilitar.

### <a name="on-premises-active-directory"></a>Active Directory local
* La versión del esquema de Active Directory y el nivel funcional del bosque deben ser Windows Server 2003 o una versión posterior. Los controladores de dominio pueden ejecutar cualquier versión siempre que se cumplan los requisitos de versión de esquema y nivel de bosque.
* Si pretende usar la característica de *escritura diferida de contraseñas*, los controladores de dominio deben estar en Windows Server 2008 R2 o versiones posteriores.
* El controlador de dominio usado por Azure AD debe ser grabable. *No se admite* el uso de un controlador de dominio de solo lectura (RODC), y Azure AD Connect no sigue las redirecciones de escritura.
* *No se admite* el uso de bosques o dominios locales con nombres de NetBIOS con puntos (el nombre contiene un punto ".").
* Se recomienda [habilitar la papelera de reciclaje de Active Directory](how-to-connect-sync-recycle-bin.md).

### <a name="azure-ad-connect-server"></a>Servidor de Azure AD Connect
El servidor de Azure AD Connect contiene datos de identidad críticos. Es importante que el acceso administrativo a este servidor esté protegido correctamente. Siga las instrucciones de [Protección del acceso con privilegios](/windows-server/identity/securing-privileged-access/securing-privileged-access). 

El servidor de Azure AD Connect debería tratarse como un componente de nivel 0, tal como se documenta en el [modelo de nivel administrativo de Active Directory](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material). 

Para más información sobre cómo proteger el entorno de Active Directory, vea [Procedimientos recomendados para proteger Active Directory](/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory).

#### <a name="installation-prerequisites"></a>Requisitos previos de instalación

- Azure AD Connect debe estar instalado en una instancia de Windows Server 2012 o una versión posterior unida a dominio. 
- Azure AD Connect no puede instalarse en Small Business Server o Windows Server Essentials antes de 2019 (se admite Windows Server Essentials 2019). El servidor debe usar Windows Server estándar o una versión superior. 
- El servidor de Azure AD Connect debe tener una GUI completa instalada. No se admite la instalación de Azure AD Connect en Windows Server Core. 
- El servidor de Azure AD Connect no debe tener la directiva de grupo de transcripción de PowerShell habilitada si usa el Asistente de Azure AD Connect para administrar la configuración de los Servicios de federación de Active Directory (AD FS). Puede habilitar la transcripción de PowerShell si usa el asistente de Azure AD Connect para administrar la configuración de sincronización. 
- Si se implementa AD FS: 
    - los servidores en los que se instale AD FS o el Proxy de aplicación web deben ser Windows Server 2012 R2 o versiones posteriores. La Administración remota de Windows debe estar habilitada en estos servidores para la instalación remota. 
    - Debe configurar los certificados TLS/SSL. Para más información, vea [Administración de los protocolos SSL/TLS y los conjuntos de cifrado para AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs) y [Administrar certificados SSL en AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-certificates-ad-fs-wap).
    - Debe configurar la resolución de nombres. 
- Si los administradores globales tienen MFA habilitado, la dirección URL https://secure.aadcdn.microsoftonline-p.com *debe* estar en la lista de sitios de confianza. Se le solicita que agregue este sitio a la lista de sitios de confianza cuando se le pide un desafío MFA y no se ha agregado antes. Puede utilizar Internet Explorer para agregarla a los sitios de confianza.

#### <a name="harden-your-azure-ad-connect-server"></a>Configuración del servidor de Azure AD Connect 
Se recomienda reforzar el servidor de Azure AD Connect para reducir la superficie de ataque de seguridad para este componente crítico de su entorno de TI. La aplicación de estas recomendaciones lo ayudará a reducir algunos riesgos de seguridad de su organización.

- Trate Azure AD Connect igual que un controlador de dominio y otros recursos de nivel 0. Para más información, vea [Modelo de nivel administrativo de Active Directory](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material).
- Restrinja el acceso administrativo al servidor de Azure AD Connect a solo los administradores del dominio o a otros grupos de seguridad rigurosamente controlados.
- Cree una [cuenta dedicada para todo el personal con privilegios de acceso](/windows-server/identity/securing-privileged-access/securing-privileged-access). Los administradores no deben explorar la Web, consultar su correo electrónico y realizar las tareas de productividad cotidianas con cuentas con privilegios elevados.
- Siga las instrucciones que se proporcionan en [Protección del acceso con privilegios](/windows-server/identity/securing-privileged-access/securing-privileged-access). 
- Deniegue el uso de la autenticación NTLM con el servidor AADConnect. A continuación, se indican algunas maneras de hacerlo: [restringir NTLM en el servidor AADConnect](/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-outgoing-ntlm-traffic-to-remote-servers) y [restringir NTLM en un dominio](/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain)
- Asegúrese de que todas las máquinas tienen una contraseña de administrador local única. Para más información, vea cómo la [solución de contraseña de administrador local (LAPS)](https://support.microsoft.com/help/3062591/microsoft-security-advisory-local-administrator-password-solution-laps) puede configurar contraseñas aleatorias únicas en cada estación de trabajo y almacenarlas en Active Directory protegidas mediante una ACL. Solo los usuarios autorizados válidos pueden leer o solicitar el restablecimiento de estas contraseñas de cuenta de administrador local. Puede obtener las soluciones de contraseña de administrador local para su uso en estaciones de trabajo y servidores desde el [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=46899#:~:text=The%20%22Local%20Administrator%20Password%20Solution,it%20or%20request%20its%20reset.). Puede encontrar orientación adicional para el funcionamiento de un entorno con soluciones de contraseña de administrador local y estaciones de trabajo de acceso privilegiado en [Estándares operativos basados en el principio del origen limpio](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material#operational-standards-based-on-clean-source-principle). 
- Implemente [estaciones de trabajo de acceso privilegiado](/windows-server/identity/securing-privileged-access/privileged-access-workstations) dedicadas para todo el personal con acceso con privilegios a los sistemas de información de su organización. 
- Siga estas [instrucciones adicionales](/windows-server/identity/ad-ds/plan/security-best-practices/reducing-the-active-directory-attack-surface) para reducir la superficie expuesta a ataques de su entorno de Active Directory.


### <a name="sql-server-used-by-azure-ad-connect"></a>SQL Server usado por Azure AD Connect
* Azure AD Connect requiere una base de datos de SQL Server para almacenar datos de identidad. De forma predeterminada, se instala SQL Server 2012 Express LocalDB (versión ligera de SQL Server Express). SQL Server Express tiene un límite de tamaño de 10 GB que le permite administrar aproximadamente 100 000 objetos. Si tiene que administrar un volumen superior de objetos de directorio, es necesario que el asistente para la instalación apunte a otra instalación de SQL Server. El tipo de instalación de SQL Server puede afectar al [rendimiento de Azure AD Connect](./plan-connect-performance-factors.md#sql-database-factors).
* Si usa una instalación diferente de SQL Server, se aplican estos requisitos:
  * Azure AD Connect admite todas las versiones de SQL Server desde 2012 (con el Service Pack más reciente) hasta SQL Server 2019. Azure SQL Database *no se admite* como una base de datos.
  * Debe usar una intercalación de SQL sin distinción de mayúsculas y minúsculas. Estas intercalaciones se identifican porque el nombre incluye \_CI_. *No se admite* el uso de una intercalación que distinga mayúsculas de minúsculas identificada por \_CS_ en su nombre.
  * Solo se puede tener un motor de sincronización por cada instancia de SQL. *No se puede* compartir una instancia de SQL con FIM/MIM Sync, DirSync o la Sincronización de Azure AD.

### <a name="accounts"></a>Cuentas
* Debe tener una cuenta de administrador global de Azure AD para el inquilino de Azure AD con el que desea realizar la integración. Esta cuenta debe ser una *cuenta profesional o educativa* y no puede ser una *cuenta Microsoft*.
* Si usa la [configuración rápida](reference-connect-accounts-permissions.md#express-settings-installation) o actualiza desde DirSync, debe tener una cuenta de administrador de empresa para la instancia de Active Directory local.
* Si usa la ruta de acceso de instalación de configuración personalizada, dispone de más opciones. Para más información, consulte [Configuración de la instalación personalizada](reference-connect-accounts-permissions.md#custom-installation-settings).

### <a name="connectivity"></a>Conectividad
* El servidor de Azure AD Connect necesita resolución DNS para intranet e Internet. El servidor DNS debe ser capaz de resolver nombres en su Active Directory local y en los puntos de conexión de Azure AD.
* Si tiene firewalls en la intranet y necesita abrir puertos entre los servidores de Azure AD Connect y los controladores de dominio, vea [La identidad híbrida requería puertos y protocolos](reference-connect-ports.md) para más información.
* Si el proxy o firewall limita a qué direcciones URL se puede acceder, deben abrirse las direcciones URL que se documentan en [URL de Office 365 e intervalos de direcciones IP](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2).
  * Si usa Microsoft Cloud en Alemania o Microsoft Azure Government Cloud, vea [Azure AD Connect: consideraciones especiales para instancias](reference-connect-instances.md) para ver las direcciones URL.
* Azure AD Connect (versión 1.1.614.0 y posteriores) usa de forma predeterminada TLS 1.2 para cifrar la comunicación entre el motor de sincronización y Azure AD. Si TLS 1.2 no está disponible en el sistema operativo subyacente, Azure AD Connect recurre de mayor a menor a los protocolos anteriores (TLS 1.1 y TLS 1.0).
* Antes de la versión 1.1.614.0, Azure AD Connect usa de forma predeterminada TLS 1.0 para cifrar la comunicación entre el motor de sincronización y Azure AD. Para cambiar este protocolo a TLS 1.2, siga los pasos que se describen en [Habilitación de TLS 1.2 en Azure AD Connect](#enable-tls-12-for-azure-ad-connect).
* Si usa un proxy de salida para realizar la conexión a Internet, se tiene que agregar la siguiente configuración del archivo **C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config** con el fin de que el Asistente para instalación y la sincronización de Azure AD Connect se puedan conectar a Internet y Azure AD. Este texto debe escribirse en la parte inferior del archivo. En este código, *&lt;PROXYADDRESS&gt;* representa el nombre de host o la dirección IP de proxy real.

    ```
        <system.net>
            <defaultProxy>
                <proxy
                usesystemdefault="true"
                proxyaddress="http://<PROXYADDRESS>:<PROXYPORT>"
                bypassonlocal="true"
                />
            </defaultProxy>
        </system.net>
    ```

* Si el servidor proxy requiere autenticación, la [cuenta de servicio](reference-connect-accounts-permissions.md#adsync-service-account) debe encontrarse en el dominio. Use la ruta de instalación de configuración personalizada para especificar una [cuenta de servicio personalizada](how-to-connect-install-custom.md#install-required-components). También necesita otro cambio en el archivo machine.config. Con este cambio en el archivo machine.config, el asistente para instalación y el motor de sincronización responden a las solicitudes de autenticación del servidor proxy. En todas las páginas del Asistente para instalación, excepto la página **Configurar**, se usan las credenciales del usuario que ha iniciado sesión. En la página **Configurar** al final del Asistente para instalación, el contexto se cambia a la [cuenta de servicio](reference-connect-accounts-permissions.md#adsync-service-account) que creó. La sección del archivo machine.config debería tener este aspecto:

    ```
        <system.net>
            <defaultProxy enabled="true" useDefaultCredentials="true">
                <proxy
                usesystemdefault="true"
                proxyaddress="http://<PROXYADDRESS>:<PROXYPORT>"
                bypassonlocal="true"
                />
            </defaultProxy>
        </system.net>
    ```

* Si la configuración del proxy se realiza en una instalación existente, el **servicio de sincronización de Microsoft Azure AD** debe reiniciarse una vez para que Azure AD Connect lea la configuración del proxy y actualice el comportamiento. 
* Cuando Azure AD Connect envía una solicitud web a Azure AD como parte de la sincronización de directorios, Azure AD puede tardar hasta 5 minutos en responder. Es habitual que los servidores proxy tengan la configuración de tiempo de espera de inactividad de conexión. Asegúrese de que la configuración esté establecida en 6 minutos o más.

Para más información, vea el artículo de MSDN sobre el [elemento de proxy predeterminado](/dotnet/framework/configure-apps/file-schema/network/defaultproxy-element-network-settings).
Para obtener más información si tiene problemas de conectividad, consulte [Solución de problemas de conectividad](tshoot-connect-connectivity.md).

### <a name="other"></a>Otros
Opcional: use una cuenta de usuario de prueba para comprobar la sincronización.

## <a name="component-prerequisites"></a>Requisitos previos de los componentes
### <a name="powershell-and-net-framework"></a>PowerShell y .NET Framework
Azure AD Connect depende de Microsoft PowerShell y .NET Framework 4.5.1. Necesita que esta versión o una posterior esté instalada en el servidor. Dependiendo de la versión de Windows Server, realice estas acciones:

* Windows Server 2012 R2
  * Microsoft PowerShell se instala de forma predeterminada, No se requiere ninguna acción.
  * .NET Framework 4.5.1 y versiones posteriores se ofrecen mediante Windows Update. Asegúrese de que ha instalado las actualizaciones más recientes de Windows Server en el panel de control.
* Windows Server 2012
  * La última versión de Microsoft PowerShell está disponible en Windows Management Framework 4.0, que encontrará en el [Centro de descarga de Microsoft](https://www.microsoft.com/downloads).
  * .NET Framework 4.5.1 y versiones posteriores están disponibles en el [Centro de descarga de Microsoft](https://www.microsoft.com/downloads).


### <a name="enable-tls-12-for-azure-ad-connect"></a>Habilitación de TLS 1.2 en Azure AD Connect
Antes de la versión 1.1.614.0, Azure AD Connect usa de forma predeterminada TLS 1.0 para cifrar la comunicación entre el servidor de sincronización y Azure AD. Puede configurar las aplicaciones .NET para que empleen TLS 1.2 de forma predeterminada en el servidor. Para más información sobre TLS 1.2, vea el [aviso de seguridad de Microsoft 2960358](/security-updates/SecurityAdvisories/2015/2960358).

1. Asegúrese de que la revisión de .NET 4.5.1 esté instalada en el sistema operativo. Para más información, vea el [aviso de seguridad de Microsoft 2960358](/security-updates/SecurityAdvisories/2015/2960358). Puede que ya tenga esta revisión o una posterior instalada en el servidor.

1. Para todos los sistemas operativos, establezca esta clave del registro y reinicie el servidor.
    ```
    HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319
    "SchUseStrongCrypto"=dword:00000001
    ```
1. Si también desea habilitar TLS 1.2 entre el servidor del motor de sincronización y una instancia de SQL Server remota, asegúrese de que tiene instaladas las versiones necesarias para que [TLS 1.2 sea compatible con Microsoft SQL Server](https://support.microsoft.com/kb/3135244).

## <a name="prerequisites-for-federation-installation-and-configuration"></a>Requisitos previos para la instalación y la configuración de la federación
### <a name="windows-remote-management"></a>Administración remota de Windows
Cuando use Azure AD Connect para implementar AD FS o el proxy de aplicación web (WAP), compruebe estos requisitos:

* Si el servidor de destino está unido al dominio, compruebe que está habilitada la opción Administración remota de Windows.
  * En una ventana de comandos de PowerShell con privilegios elevados, use el comando `Enable-PSRemoting –force`.
* Si el servidor de destino es un equipo WAP no unido al dominio, hay un par de requisitos adicionales:
  * En el equipo de destino (equipo WAP):
    * Asegúrese de que el servicio Administración remota de Windows/ WS-Management (WinRM) se está ejecutando mediante el complemento Servicios.
    * En una ventana de comandos de PowerShell con privilegios elevados, use el comando `Enable-PSRemoting –force`.
  * En el equipo en el que se está ejecutando el asistente (si el equipo de destino no está unido al dominio o el dominio no es de confianza):
    * En una ventana de comandos de PowerShell con privilegios elevados, use el comando `Set-Item.WSMan:\localhost\Client\TrustedHosts –Value <DMZServerFQDN> -Force –Concatenate`.
    * En el Administrador de servidores:
      * Agregue un host WAP de red perimetral a un grupo de máquinas. En el Administrador de servidores, seleccione **Administrar** > **Agregar servidores** y, a continuación, use la pestaña **DNS**.
      * En la pestaña **Todos los servidores del Administrador de servidores**, haga clic con el botón derecho en el servidor WAP y seleccione **Administrar como**. Escriba las credenciales locales (no de dominio) para el equipo WAP.
      * Para validar la conectividad remota de PowerShell, en la pestaña **Todos los servidores del Administrador de servidores**, haga clic con el botón derecho en el servidor WAP y seleccione **Windows PowerShell**. Debe abrirse una sesión remota de PowerShell para asegurarse de que se pueden establecer sesiones remotas de PowerShell.

### <a name="tlsssl-certificate-requirements"></a>Requisitos del certificado TLS/SSL
* Se recomienda utilizar el mismo certificado TLS/SSL en todos los nodos de la granja de AD FS, así como todos los servidores del Proxy de aplicación web.
* El certificado debe ser del tipo X 509.
* Puede usar un certificado autofirmado en servidores de federación en un entorno de laboratorio de pruebas. Para un entorno de producción, se recomienda obtener el certificado de una entidad de certificación pública.
  * Si usa un certificado que no es de confianza pública, asegúrese de que el certificado instalado en cada servidor del Proxy de aplicación web sea de confianza tanto en el servidor local como en todos los servidores de federación.
* La identidad del certificado debe coincidir con el nombre del servicio de federación (por ejemplo, sts.contoso.com).
  * La identidad es una extensión de nombre alternativo del firmante (SAN) de tipo dNSName, o bien, si no hay ninguna entrada de SAN, el nombre del firmante se especifica como un nombre común.
  * Puede haber varias entradas de SAN en el certificado, siempre que una de ellas coincida con el nombre de servicio de federación.
  * Si piensa usar Workplace Join, se necesita un SAN adicional con el valor **enterpriseregistration**, seguido del sufijo de nombre principal de usuario (UPN) de su organización, por ejemplo, enterpriseregistration.contoso.com.
* No se admiten certificados basados en claves CryptoAPI Next Generation (CNG) ni en proveedores de almacenamiento de claves (KSP). Como resultado, debe usar un certificado basado en un proveedor de servicios de cifrado (CSP) y no un proveedor de almacenamiento de claves (KSP).
* Se admiten certificados comodín.

### <a name="name-resolution-for-federation-servers"></a>Resolución de nombres para los servidores de federación
* Configure registros DNS para el nombre de AD FS (por ejemplo, sts.contoso.com) para la intranet (servidor DNS interno) y la extranet (DNS público a través de su registrador de dominios). Para el registro DNS de la intranet, asegúrese de que usa registros D y no registros CNAME. Es necesario usar registros A para que funcione correctamente la autenticación de Windows desde la máquina unida al dominio.
* Si va a implementar más de un servidor de AD FS o servidor Proxy de aplicación web, asegúrese de que ha configurado el equilibrador de carga y que los registros DNS para el nombre de AD FS (por ejemplo, sts.contoso.com) apuntan al equilibrador de carga.
* Para que la autenticación integrada de Windows funcione para las aplicaciones del explorador que usan Internet Explorer en la intranet, asegúrese de que se agrega el nombre de AD FS (por ejemplo, sts.contoso.com) a la zona de intranet en Internet Explorer. Este requisito se puede controlar mediante la directiva de grupo e implementarlo en todos los equipos unidos a un dominio.

## <a name="azure-ad-connect-supporting-components"></a>Componentes de soporte de Azure AD Connect
Azure AD Connect instala los siguientes componentes en el servidor en el que está instalado Azure AD Connect. Esta lista es para una instalación rápida básica. Si decide usar una instancia de SQL Server diferente en la página **Instalar servicios de sincronización**, SQL Express LocalDB no se instalará localmente.

* Azure AD Connect Health
* Utilidades de línea de comandos de Microsoft SQL Server 2012
* Microsoft SQL Server 2012 Express LocalDB
* Microsoft SQL Server 2012 Native Client
* Paquete de redistribución de Microsoft Visual C++ 2013

## <a name="hardware-requirements-for-azure-ad-connect"></a>Requisitos de hardware de Azure AD Connect
En la tabla siguiente se muestran los requisitos mínimos de un equipo con sincronización de Azure AD Connect.

| Cantidad de objetos en Active Directory | CPU | Memoria | Tamaño de disco duro |
| --- | --- | --- | --- |
| Menos de 10.000 |1,6 GHz |4 GB |70 GB |
| 10.000–50.000 |1,6 GHz |4 GB |70 GB |
| 50.000–100.000 |1,6 GHz |16 GB |100 GB |
| Para 100 000 objetos o más, se requiere la versión completa de SQL Server | | | |
| 100.000–300.000 |1,6 GHz |32 GB |300 GB |
| 300.000–600.000 |1,6 GHz |32 GB |450 GB |
| Más de 600.000 |1,6 GHz |32 GB |500 GB |

Los requisitos mínimos para equipos que ejecutan AD FS o servidores Proxy de aplicación web son:

* CPU: 1,6 GHz con doble núcleo o superior
* Memoria: 2 GB o superior
* VM de Azure: configuración A2 o superior

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).