---
title: Ejemplo de plano técnico de nivel de impacto 5 del DoD
description: Pasos de implementación para el ejemplo de plano técnico de nivel de impacto 5 del DoD, incluidos los detalles de los parámetros del artefacto de plano técnico.
ms.date: 09/17/2020
ms.topic: sample
ms.openlocfilehash: deb6ffa0f886cc6a4a9bea10a38d7cd82e7df8f1
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90978233"
---
# <a name="deploy-the-dod-impact-level-5-blueprint-sample"></a>Implementación del ejemplo de plano técnico de nivel de impacto 5 del DoD

Para implementar el ejemplo de plano técnico del nivel de impacto 5 del Departamento de Defensa (IL5 del DoD) de Azure Blueprints, tiene que realizar los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico **Nivel de impacto 5 de DoD** en _Otras muestras_ y seleccione **Usar esta muestra**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para la copia del ejemplo de plano técnico de nivel de impacto 5 del DoD.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

## <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adecuarla a su entorno y necesidades, pero esa modificación puede apartarla de la alineación con los controles del nivel de impacto 5 del DoD.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. Escriba **Notas de cambios**, como "Primera versión publicada del ejemplo de plano técnico DoD IL5". A continuación, seleccione **Publicar** en la parte inferior de la página.

## <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos

     - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena de antemano de forma automática en función del nombre del plano técnico.
       Cámbielo si fuera necesario o déjelo tal cual.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Versión de definición de Blueprint**: Elija una versión **publicada** de la copia del ejemplo de plano técnico.

   - Asignación de bloqueo

     Seleccione el valor de bloqueo de plano técnico para el entorno. Para más información, consulte [Bloqueo de recursos en planos técnicos](../../concepts/resource-locking.md).

   - Identidad administrada

     Deje la opción predeterminada de identidad administrada _asignada por el sistema_.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

## <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Descripción|
|-|-|-|-|
|Nivel de impacto 5 de DoD|Asignación de directiva|Lista de usuarios que deben incluirse en el grupo de administradores de VM de Windows|Lista separada por punto y coma de los usuarios que se deben incluir en el grupo Administradores local. P. ej.: Administrador; miUsuario1; miUsuario2|
|Nivel de impacto 5 de DoD|Asignación de directiva|Lista de usuarios excluidos del grupo de administradores de VM de Windows|Lista separada por punto y coma de los usuarios que se deben excluir en el grupo Administradores local; P. ej.: Administrador; miUsuario1; miUsuario2|
|Nivel de impacto 5 de DoD|Asignación de directiva|Lista de tipos de recursos que deben tener los registros de diagnóstico habilitados||
|Nivel de impacto 5 de DoD|Asignación de directiva|Id. del área de trabajo de Log Analytics para la creación de informes del agente de VM.|Id. (GUID) del área de trabajo de Log Analytics en que los agentes de VM deben realizar sus notificaciones.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Lista de regiones en las que se debe habilitar Network Watcher|Para ver una lista completa de regiones, use Get-AzLocation.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Versión de TLS mínima para servidores web con Windows|Versión mínima del protocolo TLS que debe habilitarse en los servidores web con Windows.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Versión de PHP más reciente|Versión de PHP más reciente admitida para App Services|
|Nivel de impacto 5 de DoD|Asignación de directiva|Versión de Java más reciente|Versión de Java más reciente admitida para App Services|
|Nivel de impacto 5 de DoD|Asignación de directiva|Versión más reciente de Python para Windows|Versión de Python más reciente admitida para App Services|
|Nivel de impacto 5 de DoD|Asignación de directiva|Versión más reciente de Python para Linux|Versión de Python más reciente admitida para App Services|
|Nivel de impacto 5 de DoD|Asignación de directiva|Opcional: lista de imágenes de VM Windows que admiten el agente de Log Analytics para la adición al ámbito de auditoría.|Lista de imágenes separadas por punto y coma. P. ej.: /subscriptions/<subscriptionId>/resourceGroups/YourResourceGroup/providers/Microsoft.Compute/images/ContosoStdImage.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Opcional: lista de imágenes de VM Linux que admiten el agente de Log Analytics para la adición a un ámbito de auditoría.|Lista de imágenes separadas por punto y coma. P. ej.: /subscriptions/<subscriptionId>/resourceGroups/YourResourceGroup/providers/Microsoft.Compute/images/ContosoStdImage.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Debe haber más de un propietario asignado a su suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: El cifrado de discos debe aplicarse en máquinas virtuales|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La opción para enviar notificaciones por correo electrónico al propietario de la suscripción en relación a alertas de gravedad alta debe estar habilitada|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se debe desactivar la depuración remota para Function App|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de .NET Framework es la más reciente, si se usa como parte de la aplicación de funciones.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se debe permitir el cifrado de datos transparente en bases de datos SQL|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de PHP es la más reciente si se usa como parte de la aplicación de API.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Solo se deben habilitar las conexiones seguras a Redis Cache|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La solución de protección del punto de conexión debe instalarse en las máquinas virtuales|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Auditar el acceso de red sin restricciones a cuentas de almacenamiento|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La configuración de seguridad de datos avanzada para una instancia administrada SQL debe contener una dirección de correo electrónico para recibir alertas de seguridad|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se debe habilitar la transferencia segura a las cuentas de almacenamiento|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Deben habilitarse los controles de aplicaciones adaptables en las máquinas virtuales|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La copia de seguridad con redundancia geográfica debe estar habilitada para Azure Database for PostgreSQL.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Java es la más reciente si se usa como parte de la aplicación web.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Debe designar un máximo de tres propietarios para la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Es necesario proporcionar una dirección de correo electrónico de contacto de seguridad para la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: CORS no debe permitir que todos los recursos accedan a las aplicaciones web|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Las cuentas externas con permisos de escritura deben quitarse de la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Las cuentas externas con permisos de lectura deben quitarse de la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Las cuentas en desuso deben quitarse de la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Acceso a Function App solo a través de HTTPS|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Python es la más reciente si se usa como parte de la aplicación web.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Python" es la más reciente si se usa como parte de la aplicación de funciones.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de PHP es la más reciente si se usa como parte de la aplicación web.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Python es la más reciente si se usa como parte de la aplicación de API.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La copia de seguridad con redundancia geográfica debe estar habilitada para Azure Database for MySQL.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de .NET Framework es la más reciente, si se usa como parte de la aplicación web.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se deben instalar actualizaciones del sistema en las máquinas|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Java es la más reciente si se usa como parte de la aplicación de API.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de HTTP es la más reciente si se usa para ejecutar la aplicación web.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación de API.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La configuración de seguridad de datos avanzada para SQL Server debe contener una dirección de correo electrónico para recibir alertas de seguridad|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de HTTP es la más reciente si se usa para ejecutar la aplicación de API.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La extensión IaaSAntimalware de Microsoft debe implementarse en servidores Windows.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Java es la más reciente si se usa como parte de la aplicación de funciones.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Debe restringirse el acceso a través de un punto de conexión accesible desde Internet|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Es necesario seleccionar el plan de tarifa estándar de Security Center.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Auditar el uso de reglas de RBAC personalizadas|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Acceso a la aplicación web solo a través de HTTPS|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La auditoría de SQL Server debe estar habilitada|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: El agente de Log Analytics debe instalarse en máquinas virtuales|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se debe habilitar DDoS Protection estándar|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de PHP es la más reciente si se usa como parte de la aplicación de funciones.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La opción Advanced Data Security debe estar habilitada en las instancias administradas de SQL|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Las notificaciones a los administradores y a los propietarios de la suscripción deben estar habilitadas en la configuración de seguridad avanzada de datos de la instancia administrada de SQL|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Supervisión de la falta de Endpoint Protection en Azure Security Center|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: El control de acceso de red Just-In-Time se debe aplicar en las máquinas virtuales.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Es necesario proporcionar un número de teléfono de contacto de seguridad para la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Acceso a API App solo a través de HTTPS|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Los tipos de Advanced Threat Protection deben estar establecidos en "Todos" en la configuración de Advanced Data Security de la instancia administrada de SQL Server|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: El almacenamiento con redundancia geográfica debe estar habilitado para las cuentas de almacenamiento.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de .NET Framework es la más reciente, si se usa como parte de la aplicación de API.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Las notificaciones por correo electrónico para administradores y propietarios de suscripciones deben estar habilitadas en la configuración de seguridad avanzada de datos de SQL Server|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se debe desactivar la depuración remota para las aplicaciones web|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La copia de seguridad con redundancia geográfica a largo plazo debe estar habilitada para las instancias de Azure SQL Database.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de HTTP es la más reciente si se usa para ejecutar la aplicación de funciones.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Los tipos de Advanced Threat Protection deben estar establecidos en "Todos" en la configuración de Advanced Data Security de SQL Server|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se debe desactivar la depuración remota para API Apps|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: El agente de Log Analytics debe instalarse en Virtual Machine Scale Sets|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación web.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Las cuentas externas con permisos de propietario deben quitarse de la suscripción|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación de funciones.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: [versión preliminar]: Kubernetes Services se debe actualizar a una versión de Kubernetes no vulnerable.|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|
|Nivel de impacto 5 de DoD|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades de las bases de datos SQL|Efecto de Azure Policy para esta directiva. Para más información acerca de los efectos, visite https://aka.ms/policyeffects.|

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado los pasos para implementar el ejemplo de plano técnico de nivel de impacto 5 del DoD, consulte los siguientes artículos para obtener información sobre el plano técnico y sobre la asignación de controles:

> [!div class="nextstepaction"]
> [Plano técnico de nivel de impacto 5 del DoD: introducción](./index.md)
> [Plano técnico de nivel de impacto 5 del DoD: asignación de controles](./control-mapping.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).