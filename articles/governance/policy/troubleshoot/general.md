---
title: Solución de errores comunes
description: Aprenda a solucionar problemas relacionados con la creación de definiciones de directivas, los diversos SDK y el complemento de Kubernetes.
ms.date: 08/17/2020
ms.topic: troubleshooting
ms.openlocfilehash: d4ede1703df922196c89a4c1ca4f37cbc95a6297
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545546"
---
# <a name="troubleshoot-errors-using-azure-policy"></a>Solución de problemas mediante Azure Policy

Es posible que se produzcan errores al crear definiciones de directivas, trabajar con SDK o configurar el complemento [Azure Policy para Kubernetes](../concepts/policy-for-kubernetes.md). En este artículo se describen diversos errores que pueden producirse y cómo resolverlos.

## <a name="finding-error-details"></a>Búsqueda de detalles del error

La ubicación de los detalles del error depende de la acción que provoca el error.

- Cuando trabaje con una directiva personalizada, pruébela en Azure Portal para obtener comentarios de linting o revisar los [datos de cumplimiento](../how-to/get-compliance-data.md) resultantes para ver cómo se evaluaron los recursos.
- Si trabaja con varios SDK, los SDK proporcionan detalles sobre por qué se produjo un error en la función.
- Si trabaja con el complemento para Kubernetes, empiece por [registrarse](../concepts/policy-for-kubernetes.md#logging) en el clúster.

## <a name="general-errors"></a>Errores generales

### <a name="scenario-alias-not-found"></a>Escenario: Alias no encontrado

#### <a name="issue"></a>Incidencia

Azure Policy usa [alias](../concepts/definition-structure.md#aliases) para asignarlos a las propiedades de Azure Resource Manager.

#### <a name="cause"></a>Causa

Se ha utilizado un alias incorrecto o inexistente en una definición de directiva.

#### <a name="resolution"></a>Resolución

En primer lugar, compruebe que la propiedad de Resource Manager tiene un alias. Use la [extensión de Azure Policy para Visual Studio Code](../how-to/extension-for-vscode.md), [Azure Resource Graph](../../resource-graph/samples/starter.md#distinct-alias-values) o el SDK para buscar los alias disponibles. Si el alias de una propiedad de Resource Manager no existe, cree una incidencia de soporte técnico.

### <a name="scenario-evaluation-details-not-up-to-date"></a>Escenario: Detalles de evaluación no actualizados

#### <a name="issue"></a>Incidencia

Un recurso tiene el estado "No iniciado" o los detalles de cumplimiento no son actuales.

#### <a name="cause"></a>Causa

Cualquier nueva asignación de directiva o iniciativa tarda unos 30 minutos en aplicarse. Los recursos nuevos o actualizados dentro del ámbito de una asignación existente estarán disponibles unos 15 minutos más tarde. Un examen de cumplimiento estándar se produce cada 24 horas. Para más información, consulte los [desencadenadores de evaluación](../how-to/get-compliance-data.md#evaluation-triggers).

#### <a name="resolution"></a>Resolución

En primer lugar, espere la cantidad de tiempo adecuada para que se complete una evaluación y que los resultados de cumplimiento estén disponibles en Azure Portal o el SDK. Para iniciar un nuevo examen de evaluación con Azure PowerShell o la API REST, consulte [Examen de evaluación a petición](../how-to/get-compliance-data.md#on-demand-evaluation-scan).

### <a name="scenario-evaluation-not-as-expected"></a>Escenario: La evaluación no es la esperada

#### <a name="issue"></a>Incidencia

Un recurso no está en el estado de evaluación, ya sea _Compatible_ o _No compatible_, que es lo que se espera de ese recurso.

#### <a name="cause"></a>Causa

El recurso no está en el ámbito correcto de la asignación de directiva o la definición de directiva no funciona según lo previsto.

#### <a name="resolution"></a>Resolución

- En el caso de un recurso no compatible que se esperaba que fuera compatible, empiece por [determinar las razones de la no compatibilidad](../how-to/determine-non-compliance.md). La comparación de la definición con el valor de propiedad evaluado indica por qué un recurso no era compatible.
- Para un recurso compatible que se esperaba que fuera no compatible, lea la definición de directiva de cada condición y evalúe las propiedades de los recursos. Compruebe que los operadores lógicos agrupan las condiciones correctas y que las condiciones no están invertidas.

Si la compatibilidad de una asignación de directiva muestra `0/0` recursos, se determinó que ningún recurso era aplicable en el ámbito de la asignación. Compruebe la definición de la directiva y el ámbito de la asignación.

### <a name="scenario-enforcement-not-as-expected"></a>Escenario: La aplicación no es la esperada

#### <a name="issue"></a>Incidencia

Un recurso en el que se espera que actúe Azure Policy no existe y no hay ninguna entrada en el [registro de actividad de Azure](../../../azure-monitor/platform/platform-logs-overview.md).

#### <a name="cause"></a>Causa

La asignación de directivas se ha configurado para un valor de [enforcementMode](../concepts/assignment-structure.md#enforcement-mode) de _Deshabilitado_. Aunque el modo de cumplimiento esté deshabilitado, el efecto de la directiva no se aplica y no hay ninguna entrada en el registro de actividad.

#### <a name="resolution"></a>Resolución

Actualice el valor de **enforcementMode** a _Habilitado_. Este cambio permite a Azure Policy actuar en los recursos de esta asignación de directivas y enviar entradas al registro de actividad. Si **enforcementMode** ya está habilitado, consulte [La evaluación no es la esperada](#scenario-evaluation-not-as-expected) para ver las acciones a realizar.

### <a name="scenario-denied-by-azure-policy"></a>Escenario: Denegado por Azure Policy

#### <a name="issue"></a>Incidencia

Se ha denegado la creación o actualización de un recurso.

#### <a name="cause"></a>Causa

Una asignación de directivas en el ámbito en el que se encuentra el recurso nuevo o actualizado cumple los criterios de una definición de directiva con un efecto de [denegación](../concepts/effects.md#deny). Se impide que los recursos que reúnan estas condiciones se creen o actualicen.

#### <a name="resolution"></a>Resolución

El mensaje de error de una asignación de directiva de denegación incluye los identificadores de definición de directiva y de asignación de directivas. Si se pierde la información de error en el mensaje, también está disponible en el [registro de actividad](../../../azure-monitor/platform/activity-log.md#view-the-activity-log). Use esta información para obtener más detalles para comprender las restricciones de recursos y ajustar las propiedades de los recursos de la solicitud para que coincidan con los valores permitidos.

## <a name="template-errors"></a>Errores de plantilla

### <a name="scenario-policy-supported-functions-processed-by-template"></a>Escenario: Funciones admitidas por la directiva que procesa una plantilla

#### <a name="issue"></a>Incidencia

Azure Policy admite varias funciones y funciones de plantilla de Azure Resource Manager (plantilla de ARM) que solo están disponibles en una definición de directiva. Resource Manager procesa estas funciones como parte de una implementación en lugar de como parte de una definición de directiva.

#### <a name="cause"></a>Causa

El uso de funciones admitidas, como `parameter()` o `resourceGroup()`, da como resultado el resultado procesado de la función en el momento de la implementación en lugar de mantener la función para que la definición de la directiva y el motor de Azure Policy la procesen.

#### <a name="resolution"></a>Resolución

Para pasar una función para que forme parte de una definición de directiva, escape la cadena completa con `[` de forma que la propiedad se parezca a `[[resourceGroup().tags.myTag]`. El carácter de escape hace que Resource Manager trate el valor como una cadena al procesar la plantilla. Después, Azure Policy coloca la función en la definición de la directiva para que pueda ser dinámica según lo previsto. Para obtener más información, consulte [Sintaxis y expresiones de las plantillas de Azure Resource Manager](../../../azure-resource-manager/templates/template-expressions.md).

## <a name="add-on-installation-errors"></a>Errores de instalación del complemento

### <a name="scenario-install-using-helm-chart-fails-on-password"></a>Escenario: Error en la instalación mediante el gráfico de Helm al especificar la contraseña

#### <a name="issue"></a>Incidencia

El comando `helm install azure-policy-addon` produce un error y muestra uno de los siguientes mensajes:

- `!: event not found`
- `Error: failed parsing --set data: key "<key>" has no value (cannot end with ,)`

#### <a name="cause"></a>Causa

La contraseña generada incluye una coma (`,`) que hace que el gráfico de Helm se divida.

#### <a name="resolution"></a>Resolución

Use una barra diagonal inversa (`\`) como carácter de escape de la coma (`,`) cuando se ejecute `helm install azure-policy-addon`.

### <a name="scenario-install-using-helm-chart-fails-as-name-already-exists"></a>Escenario: Error en la instalación con el gráfico de Helm porque el nombre ya existe

#### <a name="issue"></a>Incidencia

El comando `helm install azure-policy-addon` produce un error y muestra uno de los siguientes mensajes:

- `Error: cannot re-use a name that is still in use`

#### <a name="cause"></a>Causa

El gráfico de Helm llamado `azure-policy-addon` ya se ha instalado completa o parcialmente.

#### <a name="resolution"></a>Resolución

Siga las instrucciones para [quitar el complemento Azure Policy para Kubernetes](../concepts/policy-for-kubernetes.md#remove-the-add-on) y, a continuación, vuelva a ejecutar el comando `helm install azure-policy-addon`.

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece o es incapaz de resolverlo, visite uno de nuestros canales para obtener más soporte técnico:

- Obtenga respuestas de expertos a través de [Preguntas y respuestas de Microsoft](/answers/topics/azure-policy.html).
- Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport): la cuenta de Microsoft Azure oficial para mejorar la experiencia del cliente, que pone en contacto a la comunidad de Azure con los recursos adecuados: respuestas, soporte técnico y expertos.
- Si necesita más ayuda, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**.
