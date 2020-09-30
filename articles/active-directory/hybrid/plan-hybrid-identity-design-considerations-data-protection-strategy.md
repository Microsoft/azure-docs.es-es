---
title: 'Diseño de identidades híbridas: estrategia de protección de datos en Azure | Microsoft Docs'
description: Puede definir la estrategia de protección de datos para que su solución de identidad híbrida cumpla los requisitos empresariales que se indiquen.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: e76fd1f4-340a-492a-84d9-e05f3b7cc396
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: a01f5a87092f0111a5dbf9d180e785a47bd70a0b
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279064"
---
# <a name="define-data-protection-strategy-for-your-hybrid-identity-solution"></a>Definición de una estrategia de protección de datos para soluciones de identidad híbrida
En esta tarea, se definirá la estrategia de protección de datos para que una solución de identidad híbrida cumpla los requisitos empresariales que se definen en:

* [Determinación de los requisitos de protección de datos](plan-hybrid-identity-design-considerations-dataprotection-requirements.md)
* [Determinación de los requisitos de administración de contenido](plan-hybrid-identity-design-considerations-contentmgt-requirements.md)
* [Determinación de los requisitos de control de acceso](plan-hybrid-identity-design-considerations-accesscontrol-requirements.md)
* [Determinación de los requisitos de respuesta a incidentes](plan-hybrid-identity-design-considerations-incident-response-requirements.md)

## <a name="define-data-protection-options"></a>Definición de las opciones de protección de datos
Tal como se explicó en [Determinación de los requisitos de sincronización de directorios](plan-hybrid-identity-design-considerations-directory-sync-requirements.md), Microsoft Azure AD puede sincronizarse con los servicios de Active Directory Domain Services (AD DS) que estén ubicados localmente. Esta integración permite a las organizaciones usar Azure AD para verificar las credenciales de los usuarios cuando intentan obtener acceso a los recursos corporativos. Esto se puede hacer en dos escenarios: datos en reposo locales y en la nube. El acceso a los datos de Azure AD requiere autenticación del usuario mediante un servicio de token de seguridad (STS).

Una vez realizada la autenticación, el nombre principal de usuario (UPN) se lee desde el token de autenticación. A continuación, el sistema de autorización determina la partición replicada y el contenedor correspondiente al dominio del usuario. El sistema de autorización usa entonces la información sobre la existencia, el estado habilitado y el rol del usuario para determinar si el usuario de esta sesión puede acceder al inquilino de destino. Determinadas acciones autorizadas (específicamente, la creación de un usuario o el restablecimiento de contraseña) crean una pista de auditoría que un administrador de inquilinos puede usar para administrar los esfuerzos de cumplimiento o las investigaciones.

Es posible que mover datos desde un centro de datos local a Azure Storage a través de una conexión a Internet no siempre sea factibles debido al volumen de datos, a la disponibilidad del ancho de banda o a otras consideraciones. El [servicio Azure Storage Import/Export](../../storage/common/storage-import-export-service.md) ofrece una opción basada en hardware para colocar o recuperar grandes volúmenes de datos en el almacenamiento de blobs. Le permite enviar unidades de disco duro [cifradas mediante BitLocker](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn306081(v=ws.11)#BKMK_BL2012R2) directamente a un centro de datos de Azure donde operadores en la nube cargarán el contenido en su cuenta de almacenamiento o descargarán los datos de Azure a las unidades de disco que posea. En este proceso solo se aceptan discos cifrados (para lo que se usa una clave BitLocker que genera el propio servicio durante la configuración del trabajo). La clave de BitLocker se proporciona a Azure por separado, con lo que se proporciona el uso compartido de claves fuera de banda.

Dado que los datos en tránsito pueden tener lugar en distintos escenarios, también es relevante saber que Microsoft Azure usa [redes virtuales](https://azure.microsoft.com/documentation/services/virtual-network/) para aislar el tráfico de los inquilinos entre ellos, para lo que emplea medidas como firewalls de nivel de invitado y host, filtrado de paquetes IP, bloqueo de puertos y puntos de conexión HTTPS. Sin embargo, también se cifran la mayoría de las comunicaciones internas de Azure, incluida las de infraestructura con infraestructura e infraestructura con cliente (local). Otro escenario importante es las comunicaciones dentro de los centros de datos de Azure; Microsoft administra redes para asegurarse de que ninguna máquina virtual puede suplantar o escuchar en la dirección IP de otra. TLS/SSL se usa al acceder a Azure Storage o a bases de datos SQL, o bien al conectarse a Cloud Services. En ese caso, el administrador de clientes es el responsable de obtener un certificado TLS/SSL e implementarlo en la infraestructura de sus inquilinos. El movimiento del tráfico de datos entre máquinas virtuales en la misma implementación o entre inquilinos en una sola implementación mediante Microsoft Azure Virtual Network se puede proteger mediante protocolos de comunicación cifrada como HTTPS o SSL/TLS, entre otros.

En función de cómo responda a las preguntas del apartado [Determinar los requisitos de protección de datos](plan-hybrid-identity-design-considerations-dataprotection-requirements.md), puede indicar cómo desea proteger los datos y de qué forma quiere que la solución de identidad híbrida le ayude a realizar dicha tarea. En la siguiente tabla se muestran las opciones compatibles con Azure que están disponibles para cada escenario de protección de datos.

| Opciones de protección de datos | En reposo en la nube | En reposo localmente | En tránsito |
| --- | --- | --- | --- |
| Cifrado BitLocker de unidades |X |X | |
| SQL Server para cifrar bases de datos |X |X | |
| Cifrado de VM a VM | | |X |
| SSL/TLS | | |X |
| VPN | | |X |

> [!NOTE]
> Para más información acerca de las certificaciones con las que es compatible cada servicio de Azure, consulte [Cumplimiento por característica](https://azure.microsoft.com/support/trust-center/services/) en [Centro de confianza de Microsoft Azure ](https://azure.microsoft.com/support/trust-center/).
> Dado que las opciones de protección de datos usan un enfoque multicapa, la comparación entre ellas no se puede aplicar a esta tarea. Asegúrese de que saca provecho de todas las opciones disponibles en cada estado de los datos.
>
>

## <a name="define-content-management-options"></a>Definición de las opciones de administración de contenido

Una ventaja de usar Azure AD para administrar una infraestructura de identidad híbrida es que el proceso es completamente transparente desde la perspectiva del usuario final. El usuario intenta acceder a un recurso compartido, pero el recurso requiere autenticación, por lo que el usuario tendrá que enviar una solicitud de autenticación a Azure AD para obtener el token y el acceso al recurso. Todo este proceso se produce en segundo plano, sin interacción del usuario. 

Las organizaciones a las que preocupa la privacidad de los datos requieren la clasificación de datos para su solución. Si su infraestructura local actual ya usa la clasificación de datos, es posible usar Azure AD como repositorio principal de la identidad del usuario. Una herramienta común que se usa localmente para la clasificación de datos se denomina [Data Classification Toolkit](/previous-versions/tn-archive/hh204743(v=technet.10)) para Windows Server 2012 R2. Esta herramienta puede ayudar a identificar, clasificar y proteger los datos de los servidores de archivos de su nube privada. También es posible usar la [clasificación de archivos automática](/windows-server/identity/solution-guides/deploy-automatic-file-classification--demonstration-steps-) en Windows Server 2012 para lograrlo.

Si su organización no tiene la clasificación de los datos implementada, pero necesita proteger archivos confidenciales sin agregar nuevos servidores localmente, puede usar el [servicio Azure Rights Management](/azure/information-protection/what-is-azure-rms)de Microsoft.  Azure RMS usa las directivas de cifrado, identidad y autorización para ayudarle a proteger sus archivos y mensajes de correo electrónico, y funciona en varios dispositivos (teléfonos, tabletas y PC). Dado que Azure RMS es un servicio en la nube, no hay necesidad de configurar explícitamente relaciones de confianza con otras organizaciones para poder compartir contenido protegido con ellas. Si ya tienen un directorio de Office 365 o Azure AD, la colaboración entre organizaciones se admite automáticamente. También se pueden sincronizar solo los atributos de directorio que Azure RMS necesita para admitir una identidad común para las cuentas de Active Directory locales, mediante el uso de los servicios de sincronización de Azure Active Directory (sincronización de AAD) o Azure AD Connect.

Una parte fundamental de la administración de contenido es saber quién tiene acceso a cada recurso; por lo tanto, es importante que la solución de administración de identidades tenga una gran capacidad de registro. Azure AD proporciona registro durante 30 días. Dicho registro incluye:

* Cambios en la pertenencia a un rol (por ejemplo, un usuario agregado al rol de administrador global)
* Actualizaciones de credenciales (por ejemplo, cambios de contraseña)
* Administración de dominios (por ejemplo, comprobar un dominio personalizado o eliminar un dominio)
* Adición o eliminación de aplicaciones
* Administración de usuarios (por ejemplo, agregar, eliminar o actualizar un usuario)
* Adición o eliminación de licencias

> [!NOTE]
> Para obtener más información acerca de las capacidades de registro de Azure, consulte el documento [Microsoft Azure Security and Audit Log Management](https://download.microsoft.com/download/B/6/C/B6C0A98B-D34A-417C-826E-3EA28CDFC9DD/AzureSecurityandAuditLogManagement_11132014.pdf) (Seguridad de Microsoft Azure y administración del registro de auditoría).
> En función de cómo respondiera a las preguntas de [Determinación de los requisitos de administración de contenido](plan-hybrid-identity-design-considerations-contentmgt-requirements.md), podría determinar cómo desea administrar el contenido en una solución de identidad híbrida. Aunque todas las opciones que se exponen en la tabla 6 pueden integrarse con Azure AD, es importante definir la que sea más adecuada para sus necesidades empresariales.
>
>

| Opciones de administración de contenido | Ventajas | Inconvenientes |
| --- | --- | --- |
| Centralizada localmente (servidor de Active Directory Rights Management) |Control total sobre la infraestructura de servidor responsable de clasificar los datos <br> Funcionalidad integrada de Windows Server, sin necesidad de suscripción o de licencia adicional <br> Se puede integrar con Azure AD en un escenario híbrido. <br> Admite funcionalidades de Information Rights Management (IRM) en Microsoft Online Services como Exchange Online y SharePoint Online, así como Office 365. <br> Admite productos de servidor de Microsoft locales, como Exchange Server , SharePoint Server y servidores de archivo que ejecutan Windows Server e infraestructura de clasificación de archivos (FCI). |Mayor mantenimiento (para estar al día de las actualizaciones, la configuración y las actualizaciones potenciales), ya que el departamento de TI posee el servidor. <br> Requiere una infraestructura de servidor local.<br> No saca provecho de las funcionalidades de Azure de forma nativa. |
| Centralizada en la nube (Azure RMS) |Más fácil de administrar, en comparación con la solución local <br> Se puede integrar con AD DS en un escenario híbrido. <br>  Completamente integrado con Azure AD. <br> No requiere un servidor local para implementar el servicio <br> Admite productos de servidor de Microsoft locales, como Exchange Server, SharePoint Server y servidores de archivo que ejecuten Windows Server y la infraestructura de clasificación de archivos (FCI). <br> El departamento de TI puede tener control completo sobre la clave de inquilino con funcionalidad de BYOK. |Su organización debe tener una suscripción de nube que admita RMS <br> Su organización debe tener un directorio de Azure AD para admitir la autenticación de usuario de RMS |
| Híbrida (Azure RMS integrado con el servidor de Active Directory Rights Management local) |Este escenario acumula las ventajas de ambos, local centralizado y en la nube. |Su organización debe tener una suscripción de nube que admita RMS <br> Su organización debe tener un directorio de Azure AD para admitir la autenticación de usuario de RMS. <br> Requiere una conexión entre el servicio en la nube de Azure y la infraestructura local |

## <a name="define-access-control-options"></a>Definición de opciones de control de acceso
Al usar las funcionalidades de autenticación, autorización y control de acceso disponibles en Azure AD, puede permitir que su empresa use un repositorio central de identidades, al mismo tiempo que permite que los usuarios y asociados usen el inicio de sesión único (SSO), tal como se muestra en la figura siguiente:

![Administración centralizada.](./media/plan-hybrid-identity-design-considerations/centralized-management.png)

Administración centralizada e integración total con otros directorios

Azure Active Directory ofrece un inicio de sesión único a miles de aplicaciones SaaS y aplicaciones web locales. Para obtener más información sobre los SSO de terceros que Microsoft probó, consulte el artículo [Azure Active Directory federation compatibility list: third-party identity providers that can be used to implement single sign-on](how-to-connect-fed-compatibility.md) (Lista de compatibilidad de federación de Azure Active Directory: proveedores de identidades de terceros que se pueden usar para implementar el inicio de sesión único). Esta capacidad permite a la organización implementar varios escenarios de B2B sin perder el control de la administración de identidades y acceso. Sin embargo, durante el proceso de diseño B2B es importante conocer el método de autenticación que usará el asociado y validarlo, en caso de que sea compatible con Azure. Actualmente, Azure AD admite los siguientes métodos:

* Lenguaje de marcado de aserción de seguridad (SAML)
* OAuth
* Kerberos
* Tokens
* Certificados

> [!NOTE]
> Para obtener más información sobre cada protocolo y sus funcionalidades en Azure, consulte [Protocolos de autenticación de Azure Active Directory](/previous-versions/azure/dn151124(v=azure.100)) .
>
>

Gracias a la compatibilidad con Azure AD, las aplicaciones empresariales pueden usar la misma experiencia de autenticación fácil de Mobile Services para permitir a los empleados iniciar sesión en sus aplicaciones móviles con sus credenciales corporativas de Active Directory. Con esta característica, Azure AD se admite como proveedor de identidades en Mobile Services, junto con los otros proveedores de identidades que ya son compatibles (entre los que se incluyen las cuentas de Microsoft, el identificador de Facebook, el identificador de Google y el identificador de Twitter). Si las aplicaciones locales usan las credenciales del usuario ubicadas en el servicio AD DS de la empresa, el acceso de los asociados y los usuarios que provienen de la nube debe ser transparente. Puede administrar el control de acceso condicional del usuario a las aplicaciones web (basadas en la nube), la API web, los Servicios en la nube de Microsoft, las aplicaciones SaaS de terceros y las aplicaciones cliente nativas (móviles) e igualmente obtener todas las ventajas de seguridad, auditoría y generación de informes en un mismo lugar. Sin embargo, se recomienda validar la implementación en un entorno que no sea de producción o que tenga una cantidad limitada de usuarios.

> [!TIP]
> Es importante mencionar que Azure AD no tiene una directiva de grupo, mientras que AD DS sí la tiene. Para aplicar una directiva a los dispositivos, es preciso tener una solución de administración de dispositivos móviles como [Microsoft Intune](/mem/intune/).
>
>

Una vez que el usuario se autentica mediante Azure AD, es importante evaluar el nivel de acceso que tendrá este usuario. Tenga en cuenta que el nivel de acceso que tendrá el usuario en un recurso puede variar. Aunque Azure AD puede agregar un nivel de seguridad adicional mediante el control del acceso a algunos recursos, es preciso recordar que el recurso también puede tener su propia lista independiente de control de acceso como, por ejemplo, el control de acceso a los archivos que estén ubicados en un servidor de archivos. En la siguiente ilustración se resumen los niveles de control de acceso que puede haber en un escenario híbrido:

![control de acceso](./media/plan-hybrid-identity-design-considerations/accesscontrol.png)

Cada una de las interacciones del diagrama que se muestra en la Ilustración X representa un escenario de control de acceso que se pueden cubrir con Azure AD. A continuación encontrará una descripción de cada escenario:

1. Acceso condicional a las aplicaciones hospedadas localmente: puede usar dispositivos registrados con directivas de acceso en las aplicaciones configuradas para usar AD FS con Windows Server 2012 R2.

2. Control de acceso en Azure Portal:  Azure también le permite controlar el acceso al portal mediante el control de acceso basado en role de Azure (RBAC de Azure). Este método permite a la empresa restringir la cantidad de operaciones que una persona puede hacer en Azure Portal. Si se usa RBAC de Azure para controlar el acceso al portal, los administradores de TI pueden delegar el acceso con los siguientes enfoques de administración de acceso:

   - Asignación de roles basada en grupo: puede asignar acceso a los grupos de Azure AD que se pueden sincronizar desde su Active Directory local. Esto le permite aprovechar las inversiones que su organización realizó en herramientas y procesos para administrar grupos. También puede usar la característica de administración de grupos delegados de Azure AD Premium.
   - Usar los roles integrados en Azure: puede usar tres roles (Propietario, Colaborador y Lector) para asegurarse de que los usuarios y los grupos tienen permiso para realizar únicamente las tareas necesarias para sus respectivos trabajos.
   -  Acceso granular a los recursos: puede asignar roles a los usuarios y grupos para una suscripción concreta, un grupo de recursos o un recurso individual de Azure como un sitio web o una base de datos. De esta forma, puede asegurarse de que los usuarios tengan acceso a todos los recursos que necesitan y ningún acceso a los que no necesitan administrar.

   > [!NOTE]
   > Si va a crear aplicaciones y desea personalizar el control de acceso a ellas, también pueden usar los roles de aplicación de Azure AD para la autorización. Para usar esta capacidad, revise el [ejemplo WebApp-RoleClaims-DotNet](https://github.com/AzureADSamples/WebApp-RoleClaims-DotNet) de creación de una aplicación.


3. Acceso condicional para aplicaciones de Office 365 con Microsoft Intune:  Los administradores de TI pueden aprovisionar directivas de dispositivos de acceso condicional para proteger los recursos corporativos, al mismo tiempo que permiten que los trabajadores de la información con dispositivos compatibles tengan acceso a los servicios. 
  
4. Acceso condicional para aplicaciones SaaS: [esta característica](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/25/azure-ad-conditional-access-preview-update-more-apps-and-blocking-access-for-users-not-at-work/) permite la configuración de reglas de acceso de autenticación multifactor por aplicación y la capacidad de bloquear el acceso de los usuarios en una red que no es de confianza. La regla de autenticación multifactor puede aplicarse a todos los usuarios que están asignados a la aplicación o solo a los que pertenecen a grupos de seguridad especificados. Los usuarios pueden excluirse del requisito de autenticación multifactor si van a tener acceso a la aplicación desde una dirección IP que se encuentre dentro de la red de la organización.

Dado que las opciones de control de acceso usan un enfoque multicapa, la comparación entre ellas no se puede aplicar a esta tarea. Asegúrese de que saque provecho a todas las opciones disponibles para cada escenario que requiera que controle el acceso a sus recursos.

## <a name="define-incident-response-options"></a>Definición de las opciones de respuesta ante incidentes
Azure AD puede ayudar al departamento de TI a identificar posibles riesgos de seguridad en el entorno mediante la supervisión de las actividades del usuario. Asimismo, el departamento de TI puede usar los informes de acceso y uso de Azure AD para proporcionar visibilidad sobre la integridad y la seguridad del directorio de la organización. Con esta información, un administrador de TI puede determinar mejor dónde puede haber posibles riesgos de seguridad, con el fin de que puedan planear adecuadamente la mitigación de dichos riesgos.  [suscripción a Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) tiene un conjunto de informes de seguridad que pueden permitir al departamento de TI obtener esta información. Los [informes de Azure AD](../reports-monitoring/overview-reports.md) se clasifican tal como se muestra a continuación:

* **Informes de anomalías**: contienen eventos de inicio de sesión que se consideran anómalos. El objetivo aquí es que sea consciente de dicha actividad y que pueda tomar una decisión sobre si un evento es sospechoso.
* **Informe de aplicaciones integradas**: proporciona información sobre cómo se usan en la organización las aplicaciones en la nube. Azure Active Directory ofrece integración con miles de aplicaciones en la nube.
* **Informes de errores**: indican errores que se pueden producir al aprovisionar cuentas a aplicaciones externas.
* **Informes específicos del usuario**: muestran los datos de actividad de dispositivo o de inicio de sesión de un usuario concreto.
* **Registros de actividad**: contienen un registro de todos los eventos auditados en las últimas 24 horas, los últimos 7 días o los últimos 30 días, así como los cambios en la actividad del grupo y la actividad de registro y de restablecimiento de contraseña.

> [!TIP]
> Otro informe que también puede ayudar al equipo de respuesta ante incidentes que trabaja en un caso es el informe de [usuarios con credenciales perdidas](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/15/azure-active-directory-premium-reporting-now-detects-leaked-credentials/) . Este informe muestra las coincidencias entre la lista de credenciales perdidas y el inquilino.
>


Otros informes importantes integrados en Azure AD que pueden usarse durante la investigación de la respuesta ante un incidente son:

* **Actividad de restablecimiento de contraseña**: proporciona al administrador información sobre si el restablecimiento de contraseña se usa activamente en la organización.
* **Actividad de registro de restablecimiento de contraseña**: proporciona información sobre los usuarios que registraron sus métodos para restablecer la contraseña y los métodos que seleccionados.
* **Actividad de grupo**: proporciona un historial de los cambios en el grupo (por ejemplo, los usuarios agregados o eliminados) que se iniciaron en el Panel de acceso.

Además de la funcionalidad principal de generación de informes de Azure AD Premium que puede usar durante un proceso de investigación de respuesta ante incidentes, el departamento de TI también puede sacar provecho del informe de auditoría para obtener información como:

* Cambios en la pertenencia a un rol (por ejemplo, un usuario agregado al rol de administrador global).
* Actualizaciones de credenciales (por ejemplo, cambios de contraseña).
* Administración de dominios (por ejemplo, verificar un dominio personalizado o eliminar un dominio).
* Adición o eliminación de aplicaciones
* Administración de usuarios (por ejemplo, agregar, eliminar o actualizar un usuario).
* Adición o eliminación de licencias

Dado que las opciones para la respuesta ante incidentes usan un enfoque multicapa, la comparación entre ellas no se puede aplicar a esta tarea. Asegúrese de que saca provecho de todas las opciones disponibles para cada escenario que requiere que se use la capacidad de generación de informes de Azure AD como parte del proceso de respuesta ante incidentes de su compañía.

## <a name="next-steps"></a>Pasos siguientes
[Determinación de las tareas de administración de identidades híbridas](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md)

## <a name="see-also"></a>Consulte también
[Información general sobre las consideraciones de diseño](plan-hybrid-identity-design-considerations-overview.md)