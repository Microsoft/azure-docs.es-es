---
title: Guía de referencia de operaciones de administración de identidad y acceso de Azure Active Directory
description: Esta guía de referencia de operaciones describe las comprobaciones y las acciones que debe realizar para proteger las operaciones de administración de identidad y acceso.
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: fundamentals
ms.date: 10/31/2019
ms.author: martinco
ms.openlocfilehash: ab4e9f7410954292290b6acf0895197ff013b1d8
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321674"
---
# <a name="azure-active-directory-identity-and-access-management-operations-reference-guide"></a>Guía de referencia de operaciones de administración de identidad y acceso de Azure Active Directory

En esta sección de la [guía de referencia de operaciones de Azure AD](active-directory-ops-guide-intro.md) se describen las comprobaciones y las acciones que debe tener en cuenta para proteger y administrar el ciclo de vida de las identidades y sus asignaciones.

> [!NOTE]
> Estas recomendaciones están actualizadas hasta la fecha de publicación, pero pueden cambiar con el tiempo. Las organizaciones deben evaluar continuamente sus prácticas de identidad a medida que los productos y servicios de Microsoft evolucionen.

## <a name="key-operational-processes"></a>Procesos operativos clave

### <a name="assign-owners-to-key-tasks"></a>Asignación de propietarios a las tareas clave

La administración de Azure Active Directory requiere la ejecución continua de tareas y procesos operativos clave que pueden no formar parte de un proyecto de implementación. Aun así, es importante que configure estas tareas con miras al mantenimiento de su entorno. Entre las tareas clave y sus propietarios recomendados se incluyen:

| Tarea | Propietario |
| :- | :- |
| Definir el proceso de creación de suscripciones de Azure | Varía según la organización |
| Decidir quién obtiene licencias de Enterprise Mobility + Security | Equipo de operaciones IAM |
| Decidir quién obtiene licencias de Office 365 | Equipo de productividad |
| Decidir quién obtiene otras licencias, por ejemplo, Dynamics, VSO | Application Owner |
| Asignación de licencias | Equipo de operaciones IAM |
| Solución de problemas y corrección de errores de asignación de licencias | Equipo de operaciones IAM |
| Aprovisionar identidades para aplicaciones en Azure AD | Equipo de operaciones IAM |

A medida que revise la lista, es posible que tenga que asignar un propietario a las tareas que no tienen uno o ajustar la propiedad de aquellas tareas cuyos propietarios no se ajustan a las recomendaciones anteriores.

#### <a name="assigning-owners-recommended-reading"></a>Lectura recomendada sobre la asignación de propietarios

- [Asignación de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md)
- [Gobernanza en Azure](../../governance/index.yml)

## <a name="on-premises-identity-synchronization"></a>Sincronización de identidades local

### <a name="identify-and-resolve-synchronization-issues"></a>Identificación y solución de problemas de sincronización

Microsoft recomienda tener una buena línea de base y comprensión de los problemas de su entorno local que pueden derivar en problemas de sincronización en la nube. Dado que las herramientas automatizadas como [IdFix](/office365/enterprise/prepare-directory-attributes-for-synch-with-idfix) y [Azure AD Connect Health](../hybrid/whatis-azure-ad-connect.md#why-use-azure-ad-connect-health) pueden generar un gran volumen de falsos positivos, se recomienda que identifique los errores de sincronización que se han dejado sin solucionar durante más de 100 días limpiando esos objetos con error. Los errores de sincronización sin resolver a largo plazo pueden generar incidentes de soporte técnico. En [Solución de errores durante la sincronización](../hybrid/tshoot-connect-sync-errors.md) se proporciona información general sobre los distintos tipos de errores de sincronización, algunos de los posibles escenarios que provocan dichos errores y las posibles maneras de corregirlos.

### <a name="azure-ad-connect-sync-configuration"></a>Configuración de la sincronización de Azure AD Connect

Para habilitar todas las experiencias híbridas, la postura de seguridad basada en dispositivos y la integración con Azure AD, es necesario sincronizar las cuentas de usuario que los empleados usan para iniciar sesión en sus equipos de escritorio.

Si no sincroniza los bosques en que los usuarios inician sesión, debe cambiar la sincronización para que provenga del bosque adecuado.

#### <a name="synchronization-scope-and-object-filtering"></a>Ámbito de sincronización y filtrado de objetos

La supresión de cubos de objetos conocidos que no requieren sincronización tiene los siguientes beneficios operativos:

- Menos orígenes de errores de sincronización
- Ciclos de sincronización más rápidos
- Habrá menos elementos inútiles para transferir desde el entorno local; por ejemplo, la contaminación de la lista global de direcciones de las cuentas de servicios locales que no son pertinentes en la nube

> [!NOTE]
> Si se da cuenta de que está importando muchos objetos que no se están exportando a la nube, debe filtrar por unidad organizativa o atributos específicos.

Algunos ejemplos de objetos que se deben excluir son:

- Cuentas de servicio que no se usan con aplicaciones en la nube.
- Grupos que no están diseñados para usarse en escenarios en la nube, como los que se usan para conceder acceso a los recursos.
- Usuarios o contactos que son identidades externas pensados para representarse con Colaboración B2B de Azure AD.
- Cuentas de equipo en las que no está previsto que los emplados accedan a aplicaciones en la nube desde, por ejemplo, servidores.

> [!NOTE]
> Si una única identidad humana tiene varias cuentas aprovisionadas a partir de un suceso como una migración de dominio heredada, una fusión o una adquisición, solo debe sincronizar la cuenta utilizada por el usuario de forma cotidiana, por ejemplo, la que usa para iniciar sesión en su equipo.

Idealmente, querrá alcanzar un equilibrio entre la reducción del número de objetos que se van a sincronizar y la complejidad de las reglas. Por lo general, una combinación entre el [filtrado](../hybrid/how-to-connect-sync-configure-filtering.md) de unidad organizativa/contenedor más una asignación de atributo simple para el atributo cloudFiltered resulta eficaz.

> [!IMPORTANT]
> Si usa el filtrado de grupos en producción, debe contemplar la adopción de otro tipo de filtrado.

#### <a name="sync-failover-or-disaster-recovery"></a>Conmutación por error o recuperación ante desastres para la sincronización

Azure AD Connect desempeña un papel clave en el proceso de aprovisionamiento. Si por algún motivo el servidor de sincronización se queda sin conexión, los cambios en el entorno local no se actualizan en la nube y se pueden producir problemas de acceso para los usuarios. Por lo tanto, es importante definir una estrategia de conmutación por error que permita a los administradores reanudar rápidamente la sincronización después de que el servidor de sincronización se queda sin conexión. Estas estrategias se dividen en una de las siguientes categorías:

- **Implementación de los servidores de Azure AD Connect en modo de ensayo**: permite al administrador "promover" el servidor de ensayo a producción mediante un sencillo cambio de configuración.
- **Uso de virtualización**: si la conexión de Azure AD se implementa en una máquina virtual (VM), los administradores pueden aprovechar su pila de virtualización para migrar en vivo o volver a implementar rápidamente la máquina virtual y, de este modo, reanudar la sincronización.

Si su organización no dispone de una estrategia de recuperación ante desastres y conmutación por error para la sincronización, no dude en implementar Azure AD Connect en modo de ensayo. Del mismo modo, si hay una discrepancia entre la configuración de producción y de ensayo, debe volver a configurar el modo de ensayo de la línea de base de Azure AD Connect para que coincida con la configuración de producción, incluidas las versiones y las configuraciones de software.

![Captura de pantalla del modo de ensayo de Azure AD Connect](./media/active-directory-ops-guide/active-directory-ops-img1.png)

#### <a name="stay-current"></a>Mantenerse al día

Microsoft actualiza Azure AD Connect con regularidad. Manténgase al día para aprovechar las mejoras de rendimiento, las correcciones de errores y las nuevas capacidades que proporciona cada nueva versión.

Si la versión de Azure AD Connect tiene más de seis meses, debe actualizar a la versión más reciente.

#### <a name="source-anchor"></a>Delimitador de origen

El uso de **ms-DS-consistencyguid** como [delimitador de origen](../hybrid/plan-connect-design-concepts.md) permite una migración más sencilla de objetos entre bosques y dominios, tarea habitual en procesos de consolidación y limpieza, fusiones, adquisiciones y desinversiones en AD Domain.

Si actualmente está usando **ObjectGuid** como delimitador de origen, se recomienda empezar a utilizar **ms-DS-ConsistencyGuid**.

#### <a name="custom-rules"></a>Reglas personalizadas

Las reglas personalizadas de Azure AD Connect proporcionan la capacidad de controlar el flujo de atributos entre los objetos locales y los objetos en la nube. Sin embargo, la sobreutilización o el uso indebido de reglas personalizadas pueden presentar los siguientes riesgos:

- Complejidad de la solución de problemas
- Degradación del rendimiento al realizar operaciones complejas en objetos
- Mayor probabilidad de divergencias de configuración entre el servidor de producción y el servidor de ensayo
- Sobrecarga adicional al actualizar Azure AD Connect si se crean reglas personalizadas en una precedencia mayor a 100 (usada por las reglas integradas)

Si usa reglas excesivamente complejas, debe investigar las causas de la complejidad y buscar oportunidades de simplificación. Del mismo modo, si ha creado reglas personalizadas con un valor de precedencia superior a 100, debe corregir las reglas para que no estén en riesgo ni entren en conflicto con el conjunto predeterminado.

Entre los ejemplos de reglas personalizadas de uso incorrecto se incluyen:

- **Compensar los datos sucios en el directorio**: en este caso, se recomienda trabajar con los propietarios del equipo de AD y limpiar los datos en el directorio como una tarea de corrección, así como ajustar los procesos para evitar la reintroducción de datos incorrectos.
- **Corrección única de usuarios individuales**: es habitual encontrar reglas que se correlacionan con los valores atípicos de casos especiales, normalmente debido a un problema con un usuario específico.
- **"CloudFiltering" excesivamente complicado**: aunque se recomienda reducir el número de objetos, existe el riesgo de crear y complicar en exceso el ámbito de sincronización mediante el uso de muchas reglas de sincronización. Si hay una lógica compleja para incluir o excluir objetos más allá del filtrado de la unidad organizativa, se recomienda tratar esta lógica fuera de la sincronización y etiquetar los objetos con un simple atributo "cloudFiltered" que pueda fluir con una regla de sincronización simple.

#### <a name="azure-ad-connect-configuration-documenter"></a>Documentador de configuración de Azure AD Connect

El [documentador de configuración de Azure AD Connect](https://github.com/Microsoft/AADConnectConfigDocumenter) es una herramienta que puede usar para generar documentación de una instalación de Azure AD Connect a fin de permitir una mejor comprensión de la configuración de sincronización, generar confianza para hacer las cosas bien y saber qué ha cambiado cuando aplicó una nueva compilación o configuración de Azure AD Connect o agregó o actualizó reglas de sincronización personalizadas. Las funcionalidades actuales de la herramienta incluyen:

- Documentación de la configuración completa de la sincronización de Azure AD Connect.
- Documentación de los cambios en la configuración de dos servidores de sincronización de Azure AD Connect o cambios de una línea de base de configuración determinada.
- Generación de un script de implementación de PowerShell para migrar las diferencias o personalizaciones de la regla de sincronización de un servidor a otro.

## <a name="assignment-to-apps-and-resources"></a>Asignación a aplicaciones y recursos

### <a name="group-based-licensing-for-microsoft-cloud-services"></a>Concesión de licencias basada en grupo para servicios en la nube de Microsoft

Azure Active Directory simplifica la administración de licencias a través de la [concesión de licencias basada en grupos](./active-directory-licensing-whatis-azure-portal.md) para los servicios en la nube de Microsoft. De este modo, la administración de identidad y acceso proporciona la infraestructura de grupo y la administración delegada de esos grupos a los equipos adecuados en las organizaciones. Hay varias maneras de configurar la pertenencia a grupos en Azure AD, entre las que se incluyen:

- **Sincronización desde el entorno local**: los grupos pueden provenir de directorios locales, lo que podría ser una buena opción para las organizaciones que han establecido procesos de administración de grupos que se pueden ampliar para asignar licencias en Office 365.

- **Base en atributos/dinámica**: los grupos se pueden crear en la nube en función de una expresión basada en atributos de usuario, por ejemplo, Departamento igual a "ventas". Azure AD mantiene los miembros del grupo, manteniendo la coherencia con la expresión definida. El uso de este tipo de grupo para la asignación de licencias posibilita una asignación de licencias basada en atributos, que es una buena opción para las organizaciones que tienen una calidad de datos alta en su directorio.

- **Propiedad delegada**: los grupos se pueden crear en la nube y pueden ser propietarios designados. De este modo, puede permitir que los propietarios de empresas, por ejemplo, el equipo de colaboración o el equipo de BI, puedan definir quién debe tener acceso.

Si actualmente está usando un proceso manual para asignar licencias y componentes a los usuarios, se recomienda implementar las licencias basadas en grupos. Si el proceso actual no supervisa los errores de licencia o lo que está asignado frente a lo que está disponible, debe definir mejoras en el proceso para solucionar los errores relativos a las licencias y supervisar su asignación.

Otro aspecto de la administración de licencias es la definición de los planes de servicio (componentes de la licencia) que deben habilitarse en función de las funciones de trabajo de la organización. Otorgar acceso a planes de servicio que no son necesarios puede hacer que los usuarios vean herramientas en el portal de Office para las que no se han preparado o que no deberían utilizar. Puede generar un volumen adicional para el departamento de soporte técnico y un aprovisionamiento innecesario, además de poner en riesgo el cumplimiento y la gobernanza, por ejemplo, al aprovisionar OneDrive para la Empresa para individuos que podrían no tener permiso para compartir contenido.

Utilice las siguientes directrices para definir planes de servicio para los usuarios:

- Los administradores deben definir "agrupaciones" de los planes de servicio que se van a ofrecer a los usuarios en función de su rol, por ejemplo, trabajador de cuello blanco o de cuello azul.
- Cree grupos de este modo y asigne la licencia con el plan de servicio.
- Opcionalmente, se puede definir un atributo que contenga los paquetes para los usuarios.

> [!IMPORTANT]
> Las concesión de licencias basada en grupos de Azure AD incorpora el concepto de usuarios en estado de error de licencias. En caso de que se produzcan errores relativos a las licencias, debe [detectar y resolver](../users-groups-roles/licensing-groups-resolve-problems.md) de inmediato cualquier problema de asignación de licencias.

![Captura de pantalla de un equipo generada automáticamente](./media/active-directory-ops-guide/active-directory-ops-img2.png)

#### <a name="lifecycle-management"></a>Administración del ciclo de vida

Si actualmente usa una herramienta, como [Microsoft Identity Manager](/microsoft-identity-manager/) o un sistema de terceros, que se base en una infraestructura local, se recomienda descargar la asignación de la herramienta existente, implementar licencias basadas en grupos y definir una administración del ciclo de vida de los grupos basada en [grupos](../users-groups-roles/licensing-group-advanced.md#use-group-based-licensing-with-dynamic-groups). Del mismo modo, si el proceso existente no tiene en cuenta los nuevos empleados o los empleados que abandonan la organización, debe implementar licencias basadas en grupos dinámicos y definir un ciclo de vida de pertenencia a grupos. Por último, si implementa la concesión de licencias basada en grupo sobre grupos locales que carecen de administración del ciclo de vida, considere la posibilidad de usar grupos en la nube para habilitar funcionalidades como la propiedad delegada o la pertenencia dinámica basada en atributos.

### <a name="assignment-of-apps-with-all-users-group"></a>Asignación de aplicaciones con el grupo "Todos los usuarios"

Los propietarios de recursos pueden creer que el grupo **Todos los usuarios** solo contiene **Empleados de la empresa**, cuando puede que realmente contengan tanto **Empleados de la empresa** como **Invitados**. Como resultado, debe tener especial cuidado al usar el grupo **Todos los usuarios** para la asignación de aplicaciones y conceder acceso a recursos como aplicaciones o contenido de SharePoint.

> [!IMPORTANT]
> Si el grupo **Todos los usuarios** está habilitado y se usa para las directivas de acceso condicional o la asignación de recursos o aplicaciones, asegúrese de [proteger el grupo](../external-identities/use-dynamic-groups.md) si no desea que incluya a los usuarios invitados. Además, debe corregir las asignaciones de licencias creando y asignando grupos que contengan solo a los **Empleados de la empresa**. Por otro lado, si observa que el grupo **Todos los usuarios** está habilitado pero no se usa para conceder acceso a los recursos, asegúrese de que la directriz operativa de su organización sea usar intencionadamente ese grupo (que incluye tanto a los **empleados de la empresa** como a los **invitados**).

### <a name="automated-user-provisioning-to-apps"></a>Aprovisionamiento de usuarios automático a aplicaciones

El [aprovisionamiento de usuarios automático](../app-provisioning/user-provisioning.md) para aplicaciones es la mejor manera de crear un aprovisionamiento, desaprovisionamiento y ciclo de vida de identidades coherente entre varios sistemas.

Si actualmente está aprovisionando aplicaciones ad hoc o usa elementos como archivos CSV, JIT o una solución local que no se encargue de administrar el ciclo de vida, le recomendamos que [implemente el aprovisionamiento de aplicaciones](../app-provisioning/user-provisioning.md#how-do-i-set-up-automatic-provisioning-to-an-application) con Azure AD con las aplicaciones compatibles y que defina un patrón coherente para las aplicaciones que aún no son compatibles con Azure AD.

![Servicio de aprovisionamiento de Azure AD](./media/active-directory-ops-guide/active-directory-ops-img3.png)

### <a name="azure-ad-connect-delta-sync-cycle-baseline"></a>Línea de base del ciclo de sincronización diferencial de Azure AD Connect

Es importante comprender el volumen de cambios de la organización y no tardar demasiado en tener un tiempo de sincronización predecible.

La frecuencia de [sincronización diferencial predeterminada](../hybrid/how-to-connect-sync-feature-scheduler.md) es de 30 minutos. Si la sincronización diferencial tarda más de 30 minutos de manera recurrente, o hay discrepancias significativas entre el rendimiento de la sincronización diferencial de ensayo y producción, debe investigar y revisar los [factores que influyen en el rendimiento de Azure AD Connect](../hybrid/plan-connect-performance-factors.md).

#### <a name="azure-ad-connect-troubleshooting-recommended-reading"></a>Lectura recomendada para la solución de problemas de Azure AD Connect

- [Preparación de los atributos del directorio para sincronizarlos con Office 365 mediante la herramienta IdFix: Office 365](/office365/enterprise/prepare-directory-attributes-for-synch-with-idfix)
- [Azure AD Connect: Solución de errores durante la sincronización](../hybrid/tshoot-connect-sync-errors.md)

## <a name="summary"></a>Resumen

Una infraestructura de identidad segura está conformada por cinco aspectos. Esta lista le ayudará a encontrar y adoptar rápidamente las acciones necesarias para proteger y administrar el ciclo de vida de las identidades y sus derechos en su organización.

- Asignar propietarios a las tareas principales.
- Buscar y resolver problemas de sincronización.
- Definir una estrategia de conmutación por error para la recuperación ante desastres.
- Optimizar la administración de licencias y la asignación de aplicaciones.
- Automatizar el aprovisionamiento de usuarios a las aplicaciones.

## <a name="next-steps"></a>Pasos siguientes

Empiece con las [comprobaciones y las acciones de administración de la autenticación](active-directory-ops-guide-auth.md).