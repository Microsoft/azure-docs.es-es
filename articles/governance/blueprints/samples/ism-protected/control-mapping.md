---
title: Introducción al ejemplo del plano técnico Australian Government ISM PROTECTED
description: Introducción al ejemplo del plano técnico Australian Government ISM PROTECTED. Este ejemplo de plano técnico ayuda a los clientes a evaluar determinados controles de ISM PROTECTED.
ms.date: 08/03/2020
ms.topic: sample
ms.openlocfilehash: 750cc5551f0bbf05e0da6ad41d46729231243f72
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760437"
---
# <a name="control-mapping-of-the-australian-government-ism-protected-blueprint-sample"></a>Asignación de controles del ejemplo de plano técnico Australian Government ISM PROTECTED

En el artículo siguiente se detalla la correspondencia entre el ejemplo del plano técnico de Azure Blueprints Australian Government ISM PROTECTED y los controles de ISM PROTECTED. Para más información sobre los controles, consulte [ISM PROTECTED](https://www.cyber.gov.au/ism).

Las siguientes asignaciones se realiza a los controles de **ISM PROTECTED**. Use el panel de navegación de la derecha para ir directamente a una asignación de control específica. Muchos de los controles asignados se implementan con una iniciativa de [Azure Policy](../../../policy/overview.md). Para revisar la iniciativa completa, abra **Policy** en Azure Portal y seleccione la página **Definiciones**. Después, busque y seleccione la **\[versión preliminar\]: Auditar los controles protegidos del ISM del Gobierno de Australia e implementar extensiones de VM específicas para admitir los requisitos de auditoría**. Se trata de una iniciativa de directiva integrada.

> [!IMPORTANT]
> Cada control que se muestra a continuación está asociado a una o varias definiciones de [Azure Policy](../../../policy/overview.md). Estas directivas pueden ayudarle a [evaluar el cumplimiento](../../../policy/how-to/get-compliance-data.md) del control; sin embargo, a menudo no hay una coincidencia 1:1 o completa entre un control y una o varias directivas. Como tal, el **cumplimiento** con Azure Policy solo se refiere a las propias directivas; esto no garantiza que sea totalmente compatible con todos los requisitos de un control. Además, el estándar de cumplimiento incluye controles que no se abordan con las definiciones de Azure Policy en este momento. Por lo tanto, el cumplimiento en Azure Policy es solo una vista parcial del estado general de cumplimiento. Las asociaciones entre los controles y las definiciones de Azure Policy para este ejemplo de plano técnico de cumplimiento pueden cambiar con el tiempo.
> Para ver el historial de cambios, consulte el [historial de confirmación de GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/ism-protected/control-mapping.md).


## <a name="location-constraints"></a>Restricciones de ubicación

Este plano técnico le ayuda a restringir la ubicación de la implementación de todos los recursos y grupos de recursos a "Centro de Australia", "Centro de Australia2", "Este de Australia" y "Sudeste de Australia" mediante la asignación de las siguientes definiciones de Azure Policy:

- Ubicaciones permitidas (se ha codificado de forma rígida en "Centro de Australia", "Centro de Australia2", "Este de Australia" y "Sudeste de Australia")
- Ubicaciones permitidas para grupos de recursos (se ha codificado de forma rígida en "Centro de Australia", "Centro de Australia2", "Este de Australia" y "Sudeste de Australia")

## <a name="guidelines-for-personnel-security---access-to-systems-and-their-resources"></a>Instrucciones para la seguridad del personal: acceso a los sistemas y sus recursos

### <a name="0414-personnel-granted-access-to-a-system-and-its-resources-are-uniquely-identifiable"></a>0414 El personal al que se ha concedido acceso a un sistema y sus recursos se identifican de forma única

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción

### <a name="1503-standard-access-to-systems-applications-and-data-repositories-is-limited-to-that-required-for-personnel-to-undertake-their-duties"></a>1503 El acceso estándar a sistemas, aplicaciones y repositorios de datos se limita a aquel que sea necesario para que el personal pueda realizar sus tareas

- Debe designar un máximo de tres propietarios para la suscripción
- Debe haber más de un propietario asignado a su suscripción
- Mostrar los resultados de la auditoría de las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Implementar los requisitos previos para realizar una auditoría de las VM Windows en que el grupo Administradores contenga algún miembro especificado

### <a name="1507-privileged-access-to-systems-applications-and-data-repositories-is-validated-when-first-requested-and-revalidated-on-an-annual-or-more-frequent-basis"></a>1507 El acceso con privilegios a sistemas, aplicaciones y repositorios de datos se valida cuando se solicita por primera vez y se vuelve a validar con periodicidad anual o más frecuente

- Mostrar los resultados de la auditoría de las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Implementar los requisitos previos para realizar una auditoría de las VM Windows en que el grupo Administradores contenga algún miembro especificado

### <a name="1508-privileged-access-to-systems-applications-and-data-repositories-is-limited-to-that-required-for-personnel-to-undertake-their-duties"></a>1508 El acceso con privilegios a sistemas, aplicaciones y repositorios de datos se limita a aquel que sea necesario para que el personal pueda realizar sus tareas

- Debe designar un máximo de tres propietarios para la suscripción
- Debe haber más de un propietario asignado a su suscripción
- Mostrar los resultados de la auditoría de las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Implementar los requisitos previos para realizar una auditoría de las VM Windows en que el grupo Administradores contenga algún miembro especificado
- El control de acceso de red Just-In-Time se debe aplicar en las máquinas virtuales.

### <a name="0415-the-use-of-shared-user-accounts-is-strictly-controlled-and-personnel-using-such-accounts-are-uniquely-identifiable"></a>0415 El uso de cuentas de usuario compartidas está estrictamente controlado, y el personal que utiliza estas cuentas se identifica de forma única

- Mostrar los resultados de la auditoría de las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Implementar los requisitos previos para realizar una auditoría de las VM Windows en que el grupo Administradores contenga algún miembro especificado

### <a name="0445-privileged-users-are-assigned-a-dedicated-privileged-account-to-be-used-solely-for-tasks-requiring-privileged-access"></a>0445 A los usuarios con privilegios se les asigna una cuenta con privilegios dedicada que se usará únicamente para tareas que requieren acceso con privilegios

- Mostrar los resultados de la auditoría de las máquinas virtuales Windows en las que el grupo Administradores contenga algún miembro especificado
- Implementar los requisitos previos para realizar una auditoría de las VM Windows en que el grupo Administradores contenga algún miembro especificado

### <a name="0430-access-to-systems-applications-and-data-repositories-is-removed-or-suspended-on-the-same-day-personnel-no-longer-have-a-legitimate-requirement-for-access"></a>0430 El acceso a sistemas, aplicaciones y repositorios de datos se elimina o se suspende en el mismo día que el personal deja de tener un necesidad legítima de acceso

- Las cuentas en desuso deben quitarse de la suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción

### <a name="0441-when-personnel-are-granted-temporary-access-to-a-system-effective-security-controls-are-put-in-place-to-restrict-their-access-to-only-information-required-for-them-to-undertake-their-duties"></a>0441 Cuando al personal se le concede acceso temporal a un sistema, se establecen controles de seguridad eficaces para restringir su acceso únicamente a la información que necesitan para llevar a cabo sus tareas

- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de escritura deben quitarse de la suscripción
- Las cuentas en desuso deben quitarse de la suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción

## <a name="guidelines-for-system-hardening---operating-system-hardening"></a>Instrucciones para la segurización del sistema: segurización del sistema operativo

### <a name="1407-the-latest-version-n-or-n-1-version-of-an-operating-system-is-used-for-standard-operating-environments-soes"></a>1407 La versión más reciente (N), o la versión N-1, de un sistema operativo se usa para los entornos operativos estándar (SOE)

- Se deben instalar actualizaciones del sistema en las máquinas
- Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales

### <a name="0380-unneeded-operating-system-accounts-software-components-services-and-functionality-are-removed-or-disabled"></a>0380 Las cuentas de sistema operativo, software, componentes, servicios y funcionalidad que sean innecesarios se quitan o deshabilitan

- Las cuentas en desuso deben quitarse de la suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción

### <a name="1490-an-application-whitelisting-solution-is-implemented-on-all-servers-to-restrict-the-execution-of-executables-software-libraries-scripts-and-installers-to-an-approved-set"></a>1490 Se implementa una solución de listas blancas de aplicaciones en todos los servidores para restringir la ejecución de ejecutables, bibliotecas de software, scripts e instaladores a un conjunto aprobado

- Deben habilitarse los controles de aplicaciones adaptables en las máquinas virtuales

### <a name="1417-antivirus-software-is-implemented-on-workstations-and-servers-and-configured-with-signature-based-detection-enabled-and-set-to-a-high-level-heuristic-based-detection-enabled-and-set-to-a-high-level-detection-signatures-checked-for-currency-and-updated-on-at-least-a-daily-basis-automatic-and-regular-scanning-configured-for-all-fixed-disks-and-removable-media"></a>1417 El software antivirus se implementa en estaciones de trabajo y servidores y se configura con: detección basada en firma habilitada y establecida en un nivel alto, detección basada en heurística habilitada y establecida en un nivel alto, firmas de detección cuya aceptación y actualización se comprueban como mínimo diariamente, análisis automático y puntual configurado para todos los discos fijos y medios extraíbles

- La extensión IaaSAntimalware de Microsoft debe implementarse en servidores Windows.
- La solución de protección del punto de conexión debe instalarse en las máquinas virtuales
- Supervisión de la falta de Endpoint Protection en Azure Security Center

## <a name="guidelines-for-system-hardening---authentication-hardening"></a>Instrucciones para la segurización del sistema: segurización de la autenticación

### <a name="1546-users-are-authenticated-before-they-are-granted-access-to-a-system-and-its-resources"></a>1546 Se autentica a los usuarios antes de concederles acceso a un sistema y sus recursos

- Auditar el acceso de red sin restricciones a cuentas de almacenamiento
- Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente
- mostrar los resultados de las auditorías de máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña
- implementar los requisitos previos para auditar máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña
- mostrar los resultados de las auditorías de las máquinas virtuales Linux que tengan cuentas sin contraseña
- implementar los requisitos previos para realizar una auditoría de las máquinas virtuales Linux que tengan cuentas sin contraseña.

### <a name="0974-multi-factor-authentication-is-used-to-authenticate-standard-users"></a>0974 Se usa la autenticación multifactor para autenticar a los usuarios estándar

- MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción

### <a name="1173-multi-factor-authentication-is-used-to-authenticate-all-privileged-users-and-any-other-positions-of-trust"></a>1173 Se usa la autenticación multifactor para autenticar a todos los usuarios con privilegios y a cualquier otra posición de confianza

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción

### <a name="0421-passphrases-used-for-single-factor-authentication-are-a-minimum-of-14-characters-with-complexity-ideally-as-4-random-words"></a>0421 Las frases de contraseña usadas para la autenticación de un solo factor tienen un mínimo de 14 caracteres con complejidad, idealmente como 4 palabras aleatorias

- mostrar los resultados de la auditoría de las configuraciones de las máquinas virtuales Windows en "Configuración de seguridad: directivas de cuenta".
- implementar los requisitos previos para auditar las configuraciones de las máquinas virtuales Windows en "Configuración de seguridad: directivas de cuenta".

## <a name="guidelines-for-system-management---system-administration"></a>Instrucciones para la administración del sistema: administración del sistema

### <a name="1384-multi-factor-authentication-is-used-to-authenticate-users-each-time-they-perform-privileged-actions"></a>1384 Se usa la autenticación multifactor para autenticar a los usuarios cada vez que realizan acciones con privilegios

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción

### <a name="1386-management-traffic-is-only-allowed-to-originate-from-network-zones-that-are-used-to-administer-systems-and-applications"></a>1386 El tráfico de administración solo se puede originar en zonas de red que se usan para administrar sistemas y aplicaciones

- El control de acceso de red Just-In-Time se debe aplicar en las máquinas virtuales.
- Se debe desactivar la depuración remota para API Apps
- Se debe desactivar la depuración remota para Function App
- Se debe desactivar la depuración remota para las aplicaciones web

## <a name="guidelines-for-system-management---system-patching"></a>Instrucciones para la administración de sistemas: revisión de sistemas

### <a name="1144-security-vulnerabilities-in-applications-and-drivers-assessed-as-extreme-risk-are-patched-updated-or-mitigated-within-48-hours-of-the-security-vulnerabilities-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1144 Las vulnerabilidades de seguridad en las aplicaciones y los controladores que se han evaluado como de riesgo extremo se revisan, actualizan o mitigan en un plazo de 48 horas desde el momento en el que los proveedores, terceros independientes, administradores del sistema o usuarios identifican la vulnerabilidad de seguridad

- Se deben corregir las vulnerabilidades de las bases de datos SQL
- La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.
- La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.
- La valoración de vulnerabilidades debe habilitarse en las máquinas
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor
- La configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes

### <a name="0940-security-vulnerabilities-in-applications-and-drivers-assessed-as-high-risk-are-patched-updated-or-mitigated-within-two-weeks-of-the-security-vulnerability-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>0940 Las vulnerabilidades de seguridad en las aplicaciones y los controladores que se han evaluado como de alto riesgo se revisan, actualizan o mitigan en un plazo de dos semanas desde el momento en el que los proveedores, terceros independientes, administradores del sistema o usuarios identifican la vulnerabilidad de seguridad

- Se deben corregir las vulnerabilidades de las bases de datos SQL
- La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.
- La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.
- La evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales.
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor
- La configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes

### <a name="1472-security-vulnerabilities-in-applications-and-drivers-assessed-as-moderate-or-low-risk-are-patched-updated-or-mitigated-within-one-month-of-the-security-vulnerability-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1472 Las vulnerabilidades de seguridad en las aplicaciones y los controladores que se han evaluado como de riesgo bajo o moderado se revisan, actualizan o mitigan en un plazo de un mes desde el momento en el que los proveedores, terceros independientes, administradores del sistema o usuarios identifican la vulnerabilidad de seguridad

- Se deben corregir las vulnerabilidades de las bases de datos SQL
- La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.
- La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.
- La evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales.
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor
- La configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes

### <a name="1494-security-vulnerabilities-in-operating-systems-and-firmware-assessed-as-extreme-risk-are-patched-updated-or-mitigated-within-48-hours-of-the-security-vulnerabilities-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1494 Las vulnerabilidades de seguridad en los sistemas operativos y el firmware que se han evaluado como de riesgo extremo se revisan, actualizan o mitigan en un plazo de 48 horas desde el momento en el que los proveedores, terceros independientes, administradores del sistema o usuarios identifican la vulnerabilidad de seguridad

- Se deben corregir las vulnerabilidades de las bases de datos SQL
- La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.
- La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.
- La evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales.
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor
- La configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes

### <a name="1495-security-vulnerabilities-in-operating-systems-and-firmware-assessed-as-high-risk-are-patched-updated-or-mitigated-within-two-weeks-of-the-security-vulnerability-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1495 Las vulnerabilidades de seguridad en los sistemas operativos y el firmware que se han evaluado como de alto riesgo se revisan, actualizan o mitigan en un plazo de dos semanas desde el momento en el que los proveedores, terceros independientes, administradores del sistema o usuarios identifican la vulnerabilidad de seguridad

- Se deben corregir las vulnerabilidades de las bases de datos SQL
- La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.
- La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.
- La evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales.
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor
- La configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes

### <a name="1496-security-vulnerabilities-in-operating-systems-and-firmware-assessed-as-moderate-or-low-risk-are-patched-updated-or-mitigated-within-one-month-of-the-security-vulnerability-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1496 Las vulnerabilidades de seguridad en los sistemas operativos y el firmware que se han evaluado como de riesgo moderado o bajo se revisan, actualizan o mitigan en un plazo de un mes desde el momento en el que los proveedores, terceros independientes, administradores del sistema o usuarios identifican la vulnerabilidad de seguridad

- Se deben corregir las vulnerabilidades de las bases de datos SQL
- La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.
- La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.
- La evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales.
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor
- La configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes

## <a name="guidelines-for-system-management---data-backup-and-restoration"></a>Instrucciones para la administración del sistema: copia de seguridad y restauración de datos

### <a name="1511-backups-of-important-information-software-and-configuration-settings-are-performed-at-least-daily"></a>1511 Las copias de seguridad de la información importante, el software y los valores de configuración se realizan por lo menos una vez al día

- Auditoría de máquinas virtuales sin la recuperación ante desastres configurada

## <a name="guidelines-for-system-monitoring---event-logging-and-auditing"></a>Instrucciones para la supervisión del sistema: registro de eventos y auditoría

### <a name="1405-a-centralised-logging-facility-is-implemented-and-systems-are-configured-to-save-event-logs-to-the-centralised-logging-facility-as-soon-as-possible-after-each-event-occurs"></a>1405 Se implementa una instalación de registro centralizado y los sistemas se configuran para guardar registros de eventos en la instalación de registro centralizada lo antes posible desde el momento en el que se produce cada evento

- Las suscripciones a Azure deben tener un perfil de registro para el registro de actividad

### <a name="0582-the-following-events-are-logged-for-operating-systems-access-to-important-data-and-processes-application-crashes-and-any-error-messages-attempts-to-use-special-privileges-changes-to-accounts-changes-to-security-policy-changes-to-system-configurations-domain-name-system-dns-and-hypertext-transfer-protocol-http-requests-failed-attempts-to-access-data-and-system-resources-service-failures-and-restarts-system-startup-and-shutdown-transfer-of-data-to-external-media-user-or-group-management-use-of-special-privileges"></a>0582 Para los sistemas operativos se registran los siguientes eventos: acceso a datos y procesos importantes, bloqueos de aplicaciones y mensajes de error, intentos de uso de privilegios especiales, cambios en las cuentas, cambios en la directiva de seguridad, cambios en las configuraciones del sistema, solicitudes de sistema de nombres de dominio (DNS) y protocolo de transferencia de hipertexto (HTTP), intentos fallidos de acceso a datos y recursos del sistema, errores y reinicios de servicio, inicio y apagado del sistema, transferencia de datos a medios externos, administración de usuarios o grupos, uso de privilegios especiales

- \[Versión preliminar\]: Auditoría de la implementación del agente de Log Analytics: la imagen de la VM (SO) no está en la lista
- Auditoría de la implementación del agente de Log Analytics en VMSS: la imagen de la VM (SO) no está en la lista
- Auditar área de trabajo de Log Analytics en la máquina virtual: error de coincidencia del informe
- Auditar la configuración de diagnóstico

### <a name="1537-the-following-events-are-logged-for-databases-access-to-particularly-important-information-addition-of-new-users-especially-privileged-users-any-query-containing-comments-any-query-containing-multiple-embedded-queries-any-query-or-database-alerts-or-failures-attempts-to-elevate-privileges-attempted-access-that-is-successful-or-unsuccessful-changes-to-the-database-structure-changes-to-user-roles-or-database-permissions-database-administrator-actions-database-logons-and-logoffs-modifications-to-data-use-of-executable-commands"></a>1537 Para las bases de datos se registran los siguientes eventos: acceso a información especialmente importante, incorporación de nuevos usuarios, usuarios con privilegios especiales, cualquier consulta que contenga comentarios, cualquier consulta que contenga varias consultas insertadas, cualquier alerta o error de base de datos o de consulta, intentos de aumentar los privilegios, intentos de acceso tengan o no éxito, cambios en la estructura de la base de datos, cambios en los roles de usuario o permisos de base de datos, acciones de administrador de base de datos, inicios y cierres de sesión de base de datos, modificaciones de datos, uso de comandos ejecutables

- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server
- Auditar la configuración de diagnóstico
- La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL

## <a name="guidelines-for-system-monitoring---vulnerability-management"></a>Instrucciones para la supervisión del sistema: administración de vulnerabilidades

### <a name="0911-vulnerability-assessments-and-penetration-tests-are-conducted-by-suitably-skilled-personnel-before-a-system-is-deployed-after-a-significant-change-to-a-system-and-at-least-annually-or-as-specified-by-the-system-owner"></a>0911 Las evaluaciones de vulnerabilidad y las pruebas de penetración las llevan a cabo personal cualificado de manera adecuada antes de la implementación de un sistema, después de un cambio significativo en un sistema, y al menos anualmente o según lo especificado por el propietario del sistema

- Se deben corregir las vulnerabilidades de las bases de datos SQL
- La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.
- La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.
- La evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales.
- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor

## <a name="guidelines-for-database-systems-management---database-servers"></a>Instrucciones para la administración de sistemas de base de datos: servidores de bases de datos

### <a name="1425-hard-disks-of-database-servers-are-encrypted-using-full-disk-encryption"></a>1425 Los discos duros de los servidores de bases de datos se cifran mediante el cifrado de disco completo

- El cifrado de discos debe aplicarse en máquinas virtuales
- Se debe permitir el cifrado de datos transparente en bases de datos SQL

### <a name="1277-information-communicated-between-database-servers-and-web-applications-is-encrypted"></a>1277 La información comunicada entre los servidores de bases de datos y las aplicaciones web está cifrada

- Solo se deben habilitar las conexiones seguras a Redis Cache
- Se debe habilitar la transferencia segura a las cuentas de almacenamiento
- Mostrar los resultados de las auditorías de los servidores web de Windows que no estén usando los protocolos de comunicación segura
- Implementación de los requisitos previos para auditar los servidores web de Windows que no estén usando los protocolos de comunicación segura

## <a name="guidelines-for-database-systems-management---database-management-system-software"></a>Instrucciones para administración de sistemas de base de datos: software del sistema de administración de bases de datos

### <a name="1260-default-database-administrator-accounts-are-disabled-renamed-or-have-their-passphrases-changed"></a>1260 Las cuentas de administrador de bases de datos predeterminadas se deshabilitan, se cambian de nombre o se cambian sus frases de contraseña

- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server

### <a name="1262-database-administrators-have-unique-and-identifiable-accounts"></a>1262 Los administradores de base de datos tienen cuentas únicas e identificables

- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server

### <a name="1261-database-administrator-accounts-are-not-shared-across-different-databases"></a>1261 Las cuentas de administrador de base de datos no se comparten entre diferentes bases de datos

- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server

### <a name="1263-database-administrator-accounts-are-used-exclusively-for-administrative-tasks-with-standard-database-accounts-used-for-general-purpose-interactions-with-database"></a>1263 Las cuentas de administrador de base de datos se utilizan exclusivamente para tareas administrativas, y las cuentas de base de datos estándar se emplean para las interacciones de uso general con la base de datos

- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server

### <a name="1264-database-administrator-access-is-restricted-to-defined-roles-rather-than-accounts-with-default-administrative-permissions-or-all-permissions"></a>1264 El acceso de administrador de base de datos está restringido a roles definidos en lugar de cuentas con permisos administrativos predeterminados o todos los permisos

- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server

## <a name="guidelines-for-using-cryptography---cryptographic-fundamentals"></a>Instrucciones para el uso de criptografía: fundamentos criptográficos

### <a name="0459-encryption-software-used-for-data-at-rest-implements-full-disk-encryption-or-partial-encryption-where-access-controls-will-only-allow-writing-to-the-encrypted-partition"></a>0459 El software de cifrado que se usa para los datos en reposo implementa el cifrado de disco completo o el cifrado parcial, donde los controles de acceso solo permitirán escribir en la partición cifrada

- El cifrado de discos debe aplicarse en máquinas virtuales

## <a name="guidelines-for-using-cryptography---transport-layer-security"></a>Instrucciones para el uso de criptografía: seguridad de la capa de transporte

### <a name="1139-only-the-latest-version-of-tls-is-used"></a>1139 Solo se usa la versión más reciente de TLS

- La versión más reciente de TLS debe usarse en la aplicación de API.
- La versión más reciente de TLS debe usarse en la aplicación web.
- La versión más reciente de TLS debe usarse en la aplicación de funciones.
- Implementación de los requisitos previos para auditar los servidores web de Windows que no estén usando los protocolos de comunicación segura
- Mostrar los resultados de las auditorías de los servidores web de Windows que no estén usando los protocolos de comunicación segura

## <a name="guidelines-for-data-transfers-and-content-filtering---content-filtering"></a>Instrucciones para las transferencias de datos y el filtrado de contenido: filtrado de contenido

### <a name="1288-antivirus-scanning-using-multiple-different-scanning-engines-is-performed-on-all-content"></a>1288 El examen del antivirus, con varios motores de análisis diferentes, se realiza en todo el contenido

- La extensión IaaSAntimalware de Microsoft debe implementarse en servidores Windows.
- La solución de protección del punto de conexión debe instalarse en las máquinas virtuales
- Supervisión de la falta de Endpoint Protection en Azure Security Center

## <a name="guidelines-for-data-transfers-and-content-filtering---web-application-development"></a>Instrucciones para las transferencias de datos y el filtrado de contenido: desarrollo de aplicaciones web

### <a name="1552-all-web-application-content-is-offered-exclusively-using-https"></a>1552 Todo el contenido de la aplicación web se ofrece exclusivamente mediante HTTPS

- Acceso a Function App solo a través de HTTPS
- Acceso a API App solo a través de HTTPS
- Acceso a la aplicación web solo a través de HTTPS
- Solo se deben habilitar las conexiones seguras a Redis Cache

### <a name="1424-web-browser-based-security-controls-are-implemented-for-web-applications-in-order-to-help-protect-both-web-applications-and-their-users"></a>1424 Los controles de seguridad basados en explorador web se implementan para las aplicaciones web con el fin de ayudar a proteger las mismas y a sus usuarios

- CORS no debe permitir que todos los recursos accedan a las aplicaciones web

## <a name="guidelines-for-network-management---network-design-and-configuration"></a>Instrucciones para la administración de redes: diseño y configuración de red

### <a name="0520-network-access-controls-are-implemented-on-networks-to-prevent-the-connection-of-unauthorised-network-devices"></a>0520 Los controles de acceso a la red se implementan en redes para evitar la conexión de dispositivos de red no autorizados

- Auditar el acceso de red sin restricciones a cuentas de almacenamiento

### <a name="1182-network-access-controls-are-implemented-to-limit-traffic-within-and-between-network-segments-to-only-those-that-are-required-for-business-purposes"></a>1182 Los controles de acceso a la red se implementan para limitar el tráfico dentro y entre segmentos de red solo a aquellos necesarios para fines empresariales

- Las máquinas virtuales accesibles desde Internet deben estar protegidas con grupos de seguridad de red
- Auditar el acceso de red sin restricciones a cuentas de almacenamiento
- Las recomendaciones de protección de red adaptable se deben aplicar en las máquinas virtuales accesibles desde Internet

## <a name="guidelines-for-network-management---service-continuity-for-online-services"></a>Instrucciones para la administración de redes: continuidad del servicio para servicios en línea

### <a name="1431-denial-of-service-attack-prevention-and-mitigation-strategies-are-discussed-with-service-providers-specifically-their-capacity-to-withstand-denial-of-service-attacks-any-costs-likely-to-be-incurred-by-customers-resulting-from-denial-of-service-attacks-thresholds-for-notifying-customers-or-turning-off-their-online-services-during-denial-of-service-attacks-pre-approved-actions-that-can-be-undertaken-during-denial-of-service-attacks-denial-of-service-attack-prevention-arrangements-with-upstream-providers-to-block-malicious-traffic-as-far-upstream-as-possible"></a>1431 La prevención de ataques de denegación de servicio y las estrategias de mitigación se tratan con los proveedores de servicios, concretamente: su capacidad para resistir los ataques por denegación de servicio, costos en los que puedan incurrir los clientes como resultado de ataques por denegación de servicio, umbrales para notificar a los clientes o desactivar sus servicios en línea durante los ataques por denegación de servicio, acciones previamente aprobadas que se pueden realizar durante los ataques por denegación de servicio, disposiciones de prevención de ataques por denegación de servicio con proveedores ascendentes para bloquear el tráfico malintencionado en la fase más temprana posible

- Se debe habilitar DDoS Protection estándar


> [!NOTE]
> La disponibilidad de definiciones específicas de Azure Policy puede variar tanto en Azure Government como en otras nubes nacionales. 

## <a name="next-steps"></a>Pasos siguientes

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
