---
title: Controles del ejemplo de plano técnico PCI-DSS v3.2.1
description: Asignación a los controles de la muestra de plano técnico Estándar de seguridad de los datos para el sector de las tarjetas de pago v3.2.1 en Azure Policy y RBAC.
ms.date: 08/19/2020
ms.topic: sample
ms.openlocfilehash: e6133c4a847a6df8aa6a27bbca63e0fc2d047783
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88649234"
---
# <a name="control-mapping-of-the-pci-dss-v321-blueprint-sample"></a>Asignación a los controles de la muestra de plano técnico PCI-DSS v3.2.1

En el siguiente artículo se detalla cómo se asigna la muestra de plano técnico de Azure Blueprints PCI-DSS v3.2.1 a los controles de PCI-DSS v3.2.1. Para obtener más información sobre los controles, consulte [PCI-DSS v3.2.1](https://www.pcisecuritystandards.org/documents/PCI_DSS_v3-2-1.pdf).

Las siguientes asignaciones son para los controles de **PCI-DSS v3.2.1:2018**. Use el panel de navegación de la derecha para ir directamente a una asignación de control específica. Muchos de los controles asignados se implementan con una iniciativa de [Azure Policy](../../../policy/overview.md). Para revisar la iniciativa completa, abra **Policy** en Azure Portal y seleccione la página **Definiciones**. Después, busque y seleccione la iniciativa de directiva integrada **\[Versión preliminar\] Auditar los controles PCI v3.2.1:2018 e implementar extensiones de VM específicas para admitir los requisitos de auditoría**.

> [!IMPORTANT]
> Cada control que se muestra a continuación está asociado a una o varias definiciones de [Azure Policy](../../../policy/overview.md). Estas directivas pueden ayudarle a [evaluar el cumplimiento](../../../policy/how-to/get-compliance-data.md) del control; sin embargo, a menudo no hay una coincidencia 1:1 o completa entre un control y una o varias directivas. Como tal, el **cumplimiento** con Azure Policy solo se refiere a las propias directivas; esto no garantiza que sea totalmente compatible con todos los requisitos de un control. Además, el estándar de cumplimiento incluye controles que no se abordan con las definiciones de Azure Policy en este momento. Por lo tanto, el cumplimiento en Azure Policy es solo una vista parcial del estado general de cumplimiento. Las asociaciones entre los controles y las definiciones de Azure Policy para este ejemplo de plano técnico de cumplimiento pueden cambiar con el tiempo. Para ver el historial de cambios, consulte el [historial de confirmación de GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/pci-dss-3.2.1/control-mapping.md).

## <a name="132-and-134-boundary-protection"></a>Protección de límites de 1.3.2 y 1.3.4

Este plano técnico ayuda a administrar y controlar las redes mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan los grupos de seguridad de red con reglas permisivas. Las reglas demasiado permisivas pueden permitir el acceso de red no deseado y deben revisarse. Este plano técnico asigna una definición de Azure Policy que supervisa las aplicaciones, las cuentas de almacenamiento y los puntos de conexión desprotegidos. Los puntos de conexión y las aplicaciones que no están protegidos por firewall y las cuentas de almacenamiento con acceso sin restricciones pueden permitir el acceso no deseado a la información contenida en el sistema de información.

- Auditar el acceso de red sin restricciones a cuentas de almacenamiento
- Debe restringirse el acceso a través de un punto de conexión accesible desde Internet

## <a name="34a-41-41g-41h-and-653-cryptographic-protection"></a>Protección criptográfica de 3.4.a, 4.1, 4.1.g, 4.1.h y 6.5.3

Este plano técnico ayuda a instaurar la directiva con el uso de controles criptográficos mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que instauran controles criptográficos específicos y auditan el uso de configuraciones criptográficas poco seguras. Conocer dónde los recursos de Azure pueden tener configuraciones criptográficas subóptimas ayuda a tomar medidas correctivas para garantizar que los recursos se configuran conforme a la directiva de seguridad de la información. En concreto, las directivas que asigna este plano técnico requieren el cifrado de datos transparente en bases de datos SQL, la auditoría del cifrado ausente en las cuentas de almacenamiento y las variables de cuenta de automatización. También hay directivas que auditan las conexiones poco seguras a cuentas de almacenamiento, Function App, WebApp, API Apps y Redis Cache y auditan la comunicación de Service Fabric sin cifrar.

- Acceso a Function App solo a través de HTTPS
- Acceso a la aplicación web solo a través de HTTPS
- Acceso a API App solo a través de HTTPS
- Se debe permitir el cifrado de datos transparente en bases de datos SQL
- El cifrado de discos debe aplicarse en máquinas virtuales
- Las variables de cuenta de automatización deben cifrarse
- Solo se deben habilitar las conexiones seguras a Redis Cache
- Se debe habilitar la transferencia segura a las cuentas de almacenamiento
- Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric
- Se debe permitir el cifrado de datos transparente en bases de datos SQL
- Implementación del cifrado de datos transparente de SQL DB

## <a name="51-62-66-and-1121-vulnerability-scanning-and-system-updates"></a>Examen de vulnerabilidades y actualizaciones del sistema de 5.1, 6.2, 6.6 y 11.2.1

Este plano técnico ayuda a administrar las vulnerabilidades del sistema de información mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que supervisan si faltan actualizaciones del sistema, o si existen vulnerabilidades del sistema operativo, de SQL o de las máquinas virtuales de Azure Security Center. Azure Security Center proporciona funcionalidades de informes que permiten tener información en tiempo real sobre el estado de seguridad de los recursos de Azure implementados.

- Supervisión de la falta de Endpoint Protection en Azure Security Center
- Implementación de la extensión de Microsoft IaaSAntimalware predeterminada para Windows Server
- Implementación de la detección de amenazas en servidores SQL Server
- Se deben instalar actualizaciones del sistema en las máquinas
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas
- Se deben corregir las vulnerabilidades de las bases de datos SQL
- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades

## <a name="711-712-and-713-separation-of-duties"></a>7.1.1. Separación de obligaciones de 7.1.2 y 7.1.3

Tener solo un propietario de la suscripción de Azure no permite la redundancia administrativa. Por el contrario, tener demasiados propietarios de suscripción de Azure aumenta la posibilidad de infracción de seguridad mediante una cuenta de propietario en riesgo. Este plano técnico ayuda a mantener un número adecuado de propietarios de suscripción de Azure mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan el número de estos. La administración de permisos de propietario de suscripción ayuda a implementar la separación adecuada de las tareas.

- Debe haber más de un propietario asignado a su suscripción
- Debe designar un máximo de tres propietarios para la suscripción 

## <a name="32-721-831a-and-831b-management-of-privileged-access-rights"></a>Administración de derechos de acceso con privilegios de 3.2, 7.2.1, 8.3.1.a y 8.3.1.b

Este plano técnico ayuda a restringir y controlar los derechos de acceso con privilegios mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas externas con permisos de propietario, de escritura y/o de lectura y cuentas de empleados con permisos de propietario y/o de escritura que no tienen habilitada la autenticación multifactor. Azure implementa el control de acceso basado en rol (RBAC) para administrar quién tiene acceso a los recursos de Azure. Conocer dónde se implementan las reglas RBAC personalizadas ayuda a verificar que la implementación se necesita y se realiza correctamente, ya que estas suelen producir errores. Este plano técnico también asigna definiciones de [Azure Policy](../../../policy/overview.md) para auditar el uso de la autenticación de Azure Active Directory para los servidores SQL Server. El uso de la autenticación de Azure Active Directory simplifica la administración de permisos y centraliza la administración de identidades de usuarios de base de datos y otros servicios  
de Microsoft.
 
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de escritura deben quitarse de la suscripción
- Las cuentas externas con permisos de lectura deben quitarse de la suscripción
- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción
- MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción
- El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server
- Auditar el uso de reglas de RBAC personalizadas

## <a name="812-and-815-least-privilege-and-review-of-user-access-rights"></a>Privilegios mínimos y revisión de derechos de acceso de usuario de 8.1.2 y 8.1.5

Azure implementa el control de acceso basado en rol (RBAC) para ayudar a administrar quién tiene acceso a los recursos de Azure. Mediante Azure Portal, puede revisar quién tiene acceso a los recursos de Azure y sus permisos. Este plano técnico asigna definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas que deberían priorizarse para la revisión, incluidas las cuentas en desuso y las cuentas externas con permisos elevados.

- Las cuentas en desuso deben quitarse de la suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de escritura deben quitarse de la suscripción
- Las cuentas externas con permisos de lectura deben quitarse de la suscripción

## <a name="813-removal-or-adjustment-of-access-rights"></a>Eliminación o ajuste de derechos de acceso de 8.1.3

Azure implementa el control de acceso basado en rol (RBAC) para permitirle administrar quién tiene acceso a los recursos de Azure. Con Azure Active Directory y RBAC puede actualizar los roles de usuario para reflejar los cambios en la organización. Cuando sea necesario, se puede bloquear el inicio de sesión de las cuentas (o eliminar estas), lo cual eliminaría inmediatamente los derechos de acceso a los recursos de Azure. Este plano técnico asigna definiciones de [Azure Policy](../../../policy/overview.md) para auditar las cuentas en desuso cuya eliminación debería considerarse.

- Las cuentas en desuso deben quitarse de la suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción

## <a name="823ab-824ab-and-825-password-based-authentication"></a>Autenticación basada en contraseña de 8.2.3.a,b, 8.2.4.a,b y 8.2.5

Este plano técnico ayuda a instaurar contraseñas seguras mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan las máquinas virtuales Windows que no tienen unos requisitos mínimos de seguridad de contraseña y de otros tipos. Conocer las máquinas virtuales que infringen la directiva de seguridad de las contraseñas ayuda a tomar medidas correctivas para garantizar que las contraseñas de todas las cuentas de usuario de máquina virtual cumplen la directiva.

- \[Versión preliminar\]: auditoría de las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia máxima de 70 días
- \[Versión preliminar\]: implementación de los requisitos para auditar las máquinas virtuales Windows cuyas contraseñas no tengan una vigencia máxima de 70 días
- \[Versión preliminar\]: auditoría de las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas en 14 caracteres
- \[Versión preliminar\]: implementación de los requisitos para auditar las máquinas virtuales Windows que no restrinjan la longitud mínima de las contraseñas a 14 caracteres
- \[Versión preliminar\]: auditoría de las máquinas virtuales Windows que permitan volver a usar las 24 contraseñas anteriores
- \[Versión preliminar\]: implementación de los requisitos para auditar las máquinas virtuales Windows que permitan volver a usar las 24 contraseñas anteriores

## <a name="103-and-1054-audit-generation"></a>Generación de auditoría de 10.3 y 10.5.4

Este plano técnico ayuda a garantizar que los eventos del sistema se registran mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan las configuraciones de registro de los recursos de Azure.
Los registros de diagnóstico proporcionan conclusiones detalladas sobre las operaciones que se realizaron en recursos de Azure. Los registros de Azure se basan en relojes internos sincronizados para crear un registro con correlación de tiempo de eventos entre recursos.

- La configuración de seguridad avanzada de datos debe estar habilitada en SQL Server
- Auditar la configuración de diagnóstico
- Auditar configuración de auditoría de SQL en el nivel de servidor
- Implementación de auditorías en servidores SQL Server
- Se deben migrar las cuentas de almacenamiento a los nuevos recursos de Azure Resource Manager
- Se deben migrar las máquinas virtuales a nuevos recursos de Azure Resource Manager

## <a name="1236-and-1237-information-security"></a>Seguridad de la información de 12.3.6 y 12.3.7

Este plano técnico ayuda a administrar y controlar la red mediante la asignación de definiciones de [Azure Policy](../../../policy/overview.md) que auditan las ubicaciones de red aceptables y los productos de la empresa aprobados permitidos para el entorno. Cada empresa puede personalizarlas mediante los parámetros de directiva de cada una de las directivas.

- Ubicaciones permitidas
- Ubicaciones permitidas para grupos de recursos

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado la asignación de controles del plano técnico PCI-DSS v3.2.1, consulte los siguientes artículos para obtener información general y ver cómo se implementa esta muestra:

> [!div class="nextstepaction"]
> [Plano técnico PCI-DSS v3.2.1: introducción](./index.md)
> [Plano técnico PCI-DSS v3.2.1: pasos para la implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).