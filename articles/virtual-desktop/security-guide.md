---
title: 'Procedimientos recomendados de seguridad para Windows Virtual Desktop: Azure'
description: Procedimientos recomendados para proteger su entorno de Windows Virtual Desktop.
author: heidilohr
ms.topic: conceptual
ms.date: 05/07/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: a194074b75a404e5a28e86015b0d0bcea2619fc2
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88002294"
---
# <a name="security-best-practices"></a>Recomendaciones de seguridad

Windows Virtual Desktop es un servicio de escritorio virtual administrado que incluye muchas funcionalidades de seguridad para mantener protegida su organización. En una implementación de Windows Virtual Desktop, Microsoft administra parte de los servicios en nombre del cliente. El servicio tiene muchas características de seguridad avanzadas integradas, como la conexión inversa, que reduce el riesgo que conlleva tener acceso a los escritorios remotos desde cualquier lugar.

En este artículo se describen las medidas adicionales que puede tomar como administrador para proteger las implementaciones de Windows Virtual Desktop de sus clientes.

## <a name="security-responsibilities"></a>Responsabilidades de seguridad

La diferencia entre los servicios en la nube y las infraestructuras locales de escritorio virtual (VDIs) tradicionales es cómo controlan las responsabilidades de seguridad. Por ejemplo, en una VDI local tradicional, el cliente es responsable de todos los aspectos de seguridad. Sin embargo, en la mayoría de los servicios en la nube, estas responsabilidades se comparten entre el cliente y la empresa.

Cuando se usa Windows Virtual Desktop, es importante comprender que, aunque algunos componentes ya están protegidos en el entorno, deberá configurar otras áreas para adaptarse a las necesidades de seguridad de su organización.

Estos son los requisitos de seguridad que son su responsabilidad en la implementación de Windows Virtual Desktop:

| Requisito de seguridad | ¿Es responsabilidad del cliente? |
|---------------|:-------------------------:|
|Identidad|Sí|
|Dispositivos del usuario (móvil y PC)|Sí|
|Seguridad de la aplicación|Sí|
|SO del host de sesión|Sí|
|Configuración de la implementación|Sí|
|Controles de red|Sí|
|Plano de control de la virtualización|No|
|Hosts físicos|No|
|Red física|No|
|Centro de datos físico|No|

Microsoft se encarga de los requisitos de seguridad que no son responsabilidad del cliente.

## <a name="azure-security-best-practices"></a>Procedimientos recomendados de seguridad de Azure

Windows Virtual Desktop es un servicio de Azure. Para maximizar la protección de la implementación de Windows Virtual Desktop, también debe asegurarse de proteger la infraestructura de Azure circundante y el plano de administración. Para proteger la infraestructura, considere cómo encaja Windows Virtual Desktop en el ecosistema de Azure general. Para más información sobre el ecosistema de Azure, consulte [Patrones y procedimientos recomendados de seguridad en Azure](../security/fundamentals/best-practices-and-patterns.md).

En esta sección se describen los procedimientos recomendados para proteger el ecosistema de Azure.

### <a name="enable-azure-security-center"></a>Habilitación de Azure Security Center

Se recomienda habilitar Azure Security Center Estándar para las suscripciones, máquinas virtuales, almacenes de claves y cuentas de almacenamiento.

Puede hacer lo siguiente con Azure Security Center Estándar:

* Administrar vulnerabilidades.
* Evaluar el cumplimiento con marcos comunes, como PCI.
* Fortalecer la seguridad general del entorno.

Para más información, consulte [Incorporación de su suscripción a Azure al nivel Estándar de Security Center](../security-center/security-center-get-started.md).

### <a name="improve-your-secure-score"></a>Mejora de la puntuación de seguridad

La puntuación de seguridad proporciona recomendaciones y procedimientos recomendados para mejorar la seguridad general. Estas recomendaciones se clasifican por orden de prioridad para ayudarle a elegir las más importantes, mientras que las opciones de corrección rápida le ayudan a abordar las posibles vulnerabilidades rápidamente. Estas recomendaciones también se actualizan con el tiempo, lo que permite mantenerse al día con las mejores formas de mantener la seguridad del entorno. Para más información, consulte [Mejora de la puntuación de seguridad de Azure Security Center](../security-center/security-center-secure-score.md).

## <a name="windows-virtual-desktop-security-best-practices"></a>Procedimientos recomendados de seguridad para Windows Virtual Desktop

Windows Virtual Desktop tiene muchos controles de seguridad integrados. En esta sección, aprenderá sobre los controles de seguridad que puede usar para mantener protegidos a los usuarios y los datos.

### <a name="require-multi-factor-authentication"></a>Requerir autenticación multifactor

Al requerir la autenticación multifactor para todos los usuarios y administradores en Windows Virtual Desktop, mejora la seguridad de toda la implementación. Para más información, consulte [Habilitación de Azure Multi-Factor Authentication para Windows Virtual Desktop](set-up-mfa.md).

### <a name="enable-conditional-access"></a>Habilitación del acceso condicional

Al habilitar el [acceso condicional](../active-directory/conditional-access/best-practices.md), podrá administrar los riesgos antes de conceder acceso al entorno de Windows Virtual Desktop a los usuarios. A la hora de decidir a qué usuarios se concederá acceso, se recomienda considerar también quién es el usuario, cómo inicia sesión y qué dispositivo está usando.

### <a name="collect-audit-logs"></a>Recopilación de registros de auditoría

Al habilitar la recopilación de registros de auditoría, podrá ver la actividad de los usuario y administradores relacionada con Windows Virtual Desktop. Los siguientes son algunos ejemplos de registros de auditoría clave:

-   [Registro de actividad de Azure](../azure-monitor/platform/activity-log-collect.md)
-   [Registro de actividad de Azure Active Directory](../active-directory/reports-monitoring/concept-activity-logs-azure-monitor.md)
-   [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md)
-   [Hosts de sesión](../azure-monitor/platform/agent-windows.md)
-   [Registro de diagnóstico de Windows Virtual Desktop](../virtual-desktop/diagnostics-log-analytics.md)
-   [Registros de Key Vault](../key-vault/general/logging.md)

### <a name="use-remoteapps"></a>Uso de RemoteApps

Al elegir un modelo de implementación, puede proporcionar a los usuarios remotos acceso a escritorios virtuales completos o solo a aplicaciones determinadas. Las aplicaciones remotas, o RemoteApps, proporcionan una experiencia sin interrupciones a medida que el usuario trabaja con las aplicaciones en su escritorio virtual. RemoteApps reduce los riesgos, ya que solo permite al usuario trabajar con un subconjunto de la máquina remota que la aplicación expone.

### <a name="monitor-usage-with-azure-monitor"></a>Supervisión de uso con Azure Monitor

Supervise el uso y la disponibilidad del servicio de Windows Virtual Desktop con [Azure Monitor](https://azure.microsoft.com/services/monitor/). Considere la posibilidad de crear [alertas de estado del servicio](../service-health/alerts-activity-log-service-notifications.md) para que el servicio de Windows Virtual Desktop reciba notificaciones siempre que se produzca un evento que afecta al servicio.

## <a name="session-host-security-best-practices"></a>Procedimientos recomendados de seguridad del host de sesión

Los hosts de sesión son máquinas virtuales que se ejecutan dentro de una suscripción a Azure y una red virtual. La seguridad general de su implementación de Windows Virtual Desktop depende de los controles de seguridad que se aplican en los hosts de la sesión. En esta sección se describen los procedimientos recomendados para proteger los hosts de la sesión.

### <a name="enable-endpoint-protection"></a>Habilitación de Endpoint Protection

Para proteger la implementación frente a software malintencionado conocido, se recomienda habilitar Endpoint Protection en todos los hosts de la sesión. Puede usar el Antivirus de Windows Defender o un programa de terceros. Para obtener más información, consulte la [Guía de implementación para el Antivirus de Windows Defender en un entorno de VDI](/windows/security/threat-protection/windows-defender-antivirus/deployment-vdi-windows-defender-antivirus).

En el caso de las soluciones de perfil, como FSLogix u otras soluciones que montan archivos VHD, se recomienda excluir las extensiones de archivo VHD.

### <a name="install-an-endpoint-detection-and-response-product"></a>Instalación de un producto de detección y respuesta de puntos de conexión

Se recomienda instalar un producto de detección y respuesta de punto de conexión (EDR) para proporcionar funcionalidades avanzadas de detección y respuesta. En el caso de los sistemas operativos de servidor con [Azure Security Center](../security-center/security-center-services.md) habilitado, al instalar un producto de EDR se implementará ATP de Defender. En el caso de los sistemas operativos cliente, puede implementar [ATP de Defender](/windows/security/threat-protection/microsoft-defender-atp/onboarding) o un producto de terceros en esos puntos de conexión.

### <a name="enable-threat-and-vulnerability-management-assessments"></a>Habilitación de evaluaciones para administración de amenazas y vulnerabilidades

La identificación de las vulnerabilidades de software que existen en los sistemas operativos y aplicaciones es fundamental para garantizar la seguridad del entorno. Azure Security Center puede ayudarle a identificar los puntos problemáticos a través de las evaluaciones de vulnerabilidades para sistemas operativos de servidor. También puede usar ATP de Defender, que proporciona administración de amenazas y vulnerabilidades para los sistemas operativos de escritorio. También puede usar productos de terceros si así lo desea, aunque se recomienda usar Azure Security Center y ATP de Defender.

### <a name="patch-software-vulnerabilities-in-your-environment"></a>Aplicación de revisiones para las vulnerabilidades de software del entorno

Una vez identificada una vulnerabilidad, debe aplicar una revisión. Esto también se aplica a entornos virtuales, incluidos los sistemas operativos en ejecución, las aplicaciones que se hayan implementado dentro de ellos y las imágenes que se usan para crear nuevas máquinas. Siga las comunicaciones de notificación sobre revisiones del proveedor y aplique las revisiones de manera oportuna. Se recomienda aplicar revisiones a las imágenes base mensualmente para garantizar que las máquinas recién implementadas son lo más seguras posible.

### <a name="establish-maximum-inactive-time-and-disconnection-policies"></a>Establecimiento del tiempo máximo de inactividad y directivas de desconexión

Al cerrar la sesión de los usuarios cuando están inactivos, se conservan los recursos y se impide el acceso por parte de usuarios no autorizados. Se recomienda que los tiempos de espera mantengan el equilibrio entre la productividad del usuario y el uso de recursos. Para los usuarios que interactúan con aplicaciones sin estado, plantéese la posibilidad de directivas más agresivas que apaguen las máquinas y conserven los recursos. La desconexión de las aplicaciones de ejecución prolongada que continúan ejecutándose si un usuario está inactivo, como una simulación o representación de CAD, puede interrumpir el trabajo del usuario e incluso puede requerir que se reinicie el equipo.

### <a name="set-up-screen-locks-for-idle-sessions"></a>Configuración de bloqueos de pantalla para sesiones inactivas

Para evitar los accesos no deseados al sistema, configure Windows Virtual Desktop para que bloquee la pantalla de una máquina durante el tiempo de inactividad y solicite una autenticación para desbloquearla.

### <a name="establish-tiered-admin-access"></a>Establecimiento del acceso de administrador en capas

Se recomienda no conceder a los usuarios acceso de administrador para escritorios virtuales. Si necesita paquetes de software, se recomienda que estén disponibles a través de utilidades de administración de configuración, como Microsoft Endpoint Manager. En un entorno de varias sesiones, se recomienda impedir que los usuarios instalen software directamente.

### <a name="consider-which-users-should-access-which-resources"></a>Cuáles usuarios que deben tener acceso a qué recursos

Considere a los hosts de sesión como una extensión de la implementación de escritorio existente. Se recomienda controlar el acceso a los recursos de red del mismo modo que lo haría con otros equipos de escritorio del entorno, como la segmentación y filtrado de la red. De forma predeterminada, los hosts de sesión pueden conectarse a cualquier recurso de Internet. Hay varias maneras de limitar el tráfico, incluido el uso de Azure Firewall, dispositivos virtuales de red o servidores proxy. Si necesita limitar el tráfico, asegúrese de agregar las reglas adecuadas para que Windows Virtual Desktop pueda funcionar correctamente.

### <a name="manage-office-pro-plus-security"></a>Administración de la seguridad de Office Pro Plus

Además de proteger los hosts de sesión, es importante proteger también las aplicaciones que se ejecutan dentro de ellos. Office Pro Plus es una de las aplicaciones más comunes que se implementan en hosts de sesión. Para mejorar la seguridad de la implementación de Office, se recomienda usar el [Asesor de directivas de seguridad](/DeployOffice/overview-of-security-policy-advisor) para las aplicaciones de Microsoft 365 para empresas. Esta herramienta identifica las directivas que se pueden aplicar a la implementación para mayor seguridad. El Asesor de directivas de seguridad también recomienda directivas en función del impacto que tendrán en la seguridad y la productividad.

### <a name="other-security-tips-for-session-hosts"></a>Otras sugerencias de seguridad para hosts de sesión

Al restringir las funcionalidades del sistema operativo, puede mejorar la seguridad de los hosts de sesión. A continuación se incluyen algunas cosas que puede realizar:

- Controlar el redireccionamiento de dispositivos al redirigir las unidades, impresoras y dispositivos USB al dispositivo local de un usuario en una sesión de escritorio remoto. Se recomienda evaluar sus requisitos de seguridad y comprobar si estas características deben deshabilitarse o no.

- Restringir el acceso al Explorador de Windows al ocultar las asignaciones de unidades locales y remotas. De este modo, los usuarios no pueden detectar información no deseada acerca de la configuración del sistema y de los usuarios.

- Evitar el acceso directo de RDP a los hosts de sesión en su entorno. Si necesita acceso directo de RDP para administración o solución de problemas, habilite el acceso [Just-in-Time](../security-center/security-center-just-in-time.md) para limitar la posible superficie expuesta a ataques en un host de sesión.

- Conceder permisos limitados a los usuarios cuando accedan a sistemas de archivos locales y remotos. Puede restringir los permisos al garantizar que los sistemas de archivos locales y remotos usan listas de control de acceso con privilegios mínimos. De este modo, los usuarios solo pueden tener acceso a lo que necesitan y no pueden cambiar ni eliminar recursos críticos.

- Impedir que el software no deseado se ejecute en los hosts de sesión. Puede habilitar el Bloqueo de aplicación para mayor seguridad en los hosts de sesión. Así, se asegura de que solo las aplicaciones que habilite se pueden ejecutar en el host.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo habilitar la autenticación multifactor, consulte [Configuración de la autenticación multifactor](set-up-mfa.md).