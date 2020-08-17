---
title: Controles del ejemplo de plano técnico de UK OFFICIAL y UK NHS
description: Asignación de controles de los ejemplos de planos técnicos de UK OFFICIAL y UK NHS. Cada control se asigna a una o varias directivas de Azure que ayudan en la evaluación.
ms.date: 07/13/2020
ms.topic: sample
ms.openlocfilehash: 360c6bc2e1f207ca2d4470ffe62a81f524ce2a73
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078994"
---
# <a name="control-mapping-of-the-uk-official-and-uk-nhs-blueprint-samples"></a>Asignación de controles de los ejemplos de planos técnicos de UK OFFICIAL y UK NHS.

En el artículo siguiente se detalla cómo se asignan los ejemplos de planos técnicos de UK OFFICIAL y UK NHS a los controles UK OFFICIAL y UK NHS. Para obtener más información sobre los controles, consulte [UK OFFICIAL](https://www.gov.uk/government/publications/government-security-classifications).

Las asignaciones siguientes son para los controles **UK OFFICIAL** y **UK NHS**. Use el panel de navegación de la derecha para ir directamente a una asignación de control específica. Muchos de los controles asignados se implementan con una iniciativa de [Azure Policy](../../../policy/overview.md). Para revisar la iniciativa completa, abra **Policy** en Azure Portal y seleccione la página **Definiciones**. Después, busque y seleccione la iniciativa de directiva integrada **\[Versión preliminar\] Auditar los controles UK OFFICIAL y UK NHS e implementar extensiones de VM específicas para cumplir con los requisitos de la auditoría**.

> [!IMPORTANT]
> Cada control que se muestra a continuación está asociado a una o varias definiciones de [Azure Policy](../../../policy/overview.md). Estas directivas pueden ayudarle a [evaluar el cumplimiento](../../../policy/how-to/get-compliance-data.md) del control; sin embargo, a menudo no hay una coincidencia 1:1 o completa entre un control y una o varias directivas. Como tal, el **cumplimiento** con Azure Policy solo se refiere a las propias directivas; esto no garantiza que sea totalmente compatible con todos los requisitos de un control. Además, el estándar de cumplimiento incluye controles que no se abordan con las definiciones de Azure Policy en este momento. Por lo tanto, el cumplimiento en Azure Policy es solo una vista parcial del estado general de cumplimiento. Las asociaciones entre los controles y las definiciones de Azure Policy para este ejemplo de plano técnico de cumplimiento pueden cambiar con el tiempo. Para ver el historial de cambios, consulte el [historial de confirmación de GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/ukofficial/control-mapping.md).

## <a name="1-data-in-transit-protection"></a>1\. Protección de datos en tránsito

El plano técnico ayuda a garantizar que la transferencia de información con los servicios de Azure es segura. Para ello, asigna definiciones de [Azure Policy](../../../policy/overview.md) para auditar las conexiones no seguras a las cuentas de almacenamiento y a Redis Cache.

- Solo se deben habilitar las conexiones seguras a Redis Cache
- Se debe habilitar la transferencia segura a las cuentas de almacenamiento
- Mostrar los resultados de las auditorías de los servidores web de Windows que no estén usando los protocolos de comunicación segura
- Implementación de los requisitos previos para auditar los servidores web de Windows que no estén usando los protocolos de comunicación segura
- Acceso a la aplicación web solo a través de HTTPS
- Acceso a Function App solo a través de HTTPS
- Acceso a API App solo a través de HTTPS

## <a name="23-data-at-rest-protection"></a>2.3. Protección de los datos en reposo

Este plano técnico ayuda a instaurar la directiva sobre el uso de controles criptográficos mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que instauran controles criptográficos específicos y auditan el uso de configuraciones criptográficas poco seguras.
Conocer dónde los recursos de Azure pueden tener configuraciones criptográficas subóptimas ayuda a tomar medidas correctivas para garantizar que los recursos se configuran conforme a la directiva de seguridad de la información. En concreto, las directivas asignadas por este plano técnico requieren cifrado para las cuentas de Data Lake Storage y cifrado de datos transparente en las bases de datos SQL; auditan el cifrado que falta en las cuentas de almacenamiento, las bases de datos SQL, los discos de máquina virtual y las variables de cuenta de Automation; auditan las conexiones no seguras a cuentas de almacenamiento y Redis Cache; y auditan el cifrado de contraseñas no seguras de las máquinas virtuales, así como la comunicación sin cifrado de Service Fabric.

- El cifrado de discos debe aplicarse en máquinas virtuales
- Las variables de cuenta de automatización deben cifrarse
- Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric
- Se debe permitir el cifrado de datos transparente en bases de datos SQL
- Implementación del cifrado de datos transparente de SQL DB
- Exigencia de cifrado en las cuentas de Data Lake Store.
- Ubicaciones permitidas (codificadas de forma rígida como "UK SOUTH" y "UK WEST").
- Ubicaciones permitidas para grupos de recursos (codificadas de forma rígida como "UK SOUTH" y "UK WEST").

## <a name="52-vulnerability-management"></a>5.2. Administración de vulnerabilidades

Este plano técnico ayuda a administrar las vulnerabilidades del sistema de información al asignar definiciones de [Azure Policy](../../../policy/overview.md) que supervisan si falta la protección del punto de conexión, si faltan actualizaciones del sistema o si existen vulnerabilidades del sistema operativo, de SQL o de las máquinas virtuales. Esta información proporciona detalles en tiempo real sobre el estado de seguridad de los recursos implementados y ayuda a priorizar las medidas correctivas.

- Supervisión de la falta de Endpoint Protection en Azure Security Center
- Se deben instalar actualizaciones del sistema en las máquinas
- Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades de las bases de datos SQL
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades
- La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.
- La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server

## <a name="53-protective-monitoring"></a>5.3. Supervisión de protección

Este plano técnico ayuda a proteger los recursos del sistema de información mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que proporcionan supervisión de protección para el acceso sin restricciones, la actividad de la lista de permitidos y las amenazas.

- Auditar el acceso de red sin restricciones a cuentas de almacenamiento
- Deben habilitarse los controles de aplicaciones adaptables en las máquinas virtuales
- Auditoría de máquinas virtuales sin la recuperación ante desastres configurada
- Se debe habilitar DDoS Protection estándar
- Los tipos de Advanced Threat Protection deben estar establecidos en "Todos" en la configuración de Advanced Data Security de la instancia administrada de SQL Server
- Los tipos de Advanced Threat Protection deben estar establecidos en "Todos" en la configuración de Advanced Data Security de SQL Server
- Implementación de la detección de amenazas en servidores SQL Server.
- Implementación de la extensión de Microsoft IaaSAntimalware predeterminada para Windows Server

## <a name="9-secure-user-management"></a>9 Administración segura de usuarios 

Azure implementa el control de acceso basado en rol (RBAC) para ayudar a administrar quién tiene acceso a los recursos de Azure. Mediante Azure Portal, puede revisar quién tiene acceso a los recursos de Azure y sus permisos. Este plano técnico ayuda a restringir y controlar los derechos de acceso mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas externas con permisos de propietario y de lectura/escritura y las cuentas con permisos de propietario y de lectura/escritura que no tienen habilitada la autenticación multifactor.

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de escritura deben quitarse de la suscripción
- Las cuentas externas con permisos de lectura deben quitarse de la suscripción

## <a name="10-identity-and-authentication"></a>10 Identidad y autenticación

Este plano técnico ayuda a restringir y controlar los derechos de acceso mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas externas con permisos de propietario y de lectura/escritura y las cuentas con permisos de propietario y de lectura/escritura que no tienen habilitada la autenticación multifactor.

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de escritura deben quitarse de la suscripción
- Las cuentas externas con permisos de lectura deben quitarse de la suscripción

Este plano técnico asigna definiciones de Azure Policy para auditar el uso de la autenticación de Azure Active Directory para servidores SQL Server y Service Fabric. El uso de la autenticación de Azure Active Directory simplifica la administración de permisos y centraliza la administración de identidades de usuarios de base de datos y otros servicios de Microsoft.

- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server
- Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente

Este plano técnico también asigna definiciones de Azure Policy para auditar las cuentas que deberían priorizarse para la revisión, incluidas las cuentas en desuso y externas. Cuando sea necesario, se puede bloquear el inicio de sesión de las cuentas (o eliminar estas), lo cual eliminaría inmediatamente los derechos de acceso a los recursos de Azure. Este plano técnico asigna dos definiciones de Azure Policy para auditar las cuentas en desuso cuya eliminación debería considerarse.

- Las cuentas en desuso deben quitarse de la suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de escritura deben quitarse de la suscripción

Este plano técnico también asigna una definición de Azure Policy que audita los permisos de los archivos con contraseña de máquina virtual Linux y alerta si la configuración es incorrecta. Este diseño permite tomar medidas correctivas para garantizar que los autenticadores no están en riesgo.

- mostrar los resultados de las auditorías de las máquinas virtuales Linux que no tengan los permisos del archivo de contraseñas establecidos en 0644

Este plano técnico ayuda a instaurar contraseñas seguras mediante la asignación de definiciones de Azure Policy que auditan las máquinas virtuales Windows que no tienen unos requisitos mínimos de seguridad de contraseña y de otros tipos. Conocer las máquinas virtuales que infringen la directiva de seguridad de las contraseñas ayuda a tomar medidas correctivas para garantizar que las contraseñas de todas las cuentas de usuario de máquina virtual cumplen la directiva.

- implementar los requisitos previos para auditar las máquinas virtuales que no tengan habilitada la configuración de complejidad de la contraseña
- implementar los requisitos previos para auditar las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia máxima de 70 días
- implementar los requisitos previos para auditar las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia mínima de 1 día
- implementar los requisitos previos para auditar las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas a 14 caracteres
- implementar los requisitos previos para auditar las máquinas virtuales Windows que permitan volver a usar las 24 contraseñas anteriores
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que no tengan habilitada la configuración de complejidad de la contraseña
- mostrar los resultados de las auditorías de las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia máxima de 70 días
- mostrar los resultados de las auditorías de las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia mínima de 1 día
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas en 14 caracteres
- mostrar los resultados de las auditorías de las máquinas virtuales Windows que permitan volver a usar las 24 contraseñas anteriores

Este plano técnico también ayuda a controlar el acceso a los recursos de Azure mediante la asignación de definiciones de Azure Policy. Estas directivas auditan el uso de los tipos de recursos y las configuraciones que pueden permitir un acceso más flexible a los recursos. Conocer los recursos que infringen estas directivas ayuda a tomar medidas correctivas para garantizar que el acceso a los recursos de Azure se limita a los usuarios autorizados.

- implementación de los requisitos para auditar las máquinas virtuales Linux que tengan cuentas sin contraseña
- implementación de los requisitos para auditar máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña
- mostrar los resultados de las auditorías de las máquinas virtuales Linux que tengan cuentas sin contraseña
- mostrar los resultados de las auditorías de máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña
- Se deben migrar las cuentas de almacenamiento a los nuevos recursos de Azure Resource Manager
- Se deben migrar las máquinas virtuales a nuevos recursos de Azure Resource Manager
- Auditar las máquinas virtuales que no utilizan discos administrados

## <a name="11-external-interface-protection"></a>11. Protección de la interfaz externa

Además de usar más de 25 directivas para garantizar una administración segura de usuarios adecuada, este plano técnico ayuda a proteger las interfaces de servicio contra el acceso no autorizado mediante la asignación de una definición de [Azure Policy](../../../policy/overview.md) que supervisa las cuentas de almacenamiento sin restricciones. Las cuentas de almacenamiento con acceso sin restricciones pueden permitir el acceso no deseado a la información contenida en el sistema de información. Este plano técnico también asigna una directiva que habilita controles de aplicación adaptables en máquinas virtuales.

- Auditar el acceso de red sin restricciones a cuentas de almacenamiento
- Deben habilitarse los controles de aplicaciones adaptables en las máquinas virtuales
- Debe restringirse el acceso a través de un punto de conexión accesible desde Internet
- Las recomendaciones de protección de red adaptable se deben aplicar en las máquinas virtuales accesibles desde Internet
- La solución de protección del punto de conexión debe instalarse en las máquinas virtuales
- El control de acceso de red Just-In-Time se debe aplicar en las máquinas virtuales.
- Recomendación de desactivación de la depuración remota para Function App
- Recomendación de desactivación de la depuración remota para aplicaciones web
- Se debe desactivar la depuración remota para aplicaciones de API

## <a name="13-audit-information-for-users"></a>13. Información de auditoría para los usuarios

Este plano técnico ayuda a garantizar que los eventos del sistema se registran mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan las configuraciones de registro de los recursos de Azure. Una directiva asignada también audita si las máquinas virtuales no envían registros a un área de trabajo específica de Log Analytics.

- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Auditar la configuración de diagnóstico
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux
- \[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows
- Implementar Network Watcher al crear redes virtuales.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado la asignación de controles de los planos técnicos UK OFFICIAL y UK NHS, visite los siguientes artículos para obtener información sobre la arquitectura y cómo implementar este ejemplo:

> [!div class="nextstepaction"]
> [Planos técnicos de UK OFFICIAL y UK NHS: introducción](./index.md)
> [Planos técnicos de UK OFFICIAL y UK NHS: pasos de implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
