---
title: Implementación de la protección con contraseña de Azure AD local
description: Aprenda a planear e implementar la protección con contraseña de Azure AD en un entorno de Active Directory Domain Services local.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 03/05/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 759a5fa2be5a3df50160d2fd0ac4231c9f49329b
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718958"
---
# <a name="plan-and-deploy-on-premises-azure-active-directory-password-protection"></a>Planeación e implementación de la protección con contraseña de Azure Active Directory local

Los usuarios suelen crear contraseñas que usan palabras comunes locales, como una escuela, un equipo deportivo o una persona famosa. Estas contraseñas son fáciles de adivinar y poco seguras contra ataques basados en diccionarios. Para aplicar contraseñas seguras en su organización, la protección con contraseña de Azure Active Directory (Azure AD) proporciona una lista de contraseñas prohibidas global y personalizada. Se produce un error en una solicitud de cambio de contraseña si hay una coincidencia en esta lista de contraseñas prohibidas.

Para proteger el entorno de Active Directory Domain Services (AD DS) local, puede instalar y configurar la protección con contraseña de Azure AD para que funcione con el controlador de dominio local. En este artículo se muestra cómo instalar y registrar el servicio de proxy de protección con contraseña de Azure AD y el agente de controlador de dominio de protección con contraseña de Azure AD en el entorno local.

Para más información sobre cómo funciona la protección con contraseña de Azure AD en un entorno local, consulte [Aplicación de Protección con contraseña de Azure AD para Windows Server Active Directory](concept-password-ban-bad-on-premises.md).

## <a name="deployment-strategy"></a>Estrategia de implementación

En el diagrama siguiente se muestra cómo funcionan conjuntamente los componentes básicos de la protección con contraseña de Azure AD en un entorno de Active Directory local:

![Funcionamiento conjunto de los componentes de la protección con contraseña de Azure AD](./media/concept-password-ban-bad-on-premises/azure-ad-password-protection.png)

Es una buena idea revisar cómo funciona el software antes de implementarlo. Para más información, consulte la [introducción al concepto de la protección con contraseña de Azure AD](concept-password-ban-bad-on-premises.md).

Le recomendamos que inicie las implementaciones en modo de *auditoría*. El modo de auditoría es el valor inicial predeterminad, donde se puede seguir estableciendo contraseñas. Las contraseñas que estuvieran bloqueadas se registran en el registro de eventos. Después de implementar los servidores proxy y los agentes de controlador de dominio en el modo de auditoría, supervise el impacto que tendrá la directiva de contraseñas en los usuarios al aplicar la directiva.

Durante la fase de auditoría, numerosas organizaciones han observado que se aplican las siguientes situaciones:

* Necesitan mejorar los procesos operativos existentes para usar contraseñas más seguras.
* Los usuarios suelen usan contraseñas no seguras.
* Debe informar a los usuarios sobre el próximo cambio en el cumplimiento de seguridad, el posible impacto para ellos y cómo elegir contraseñas más seguras.

También es posible que la validación de contraseñas más seguras afecte a la automatización de la implementación de controladores de dominio de la instancia de Active Directory existente. Es aconsejable que se produzcan, al menos, una promoción y una degradación de controladores de dominio durante la evaluación del período de auditoría para ayudar a descubrir dichos problemas. Para más información, consulte los siguientes artículos.

* [Ntdsutil. exe no puede establecer una contraseña no segura de Modo de reparación de servicios de directorio](howto-password-ban-bad-on-premises-troubleshoot.md#ntdsutilexe-fails-to-set-a-weak-dsrm-password)
* [No se puede realizar la promoción de la réplica del controlador de dominio debido a una contraseña de Modo de reparación de servicios de directorio no segura](howto-password-ban-bad-on-premises-troubleshoot.md#domain-controller-replica-promotion-fails-because-of-a-weak-dsrm-password)
* [No se puede realizar la degradación del controlador de dominio debido a una contraseña no segura del administrador local](howto-password-ban-bad-on-premises-troubleshoot.md#domain-controller-demotion-fails-due-to-a-weak-local-administrator-password)

Después de que la característica se haya ejecutado en modo de auditoría durante un período razonable, puede cambiar la configuración de *Auditoría* a *Exigir* para requerir contraseñas más seguras. Es conveniente enfocar una supervisión adicional durante dicho período.

### <a name="multiple-forest-considerations"></a>Consideraciones sobre varios bosques

No hay ningún requisito adicional para implementar la protección con contraseña de Azure AD en varios bosques.

Cada bosque se configura de forma independiente, tal como se describe en la sección siguiente para [implementar la protección con contraseña de Azure AD local](#download-required-software). Cada proxy de protección con contraseña de Azure AD solo puede admitir controladores de dominio del bosque al que está unido.

El software de protección con contraseña de Azure AD de cualquier bosque desconoce el software de protección con contraseña que se implementa en otros bosques, independientemente de las configuraciones de confianza de Active Directory.

### <a name="read-only-domain-controller-considerations"></a>Consideraciones sobre controladores de dominio de solo lectura

Los eventos de cambios o establecimientos de contraseña no se procesan ni conservan en los controladores de dominio de solo lectura (RODC). En su lugar, se reenvían a controladores de dominio de escritura. No tiene que instalar el software del agente de controlador de dominio de protección con contraseña de Azure AD en RODC.

Además, no se admite la ejecución del servicio de proxy de protección con contraseña de Azure AD en un controlador de dominio de solo lectura.

### <a name="high-availability-considerations"></a>Consideraciones sobre la alta disponibilidad

El problema principal para la protección con contraseña es la disponibilidad de los servidores proxy de protección con contraseña de Azure AD cuando los controladores de dominio de un bosque intentan descargar nuevas directivas u otros datos de Azure. Cada agente de controlador de dominio de protección con contraseña de Azure AD usa un algoritmo de tipo round-robin simple al decidir a qué servidor proxy llamar. El agente omite los servidores proxy que no responden.

Para la mayoría de las implementaciones de Active Directory completamente conectadas que tengan una replicación correcta del estado del directorio y de la carpeta sysvol, dos servidores proxy de protección con contraseña de Azure AD son suficientes para garantizar la disponibilidad. Esta configuración da como resultado la descarga oportuna de nuevas directivas y otros datos. Si lo desea, puede implementar servidores proxy de protección con contraseña de Azure AD adicionales.

El diseño del software del agente de controlador de dominio de protección con contraseña de Azure AD mitiga los problemas habituales asociados con la alta disponibilidad. El agente de controlador de dominio de protección con contraseña de Azure AD mantiene una memoria caché local de la directiva de contraseñas descargada más recientemente. Incluso si todos los servidores proxy registrados dejan de estar disponibles, los agentes de controlador de dominio de protección con contraseña de Azure AD siguen aplicando su directiva de contraseñas en caché.

Una frecuencia de actualización razonable de las directivas de contraseñas en una implementación de gran tamaño suele ser días, no horas o menos. Por lo tanto, las interrupciones breves de los servidores proxy no afectan significativamente a la protección con contraseña de Azure AD.

## <a name="deployment-requirements"></a>Requisitos de implementación

Para más información sobre las licencias, consulte los [requisitos de licencia de protección con contraseña de Azure AD](concept-password-ban-bad.md#license-requirements).

Se aplican los siguientes requisitos principales:

* Todas las maquinas, incluidos los controladores de dominio, en las que se instalen componentes de protección con contraseña de Azure AD deben tener instalado Universal C Runtime.
    * Puede obtener el tiempo de ejecución al asegurarse de que tenga todas las actualizaciones de Windows Update. O bien, puede obtenerlo en un paquete de actualización del sistema operativo específico. Para más información, consulte [Actualización para Universal C RunTime en Windows](https://support.microsoft.com/help/2999226/update-for-uniersal-c-runtime-in-windows).
* Necesita una cuenta con privilegios de administrador de dominio de Active Directory en el dominio raíz del bosque para registrar el bosque de Windows Server Active Directory en Azure AD.
* El servicio de distribución de claves debe habilitarse en todos los controladores de dominio del dominio que ejecutan Windows Server 2012. De manera predeterminada, este servicio se habilita a través de inicio de un desencadenador manual.
* Debe existir conectividad de red entre al menos un controlador de dominio de cada dominio y un servidor que hospede el servicio de proxy para la protección con contraseña de Azure AD. Esta conectividad debe permitir que el controlador de dominio acceda al puerto 135 del asignador de puntos de conexión RPC y al puerto del servidor RPC del servicio de proxy.
    * De manera predeterminada, el puerto del servidor RPC es un puerto RPC dinámico, pero se puede configurar para [usar un puerto estático](#static).
* Todas las máquinas donde se instalará el servicio de proxy de Protección con contraseña de Azure AD deben tener acceso de red a los puntos de conexión siguientes:

    |**Punto de conexión**|**Propósito**|
    | --- | --- |
    |`https://login.microsoftonline.com`|Solicitudes de autenticación|
    |`https://enterpriseregistration.windows.net`|Funcionalidad de protección con contraseña de Azure AD|

### <a name="azure-ad-password-protection-dc-agent"></a>Agente de controlador de dominio de protección con contraseña de Azure AD

Los siguientes requisitos se aplican al agente de controlador de dominio de protección con contraseña de Azure AD:

* Todas las máquinas donde se instale el software del agente de controlador de dominio de protección con contraseña de Azure AD deben ejecutar Windows Server 2012 o posterior.
    * El dominio o el bosque de Active Directory no tiene necesariamente que estar en el nivel funcional del dominio (DFL) o del bosque (FFL) de Windows Server 2012. Tal como se mencionó en [Principios de diseño](concept-password-ban-bad-on-premises.md#design-principles), no se necesita ningún DFL o FFL mínimo para la ejecución del agente de controlador de dominio o software de proxy.
* Todas las maquinas que ejecutan el agente de controlador de dominio de protección con contraseña de Azure AD deben tener .NET 4.5 instalado.
* Cualquier dominio de Active Directory que ejecute el servicio de agente de controlador de dominio de protección con contraseña de Azure AD debe usar la replicación del Sistema de archivos distribuido (DFSR) para la replicación de sysvol.
   * Si el dominio no usa aún DFSR, debe migrarlo antes de instalar la protección con contraseña de Azure AD. Para más información, consulte la [guía de migración de la replicación de SYSVOL: Replicación de FRS a DFS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd640019(v=ws.10))

    > [!WARNING]
    > El software del agente de controlador de dominio de protección con contraseña de Azure AD se instalará actualmente en los controladores de dominio de los dominios que usan aún FRS (la tecnología predecesora a DFSR) para la replicación de sysvol, pero NO funcionará correctamente en este entorno.
    >
    > Otros efectos secundarios negativos incluyen archivos que no se pueden replicar y procedimientos de restauración de SYSVOL que aparentemente funcionan pero que en realidad no pueden replicar todos los archivos.
    >
    > Migre el dominio para usar DFSR lo antes posible, tanto por las ventajas inherentes de DFSR como para desbloquear la implementación de la protección con contraseña de Azure AD. Las versiones futuras del software se deshabilitarán automáticamente cuando se ejecuten en un dominio que aún use FRS.

### <a name="azure-ad-password-protection-proxy-service"></a>Servicio de proxy de protección con contraseña de Azure AD

Los siguientes requisitos se aplican al servicio de proxy de protección con contraseña de Azure AD:

* Todas las máquinas en las que se instale el servicio de proxy de protección con contraseña de Azure AD deben ejecutar Windows Server 2012 R2 o posterior.

    > [!NOTE]
    > La implementación del servicio de proxy de protección con contraseña de Azure AD es un requisito obligatorio para la implementación de protección con contraseña de Azure AD, aunque el controlador de dominio pueda tener conectividad saliente directa a Internet.

* Todas las máquinas en las que se instale el servicio de proxy de protección con contraseña de Azure AD deben tener .NET 4.7 instalado.
    * NET 4.7 ya debería estar instalado en un servidor Windows completamente actualizado. Si es necesario, descargue y ejecute el instalador que se encuentra en [ El instalador fuera de línea de .NET Framework 4.7 para Windows ](https://support.microsoft.com/help/3186497/the-net-framework-4-7-offline-installer-for-windows).
* Todas las máquinas que hospedan el servicio de proxy de protección con contraseña de Azure AD deben estar configuradas para conceder a los controladores de dominio la posibilidad de iniciar sesión en el servicio de proxy. Esta capacidad se controla a través de la asignación del privilegio "Tener acceso a este equipo desde la red".
* Todas las máquinas que hospedan el servicio de proxy de protección con contraseña de Azure AD deben estar configuradas para permitir el tráfico TLS 1.2 HTTP de salida.
* Una cuenta de *administrador global* para registrar el servicio de proxy de protección con contraseña de Azure AD y el bosque con Azure AD.
* El acceso a la red debe estar habilitado para el conjunto de puertos y direcciones URL especificados en los [procedimientos de configuración del entorno de proxy de aplicación](../manage-apps/application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment).

### <a name="microsoft-azure-ad-connect-agent-updater-prerequisites"></a>Requisitos previos de Agent Updater de Microsoft Azure AD Connect

El servicio de actualización de Agent Updater de Microsoft Azure AD Connect se instala junto con el servicio Proxy de protección de contraseña de Azure AD. Se requiere una configuración adicional para que el servicio de Agent Updater de Microsoft Azure AD Connect pueda funcionar:

* Si su entorno utiliza un servidor proxy HTTP, debe seguir las pautas especificadas en [Trabajo con servidores proxy locales existentes](../manage-apps/application-proxy-configure-connectors-with-proxy-servers.md).
* El servicio Agent Updater de Microsoft Azure AD Connect también requiere los pasos de TLS 1.2 especificados en [Requisitos de TLS](../manage-apps/application-proxy-add-on-premises-application.md#tls-requirements).

> [!WARNING]
> El proxy de protección con contraseña de Azure AD y Azure AD Application Proxy instalan diferentes versiones del servicio de Agent Updater de Microsoft Azure AD Connect, por lo que las instrucciones se refieren al contenido de Application Proxy. Estas versiones diferentes son incompatibles cuando se instalan en paralelo y, si lo hace, impedirá que el servicio de actualización del agente se ponga en contacto con Azure para las actualizaciones de software, por lo que nunca debe instalar el servicio proxy de protección de contraseña de Azure AD y Application Proxy en el mismo equipo.

## <a name="download-required-software"></a>Descarga del software necesario

Hay dos instaladores requeridos para una implementación de la protección con contraseña de Azure AD local:

* Agente de controlador de dominio de protección con contraseña de Azure AD (*AzureADPasswordProtectionDCAgentSetup.msi*)
* Proxy de protección con contraseña de Azure AD (*AzureADPasswordProtectionProxySetup.exe*)

Descargue ambos instaladores en el [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=57071).

## <a name="install-and-configure-the-proxy-service"></a>Instalación y configuración del servicio de proxy

El servicio de proxy de protección con contraseña de Azure AD suele estar en un servidor miembro del entorno de AD DS local. Una vez instalado, el servicio de proxy de protección con contraseña de Azure AD se comunica con Azure AD para mantener una copia de las listas de contraseñas prohibidas globales y personalizadas del cliente para el inquilino de Azure AD.

En la siguiente sección, instalará los agentes de controlador de dominio de protección con contraseña de Azure AD en los controladores de dominio del entorno de AD DS local. Estos agentes de controlador de dominio se comunican con el servicio de proxy para obtener las listas de contraseñas prohibidas más recientes para utilizarlas al procesar eventos de cambio de contraseña dentro del dominio.

Elija uno o varios servidores para hospedar el servicio de proxy de protección con contraseña de Azure AD. Las siguientes consideraciones se aplican a este servidor o servidores:

* Cada servicio de este tipo solo puede proporcionar directivas de contraseñas para un único bosque. El equipo host debe estar unido a un dominio de ese bosque. Se admiten los dominios raíz y secundarios. Se necesita conectividad de red entre al menos un controlador de dominio en cada dominio del bosque y el equipo con protección con contraseña.
* Puede ejecutar el servicio de proxy de protección con contraseña de Azure AD en un controlador de dominio para pruebas, pero este controlador de dominio requiere conectividad a Internet. Esta conectividad puede ser un problema de seguridad. Esta configuración es recomendable solo para realizar pruebas.
* Se recomienda al menos dos servidores proxy de protección con contraseña de Azure AD para la redundancia, tal como se indica en la sección anterior acerca de las [consideraciones sobre la alta disponibilidad](#high-availability-considerations).
* No se admite la ejecución del servicio de proxy de protección con contraseña de Azure AD en un controlador de dominio de solo lectura.

Para instalar el servicio de proxy de protección con contraseña de Azure AD, siga estos pasos:

1. Para instalar el servicio de proxy de protección con contraseña de Azure AD, ejecute el instalador de software `AzureADPasswordProtectionProxySetup.exe`.

    La instalación de software no requiere un reinicio y se puede automatizar mediante procedimientos estándar de MSI, como en el ejemplo siguiente:
    
    ```console
    AzureADPasswordProtectionProxySetup.exe /quiet
    ```
    
    > [!NOTE]
    > Para evitar errores de instalación, el servicio Firewall de Windows debe estar en ejecución antes de instalar el paquete `AzureADPasswordProtectionProxySetup.exe`.
    >
    > Si la instancia de Windows Firewall está configurada para no ejecutarse, la solución alternativa es habilitar temporalmente el servicio de firewall y ejecutarlo durante la instalación. El software de proxy no tiene ninguna dependencia específica de Firewall de Windows después de la instalación.
    >
    > Si usa un firewall de terceros, aún debe configurarse para satisfacer los requisitos de implementación. Entre ellos se incluye permitir el acceso de entrada en el puerto 135 y el puerto RPC del servidor proxy. Para más información, consulte la sección anterior sobre los [requisitos de implementación](#deployment-requirements).

1. El software del proxy de protección con contraseña de Azure AD incluye un nuevo módulo de PowerShell, `AzureADPasswordProtection`. Los pasos siguientes ejecutan distintos cmdlets de este módulo de PowerShell.

    Para usar este módulo, abra una ventana de PowerShell como administrador e importe el nuevo módulo de la siguiente manera:
    
    ```powershell
    Import-Module AzureADPasswordProtection
    ```

1. Para comprobar que el servicio de proxy de protección con contraseña de Azure AD se está ejecutando, use el siguiente comando de PowerShell:

    ```powershell
    Get-Service AzureADPasswordProtectionProxy | fl
    ```

    El resultado debería mostrar el **estado** *Running*.

1. El servicio de proxy se ejecuta en la máquina, pero aún no dispone de credenciales para comunicarse con Azure AD. Registre el servidor proxy de protección con contraseña de Azure AD con Azure AD mediante el cmdlet `Register-AzureADPasswordProtectionProxy`.

    Este cmdlet requiere credenciales de administrador global para el inquilino de Azure. También necesita privilegios de administrador de dominio de Active Directory de forma local en el dominio raíz del bosque. Este cmdlet también se debe ejecutar mediante una cuenta con privilegios de administrador local:

    Cuando este comando se ejecute correctamente una vez para un servicio de proxy de protección con contraseña de Azure AD, las invocaciones adicionales se realizarán correctamente, pero no son necesarias.

    El cmdlet `Register-AzureADPasswordProtectionProxy` admite los siguientes tres modos de autenticación. Los dos primeros modos son compatibles con Azure Multi-Factor Authentication, pero el tercero no.

    > [!TIP]
    > Puede haber un retraso notable antes de completarse la primera vez que se ejecuta este cmdlet para un inquilino de Azure específico. A menos que se notifica un error, no se preocupe por este retraso.

     * Modo de autenticación interactiva:

        ```powershell
        Register-AzureADPasswordProtectionProxy -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com'
        ```

        > [!NOTE]
        > Este modo no funcionará en sistemas operativos Server Core. En su lugar, use uno de los siguientes modos de autenticación. Además, es posible que se produzca un error en este modo si se habilita la configuración de seguridad mejorada de Internet Explorer. La solución alternativa consiste en deshabilitar esa configuración, registrar el servidor proxy y luego volver a habilitarla.

     * Modo de autenticación con codificación del dispositivo:

        ```powershell
        Register-AzureADPasswordProtectionProxy -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com' -AuthenticateUsingDeviceCode
        ```

        Cuando se le solicite, siga el vínculo para abrir un explorador web y escriba el código de autenticación.

     * Modo de autenticación silenciosa (basada en contraseña):

        ```powershell
        $globalAdminCredentials = Get-Credential
        Register-AzureADPasswordProtectionProxy -AzureCredential $globalAdminCredentials
        ```

        > [!NOTE]
        > Se produce un error en este modo si se requiere Azure Multi-Factor Authentication para su cuenta. En ese caso, use uno de los dos modos de autenticación anteriores, o use una cuenta diferente que no requiera MFA.
        >
        > Puede que también vea que es necesaria la MFA si el registro de dispositivos de Azure (que usa en segundo plano la protección con contraseña de Azure AD) se ha configurado para requerir MFA de forma global. Para solucionar este requisito, puede usar una cuenta diferente que admita MFA con uno de los dos modos de autenticación anteriores, o bien también puede relajar temporalmente el requisito de MFA de registro de dispositivos de Azure.
        >
        > Para realizar este cambio, busque y seleccione **Azure Active Directory** en Azure Portal y, a continuación, seleccione **Dispositivos > Configuración de dispositivo**. Establezca **Requerir Multi-factor Authentication para conectar dispositivos** en *No*. Asegúrese de volver a configurar esta opción en *Sí* una vez que se haya completado el registro.
        >
        > Se recomienda omitir los requisitos de MFA solo con fines de prueba.

    Actualmente, no se requiere especificar el parámetro *-ForestCredential*, que está reservado para una futura funcionalidad.

    El registro del servicio de proxy de protección con contraseña de Azure AD solo se debe hacer una vez a lo largo del ciclo de vida del servicio. A partir de aquí, el servicio de proxy de protección con contraseña de Azure AD realizará automáticamente todas las demás tareas de mantenimiento necesarias.

1. Registre ahora el bosque de Active Directory local con las credenciales necesarias para comunicarse con Azure mediante el cmdlet `Register-AzureADPasswordProtectionForest` de PowerShell.

    > [!NOTE]
    > Si hay varios servidores proxy de protección con contraseña de Azure AD instalados en el entorno, da igual el servidor proxy que utilice para registrar el bosque.

    Este cmdlet requiere credenciales de administrador global para el inquilino de Azure. También debe ejecutar este cmdlet mediante una cuenta con privilegios de administrador local. También requiere privilegios de administrador de empresa de Active Directory local. Este paso se ejecuta una vez por bosque.

    El cmdlet `Register-AzureADPasswordProtectionForest` admite los siguientes tres modos de autenticación. Los dos primeros modos son compatibles con Azure Multi-Factor Authentication, pero el tercero no.

    > [!TIP]
    > Puede haber un retraso notable antes de completarse la primera vez que se ejecuta este cmdlet para un inquilino de Azure específico. A menos que se notifica un error, no se preocupe por este retraso.

     * Modo de autenticación interactiva:

        ```powershell
        Register-AzureADPasswordProtectionForest -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com'
        ```

        > [!NOTE]
        > Este modo no funcionará en sistemas operativos Server Core. En su lugar, use uno de los dos siguientes modos de autenticación. Además, es posible que se produzca un error en este modo si se habilita la configuración de seguridad mejorada de Internet Explorer. La solución alternativa consiste en deshabilitar esa configuración, registrar el bosque y luego volver a habilitarla.  

     * Modo de autenticación con codificación del dispositivo:

        ```powershell
        Register-AzureADPasswordProtectionForest -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com' -AuthenticateUsingDeviceCode
        ```

        Cuando se le solicite, siga el vínculo para abrir un explorador web y escriba el código de autenticación.

     * Modo de autenticación silenciosa (basada en contraseña):

        ```powershell
        $globalAdminCredentials = Get-Credential
        Register-AzureADPasswordProtectionForest -AzureCredential $globalAdminCredentials
        ```

        > [!NOTE]
        > Se produce un error en este modo si se requiere Azure Multi-Factor Authentication para su cuenta. En ese caso, use uno de los dos modos de autenticación anteriores, o use una cuenta diferente que no requiera MFA.
        >
        > Puede que también vea que es necesaria la MFA si el registro de dispositivos de Azure (que usa en segundo plano la protección con contraseña de Azure AD) se ha configurado para requerir MFA de forma global. Para solucionar este requisito, puede usar una cuenta diferente que admita MFA con uno de los dos modos de autenticación anteriores, o bien también puede relajar temporalmente el requisito de MFA de registro de dispositivos de Azure.
        >
        > Para realizar este cambio, busque y seleccione **Azure Active Directory** en Azure Portal y, a continuación, seleccione **Dispositivos > Configuración de dispositivo**. Establezca **Requerir Multi-factor Authentication para conectar dispositivos** en *No*. Asegúrese de volver a configurar esta opción en *Sí* una vez que se haya completado el registro.
        >
        > Se recomienda omitir los requisitos de MFA solo con fines de prueba.

       Los ejemplos anteriores solo generarán un resultado correcto si el usuario que haya iniciado sesión también es un administrador de dominios de Active Directory para el dominio raíz. De no ser así, puede proporcionar credenciales de dominio alternativas a través del parámetro *-ForestCredential*.

    El registro del bosque de Active Directory es se tiene que hacer una vez durante el ciclo de vida del bosque. A partir de aquí, los agentes de controlador de dominio de protección con contraseña de Azure AD en el bosque realizan automáticamente todas las demás tareas de mantenimiento necesarias. Después de que `Register-AzureADPasswordProtectionForest` se ejecute correctamente para un bosque, las invocaciones adicionales del cmdlet se realizan correctamente, pero no son necesarias.
    
    Para que `Register-AzureADPasswordProtectionForest` funcione correctamente, debe haber al menos un controlador de dominio que ejecute Windows Server 2012 o una versión posterior disponible en el dominio del servidor proxy de protección con contraseña de Azure AD. No hace falta instalar el software de agente de controlador de dominio de protección con contraseña de Azure AD en ningún controlador de dominio antes de este paso.

### <a name="configure-the-proxy-service-to-communicate-through-an-http-proxy"></a>Configuración del servicio de proxy para que se comunique mediante un proxy HTTP

Si su entorno requiere el uso de un proxy HTTP específico para comunicarse con Azure, siga estos pasos para configurar el servicio de protección con contraseña de Azure AD.

Cree un archivo *AzureADPasswordProtectionProxy.exe.config* en la carpeta `%ProgramFiles%\Azure AD Password Protection Proxy\Service`. Incluya el siguiente contenido:

   ```xml
   <configuration>
      <system.net>
         <defaultProxy enabled="true">
         <proxy bypassonlocal="true"
            proxyaddress="http://yourhttpproxy.com:8080" />
         </defaultProxy>
      </system.net>
   </configuration>
   ```

Si el proxy de HTTP requiere autenticación, agregue la etiqueta *useDefaultCredentials*:

   ```xml
   <configuration>
      <system.net>
         <defaultProxy enabled="true" useDefaultCredentials="true">
         <proxy bypassonlocal="true"
            proxyaddress="http://yourhttpproxy.com:8080" />
         </defaultProxy>
      </system.net>
   </configuration>
   ```

En ambos casos, reemplace `http://yourhttpproxy.com:8080` con la dirección y el puerto del servidor proxy HTTP específico.

Si el proxy HTTP se configuró para usar una directiva de autorización, debe conceder acceso a la cuenta del equipo de Active Directory del equipo que hospeda el servicio de proxy para la protección con contraseña.

Se recomienda detener y reiniciar el servicio de proxy de protección con contraseña de Azure AD después de crear o actualizar el archivo *AzureADPasswordProtectionProxy.exe.config*.

El servicio de proxy no admite el uso de credenciales específicas para conectarse a un servidor proxy de HTTP.

### <a name="configure-the-proxy-service-to-listen-on-a-specific-port"></a>Configuración del servicio de proxy para la escucha en un puerto específico

El software de agente de controlador de dominio de protección con contraseña de Azure AD usa RPC a través de TCP para comunicarse con el servicio de proxy. De forma predeterminada, el servicio de proxy de protección con contraseña de Azure AD escucha en cualquier punto de conexión RPC dinámico disponible. Puede configurar el servicio de modo que escuche en un puerto TCP específico, si fuera necesario debido a la topología de red o los requisitos de firewall del entorno.

<a id="static" /></a>Para configurar el servicio de modo que se ejecute en un puerto estático, use el cmdlet `Set-AzureADPasswordProtectionProxyConfiguration` del modo siguiente:

```powershell
Set-AzureADPasswordProtectionProxyConfiguration –StaticPort <portnumber>
```

> [!WARNING]
> Debe detener y reiniciar el servicio de proxy de protección con contraseña de Azure AD para que estos cambios surtan efecto.

Para configurar el servicio de modo que se ejecute en un puerto dinámico, aplique el mismo procedimiento, pero vuelva a establecer el valor de *Puerto estático* en cero:

```powershell
Set-AzureADPasswordProtectionProxyConfiguration –StaticPort 0
```

> [!WARNING]
> Debe detener y reiniciar el servicio de proxy de protección con contraseña de Azure AD para que estos cambios surtan efecto.

El servicio de proxy de protección con contraseña de Azure AD requiere un reinicio manual después de cualquier cambio de configuración del puerto. No tendrá que reiniciar el servicio de agente de controlador de dominio de protección con contraseña de Azure AD en los controladores de dominio después de realizar estos cambios de configuración.

Para consultar la configuración actual del servicio, use el cmdlet `Get-AzureADPasswordProtectionProxyConfiguration` tal como se muestra en el ejemplo siguiente:

```powershell
Get-AzureADPasswordProtectionProxyConfiguration | fl
```

El siguiente ejemplo de salida muestra que el servicio de proxy de protección con contraseña de Azure AD usa un puerto dinámico:

```output
ServiceName : AzureADPasswordProtectionProxy
DisplayName : Azure AD password protection Proxy
StaticPort  : 0
```

## <a name="install-the-dc-agent-service"></a>Instalación del servicio de agente de controlador de dominio

Para instalar el servicio de agente de controlador de dominio de protección con contraseña de Azure AD, ejecute el paquete `AzureADPasswordProtectionDCAgentSetup.msi`.

Puede automatizar la instalación de software mediante procedimientos estándares de MSI, tal como se muestra en el ejemplo siguiente:

```console
msiexec.exe /i AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn /norestart
```

Se puede omitir la marca `/norestart` si prefiere que el instalador reinicie automáticamente la máquina.

La instalación de software, o la desinstalación, requiere un reinicio. Este requisito se debe a que los archivos DLL de filtro de contraseña solo se cargan o descargan al reiniciar.

La instalación de la protección con contraseña de Azure AD local se habrá completado después de que se haya instalado el software de agente de controlador de dominio en un controlador de dominio y ese equipo se reinicie. No se requiere ni se permite ninguna otra configuración. Los eventos de cambio de contraseña en los controladores de dominio locales usan las listas de contraseñas prohibidas configuradas de Azure AD.

Para habilitar la protección con contraseña de Azure AD local desde Azure Portal o configurar contraseñas prohibidas personalizadas, consulte [Procedimientos operativos de protección con contraseña de Azure AD](howto-password-ban-bad-on-premises-operations.md).

> [!TIP]
> Puede instalar el agente de controlador de dominio de protección con contraseña de Azure AD en una máquina que todavía no sea un controlador de dominio. En este caso, el servicio se inicia y ejecuta, pero estará inactivo hasta que la máquina se promueva para que se convierta en un controlador de dominio.

## <a name="upgrading-the-proxy-service"></a>Actualización del servicio de proxy

El servicio de proxy de protección con contraseña de Azure AD admite la actualización automática. La actualización automática utiliza el servicio de Agent Updater de Microsoft Azure AD Connect, que se instala junto con el servicio de proxy. La actualización automática está activada de forma predeterminada y puede habilitarse o deshabilitarse mediante el cmdlet `Set-AzureADPasswordProtectionProxyConfiguration`.

La configuración actual se puede consultar mediante el cmdlet `Get-AzureADPasswordProtectionProxyConfiguration`. Se recomienda que la configuración de actualización automática siempre esté habilitada.

El cmdlet `Get-AzureADPasswordProtectionProxy` se puede usar para consultar la versión del software de todos los servidores proxy de protección con contraseña de Azure AD instalados actualmente en un bosque.

### <a name="manual-upgrade-process"></a>Proceso de actualización manual

Una actualización manual se realiza mediante la ejecución de la versión más reciente del instalador de software `AzureADPasswordProtectionProxySetup.exe`. La versión más reciente del software está disponible en el [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=57071).

No es necesario desinstalar la versión actual del servicio de proxy de protección con contraseña de Azure AD: el instalador realiza una actualización en contexto. Cuando se actualiza el servicio de proxy no es preciso reiniciar. La actualización del software puede automatizarse mediante procedimientos MSI estándar, como `AzureADPasswordProtectionProxySetup.exe /quiet`.

## <a name="upgrading-the-dc-agent"></a>Actualización del agente de controlador de dominio

Cuando haya disponible una versión más reciente del software de agente de controlador de dominio de protección con contraseña de Azure AD, la actualización se realiza mediante la ejecución de la versión más reciente del paquete de software `AzureADPasswordProtectionDCAgentSetup.msi`. La versión más reciente del software está disponible en el [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=57071).

No es necesario desinstalar la versión actual del software de agente de controlador de dominio: el instalador realiza una actualización en contexto. Al actualizar el software del agente de controlador de dominio siempre es preciso reiniciar el equipo, este requisito es causado por el comportamiento principal de Windows.

La actualización del software puede automatizarse mediante procedimientos MSI estándar, como `msiexec.exe /i AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn /norestart`.

Puede omitir la marca `/norestart` si prefiere que el instalador reinicie automáticamente el equipo.

El cmdlet `Get-AzureADPasswordProtectionDCAgent` se puede usar para consultar la versión del software de todos los agentes de controlador de dominio de protección con contraseña de Azure AD instalados actualmente en un bosque.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya ha instalado los servicios necesarios para la protección con contraseña de Azure AD en los servidores locales, [habilite la protección con contraseña de Azure AD local en Azure Portal](howto-password-ban-bad-on-premises-operations.md) para completar la implementación.