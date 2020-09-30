---
title: Introducción al ejemplo de plano técnico de HIPAA HITRUST 9.2
description: Introducción al ejemplo de plano técnico de HIPAA HITRUST 9.2. Este ejemplo de plano técnico ayuda a los clientes a evaluar los controles concretos de HIPAA HITRUST 9.2.
ms.date: 09/04/2020
ms.topic: sample
ms.openlocfilehash: 4df6f05019976b3de1172cc5127c27ac00fe3c44
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89493246"
---
# <a name="hipaa-hitrust-92-blueprint-sample"></a>Ejemplo de plano técnico de HITRUST HIPAA 9.2

En el ejemplo de plano técnico de HIPAA HITRUST 9.2 se proporcionan directivas de gobernanza mediante [Azure Policy](../../policy/overview.md) que le ayudarán a evaluar los controles específicos de HIPAA HITRUST 9.2. Este plano técnico ayuda a los clientes a implementar un conjunto esencial de directivas para cualquier arquitectura implementada en Azure que deba implantar controles de HIPAA HITRUST 9.2.

## <a name="control-mapping"></a>Asignación de controles

La [asignación de controles de Azure Policy](../../policy/samples/hipaa-hitrust-9-2.md) proporciona detalles sobre las definiciones de directiva incluidas en este plano técnico y cómo se asignan a los **dominios de cumplimiento** y los **controles** en HIPAA HITRUST 9.2. Cuando se asignan a una arquitectura, Azure Policy evalúa los recursos para detectar posibles incumplimientos de las definiciones de directiva asignadas. Para más información, consulte [Azure Policy](../../policy/overview.md).

## <a name="deploy"></a>Implementación

Para implementar el ejemplo de plano técnico de HIPAA HITRUST 9.2 de Azure Blueprints, debe realizar los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

### <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico **HIPAA HITRUST** en _Otras muestras_ y seleccione **Usar esta muestra**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para la copia del ejemplo de plano técnico de HIPAA HITRUST 9.2.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

### <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adecuarla a su entorno y necesidades, pero esa modificación puede impedir que se alinee con los controles de HIPAA HITRUST 9.2.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. En **Notas de cambios** escriba, por ejemplo, "Primera versión publicada del ejemplo de plano técnico de HIPAA HITRUST 9.2". A continuación, seleccione **Publicar** en la parte inferior de la página.

### <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos

     - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena de antemano de forma automática en función del nombre del plano técnico.
       Cámbielo si fuera necesario o déjelo tal cual.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Versión de definición de Blueprint**: Elija una versión **publicada** de la copia del ejemplo de plano técnico.

   - Asignación de bloqueo

     Seleccione el valor de bloqueo de plano técnico para el entorno. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

   - Identidad administrada

     Deje la opción predeterminada de identidad administrada _asignada por el sistema_.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

### <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto |Nombre de parámetro |Descripción |
|---|---|---|
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Debe restringirse el acceso a través de un punto de conexión accesible desde Internet |Habilita o deshabilita la supervisión de reglas de NSG de entrada excesivamente permisivas. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Cuentas: estado de la cuenta de invitado |Especifica si la cuenta de invitado local está deshabilitada. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Deben habilitarse los controles de aplicaciones adaptables en las máquinas virtuales |Habilita o deshabilita la supervisión de una lista de aplicaciones permitidas en Azure Security Center. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Permitir conexiones simultáneas a Internet o a un dominio de Windows. |Especifique si quiere evitar que los equipos se conecten a redes basadas y no basadas en dominio al mismo tiempo. Un valor de 0 permite conexiones simultáneas y un valor de 1 las bloquea. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Acceso a API App solo a través de HTTPS V2 |Habilita o deshabilita la supervisión del uso del protocolo HTTPS en la aplicación de API V2. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Nombres de aplicación (se admiten caracteres comodín) |Lista separada por punto y coma de los nombres de las aplicaciones que deben estar instaladas. Ejemplo "Microsoft SQL Server 2014 (64 bits); Microsoft Visual Studio Code" o "Microsoft SQL Server 2014*" (para que coincida con las aplicaciones que comienzan con "Microsoft SQL Server 2014") |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Auditar finalización de procesos |Especifica si se generan eventos de auditoría cuando se sale de un proceso. Se recomienda para supervisar la finalización de procesos críticos. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Auditar el acceso de red sin restricciones a cuentas de almacenamiento |Habilita o deshabilita la supervisión del acceso de red a la cuenta de almacenamiento. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Auditoría: apagar el sistema de inmediato si no se pueden registrar las auditorías de seguridad |Audita si el sistema se apagará cuando no se puedan registrar eventos de seguridad. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Huellas digitales de certificado |Lista separada por punto y coma de huellas digitales de certificado que deben existir en el almacén de certificados raíz de confianza (Cert:\LocalMachine\Root). Por ejemplo, THUMBPRINT1;THUMBPRINT2;THUMBPRINT3 |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Se deben habilitar los registros de diagnóstico en las cuentas de Batch |Habilita o deshabilita la supervisión de registros de diagnóstico en las cuentas de Batch. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Los registros de diagnóstico del centro de eventos deben estar habilitados |Habilita o deshabilita la supervisión de registros de diagnóstico en cuentas de Event Hub. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Los registros de diagnóstico de los servicios de Search deben estar habilitados. |Habilita o deshabilita la supervisión de registros de diagnóstico en el servicio Azure Search. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Los registros de diagnóstico de Virtual Machine Scale Sets deben estar habilitados |Habilita o deshabilita la supervisión de registros de diagnóstico en Service Fabric. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |El cifrado de discos debe aplicarse en máquinas virtuales |Habilita o deshabilita la supervisión del cifrado de disco de máquina virtual. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Habilitar los inicios de sesión de invitado no seguros. |Especifica si el cliente SMB permitirá inicios de sesión de invitado no seguros en un servidor SMB. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |El control de acceso de red Just-In-Time se debe aplicar en las máquinas virtuales. |Habilita o deshabilita la supervisión del acceso de red Just-In-Time. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Se deben cerrar los puertos de administración en las máquinas virtuales |Habilita o deshabilita la supervisión de los puertos de administración abiertos en Virtual Machines. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción |Habilita o deshabilita la supervisión de MFA en las cuentas con permisos de escritura en la suscripción. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción |Habilita o deshabilita la supervisión de MFA en las cuentas con permisos de propietario en la suscripción. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Acceso a redes: rutas y subrutas del Registro accesibles remotamente |Especifica qué rutas de registro serán accesibles a través de la red, independientemente de los usuarios o grupos enumerados en la lista de control de acceso (ACL) de la clave del Registro `winreg`. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Acceso de red: rutas y subrutas del Registro accesibles de forma remota |Especifica qué rutas y subrutas del Registro serán accesibles a través de la red, con independencia de los usuarios o grupos enumerados en la lista de control de acceso (ACL) de la clave del Registro `winreg`. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Acceso a redes: recursos compartidos accesibles anónimamente |Especifica los recursos compartidos de red a los que pueden tener acceso los usuarios anónimos. La configuración predeterminada de esta opción de directiva tiene poco efecto, ya que todos los usuarios deben autenticarse para poder acceder a los recursos compartidos en el servidor. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Consola de recuperación: permitir la copia de disquetes y el acceso a todas las unidades y carpetas |Especifica si el comando SET de la consola de recuperación estará disponible, lo que permite establecer las variables de entorno de dicha consola. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Se debe desactivar la depuración remota para aplicaciones de API |Habilita o deshabilita la supervisión de la depuración remota en la aplicación de API. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Recomendación de desactivación de la depuración remota para aplicaciones web |Habilita o deshabilita la supervisión de la depuración remota para Web App. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Retención requerida (en días) para los registros en las cuentas de Batch |Período de retención de registros de diagnóstico requerido en días. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Retención requerida (en días) de los registros en el servicio Azure Search |Período de retención de registros de diagnóstico requerido en días. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Retención requerida (en días) de los registros en las cuentas del centro de eventos |Período de retención de registros de diagnóstico requerido en días. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Nombre del grupo de recursos de la cuenta de almacenamiento (debe existir) para implementar la configuración de diagnóstico de los grupos de seguridad de red |El grupo de recursos en el que se creará la cuenta de almacenamiento. Este grupo de recursos ya debe existir. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |En Kubernetes Services se debe usar el control de acceso basado en rol (RBAC) |Habilita o deshabilita la supervisión de los servicios de Kubernetes sin RBAC habilitado. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |El protector de TDE de la instancia administrada SQL debe estar cifrado con su propia clave |Habilita o deshabilita la supervisión del Cifrado de datos transparente (TDE) con su propia compatibilidad de claves. Esta opción le proporcionará una mayor transparencia y control sobre el protector de TDE, mayor seguridad con un servicio externo respaldado por HSM y promoción de la separación de obligaciones. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |El protector de TDE de SQL Server debe estar cifrado con su propia clave |Habilita o deshabilita la supervisión del Cifrado de datos transparente (TDE) con su propia compatibilidad de claves. Esta opción le proporcionará una mayor transparencia y control sobre el protector de TDE, mayor seguridad con un servicio externo respaldado por HSM y promoción de la separación de obligaciones. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Prefijo de la cuenta de almacenamiento de la cuenta de almacenamiento regional para implementar la configuración de diagnóstico de los grupos de seguridad de red |Este prefijo se combinará con la ubicación del grupo de seguridad de red para formar el nombre de la cuenta de almacenamiento que se creó. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales |Habilita o deshabilita los informes de actualizaciones del sistema de conjuntos de escalado de máquinas virtuales. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales |Habilita o deshabilita los informes de actualizaciones del sistema de conjuntos de escalado de máquinas virtuales. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Desactivar la resolución de nombres de multidifusión |Especifica si LLMNR, un protocolo de resolución de nombres secundario que transmite mediante multidifusión a través de un vínculo de subred local de una única subred, está habilitado. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Se deben migrar las máquinas virtuales a nuevos recursos de Azure Resource Manager |Habilita o deshabilita la supervisión de máquinas virtuales de proceso clásicas. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales |Habilita o deshabilita la supervisión de vulnerabilidades del sistema operativo de los conjuntos de escalado de máquinas virtuales. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades |Habilita o deshabilita la detección de vulnerabilidades de máquina virtual mediante una solución de evaluación de vulnerabilidades. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL. |Audita las instancias administradas de SQL que no tienen habilitados los exámenes de evaluación de vulnerabilidades periódicos. La evaluación de vulnerabilidades permite detectar las vulnerabilidades potenciales de la base de datos, así como hacer un seguimiento y ayudar a corregirlas. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (dominio): Aplicar reglas de firewall local |Especifica si los administradores locales pueden crear reglas de firewall locales que se apliquen junto con las reglas de firewall que configura la directiva de grupo para el perfil de dominio. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (dominio): Comportamiento de las conexiones salientes |Especifica el comportamiento de las conexiones salientes del perfil de dominio que no coinciden con una regla de firewall de salida. El valor predeterminado de 0 significa que se permiten las conexiones y un valor de 1 significa que se bloquean las conexiones. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (dominio): Comportamiento de las conexiones salientes |Especifica el comportamiento de las conexiones salientes del perfil de dominio que no coinciden con una regla de firewall de salida. El valor predeterminado de 0 significa que se permiten las conexiones y un valor de 1 significa que se bloquean las conexiones. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (dominio): Mostrar notificaciones |Especifica si el Firewall de Windows con seguridad avanzada muestra notificaciones al usuario cuando se impide a un programa recibir conexiones entrantes para el perfil de dominio. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (dominio): Usar configuración de perfil |Especifica si el Firewall de Windows con seguridad avanzada usa la configuración del perfil de dominio para filtrar el tráfico de red. Si selecciona Inactivo, Firewall de Windows con seguridad avanzada no usará ninguna regla de firewall ni de seguridad de conexión en este perfil. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (privado): Aplicar reglas de seguridad de conexión local |Especifica si los administradores locales pueden crear reglas de seguridad de conexión que se apliquen junto con las reglas de seguridad de conexión que configura la directiva de grupo para el perfil privado. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (privado): Aplicar reglas de firewall local |Especifica si los administradores locales pueden crear reglas de firewall locales que se apliquen junto con las reglas de firewall que configura la directiva de grupo para el perfil privado. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (privado): Comportamiento de las conexiones salientes |Especifica el comportamiento de las conexiones salientes del perfil privado que no coinciden con una regla de firewall de salida. El valor predeterminado de 0 significa que se permiten las conexiones y un valor de 1 significa que se bloquean las conexiones. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (privado): Mostrar notificaciones |Especifica si el Firewall de Windows con seguridad avanzada muestra notificaciones al usuario cuando se impide a un programa recibir conexiones entrantes para el perfil privado. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (privado): Usar configuración de perfil |Especifica si el Firewall de Windows con seguridad avanzada usa la configuración del perfil privado para filtrar el tráfico de red. Si selecciona Inactivo, Firewall de Windows con seguridad avanzada no usará ninguna regla de firewall ni de seguridad de conexión en este perfil. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (público): Aplicar reglas de seguridad de conexión local |Especifica si los administradores locales pueden crear reglas de seguridad de conexión que se apliquen junto con las reglas de seguridad de conexión que configura la directiva de grupo para el perfil público. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (público): Aplicar reglas de firewall local |Especifica si los administradores locales pueden crear reglas de firewall locales que se apliquen junto con las reglas de firewall que configura la directiva de grupo para el perfil público. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (público): Comportamiento de las conexiones salientes |Especifica el comportamiento de las conexiones salientes del perfil público que no coinciden con una regla de firewall de salida. El valor predeterminado de 0 significa que se permiten las conexiones y un valor de 1 significa que se bloquean las conexiones. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (público): Mostrar notificaciones |Especifica si el Firewall de Windows con seguridad avanzada muestra notificaciones al usuario cuando se impide a un programa recibir las conexiones entrantes para el perfil público. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows (público): Usar configuración de perfil |Especifica si el Firewall de Windows con seguridad avanzada usa la configuración del perfil público para filtrar el tráfico de red. Si selecciona Inactivo, Firewall de Windows con seguridad avanzada no usará ninguna regla de firewall ni de seguridad de conexión en este perfil. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows: Dominio: Permitir respuesta de unidifusión |Especifica si el Firewall de Windows con seguridad avanzada permite al equipo local recibir respuestas de unidifusión a sus mensajes de difusión o multidifusión salientes para el perfil de dominio. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows: Privado: Permitir respuesta de unidifusión |Especifica si el Firewall de Windows con seguridad avanzada permite al equipo local recibir respuestas de unidifusión a sus mensajes de difusión o multidifusión salientes para el perfil privado. |
|Auditar los controles HITRUST/HIPAA e implementar extensiones de máquina virtual específicas para admitir los requisitos de auditoría |Firewall de Windows: Público: Permitir respuesta de unidifusión |Especifica si el Firewall de Windows con seguridad avanzada permite al equipo local recibir respuestas de unidifusión a sus mensajes de difusión o multidifusión salientes para el perfil público. |

## <a name="next-steps"></a>Pasos siguientes

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
