---
title: Configuración de los valores de diagnóstico del almacén a gran escala
description: Configuración de diagnóstico de Log Analytics de diagnóstico para todos los almacenes de un ámbito determinado mediante Azure Policy
ms.topic: conceptual
ms.date: 02/14/2020
ms.openlocfilehash: 25946bb742c7666e7d394477318f647009b2f50e
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378056"
---
# <a name="configure-vault-diagnostics-settings-at-scale"></a>Configuración de los valores de diagnóstico del almacén a gran escala

La solución de informes proporcionada por Azure Backup utiliza Log Analytics (LA). En el caso de los datos de un almacén determinado que se va a enviar a LA, se debe crear una [configuración de diagnóstico](./backup-azure-diagnostic-events.md) para ese almacén.

A menudo, agregar manualmente una configuración de diagnóstico por almacén puede ser una tarea complicada. Además, cualquier nuevo almacén creado también debe tener habilitada la configuración de diagnóstico para poder ver los informes que contiene.

Para simplificar la creación de la configuración de diagnóstico a gran escala (con LA como destino), Azure Backup tiene una instancia de [Azure Policy](../governance/policy/index.yml) integrada. Esta directiva agrega una configuración de diagnóstico de LA a todos los almacenes de una suscripción o un grupo de recursos determinado. En las secciones siguientes se proporcionan instrucciones para usar esta directiva.

## <a name="supported-scenarios"></a>Escenarios admitidos

* La directiva se puede aplicar al mismo tiempo a todos los almacenes de Recovery Services de una suscripción determinada (o a un grupo de recursos dentro de la suscripción). El usuario que asigna la directiva debe tener acceso de **propietario** a la suscripción a la que se asigna la directiva.

* El área de trabajo de LA especificada por el usuario (a la que se enviarán los datos de diagnóstico) puede estar en una suscripción distinta de la de los almacenes a los que se asigna la directiva. El usuario debe tener acceso de **lector**, **colaborador** o **propietario** a la suscripción en la que existe el área de trabajo de LA especificado.

* Actualmente no se admite el ámbito del grupo de administración.

* La directiva integrada no está disponible actualmente en nubes nacionales.

## <a name="assigning-the-built-in-policy-to-a-scope"></a>Asignación de la directiva integrada a un ámbito

Para asignar la directiva a almacenes del ámbito necesario, siga estos pasos:

1. Inicie sesión en Azure Portal y vaya al panel **Directiva**.
2. Seleccione **Definiciones** en el menú de la izquierda para obtener una lista de todas las directivas integradas en los recursos de Azure.
3. Filtre la lista por **Categoría = Supervisión**. Busque la directiva denominada **[Preview]: Deploy Diagnostic Settings for Recovery Services Vault to Log Analytics workspace for resource specific categories** ([Versión preliminar]: Implementación de la configuración de diagnóstico para el almacén de Recovery Services en el área de trabajo de Log Analytics para categorías de recursos específicas).

    ![Panel de definición de directiva](./media/backup-azure-policy-configure-diagnostics/policy-definition-blade.png)

4. Seleccione el nombre de la directiva. Se le redirigirá a la definición detallada de dicha directiva.

    ![Definición de directiva detallada](./media/backup-azure-policy-configure-diagnostics/detailed-policy-definition.png)

5. Seleccione el botón **Asignar** en la parte superior del panel. Esta acción le redirigirá al panel **Asignar directiva**.

6. En **Conceptos básicos**, seleccione los tres puntos situados junto al campo **Ámbito**. Se abrirá el panel de contexto adecuado, donde podrá seleccionar la suscripción en la que se va a aplicar la directiva. También puede seleccionar un grupo de recursos, de modo que la directiva se aplique solo a los almacenes de un grupo de recursos determinado.

    ![Conceptos básicos de la asignación de directiva](./media/backup-azure-policy-configure-diagnostics/policy-assignment-basics.png)

7. En **Parámetros**, escriba la siguiente información:

    * **Nombre del perfil**: nombre que se asignará a la configuración de diagnóstico creada por la directiva.
    * **Área de trabajo de Log Analytics**: área de trabajo de Log Analytics a la que se debe asociar la configuración de diagnóstico. Los datos de diagnóstico de todos los almacenes del ámbito de la asignación de directiva se insertarán en el área de trabajo de LA especificada.

    * **Exclusion Tag Name [Nombre de etiqueta de exclusión (opcional)] y Exclusion Tag Value [Valor de etiqueta de exclusión (opcional)]** : puede optar por excluir de la asignación de directiva aquellos almacenes que contengan un determinado nombre y valor de etiqueta. Por ejemplo, si **no** desea que se agregue una configuración de diagnóstico a los almacenes que tengan una etiqueta "isTest" establecida en el valor "yes", debe escribir "isTest" en el campo **Nombre de etiqueta de exclusión** y "yes" en el campo **Valor de etiqueta de exclusión**. Si se deja en blanco alguno de estos dos campos (o ambos), la directiva se aplicará a todos los almacenes pertinentes, sean cuales sean las etiquetas que contengan.

    ![Parámetros de la asignación de directiva](./media/backup-azure-policy-configure-diagnostics/policy-assignment-parameters.png)

8. **Crear una tarea de corrección**: una vez que se asigna la directiva a un ámbito, los nuevos almacenes creados en ese ámbito obtienen automáticamente la configuración de diagnóstico de LA establecida (en un plazo de 30 minutos desde la hora de creación del almacén). Para agregar una configuración de diagnóstico a los almacenes existentes en el ámbito, puede desencadenar una tarea de corrección a la hora de asignación de la directiva. Para desencadenar una tarea de corrección, active la casilla **Crear una tarea de corrección**.

    ![Corrección de la asignación de directiva](./media/backup-azure-policy-configure-diagnostics/policy-assignment-remediation.png)

9. Vaya a pestaña **Revisar y crear** y seleccione **Crear**.

## <a name="under-what-conditions-will-the-remediation-task-apply-to-a-vault"></a>¿En qué condiciones se aplica la tarea de corrección a un almacén?

La tarea de corrección se aplica a los almacenes que no son compatibles según la definición de la directiva. Un almacén no es compatible si cumple alguna de las condiciones siguientes:

* No hay ninguna configuración de diagnóstico para el almacén.
* La configuración de diagnóstico está presente para el almacén, pero ninguna de las opciones tiene **todos** los eventos específicos del recurso habilitados con LA como destino y **Específico del recurso** seleccionado en el botón de alternancia.

Por tanto, aunque un usuario tenga un almacén con el evento AzureBackupReport habilitado en el modo AzureDiagnostics (que es compatible con Informes de copia de seguridad), la tarea de corrección se seguirá aplicando a este almacén, ya que el modo Específico del recurso es la manera recomendada de crear la configuración de diagnóstico [en adelante](./backup-azure-diagnostic-events.md#legacy-event).

Además, si un usuario tiene un almacén con solo un subconjunto de los seis eventos específicos del recurso habilitados, la tarea de corrección se aplicará a este almacén, ya que Informes de copia de seguridad funcionará según lo previsto solo si los seis eventos específicos del recurso están habilitados.

> [!NOTE]
>
> Si un almacén tiene una configuración de diagnóstico existente con un **subconjunto de categorías Específico del recurso** habilitadas, que se ha configurado para enviar datos a un área de trabajo de LA específica (por ejemplo, "Área de trabajo X"), se producirá un error en la tarea de corrección (solo para ese almacén) si el área de trabajo de LA de destino proporcionada en la asignación de directiva es la **misma** "Área de trabajo X".
>
>Esto se debe a que si los eventos habilitados por dos configuraciones de diagnóstico diferentes en el mismo recurso se **superponen** de alguna forma, la configuración no puede tener la misma área de trabajo de LA que el destino. Tendrá que resolver manualmente este error; para ello, vaya al almacén pertinente y establezca una configuración de diagnóstico con un área de trabajo de LA distinta de la del destino.
>
> Tenga en cuenta que **no** se producirá un error en la tarea de corrección si la configuración de diagnóstico existente solo está habilitada por AzureBackupReport con Área de trabajo X como destino, ya que en este caso no habrá ninguna superposición entre los eventos habilitados por la configuración existente y los eventos habilitados por la configuración creada con la tarea de corrección.

## <a name="next-steps"></a>Pasos siguientes

* [Información sobre el uso de Informes de copia de seguridad](./configure-reports.md)
* [Más información acerca de Azure Policy](../governance/policy/index.yml)
* [Uso de Azure Policy para habilitar automáticamente la copia de seguridad de todas las VM de un ámbito determinado](./backup-azure-auto-enable-backup.md)
