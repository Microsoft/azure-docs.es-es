---
title: 'Organización de los recursos con grupos de administración: Servicios de gobernanza de Azure'
description: Más información sobre los grupos de administración, el funcionamiento de sus permisos y cómo utilizarlos.
ms.date: 07/06/2020
ms.topic: overview
ms.openlocfilehash: 787658cebcb8345edd616bcdde485883ea43e8dc
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87529354"
---
# <a name="what-are-azure-management-groups"></a>¿Qué son los grupos de administración de Azure?

Si su organización tiene varias suscripciones, puede que necesite una manera de administrar el acceso, las directivas y el cumplimiento de esas suscripciones de forma eficaz. Los grupos de administración de Azure proporcionan un nivel de ámbito por encima de las suscripciones. Las suscripciones se organizan en contenedores llamados "grupos de administración" y aplican sus condiciones de gobernanza a los grupos de administración. Todas las suscripciones dentro de un grupo de administración heredan automáticamente las condiciones que se aplican al grupo de administración. Los grupos de administración proporcionan capacidad de administración de nivel empresarial a gran escala con independencia del tipo de suscripciones que tenga.
Todas las suscripciones de un único grupo de administración deben confiar en el mismo inquilino de Azure Active Directory.

A modo de ejemplo, puede aplicar directivas a un grupo de administración que limite las regiones disponibles para la creación de máquinas virtuales. Esta directiva se aplicaría a todos los grupos de administración, las suscripciones y los recursos de ese grupo de administración, al permitir únicamente que se creen máquinas virtuales en esa región.

## <a name="hierarchy-of-management-groups-and-subscriptions"></a>Jerarquía de los grupos de administración y las suscripciones

Puede compilar una estructura flexible de grupos de administración y suscripciones para organizar los recursos en una jerarquía para una administración unificada de las directivas y el acceso. El siguiente diagrama muestra un ejemplo de creación de una jerarquía para la gobernanza mediante grupos de administración.

:::image type="content" source="./media/tree.png" alt-text="Ejemplo de árbol de jerarquía de un grupo de administración" border="false":::

Puede crear una jerarquía que aplique una directiva, por ejemplo, que limite las ubicaciones de las máquinas virtuales a la región Oeste de EE. UU. en el grupo denominado "Producción". Esta directiva se heredará en todas las suscripciones con Contrato Enterprise descendientes de ese grupo de administración y se aplicará a todas las máquinas virtuales de esas suscripciones. El propietario de recursos o suscripciones no puede modificar esta directiva de seguridad, lo que permite una gobernanza mejorada.

Otro escenario en el que usaría grupos de administración es para proporcionar acceso de usuario a varias suscripciones. Al mover muchas suscripciones bajo ese grupo de administración, puede crear una asignación de [control de acceso basado en rol](../../role-based-access-control/overview.md) (RBAC) en el grupo de administración que heredará ese acceso en todas las suscripciones. Una asignación en el grupo de administración puede permitir a los usuarios acceder a todo lo que necesitan en lugar de realizar scripting para proporcionar control de acceso basado en rol sobre las distintas suscripciones.

### <a name="important-facts-about-management-groups"></a>Hechos importantes acerca de los grupos de administración

- Se admiten 10 000 grupos de administración en un único directorio.
- Un árbol de grupo de administración puede admitir hasta seis niveles de profundidad.
  - Este límite no incluye el nivel raíz o de suscripción.
- Cada grupo de administración y suscripción admite solo un elemento primario.
- Cada grupo de administración puede tener muchos elementos secundarios.
- Todas las suscripciones y grupos de administración están dentro de una única jerarquía en cada directorio. Consulte [Hechos importantes acerca de los grupos de administración raíz](#important-facts-about-the-root-management-group).

## <a name="root-management-group-for-each-directory"></a>Un grupo de administración raíz para cada directorio

Cada directorio tiene un grupo de administración de nivel superior único denominado "raíz". Este grupo de administración raíz está integrado en la jerarquía de manera que contiene todos los grupos de administración y suscripciones. Este grupo de administración raíz permite la aplicación de directivas globales y de asignaciones de roles de Azure a nivel de directorio. Los [administradores globales de Azure AD necesitan elevar sus privilegios inicialmente](../../role-based-access-control/elevate-access-global-admin.md) al rol Administrador de acceso de usuario de este grupo raíz. Después de esta elevación de los privilegios de acceso, puede asignar cualquier rol de Azure a otros usuarios o grupos del directorio para administrar la jerarquía. Como administrador, puede asignar su propia cuenta como propietario del grupo de administración raíz.

### <a name="important-facts-about-the-root-management-group"></a>Hechos importantes acerca de los grupos de administración raíz

- De forma predeterminada, el nombre para mostrar del grupo de administración raíz es **Grupo raíz de inquilino**. El identificador es el de Azure Active Directory.
- Para cambiar el nombre para mostrar, se debe asignar a su cuenta el rol de Propietario o Colaborador en el grupo de administración raíz. Para actualizar el nombre de un grupo de administración, consulte [Cambio del nombre de un grupo de administración](manage.md#change-the-name-of-a-management-group).
- El grupo de administración raíz no se puede mover ni eliminar, a diferencia de los demás.  
- Todas las suscripciones y los grupos de administración se incluyen en el grupo de administración raíz único del directorio.
  - Todos los recursos del directorio pertenecen al grupo de administración raíz para la administración global.
  - Las suscripciones nuevas pertenecen de manera predeterminada y automática al grupo de administración raíz cuando se crean.
- Todos los clientes de Azure pueden ver el grupo de administración raíz, pero no todos los clientes tienen acceso para administrar ese grupo de administración raíz.
  - Todos los usuarios con acceso a una suscripción pueden ver el contexto de dónde está esa suscripción en la jerarquía.  
  - A ninguno se le concede acceso de forma predeterminada al grupo de administración raíz. Los administradores globales de Azure AD son los únicos usuarios que pueden elevar sus propios privilegios para obtener acceso. Una vez que tienen acceso al grupo de administración raíz, los administradores globales pueden asignar cualquier rol de Azure a otros usuarios para administrarlo  
    .
- En SDK, el grupo de administración raíz o la "raíz de inquilino" funciona como un grupo de administración.

> [!IMPORTANT]
> Todas las asignaciones de acceso de los usuarios o de directivas en el grupo de administración raíz **se aplican a todos los recursos dentro del directorio**. Por este motivo, todos los clientes deben evaluar la necesidad de tener los elementos definidos en este ámbito. Las asignaciones de directivas y de acceso de usuario deberían ser obligatorias solo en este  
> ámbito.

## <a name="initial-setup-of-management-groups"></a>Instalación inicial de los grupos de administración

Cuando algún usuario comienza usando grupos de administración, se produce un proceso de configuración inicial. El primer paso es que el grupo de administración raíz se crea en el directorio. Una vez creado este grupo, todas las suscripciones existentes que existen en el directorio se convierten en elementos secundarios del grupo de administración raíz.
El motivo de este proceso es asegurarse de que hay solo una jerarquía de grupos de administración en un directorio. La jerarquía única dentro del directorio permite a los clientes administrativos aplicar directivas y un acceso global que otros clientes dentro del directorio no puedan omitir. Nada que se haya asignado en la raíz se aplicará a toda jerarquía, lo cual incluye todos los grupos de administración, las suscripciones, los grupos de recursos y los recursos dentro de ese inquilino de Azure AD.

## <a name="trouble-seeing-all-subscriptions"></a>Problemas para ver todas las suscripciones

Algunos de los directorios que empezaron a usar grupos de administración durante la versión preliminar (antes del 25 de junio de 2018) podrían experimentar el problema de que no estén todas las suscripciones en la jerarquía. El proceso para que todas las suscripciones estén en la jerarquía se implementó después de realizar una asignación de roles o directivas en el grupo de administración raíz del directorio.

### <a name="how-to-resolve-the-issue"></a>Cómo resolver el problema

Hay dos opciones para resolver este problema.

- Eliminar todas las asignaciones de roles y directivas del grupo de administración raíz
  - Mediante la eliminación de todas las asignaciones de roles y directivas del grupo de administración raíz, el servicio repondrá todas las suscripciones en la jerarquía durante el siguiente ciclo nocturno. Este proceso es para asegurarse de que no se ha dado ningún acceso accidental ni asignación de directiva a todas las suscripciones de los inquilinos.
  - La mejor manera de realizar este proceso sin que afecte a los servicios es aplicar las asignaciones de roles o directivas un nivel por debajo del grupo de administración raíz. Después, puede quitar todas las asignaciones del ámbito raíz.
- Llamar a la API directamente para iniciar el proceso de reposición
  - Cualquier cliente del directorio puede llamar a las API _TenantBackfillStatusRequest_ o _StartTenantBackfillRequest_. Cuando se llama a StartTenantBackfillRequest API, esta comienza el proceso de configuración inicial de mover todas las suscripciones a la jerarquía. Este proceso también inicia la aplicación de todas las suscripciones nuevas para que constituyan un elemento secundario del grupo de administración raíz.
    Este proceso se puede realizar sin cambiar ninguna asignación en el nivel raíz. Mediante una llamada a la API, acepta que todas las asignaciones de directivas o de acceso en la raíz se pueden aplicar a todas las suscripciones.

Si tiene preguntas acerca de este proceso de reposición, póngase en contacto con: `managementgroups@microsoft.com`
  
## <a name="management-group-access"></a>Acceso al grupo de administración

Los grupos de administración de Azure admiten el [control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/overview.md) en los accesos a todos los recursos y definiciones de roles.
Estos permisos se heredan en los recursos secundarios que existen en la jerarquía. Cualquier rol de Azure puede asignarse a un grupo de administración que heredará la jerarquía para los recursos. Por ejemplo, el rol de Azure Colaborador de máquina virtual se puede asignar a un grupo de administración. Este rol no tiene ninguna acción en el grupo de administración, pero se heredará en todas las máquinas virtuales de ese grupo de administración.

El gráfico siguiente muestra la lista de roles y las acciones admitidas en los grupos de administración.

| Nombre de rol de Azure             | Crear | Cambiar nombre | Mover\*\* | Eliminar | Asignar acceso | Asignar directiva | Lectura  |
|:-------------------------- |:------:|:------:|:--------:|:------:|:-------------:| :------------:|:-----:|
|Propietario                       | X      | X      | X        | X      | X             | X             | X     |
|Colaborador                 | X      | X      | X        | X      |               |               | X     |
|Colaborador MG\*            | X      | X      | X        | X      |               |               | X     |
|Lector                      |        |        |          |        |               |               | X     |
|Lector MG\*                 |        |        |          |        |               |               | X     |
|Colaborador de directivas de recursos |        |        |          |        |               | X             |       |
|Administrador de acceso de usuario   |        |        |          |        | X             | X             |       |

\*: Colaborador MG y Lector MG solo permiten a los usuarios realizar esas acciones en el ámbito del grupo de administración.  
\*\*: Las asignaciones de roles en el grupo de administración raíz no son necesarias para mover una suscripción o grupo de administración a este grupo y desde este. Consulte [Administración de los recursos con grupos de administración](manage.md) para más información acerca de cómo mover elementos dentro de la jerarquía.

## <a name="azure-custom-role-definition-and-assignment"></a>Asignación y definición de roles personalizados de Azure

La compatibilidad de los roles personalizados de Azure con los grupos de administración está actualmente en versión preliminar con algunas [limitaciones](#limitations). El ámbito de los grupos de administración se puede definir en el ámbito asignable de Definición de roles. Ese rol personalizado de Azure estará disponible para su asignación no solo en ese grupo de administración, sino también en todos los grupos de administración, suscripciones, grupos de recursos o recursos que haya debajo de él. Este rol personalizado heredará la jerarquía, como cualquier rol integrado.  

### <a name="example-definition"></a>Definición de ejemplo

La [definición y creación de un rol personalizado](../../role-based-access-control/custom-roles.md) no cambia con la inclusión de grupos de administración. Use la ruta de acceso completa para definir el grupo de administración **/providers/Microsoft.Management/managementgroups/{groupId}** .

Use el identificador del grupo de administración, no su nombre para mostrar. Este error común se produce porque ambos son campos definidos personalizados al crear un grupo de administración.

```json
...
{
  "Name": "MG Test Custom Role",
  "Id": "id", 
  "IsCustom": true,
  "Description": "This role provides members understand custom roles.",
  "Actions": [
    "Microsoft.Management/managementgroups/delete",
    "Microsoft.Management/managementgroups/read",
    "Microsoft.Management/managementgroup/write",
    "Microsoft.Management/managementgroup/subscriptions/delete",
    "Microsoft.Management/managementgroup/subscriptions/write",
    "Microsoft.resources/subscriptions/read",
    "Microsoft.Authorization/policyAssignments/*",
    "Microsoft.Authorization/policyDefinitions/*",
    "Microsoft.Authorization/policySetDefinitions/*",
    "Microsoft.PolicyInsights/*",
    "Microsoft.Authorization/roleAssignments/*",
    "Microsoft.Authorization/roledefinitions/*"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
        "/providers/microsoft.management/managementGroups/ContosoCorporate"
  ]
}
...
```

### <a name="issues-with-breaking-the-role-definition-and-assignment-hierarchy-path"></a>Problemas al desasociar la definición del rol y la ruta de la jerarquía de asignación

Las definiciones de roles son un ámbito asignable en cualquier parte de la jerarquía de grupos de administración. Se puede definir una definición de roles en un grupo de administración primario mientras la asignación de roles real exista en la suscripción secundaria. Al haber una relación entre ambos elementos, aparecerá un error al intentar separar la asignación de su definición.

Por ejemplo, examinemos una pequeña sección de una jerarquía en un objeto visual.

:::image type="content" source="./media/subtree.png" alt-text="subárbol" border="false":::

Supongamos que hay un rol personalizado definido en el grupo de administración Marketing. Dicho rol personalizado se asigna en las dos suscripciones de prueba gratuitas.  

Si intentamos mover una de esas suscripciones para que sea un elemento secundario del grupo de administración Producción, se desasociaría la ruta de asignación de roles de la suscripción de la definición del rol del grupo de administración Marketing. En este escenario, recibirá un error que indica que no se permite el movimiento porque interrumpirá esta relación.  

Para corregir este escenario hay varias opciones:
- Quitar la asignación de roles de la suscripción antes de mover esta al nuevo grupo de administración primario.
- Agregar la suscripción al ámbito asignable de la definición de roles.
- Cambiar el ámbito asignable en la definición de roles. En el ejemplo anterior, se pueden actualizar los ámbitos asignables de Marketing a Grupo de administración raíz para que las dos ramas de la jerarquía puedan acceder a la definición.  
- Crear un rol personalizado adicional que se definirá en la otra rama. Este nuevo rol requerirá que la asignación de roles se cambie también en la suscripción.  

### <a name="limitations"></a>Limitaciones  

Existen limitaciones al usar roles personalizados en grupos de administración. 

 - En los ámbitos asignables de un nuevo rol no se puede definir más de un grupo de administración. Esta limitación se ha establecido para reducir el número de situaciones en las que las definiciones de roles y las asignaciones de roles están desconectadas. Esta situación aparece cuando una suscripción o un grupo de administración con una asignación de roles se mueven a un elemento primario diferente que no tiene la definición de roles.  
 - No se pueden definir acciones del plano de datos de RBAC en los roles personalizados del grupo de administración. Esta restricción se ha establecido porque hay un problema de latencia cuando las acciones de RBAC actualizan los proveedores de recursos del plano de datos.
   Se está trabajando en dicho problema y estas acciones se deshabilitarán de la definición de roles para reducir los riesgos.
 - Azure Resource Manager no valida la existencia del grupo de administración en el ámbito asignable de la definición de roles. Aunque haya algún error de escritura o un identificador de grupo de administración incorrecto en la lista, se creará la definición de roles.  

## <a name="moving-management-groups-and-subscriptions"></a>Movimiento de grupos de administración y suscripciones 

Para que un grupo de administración o una suscripción sean un elemento secundario de otro grupo de administración, es preciso evaluar tres reglas como verdaderas.

Si va a realizar la acción de movimiento, necesitará lo siguiente: 

- Permisos de escritura de grupos de administración y de escritura de la asignación de roles en la suscripción o en el grupo de administración secundarios.
  - Ejemplo de rol integrado: **Propietario**
- Acceso de escritura de grupos de administración en el grupo de administración primario de destino.
  - Ejemplo de rol integrado: **Propietario**, **Colaborador**, **Colaborador de grupo de administración**
- Acceso de escritura de grupos de administración en el grupo de administración primario existente.
  - Ejemplo de rol integrado: **Propietario**, **Colaborador**, **Colaborador de grupo de administración**

**Excepción**: Si el grupo de administración primario existente o de destino es el grupo de administración raíz, no se aplican los requisitos de permisos. Puesto que el grupo de administración raíz es la zona de aterrizaje predeterminada de todos los nuevos grupos de administración y suscripciones, no necesita permisos sobre él para mover un elemento.

Si el rol de propietario de la suscripción se hereda del grupo de administración actual, los destinos de movimiento están limitados. Solo puede mover la suscripción a otro grupo de administración en el que tenga el rol de propietario. No puede moverla a un grupo de administración en el que sea colaborador porque perdería la propiedad de la suscripción. Si se le asigna directamente el rol de propietario de la suscripción (no se hereda del grupo de administración), puede moverlo a cualquier grupo de administración donde sea colaborador.

## <a name="audit-management-groups-using-activity-logs"></a>Auditoría de los grupos de administración mediante registros de actividad

Se admiten grupos de administración en el [registro de actividad de Azure](../../azure-monitor/platform/platform-logs-overview.md). Puede buscar todos los eventos que se producen en un grupo de administración en la misma ubicación central que otros recursos de Azure. Por ejemplo, puede ver todos los cambios de asignaciones de roles o de asignación de directiva efectuados en un grupo de administración concreto.

:::image type="content" source="./media/al-mg.png" alt-text="Registros de actividad con grupos de administración" border="false":::

Si observa las consultas en los grupos de administración fuera de Azure Portal, el ámbito de destino de los grupos de administración se parece a **"/providers/Microsoft.Management/managementGroups/{yourMgID}"** .

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los grupos de administración, consulte:

- [Creación de grupos de administración para organizar los recursos de Azure](./create.md)
- [Cambio, eliminación y administración de los grupos de administración](./manage.md)
- Consulte las opciones de [Procedimiento para proteger la jerarquía de recursos](./how-to/protect-resource-hierarchy.md)