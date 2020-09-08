---
title: Controles del ejemplo de plano técnico de la norma SWIFT CSP-CSCF v2020
description: Asignación de controles del ejemplo de plano técnico SWIFT CSP-CSCF v2020. Cada control se asigna a una o varias directivas de Azure que ayudan en la evaluación.
ms.date: 08/18/2020
ms.topic: sample
ms.openlocfilehash: 06a334ff05db1cfc7ad1fb09e370963486ee1515
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297424"
---
# <a name="control-mapping-of-the-swift-csp-cscf-v2020-blueprint-sample"></a>Asignación de controles del ejemplo de plano técnico SWIFT CSP-CSCF v2020

En el siguiente artículo se detalla cómo se asigna el ejemplo de plano técnico SWIFT CSP-CSCF v2020 de Azure Blueprints a los controles de SWIFT CSP-CSCF v2020. Para más información sobre los controles, consulte [SWIFT CSP-CSCF v2020](https://www.swift.com/myswift/customer-security-programme-csp).

Las siguientes asignaciones se realizan en los controles de **SWIFT CSP-CSCF v2020**. Use el panel de navegación de la derecha para ir directamente a una asignación de control específica. Muchos de los controles asignados se implementan con una iniciativa de [Azure Policy](../../../policy/overview.md). Para revisar la iniciativa completa, abra **Policy** en Azure Portal y seleccione la página **Definiciones**. Después, busque y seleccione la **\[versión preliminar\]: Audite los controles SWIFT CSP-CSCF v2020 e implemente extensiones de máquina virtual específicas para admitir la iniciativa de directiva integrada** de los requisitos de auditoría.

> [!IMPORTANT]
> Cada control que se muestra a continuación está asociado a una o varias definiciones de [Azure Policy](../../../policy/overview.md). Estas directivas pueden ayudarle a [evaluar el cumplimiento](../../../policy/how-to/get-compliance-data.md) del control; sin embargo, a menudo no hay una coincidencia 1:1 o completa entre un control y una o varias directivas. Como tal, el **cumplimiento** con Azure Policy solo se refiere a las propias directivas; esto no garantiza que sea totalmente compatible con todos los requisitos de un control. Además, el estándar de cumplimiento incluye controles que no se abordan con las definiciones de Azure Policy en este momento. Por lo tanto, el cumplimiento en Azure Policy es solo una vista parcial del estado general de cumplimiento. Las asociaciones entre los controles y las definiciones de Azure Policy para este ejemplo de plano técnico de cumplimiento pueden cambiar con el tiempo. Para ver el historial de cambios, consulte el [historial de confirmación de GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/swift-2020/control-mapping.md).

## <a name="12-and-51-account-management"></a>1.2 y 5.1 Administración de cuentas

Este proyecto le ayudará a revisar las cuentas que pueda que no cumplan los requisitos de administración de cuentas de su organización. Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) que auditan las cuentas externas con permisos de lectura, escritura y propietario en una suscripción y en las cuentas en desuso. Al revisar las cuentas auditadas por estas directivas, puede tomar las medidas adecuadas para garantizar que se cumplen los requisitos de administración de cuentas.

- Las cuentas en desuso deben quitarse de la suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de lectura deben quitarse de la suscripción
- Las cuentas externas con permisos de escritura deben quitarse de la suscripción

## <a name="26-51-64-and-65a-account-management--role-based-schemes"></a>2.6, 5.1, 6.4 y 6.5A Administración de cuentas | Esquemas basados en roles

Azure implementa el [control de acceso basado en rol](../../../../role-based-access-control/overview.md) (RBAC) para permitirle administrar quién tiene acceso a los recursos de Azure. Mediante Azure Portal, puede revisar quién tiene acceso a los recursos de Azure y sus permisos. Este proyecto también asigna definiciones de [Azure Policy](../../../policy/overview.md) para auditar el uso de la autenticación de Azure Active Directory de los servidores SQL Server y Service Fabric. El uso de la autenticación de Azure Active Directory simplifica la administración de permisos y centraliza la administración de identidades de usuarios de base de datos y otros servicios de Microsoft. Además, este proyecto asigna una definición de Azure Policy para auditar el uso de reglas RBAC personalizadas. Conocer dónde se implementan las reglas RBAC personalizadas ayuda a verificar que la implementación se necesita y se realiza correctamente, ya que estas suelen producir errores.

- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server
- Auditar las máquinas virtuales que no utilizan discos administrados
- Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente

## <a name="29a--account-management--account-monitoring--atypical-usage"></a>2.9A Administración de cuentas | Supervisión o uso inusual de cuentas

El acceso a máquinas virtuales Just-In-Time (JIT) bloquea el tráfico entrante a las máquinas virtuales de Azure, lo que reduce la exposición a ataques y, al mismo tiempo, se proporciona acceso sencillo para conectarse a las máquinas virtuales cuando sea necesario. Todas las solicitudes JIT para tener acceso a las máquinas virtuales se registran en el registro de actividad que le permiten supervisar el uso inusual. Esta instancia de proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales que pueden admitir el acceso just-in-time pero aún no se han configurado.

- Los puertos de administración de las máquinas virtuales deben protegerse con el control de acceso de red Just-In-Time.

## <a name="13-51-and-64-separation-of-duties"></a>1.3, 5.1 y 6.4 Separación de obligaciones

Tener solo un propietario de la suscripción de Azure no permite la redundancia administrativa. Por el contrario, tener demasiados propietarios de suscripción de Azure aumenta la posibilidad de infracción de seguridad mediante una cuenta de propietario en riesgo. Este proyecto ayuda a mantener un número adecuado de propietarios de suscripción de Azure mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan el número de estos. Este proyecto también asigna definiciones de Azure Policy que le ayudan a controlan la pertenencia del grupo Administradores en las máquinas virtuales Windows. La administración de permisos de administrador de máquina virtual y de propietario de suscripción ayuda a implementar la separación adecuada de las tareas.

- Debe designar un máximo de tres propietarios para la suscripción
- Mostrar los resultados de las auditorías de las máquinas virtuales Windows en las que el grupo Administradores no contenga todos los miembros especificados
- Implementar los requisitos previos para auditar las máquinas virtuales Windows en las que el grupo Administradores no contenga todos los miembros especificados
- Debe haber más de un propietario asignado a su suscripción

## <a name="13-51-and-64-least-privilege--review-of-user-privileges"></a>1.3, 5.1 y 6.4 Privilegios mínimos | Revisión de privilegios de usuario

Azure implementa el [control de acceso basado en rol](../../../../role-based-access-control/overview.md) (RBAC) para permitirle administrar quién tiene acceso a los recursos de Azure. Mediante Azure Portal, puede revisar quién tiene acceso a los recursos de Azure y sus permisos. Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas cuya revisión es prioritaria. Revisar estos indicadores de la cuenta puede ayudarle a asegurarse de que se han implementado los controles de privilegios mínimos.

- Debe designar un máximo de tres propietarios para la suscripción
- Mostrar los resultados de la auditoría de las VM Windows que no estén unidas al dominio especificado
- Implementar los requisitos previos para realizar una auditoría de las VM Windows que no estén unidas al dominio especificado
- Debe haber más de un propietario asignado a su suscripción

## <a name="22-and-27-security-attributes"></a>2.2 y 2.7 Atributos de seguridad

La funcionalidad de detección y clasificación de datos de seguridad avanzada de datos de Azure SQL Database proporciona funcionalidades para detectar, clasificar, etiquetar y proteger los datos confidenciales de las bases de datos. Se puede utilizar para proporcionar visibilidad del estado de clasificación de una base de datos y para realizar un seguimiento del acceso a información confidencial dentro de la base de datos y más allá de sus límites. La seguridad avanzada de datos puede ayudarle a garantizar que la información está asociada con los atributos de seguridad adecuados para su organización. Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) para supervisar y exigir el uso de la seguridad avanzada de datos en SQL Server. 

- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Implementación de Advanced Data Security en los servidores de SQL Server

## <a name="22-27-41-and-61-remote-access--automated-monitoring--control"></a>2.2, 2.7, 4.1 y 6.1 Acceso remoto | Control y supervisión automatizados

Este proyecto le ayuda a supervisar y controlar el acceso remoto mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) para supervisar que la depuración remota de la aplicación de Azure App Service está desactivada y para auditar las máquinas virtuales Linux que permiten las conexiones remotas desde cuentas sin contraseñas. Este proyecto también asigna una definición de Azure Policy que le ayuda a supervisar el acceso sin restricciones a las cuentas de almacenamiento. La supervisión de estos indicadores puede ayudarle a garantizar que los métodos de acceso remoto cumplan con la directiva de seguridad.

- mostrar los resultados de las auditorías de máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña
- implementar los requisitos previos para auditar máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña
- Se debe restringir el acceso de red a las cuentas de almacenamiento.
- Se debe desactivar la depuración remota para aplicaciones de API
- Recomendación de desactivación de la depuración remota para Function App
- Recomendación de desactivación de la depuración remota para aplicaciones web

## <a name="13-and-64-content-of-audit-records--centralized-management-of-planned-audit-record-content"></a>1.3 y 6.4 Contenido de los registros de auditoría | Administración centralizada del contenido de los registros de auditoría planeada

Los datos de registro que recopila Azure Monitor se almacenan en un área de trabajo de Log Analytics que permite la configuración y administración centralizados. Este proyecto le ayuda a garantizar que los eventos se registran mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan y exigen la implementación del agente de Log Analytics en máquinas virtuales de Azure.

- \[Versión preliminar\]: Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista
- Implementar el agente de Log Analytics en máquinas virtuales Linux
- Implementar el agente de Log Analytics en máquinas virtuales Windows

## <a name="22-27-and-64-response-to-audit-processing-failures"></a>2.2, 2.7 y 6.4 Respuesta a errores de procesamiento de auditoría

Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) que supervisan las configuraciones del registro de eventos y auditoría. La supervisión de estas configuraciones puede proporcionar un indicador del error del sistema de auditoría o de una configuración incorrecta, y le ayudarán a tomar medidas correctivas.

- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Auditar la configuración de diagnóstico
- La auditoría de SQL Server debe estar habilitada

## <a name="13-and-64-audit-review-analysis-and-reporting--central-review-and-analysis"></a>1.3 y 6.4 Revisión, análisis e informes de auditorías | Revisión y análisis centralizados

Los datos de registro que recopila Azure Monitor se almacenan en un área de trabajo de Log Analytics que permite el análisis y los informes centralizados. Este proyecto le ayuda a garantizar que los eventos se registran mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan y exigen la implementación del agente de Log Analytics en máquinas virtuales de Azure.

- \[Versión preliminar\]: Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista
- Implementar el agente de Log Analytics en máquinas virtuales Linux
- Implementar el agente de Log Analytics en máquinas virtuales Windows

## <a name="13-22-27-64-and-65a-audit-generation"></a>1.3, 2.2, 2.7, 6.4 y 6.5A Generación de auditoría

Este plano técnico ayuda a garantizar que los eventos del sistema se registran mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan las configuraciones de registro de los recursos de Azure. Estas definiciones de directiva auditan y aplican la implementación del agente de Log Analytics en las máquinas virtuales Azure y la configuración de la auditoría para otros tipos de recursos de Azure. Estas definiciones de directiva también auditan la configuración de los registros de diagnóstico para proporcionar información detallada sobre las operaciones realizadas dentro de los recursos de Azure. Además, la auditoría y Advanced Data Security se configuran en los servidores SQL Server.

- Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista
- Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Linux
- Implementar el agente de Log Analytics en máquinas virtuales Linux
- Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Windows
- Implementar el agente de Log Analytics en máquinas virtuales Windows
- Auditar la configuración de diagnóstico
- Auditar configuración de auditoría de SQL en el nivel de servidor
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Implementación de Advanced Data Security en los servidores de SQL Server
- La auditoría de SQL Server debe estar habilitada
- Implementación de la configuración de diagnóstico de grupos de seguridad de red

## <a name="11-least-functionality--prevent-program-execution"></a>1.1 Funcionalidad mínima | Evitación de la ejecución de programas

El control de aplicaciones adaptable en Azure Security Center es una solución de filtrado de aplicaciones de un extremo a otro, inteligente y automatizada que puede bloquear o impedir que se ejecute software específico en las máquinas virtuales. El control de aplicaciones puede ejecutarse en modo de aplicación que prohíbe que la aplicación no autorizada se ejecute. Este plano técnico asigna una definición de Azure Policy que le ayuda a supervisar las máquinas virtuales cuando se recomienda la existencia de una lista de aplicaciones permitidas, pero esta aún no se ha configurado.

- Los controles de aplicaciones adaptables para definir aplicaciones seguras deben estar habilitados en las máquinas.

## <a name="11-least-functionality--authorized-software--whitelisting"></a>1.1 Funcionalidad mínima | Inclusión en listas blancas de software autorizado

El control de aplicaciones adaptable en Azure Security Center es una solución de filtrado de aplicaciones de un extremo a otro, inteligente y automatizada que puede bloquear o impedir que se ejecute software específico en las máquinas virtuales. El control de aplicaciones le permite crear listas de aplicaciones aprobadas para las máquinas virtuales. Este plano técnico asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales cuando se recomienda la existencia de una lista de aplicaciones permitidas, pero esta aún no se ha configurado.

- Los controles de aplicaciones adaptables para definir aplicaciones seguras deben estar habilitados en las máquinas.

## <a name="11-user-installed-software"></a>1.1 Software instalado por el usuario

El control de aplicaciones adaptable en Azure Security Center es una solución de filtrado de aplicaciones de un extremo a otro, inteligente y automatizada que puede bloquear o impedir que se ejecute software específico en las máquinas virtuales. El control de aplicaciones puede ayudarle a aplicar y supervisar el cumplimiento con las directivas de restricción de software. Este plano técnico asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales cuando se recomienda la existencia de una lista de aplicaciones permitidas, pero esta aún no se ha configurado.

- Los controles de aplicaciones adaptables para definir aplicaciones seguras deben estar habilitados en las máquinas.
- Se deben migrar las máquinas virtuales a nuevos recursos de Azure Resource Manager

## <a name="42-identification-and-authentication-organizational-users--network-access-to-privileged-accounts"></a>4.2 Identificación y autenticación (usuarios de la organización) | Acceso de red a cuentas con privilegios

Este proyecto ayuda a restringir y controlar el acceso con privilegios mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas con permisos de propietario o escritura que no tengan habilitada la autenticación multifactor. La autenticación multifactor ayuda a mantener las cuentas seguras aunque algún dato de la autenticación esté en riesgo. Mediante la supervisión de las cuentas que no tienen la autenticación multifactor habilitada puede identificar las cuentas en mayor o menor riesgo.

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción

## <a name="42-identification-and-authentication-organizational-users--network-access-to-non-privileged-accounts"></a>4.2 Identificación y autenticación (usuarios de la organización) | Acceso de red a cuentas sin privilegios

Este proyecto ayuda a restringir y controlar el acceso; para ello, asigna una definición de [Azure Policy](../../../policy/overview.md) para auditar las cuentas con permisos de lectura que no tengan habilitada la autenticación multifactor. La autenticación multifactor ayuda a mantener las cuentas seguras aunque algún dato de la autenticación esté en riesgo. Mediante la supervisión de las cuentas que no tienen la autenticación multifactor habilitada puede identificar las cuentas en mayor o menor riesgo.

- MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción

## <a name="23-and-41-authenticator-management"></a>2.3 y 4.1 Administración de autenticadores

Este proyecto asigna definiciones de [Azure Policy](../../../policy/overview.md) que auditan las máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña o que tengan permisos incorrectos establecidos en el archivo de contraseñas. Este proyecto también asigna definiciones de directiva que auditan la configuración del tipo de cifrado de contraseña en las máquinas virtuales Windows. La supervisión de estos indicadores le ayuda a asegurarse de que los autenticadores del sistema cumplen con la directiva de identificación y autenticación de la organización.

- mostrar los resultados de las auditorías de las máquinas virtuales Linux que no tengan los permisos del archivo de contraseñas establecidos en 0644
- implementación de los requisitos para auditar las máquinas virtuales Linux que no tengan los permisos del archivo de contraseñas establecidos en 0644
- mostrar los resultados de las auditorías de las máquinas virtuales Linux que tengan cuentas sin contraseña
- implementación de los requisitos para auditar las máquinas virtuales Linux que tengan cuentas sin contraseña
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que no almacenen contraseñas mediante el cifrado reversible
- implementación de los requisitos para auditar las máquinas virtuales Windows que no almacenen las contraseñas mediante el cifrado reversible

## <a name="23-and-41-authenticator-management--password-based-authentication"></a>2.3 y 4.1 Administración de autenticadores | Autenticación basada en contraseña

Este proyecto ayuda a exigir contraseñas seguras mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan las máquinas virtuales Windows que no exigen unos requisitos mínimos de seguridad de contraseña y de otros tipos. Conocer las máquinas virtuales que infringen la directiva de seguridad de las contraseñas ayuda a tomar medidas correctivas para garantizar que las contraseñas de todas las cuentas de usuario de máquina virtual cumplen la directiva de contraseñas de la organización.

- mostrar los resultados de las auditorías de las máquinas virtuales Windows que permitan volver a usar las 24 contraseñas anteriores
- mostrar los resultados de las auditorías de las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia máxima de 70 días
- mostrar los resultados de las auditorías de las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia mínima de 1 día
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que no tengan habilitada la configuración de complejidad de la contraseña
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas en 14 caracteres
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que no almacenen contraseñas mediante el cifrado reversible
- implementar los requisitos previos para auditar las máquinas virtuales Windows que permitan volver a usar las 24 contraseñas anteriores
- implementar los requisitos previos para auditar las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia máxima de 70 días
- implementar los requisitos previos para auditar las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia mínima de 1 día
- implementar los requisitos previos para auditar las máquinas virtuales que no tengan habilitada la configuración de complejidad de la contraseña
- implementar los requisitos previos para auditar las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas a 14 caracteres
- implementar los requisitos previos para auditar las máquinas virtuales Windows que no almacenen las contraseñas mediante el cifrado reversible

## <a name="22-and-27-vulnerability-scanning"></a>2.2 y 2.7 Examen de vulnerabilidades

Este proyecto ayuda a administrar las vulnerabilidades del sistema de información mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan las vulnerabilidades del sistema operativo, las de SQL o las de las máquinas virtuales en Azure Security Center. Azure Security Center proporciona funcionalidades de informes que permiten tener información en tiempo real sobre el estado de seguridad de los recursos de Azure implementados. Este proyecto asigna también definiciones de directiva que auditan y exigen Advanced Data Security en servidores SQL. Advanced Data Security incluía la evaluación de vulnerabilidades y funcionalidades de protección contra amenazas avanzada para ayudarle a entender las vulnerabilidades en los recursos implementados.

- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- La auditoría de SQL Server debe estar habilitada
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades de las bases de datos SQL 
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas

## <a name="13-denial-of-service-protection"></a>1.3 Protección ante la denegación de servicio

El nivel estándar de denegación de servicio distribuido (DDoS) de Azure proporciona características adicionales y funcionalidades de mitigación en el nivel de servicio básico. Estas características adicionales incluyen la integración de Azure Monitor y la capacidad de revisar los informes de mitigación posteriores al ataque. Este proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) que audita si está habilitado el nivel estándar de DDoS. Conocer la diferencia de capacidad entre los niveles de servicio puede ayudarle a seleccionar la mejor solución para abordar la denegación de protecciones de servicio para el entorno de Azure.

- Azure DDoS Protection Estándar debe estar habilitado.

## <a name="11-and-61-boundary-protection"></a>1.1 y 6.1 Protección de límites

Este proyecto le ayuda a administrar y controlar el límite del sistema mediante la asignación de una definición de [Azure Policy](../../../policy/overview.md) que supervisa las recomendaciones de protección del grupo de seguridad de red en Azure Security Center. Azure Security Center analiza los patrones de tráfico de las máquinas virtuales de Internet y proporciona recomendaciones de reglas de grupo de seguridad de red para reducir la superficie potencial de ataque.
Además, este proyecto también asigna definiciones de directiva que supervisan las cuentas de almacenamiento, las aplicaciones y los puntos de conexión desprotegidos. Los puntos de conexión y las aplicaciones que no están protegidos por firewall y las cuentas de almacenamiento con acceso sin restricciones pueden permitir el acceso no deseado a la información contenida en el sistema de información.

- Las recomendaciones de protección de red adaptable se deben aplicar en las máquinas virtuales accesibles desde Internet
- Debe restringirse el acceso a través de un punto de conexión accesible desde Internet
- Auditar el acceso de red sin restricciones a cuentas de almacenamiento

## <a name="29a-boundary-protection--access-points"></a>2.9A Protección de límites | Puntos de acceso

El acceso a máquinas virtuales Just-In-Time (JIT) bloquea el tráfico entrante a las máquinas virtuales de Azure, lo que reduce la exposición a ataques y, al mismo tiempo, se proporciona acceso sencillo para conectarse a las máquinas virtuales cuando sea necesario. El acceso a máquinas virtuales JIT le ayuda a limitar el número de conexiones externas a los recursos de Azure. Esta instancia de proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales que pueden admitir el acceso just-in-time pero aún no se han configurado.

- Los puertos de administración de las máquinas virtuales deben protegerse con el control de acceso de red Just-In-Time.

## <a name="29a-boundary-protection--external-telecommunications-services"></a>2.9A Protección de límites | Servicios de telecomunicaciones externos

El acceso a máquinas virtuales Just-In-Time (JIT) bloquea el tráfico entrante a las máquinas virtuales de Azure, lo que reduce la exposición a ataques y, al mismo tiempo, se proporciona acceso sencillo para conectarse a las máquinas virtuales cuando sea necesario. El acceso a máquinas virtuales JIT le ayuda a administrar las excepciones a la directiva de flujo de tráfico al facilitar los procesos de solicitud y aprobación de acceso. Esta instancia de proyecto asigna una definición de [Azure Policy](../../../policy/overview.md) que le ayuda a supervisar las máquinas virtuales que pueden admitir el acceso just-in-time pero aún no se han configurado.

- Los puertos de administración de las máquinas virtuales deben protegerse con el control de acceso de red Just-In-Time.

## <a name="21-24-24a-25a-and-26-transmission-confidentiality-and-integrity--cryptographic-or-alternate-physical-protection"></a>2.1, 2.4, 2.4A, 2.5A y 2.6 Integridad y confidencialidad de la transmisión | Protección criptográfica o protección física alternativa

Este proyecto le ayuda a proteger la confidencialidad y la integridad de la información transmitida mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que le facilitan la supervisión del mecanismo criptográfico implementado para los protocolos de comunicaciones. Asegurar que las comunicaciones están correctamente cifradas puede ayudarle a satisfacer los requisitos de su organización o a proteger la información frente a la divulgación y la modificación no autorizadas.

- Acceso a API App solo a través de HTTPS
- Mostrar los resultados de las auditorías de los servidores web de Windows que no estén usando los protocolos de comunicación segura
- Implementación de los requisitos previos para auditar los servidores web de Windows que no estén usando los protocolos de comunicación segura
- Acceso a Function App solo a través de HTTPS
- Solo se deben habilitar las conexiones seguras a Redis Cache
- Se debe habilitar la transferencia segura a las cuentas de almacenamiento
- Acceso a la aplicación web solo a través de HTTPS

## <a name="22-23-25-41-and-27-protection-of-information-at-rest--cryptographic-protection"></a>2.2, 2.3, 2.5, 4.1 y 2.7 Protección de la información en reposo | Protección criptográfica

Este proyecto ayuda a exigir la directiva sobre el uso de controles criptográficos para proteger la información en reposo mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que exigen controles criptográficos específicos y auditan el uso de configuraciones criptográficas poco seguras. Conocer dónde los recursos de Azure pueden tener configuraciones criptográficas subóptimas ayuda a tomar medidas correctivas para garantizar que los recursos se configuran conforme a la directiva de seguridad de la información. En concreto, las definiciones de directiva asignadas por este proyecto requieren cifrado para las cuentas de almacenamiento del lago de datos, requerir el cifrado de datos transparente en las bases de datos SQL y auditar si falta el cifrado en las bases de datos SQL, los discos de máquinas virtuales y las variables de cuenta de automatización.

- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Implementación de Advanced Data Security en los servidores de SQL Server
- Implementación del cifrado de datos transparente de SQL DB
- Se debe permitir el cifrado de datos transparente en bases de datos SQL

## <a name="13-22-and-27-flaw-remediation"></a>1.3, 2.2 y 2.7 Corrección de errores

Este proyecto ayuda a administrar los errores del sistema de información mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan si faltan actualizaciones del sistema o si existen vulnerabilidades del sistema operativo, de SQL o de las máquinas virtuales de Azure Security Center. Azure Security Center proporciona funcionalidades de informes que permiten tener información en tiempo real sobre el estado de seguridad de los recursos de Azure implementados. Este proyecto también asigna una definición de directiva que garantiza la aplicación automática de revisiones del sistema operativo para los conjuntos de escalado de máquinas virtuales.

- Exigir la aplicación automática de revisiones de imágenes del sistema operativo en Virtual Machine Scale Sets
- Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales
- Se deben instalar las actualizaciones del sistema en las máquinas virtuales
- Auditoría de la implementación de Dependency Agent en conjuntos de escalado de máquinas virtuales: la imagen (SO) de la máquina virtual no está en la lista
- Las variables de cuenta de automatización deben cifrarse
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades en la configuración de seguridad de las máquinas virtuales
- Se deben corregir las vulnerabilidades de las bases de datos SQL

## <a name="61-malicious-code-protection"></a>6.1 Protección frente a código malintencionado

Este proyecto le ayuda a administrar la protección de los puntos de conexión, incluida la protección del código malintencionado, mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan que no falte la protección de los puntos de conexión en las máquinas virtuales en Azure Security Center y aplican la solución antimalware de Microsoft en las máquinas virtuales Windows.

- Implementación de la extensión de Microsoft IaaSAntimalware predeterminada para Windows Server
- La solución de protección del punto de conexión debe instalarse en las máquinas virtuales
- Supervisión de la falta de Endpoint Protection en Azure Security Center
- Se deben migrar las cuentas de almacenamiento a los nuevos recursos de Azure Resource Manager

## <a name="61-malicious-code-protection--central-management"></a>6.1 Protección frente a código malintencionado | Administración central

Este proyecto le ayuda a administrar la protección de los puntos de conexión, incluida la protección del código malintencionado, mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan que no falte la protección de los puntos de conexión en las máquinas virtuales en Azure Security Center. Azure Security Center proporciona funcionalidades de informes y administración centralizada que permiten tener información en tiempo real sobre el estado de seguridad de los recursos de Azure implementados.

- La solución de protección del punto de conexión debe instalarse en las máquinas virtuales
- Supervisión de la falta de Endpoint Protection en Azure Security Center

## <a name="11-13-22-27-28-and-64-information-system-monitoring"></a>1.1, 1.3, 2.2, 2.7, 2.8 y 6.4 Supervisión del sistema de información

Este proyecto le ayuda a supervisar el sistema de auditoría y cumplimiento de seguridad de datos y registro en los recursos de Azure. En concreto, las directivas asignaban la auditoría e implementación del agente de Log Analytics, y mejoraban la configuración de seguridad de las bases de datos SQL, las cuentas de almacenamiento y los recursos de red. Estas funcionalidades pueden ayudarle a detectar los comportamientos anómalos y los indicadores de ataques para que pueda tomar las acciones adecuadas.

- Mostrar los resultados de la auditoría de las máquinas virtuales Windows en que el agente de Log Analytics no esté conectado según lo esperado
- Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Linux
- Implementar el agente de Log Analytics en máquinas virtuales Linux
- Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Windows
- Implementar el agente de Log Analytics en máquinas virtuales Windows
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- La configuración de seguridad de datos avanzada para SQL Server debe contener una dirección de correo electrónico para recibir alertas de seguridad
- Se deben habilitar los registros de diagnóstico en Azure Stream Analytics
- Implementación de Advanced Data Security en los servidores de SQL Server
- Implementación de auditorías en servidores SQL Server
- Implementar Network Watcher al crear redes virtuales.
- Implementación de la detección de amenazas en servidores SQL Server.

## <a name="22-and-28-information-system-monitoring--analyze-traffic--covert-exfiltration"></a>2.2 y 2.8 Supervisión del sistema de información | Análisis del tráfico o la exfiltración encubierta

Advanced Threat Protection para Azure Storage detecta los intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de almacenamiento o de vulnerarlas. Las alertas de protección incluyen patrones anómalos de acceso, cargas o extracciones anómalas, y actividad sospechosa de almacenamiento. Estos indicadores pueden ayudarle a detectar la exfiltración encubierta de información.

- Implementación de la detección de amenazas en servidores SQL Server.

> [!NOTE]
> La disponibilidad de definiciones específicas de Azure Policy puede variar tanto en Azure Government como en otras nubes nacionales.

## <a name="next-steps"></a>Pasos siguientes

Tras leer la información relativa a la asignación de controles del plano técnico SWIFT CSP-CSCF v2020, consulte los siguientes artículos para obtener información acerca del plano técnico y de cómo implementar este ejemplo:

> [!div class="nextstepaction"]
> [Plano técnico SWIFT CSP-CSCF v2020: introducción](./index.md)
> [Plano técnico SWIFT CSP-CSCF v2020: pasos de implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
