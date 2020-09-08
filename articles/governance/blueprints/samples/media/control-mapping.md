---
title: Controles de ejemplo de planos técnicos de Media
description: Asignación de controles de los ejemplos de plano técnico de Media. Cada control se asigna a una o varias directivas de Azure que ayudan en la evaluación.
ms.date: 08/13/2020
ms.topic: sample
ms.openlocfilehash: 4101b9030425c22b8806870d467581521870d639
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88208460"
---
# <a name="control-mapping-of-the-media-blueprint-sample"></a>Asignación de controles del ejemplo de plano técnico de Media

En el siguiente artículo se detalla cómo se asigna el ejemplo de plano técnico según la norma Azure Blueprints Media a los controles de Media. Para más información sobre los controles, consulte [Media](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html).

Las siguientes asignaciones son para los controles de **Media**. Use el panel de navegación de la derecha para ir directamente a una asignación de control específica. Muchos de los controles asignados se implementan con una iniciativa de [Azure Policy](../../../policy/overview.md). Para revisar la iniciativa completa, abra **Policy** en Azure Portal y seleccione la página **Definiciones**. Después, busque y seleccione la **\[versión preliminar\]: Auditoría de los controles de Media**, que es una iniciativa de directiva integrada.

> [!IMPORTANT]
> Cada control que se muestra a continuación está asociado a una o varias definiciones de [Azure Policy](../../../policy/overview.md). Estas directivas pueden ayudarle a [evaluar el cumplimiento](../../../policy/how-to/get-compliance-data.md) del control; sin embargo, a menudo no hay una coincidencia 1:1 o completa entre un control y una o varias directivas. Como tal, el **cumplimiento** con Azure Policy solo se refiere a las propias directivas; esto no garantiza que sea totalmente compatible con todos los requisitos de un control. Además, el estándar de cumplimiento incluye controles que no se abordan con las definiciones de Azure Policy en este momento. Por lo tanto, el cumplimiento en Azure Policy es solo una vista parcial del estado general de cumplimiento. Las asociaciones entre los controles y las definiciones de Azure Policy para este ejemplo de plano técnico de cumplimiento pueden cambiar con el tiempo. Para ver el historial de cambios, consulte el [historial de confirmación de GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/media/control-mapping.md).

## <a name="access-control"></a>Control de acceso

### <a name="ac-11--ensure-no-root-access-key-exists"></a>AC-1.1: asegúrese de que no existe ninguna clave de acceso raíz

- \[Versión preliminar\]: se implementan los requisitos previos para realizar una auditoría de las VM Windows que no contengan los certificados especificados en la raíz de confianza

### <a name="ac-12---passwords-pins-and-tokens-must-be-protected"></a>AC-1.2: se deben proteger las contraseñas, los PIN y los tokens

- \[Versión preliminar\]: implementar los requisitos previos para auditar las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas a 14 caracteres

### <a name="ac-18---shared-account-access-is-prohibited"></a>AC-1.8: se prohíbe el acceso a las cuentas compartidas

- Todas las reglas de autorización excepto RootManageSharedAccessKey se deben eliminar del espacio de nombres de Service Bus

### <a name="ac-19--system-must-restrict-access-to-authorized-users"></a>AC-1.9: el sistema debe restringir el acceso a los usuarios autorizados

- Auditar el acceso de red sin restricciones a cuentas de almacenamiento

### <a name="ac--114--system-must-enforce-access-rights"></a>AC-1.14: el sistema debe aplicar derechos de acceso

- \[Versión preliminar\]: se implementan los requisitos previos para auditar las configuraciones de las VM Windows en "Asignación de derechos de usuario"

### <a name="ac--115--prevent-unauthorized-access-to-security-relevant-information-or-functions"></a>AC-1.15: evite el acceso no autorizado a las funciones o información pertinentes para la seguridad

- \[Versión preliminar\]: se muestran los resultados de la auditoría de las configuraciones de las VM Windows en "Opciones de seguridad: Configuración del sistema"

### <a name="ac-1-21---separation-of-duties-must-be-enforced-through-appropriate-assignment-of-role"></a>AC-1-21: la separación de deberes debe aplicarse mediante la asignación adecuada de rol

- [Versión preliminar\]: En Kubernetes Services se debe usar el control de acceso basado en rol (RBAC)

### <a name="ac-140--ensure-that-systems-are-not-connecting-trusted-network-and-untrusted-networks-at-the-same-time"></a>AC-1.40: asegúrese de que los sistemas no se conectan a la red de confianza y a las redes que no son de confianza al mismo tiempo

- \[Versión preliminar\]: se implementan los requisitos previos para auditar las configuraciones de las VM Windows en "Opciones de seguridad: Acceso a la red"

### <a name="ac-142--ac--143---remote-access-for-non-employees-must-be-restricted-to-allow-access-only-to-specifically-approved-information-systems"></a>AC-1.42 y AC-1.43: el acceso remoto para los usuarios que no son empleados debe restringirse para permitir el acceso únicamente a los sistemas de información aprobados específicamente

- \[Versión preliminar\]: mostrar los resultados de las auditorías de máquinas virtuales Linux que permitan conexiones remotas desde cuentas sin contraseña

### <a name="ac-150--log-security-related-events-for-all-information-system-components"></a>AC-1.50: eventos relacionados con la seguridad del registro para todos los componentes del sistema de información

- Los registros de diagnóstico de Logic Apps deben estar habilitados

### <a name="ac-154--ensure-multi-factor-authentication-mfa-is-enabled-for-all-cloud-console-users"></a>AC-1.54: asegúrese de que la autenticación multifactor (MFA) está habilitada para todos los usuarios de la consola en la nube

- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción
- Multi-Factor Authentication (MFA) debe estar habilitada para todas las cuentas de la suscripción que tengan permisos de escritura, a fin de evitar una brecha de seguridad en las cuentas o los recursos.

## <a name="auditing--logging"></a>Auditoría y registros

### <a name="al-21--successful-and-unsuccessful-events-must-be-logged"></a>AL-2.1: se deben registrar los eventos correctos y fallidos

- Los registros de diagnóstico de los servicios de Search deben estar habilitados.

### <a name="al--216---network-devicesinstances-must-log-any-event-classified-as-a-critical-security-event-by-that-network-deviceinstance-elbs-web-application-firewalls-etc"></a>AL-2.16: los dispositivos o las instancias de red deben registrar cualquier evento que esa instancia o dispositivo de red (ELB, firewalls de aplicaciones web, etc.) clasifique como de seguridad crítico

- \[Versión preliminar\]: se muestran los resultados de la auditoría de las configuraciones de las VM Windows en "Opciones de seguridad: Cuentas"

### <a name="al-217--serversinstances-must-log-any-event-classified-as-a-critical-security-event-by-that-serverinstance"></a>AL-2.17: los servidores o instancias deben registrar cualquier evento que ese servidor o instancia clasifique como de seguridad crítico

- \[Versión preliminar\]: se muestran los resultados de la auditoría de las configuraciones de las VM Windows en "Opciones de seguridad: Cuentas"

### <a name="al-219---domain-events-must-log-any-event-classified-as-a-critical-or-high-security-event-by-the-domain-management-software"></a>AL-2.19: los eventos de dominio deben registrar cualquier evento que el software de administración de dominios clasifique como un evento de seguridad crítico o de alto nivel

- \[Versión preliminar\]: se muestran los resultados de la auditoría de las configuraciones de las VM Windows en "Opciones de seguridad: Cuentas"
- \[Versión preliminar\]: se implementan los requisitos previos para auditar las configuraciones de las VM Windows en "Opciones de seguridad: Cliente de redes de Microsoft"

### <a name="al-220--domain-events-must-log-any-event-classified-as-a-critical-security-event-by-domain-security-controls"></a>AL-2.20: los eventos de dominio deben registrar cualquier evento que los controles de seguridad de dominios clasifique como un evento de seguridad crítico

- \[Versión preliminar\]: se muestran los resultados de la auditoría de las configuraciones de las VM Windows en "Opciones de seguridad: Cuentas"

### <a name="al-221--domain-events-must-log-any-access-or-changes-to-the-domain-log"></a>AL-2.21: los eventos de dominio deben registrar cualquier acceso o cambio en el registro de dominio

- \[Versión preliminar\]: se muestran los resultados de la auditoría de las configuraciones de las VM Windows en "Opciones de seguridad: Consola de recuperación"

## <a name="cryptographic-controls"></a>Controles criptográficos

### <a name="cc-42--applications-and-systems-must-use-current-cryptographic-solutions-for-protecting-data"></a>CC-4.2: las aplicaciones y los sistemas deben usar las soluciones criptográficas actuales para proteger los datos

- Se debe permitir el cifrado de datos transparente en bases de datos SQL
- El cifrado de datos transparente debe estar habilitado para proteger los datos en reposo y satisfacer los requisitos de cumplimiento.

### <a name="cc-45--digital-certificates-must-be-signed-by-an-approved-certificate-authority"></a>CC-4.5: los certificados digitales deben estar firmados por una entidad de certificación aprobada

- \[Versión preliminar\]: se muestran los resultados de la auditoría de las VM Windows que contengan certificados que expirarán en el plazo de días especificado

### <a name="cc-46--digital-certificates-must-be-uniquely-assigned-to-a-user-or-device"></a>CC-4.6: los certificados digitales deben asignarse de forma única a un usuario o un dispositivo

- \[Versión preliminar\]: se implementan los requisitos previos para realizar una auditoría de las VM Windows que contengan certificados que expirarán en el plazo de días especificado

### <a name="cc-47--cryptographic-material-must-be-stored-to-enable-decryption-of-the-records-for-the-length-of-time-the-records-are-retained"></a>CC-4.7: el material criptográfico debe almacenarse para habilitar el descifrado de los registros en el período de tiempo durante el que se conservan los registros

- El cifrado de discos debe aplicarse en máquinas virtuales
- Azure Security Center supervisará las VM sin el cifrado de disco habilitado como recomendaciones.

### <a name="cc-48--secret-and-private-keys-must-be-stored-securely"></a>CC-4.8: el secreto y las claves privadas se deben almacenar de forma segura

- Se debe permitir el cifrado de datos transparente en bases de datos SQL
- El cifrado de datos transparente debe estar habilitado para proteger los datos en reposo y satisfacer los requisitos de cumplimiento.

## <a name="change--config-management"></a>Administración de cambios y configuración

### <a name="cm-52--only-authorized-users-may-implement-approved-changes-on-the-system"></a>CM-5.2: solo los usuarios autorizados pueden implementar los cambios aprobados en el sistema

- Se deben instalar actualizaciones del sistema en las máquinas
- Azure Security Center supervisará las actualizaciones del sistema de seguridad que faltan en los servidores como recomendaciones.

### <a name="cm-512--maintain-an-up-to-date-complete-accurate-and-readily-available-baseline-configuration-of-the-information-system"></a>CM-5.12: mantenga la configuración de base de referencia del sistema de información actualizada, completa, precisa y disponible

- Se deben instalar actualizaciones del sistema en las máquinas
- Azure Security Center supervisará las actualizaciones del sistema de seguridad que faltan en los servidores como recomendaciones.

### <a name="cm-513--employ-automated-tools-to-maintain-a-baseline-configuration-of-the-information-system"></a>CM-5.13: emplee herramientas automatizadas para mantener una configuración de base de referencia del sistema de información

- Se deben instalar actualizaciones del sistema en las máquinas
- Azure Security Center supervisará las actualizaciones del sistema de seguridad que faltan en los servidores como recomendaciones.

### <a name="cm-514--identify-and-disable-unnecessary-andor-non-secure-functions-ports-protocols-and-services"></a>CM-5.14: identifique y deshabilite las funciones, puertos, protocolos y servicios innecesarios o no seguros.

- Las interfaces de red deben deshabilitar el reenvío IP
- \[Versión preliminar\]: el reenvío de IP en la máquina virtual debe estar deshabilitado

### <a name="cm-519--monitor-changes-to-the-security-configuration-settings"></a>CM-5.19: supervise los cambios en la configuración de seguridad

- Implementación de la configuración de diagnóstico de grupos de seguridad de red

### <a name="cm-522--ensure-that-only-authorized-software-and-updates-are-installed-on-company-systems"></a>CM-5.22: asegúrese de que solo el software y las actualizaciones autorizadas están instaladas en los sistemas de la empresa

- Se deben instalar actualizaciones del sistema en las máquinas
- Azure Security Center supervisará las actualizaciones del sistema de seguridad que faltan en los servidores como recomendaciones.

## <a name="identity--authentication"></a>Autenticación e identidad

### <a name="ia-71--user-accounts-must-be-uniquely-assigned-to-individuals-for-access-to-information-that-is-not-classified-as-public-account-ids-must-be-constructed-using-a-standardized-logical-format"></a>IA-7.1: las cuentas de usuario deben asignarse de forma única a los usuarios para acceder a la información que no esté clasificada como pública. Los identificadores de cuenta se deben construir con un formato lógico normalizado.

- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción a fin de evitar el acceso no supervisado.

## <a name="network-security"></a>Seguridad de redes

### <a name="ns-92--access-to-network-device-management-functionality-is-restricted-to-authorized-users"></a>NS-9.2: el acceso a la funcionalidad de administración de dispositivos de red está restringido a los usuarios autorizados

- \[Versión preliminar\]: se implementan los requisitos previos para auditar las configuraciones de las VM Windows en "Opciones de seguridad: Acceso a la red"

### <a name="ns-93--all-network-devices-must-be-configured-using-their-most-secure-configurations"></a>NS-9.3: todos los dispositivos de red deben configurarse con sus configuraciones más seguras

- \[Versión preliminar\]: se implementan los requisitos previos para auditar las configuraciones de las VM Windows en "Opciones de seguridad: Acceso a la red"

### <a name="ns-95--all-network-connections-to-a-system-through-a-firewall-must-be-approved-and-audited-on-a-regular-basis"></a>NS-9.5: todas las conexiones de red a un sistema con un firewall deben aprobarse y auditarse de forma regular

- \[Versión preliminar\]: se muestran los resultados de la auditoría de las configuraciones de las VM Windows en "Propiedades de Firewall de Windows"

### <a name="ns-97--appropriate-controls-must-be-present-at-any-boundary-between-a-trusted-network-and-any-untrusted-or-public-network"></a>NS-9.7: los controles adecuados deben estar presentes en cualquier límite entre una red de confianza y cualquier red pública o que no sea de confianza

- \[Versión preliminar\]: se implementan los requisitos previos para auditar las configuraciones de las VM Windows en "Propiedades de Firewall de Windows"

## <a name="security-planning"></a>Planeación de seguridad

### <a name="sp-113--threats-must-be-identified-that-could-negatively-impact-the-confidentiality-integrity-or-availability-of-company-information-and-content-along-with-the-likelihood-of-their-occurrence"></a>SP-11.3: se deben identificar las amenazas que puedan afectar de forma desfavorable a la confidencialidad, la integridad o la disponibilidad del contenido y la información de la empresa, junto con la probabilidad de que se produzcan

- Los tipos de Advanced Threat Protection deben estar establecidos en "Todos" en la configuración de Advanced Data Security de la instancia administrada de SQL Server

### <a name="security-continuity"></a>Continuidad de la seguridad

## <a name="sc-125--data-in-long-term-storage-must-be-accessible-throughout-the-retention-period-and-protected-against-media-degradation-and-technology-changes"></a>SC-12.5: los datos en el almacenamiento a largo plazo deben ser accesibles durante todo el período de retención y estar protegidos contra la degradación multimedia y los cambios tecnológicos.

- Debe configurar los servidores SQL Server para que realicen una retención de la auditoría durante más de 90 días.
- Audite los servidores SQL Server configurados con un período de retención de auditoría de menos de 90 días.

## <a name="system-integrity"></a>Integridad del sistema

### <a name="si-143--only-authorized-personnel-may-monitor-network-and-user-activities"></a>SI-14.3: solo el personal autorizado puede supervisar las actividades de red y de usuario.

- Se deben corregir las vulnerabilidades de las bases de datos SQL
- Supervise los resultados del análisis de evaluación de puntos vulnerables y las recomendaciones para solucionar los de las bases de datos.

### <a name="si-144--internet-facing-systems-must-have-intrusion-detection"></a>SI-14.4: los sistemas orientados a Internet deben tener detección de intrusiones

- Implementación de la detección de amenazas en servidores SQL Server.

### <a name="si-1413--standardized-centrally-managed-anti-malware-software-should-be-implemented-across-the-company"></a>SI-14.13: debe implementarse en toda la empresa software antimalware administrado centralmente

- Implementación de la extensión de Microsoft IaaSAntimalware predeterminada para Windows Server

### <a name="si-1414--anti-malware-software-must-scan-computers-and-media-weekly-at-a-minimum"></a>SI-14.14: el software antimalware debe examinar los equipos y los medios cada semana, como mínimo

- Implementación de la extensión de Microsoft IaaSAntimalware predeterminada para Windows Server

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

### <a name="vm-154--ensure-that-applications-are-scanned-for-vulnerabilities-on-a-monthly-basis"></a>VM-15.4: asegúrese de que las aplicaciones se examinan para detectar puntos vulnerables cada mes

- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Audite las vulnerabilidades del sistema operativo en los conjuntos de escalado de máquinas virtuales para protegerlos frente a ataques.

### <a name="vm-155--ensure-that-vulnerabilities-are-identified-paired-to-threats-and-evaluated-for-risk"></a>VM-15.5: asegúrese de que se identifican los puntos vulnerables, se emparejan con las amenazas y se evalúa el riesgo

- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Audite las vulnerabilidades del sistema operativo en los conjuntos de escalado de máquinas virtuales para protegerlos frente a ataques.

### <a name="vm-156--ensure-that-identified-vulnerabilities-have-been-remediated-within-a-mutually-agreed-upon-timeline"></a>VM-15.6: asegúrese de que se han corregido los puntos vulnerables identificados dentro de una escala de tiempo acordada mutuamente

- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Audite las vulnerabilidades del sistema operativo en los conjuntos de escalado de máquinas virtuales para protegerlos frente a ataques.

### <a name="vm-157--access-to-and-use-of-vulnerability-management-systems-must-be-restricted-to-authorized-personnel"></a>VM-15.7: el acceso y el uso de los sistemas de administración de puntos vulnerables deben restringirse al personal autorizado

- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales
- Audite las vulnerabilidades del sistema operativo en los conjuntos de escalado de máquinas virtuales para protegerlos frente a ataques.

> [!NOTE]
> La disponibilidad de definiciones específicas de Azure Policy puede variar tanto en Azure Government como en otras nubes nacionales. 

## <a name="next-steps"></a>Pasos siguientes

Ya ha leído la asignación de controles del ejemplo de plano técnico de Media. Ahora, consulte los siguientes artículos para ver una introducción y aprender a implementar este ejemplo:

> [!div class="next step action"]
> [Plano técnico de Media Framework: asignación de controles](./control-mapping.md)
> [Plano técnico de Media Framework: pasos de implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
