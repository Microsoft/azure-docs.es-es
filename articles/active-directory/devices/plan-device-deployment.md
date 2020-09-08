---
title: Planeación de la implementación de dispositivos de Azure Active Directory
description: Elija las estrategias de integración de dispositivos Azure AD que satisfagan sus necesidades organizativas.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 06/15/2020
ms.author: baselden
author: BarbaraSelden
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3f17f6fd881243d91428c6025fd63469a38afa0f
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268815"
---
# <a name="plan-your-azure-active-directory-device-deployment"></a>Planeación de la implementación de dispositivos de Azure Active Directory

Este artículo le ayuda a evaluar los métodos para integrar el dispositivo con Azure AD y a elegir el plan de implementación, y proporciona vínculos clave a las herramientas de administración de dispositivos admitidas.

La variedad de dispositivos desde los cuales los usuarios inician sesión no deja de crecer. Las organizaciones pueden proporcionar equipos de escritorio, equipos portátiles, teléfonos, tabletas y otros dispositivos. Los usuarios pueden traer sus propios dispositivos y acceder a información desde distintas ubicaciones. En este entorno, su trabajo como administrador es proteger los recursos de la organización en todos los dispositivos.

Azure Active Directory (Azure AD) permite a las organizaciones cumplir estos objetivos mediante la administración de identidades de dispositivos. Ahora puede tener sus dispositivos en Azure AD y controlarlos desde un lugar central en [Azure Portal](https://portal.azure.com/). Esto le ofrece una experiencia unificada, mejora la seguridad y reduce el tiempo necesario para configurar un nuevo dispositivo.

Hay varios métodos para integrar los dispositivos en Azure AD:

* Puede [registrar los dispositivos](concept-azure-ad-register.md) manualmente con Azure AD.

* [Unir dispositivos](concept-azure-ad-join.md) a Azure AD (solo en la nube) o

* [Crear una unión a Azure AD híbrido](concept-azure-ad-join-hybrid.md) entre los dispositivos de su entorno local de Active Directory y Azure AD. 

## <a name="learn"></a>Obtener información

Antes de empezar, asegúrese de que está familiarizado con la [administración de identidades de dispositivos](overview.md).

### <a name="benefits"></a>Ventajas

Principales ventajas de ofrecer a los dispositivos una identidad de Azure AD:

* Mayor productividad: con Azure AD, los usuarios pueden usar el [inicio de sesión único (SSO) sin problemas](./azuread-join-sso.md) con los recursos locales y en la nube, lo que les permite ser productivos dondequiera que estén.

* Mayor seguridad: los dispositivos de Azure AD permiten aplicar [directivas de acceso condicional](../conditional-access/require-managed-devices.md) a los recursos en función de la identidad del dispositivo o el usuario. Las directivas de acceso condicional pueden ofrecer protección adicional mediante [Azure AD Identity Protection](../identity-protection/overview-identity-protection.md). Unir un dispositivo a Azure AD es un requisito previo para aumentar la seguridad con una estrategia de [autenticación sin contraseñas](../authentication/concept-authentication-passwordless.md).

* Mejor experiencia del usuario: tener las identidades de dispositivo en Azure AD permite ofrecer a los usuarios un acceso sencillo a los recursos basados en la nube de su organización desde sus dispositivos personales y corporativos. Los administradores pueden habilitar [Enterprise State Roaming](enterprise-state-roaming-overview.md) para obtener una experiencia unificada en todos los dispositivos Windows.

* Implementación y administración simplificadas: la administración de identidades de dispositivos simplifica el proceso de incorporación de dispositivos a Azure AD mediante [Windows AutoPilot](/windows/deployment/windows-autopilot/windows-10-autopilot), [aprovisionamiento masivo](/mem/intune/enrollment/windows-bulk-enroll) y [autoservicio: configuración rápida (OOBE)](../user-help/user-help-join-device-on-network.md). Puede administrar los dispositivos con herramientas de administración de dispositivos móviles (MDM) como [Microsoft Intune](/mem/intune/fundamentals/what-is-intune), y sus identidades en [Azure Portal](https://portal.azure.com/).

### <a name="training-resources"></a>Recursos de aprendizaje

Vídeo:  [Acceso condicional con controles de dispositivos](https://youtu.be/NcONUf-jeS4)

Preguntas más frecuentes: [Preguntas más frecuentes sobre la administración de dispositivos de Azure Active Directory](faq.md) y [Preguntas más frecuentes sobre itinerancia de datos y configuración](enterprise-state-roaming-faqs.md) 

## <a name="plan-the-deployment-project"></a>Planeamiento del proyecto de implementación

Tenga en cuenta las necesidades de su organización al determinar la estrategia de esta implementación en su entorno.

### <a name="engage-the-right-stakeholders"></a>Interactuar con las partes interesadas adecuadas

Cuando fracasan los proyectos tecnológicos, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de involucrar a las partes interesadas adecuadas](https://aka.ms/deploymentplans) y de que estas conozcan bien sus roles. 

Para este plan, agregue las siguientes partes interesadas a la lista:

| Role| Descripción |
| - | - |
| Administrador de dispositivos| Un representante del equipo de dispositivos que pueda comprobar que el plan cumplirá los requisitos de dispositivos de la organización. |
| Administrador de red| Un representante del equipo de red que pueda asegurarse de que se cumplen los requisitos de red. |
| Equipo de administración de dispositivos| Equipo que administra el inventario de dispositivos. |
| Equipos de administración específicos del sistema operativo| Equipos que administran versiones específicas del sistema operativo y les dan soporte técnico. Por ejemplo, puede haber un equipo para Mac o iOS. |

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para el éxito de cualquier servicio nuevo. Comunique de forma proactiva a los usuarios cómo y cuándo va a cambiar su experiencia, y cómo obtener soporte técnico si tienen cualquier problema.

### <a name="plan-a-pilot"></a>Planeamiento de un piloto

Se recomienda que la configuración inicial del método de integración esté en un entorno de pruebas, o se haga con un pequeño grupo de dispositivos de prueba. Consulte [Procedimientos recomendados para un piloto](../fundamentals/active-directory-deployment-plans.md).

La implementación de una unión a Azure AD híbrido es sencilla y la realiza totalmente el administrador sin necesidad de que intervenga el usuario final. Obtenga información sobre cómo realizar una [validación controlada de la unión a Azure AD híbrido](hybrid-azuread-join-control.md) antes de habilitarla en toda la organización.

## <a name="choose-your-integration-methods"></a>Elección de los métodos de integración

Su organización puede usar varios métodos de integración de dispositivos en un único inquilino de Azure AD. El objetivo es elegir los métodos adecuados para que los dispositivos se administren de forma segura en Azure AD. Hay muchos parámetros que influyen en esta decisión: la propiedad, los tipos de dispositivos, la audiencia principal y la infraestructura de su organización.

La siguiente información puede ayudarle a decidir qué métodos de integración deberá usar.

### <a name="decision-tree-for-devices-integration"></a>Árbol de decisión para la integración de dispositivos

Use este árbol para determinar las opciones para los dispositivos que son propiedad de la organización. 

> [!NOTE]
> Los escenarios de dispositivos personales o "traiga su propio dispositivo" (BYOD) no se muestran en este diagrama. Siempre dan como resultado un registro de Azure AD.

 ![Árbol de decisión](./media/plan-device-deployment/flowchart.png)

### <a name="comparison-matrix"></a>Tabla comparativa

Los dispositivos iOS y Android solo se pueden registrar en Azure AD. En la tabla siguiente se presentan consideraciones generales para dispositivos cliente Windows. Úsela para obtener información general y, después, explore los distintos métodos de integración con más detalle.

| Consideración | Registrado en Azure AD| Unión a Azure AD| Unión a Azure AD híbrido |
| - | - | - | - |
| **Sistemas operativos cliente**| | |  |
| Dispositivos Windows 10| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
| Dispositivos Windows de nivel inferior (Windows 8.1 o Windows 7)| | | ![check](./media/plan-device-deployment/check.png) |
|**Opciones de inicio de sesión**| | |  |
| Credenciales locales del usuario final| ![check](./media/plan-device-deployment/check.png)| |  |
| Contraseña| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
| PIN del dispositivo| ![check](./media/plan-device-deployment/check.png)| |  |
| Windows Hello| ![check](./media/plan-device-deployment/check.png)| |  |
| Windows Hello para empresas| | ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
| Claves de seguridad de FIDO 2.0| | ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
| Aplicación Microsoft Authenticator (sin contraseña)| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
|**Principales funcionalidades**| | |  |
| SSO a los recursos en la nube| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
| Inicio de sesión único en recursos locales| | ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
| Acceso condicional <br> (Requiere que los dispositivos estén marcados como conformes) <br> (Debe administrarse con MDM)| ![check](./media/plan-device-deployment/check.png) | ![check](./media/plan-device-deployment/check.png)|![check](./media/plan-device-deployment/check.png) |
Acceso condicional <br>(Requiere dispositivos con unión a Azure AD híbrido)| | | ![check](./media/plan-device-deployment/check.png)
| Autoservicio de restablecimiento de contraseña desde la pantalla de inicio de sesión de Windows| | ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
| Restablecimiento del PIN de Windows Hello| | ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |
| Enterprise State Roaming entre dispositivos| | ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png) |


## <a name="azure-ad-registration"></a>Registro en Azure AD 

Los dispositivos registrados se suelen administrar con [Microsoft Intune](/mem/intune/enrollment/device-enrollment). Los dispositivos se inscriben en Intune de varias maneras, según el sistema operativo. 

Los dispositivos registrados en Azure AD permiten que los dispositivos BYOD y los dispositivos propiedad de la empresa usen el inicio de sesión único en los recursos en la nube. El acceso a los recursos se basa en las [directivas de acceso condicional](../conditional-access/require-managed-devices.md) de Azure AD que se aplican al dispositivo y al usuario.

### <a name="registering-devices"></a>Registro de dispositivos

Los dispositivos registrados se suelen administrar con [Microsoft Intune](/mem/intune/enrollment/device-enrollment). Los dispositivos se inscriben en Intune de varias maneras, según el sistema operativo. 

Los usuarios que instalan la aplicación Portal de empresa registran los dispositivos móviles BYOD y propiedad de la empresa.

* [iOS](/mem/intune/user-help/install-and-sign-in-to-the-intune-company-portal-app-ios)

* [Android](/mem/intune/user-help/enroll-device-android-company-portal)

* [Windows 10](/mem/intune/user-help/enroll-windows-10-device)

Si el registro de sus dispositivos es la mejor opción para su organización, consulte los siguientes recursos:

* Esta introducción a los [dispositivos registrados de Azure AD](concept-azure-ad-register.md).

* Esta documentación para el usuario final sobre cómo [registrar un dispositivo personal en la red de la organización](../user-help/user-help-register-device-on-network.md).

## <a name="azure-ad-join"></a>Unión a Azure AD

La unión a Azure AD permite realizar una transición hacia un modelo de "primero en la nube" con Windows. Es una base estupenda si tiene previsto modernizar la administración de dispositivos y reducir los costos de TI relacionados con los dispositivos. La unión a Azure AD funciona solo con dispositivos Windows 10. Considérelo como la primera opción para los nuevos dispositivos.

Sin embargo, [los dispositivos unidos a Azure AD pueden usar el inicio de sesión único en recursos locales](azuread-join-sso.md) cuando se encuentran en la red de la organización y pueden autenticarse en servidores locales como archivos, impresiones y aplicaciones.

Si esta es la mejor opción para su organización, consulte los siguientes recursos:

* Esta introducción a los [dispositivos unidos a Azure AD](concept-azure-ad-join.md).

* Familiarícese con el [plan de implementación de Unión a Azure AD](azureadjoin-plan.md).

### <a name="provisioning-azure-ad-join-to-your-devices"></a>Aprovisionamiento de la unión a Azure AD en los dispositivos

Para aprovisionar la unión a Azure AD, hay varios enfoques:

* Autoservicio: [Primera ejecución de Windows 10](azuread-joined-devices-frx.md)

Si tiene Windows 10 Professional o Windows 10 Enterprise instalado en un dispositivo, la experiencia predeterminada es el proceso de configuración de los dispositivos de la empresa.

* [Configuración rápida (OOBE) de Windows o desde la Configuración de Windows](../user-help/user-help-join-device-on-network.md)

* [Windows Autopilot](/windows/deployment/windows-autopilot/windows-autopilot)

* [Inscripción masiva](/mem/intune/enrollment/windows-bulk-enroll)

Elija el procedimiento de implementación después [comparar cuidadosamente estos enfoques](azureadjoin-plan.md).

Puede determinar que la unión a Azure AD es la mejor solución para un dispositivo y que el dispositivo ya está en un estado diferente. Estas son algunas consideraciones sobre la actualización.

| Estado actual del dispositivo| Estado deseado del dispositivo| Instrucciones |
| - | - | - |
| Unido a dominio local| Azure AD Join| Separe el dispositivo del dominio local antes de unirlo a Azure AD |
| Unión a Azure AD híbrido| Azure AD Join| Separe el dispositivo del dominio local y de Azure AD antes de unirlo a Azure AD |
| Registrado en Azure AD| Azure AD Join| Anule el registro del dispositivo antes de unirlo a Azure AD |


## <a name="hybrid-azure-ad-join"></a>Unión a Azure AD híbrido

Si tiene un entorno local de Active Directory y quiere unir a Azure AD sus equipos unidos a un dominio de Active Directory, puede hacerlo con una unión a Azure AD híbrido. Admite una [amplia variedad de dispositivos Windows](hybrid-azuread-join-plan.md), incluidos los dispositivos Windows actuales y más antiguos.

La mayoría de las organizaciones ya tienen dispositivos unidos a un dominio y los administran mediante directivas de grupo o System Center Configuration Manager (SCCM). En ese caso, se recomienda configurar la unión a Azure AD híbrido para empezar a obtener ventajas y aprovechar las inversiones existentes.

Si la unión a Azure AD híbrido es la mejor opción para su organización, consulte los siguientes recursos:

* Esta introducción a los [dispositivos unidos a Azure AD híbrido](concept-azure-ad-join-hybrid.md).

* Familiarícese con el [plan de implementación de Unión a Azure AD híbrido](hybrid-azuread-join-plan.md).

### <a name="provisioning-hybrid-azure-ad-join-to-your-devices"></a>Aprovisionamiento de la unión a Azure AD híbrido en los dispositivos

[Revise su infraestructura de identidad](hybrid-azuread-join-plan.md). Azure AD Connect proporciona un asistente para configurar la unión a Azure AD híbrido para:

* [Dominios federados](hybrid-azuread-join-federated-domains.md)

* [Dominios administrados](hybrid-azuread-join-managed-domains.md)

Si no le es posible instalar la versión requerida de Azure AD Connect, consulte [cómo configurar manualmente la unión a Azure AD híbrido](hybrid-azuread-join-manual.md). 

> [!NOTE] 
> El dispositivo Windows 10 unido a un dominio local intenta unirse automáticamente a Azure AD para convertirse en unión a Azure AD híbrido de forma predeterminada. Esto solo se hará correctamente si tiene configurado el entorno adecuado. 

Puede determinar que la unión a Azure AD híbrido es la mejor solución para un dispositivo y que el dispositivo ya está en un estado diferente. Estas son algunas consideraciones sobre la actualización.

| Estado actual del dispositivo| Estado deseado del dispositivo| Instrucciones |
| - | - | - |
| Unión a un dominio local| Unión a Azure AD híbrido| Use Azure AD Connect o AD FS para unirse a Azure |
| Unido a un grupo de trabajo local o nuevo| Unión a Azure AD híbrido| Admitido con [Windows Autopilot](/windows/deployment/windows-autopilot/windows-autopilot) De lo contrario, el dispositivo debe estar unido a un dominio local antes de la unión a Azure AD híbrido |
| Unido a Azure AD| Unión a Azure AD híbrido| Separar de Azure AD; esto lo coloca en el grupo de trabajo local o en estado Nuevo. |
| Registrado en Azure AD| Unión a Azure AD híbrido| Depende de la versión de Windows. [Vea estas consideraciones](hybrid-azuread-join-plan.md). |

## <a name="manage-your-devices"></a>Administración de los dispositivos

Una vez que haya registrado o unido sus dispositivos a Azure AD, use [Azure Portal](https://portal.azure.com/) como ubicación central para administrar las identidades de los dispositivos. La página de dispositivos de Azure Active Directory le permite:

* [Configuración de las opciones del dispositivo](device-management-azure-portal.md#configure-device-settings)
* Debe ser administrador local para administrar dispositivos Windows. [Azure AD actualiza esta pertenencia para los dispositivos unidos a Azure AD](assign-local-admin.md) y agrega automáticamente a los usuarios que tengan el rol de administrador de dispositivos como administradores a todos los dispositivos unidos.

Para mantener el entorno limpio, no olvide [administrar los dispositivos obsoletos](manage-stale-devices.md) y centre sus recursos en la administración de los dispositivos actuales.

* [Revisión de los registros de auditoría relacionados con los dispositivos](device-management-azure-portal.md#audit-logs)

### <a name="supported-device-management-tools"></a>Herramientas de administración de dispositivos admitidas

Los administradores pueden proteger y controlar aún más estos dispositivos registrados y unidos con otras herramientas de administración de dispositivos. Estas herramientas proporcionan una manera de aplicar las configuraciones que requiere la organización, como el cifrado del almacenamiento, la complejidad de las contraseñas y las instalaciones y actualizaciones de software. 

Revise las plataformas admitidas y no admitidas para los dispositivos integrados:

| Herramientas de administración de dispositivos| Registrado en Azure AD| Unión a Azure AD| Unión a Azure AD híbrido|
| - | - | - | - |
| [Administración de dispositivos móviles (MDM)](/windows/client-management/mdm/azure-active-directory-integration-with-mdm) <br>Ejemplo: Microsoft Intune| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png)|  |
| [Administración conjunta con Microsoft Intune y Microsoft Endpoint Configuration Manager](/mem/configmgr/comanage/overview) <br>(Windows 10 y versiones posteriores)| | ![check](./media/plan-device-deployment/check.png)| ![check](./media/plan-device-deployment/check.png)|  |
| [Directiva de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831791(v=ws.11))<br>(Solo Windows)| | | ![check](./media/plan-device-deployment/check.png)|  |



 Para los dispositivos iOS o Android registrados, es recomendable usar la [administración de aplicaciones móviles (MAM) de Microsoft Intune](/mem/intune/apps/app-management) con o sin administración de dispositivos.

 Los administradores también pueden [implementar plataformas de infraestructura de escritorio virtual (VDI)](howto-device-identity-virtual-desktop-infrastructure.md) para hospedar los sistemas operativos Windows en sus organizaciones con el fin de centralizar y consolidar los recursos y así simplificar la administración y reducir los costos. 

### <a name="troubleshoot-device-identities"></a>Solución de problemas de identidades de dispositivo

* [Solución de problemas de dispositivos con el comando dsregcmd](troubleshoot-device-dsregcmd.md)

* [Solución de problemas de la configuración de Enterprise State Roaming en Azure Active Directory](enterprise-state-roaming-troubleshooting.md)

Si tiene problemas para completar la unión a Azure AD híbrido para los dispositivos de Windows unidos a un dominio, consulte:

* [Solución de problemas de la unión a Azure AD híbrido para dispositivos actuales de Windows](troubleshoot-hybrid-join-windows-current.md)

* [Solución de problemas de la unión a Azure AD híbrido para dispositivos Windows de nivel inferior](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Pasos siguientes

* [Planeación de la implementación de la unión a Azure AD](azureadjoin-plan.md)
* [Planeación de la implementación de la unión a Azure AD híbrido](hybrid-azuread-join-plan.md)
* [Administración de identidades de dispositivo](device-management-azure-portal.md)