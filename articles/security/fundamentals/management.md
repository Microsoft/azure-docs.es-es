---
title: Mejora de la seguridad de administración remota en Azure | Microsoft Docs
description: En este artículo se describen los pasos para mejorar la seguridad en la administración remota de entornos de Microsoft Azure, incluidos servicios en la nube, máquinas virtuales y aplicaciones personalizadas.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
editor: TomSh
ms.assetid: 2431feba-3364-4a63-8e66-858926061dd3
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/08/2020
ms.author: terrylan
ms.openlocfilehash: 73d82efed438d447c7af3bfc54d5c3fc22cdd819
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921935"
---
# <a name="security-management-in-azure"></a>Administración de la seguridad en Azure
Los suscriptores de Azure pueden administrar sus entornos de nube desde diversos dispositivos, incluidas estaciones de trabajo de administración, equipos de desarrollador e incluso dispositivos de usuario final con privilegios que tengan permisos específicos para la tarea. En algunos casos, las funciones administrativas se realizan mediante consolas web tales como [Azure Portal](https://azure.microsoft.com/features/azure-portal/). En otros casos, puede haber conexiones directas a Azure desde sistemas locales a través de redes privadas virtuales (VPN), Terminal Services, protocolos de aplicación de cliente o Azure Service Management API (SMAPI). Además, los puntos de conexión de cliente pueden estar unidos a un dominio o aislados y no administrados, como tabletas o smartphones.

Aunque las diversas funcionalidades de administración y acceso proporcionan un amplio conjunto de opciones, esta variabilidad puede suponer un riesgo importante para una implementación en la nube y dificultar la administración, el seguimiento y la auditoría de las acciones administrativas. Esta variabilidad también puede presentar amenazas para la seguridad debidas al acceso no regulado a los puntos de conexión de cliente que se usan para administrar los servicios en la nube. El uso de estaciones de trabajo personales o generales para desarrollar y administrar infraestructuras abre la puerta a amenazas impredecibles que se producen tanto en la exploración web (por ejemplo, ataques a los sitios web más utilizados) como en el correo electrónico (tales como ingeniería social y suplantación de identidad [phishing]).

![Diagrama que muestra las distintas formas en que una amenaza puede perpetrar ataques.](./media/management/typical-management-network-topology.png)

En este tipo de entorno, el riesgo de ataques es mayor porque es difícil construir las directivas y los mecanismos de seguridad para administrar correctamente el acceso a las interfaces de Azure (como SMAPI) desde puntos de conexión muy diversos.

### <a name="remote-management-threats"></a>Amenazas de la administración remota
Para obtener acceso con privilegios, los atacantes intentan comprometer las credenciales de cuenta (por ejemplo, forzando la contraseña, mediante suplantación de identidad [phishing] o recopilando credenciales), o intentan engañar a los usuarios para que ejecuten código perjudicial (por ejemplo, desde sitios web malintencionados con descargas ocultas o desde datos adjuntos de correo electrónico perjudiciales). En un entorno de nube administrado de forma remota, las infracciones de cuenta permiten el acceso desde cualquier lugar y en cualquier momento, lo que provoca un riesgo mayor.

Incluso con controles estrictos en las cuentas de los administradores principales, las cuentas de usuario de niveles inferiores se pueden usar para vulnerar los puntos débiles de la estrategia de seguridad de cada uno. La falta de un aprendizaje de seguridad adecuado también puede provocar infracciones debidas a la revelación o la exposición accidental de información de la cuenta.

Cuando una estación de trabajo de usuario se usa también para realizar tareas administrativas, puede presentar muchos puntos de riesgo distintos, por ejemplo, al explorar la Web, al utilizar herramientas de código abierto y de terceros o al abrir un documento perjudicial que contiene un troyano.

En general, la mayoría de los ataques dirigidos que provocan infracciones de datos pueden rastrearse hasta vulnerabilidades del navegador, complementos (por ejemplo, Flash, PDF, Java) y suplantación de identidad (correo electrónico) en equipos de escritorio. Estos equipos pueden tener permisos de nivel administrativo o de nivel de servicio para tener acceso a los servidores activos o los dispositivos de red para operaciones cuando se usan para el desarrollo o la administración de otros activos.

### <a name="operational-security-fundamentals"></a>Fundamentos de la seguridad operativa
Para que la administración y las operaciones sean más seguras, puede minimizar la superficie de ataque de un cliente reduciendo el número de posibles puntos de entrada. Esto puede hacerse a través de los principios de seguridad: "segregación de controles" y "segregación de entornos".

Aislamiento de funciones confidenciales entre sí para reducir la probabilidad de que un error en un nivel dé lugar a una infracción de seguridad en otro. Ejemplos:

* Las tareas administrativas no deben combinarse con las actividades que pueden suponer un riesgo (por ejemplo, un malware en el correo electrónico de un administrador que luego infecte un servidor de la infraestructura).
* Una estación de trabajo que se usa para operaciones de alta confidencialidad no debería estar en el mismo sistema que se usa para acciones de alto riesgo, como explorar Internet.

Reducción de la superficie de ataque del sistema mediante la eliminación de software innecesario. Ejemplo:

* Por ejemplo, una estación de trabajo administrativa, de soporte técnico o de desarrollo estándar no debería requerir la instalación de un cliente de correo electrónico o de otras aplicaciones de productividad si la finalidad principal del dispositivo es administrar servicios en la nube.

Sistemas cliente que tienen acceso de administrador a los componentes de la infraestructura deben estar sujetos a la directiva más estricta posible para reducir los riesgos de seguridad. Ejemplos:

* Entre las directivas de seguridad pueden incluirse la configuración de directiva de grupo que deniega el acceso a Internet abierto desde el dispositivo y el uso de una configuración restrictiva de firewall.
* Uso de redes VPN con protocolo de seguridad de Internet (IPsec) si se necesita acceso directo.
* Configuración de dominios de Active Directory de desarrollo y administración independientes.
* Aislamiento y filtrado del tráfico de red de las estaciones de trabajo de administración.
* Uso de software antimalware.
* Implementación de la autenticación multifactor para reducir el riesgo de credenciales robadas.

La consolidación de los recursos de acceso y la eliminación de puntos de conexión no administrados también simplifica las tareas de administración.

### <a name="providing-security-for-azure-remote-management"></a>Seguridad para la administración remota de Azure
Azure proporciona mecanismos de seguridad para ayudar a los administradores que administran servicios en la nube y máquinas virtuales de Azure. Estos mecanismos incluyen:

* Autenticación y [control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md).
* Supervisión, registro y auditoría.
* Certificados y comunicación cifrada.
* Un portal de administración web.
* Filtrado de paquetes de red.

Mediante la configuración de la seguridad del lado cliente y de la implementación de una puerta de enlace de administración en el centro de datos, es posible restringir y supervisar el acceso del administrador a datos y aplicaciones en la nube.

> [!NOTE]
> Algunas de las recomendaciones de este artículo pueden provocar un aumento del uso de datos, de la red o de los recursos de proceso, lo que podría incrementar los costos de las licencias o las suscripciones.
>
>

## <a name="hardened-workstation-for-management"></a>Estación de trabajo protegida para administración
El objetivo de proteger una estación de trabajo es eliminar todas las funciones excepto las más necesarias para que funcione, reduciendo así al mínimo la superficie de ataque. Para proteger el sistema se minimiza también el número de servicios y aplicaciones instalados, se limita la ejecución de aplicaciones, se restringe el acceso de red a lo estrictamente necesario y el sistema se mantiene siempre actualizado. Además, el uso de estaciones de trabajo protegidas para la administración aísla las herramientas y las actividades administrativas de las demás tareas del usuario final.

Dentro de un entorno empresarial local, puede limitar la superficie de ataque de la infraestructura física mediante redes de administración dedicadas, salas de servidores con acceso por tarjeta y estaciones de trabajo que se ejecutan en las áreas protegidas de la red. En un modelo de TI de nube o híbrido, ser diligente con los servicios de administración segura puede resultar más complejo debido a la falta de acceso físico a los recursos de TI. La implementación de soluciones de protección requiere una cuidadosa configuración del software, procesos centrados en la seguridad y directivas completas.

Usar una huella digital de software minimizada con privilegios mínimos en una estación de trabajo bloqueada para la administración en la nube, así como para el desarrollo de aplicaciones, normaliza los entornos de desarrollo y administración remota y reduce el riesgo de incidentes de seguridad. La configuración de una estación de trabajo protegida puede ayudar a cerrar muchas de las vías vulnerables normalmente usadas por el malware, lo que ayuda a evitar riesgos en las cuentas que se usan para administrar recursos de nube críticos. En concreto, puede usar [Windows AppLocker](https://technet.microsoft.com/library/dd759117.aspx) y la tecnología Hyper-V para controlar y aislar el comportamiento del sistema de cliente y mitigar las amenazas, incluido el correo electrónico o la exploración de Internet.

En una estación de trabajo protegida, el administrador ejecuta una cuenta de usuario estándar (que impide la ejecución en el nivel administrativo) y las aplicaciones asociadas se controlan mediante una lista de permitidos. Los elementos básicos de una estación de trabajo protegida son los siguientes:

* Análisis y revisión activos. Implementar software antimalware, realizar un análisis regular de vulnerabilidades y actualizar todas las estaciones de trabajo con la actualización de seguridad más reciente de manera oportuna.
* Funcionalidad limitada Desinstale todas las aplicaciones que no sean necesarias y deshabilite los servicios innecesarios (inicio).
* Protección de la red. Use reglas de Firewall de Windows para permitir solo direcciones IP, puertos y direcciones URL válidos relacionados con la administración de Azure. Asegúrese de bloquear también las conexiones remotas entrantes a la estación de trabajo.
* Restricción de la ejecución. Permita solo el conjunto de archivos ejecutables predefinidos que se necesitan para realizar la administración (lo que se conoce como "denegación predeterminada"). De forma predeterminada, se deberá denegar el permiso a los usuarios para ejecutar cualquier programa a menos que esté definido explícitamente en la lista de permitidos.
* Privilegios mínimos. Los usuarios de las estaciones de trabajo de administración no deben tener privilegios administrativos en el propio equipo local. De este modo, no pueden cambiar la configuración del sistema ni los archivos del sistema, ya sea intencionadamente o por accidente.

Para aplicar todo esto, puede usar [objetos de directiva de grupo](../../active-directory-domain-services/manage-group-policy.md) (GPO) en Active Directory Domain Services (AD DS) y aplicarlos mediante su dominio de administración (local) a todas las cuentas de administración.

### <a name="managing-services-applications-and-data"></a>Administración de servicios, aplicaciones y datos
La configuración de servicios en la nube de Azure se realiza mediante el Portal de Azure o SMAPI, mediante la interfaz de línea de comandos de Windows PowerShell o mediante una aplicación personalizada que aprovecha estas interfaces de RESTful. Los servicios que usan estos mecanismos son Azure Active Directory (Azure AD), Azure Storage, Azure Websites y Azure Virtual Network, entre otros.

Máquina virtual: las aplicaciones implementadas proporcionan sus propias herramientas e interfaces de cliente según sea necesario, como Microsoft Management Console (MMC), una consola de administración empresarial (Microsoft System Center o Windows Intune) u otra aplicación de administración (Microsoft SQL Server Management Studio, por ejemplo). Estas herramientas suelen residir en una red de cliente o en un entorno empresarial. Pueden depender de protocolos de red específicos, como el Protocolo de escritorio remoto (RDP), que requiere conexiones directas con estado. Algunos pueden tener interfaces habilitadas para web que no se deben publicar ni ser accesibles a través de Internet.

Puede restringir el acceso a la administración de los servicios de infraestructura y plataforma en Azure mediante la [autenticación multifactor](/azure/active-directory/authentication/multi-factor-authentication), los [certificados de administración X.509](https://blogs.msdn.microsoft.com/azuresecurity/2015/07/13/certificate-management-in-azure-dos-and-donts/) y las reglas de firewall. El Portal de Azure y SMAPI requieren Seguridad de capa de transporte (TLS). Sin embargo, los servicios y las aplicaciones que implemente en Azure requieren que tome medidas de protección adecuadas en función de la aplicación. Estos mecanismos se suelen habilitar más fácilmente mediante una configuración de estación de trabajo protegida estándar.

### <a name="management-gateway"></a>Puerta de enlace de administración
Para centralizar todo el acceso administrativo y simplificar la supervisión y el registro, puede implementar un servidor de [Puerta de enlace de Escritorio remoto](https://technet.microsoft.com/library/dd560672) dedicado en su red local, conectado a su entorno de Azure.

Puerta de enlace de Escritorio remoto es un servicio de proxy RDP basado en directivas que aplica requisitos de seguridad. La implementación de Puerta de enlace de Escritorio remoto junto con Windows Server Network Access Protection (NAP) ayuda a garantizar que solo podrán conectarse los clientes que cumplen los criterios de mantenimiento de seguridad específicos establecidos por Active Directory Domain Services (AD DS) y Objetos de directiva de grupo (GPO). Además:

* Aprovisione un [certificado de administración de Azure](https://msdn.microsoft.com/library/azure/gg551722.aspx) en Puerta de enlace de Escritorio remoto para que sea el único host con permiso para acceder a Azure Portal.
* Una Puerta de enlace de Escritorio remoto al mismo [dominio de administración](https://technet.microsoft.com/library/bb727085.aspx) que las estaciones de trabajo de administrador. Esto es necesario cuando se usa una VPN IPsec de sitio a sitio o ExpressRoute dentro de un dominio que tenga una confianza unidireccional con Azure AD, o si federa las credenciales entre su instancia de AD DS local y Azure AD.
* Configure una [directiva de autorización de conexión de cliente](https://technet.microsoft.com/library/cc753324.aspx) para que Puerta de enlace de Escritorio remoto pueda comprobar si el nombre de la máquina cliente es válido (unido a un dominio) y tiene permiso para acceder a Azure Portal.
* Use IPsec para que la [VPN de Azure](https://azure.microsoft.com/documentation/services/vpn-gateway/) pueda proteger aún más el tráfico de administración contra la interceptación y el robo de tokens, o podría usar un vínculo a Internet aislado mediante [Azure ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/).
* Habilite la autenticación multifactor (mediante [Azure Multi-Factor Authentication](/azure/active-directory/authentication/multi-factor-authentication)) o la autenticación de tarjeta inteligente para los administradores que inician sesión mediante Puerta de enlace de Escritorio remoto.
* Configure las [restricciones de direcciones IP](https://azure.microsoft.com/blog/2013/08/27/confirming-dynamic-ip-address-restrictions-in-windows-azure-web-sites/) de origen o los [grupos de seguridad de red](/azure/virtual-network/security-overview) en Azure para minimizar el número de puntos de conexión de administración permitidos.

## <a name="security-guidelines"></a>Directrices de seguridad
En general, para ayudar a proteger las estaciones de trabajo de administrador para su uso con la nube se usan procedimientos similares a los usados para cualquier estación de trabajo local; por ejemplo, compilación minimizada y permisos restrictivos. Algunos aspectos únicos de la administración en la nube son más parecidos a la administración remota o empresarial fuera de banda. Estos incluyen el uso y la auditoría de credenciales, el acceso remoto con seguridad mejorada y la detección de amenazas y respuesta a las mismas.

### <a name="authentication"></a>Authentication
Puede usar las restricciones de inicio de sesión de Azure para impedir que direcciones IP de origen tengan acceso a herramientas administrativas y solicitudes de acceso de auditoría. Para ayudar a Azure a identificar los clientes de administración (estaciones de trabajo o aplicaciones), puede configurar SMAPI (mediante herramientas desarrolladas por el cliente tales como cmdlets de Windows PowerShell) y Azure Portal para que requieran la instalación de certificados de administración del cliente, además de los certificados SSL/TLS. Se recomienda también que el acceso de administrador requiera autenticación multifactor.

Algunas aplicaciones o servicios que se implementan en Azure pueden tener sus propios mecanismos de autenticación para el acceso de usuario final y de administrador, mientras que otras aprovechan toda la funcionalidad de Azure AD. Dependiendo de si desea federar credenciales mediante Servicios de federación de Active Directory (AD FS), usar la sincronización de directorios o mantener las cuentas de usuario únicamente en la nube, el uso de [Microsoft Identity Manager](https://technet.microsoft.com/library/mt218776.aspx) (parte de Azure AD Premium) lo ayuda a administrar los ciclos de vida de las identidades entre los recursos.

### <a name="connectivity"></a>Conectividad
Existen varios mecanismos para ayudar a proteger las conexiones de cliente de las redes virtuales de Azure. Dos de estos mecanismos, [VPN de sitio a sitio](https://channel9.msdn.com/series/Azure-Site-to-Site-VPN) (S2S) y [VPN de punto a sitio](/azure/vpn-gateway/vpn-gateway-point-to-site-create) (P2S), permiten usar el estándar del sector IPsec (S2S) o el [Protocolo de túnel de sockets seguros](https://technet.microsoft.com/magazine/2007.06.cableguy.aspx) (SSTP) (P2S) para el cifrado y la tunelización. Cuando Azure se conecta a la administración de servicios de Azure accesibles desde Internet, como Azure Portal, requiere el protocolo HTTPS.

Una estación de trabajo protegida independiente que no se conecta a Azure mediante Puerta de enlace de Escritorio remoto debe usar VPN de punto a sitio basada en SSTP para crear la conexión inicial a Azure Virtual Network y, después, establecer la conexión RDP a las máquinas virtuales individuales desde el túnel VPN.

### <a name="management-auditing-vs-policy-enforcement"></a>Auditoría de administración y aplicación de directivas
Normalmente, hay dos enfoques para proteger los procesos de administración: auditoría y aplicación de directivas. Ambos proporcionarán controles completos, pero que pueden no ser posibles en todas las situaciones. Además, cada método tiene distintos niveles de riesgo, costo y esfuerzo asociados con la administración de la seguridad, especialmente en relación con el nivel de confianza en las personas y las arquitecturas del sistema.

La supervisión, el registro y la auditoría proporcionan una base para el seguimiento y la descripción de las actividades administrativas, pero puede que no siempre sea factible auditar todas las acciones con detalle debido a la cantidad de datos generados. Sin embargo, auditar la efectividad de las directivas de administración es un procedimiento recomendado.

La aplicación de directivas que incluyen controles estrictos de acceso coloca mecanismos de programación que rigen la acciones del administrador, y ayuda a garantizar que se están usando todas las medidas de protección posibles. El registro proporciona una prueba de cumplimiento, además de un registro de quién hizo qué, desde dónde y cuándo. El registro también permite auditar y verificar la información acerca de cómo los administradores usan las directivas, y proporciona pruebas de las actividades.

## <a name="client-configuration"></a>Configuración de cliente
Se recomiendan tres configuraciones principales para una estación de trabajo protegida. Los principales puntos de diferencia entre ellas son el costo, la facilidad de uso y la accesibilidad, pero el perfil de seguridad es similar en todas las opciones. La tabla siguiente proporciona un breve análisis de las ventajas y los riesgos de cada una. (Observe que "equipos corporativos" hace referencia a una configuración de PC de escritorio estándar que se puede implementar para todos los usuarios del dominio, independientemente de sus roles).

| Configuración | Ventajas | Desventajas |
| --- | --- | --- |
| Estación de trabajo protegida independiente |Estación de trabajo estrechamente controlada |mayor costo para escritorios dedicados |
| - | Menor riesgo de vulnerabilidades de la aplicación |Mayor esfuerzo de administración |
| - | Clara separación de las tareas | - |
| Equipo corporativo como máquina virtual |Menores costos de hardware | - |
| - | Separación de roles y aplicaciones | - |

Es importante que la estación de trabajo protegida sea el host y no el invitado, y que no haya nada entre el sistema operativo host y el hardware. Para seguir el "principio del origen limpio" (también conocido como "origen seguro"), el host debe estar lo más protegido posible. De lo contrario, la estación de trabajo protegida (invitado) puede ser objeto de ataques en el sistema en el que se hospeda.

Puede separar aún más las funciones administrativas mediante imágenes de sistema dedicadas para cada estación de trabajo protegida, que solo tengan las herramientas y los permisos necesarios para administrar determinadas aplicaciones de Azure y en la nube, con GPO de AD DS locales específicos para las tareas en la nube.

Para los entornos de TI que carecen de infraestructura local (por ejemplo, sin acceso a una instancia de AD DS local para los GPO porque todos los servidores se encuentran en la nube), un servicio como [Microsoft Intune](https://technet.microsoft.com/library/jj676587.aspx) puede simplificar la implementación y el mantenimiento de configuraciones de estación de trabajo.

### <a name="stand-alone-hardened-workstation-for-management"></a>Estación de trabajo protegida independiente para administración
Con una estación de trabajo protegida independiente, los administradores tienen un equipo o un portátil que usan para las tareas administrativas, y otro equipo o portátil diferente para las tareas no administrativas. Una estación de trabajo dedicada para administrar los servicios de Azure no necesita tener otras aplicaciones instaladas. Además, el uso de estaciones de trabajo que admiten un [Módulo de plataforma segura](https://technet.microsoft.com/library/cc766159) (TPM) o una tecnología de criptografía de nivel de hardware similar ayuda a autenticar dispositivos e impedir ciertos ataques. TPM también admite la protección del volumen completo de la unidad del sistema mediante [Cifrado de unidad BitLocker](https://technet.microsoft.com/library/cc732774.aspx).

En el escenario de la estación de trabajo protegida independiente (se muestra a continuación), la instancia local de Firewall de Windows (o un firewall de cliente no sea de Microsoft) está configurada para bloquear las conexiones entrantes, como RDP. El administrador puede iniciar sesión en la estación de trabajo protegida e iniciar una sesión de RDP que se conecta a Azure después de establecer una conexión de VPN con Azure Virtual Network, pero no puede iniciar sesión en un equipo corporativo ni usar RDP para conectarse a la propia estación de trabajo protegida.

![Diagrama que muestra el escenario de una estación de trabajo protegida independiente.](./media/management/stand-alone-hardened-workstation-topology.png)

### <a name="corporate-pc-as-virtual-machine"></a>Equipo corporativo como máquina virtual
En aquellos casos en los que una estación de trabajo independiente protegida sea muy cara o no resulte conveniente, la estación de trabajo protegida puede hospedar una máquina virtual para llevar a cabo tareas no administrativas.

![Diagrama que muestra la estación de trabajo protegida que hospeda una máquina virtual para realizar tareas no administrativas.](./media/management/hardened-workstation-enabled-with-hyper-v.png)

Para evitar varios riesgos de seguridad que pueden surgir del uso de una estación de trabajo para la administración de sistemas y otras tareas del trabajo diario, puede implementar una máquina virtual de Windows Hyper-V en la estación de trabajo protegido. Esta máquina virtual puede usarse como equipo corporativo. El entorno del equipo corporativo puede permanecer aislado del host, lo que reduce la superficie de ataque y evita que las actividades diarias del usuario (por ejemplo, el correo electrónico) coexistan con tareas administrativas importantes.

La máquina virtual del equipo corporativo se ejecuta en un espacio protegido y proporciona aplicaciones de usuario. El host sigue siendo un "origen limpio" y aplica directivas de red estrictas en el sistema operativo de raíz (por ejemplo, bloqueo del acceso RDP desde la máquina virtual).

## <a name="best-practices"></a>Procedimientos recomendados
Tenga en cuenta las siguientes directrices adicionales cuando administre aplicaciones y datos en Azure.

### <a name="dos-and-donts"></a>Consejos
No dé por sentado que si una estación de trabajo está bloqueada ya no es necesario cumplir otros requisitos de seguridad comunes. El riesgo potencial es mayor debido a los niveles de acceso con privilegios elevados que normalmente poseen las cuentas de administrador. En la tabla siguiente se muestran ejemplos de riesgos y los procedimientos de seguridad alternativos.

| Lo que debe evitar: | Lo que es necesario hacer: |
| --- | --- |
| No envíe por correo electrónico credenciales de acceso de administrador ni otros secretos (por ejemplo, certificados TLS/SSL o de administración) |Mantenga la confidencialidad; para ello, entregue los nombres y las contraseñas de las cuentas de palabra (pero no los almacene en buzones de voz), realice una instalación remota de los certificados de cliente y servidor (a través de una sesión cifrada), descárguelos desde un recurso compartido de red protegido o distribúyalos manualmente mediante soportes físicos extraíbles. |
| - | Administre activamente los ciclos de vida de sus certificados de administración. |
| No almacene contraseñas de cuentas sin cifrar o sin hash en almacenamiento de aplicaciones (por ejemplo, en hojas de cálculo, recursos compartidos de archivos o sitios de SharePoint). |Establezca principios de administración de seguridad y directivas de protección del sistema, y aplíquelos a su entorno de desarrollo. |
| - | Use reglas de asignación de certificados de [Enhanced Mitigation Experience Toolkit 5.5](https://technet.microsoft.com/security/jj653751) para garantizar el acceso correcto a los sitios SSL/TLS de Azure. |
| No comparta cuentas ni contraseñas entre los administradores o reutilice contraseñas en diferentes cuentas de usuario o servicios, especialmente las de medios sociales o de otras actividades no administrativas. |Cree una cuenta de Microsoft dedicada para administrar su suscripción de Azure, una cuenta que no se use para el correo electrónico personal. |
| No envíe archivos de configuración por correo electrónico. |Los perfiles y los archivos de configuración deben instalarse desde un origen de confianza (por ejemplo, una unidad flash USB cifrada), no desde un mecanismo que pueda verse comprometido fácilmente, como el correo electrónico. |
| No use contraseñas de inicio de sesión simples o débiles. |Aplique directivas de contraseña segura, ciclos de expiración (cambiar en el primer uso), tiempos de espera de la consola y bloqueos de cuentas automáticos. Use un sistema de administración de contraseñas de cliente con autenticación multifactor para el acceso al almacén de contraseñas. |
| No exponga los puertos de administración a Internet. |Bloquee los puertos y las direcciones IP de Azure para restringir el acceso de administración. Para más información, consulte las notas del producto de [Azure Network Security](https://download.microsoft.com/download/4/3/9/43902EC9-410E-4875-8800-0788BE146A3D/Windows%20Azure%20Network%20Security%20Whitepaper%20-%20FINAL.docx) (Seguridad de Red de Azure). |
| - | Use NAP, VPN y firewalls para todas las conexiones de administración. |

## <a name="azure-operations"></a>Operaciones de Azure
Dentro de las operaciones que Microsoft realiza con Azure, los ingenieros de operaciones y el personal de soporte técnico que tienen acceso a los sistemas de producción de Azure usan [equipos de estación de trabajo protegidos con máquinas virtuales](#stand-alone-hardened-workstation-for-management) aprovisionadas en ellos para el acceso a las aplicaciones y a la red corporativa interna (como correo electrónico, intranet, etc.). Todos los equipos de estación de trabajo de administración tienen TPM, la unidad de arranque del host está cifrada con BitLocker y están unidas a una unidad organizativa (OU) especial en el dominio corporativo principal de Microsoft.

La protección del sistema se aplica mediante la directiva de grupo, con actualizaciones de software centralizadas. Para auditoría y análisis, los registros de eventos (por ejemplo, seguridad y AppLocker) se recopilan desde estaciones de trabajo de administración y se guardan en una ubicación central.

Además, en la red de Microsoft se usan cuadros de salto dedicados que requieren autenticación en dos fases para conectarse a la red de producción de Azure.

## <a name="azure-security-checklist"></a>Lista de comprobación de seguridad de Azure
Reducir el número de tareas que los administradores pueden realizar en una estación de trabajo protegida ayuda a minimizar la superficie de ataque en su entorno de administración y desarrollo. Use las siguientes tecnologías para ayudar a asegurar su estación de trabajo protegida:

* Protección de Internet Explorer. El explorador Internet Explorer (o, en este caso, cualquier explorador web) es un punto de entrada clave para el código perjudicial debido a sus amplias interacciones con servidores externos. Revise las directivas de cliente y aplique la ejecución en modo protegido, la deshabilitación de complementos, la deshabilitación de descargas de archivos y el uso del filtrado [Microsoft SmartScreen](https://technet.microsoft.com/library/jj618329.aspx). Asegúrese de que se muestran las advertencias de seguridad. Aproveche las ventajas de las zonas de Internet y crear una lista de sitios de confianza para los que ha configurado una protección razonable. Bloquee todos los demás sitios y el código en el explorador, como ActiveX y Java.
* Usuario estándar Trabajar como usuario estándar ofrece una serie de ventajas, la principal es que dificulta el robo de las credenciales de administrador mediante software malintencionado. Además, una cuenta de usuario estándar no tiene privilegios elevados en el sistema operativo raíz, y muchas opciones de configuración y API están bloqueadas de forma predeterminada.
* AppLocker. Puede usar [AppLocker](https://technet.microsoft.com/library/ee619725.aspx) para restringir los programas y los scripts que los usuarios pueden ejecutar. Puede ejecutar AppLocker en modo auditoría o cumplimiento. De forma predeterminada, AppLocker tiene una regla de permiso que permite a los usuarios que tengan un token de administrador ejecutar todo el código en el cliente. Esta regla existe para impedir que los administradores se bloqueen a sí mismos y solo se aplica a los tokens con privilegios elevados. Consulte también el tema sobre la integridad del código, que forma parte de la [seguridad fundamental](https://technet.microsoft.com/library/dd348705.aspx) de Windows Server.
* Firma del código. La firma del código de todas las herramientas y scripts que usan los administradores es un mecanismo fácil de administrar para la implementación de directivas de bloqueo de aplicaciones. Los valores hash no se escalan con cambios rápidos al código y las rutas de acceso de archivo no proporcionan un alto nivel de seguridad. Debe combinar reglas de AppLocker con una [directiva de ejecución](https://technet.microsoft.com/library/ee176961.aspx) de PowerShell que solo permita la [ejecución](https://technet.microsoft.com/library/hh849812.aspx) de código y scripts específicos firmados.
* Directiva de grupo. Cree una directiva administrativa global que se aplique a cualquier estación de trabajo del dominio que se use para administración (y bloquee el acceso de todas los demás), así como a las cuentas de usuario autenticadas en esas estaciones de trabajo.
* Aprovisionamiento con seguridad mejorada. Proteja la imagen de su estación de trabajo protegida de referencia contra la manipulación. Use medidas de seguridad tales como cifrado y aislamiento para almacenar imágenes, máquinas virtuales y scripts, y restrinja el acceso (quizás con un proceso de inserción y extracción del repositorio que se pueda auditar).
* Aplicación de revisiones. Mantenga una compilación coherente (o use imágenes diferentes para desarrollo, operaciones y otras tareas administrativas), examine de forma rutinaria si hay cambios y malware, mantenga la compilación al día y active los equipos solo cuando se necesiten.
* Cifrado. Asegúrese de que las estaciones de trabajo de administración tengan TPM para habilitar [Sistema de cifrado de archivos](https://technet.microsoft.com/library/cc700811.aspx) (EFS) y BitLocker de forma más segura.
* Gobernanza. Use GPO de AD DS para controlar todas las interfaces de Windows de los administradores, como el uso compartido de archivos. Incluya las estaciones de trabajo de administración de procesos de auditoría, supervisión y registro. Realice un seguimiento del acceso y uso de los administradores y programadores.

## <a name="summary"></a>Resumen
Usar una configuración de estación de trabajo protegida para administrar los servicios en la nube de Azure, las máquinas virtuales y las aplicaciones puede ayudarle a evitar muchos riesgos y amenazas que pueden derivar de la administración remota de la infraestructura de TI esencial. Tanto Azure como Windows proporcionan mecanismos que puede emplear para ayudar a proteger y controlar el comportamiento de las comunicaciones, la autenticación y los clientes.

## <a name="next-steps"></a>Pasos siguientes
Los siguientes recursos proporcionan más información general acerca de los servicios de Azure y otros servicios de Microsoft relacionados, así como elementos específicos a los que se hace referencia en este documento:

* [Securing Privileged Access](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/securing-privileged-access) (Protección del acceso con privilegios): consulte la información técnica para diseñar y crear una estación de trabajo administrativa segura para la administración de Azure.
* [Centro de confianza de Microsoft](https://microsoft.com/en-us/trustcenter/cloudservices/azure): conozca las funcionalidades de la plataforma Azure que protegen el tejido de Azure y las cargas de trabajo que se ejecutan en Azure.
* [Microsoft Security Response Center](https://www.microsoft.com/msrc): aquí podrá notificar vulnerabilidades de seguridad de Microsoft, incluidos problemas con Azure, o también mediante correo electrónico a [secure@microsoft.com](mailto:secure@microsoft.com).
