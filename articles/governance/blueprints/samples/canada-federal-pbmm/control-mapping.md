---
title: Controles del ejemplo de plano técnico Canada Federal PBMM
description: Asignación de controles de los ejemplos de plano técnico Canada Federal PBMM. Cada control se asigna a una o varias directivas de Azure que ayudan en la evaluación.
ms.date: 07/31/2020
ms.topic: sample
ms.openlocfilehash: 3022dc32f689d1cbfc8b5b6545d1252ce47ca33d
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500136"
---
# <a name="control-mapping-of-the-canada-federal-pbmm-blueprint-sample"></a>Asignación de controles del ejemplo de plano técnico de Canada Federal PBMM

En el siguiente artículo se explica detalladamente la forma en que el ejemplo de plano técnico de Azure Blueprints Canada Federal Protected B, Medium Integrity, Medium Availability (PBMM) se asigna a los controles de Canada Federal PBMM. Para más información sobre los controles, consulte [Canada Federal PBMM](https://www.canada.ca/en/government/system/digital-government/digital-government-innovations/cloud-services/government-canada-security-control-profile-cloud-based-it-services.html).

Las siguientes asignaciones son para los controles de **Canada Federal PBMM**. Use el panel de navegación de la derecha para ir directamente a una asignación de control específica. Muchos de los controles asignados se implementan con una iniciativa de [Azure Policy](../../../policy/overview.md). Para revisar la iniciativa completa, abra **Policy** en Azure Portal y seleccione la página **Definiciones**. Después, busque y seleccione la **\[versión preliminar\]: controles de Audit Canada Federal PBMM** de la iniciativa de directivas integradas.

> [!IMPORTANT]
> Cada control que se muestra a continuación está asociado a una o varias definiciones de [Azure Policy](../../../policy/overview.md). Estas directivas pueden ayudarle a [evaluar el cumplimiento](../../../policy/how-to/get-compliance-data.md) del control; sin embargo, a menudo no hay una coincidencia 1:1 o completa entre un control y una o varias directivas. Como tal, el **cumplimiento** con Azure Policy solo se refiere a las propias directivas; esto no garantiza que sea totalmente compatible con todos los requisitos de un control. Además, el estándar de cumplimiento incluye controles que no se abordan con las definiciones de Azure Policy en este momento. Por lo tanto, el cumplimiento en Azure Policy es solo una vista parcial del estado general de cumplimiento. Las asociaciones entre los controles y las definiciones de Azure Policy para este ejemplo de plano técnico de cumplimiento pueden cambiar con el tiempo. Para ver el historial de cambios, consulte el [historial de confirmación de GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/canada-federal-pbmm/control-mapping.md).

## <a name="location-constraints"></a>Restricciones de ubicación

Este plano técnico le ayuda a restringir la ubicación de la implementación de todos los recursos y grupos de recursos a "Centro de Canadá" y "Este de Canadá" mediante la asignación de las siguientes definiciones de Azure Policy:

- Ubicaciones permitidas (se ha codificado de forma rígida en "Centro de Canadá" y "Este de Canadá")
- Ubicaciones permitidas para grupos de recursos (se ha codificado de forma rígida en "Centro de Canadá" y "Este de Canadá")

## <a name="ac-2-account-management"></a>Administración de cuentas AC-2

Este plano técnico le ayudará a revisar aquellas cuentas que es posible que no cumplan los requisitos de administración de cuentas de su organización. Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) que auditan las cuentas externas con permisos de lectura, escritura y propietario en una suscripción y en las cuentas en desuso. Al revisar las cuentas auditadas por estas directivas, puede tomar las medidas adecuadas para garantizar que se cumplen los requisitos de administración de cuentas.

- Las cuentas en desuso deben quitarse de la suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de lectura deben quitarse de la suscripción
- Las cuentas externas con permisos de escritura deben quitarse de la suscripción


## <a name="ac-2-7-account-management--role-based-schemes"></a>Administración de cuentas AC-2 (7) | Esquemas basados en roles

Azure implementa el [control de acceso basado en rol](../../../../role-based-access-control/overview.md) (RBAC) para permitirle administrar quién tiene acceso a los recursos de Azure. Mediante Azure Portal, puede revisar quién tiene acceso a los recursos de Azure y sus permisos. Este proyecto también asigna definiciones de [Azure Policy](../../../policy/overview.md) para auditar el uso de la autenticación de Azure Active Directory de los servidores SQL Server y Service Fabric. El uso de la autenticación de Azure Active Directory simplifica la administración de permisos y centraliza la administración de identidades de usuarios de base de datos y otros servicios de Microsoft. Además, este proyecto asigna una definición de Azure Policy para auditar el uso de reglas RBAC personalizadas. Conocer dónde se implementan las reglas RBAC personalizadas ayuda a comprobar que la implementación se necesita y se realiza correctamente, ya que estas suelen producir errores.

- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server
- Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente

## <a name="ac-4-information-flow-enforcement"></a>Aplicación del flujo de información AC-4

El uso compartido de recursos entre orígenes (CORS) puede permitir que los recursos de App Services sean solicitados desde un dominio externo. Microsoft recomienda que solo permita que los dominios requeridos interactúen con las aplicaciones web, las funciones y las API. Este proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) para ayudar a supervisar las restricciones de acceso de recursos CORS en Azure Security Center.
Conocer las implementaciones de CORS puede ayudarle a comprobar que se han implementado los controles de flujo de información.

- CORS no debe permitir que todos los recursos accedan a las aplicaciones web.

## <a name="ac-5-separation-of-duties"></a>Separación de obligaciones AC-5

Tener solo un propietario de la suscripción de Azure no permite la redundancia administrativa. Por el contrario, tener demasiados propietarios de suscripción de Azure aumenta la posibilidad de infracción de seguridad mediante una cuenta de propietario en riesgo. Este proyecto ayuda a mantener un número adecuado de propietarios de suscripción de Azure mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan el número de estos. Este proyecto también asigna definiciones de Azure Policy que le ayudan a controlan la pertenencia del grupo Administradores en las máquinas virtuales Windows. La administración de permisos de administrador de máquina virtual y de propietario de suscripción ayuda a implementar la separación adecuada de las tareas.

- Debe designar un máximo de tres propietarios para la suscripción
- Debe haber más de un propietario asignado a su suscripción
- Mostrar los resultados de la auditoría de las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Mostrar los resultados de las auditorías de las máquinas virtuales Windows en las que el grupo Administradores no contenga todos los miembros especificados
- Implementar los requisitos para auditar las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Implementar los requisitos para auditar las máquinas virtuales Windows en las que el grupo Administradores no contenga todos los miembros especificados

## <a name="ac-6-least-privilege"></a>Privilegios mínimos AC-6

Azure implementa el [control de acceso basado en rol](../../../../role-based-access-control/overview.md) (RBAC) para permitirle administrar quién tiene acceso a los recursos de Azure. Mediante Azure Portal, puede revisar quién tiene acceso a los recursos de Azure y sus permisos. Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas cuya revisión es prioritaria. Revisar estos indicadores de la cuenta puede ayudarle a asegurarse de que se han implementado los controles de privilegios mínimos.

- Debe designar un máximo de tres propietarios para la suscripción
- Debe haber más de un propietario asignado a su suscripción
- Mostrar los resultados de la auditoría de las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Mostrar los resultados de las auditorías de las máquinas virtuales Windows en las que el grupo Administradores no contenga todos los miembros especificados
- Implementar los requisitos para auditar las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Implementar los requisitos para auditar las máquinas virtuales Windows en las que el grupo Administradores no contenga todos los miembros especificados

## <a name="ac-7-security-attributes"></a>Atributos de seguridad AC-7

La funcionalidad de detección y clasificación de datos de seguridad avanzada de datos de Azure SQL Database proporciona funcionalidades para detectar, clasificar, etiquetar y proteger los datos confidenciales de las bases de datos. Se puede utilizar para proporcionar visibilidad del estado de clasificación de una base de datos y para realizar un seguimiento del acceso a información confidencial dentro de la base de datos y más allá de sus límites. La seguridad avanzada de datos puede ayudarle a garantizar que la información está asociada con los atributos de seguridad adecuados para su organización. Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) para supervisar y exigir el uso de la seguridad avanzada de datos en SQL Server.

- La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Implementación de Advanced Data Security en los servidores de SQL Server

## <a name="ac-17-1-remote-access--automated-monitoring--control"></a>Acceso remoto AC-17 (1) | Control y supervisión automatizados

Este plano técnico le ayuda a supervisar y controlar el acceso remoto mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) para supervisar que la depuración remota de la aplicación Azure App Service está desactivada. El plano técnico también asigna definiciones de directiva que auditan las máquinas virtuales Linux que permiten conexiones remotas desde cuentas sin contraseñas. Además, el plano técnico asigna una definición de Azure Policy que le ayuda a supervisar el acceso sin restricciones a las cuentas de almacenamiento. La supervisión de estos indicadores puede ayudarle a garantizar que los métodos de acceso remoto cumplan con la directiva de seguridad.

- \[Versión preliminar\]: mostrar los resultados de las auditorías de máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña
- \[Versión preliminar\]: implementación de los requisitos para auditar máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña
- Auditar el acceso de red sin restricciones a cuentas de almacenamiento
- Se debe desactivar la depuración remota para aplicaciones de API
- Recomendación de desactivación de la depuración remota para Function App
- Recomendación de desactivación de la depuración remota para aplicaciones web

## <a name="au-3-2-content-of-audit-records"></a>Contenido de registros de auditoría AU-3 (2)

Los datos de registro que recopila Azure Monitor se almacenan en un área de trabajo de Log Analytics que permite la configuración y administración centralizados. Este proyecto le ayuda a garantizar que los eventos se registran mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan y exigen la implementación del agente de Log Analytics en máquinas virtuales de Azure.

- \[Versión preliminar\]: Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista
- \[Versión preliminar\]: Auditoría de la implementación del agente de Log Analytics en VMSS: la imagen de la VM (SO) no está en la lista
- \[Versión preliminar\]: Auditar área de trabajo de Log Analytics en la máquina virtual: error de coincidencia del informe
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows

## <a name="au-5-response-to-audit-processing-failures"></a>Respuesta a errores de procesamiento de auditoría AU-5

Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) que supervisan las configuraciones del registro de eventos y auditoría. La supervisión de estas configuraciones puede proporcionar un indicador del error del sistema de auditoría o de una configuración incorrecta, y le ayudarán a tomar medidas correctivas.

- Auditar la configuración de diagnóstico
- La auditoría de SQL Server debe estar habilitada
- La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server

## <a name="au-6-4-audit-review-analysis-and-reporting--central-review-and-analysis"></a>Revisión, análisis e informes de auditoría AU-6 (4) | Revisión y análisis central

Los datos de registro que recopila Azure Monitor se almacenan en un área de trabajo de Log Analytics que permite el análisis y los informes centralizados. Este proyecto le ayuda a garantizar que los eventos se registran mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan y exigen la implementación del agente de Log Analytics en máquinas virtuales de Azure.

- \[Versión preliminar\]: Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista
- Auditoría de la implementación del agente de Log Analytics en VMSS: la imagen de la VM (SO) no está en la lista
- Auditar área de trabajo de Log Analytics en la máquina virtual: error de coincidencia del informe
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows

## <a name="au-12-audit-generation"></a>Generación de auditoría AU-12

Este plano técnico ayuda a garantizar que los eventos del sistema se registran mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan las configuraciones de registro de los recursos de Azure. Estas definiciones de directiva auditan y aplican la implementación del agente de Log Analytics en las máquinas virtuales Azure y la configuración de la auditoría para otros tipos de recursos de Azure. Estas definiciones de directiva también auditan la configuración de los registros de diagnóstico para proporcionar información detallada sobre las operaciones realizadas dentro de los recursos de Azure. Además, la auditoría y Advanced Data Security se configuran en los servidores SQL Server.

- \[Versión preliminar\]: Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista
- Auditoría de la implementación del agente de Log Analytics en VMSS: la imagen de la VM (SO) no está en la lista
- Área de trabajo de Audit Log Analytics para máquina virtual: informe 

- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows
- Auditar la configuración de diagnóstico
- La auditoría de SQL Server debe estar habilitada
- La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Implementación de Advanced Data Security en los servidores de SQL Server
- Implementación de auditorías en servidores SQL Server
- Implementación de la configuración de diagnóstico de grupos de seguridad de red

## <a name="cm-7-5-least-functionality--authorized-software--whitelisting"></a>Funcionalidad mínima CM-7 (5) | Creación de listas de software permitido o autorizado

El control de aplicaciones adaptable en Azure Security Center es una solución de creación de listas de aplicaciones permitidas integral, inteligente y automatizada que puede bloquear o impedir que se ejecute software específico en las máquinas virtuales. El control de aplicaciones le permite crear listas de aplicaciones aprobadas para las máquinas virtuales. Este proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales cuando se recomienda crear una lista de aplicaciones permitidas, pero aún no se ha configurado.

- Deben habilitarse los controles de aplicaciones adaptables en las máquinas virtuales

## <a name="cm-11-user-installed-software"></a>Software instalado por el usuario CM-11

El control de aplicaciones adaptable en Azure Security Center es una solución de creación de listas de aplicaciones permitidas integral, inteligente y automatizada que puede bloquear o impedir que se ejecute software específico en las máquinas virtuales. El control de aplicaciones puede ayudarle a aplicar y supervisar el cumplimiento con las directivas de restricción de software. Este proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales cuando se recomienda crear una lista de aplicaciones permitidas, pero aún no se ha configurado.

- Deben habilitarse los controles de aplicaciones adaptables en las máquinas virtuales

## <a name="cp-7-alternate-processing-site"></a>Sitio de procesamiento alternativo CP-7

Azure Site Recovery replica las cargas de trabajo que se ejecutan en máquinas virtuales desde una ubicación remota a una ubicación secundaria. Si se produce una interrupción en el sitio primario, la carga de trabajo conmutar por error a la ubicación secundaria. Este proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) que audita las máquinas virtuales que no tienen configurada la recuperación ante desastres. Supervisar este indicador puede ayudarle a asegurarse de que los controles de contingencia necesarios están en su lugar.

- Auditoría de máquinas virtuales sin la recuperación ante desastres configurada

## <a name="ia-2-1-identification-and-authentication-organizational-users--network-access-to-privileged-accounts"></a>Identificación y autenticación IA-2 (1) (usuarios de la organización) | Acceso de red a cuentas con privilegios

Este proyecto ayuda a restringir y controlar el acceso con privilegios mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas con permisos de propietario o escritura que no tengan habilitada la autenticación multifactor. La autenticación multifactor ayuda a mantener las cuentas seguras aunque algún dato de la autenticación esté en riesgo. Mediante la supervisión de las cuentas que no tienen la autenticación multifactor habilitada puede identificar las cuentas en mayor o menor riesgo.

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción

## <a name="ia-5-authenticator-management"></a>Administración de autenticadores IA-5

Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) que auditan las máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña o que tengan permisos incorrectos establecidos en el archivo de contraseñas. Este proyecto también asigna definiciones de directiva que auditan la configuración del tipo de cifrado de contraseña en las máquinas virtuales Windows. La supervisión de estos indicadores le ayuda a asegurarse de que los autenticadores del sistema cumplen con la directiva de identificación y autenticación de la organización.

- mostrar los resultados de las auditorías de las máquinas virtuales Linux que no tengan los permisos del archivo de contraseñas establecidos en 0644
- mostrar los resultados de la auditoría de las máquinas virtuales Linux que tengan cuentas sin contraseña
- implementación de los requisitos para auditar las máquinas virtuales Linux que no tengan los permisos del archivo de contraseñas establecidos en 0644
- implementación de los requisitos para auditar las máquinas virtuales Linux que tengan cuentas sin contraseña

## <a name="ia-5-1-authenticator-management--password-based-authentication"></a>Administración de autenticadores IA-5 (1) | Autenticación basada en contraseña

Este proyecto ayuda a exigir contraseñas seguras mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan las máquinas virtuales Windows que no exigen unos requisitos mínimos de seguridad de contraseña y de otros tipos. Conocer las máquinas virtuales que infringen la directiva de seguridad de las contraseñas ayuda a tomar medidas correctivas para garantizar que las contraseñas de todas las cuentas de usuario de máquina virtual cumplen la directiva de contraseñas de la organización.

- mostrar los resultados de las auditorías de las máquinas virtuales Windows que permitan volver a usar las 24 contraseñas anteriores
- mostrar los resultados de las auditorías de las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia máxima de 70 días
- mostrar los resultados de las auditorías de las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia mínima de 1 día
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que no tengan habilitada la configuración de complejidad de la contraseña
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas en 14 caracteres
- implementación de los requisitos para auditar las máquinas virtuales Windows que permitan volver a usar las 24 contraseñas anteriores
- implementación de los requisitos para auditar las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia máxima de 70 días
- implementación de los requisitos para auditar las máquina virtuales Windows cuyas contraseñas no tengan una vigencia mínima de 1 día
- implementación de los requisitos para auditar las máquinas virtuales que no tengan habilitada la configuración de complejidad de la contraseña
- implementación de los requisitos para auditar las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas a 14 caracteres

## <a name="ia-8-100-identification-and-authentication-non-organizational-users--identity-and-credential-assurance-levels"></a>Identificación y autenticación (usuarios que no pertenecen a la organización) | Niveles de comprobación de identidad y credenciales IA-8 (100)

Este proyecto ayuda a restringir y controlar el acceso con privilegios mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas con permisos de propietario o escritura que no tengan habilitada la autenticación multifactor. La autenticación multifactor ayuda a mantener las cuentas seguras aunque algún dato de la autenticación esté en riesgo. Mediante la supervisión de las cuentas que no tienen la autenticación multifactor habilitada puede identificar las cuentas en mayor o menor riesgo.

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción

## <a name="ra-5-vulnerability-scanning"></a>Examen de vulnerabilidades RA-5

Este proyecto ayuda a administrar las vulnerabilidades del sistema de información mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan las vulnerabilidades del sistema operativo, las de SQL o las de las máquinas virtuales en Azure Security Center. Azure Security Center proporciona funcionalidades de informes que permiten tener información en tiempo real sobre el estado de seguridad de los recursos de Azure implementados. Este proyecto asigna también definiciones de directiva que auditan y exigen Advanced Data Security en servidores SQL. Advanced Data Security incluía la evaluación de vulnerabilidades y funcionalidades de protección contra amenazas avanzada para ayudarle a entender las vulnerabilidades en los recursos implementados.

- La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Implementación de Advanced Data Security en los servidores de SQL Server
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades en la configuración de seguridad de las máquinas virtuales
- Se deben corregir las vulnerabilidades de las bases de datos SQL
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades

## <a name="sc-5-denial-of-service-protection"></a>Protección ante la denegación de servicio SC-5

El nivel estándar de denegación de servicio distribuido (DDoS) de Azure proporciona características adicionales y funcionalidades de mitigación en el nivel de servicio básico. Estas características adicionales incluyen la integración de Azure Monitor y la capacidad de revisar los informes de mitigación posteriores al ataque. Este proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) que audita si está habilitado el nivel estándar de DDoS. Conocer la diferencia de capacidad entre los niveles de servicio puede ayudarle a seleccionar la mejor solución para abordar la denegación de protecciones de servicio para el entorno de Azure.

- Se debe habilitar DDoS Protection estándar

## <a name="sc-7-boundary-protection"></a>Protección de límites SC-7

Este proyecto le ayuda a administrar y controlar el límite del sistema mediante la asignación de una definición de [Azure Policy](../../../policy/overview.md) que supervisa las recomendaciones de protección del grupo de seguridad de red en Azure Security Center. Azure Security Center analiza los patrones de tráfico de las máquinas virtuales de Internet y proporciona recomendaciones de reglas de grupo de seguridad de red para reducir la superficie potencial de ataque.
Además, este proyecto también asigna definiciones de directiva que supervisan las cuentas de almacenamiento, las aplicaciones y los puntos de conexión desprotegidos. Los puntos de conexión y las aplicaciones que no están protegidos por firewall y las cuentas de almacenamiento con acceso sin restricciones pueden permitir el acceso no deseado a la información contenida en el sistema de información.

- Las recomendaciones de protección de red adaptable se deben aplicar en las máquinas virtuales accesibles desde Internet
- Debe restringirse el acceso a través de un punto de conexión accesible desde Internet
- Auditar el acceso de red sin restricciones a cuentas de almacenamiento

## <a name="sc-7-3-boundary-protection--access-points"></a>Protección de límites SC-7 (3) | Puntos de acceso

El acceso a máquinas virtuales Just-In-Time (JIT) bloquea el tráfico entrante a las máquinas virtuales de Azure, lo que reduce la exposición a ataques y, al mismo tiempo, se proporciona acceso sencillo para conectarse a las máquinas virtuales cuando sea necesario. El acceso a máquinas virtuales JIT le ayuda a limitar el número de conexiones externas a los recursos de Azure. Este plano técnico asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales que pueden admitir el acceso Just-In-Time pero aún no se han configurado.

- El control de acceso de red Just-In-Time se debe aplicar en las máquinas virtuales.

## <a name="sc-7-4-boundary-protection--external-telecommunications-services"></a>Protección de límites SC-7 (4) | Servicios de telecomunicaciones externos

El acceso a máquinas virtuales Just-In-Time (JIT) bloquea el tráfico entrante a las máquinas virtuales de Azure, lo que reduce la exposición a ataques y, al mismo tiempo, se proporciona acceso sencillo para conectarse a las máquinas virtuales cuando sea necesario. El acceso a máquinas virtuales JIT le ayuda a administrar las excepciones a la directiva de flujo de tráfico al facilitar los procesos de solicitud y aprobación de acceso. Este plano técnico asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales que pueden admitir el acceso Just-In-Time pero aún no se han configurado.

- El control de acceso de red Just-In-Time se debe aplicar en las máquinas virtuales.

## <a name="sc-8-1-transmission-confidentiality-and-integrity--cryptographic-or-alternate-physical-protection"></a>SC-8 (1) Integridad y confidencialidad de la transmisión | Protección criptográfica o protección física alternativa

Este proyecto le ayuda a proteger la confidencialidad y la integridad de la información transmitida mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que le facilitan la supervisión del mecanismo criptográfico implementado para los protocolos de comunicaciones. Asegurar que las comunicaciones están correctamente cifradas puede ayudarle a satisfacer los requisitos de su organización o a proteger la información frente a la divulgación y la modificación no autorizadas.

- Acceso a API App solo a través de HTTPS
- Muestra de los resultados de la auditoría de los servidores web de Windows que no estén usando protocolos de comunicación segura.
- Implementación de los requisitos para auditar los servidores web de Windows que no estén usando los protocolos de comunicación segura
- Acceso a Function App solo a través de HTTPS
- Solo se deben habilitar las conexiones seguras a Redis Cache
- Acceso a la aplicación web solo a través de HTTPS
- Se debe habilitar la transferencia segura a las cuentas de almacenamiento

## <a name="sc-28-1-protection-of-information-at-rest"></a>Protección de la información en reposo SC-28 (1)

Este proyecto ayuda a exigir la directiva sobre el uso de controles criptográficos para proteger la información en reposo mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que exigen controles criptográficos específicos y auditan el uso de configuraciones criptográficas poco seguras. Conocer dónde los recursos de Azure pueden tener configuraciones criptográficas subóptimas ayuda a tomar medidas correctivas para garantizar que los recursos se configuran conforme a la directiva de seguridad de la información. En concreto, las definiciones de directiva asignadas por este proyecto requieren cifrado para las cuentas de almacenamiento del lago de datos, requerir el cifrado de datos transparente en las bases de datos SQL y auditar si falta el cifrado en las bases de datos SQL, los discos de máquinas virtuales y las variables de cuenta de automatización.

- La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Implementación de Advanced Data Security en los servidores de SQL Server
- Implementación del cifrado de datos transparente de SQL DB
- El cifrado de discos debe aplicarse en máquinas virtuales
- Exigencia de cifrado en las cuentas de Data Lake Store.
- Se debe permitir el cifrado de datos transparente en bases de datos SQL

## <a name="si-2-flaw-remediation"></a>Corrección de errores SI-2

Este proyecto ayuda a administrar los errores del sistema de información mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan si faltan actualizaciones del sistema o si existen vulnerabilidades del sistema operativo, de SQL o de las máquinas virtuales de Azure Security Center. Azure Security Center proporciona funcionalidades de informes que permiten tener información en tiempo real sobre el estado de seguridad de los recursos de Azure implementados. Este proyecto también asigna una definición de directiva que garantiza la aplicación automática de revisiones del sistema operativo para los conjuntos de escalado de máquinas virtuales.

- Exigir la aplicación automática de revisiones de imágenes del sistema operativo en Virtual Machine Scale Sets
- Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales
- Se deben instalar las actualizaciones del sistema en las máquinas virtuales
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades en la configuración de seguridad de las máquinas virtuales
- Se deben corregir las vulnerabilidades de las bases de datos SQL
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades

## <a name="si-3-malicious-code-protection"></a>Protección frente a código malintencionado SI-3

Este proyecto le ayuda a administrar la protección de los puntos de conexión, incluida la protección del código malintencionado, mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan que no falte la protección de los puntos de conexión en las máquinas virtuales en Azure Security Center y aplican la solución antimalware de Microsoft en las máquinas virtuales Windows.

- Implementación de la extensión de Microsoft IaaSAntimalware predeterminada para Windows Server
- La solución de protección del punto de conexión debe instalarse en las máquinas virtuales
- Supervisión de la falta de Endpoint Protection en Azure Security Center

## <a name="si-3-1-malicious-code-protection--central-management"></a>Protección frente a código malintencionado SI-3 (1) | Administración central

Este proyecto le ayuda a administrar la protección de los puntos de conexión, incluida la protección del código malintencionado, mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan que no falte la protección de los puntos de conexión en las máquinas virtuales en Azure Security Center. Azure Security Center proporciona funcionalidades de informes y administración centralizada que permiten tener información en tiempo real sobre el estado de seguridad de los recursos de Azure implementados.

- La solución de protección del punto de conexión debe instalarse en las máquinas virtuales
- Supervisión de la falta de Endpoint Protection en Azure Security Center

## <a name="si-4-information-system-monitoring"></a>Supervisión del sistema de información SI-4

Este proyecto le ayuda a supervisar el sistema de auditoría y cumplimiento de seguridad de datos y registro en los recursos de Azure. En concreto, las directivas asignaban la auditoría e implementación del agente de Log Analytics, y mejoraban la configuración de seguridad de las bases de datos SQL, las cuentas de almacenamiento y los recursos de red. Estas funcionalidades pueden ayudarle a detectar los comportamientos anómalos y los indicadores de ataques para que pueda tomar las acciones adecuadas.

- \[Versión preliminar\]: Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista
- Auditoría de la implementación del agente de Log Analytics en VMSS: la imagen de la VM (SO) no está en la lista
- \[Versión preliminar\]: Auditar área de trabajo de Log Analytics en la máquina virtual: error de coincidencia del informe
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows
- La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Implementación de Advanced Data Security en los servidores de SQL Server
- Implementar Advanced Threat Protection en las cuentas de almacenamiento
- Implementación de auditorías en servidores SQL Server
- Implementar Network Watcher al crear redes virtuales.
- Implementación de la detección de amenazas en servidores SQL Server.

> [!NOTE]
> La disponibilidad de definiciones específicas de Azure Policy puede variar tanto en Azure Government como en otras nubes nacionales. 

## <a name="next-steps"></a>Pasos siguientes

Ya ha revisado la asignación de control del ejemplo de plano técnico de Canada Federal PBMM. Ahora, consulte los siguientes artículos para ver una introducción y aprender a implementar este ejemplo:

> [!div class="nextstepaction"]
> [Plano técnico de Canada Federal PBMM: Introducción](./control-mapping.md)
> [Plano técnico de Canada Federal PBMM: Pasos de implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
