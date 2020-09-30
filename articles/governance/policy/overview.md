---
title: Introducción a Azure Policy
description: Azure Policy es un servicio de Azure que se usa para crear, asignar y administrar las definiciones de directivas en el entorno de Azure.
ms.date: 09/22/2020
ms.topic: overview
ms.openlocfilehash: 662a7510013e2008d8c16cf21376b11c247e0bc0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905980"
---
# <a name="what-is-azure-policy"></a>¿Qué es Azure Policy?

Azure Policy ayuda a aplicar los estándares de la organización y a evaluar el cumplimiento a escala. A través de su panel de cumplimiento, proporciona una vista agregada para evaluar el estado general del entorno, con la posibilidad de explorar en profundidad hasta el nivel de recurso y directiva. También ayuda al cumplimiento de los recursos gracias a la corrección masiva de los recursos existentes y la corrección automática de nuevos recursos.

Entre los casos de uso comunes de Azure Policy se incluye la implementación de la gobernanza para la coherencia de los recursos, el cumplimiento normativo, la seguridad, el costo y la administración. Las definiciones de directiva de estos casos de uso comunes ya están disponibles en el entorno de Azure como complementos para ayudarle a comenzar.

## <a name="overview"></a>Información general

Para evaluar los recursos de Azure, Azure Policy compara las propiedades de esos recursos con las reglas de negocio. Estas reglas de negocios, descritas en [formato JSON](./concepts/definition-structure.md), se conocen como [definiciones de directiva](#policy-definition). Para simplificar la administración, se pueden agrupar varias reglas de negocio para formar una [iniciativa de directiva](#initiative-definition) (a veces conocida como _policySet_). Una vez formadas las reglas de negocio, se [asigna](#assignments) la definición o la iniciativa de directiva a cualquier ámbito de recursos que admita Azure, como [grupos de administración](../management-groups/overview.md), suscripciones, [grupos de recursos](../../azure-resource-manager/management/overview.md#resource-groups) o recursos individuales. La asignación se aplica a todos los recursos dentro del [ámbito de Resource Manager](../../azure-resource-manager/management/overview.md#understand-scope) de esa asignación. Si es necesario, se pueden excluir los subámbitos. Para más información, consulte [Ámbito de Azure Policy](./concepts/scope.md).

Azure Policy usa un [formato JSON](./concepts/definition-structure.md) para formar la lógica que la evaluación emplea para determinar si un recurso es compatible o no. Las definiciones incluyen metadatos y la regla de directiva. La regla definida puede usar funciones, parámetros, operadores lógicos, condiciones y [alias](./concepts/definition-structure.md#aliases) de propiedad para coincidir exactamente con el escenario deseado. La regla de directiva determina qué recursos del ámbito de la asignación se evalúan.

### <a name="understand-evaluation-outcomes"></a>Descripción de los resultados de evaluación

Los recursos se evalúan en momentos concretos durante el ciclo de vida de los recursos, el ciclo de vida de la asignación de directivas y la evaluación de cumplimiento periódica continua. Estos son los acontecimientos o eventos que hacen que se evalúe un recurso:

- Un recurso se crea, actualiza o elimina en un ámbito con una asignación de directiva.
- Una directiva o iniciativa se asigna recientemente a un ámbito.
- Una directiva o iniciativa que ya está asignada a un ámbito se actualiza.
- Durante el ciclo de evaluación de cumplimiento estándar, que se produce una vez cada 24 horas.

Para más información sobre cuándo y cómo se realiza la evaluación de directivas, consulte [Desencadenadores de evaluación](./how-to/get-compliance-data.md#evaluation-triggers).

### <a name="control-the-response-to-an-evaluation"></a>Control de la respuesta a una evaluación

Las reglas de negocio para administrar recursos no compatibles varían considerablemente según la organización. Entre los ejemplos de cómo una organización desea que la plataforma responda a un recurso no compatibles están los siguientes:

- Denegar el cambio de recursos
- Registrar el cambio en el recurso
- Modificar el recurso antes del cambio
- Modificar el recurso después del cambio
- Implementar recursos compatibles relacionados

Azure Policy hace posible cada una de estas respuestas empresariales mediante la aplicación de [efectos](./concepts/effects.md). Los efectos se establecen en la parte de la **definición de directiva** de la [regla de directiva](./concepts/definition-structure.md).

### <a name="remediate-non-compliant-resources"></a>Corrección de recursos no compatibles

Aunque estos efectos influyen principalmente sobre un recurso cuando este se crea o actualiza, Azure Policy también admite el tratamiento de recursos no compatibles existentes sin necesidad de modificar ese recurso. Para más información sobre cómo conseguir que los recursos existentes sean compatibles, consulte [cómo corregir los recursos](./how-to/remediate-resources.md).

### <a name="video-overview"></a>Introducción en vídeo

La siguiente introducción de Azure Policy es de la compilación 2018. Para descargar diapositivas o el vídeo, visite [Govern your Azure environment through Azure Policy](https://channel9.msdn.com/events/Build/2018/THR2030) (Gobierno de un entorno de Azure mediante Azure Policy) en Channel 9.

> [!VIDEO https://www.youtube.com/embed/dxMaYF2GB7o]

## <a name="getting-started"></a>Introducción

### <a name="azure-policy-and-rbac"></a>Azure Policy y RBAC

Hay algunas diferencias importantes entre Azure Policy y el control de acceso basado en rol (RBAC). Para evaluar el estado, Azure Policy examina las propiedades de los recursos que se representan en Resource Manager y las propiedades de algunos proveedores de recursos. Azure Policy no restringe las acciones (también denominadas _operaciones_). Azure Policy garantiza que el estado de los recursos sea compatible con las reglas de negocio sin importar quién hizo el cambio o quién tiene permisos para hacerlo.

RBAC se centra en la administración de [acciones](../../role-based-access-control/resource-provider-operations.md) del usuario en ámbitos diferentes. Si es necesario controlar una acción, RBAC es la herramienta correcta para ello. Incluso si un individuo tiene acceso para realizar una acción, si el resultado es un recurso no compatible, Azure Policy sigue bloqueando la creación o la actualización.

La combinación de RBAC y Azure Policy proporciona un control completo del ámbito en Azure.

### <a name="rbac-permissions-in-azure-policy"></a>Permisos de RBAC en Azure Policy

Azure Policy dispone de varios permisos, conocidos como operaciones, en dos proveedores de recursos:

- [Microsoft.Authorization](../../role-based-access-control/resource-provider-operations.md#microsoftauthorization)
- [Microsoft.PolicyInsights](../../role-based-access-control/resource-provider-operations.md#microsoftpolicyinsights)

Muchos roles integrados conceden permiso a recursos de Azure Policy. El rol **Colaborador de directiva de recursos** incluye la mayoría de las operaciones de Azure Policy. El rol **Propietario** tiene derechos completos. Tanto el rol **Colaborador** como el rol **Lector** tienen acceso a todas las operaciones de _lectura_ de Azure Policy. El rol **Colaborador** puede desencadenar la corrección de recursos, pero no puede _crear_ definiciones o asignaciones.

Si ninguno de los roles integrados tiene los permisos necesarios, cree un [rol personalizado](../../role-based-access-control/custom-roles.md).

> [!NOTE]
> La identidad administrada de una asignación de directiva **deployIfNotExists** necesita permisos suficientes para crear o actualizar los recursos incluidos en la plantilla. Para más información, consulte [Configurar definiciones de directiva para la corrección](./how-to/remediate-resources.md#configure-policy-definition).

### <a name="resources-covered-by-azure-policy"></a>Recursos que abarca Azure Policy

Azure Policy evalúa todos los recursos de Azure. En el caso de algunos proveedores de recursos, como [Configuración de invitado](./concepts/guest-configuration.md), [Azure Kubernetes Service](../../aks/intro-kubernetes.md) y [Azure Key Vault](../../key-vault/general/overview.md), hay una integración más profunda para administrar valores de configuración y objetos. Para más información, consulte [Modos del proveedor de recursos](./concepts/definition-structure.md).

### <a name="recommendations-for-managing-policies"></a>Recomendaciones para la administración de directivas

A continuación, se ofrecen algunos consejos e indicaciones que se deben tener en cuenta:

- Comience con un efecto de auditoría en lugar de un efecto de denegación para realizar un seguimiento del impacto de la definición de directiva en los recursos de su entorno. Si ya tiene scripts implementados para escalar automáticamente las aplicaciones, establecer un efecto de denegación puede dificultar las tareas de automatización que ya tiene implementadas.

- Considere las jerarquías organizativas al crear definiciones y asignaciones. Se recomienda crear definiciones a niveles más altos, como suscripción o grupo de administración. A continuación, cree la asignación en el siguiente nivel secundario. Si crea una definición en un grupo de administración, se puede limitar la asignación a una suscripción o un grupo de recursos dentro de ese grupo de administración.

- Se recomienda crear y asignar las definiciones de iniciativa incluso para una definición de directiva única.
  Por ejemplo, tiene la definición de directiva _policyDefA_ y la crea en la definición de iniciativa _initiativeDefC_. Si crea otra definición de directiva más adelante para _policyDefB_ con objetivos similares a _policyDefA_, puede agregarla a _initiativeDefC_ y realizar un seguimiento conjunto.

- Una vez creada una asignación de iniciativa, las definiciones agregadas a la iniciativa también forman parte de esas asignaciones de iniciativas.

- Cuando se evalúa una asignación de iniciativa, también se evalúan todas las directivas incluidas en la iniciativa.
  Si necesita evaluar una directiva de forma individual, es mejor no incluirla en una iniciativa.

## <a name="azure-policy-objects"></a>Objetos de Azure Policy

### <a name="policy-definition"></a>Definición de directiva

El proceso de creación e implementación de una directiva en Azure Policy comienza con la creación de una definición de directiva. Cada definición de directiva tiene condiciones que regulan su aplicación. Además, tiene un efecto definido que se produce cuando se cumplen las condiciones.

En Azure Policy, se ofrecen varias directivas integradas que están disponibles de manera predeterminada. Por ejemplo:

- **Allowed Storage Account SKUs** (SKU permitidas de cuentas de almacenamiento) (denegar): determina si una cuenta de almacenamiento que se va a implementar se engloba dentro de un conjunto de tamaños de SKU. Su efecto es denegar todas las cuentas de almacenamiento que no cumplen el conjunto de tamaños de SKU definidos.
- **Allowed Resource Type** (Tipo de recurso permitido) (denegar): define los tipos de recursos que se pueden implementar. Su efecto es denegar todos los recursos que no forman parte de esta lista definida.
- **Ubicaciones permitidas** (denegar): restringe las ubicaciones disponibles para los nuevos recursos. Su efecto se utiliza para exigir los requisitos de cumplimiento de replicación geográfica.
- **Allowed Virtual Machine SKUs** (SKU de máquina virtual permitidas) (denegar): especifica un conjunto de SKU de máquina virtual que se pueden implementar.
- **Add a tag to resources** (Agregar una etiqueta a los recursos) (modificar): aplica una etiqueta obligatoria y su valor predeterminado si la solicitud de implementación no la especifica.
- **Append tag and its default value** (Anexar la etiqueta y su valor predeterminado) (anexar): aplica una etiqueta obligatoria y su valor a un recurso.
- **Not allowed resource types** (Tipos de recursos no permitidos) (denegar): impide la implementación de una lista de tipos de recursos.

Para implementar estas definiciones de directiva (tanto las definiciones integradas como las personalizadas), será preciso asignarlas. Puede asignar cualquiera de estas directivas a través de Azure Portal, PowerShell o la CLI de Azure.

La evaluación de la directiva se realiza con distintas acciones, como la asignación de directivas o las actualizaciones de directiva. Para obtener una lista completa, vea [desencadenadores de evaluación de directivas](./how-to/get-compliance-data.md#evaluation-triggers).

Para obtener más información sobre las estructuras de las definiciones de directiva, consulte [Estructura de definición de directiva](./concepts/definition-structure.md).

Los parámetros de directiva permiten simplificar la administración de directivas reduciendo el número de definiciones de directiva que debe crear. Cuando crea una definición de directiva, puede definir parámetros para que sea más genérica. Después, puede volver a usar esa definición de directiva para diferentes escenarios. Para ello, transmita distintos valores al asignar la definición de directiva. Por ejemplo, especifique un conjunto de ubicaciones para una suscripción.

Los parámetros se definen cuando se crea una definición de directiva. Cuando se define un parámetro, se le asigna un nombre y, de manera opcional, un valor. Por ejemplo, podría definir un parámetro para una directiva denominada _location_. Después, puede asignarle valores diferentes, como _EastUS_ o _WestUS_, al asignar una directiva.

Para más información sobre los parámetros de directiva, vea [Estructura de definición: Parámetros](./concepts/definition-structure.md#parameters).

### <a name="initiative-definition"></a>Definición de iniciativa

Una definición de iniciativa es una colección de definiciones de directiva personalizadas para alcanzar un único objetivo general. Las definiciones de iniciativa simplifican la administración y asignación de las definiciones de directiva. Tal simplificación se realiza mediante la agrupación de un conjunto de directivas como un solo elemento. Por ejemplo, podría crear una iniciativa titulada **Habilitar la supervisión en Azure Security Center**, con el objetivo de supervisar todas las recomendaciones de seguridad disponibles en Azure Security Center.

> [!NOTE]
> El SDK, como CLI de Azure y Azure PowerShell, usa propiedades y parámetros denominados **PolicySet** para hacer referencia a las iniciativas.

En esta iniciativa, tendría definiciones de directiva como las siguientes:

- **Supervisar SQL Database sin cifrar en Security Center**, para supervisar servidores y bases de datos SQL sin cifrar.
- **Supervisar las vulnerabilidades del SO en Security Center**, para supervisar los servidores que no cumplen con la base de referencia configurada.
- **Supervisar Endpoint Protection omitido en Security Center**, para supervisar los servidores que no tienen instalado un agente de protección de los puntos de conexión.

Al igual que los parámetros de directiva, los parámetros de iniciativa permiten simplificar la administración de iniciativas mediante la reducción de la redundancia. Los parámetros de iniciativa son aquellos que las definiciones de directiva usan dentro de la iniciativa.

Por ejemplo, imagine un escenario en el que tiene la definición de una iniciativa (**initiativeC**), con las definiciones de directiva **policyA** y **policyB**, y que cada una de ellas espera un tipo diferente de parámetro:

| Directiva | Nombre del parámetro |tipo del parámetro  |Nota: |
|---|---|---|---|
| policyA | allowedLocations | array  |Este parámetro espera una lista de cadenas para un valor, porque el tipo de parámetro está definido como una matriz |
| policyB | allowedSingleLocation |string |Este parámetro espera una palabra para un valor, porque el tipo de parámetro se definió como una cadena |

En este escenario, tiene tres opciones en el momento de definir los parámetros de iniciativa para **initiativeC**:

- Use los parámetros de las definiciones de directiva dentro de esta iniciativa: en este ejemplo, _allowedLocations_ y _allowedSingleLocation_ se convierten en parámetros de iniciativa para **initiativeC**.
- Proporcione valores a los parámetros de las definiciones de directiva dentro de esta definición de iniciativa. En este ejemplo, puede proporcionar una lista de ubicaciones al parámetro de **policyA**, **allowedLocations**, y al parámetro de **policyB**, **allowedSingleLocation**. También puede proporcionar valores cuando asigne esta iniciativa.
- Proporcione una lista de las opciones de _value_ que se puede usar cuando se asigna esta iniciativa. Cuando asigna esta iniciativa, los parámetros inherentes de las definiciones de directiva dentro de la iniciativa solo pueden tener valores provenientes de esta lista.

Al crear las opciones de valor en una definición de iniciativa, no se puede proporcionar un valor diferente durante la asignación de la iniciativa, porque no forma parte de la lista.

Para más información sobre las estructuras de las definiciones de iniciativa, consulte [Estructura de definición de iniciativa](./concepts/initiative-definition-structure.md).

### <a name="assignments"></a>Assignments

Una asignación es una definición o una iniciativa de directiva que se ha asignado para que ocurra dentro de un ámbito específico. Este ámbito puede ir desde un [grupo de administración](../management-groups/overview.md) a un recurso individual. El término _ámbito_ hace referencia a todos los recursos, grupos de recursos, suscripciones o grupos de administración a los que se asigna la definición. Todos los recursos secundarios heredan las asignaciones. Este diseño conlleva que una definición aplicada a un grupo de recursos también se aplique a los recursos de dicho grupo. Sin embargo, puede excluir un subámbito de la asignación.

Por ejemplo, en el ámbito de la suscripción, puede asignar una definición que impida la creación de recursos de red. También podría excluir un grupo de recursos en la suscripción que está diseñado para la infraestructura de red. De esta forma, concede acceso a este grupo de recursos de red a los usuarios de confianza con la creación de recursos de red.

En otro ejemplo, podría asignar una definición de lista de tipos de recursos permitidos en el nivel de grupo de administración. Luego, puede asignar una directiva más permisiva (que permita más tipos de recursos) en un grupo de administración secundario o incluso directamente en las suscripciones. Sin embargo, este ejemplo podría no funcionar porque Azure Policy es un sistema de denegación explícito. En su lugar, debe excluir el grupo de administración secundario o la suscripción de la asignación de nivel de grupo de administración. Después, puede asignar la definición más permisiva en el grupo de administración secundario o en el nivel de suscripción. Si alguna asignación tiene como resultado la denegación de un recurso, la única manera de permitir el recurso es modificar la directiva de denegación.

Para más información sobre la configuración de asignaciones mediante el portal, consulte [Creación de una asignación de directiva para identificar recursos no compatibles en el entorno de Azure](./assign-policy-portal.md). También hay pasos disponibles para [PowerShell](./assign-policy-powershell.md) y la [CLI de Azure](./assign-policy-azurecli.md). Para más información sobre la estructura de asignación, consulte la [estructura de asignaciones](./concepts/assignment-structure.md).

## <a name="maximum-count-of-azure-policy-objects"></a>Número máximo de objetos de Azure Policy

[!INCLUDE [policy-limits](../../../includes/azure-policy-limits.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene información general acerca de Azure Policy y algunos de los conceptos clave, se recomienda seguir estos pasos:

- [Revise la estructura de la definición de directiva](./concepts/definition-structure.md).
- [Asignación de una definición de directiva con el portal](./assign-policy-portal.md).
