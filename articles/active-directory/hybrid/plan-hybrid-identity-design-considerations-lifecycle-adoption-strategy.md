---
title: 'Diseño de identidades híbridas: estrategia de adopción de ciclo de vida en Azure | Microsoft Docs'
description: Ayuda a definir las tareas de administración de identidades híbridas según las opciones disponibles para cada fase del ciclo de vida.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: 420b6046-bd9b-4fce-83b0-72625878ae71
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/30/2018
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: a6d2638ff86d110cb79cc72519dfabb2c68b3794
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89278435"
---
# <a name="determine-hybrid-identity-lifecycle-adoption-strategy"></a>Determinación de la estrategia de adopción de ciclo de vida de identidad híbrida
En esta tarea, va a definir la estrategia de administración de identidades para que su solución de identidad híbrida cumpla los requisitos empresariales que definió en [Determinación de las tareas de administración de identidad híbrida](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md).

Para definir las tareas de administración de identidades híbridas según el ciclo de vida de identidad completo presentado antes en este paso, tendrá que considerar las opciones disponibles para cada fase del ciclo de vida.

## <a name="access-management-and-provisioning"></a>Aprovisionamiento y administración del acceso
Con una buena solución de administración de acceso de cuentas, su organización puede controlar con precisión quién tiene acceso a qué información en toda la organización.

El control de acceso es una función crucial en un sistema centralizado de aprovisionamiento de único punto. Además de proteger la información confidencial, los controles de acceso exponen las cuentas existentes que tengan autorizaciones no aprobadas o que ya no sean necesarias. Para controlar las cuentas obsoletas, el sistema de aprovisionamiento vincula la información de las cuentas con información autorizada sobre los usuarios que poseen las cuentas. La información de identidad autorizada sobre usuarios normalmente se mantiene en las bases de datos y los directorios de recursos humanos.

Las cuentas en empresas de TI sofisticadas incluyen cientos de parámetros que definen las autoridades y estos detalles se pueden controlar mediante el sistema de aprovisionamiento. Los nuevos usuarios pueden identificarse con los datos que proporcione desde el origen de autoridad. La capacidad de aprobación de solicitudes de acceso inicia los procesos que aprueban (o rechazan) el aprovisionamiento de recursos para ellos.

| Fase de administración del ciclo de vida | En el entorno local | Nube | Híbrido |
| --- | --- | --- | --- |
| Aprovisionamiento y administración de cuentas |Mediante el rol del servidor Servicios de dominio de Active Directory® (AD DS), puede crear una infraestructura escalable, segura y administrable para la administración de recursos y usuarios, así como proporcionar compatibilidad con aplicaciones habilitadas para directorio, como Microsoft® Exchange Server. <br><br> [Puede aprovisionar grupos en AD DS a través de un administrador de identidad.](/previous-versions/mim/ff686261(v=ws.10)) <br>[Puede aprovisionar usuarios en AD DS](/previous-versions/mim/ff686263(v=ws.10)). <br><br> Los administradores pueden usar el control de acceso para administrar el acceso de usuarios a los recursos compartidos por seguridad. En Active Directory, el control de acceso se administra en el nivel de objeto; para ello, se establecen distintos niveles de acceso, o permisos, a los objetos, como Control total, Escritura, Lectura o Sin acceso. El control de acceso en Active Directory define cómo los distintos usuarios pueden usar objetos de Active Directory. De forma predeterminada, los permisos de los objetos de Active Directory se establecen en la configuración más segura. |Hay que crear una cuenta para cada usuario que vaya a tener acceso a un servicio de nube de Microsoft. También puede cambiar las cuentas de usuario o eliminarlas cuando ya no sean necesarias. De forma predeterminada, los usuarios no tienen permisos de administrador, pero puede asignárselos si lo desea. <br><br> Dentro de Azure Active Directory, una de las principales características es la capacidad para administrar el acceso a los recursos. Estos recursos pueden formar parte del directorio, como en el caso de los permisos para administrar objetos a través de roles en el directorio o los recursos externos al directorio, como las aplicaciones SaaS, los servicios de Azure y los sitios de SharePoint o los recursos publicados en modo local. <br><br> En el centro de la solución de administración de acceso de Azure Active Directory se encuentra el grupo de seguridad. El propietario de los recursos (o el administrador del directorio) puede asignar un grupo para proporcionar determinados derechos de acceso a los recursos que posee. Los miembros del grupo recibirán el derecho de acceso y el propietario del recurso puede delegar el derecho de administración de la lista de miembros de un grupo en otra persona como, por ejemplo, un administrador de departamento o un administrador de soporte técnico.<br> <br> La sección Administración de grupos en Azure AD proporciona más información sobre la administración del acceso mediante grupos. |Amplíe las identidades de Active Directory a la nube por medio de la sincronización y la federación. |

## <a name="role-based-access-control"></a>Control de acceso basado en rol
El control de acceso basado en rol (RBAC) de Azure usa roles y directivas de aprovisionamiento para evaluar, probar y aplicar las reglas y los procesos de negocio para conceder acceso a los usuarios. Los administradores clave crean directivas de aprovisionamiento, asignan usuarios a roles y definen conjuntos de derechos a recursos para estos roles. RBAC de Azure amplía la solución de administración de identidades para usar los procesos basados en software y reducir la interacción manual del usuario en el proceso de aprovisionamiento.
RBAC de Azure habilita que la empresa restrinja la cantidad de operaciones que una persona puede hacer una vez que acceden a Azure Portal. Si se usa RBAC de Azure para controlar el acceso al portal, los administradores de TI pueden delegar el acceso con los siguientes enfoques de administración de acceso:

* **Asignación de roles basada en grupo**: puede asignar acceso a los grupos de Azure AD que se pueden sincronizar desde su Active Directory local. Esto le permite aprovechar las inversiones que su organización realizó en herramientas y procesos para administrar grupos. También puede usar la característica de administración de grupos delegados de Azure AD Premium.
* **Aprovechar los roles integrados de Azure**: puede usar tres roles (Propietario, Colaborador y Lector) para asegurarse de que los usuarios y los grupos tienen permiso para realizar únicamente las tareas necesarias para sus respectivos trabajos.
* **Acceso granular a los recursos**: puede asignar roles a los usuarios y grupos para una suscripción concreta, un grupo de recursos o un recurso individual de Azure, como un sitio web o una base de datos. De esta forma, puede asegurarse de que los usuarios tengan acceso a todos los recursos que necesitan y ningún acceso a los que no necesitan administrar.

## <a name="provisioning-and-other-customization-options"></a>Aprovisionamiento y otras opciones de personalización
El equipo puede usar planes y requisitos empresariales para decidir hasta qué punto personalizar la solución de identidad. Por ejemplo, una gran empresa podría requerir un plan de implementación por fases para flujos de trabajo y adaptadores personalizados que se base en una escala de tiempo para aprovisionar de forma incremental aplicaciones que se usan ampliamente en diversas zonas del mundo. Otro plan de personalización podría contemplar el aprovisionamiento de dos o más aplicaciones en toda una organización, después de completarse las pruebas. Se puede personalizar la interacción entre la aplicación y los usuarios, y se pueden cambiar los procedimientos de aprovisionamiento de recursos para dar cabida al aprovisionamiento automatizado.

Puede desaprovisionar para quitar un servicio o componente. Por ejemplo, si se desaprovisiona una cuenta, la cuenta se elimina de un recurso.

El modelo híbrido de aprovisionamiento de recursos combina el enfoque basado en solicitudes con el basado en roles, ambos compatibles con Azure AD. Para un subconjunto de empleados o sistemas administrados, es posible que una empresa quiera automatizar el acceso con la asignación basada en roles. Un negocio también podría controlar todas las demás solicitudes de acceso o excepciones por medio de un modelo basado en solicitudes. Puede que algunas empresas comiencen con la asignación manual y evolucionen hacia un modelo híbrido, con la intención de llegar a una implementación totalmente basada en roles en el futuro.

En otros casos, puede que el aprovisionamiento enteramente basado en roles no resulte práctico por motivos empresariales y que la meta deseada sea un enfoque híbrido. También es posible que otros negocios estén satisfechos únicamente con el aprovisionamiento basado en solicitudes y no deseen invertir más esfuerzo en definir y administrar directivas de aprovisionamiento automatizadas basadas en roles.

## <a name="license-management"></a>Administración de licencias
La administración de licencias basada en grupos en Azure AD permite a los administradores asignar usuarios a un grupo de seguridad y Azure AD asigna automáticamente licencias a todos los miembros del grupo. Si después se agrega un usuario al grupo o se le quita de él, se asignará o eliminará una licencia automáticamente según corresponda.

Puede usar los grupos que sincronice desde un AD local o que administre en Azure AD. Si combina esto con la administración de grupos de autoservicio de Azure AD Premium, puede delegar fácilmente la asignación de licencias en los responsables de la toma de decisiones adecuados. Puede estar seguro de que los problemas como los conflictos de licencias y la falta de datos de ubicación se solucionarán automáticamente.

## <a name="self-regulating-user-administration"></a>Administración de usuarios autorregulada
Cuando su organización empieza a aprovisionar recursos en todas las organizaciones internas, se implementa la capacidad de administración de usuarios autorregulada. Puede obtener las ventajas de aprovisionar usuarios a través de los límites de la organización. En este entorno, un cambio en el estado de un usuario se refleja automáticamente en los derechos de acceso en las diversas ubicaciones geográficas y a través de los límites de la organización. Puede reducir los costos de aprovisionamiento y optimizar los procesos de aprobación y acceso. La implementación logra todo el potencial de implementar el control de acceso basada en roles para la administración integral del acceso en su organización. Puede reducir los costos administrativos mediante procedimientos automatizados para regular el aprovisionamiento de usuarios. Puede mejorar la seguridad mediante la automatización de la aplicación de directivas de seguridad, así como optimizar y centralizar la administración del ciclo de vida de usuario y el aprovisionamiento de recursos para grandes grupos de usuarios.

> [!NOTE]
> Para obtener más información, consulte Configuración de Azure AD para la administración del acceso a la aplicación de autoservicio.
> 
> 

Los servicios de Azure AD basados en licencias (basados en derechos) funcionan mediante la activación de una suscripción en el inquilino del servicio/directorio de Azure AD. Una vez que la suscripción está activa, los administradores de servicios/directorios pueden gestionar las capacidades del servicio y los usuarios con licencia pueden usarlas. 

## <a name="integration-with-other-3rd-party-providers"></a>Integración con otros proveedores

Azure Active Directory ofrece inicio de sesión único y seguridad mejorada de acceso a aplicaciones para miles de aplicaciones SaaS y aplicaciones web locales. Para más información, consulte [Integración de aplicaciones con Azure Active Directory](../develop/quickstart-register-app.md).

## <a name="define-synchronization-management"></a>Definición de la administración de sincronización
la integración de directorios locales con Azure AD hace que los usuarios sean más productivos proporcionando una identidad común para tener acceso a recursos de nube y locales. Con esta integración, los usuarios y las organizaciones pueden aprovechar lo siguiente:

* Las organizaciones pueden proporcionar a los usuarios una identidad híbrida común entre servicios en la nube o locales al aprovechar Windows Server Active Directory y, a continuación, conectarse a Active Directory de Azure.
* Los administradores pueden proporcionar acceso condicional basado en el recurso de la aplicación, dispositivo e identidad de usuario, ubicación de red y autenticación multifactor.
* Los usuarios pueden aprovechar su identidad común mediante cuentas de Azure AD a Office 365, Intune, aplicaciones SaaS y aplicaciones de otros fabricantes.
* Los desarrolladores pueden crear aplicaciones que aprovechan el modelo de identidad común, la integración de aplicaciones en Active Directory local o en Azure para aplicaciones basadas en la nube

En la ilustración siguiente, se muestra un ejemplo de una visión general del proceso de sincronización de identidades.

![Sync](./media/plan-hybrid-identity-design-considerations/identitysync.png)

Proceso de sincronización de identidades

Revise la siguiente tabla para comparar las opciones de sincronización:

| Opción de administración de sincronización | Ventajas | Inconvenientes |
| --- | --- | --- |
| Basada en la sincronización (a través de DirSync o AADConnect) |Usuarios y grupos se sincronizan desde una ubicación local y la nube. <br>  **Control de directivas**: Se pueden establecer directivas de cuenta a través de Active Directory, lo que proporciona al administrador la capacidad de administrar directivas de contraseñas, estaciones de trabajo, restricciones, controles de bloqueo, etc., sin tener que hacer más tareas en la nube.  <br>  **Control de acceso**: Puede restringir el acceso al servicio en la nube para que se pueda acceder a los servicios a través del entorno corporativo, a través de servidores en línea o ambos. <br>  Menos llamadas al soporte técnico: Si los usuarios tienen que recordar menos contraseñas, es menos probable que las olviden. <br>  Seguridad: La información y las identidades de usuarios están protegidas porque todos los servidores y servicios usados en el inicio de sesión único se controlan de forma local. <br>  Compatibilidad con la autenticación sólida: Puede usar la autenticación sólida (también denominada autenticación en dos fases) con el servicio en la nube. Sin embargo, si usa la autenticación sólida, debe emplear el inicio de sesión único. | |
| Basada en la federación (a través de AD FS) |Habilitada por el servicio de token de seguridad (STS). Al configurar un STS para proporcionar acceso de inicio de sesión único con un servicio en la nube de Microsoft, va a crear una confianza federada entre su STS local y el dominio federado que especificó en el inquilino de Azure AD. <br> Permite a los usuarios finales utilizar el mismo conjunto de credenciales para obtener acceso a varios recursos <br>los usuarios finales no tienen que mantener varios conjuntos de credenciales. Sin embargo, los usuarios deben proporcionar sus credenciales para cada uno de los recursos participantes. Se admiten escenarios de B2B y B2C. |Requiere personal especializado para la implementación y el mantenimiento de servidores AD FS locales dedicados. Existen restricciones para el uso de la autenticación sólida, si va a usar AD FS para su STS. Para obtener más información, consulte la página sobre la [configuración de opciones avanzadas de AD FS 2.0](https://go.microsoft.com/fwlink/?linkid=235649). |

> [!NOTE]
> Para obtener más información, consulte [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).
> 
> 

## <a name="see-also"></a>Consulte también
[Información general sobre las consideraciones de diseño](plan-hybrid-identity-design-considerations-overview.md)