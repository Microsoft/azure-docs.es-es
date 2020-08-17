---
title: 'Creación de una estrategia de administración de control de acceso resistente: Azure AD'
description: En este documento se proporcionan instrucciones sobre las estrategias que una organización debe adoptar para proporcionar resistencia para reducir el riesgo de bloqueo durante interrupciones imprevistas
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.workload: identity
ms.date: 06/08/2020
ms.author: martinco
ms.collection: M365-identity-device-management
ms.openlocfilehash: ec20a1bda8021e61f5147142a8e6bddd6cf5d166
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87027621"
---
# <a name="create-a-resilient-access-control-management-strategy-with-azure-active-directory"></a>Crear una estrategia de administración de control de acceso resistente con Azure Active Directory

>[!NOTE]
> La información contenida en este documento representa la visión actual de Microsoft Corporation sobre los asuntos tratados a fecha de su publicación. Dado que Microsoft debe responder a condiciones de mercado variables, no debe interpretarse como un compromiso por parte de Microsoft, y Microsoft no puede garantizar la precisión de la información presentada tras la fecha de publicación.

Las organizaciones que se basan en un control de acceso único, como la autenticación multifactor (MFA) o una ubicación de red única, para proteger sus sistemas de TI, son susceptibles a los errores de acceso a sus aplicaciones y recursos si ese control de acceso no está disponible o está mal configurado. Por ejemplo, un desastre natural puede tener como resultado que dejen de estar disponibles grandes segmentos de la infraestructura de telecomunicaciones o las redes corporativas. Una interrupción de este tipo podría impedir el inicio de sesión a los usuarios finales y a los administradores.

En este documento se proporcionan instrucciones sobre las estrategias que una organización debe adoptar para proporcionar resistencia para reducir el riesgo de bloqueo durante interrupciones imprevistas con los siguientes escenarios:

 1. Las organizaciones pueden aumentar su resistencia para reducir el riesgo de bloqueo **antes de una interrupción** mediante la implementación de estrategias de mitigación o planes de contingencia.
 2. Las organizaciones pueden acceder a aplicaciones y recursos que elijan **durante una interrupción** estableciendo estrategias de mitigación y planes de contingencia.
 3. Las organizaciones deben asegurarse de que conservan la información, como los registros, **después de una interrupción** y antes de que reviertan las contingencias que implementaron.
 4. Es posible que las organizaciones que no hayan implementado estrategias de prevención o planes alternativos puedan implementar **opciones de emergencia** para gestionar la interrupción.

## <a name="key-guidance"></a>Instrucciones clave

Hay cuatro puntos clave en este documento:

* Evite el bloqueo de administrador mediante el uso de cuentas de acceso de emergencia.
* Implemente MFA mediante el acceso condicional (CA) en lugar de MFA por usuario.
* Mitigue el bloqueo de usuario mediante el uso de varios controles de acceso condicional (CA).
* Mitigue el bloqueo de usuario mediante el aprovisionamiento de varios métodos de autenticación o equivalentes para cada usuario.

## <a name="before-a-disruption"></a>Antes de una interrupción

Mitigar una interrupción real debe ser el objetivo principal de la organización a la hora de lidiar con problemas de control de acceso que puedan surgir. La mitigación incluye la planeación de un evento real además de implementar estrategias para hacer que los controles de acceso seguro y las operaciones no se vean afectados durante las interrupciones.

### <a name="why-do-you-need-resilient-access-control"></a>¿Por qué necesita un control de acceso resistente?

 La identidad es el plano de control de los usuarios que acceden a aplicaciones y recursos. El sistema de identidad controla qué usuarios obtienen acceso a las aplicaciones y en qué condiciones, como los controles de acceso o los requisitos de autenticación. Cuando uno o varios requisitos de control de acceso o autenticación no están disponibles para que los usuarios se autentiquen debido a una circunstancia imprevista, las organizaciones pueden experimentar uno de los siguientes problemas, o ambos:

* **Bloqueo de administrador:** Los administradores no pueden administrar el inquilino o los servicios.
* **Bloqueo de usuario:** Los usuarios no pueden acceder a aplicaciones o recursos.

### <a name="administrator-lockout-contingency"></a>Medida de contingencia frente al bloqueo de administrador

Para desbloquear el acceso de administrador a su inquilino, debe crear cuentas de acceso de emergencia. Estas cuentas de acceso de emergencia, también conocidas como cuentas *de emergencia*, permiten el acceso para administrar la configuración de Azure AD cuando los procedimientos de acceso con cuentas con privilegios normales no están disponibles. Se deben crear al menos dos cuentas de acceso de emergencia siguiendo las [recomendaciones sobre cuentas de acceso de emergencia]( https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-emergency-access).

### <a name="mitigating-user-lockout"></a>Mitigación del bloqueo de usuario

 Para mitigar el riesgo de bloqueo de usuario, use directivas de acceso condicional con varios controles para proporcionar a los usuarios una opción para acceder a los recursos y aplicaciones. Al permitir a un usuario elegir entre, por ejemplo, iniciar sesión con MFA **o** iniciar sesión desde un dispositivo administrado **o** iniciar sesión desde la red corporativa, si uno de los controles de acceso no está disponible el usuario tiene otras opciones para continuar trabajando.

#### <a name="microsoft-recommendations"></a>Recomendaciones de Microsoft

Incorpore los siguientes controles de acceso en las directivas de acceso condicional existentes para la organización:

1. Aprovisione varios métodos de autenticación para cada usuario que se basan en diferentes canales de comunicación, por ejemplo la aplicación Microsoft Authenticator (basada en Internet), el token de OATH (generado en el dispositivo) y SMS (telefónica). El siguiente script de PowerShell le ayudará a identificar de antemano los métodos adicionales que los usuarios deben registrar: [Script para el análisis de los métodos de autenticación de Azure MFA](https://docs.microsoft.com/samples/azure-samples/azure-mfa-authentication-method-analysis/azure-mfa-authentication-method-analysis/).
2. Implemente Windows Hello para empresas en dispositivos Windows 10 para satisfacer los requisitos de MFA directamente desde el inicio de sesión de dispositivo.
3. Use dispositivos de confianza a través de [Unión a Azure AD híbrido](https://docs.microsoft.com/azure/active-directory/devices/overview) o de [dispositivos administrados de Microsoft Intune](https://docs.microsoft.com/intune/planning-guide). Los dispositivos de confianza mejorarán la experiencia del usuario porque el dispositivo de confianza puede satisfacer los requisitos de autenticación sólida de la directiva sin un desafío de MFA al usuario. Después MFA se solicitará al inscribir un dispositivo nuevo y al acceder a aplicaciones o recursos desde dispositivos de confianza.
4. Use directivas basadas en riesgos de protección de identidad de Azure AD que impidan el acceso cuando el usuario o el inicio de sesión esté en riesgo, en lugar de las directivas MFA fijas.
5. Si va a proteger el acceso de VPN mediante la extensión NPS de Azure MFA, considere la posibilidad de federar la solución VPN como una [aplicación SAML](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-single-sign-on-non-gallery-applications) y determinar la categoría de la aplicación como se recomienda a continuación. 

>[!NOTE]
> Las directivas basadas en riesgos requieren licencias [Azure AD Premium P2](https://azure.microsoft.com/pricing/details/active-directory/).

El siguiente ejemplo describe las directivas que debe crear para proporcionar un control de acceso resistente para que los usuarios accedan a sus aplicaciones y recursos. En este ejemplo, necesitará un grupo de seguridad **AppUsers** con los usuarios de destino a los que quiere dar acceso, un grupo denominado **CoreAdmins** con los administradores principales y un grupo denominado  **EmergencyAccess** con las cuentas de acceso de emergencia.
Este conjunto de directivas de ejemplo concederá a los usuarios seleccionados en **AppUsers** acceso a las aplicaciones seleccionadas si se están conectando desde un dispositivo de confianza o proporcionan una autenticación sólida, como por ejemplo MFA. Excluye las cuentas de emergencias y los administradores principales.

**Conjunto de directivas de mitigación de CA:**

* Directiva 1: Bloquee el acceso a los usuarios ajenos a los grupos de destino
  * Usuarios y grupos: Incluya a todos los usuarios. Excluya a AppUsers, a CoreAdmins y a EmergencyAccess
  * Aplicaciones en la nube: Incluya todas las aplicaciones
  * Condiciones: (Ninguna)
  * Conceder control: Block
* Directiva 2: Conceda acceso a AppUsers requiriendo MFA o un dispositivo de confianza.
  * Usuarios y grupos: Incluya a AppUsers. Excluya a CoreAdmins y a EmergencyAccess
  * Aplicaciones en la nube: Incluya todas las aplicaciones
  * Condiciones: (Ninguna)
  * Conceder control: Conceda acceso, requiera autenticación multifactor, requiera que el dispositivo sea compatible. Para varios controles: Requiera uno de los controles seleccionados.

### <a name="contingencies-for-user-lockout"></a>Contingencias para el bloqueo de usuario

Como alternativa, su organización también puede crear directivas de contingencia. Para crear directivas de contingencia, debe definir los criterios de equilibrio entre la continuidad del negocio, los costos operacionales, los costos financieros y los riesgos de seguridad. Por ejemplo, puede activar una directiva de contingencia solo a un subconjunto de usuarios, para un subconjunto de aplicaciones, para un subconjunto de clientes, o desde un subconjunto de ubicaciones. Las directivas de contingencia darán acceso a los administradores y usuarios finales a aplicaciones y recursos, durante una interrupción cuando no se implementase ningún método de mitigación. Microsoft recomienda habilitar las directivas de contingencia en [modo de solo informe](https://docs.microsoft.com/azure/active-directory/conditional-access/howto-conditional-access-report-only) cuando no se usan, para que los administradores puedan supervisar el posible impacto de las directivas en caso de que deban activarse.

 Comprender la exposición durante una interrupción ayuda a reducir el riesgo y es una parte fundamental del proceso de planeación. Para crear el plan de contingencia, primero determine los siguientes requisitos empresariales de su organización:

1. Determine las aplicaciones críticas con antelación: ¿Cuáles son las aplicaciones a las que debe dar acceso, incluso con una postura de riesgo de seguridad más baja? Cree una lista de estas aplicaciones y asegúrese de que las demás partes interesadas (empresa, seguridad, legales, dirección) están de acuerdo con que, si se pierde todo el control de acceso, estas aplicaciones deberán seguir ejecutándose. Probablemente acabará teniendo categorías de:
   * **Categoría 1 aplicaciones críticas** que no pueden dejar de estar disponibles durante más de unos minutos, por ejemplo las aplicaciones que afectan directamente a los ingresos de la organización.
   * **Categoría 2 aplicaciones importantes** a las que la empresa necesita poder acceder en el plazo de unas pocas horas.
   * **Categoría 3 aplicaciones de prioridad baja** que pueden resistir una interrupción de unos días.
2. Para las aplicaciones en las categorías 1 y 2, Microsoft recomienda planear previamente el tipo de nivel de acceso que desea permitir:
   * ¿Desea permitir acceso completo o sesiones restringidas, como limitar las descargas?
   * ¿Desea permitir el acceso a parte de la aplicación, pero no a toda la aplicación?
   * ¿Desea permitir el acceso de trabajador de información y bloquear el acceso de administrador hasta que se restaure el control de acceso?
3. Para esas aplicaciones, Microsoft también recomienda que planee qué vías de acceso abrirá deliberadamente y cuáles cerrará:
   * ¿Desea permitir el acceso solo mediante explorador y bloquear clientes enriquecidos que pueden guardar datos sin conexión?
   * ¿Desea permitir el acceso únicamente a los usuarios dentro de la red corporativa y mantener fuera a los usuarios bloqueados?
   * ¿Desea limitar el acceso desde determinados países o regiones durante la interrupción?
   * ¿Desea que las directivas de contingencia, especialmente para aplicaciones críticas, produzcan un error o funcionen correctamente si un control de acceso alternativo no está disponible?

#### <a name="microsoft-recommendations"></a>Recomendaciones de Microsoft

Una directiva de acceso condicional de contingencia es una **directiva de respaldo** que omite los controles de Azure MFA, MFA de terceros y los controles basados en el riesgo y en el dispositivo. Para minimizar una interrupción inesperada cuando se habilita una directiva de contingencia, la directiva debe permanecer en modo de solo informe cuando no esté en uso. Los administradores pueden supervisar el impacto potencial de las directivas de contingencia mediante el libro de información del acceso condicional. Cuando la organización decida activar el plan de contingencia, los administradores pueden habilitar la directiva y deshabilitar las directivas normales basadas en controles.

>[!IMPORTANT]
> Deshabilitar directivas que aplican seguridad en los usuarios, incluso de forma temporal, reducirá la postura de seguridad mientras se aplique el plan de contingencia.

* Configure un conjunto de directivas de reserva si una interrupción en un tipo de credencial o un mecanismo de control de acceso afecta al acceso a sus aplicaciones. Configure una directiva en estado de solo informe que requiera la unión a un dominio como un control como una copia de seguridad para una directiva activa que requiera un proveedor de MFA de terceros.
* Reduzca el riesgo de que actores malintencionados adivinen las contraseñas cuando no se requiere MFA, siguiendo las prácticas en la documentación técnica de [Password Guidance](https://aka.ms/passwordguidance) (Instrucciones sobre contraseñas).
* Implemente el [Autoservicio de restablecimiento de contraseña de Azure AD (SSPR)](https://docs.microsoft.com/azure/active-directory/authentication/quickstart-sspr) y la [Protección con contraseña de Azure AD](https://docs.microsoft.com/azure/active-directory/authentication/howto-password-ban-bad-on-premises-deploy) para asegurarse de que los usuarios no usan contraseñas comunes y términos que decide prohibir.
* Use directivas que restrinjan el acceso dentro de las aplicaciones si no se alcanza un cierto nivel de autenticación en lugar de simplemente retroceder al acceso completo. Por ejemplo:
  * Configure una directiva de copia de seguridad que envíe la reclamación de sesión restringida a Exchange y SharePoint.
  * Si la organización usa Microsoft Cloud App Security, considere la posibilidad de recurrir a una directiva que interactúe con MCAS y después MCAS permita acceso de solo lectura, pero no cargas.
* Ponga nombre a las directivas para encontrarlas con facilidad durante una interrupción. Incluya los siguientes elementos en el nombre de la directiva:
  * Un *número de etiqueta* para la directiva.
  * Texto para mostrar, esta directiva es solo para casos de emergencia. Por ejemplo: **HABILITAR EN CASO DE EMERGENCIA**
  * La *interrupción* a la que se aplica. Por ejemplo: **Durante una interrupción de MFA**
  * Un *número de secuencia* para mostrar el orden en que debe activar las directivas.
  * Las *aplicaciones* a las que se aplica.
  * Los *controles* que aplicará.
  * Las *condiciones* que requiere.
  
Este estándar de nomenclatura para las directivas de contingencia será del modo siguiente: 

```
EMnnn - ENABLE IN EMERGENCY: [Disruption][i/n] - [Apps] - [Controls] [Conditions]
```

En el ejemplo siguiente: **Ejemplo A: directiva de CA de contingencia para restaurar el acceso a aplicaciones de colaboración críticas**, es una contingencia corporativa típica. En este escenario, la organización normalmente requiere MFA para todos los accesos de Exchange Online y SharePoint Online y, en este caso, la interrupción se da porque el proveedor de MFA para el cliente tiene una interrupción del servicio (ya sea Azure MFA, un proveedor de MFA local o MFA de terceros). Esta directiva mitiga esta interrupción al permitir que los usuarios de destino específicos accedan a estas aplicaciones desde dispositivos Windows de confianza solo cuando acceden a la aplicación desde su red corporativa de confianza. También excluirá las cuentas de emergencia y los administradores principales de estas restricciones. Los usuarios de destino tendrán entonces acceso a Exchange Online y SharePoint Online, mientras que otros usuarios seguirán sin tener acceso a las aplicaciones debido a la interrupción. En este ejemplo se necesita una ubicación de red denominada **CorpNetwork** y un grupo de seguridad **ContingencyAccess** con los usuarios de destino, un grupo denominado **CoreAdmins** con los administradores principales y un grupo denominado **EmergencyAccess** con las cuentas de acceso de emergencia. La contingencia requiere cuatro directivas para proporcionar el acceso deseado. 

**Ejemplo A. Directiva de CA de contingencia para restaurar el acceso a aplicaciones de colaboración críticas:**

* Directiva 1: Requiera dispositivos unidos a un dominio para Exchange y SharePoint
  * Nombre: EM001 - HABILITAR EN CASO DE EMERGENCIA: Interrupción de MFA [1/4]: Exchange SharePoint: Requerir la unión a Azure AD híbrido
  * Usuarios y grupos: Incluya a ContingencyAccess. Excluya a CoreAdmins y a EmergencyAccess
  * Aplicaciones en la nube: Exchange Online y SharePoint Online
  * Condiciones: Any
  * Conceder control: Requerir unión a un dominio
  * Estado: Solo informe
* Directiva 2: Plataformas de bloque que no sean de Windows
  * Nombre: EM002 - HABILITAR EN CASO DE EMERGENCIA: Interrupción de MFA [2/4] - Exchange SharePoint: Bloquear el acceso excepto a Windows
  * Usuarios y grupos: Incluya a todos los usuarios. Excluya a CoreAdmins y a EmergencyAccess
  * Aplicaciones en la nube: Exchange Online y SharePoint Online
  * Condiciones: Las plataformas de dispositivo incluyen todas las plataformas, excepto Windows
  * Conceder control: Block
  * Estado: Solo informe
* Directiva 3: Bloque de redes que no sean CorpNetwork
  * Nombre: EM003 - HABILITAR EN CASO DE EMERGENCIA: Interrupción de MFA [3/4] - Exchange SharePoint: Bloquear el acceso excepto a la red corporativa
  * Usuarios y grupos: Incluya a todos los usuarios. Excluya a CoreAdmins y a EmergencyAccess
  * Aplicaciones en la nube: Exchange Online y SharePoint Online
  * Condiciones: Las ubicaciones incluyen cualquier ubicación, excepto CorpNetwork
  * Conceder control: Block
  * Estado: Solo informe
* Directiva 4: Bloquear explícitamente EAS
  * Nombre: EM004 - HABILITAR EN CASO DE EMERGENCIA: Interrupción de MFA [4/4] - Exchange: Bloquear EAS para todos los usuarios
  * Usuarios y grupos: Incluir a todos los usuarios
  * Aplicaciones en la nube: Incluir a Exchange Online
  * Condiciones: Aplicaciones cliente: Exchange Active Sync
  * Conceder control: Block
  * Estado: Solo informe

Orden de activación:

1. Excluya ContingencyAccess, CoreAdmins y EmergencyAccess de la directiva de MFA existente. Compruebe que un usuario en ContingencyAccess puede acceder a SharePoint Online y Exchange Online.
2. Habilitar Directiva 1: Compruebe que los usuarios en dispositivos unidos a un dominio que no están en los grupos de exclusión pueden acceder a Exchange Online y SharePoint Online. Compruebe que los usuarios del grupo de exclusión pueden acceder a SharePoint Online y Exchange desde cualquier dispositivo.
3. Habilitar Directiva 2: Compruebe que los usuarios que no están en el grupo de exclusión no pueden acceder a SharePoint Online y Exchange Online desde sus dispositivos móviles. Compruebe que los usuarios del grupo de exclusión pueden acceder a SharePoint y Exchange desde cualquier dispositivo (Windows, iOS o Android).
4. Habilitar Directiva 3: Compruebe que los usuarios que no están en los grupos de exclusión no pueden acceder a SharePoint y Exchange fuera de la red corporativa, incluso con una máquina unida a dominio. Compruebe que los usuarios del grupo de exclusión pueden acceder a SharePoint y Exchange desde cualquier red.
5. Habilitar Directiva 4: Compruebe que ningún usuario puede acceder a Exchange Online desde las aplicaciones de correo nativo en dispositivos móviles.
6. Deshabilite la directiva de MFA existente para SharePoint Online y Exchange Online.

En el ejemplo siguiente, **Ejemplo B. Directivas de CA de contingencia para permitir el acceso móvil a Salesforce**, se restaurará el acceso de una aplicación empresarial. En este escenario, el cliente normalmente requiere que el acceso de los empleados de ventas a Salesforce (configurada para inicio de sesión único con Azure AD) desde dispositivos móviles solo se permita desde dispositivos compatibles. La interrupción en este caso es que hay un problema con la evaluación del cumplimiento de dispositivos y la interrupción se produce en un momento delicado en el que el equipo de ventas necesita acceder a Salesforce para cerrar acuerdos. Estas directivas de contingencia conceden a los usuarios críticos acceso a Salesforce desde un dispositivo móvil para que sigan cerrando tratos y, de esta forma, que el negocio no se vea afectado. En este ejemplo, **SalesforceContingency** contiene todos los empleados de ventas que necesitan conservar el acceso y **SalesAdmins** contiene los administradores necesarios de Salesforce.

**Ejemplo B. Directivas de CA de contingencia:**

* Directiva 1: Bloquear a todos los usuarios que no están en el equipo SalesContingency
  * Nombre: EM001 - HABILITAR EN CASO DE EMERGENCIA: Interrupción del cumplimiento de dispositivo [1/2]: Salesforce: Bloquear todos los usuarios excepto SalesforceContingency
  * Usuarios y grupos: Incluya a todos los usuarios. Excluir a SalesAdmins y a SalesforceContingency
  * Aplicaciones en la nube: Salesforce.
  * Condiciones: None
  * Conceder control: Block
  * Estado: Solo informe
* Directiva 2: Bloquear el acceso del equipo de ventas desde cualquier plataforma que no sea móvil (para reducir la superficie de ataque)
  * Nombre: EM002 - HABILITAR EN CASO DE EMERGENCIA: Interrupción del cumplimiento de dispositivo [2/2]: Salesforce: Bloquear todas las plataformas excepto iOS y Android
  * Usuarios y grupos: Incluya a SalesforceContingency. Excluir a SalesAdmins
  * Aplicaciones en la nube: Salesforce
  * Condiciones: Las plataformas de dispositivo incluyen todas las plataformas, excepto iOS y Android
  * Conceder control: Block
  * Estado: Solo informe

Orden de activación:

1. Excluya a SalesAdmins y a SalesforceContingency de la directiva de cumplimiento de dispositivo existente para Salesforce. Compruebe que un usuario en el grupo SalesforceContingency puede acceder a Salesforce.
2. Habilitar Directiva 1: Compruebe que los usuarios fuera de SalesContingency no pueden acceder a esta aplicación. Compruebe que los usuarios de SalesAdmins y SalesforceContingency pueden acceder a Salesforce.
3. Habilitar Directiva 2: Compruebe que los usuarios del grupo SalesContingency no pueden acceder a Salesforce desde sus equipos portátiles Windows o Mac pero todavía pueden acceder desde sus dispositivos móviles. Compruebe que SalesAdmin todavía puede acceder a Salesforce desde cualquier dispositivo.
4. Deshabilite la directiva de cumplimiento de dispositivo existente para Salesforce.

### <a name="contingencies-for-user-lockout-from-on-prem-resources-nps-extension"></a>Contingencias para el bloqueo de usuarios de los recursos locales (extensión NPS)

Si va a proteger el acceso de VPN mediante la extensión NPS de Azure MFA, considere la posibilidad de federar la solución VPN como una [aplicación SAML](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-single-sign-on-non-gallery-applications) y determinar la categoría de la aplicación como se recomienda a continuación. 

Si ha implementado la extensión NPS de MFA de Azure AD para proteger con MFA los recursos locales, como la VPN y la puerta de enlace de Escritorio remoto, debe tener en cuenta de antemano si está listo para deshabilitar MFA en caso de emergencia.

En este caso, puede deshabilitar la extensión NPS; como resultado, el servidor NPS solo comprobará la autenticación principal y no aplicará la autenticación multifactor a los usuarios.

Deshabilitar la extensión NPS: 
-   Exporte la clave del registro HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\AuthSrv\Parameters como copia de seguridad. 
-   Elimine los valores del registro de "AuthorizationDLLs" y "ExtensionDLLs", no la clave Parameters. 
-   Reinicie el servicio de directivas de redes (IAS) para que los cambios surtan efecto. 
-   Determine si la autenticación principal para la VPN se realiza correctamente.

Una vez que se haya recuperado el servicio y esté listo para volver a aplicar MFA a los usuarios, habilite la extensión NPS: 
-   Importe la clave del registro HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\AuthSrv\Parameters desde la copia de seguridad. 
-   Reinicie el servicio de directivas de redes (IAS) para que los cambios surtan efecto. 
-   Determine si la autenticación principal y la autenticación secundaria para la VPN se han realizado correctamente.
-   Revise el servidor NPS y el registro de VPN para determinar qué usuarios han iniciado sesión durante la ventana de emergencia.

### <a name="deploy-password-hash-sync-even-if-you-are-federated-or-use-pass-through-authentication"></a>Implemente la sincronización de hash de contraseñas, incluso si usa federación o la autenticación de paso a través

El bloqueo de usuarios también puede producirse si se cumplen las siguientes condiciones:

- Su organización usa una solución de identidad híbrida con autenticación de paso a través o federación.
- Los sistemas de identidad local (por ejemplo, Active Directory, AD FS o un componente dependiente) no están disponibles. 
 
Para que sea más resistente, su organización debe [habilitar la sincronización de hash de contraseñas](https://docs.microsoft.com/azure/security/fundamentals/choose-ad-authn), ya que le permite [cambiar al uso de la sincronización de hash de contraseña](https://docs.microsoft.com/azure/active-directory/hybrid/plan-connect-user-signin) si sus sistemas de identidad locales están inactivos.

#### <a name="microsoft-recommendations"></a>Recomendaciones de Microsoft
 Habilite la sincronización de hash de contraseñas mediante el Asistente de Azure AD Connect, independientemente de si su organización usa la autenticación de paso a través o la federación.

>[!IMPORTANT]
> No es necesario convertir a los usuarios federados a la autenticación administrada para usar la sincronización de hash de contraseñas.

## <a name="during-a-disruption"></a>Durante una interrupción

Si ha optado por implementar un plan de mitigación, podrá sobrevivir a una interrupción del control de acceso único automáticamente. Pero si ha elegido crear un plan de contingencia, podrá activar las directivas de contingencia durante la interrupción del control de acceso:

1. Habilite las directivas de contingencia que conceden los usuarios de destino acceso a aplicaciones específicas desde redes específicas.
2. Deshabilite las directivas basadas en control normales.

### <a name="microsoft-recommendations"></a>Recomendaciones de Microsoft

En función de las mitigaciones o contingencias que se usen durante una interrupción, la organización podría conceder acceso con solo contraseñas. Carecer de métodos de protección es un riesgo de seguridad considerable que debe valorarse cuidadosamente. Las organizaciones deben:

1. Como parte de su estrategia de control de cambios, documentar todos los cambios y el estado anterior para poder revertir cualquier contingencia que haya implementado tan pronto como los controles de acceso estén completamente operativos.
2. Suponer que actores malintencionados intentarán obtener contraseñas a través de ataques de difusión de contraseña o de suplantación de identidad mientras la MFA esté deshabilitada. Además, es posible que los actores malintencionados ya tengan contraseñas que anteriormente no concediesen acceso a ningún recurso que pueden probar durante este período. Para los usuarios críticos como los ejecutivos, puede mitigar parcialmente este riesgo si restablece sus contraseñas antes de deshabilitar MFA para ellos.
3. Archivar todas las actividades de inicio de sesión para identificar quién accedió a qué durante el tiempo que se deshabilitó la MFA.
4. [Evalúe todas las detecciones de riesgos notificadas](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-sign-ins) durante este período.

## <a name="after-a-disruption"></a>Después de una interrupción

Deshaga los cambios realizados como parte del plan de contingencia activado una vez se restaure el servicio que provocó la interrupción. 

1. Habilitar las directivas normales
2. Vuelva a establecer las directivas de contingencia en el modo de solo informe. 
3. Revierta los cambios realizados y documentados durante la interrupción.
4. Si usa una cuenta de acceso de emergencia, no olvide volver a generar las credenciales y proteger físicamente la nueva información de credenciales como parte de los procedimientos de la cuenta de acceso de emergencia.
5. Siga [evaluando todas las detecciones de riesgos notificadas](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-sign-ins) después de la interrupción para detectar actividad sospechosa.
6. Revoque todos los tokens de actualización que se emitieron [mediante PowerShell](https://docs.microsoft.com/powershell/module/azuread/revoke-azureaduserallrefreshtoken?view=azureadps-2.0) que tenían como destino un conjunto de usuarios. Revocar todos los tokens de actualización es importante para las cuentas con privilegios que se usaron durante la interrupción y hacerlo les obligará a volver autenticarse y cumplir con el control de las directivas restauradas.

## <a name="emergency-options"></a>Opciones de emergencia

 En caso de que se produzca una emergencia y su organización no implementase anteriormente un plan de contingencia o mitigación, siga las recomendaciones de la sección [Contingencias para el bloqueo de usuario](#contingencies-for-user-lockout) si ya usan directivas acceso condicional para exigir MFA.
Si su organización usa directivas heredadas de MFA por usuario, puede considerar la siguiente alternativa:

1. Si tiene la dirección IP saliente de red corporativa, puede agregarlos como direcciones IP de confianza para habilitar la autenticación solo a la red corporativa.
   1. Si no tiene el inventario de direcciones IP de salida o tiene que habilitar el acceso dentro y fuera de la red corporativa, puede agregar todo el espacio de direcciones IPv4 como direcciones IP de confianza especificando 0.0.0.0/1 y 128.0.0.0/1.

>[!IMPORTANT]
 > Si amplía las direcciones IP de confianza para desbloquear el acceso, no se generarán las detecciones de riesgos asociados con las direcciones IP (por ejemplo, viajes imposibles o ubicaciones desconocidas).

>[!NOTE]
 > La configuración de [direcciones IP de confianza](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-mfasettings) solo está disponible para Azure MFA con [licencias de Azure AD Premium](https://docs.microsoft.com/azure/active-directory/authentication/concept-mfa-licensing).

## <a name="learn-more"></a>Más información

* [Configuración de Servidor Azure Multi-Factor Authentication para aplicaciones web de IIS](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfaserver-iis)
* [Administración de cuentas administrativas de acceso de emergencia en Azure AD](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-emergency-access)
* [Inicio rápido: Configuración de ubicaciones con nombre en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations)
  * [Set-MsolDomainFederationSettings](https://docs.microsoft.com/powershell/module/msonline/set-msoldomainfederationsettings?view=azureadps-1.0)
* [Planeamiento de la implementación de unión a Azure Active Directory híbrido](https://docs.microsoft.com/azure/active-directory/devices/hybrid-azuread-join-plan)
* [Guía de implementación de Windows Hello para empresas](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-deployment-guide)
  * [Password Guidance - Microsoft Research](https://research.microsoft.com/pubs/265143/microsoft_password_guidance.pdf) (Instrucciones de contraseñas: Microsoft Research)
* [¿Qué son las condiciones en el acceso condicional de Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/conditions)
* [¿Qué son los controles de acceso en el acceso condicional de Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/controls)
* [¿Qué es el modo de solo informe del acceso condicional?](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-report-only)
