---
title: Preguntas más frecuentes sobre Azure DevTest Labs | Microsoft Docs
description: En este artículo se responden algunas de las preguntas más frecuentes (P+F) relativas a Azure DevTest Labs.
ms.topic: article
ms.date: 07/17/2020
ms.openlocfilehash: 3f36b8238ccb69b3b1f14166b522e47d5debe54e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289369"
---
# <a name="azure-devtest-labs-faq"></a>Preguntas más frecuentes sobre Azure DevTest Labs
En este artículo se ofrecen respuestas a algunas de las preguntas más frecuentes sobre Azure DevTest Labs.

## <a name="blog-post"></a>Entrada de blog
Nuestro blog del equipo de DevTest Labs lleva retirado desde el 20 de marzo de 2019. 

### <a name="where-can-i-track-feature-updates-from-now-on"></a>¿Dónde puedo realizar un seguimiento de las actualizaciones de características a partir de ahora?
De ahora en adelante, publicaremos las actualizaciones de características y las entradas de blog informativas en el blog de Azure y en las actualizaciones de Azure. Estas entradas de blog también se vincularán a nuestra documentación siempre que sea necesario.

Suscríbase al [blog de Azure DevTest Labs](https://azure.microsoft.com/blog/tag/azure-devtest-labs/) y a las [actualizaciones de Azure DevTest Labs](https://azure.microsoft.com/updates/?product=devtest-lab) para estar al tanto de las nuevas características de DevTest Labs.

### <a name="what-happens-to-the-existing-blog-posts"></a>¿Qué ocurre con las entradas de blog existentes?
Actualmente trabajamos en la migración de las entradas de blog existentes (excepto las actualizaciones de interrupciones) a nuestra [documentación de DevTest Labs](devtest-lab-overview.md). Si el blog de MSDN está en desuso, se redirigirá a la información general sobre la documentación para DevTest Labs. Una vez redirigido, podrá buscar el artículo deseado en el título "Filtrar por". Aún no hemos migrado todas las publicaciones, pero debemos hacerlo antes de finalizar este mes. 


### <a name="where-do-i-see-outage-updates"></a>¿Dónde se pueden ver las actualizaciones de interrupciones?
Publicaremos las actualizaciones de interrupciones con nuestro identificador de Twitter de ahora en adelante. Síganos en Twitter para obtener las últimas actualizaciones sobre interrupciones y errores conocidos.

### <a name="twitter"></a>Twitter
Nuestro identificador de Twitter: [@azlabservices](https://twitter.com/azlabservices)

## <a name="general"></a>General
### <a name="what-if-my-question-isnt-answered-here"></a>Mi pregunta no está respondida aquí. ¿Qué debo hacer?
Si su pregunta no aparece aquí, háganoslo saber para que podamos ayudarlo a encontrar una respuesta.

- Publique una pregunta al final de este documento.
- Para llegar a un público más amplio, publique una pregunta en la [página de preguntas y respuestas de Microsoft sobre Azure DevTest Labs](/answers/topics/azure-devtestlabs.html). Póngase en contacto con el equipo de Azure DevTest Labs y otros miembros de la comunidad.
- Para solicitudes de características, envíe sus solicitudes e ideas al sitio [Azure DevTest Labs User Voice](https://feedback.azure.com/forums/320373-azure-devtest-labs).

### <a name="what-is-a-microsoft-account"></a>¿Qué es una cuenta Microsoft?
Usa una cuenta de Microsoft para casi todo lo que hace con los servicios y los dispositivos de Microsoft. Es una dirección de correo electrónico y una contraseña que utiliza para iniciar sesión en Skype, Outlook.com, OneDrive, Windows Phone y Xbox Live. Tener una sola cuenta significa que puede disponer de los archivos, fotografías, contactos y configuración desde cualquier dispositivo.

> [!NOTE]
> La cuenta de Microsoft antes se llamaba Windows Live ID.

### <a name="why-should-i-use-azure-devtest-labs"></a>¿Por qué debería usar Azure DevTest Labs?
Azure DevTest Labs puede ahorrarle a su equipo tiempo y dinero. Los desarrolladores pueden crear sus propios entornos usando varias bases de datos diferentes. También puede utilizar artefactos para implementar y configurar aplicaciones rápidamente. Mediante el uso de fórmulas e imágenes personalizadas, puede guardar máquinas virtuales (VM) como plantillas y reproducirlas fácilmente en el equipo. DevTest Labs ofrece también varias directivas configurables que permiten a los administradores de los laboratorios reducir los residuos y administrar los entornos de un equipo. Estas directivas incluyen el apagado automático, el umbral de costo, el número máximo de máquinas virtuales por usuario y el tamaño máximo de cada máquina virtual. Para ver una explicación más detallada de DevTest Labs, lea la [información general](devtest-lab-overview.md) o consulte el [vídeo de introducción](https://channel9.msdn.com/Blogs/Azure/what-is-azure-devtest-labs).

### <a name="what-does-worry-free-self-service-mean"></a>¿Qué significa "autoservicio sin preocupaciones"?
Autoservicio sin preocupaciones significa que los desarrolladores y evaluadores crean sus propios entornos según sea necesario. Los administradores tienen la seguridad de que DevTest Labs puede ayudar a establecer el acceso adecuado, optimizar los recursos y controlar los costos. Los administradores pueden especificar qué tamaños de máquina virtual se permiten, el número máximo de máquinas virtuales y cuándo iniciar y apagar las máquinas virtuales. DevTest Labs también permite supervisar fácilmente los costos y definir alertas para ayudarle a estar al tanto de cómo se usan los recursos de laboratorios.

### <a name="how-can-i-use-devtest-labs"></a>¿Cómo se usa DevTest Labs?
DevTest Labs es útil siempre que necesite desarrollar o probar entornos y quiera reproducirlos rápidamente o administrarlos con directivas de ahorro de costos.
Estos son algunos escenarios en los que nuestros clientes usan DevTest Labs:

- Administración de entornos de desarrollo y pruebas en un solo lugar. Uso de directivas para reducir los costos y crear imágenes personalizadas para compartir en un equipo de personas.
- Desarrollo de una aplicación con imágenes personalizadas para guardar el estado del disco en todas las etapas del desarrollo.
- Seguimiento del costo en relación con el progreso.
- Creación de entornos de prueba masivos para pruebas de garantía de calidad.
- Uso de artefactos y fórmulas para configurar y reproducir fácilmente una aplicación en diversos entornos.
- Distribución de máquinas virtuales para encuentros de programadores (hackathons) (trabajo de desarrollo o prueba en colaboración) y desaprovisionamiento fácil de estas máquinas cuando el evento finaliza.

### <a name="how-am-i-billed-for-devtest-labs"></a>¿Cómo se factura DevTest Labs?
DevTest Labs es un servicio gratuito. Crear laboratorios y configurar las directivas, las plantillas y los artefactos en DevTest Labs no tiene ningún costo. Solo se paga por los recursos de Azure usados en los laboratorios, como máquinas virtuales, cuentas de almacenamiento y redes virtuales. Para más información sobre el costo de los recursos de laboratorio, consulte los [precios de Azure DevTest Labs](https://azure.microsoft.com/pricing/details/devtest-lab/).

## <a name="security"></a>Seguridad

### <a name="what-are-the-different-security-levels-in-devtest-labs"></a>¿Cuáles son los diferentes niveles de seguridad en DevTest Labs?
El acceso de seguridad viene determinado por el Control de acceso basado en rol (RBAC). Para aprender cómo funciona el acceso, es importante comprender las diferencias entre un permiso, un rol y un ámbito según se define en RBAC.

- **Permiso**: un permiso es un acceso definido para una acción específica. Por ejemplo, un permiso podría ser el acceso de lectura a todas las máquinas virtuales.
- **Rol**: un rol es un conjunto de permisos que se pueden agrupar y asignar a un usuario. Por ejemplo, un usuario con el rol de propietario de la suscripción tiene acceso a todos los recursos dentro de una suscripción.
- **Ámbito**: un ámbito es un nivel dentro de la jerarquía de recursos de Azure. Por ejemplo, un ámbito puede ser un grupo de recursos, un único laboratorio o toda la suscripción.

Dentro del ámbito de DevTest Labs, hay dos tipos de roles para definir los permisos de usuario:

- **Propietario de laboratorio**: tiene acceso a todos los recursos del laboratorio. El propietario de un laboratorio puede modificar las directivas, leer y escribir en todas las máquinas virtuales, cambiar la red virtual, etc.
- **Usuario de laboratorio**: puede ver todos los recursos del laboratorio, como las máquinas virtuales, las directivas y las redes virtuales. Sin embargo, un usuario de laboratorio no puede modificar las directivas ni ninguna máquina virtual creada por otros usuarios.

También puede crear roles personalizados en DevTest Labs. Para aprender a crear roles personalizados en DevTest Labs, consulte [Concesión de permisos de usuario a directivas específicas de laboratorio](devtest-lab-grant-user-permissions-to-specific-lab-policies.md).

Puesto que los ámbitos son jerárquicos, cuando un usuario tiene permisos en un ámbito determinado, también se le conceden automáticamente en cada ámbito de nivel inferior. Por ejemplo, si un usuario está asignado al rol de propietario de la suscripción, tiene acceso a todos los recursos de una suscripción. Estos recursos incluyen máquinas virtuales, redes virtuales y laboratorios. El propietario de una suscripción hereda automáticamente el rol de propietario de laboratorio. Sin embargo, lo contrario no es cierto. El propietario de un laboratorio tiene acceso a un laboratorio, que es un ámbito inferior al del nivel de suscripción. Por lo tanto, no puede ver las máquinas virtuales, las redes virtuales ni ningún otro recurso que esté fuera del laboratorio.

### <a name="how-do-i-define-role-based-access-control-for-my-devtest-labs-environments-to-ensure-that-it-can-govern-while-developerstest-can-do-their-work"></a>¿Cómo puedo definir el control de acceso basado en roles para mis entornos DevTest Labs para asegurarme de que el departamento de TI puede regir al mismo tiempo que los desarrolladores y probadores pueden hacer su trabajo?
Hay un amplio modelo, sin embargo, habrá detalles específicos que dependerán de su organización.

El departamento de TI central debe poseer solo lo que es necesario y permitir que los equipos del proyecto y la aplicación tengan el nivel necesario de control. Normalmente, esto significa que esa departamento de TI central posee la suscripción y las controla las funciones de TI básicas, como las configuraciones de redes. El conjunto de **propietarios** para una suscripción debe ser pequeño. Estos propietarios pueden designar a propietarios adicionales cuando sea necesario, o aplicar directivas de nivel de suscripción, por ejemplo “sin dirección IP pública”.

Puede haber un subconjunto de usuarios que requieren acceso a través de una suscripción, como la compatibilidad con Tier1 o Tier2. En este caso, se recomienda dar a dichos usuarios acceso de **colaborador** para que puedan administrar los recursos, pero no puedan proporcionar acceso de usuario o ajustar las directivas.

Los propietarios del recurso de DevTest Labs deben estar cerca del equipo de proyecto o aplicación. De esta forma entenderán sus requisitos para las máquinas y el software necesario. En la mayoría de las organizaciones, el propietario de este recurso de DevTest Labs suele ser el responsable de desarrollo o del proyecto. Este propietario puede administrar los usuarios y las directivas en el entorno de laboratorio y puede administrar todas las máquinas virtuales en el entorno de DevTest Labs.

Los miembros del equipo de proyecto o aplicación se deben agregar al rol de **usuarios de DevTest Labs**. Estos usuarios pueden crear máquinas virtuales (en línea con las directivas de nivel de suscripción y de laboratorio). También pueden administrar sus propias máquinas virtuales. No pueden administrar las máquinas virtuales que pertenecen a otros usuarios.

Para obtener más información vea [la documentación de Scaffold empresarial de Azure: gobernanza de suscripción prescriptiva](/azure/architecture/cloud-adoption/appendix/azure-scaffold).


### <a name="how-do-i-create-a-role-to-allow-users-to-do-a-specific-task"></a>¿Cómo se crea un rol para permitir que los usuarios realicen una tarea específica?
Para un artículo completo sobre cómo crear roles personalizados y asignar permisos a un rol, consulte [Concesión de permisos de usuario para directivas específicas de laboratorio](devtest-lab-grant-user-permissions-to-specific-lab-policies.md). Este es un ejemplo de script que crea el rol **Usuario avanzado de DevTest Labs**, que tiene permiso para iniciar y detener todas las máquinas virtuales del laboratorio:


```powershell
$policyRoleDef = Get-AzRoleDefinition "DevTest Labs User"
$policyRoleDef.Actions.Remove('Microsoft.DevTestLab/Environments/*')
$policyRoleDef.Id = $null
$policyRoleDef.Name = "DevTest Labs Advanced User"
$policyRoleDef.IsCustom = $true
$policyRoleDef.AssignableScopes.Clear()
$policyRoleDef.AssignableScopes.Add("subscriptions/<subscription Id>")
$policyRoleDef.Actions.Add("Microsoft.DevTestLab/labs/virtualMachines/Start/action")
$policyRoleDef.Actions.Add("Microsoft.DevTestLab/labs/virtualMachines/Stop/action")
$policyRoleDef = New-AzRoleDefinition -Role $policyRoleDef  
```

### <a name="how-can-an-organization-ensure-corporate-security-policies-are-in-place"></a>¿Cómo puede una organización asegurarse de que las directivas de seguridad corporativas están en vigor?
Una organización puede lograrlo realizando las siguientes acciones:

- Desarrollar y publicar una directiva de seguridad integral. La directiva articula las reglas de uso aceptable asociadas con el uso de recursos de nube y software. También define lo que claramente infringe la directiva.
- Desarrollar una imagen personalizada, artefactos personalizados y un proceso de implementación que permita la orquestación dentro del ámbito de la seguridad que se define con el directorio activo. Este enfoque refuerza los límites corporativos y establece un conjunto común de controles ambientales. Estos controles contra el entorno que un desarrollador puede considerar al desarrollar y seguir un ciclo de vida de desarrollo seguro como parte de su proceso general. El objetivo también es proporcionar un entorno que no sea excesivamente restrictivo y que pueda obstaculizar el desarrollo, sino un conjunto razonable de controles. Las directivas de grupo en la unidad organizativa que contiene máquinas virtuales de laboratorio podrían ser un subconjunto del total de directivas de grupo que se encuentran en producción. Además, pueden ser un conjunto adicional para mitigar adecuadamente cualquier riesgo identificado.

### <a name="how-can-an-organization-ensure-data-integrity-to-ensure-that-remoting-developers-cant-remove-code-or-introduce-malware-or-unapproved-software"></a>¿Cómo puede una organización garantizar la integridad de datos para asegurar que los desarrolladores remotos no puedan quitar código o introducir malware o software no aprobado?
Hay varias capas de control para mitigar la amenaza de consultores externos, contratistas o empleados remotos para colaborar en DevTest Labs.

Como se dijo anteriormente, el primer paso debe tener una directiva de uso aceptable redactada y definida que describa claramente las consecuencias cuando alguien infringe la directiva.

La primera capa de controles para el acceso remoto es aplicar una directiva de acceso remoto mediante una conexión VPN que no esté conectada directamente al laboratorio.

La segunda capa de controles consiste en aplicar un conjunto de objetos de directiva de grupo que impiden copiar y pegar mediante el escritorio remoto. Se podría implementar una directiva de red para no permitir que los servicios salientes del entorno, como los servicios FTP y RDP, queden fuera del entorno. El enrutamiento definido por el usuario podría obligar a todo el tráfico de red de Azure de nuevo a un entorno local, pero el enrutamiento no podría dar cuenta de todas las direcciones URL que podrían permitir la carga de datos a menos que se controlen mediante un proxy para escanear el contenido y las sesiones. Las direcciones IP públicas podrían restringirse dentro de la red virtual que admite DevTest Labs para no permitir el protocolo de puente de un recurso de red externo.

En última instancia, es necesario aplicar el mismo tipo de restricciones en toda la organización, lo que tendría en cuenta todos los métodos posibles de medios extraíbles o direcciones URL externas que podrían aceptar un mensaje de contenido. Consulte con el profesional de seguridad para revisar e implementar una directiva de seguridad. Para más recomendaciones, consulte [Ciberseguridad de Microsoft](https://www.microsoft.com/security/default.aspx?&WT.srch=1&wt.mc_id=AID623240_SEM_sNYnsZDs).

## <a name="lab-configuration"></a>Configuración del laboratorio

### <a name="how-do-i-create-a-lab-from-a-resource-manager-template"></a>¿Cómo se crea un laboratorio a partir de una plantilla de Resource Manager?
Hemos ofrecido un [repositorio de GitHub de plantillas de Azure Resource Manager para laboratorios](https://azure.microsoft.com/resources/templates/101-dtl-create-lab) que se pueden implementar tal cual o modificar para crear plantillas personalizadas para los laboratorios. Cada plantilla tiene un vínculo para implementar el laboratorio tal cual está en su propia suscripción de Azure. O bien, puede personalizar la plantilla e [implementarla con PowerShell o la CLI de Azure](../azure-resource-manager/templates/deploy-powershell.md).


### <a name="can-i-have-all-virtual-machines-to-be-created-in-a-common-resource-group-instead-having-each-machine-in-its-own-resource-group"></a>¿Es posible tener todas las máquinas virtuales que se van a crear en un grupo de recursos común en lugar de tener cada máquina en su propio grupo de recursos?
Sí, como propietario de laboratorio, puede permitir que el laboratorio controle la asignación del grupo de recursos automáticamente o tener todas las máquinas virtuales creadas en un grupo de recursos común que especifique.

Escenario del grupo de recursos independiente:
-   DevTest Labs crea un nuevo grupo de recursos para todas las máquinas virtuales de IP pública o privada que establezca.
-   DevTest Labs crea un grupo de recursos para máquinas de IP compartida que pertenecen al mismo tamaño.

Escenario del grupo de recursos común:
-   Todas las máquinas virtuales se establecen en el grupo de recursos común que especifique. Obtenga más información en el artículo sobre la [asignación del grupo de recursos para el laboratorio](https://aka.ms/RGControl).

### <a name="how-do-i-maintain-a-naming-convention-across-my-devtest-labs-environment"></a>¿Cómo se puede mantener una convención de nomenclatura en un entorno de DevTest Labs?
Es posible que desee ampliar las convenciones de nomenclatura empresariales actuales para las operaciones de Azure y que sean coherentes en todo el entorno de DevTest Labs. Al implementar DevTest Labs, se recomienda que tenga directivas específicas de partida. Implemente estas directivas mediante plantillas JSON y un script central para garantizar la coherencia. Las directivas de nomenclatura se pueden implementar mediante directivas de Azure aplicadas a nivel de suscripción. Para consultar ejemplos de JSON para Azure Policy, vea [Ejemplos de Azure Policy](../governance/policy/samples/index.md).

### <a name="how-many-labs-can-i-create-under-the-same-subscription"></a>¿Cuántos laboratorios se pueden crear con una misma suscripción?
No hay ningún límite específico en el número de laboratorios que se pueden crear por cada suscripción. Sin embargo, la cantidad de recursos que se usan para cada suscripción está limitada. Puede leer sobre los [límites y las cuotas de las suscripciones de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md) y [cómo aumentar estos límites](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests).


### <a name="how-many-vms-can-i-create-per-lab"></a>¿Cuántas máquinas virtuales se pueden crear por laboratorio?
No hay ningún límite específico en el número de máquinas virtuales que se pueden crear por cada laboratorio. Sin embargo, los recursos (los núcleos de máquinas virtuales, las direcciones IP públicas, etc) están limitados para cada suscripción. Puede leer sobre los [límites y las cuotas de las suscripciones de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md) y [cómo aumentar estos límites](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests).

### <a name="how-do-i-determine-the-ratio-of-users-per-lab-and-the-overall-number-of-labs-that-are-needed-across-an-organization"></a>¿Cómo se puede determinar la proporción de usuarios por laboratorio y el número total de laboratorios necesarios en toda la organización?
Se recomienda que las unidades de negocio y los grupos de desarrollo que están asociados con el mismo proyecto de desarrollo estén asociados con el mismo laboratorio. Esto permite aplicar a ambos grupos los mismos tipos de directivas, imágenes y directivas de cierre.

También debe tener en cuenta los límites geográficos. Por ejemplo, los desarrolladores del Nordeste de Estados Unidos (EE. UU.) pueden usar un laboratorio aprovisionado en el Este de EE. UU. 2. Además, los desarrolladores de Dallas (Texas) y Denver (Colorado) pueden ser remitidos a usar un recurso en Centro y Sur de EE. UU. Si existe algún trabajo de colaboración con un tercero externo, se les podría asignar a un laboratorio que no utilizan los desarrolladores internos.

También puede usar un laboratorio para un proyecto específico en los proyectos de Azure DevOps. A continuación, aplique seguridad en un grupo específico de Azure Active Directory, lo que permite acceder a ambos conjuntos de recursos. La red virtual asignada al laboratorio puede ser otro límite para consolidar a los usuarios.

### <a name="how-can-we-prevent-the-deletion-of-resources-within-a-lab"></a>¿Cómo se puede evitar la eliminación de recursos de un laboratorio?
Se recomienda establecer los permisos adecuados a nivel de laboratorio para que solo los usuarios autorizados puedan eliminar los recursos o cambiar las directivas del laboratorio. Los desarrolladores deben incorporarse al grupo de **usuarios de DevTest Labs**. El jefe de desarrollo o el responsable de la infraestructura debe ser el **propietario de DevTest Labs**. Se recomienda que haya solo dos propietarios del laboratorio. Esta directiva se extiende al repositorio de código para evitar daños. Los usuarios del laboratorio tienen derechos para usar los recursos, pero no pueden actualizar las directivas del laboratorio. Consulte el artículo siguiente en el que se enumeran los roles y derechos que cada grupo integrado tiene dentro de un laboratorio: [Adición de propietarios y usuarios en Azure DevTest Labs](devtest-lab-add-devtest-user.md).

### <a name="how-do-i-share-a-direct-link-to-my-lab"></a>¿Cómo se puede compartir un vínculo directo a mi laboratorio?

1. En [Azure Portal](https://portal.azure.com), vaya al laboratorio.
2. Copie la **dirección URL del laboratorio** desde el explorador y compártala con los usuarios del laboratorio.

> [!NOTE]
> Si un usuario del laboratorio es externo y tiene una cuenta de Microsoft, pero no es un miembro de la instancia de Active Directory de su organización, podría ver un mensaje de error cuando intente tener acceso al vínculo compartido. Si un usuario externo ve un mensaje de error, pídale que seleccione primero su nombre en la esquina superior derecha de Azure Portal. A continuación, en la sección Directorio del menú, el usuario puede seleccionar el directorio donde se encuentra el laboratorio.

## <a name="virtual-machines"></a>Máquinas virtuales

### <a name="why-cant-i-see-vms-on-the-virtual-machines-page-that-i-see-in-devtest-labs"></a>¿Por qué no se pueden ver las máquinas virtuales en la página Máquinas virtuales que se ven en DevTest Labs?
Cuando crea una máquina virtual en DevTest Labs, se le otorgan permisos para acceder a dicha máquina virtual. Puede ver la máquina virtual tanto en la página de laboratorios como en la página **Máquinas virtuales**. Los usuarios asignados al rol de **propietario de DevTest Labs** pueden ver todas las máquinas virtuales creadas en el laboratorio en la página **Todas las máquinas virtuales** de este. No obstante, a los usuarios que tengan el rol de **usuario de DevTest Labs** no se les concede automáticamente acceso de lectura a los recursos de máquina virtual creados por otros usuarios. Por lo tanto, esas máquinas virtuales no se muestran en la página **Máquinas virtuales**.


### <a name="how-do-i-create-multiple-vms-from-the-same-template-at-once"></a>¿Cómo se crean varias máquinas virtuales desde la misma plantilla a la vez?
Tiene dos opciones para crear al mismo tiempo varias máquinas virtuales desde la misma plantilla:

- Puede usar la [extensión Tareas de Azure DevOps](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks).
- Puede [generar una plantilla de Resource Manager](devtest-lab-add-vm.md#save-azure-resource-manager-template) mientras crea una máquina virtual e [implementar la plantilla de Resource Manager desde Windows PowerShell](../azure-resource-manager/templates/deploy-powershell.md).
- También puede especificar más de una instancia de una máquina que se va a crear durante la creación de la máquina virtual. Para obtener más información sobre cómo crear varias instancias de máquinas virtuales, consulte la documentación sobre cómo [crear una máquina virtual de laboratorio](devtest-lab-add-vm.md).

### <a name="how-do-i-move-my-existing-azure-vms-into-my-devtest-labs-lab"></a>¿Cómo se pueden mover las máquinas virtuales de Azure existentes a mi laboratorio de DevTest Labs?
Para copiar las máquinas virtuales existentes a los laboratorios de DevTest Labs:

1.  Copie el archivo VHD de la máquina virtual existente mediante un [script de Windows PowerShell](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/Scripts/CopyVirtualMachines/CopyAzVHDFromVMToLab.ps1).
2.  Cree la [imagen personalizada](devtest-lab-create-template.md) dentro del laboratorio de DevTest Labs.
3.  Cree una máquina virtual en el laboratorio a partir de su imagen personalizada.

### <a name="can-i-attach-multiple-disks-to-my-vms"></a>¿Se pueden conectar varios discos a las máquinas virtuales?

Sí, puede adjuntar varios discos a sus máquinas virtuales.

### <a name="are-gen-2-images-supported-by-devtest-labs"></a>¿Admite DevTest Labs imágenes de Gen 2?
Sí. El servicio DevTest Labs admite [imágenes de Gen 2](../virtual-machines/windows/generation-2.md). Sin embargo, si hay versiones de Gen 1 y Gen 2 disponibles para una imagen, DevTest Labs solo muestra la versión de Gen 1 de la imagen al crear una máquina virtual. Verá la imagen si solo está disponible su versión de Gen 2. 

### <a name="if-i-want-to-use-a-windows-os-image-for-my-testing-do-i-have-to-purchase-an-msdn-subscription"></a>Si quiero usar una imagen de sistema operativo Windows para mi prueba, ¿tengo que comprar una suscripción a MSDN?
Para usar imágenes de sistema operativo cliente de Windows (Windows 7 o una versión posterior) para el desarrollo o las pruebas en Azure, siga uno de los siguientes pasos:

- [Adquirir una suscripción a MSDN](https://www.visualstudio.com/products/how-to-buy-vs).
- Si tiene un Contrato Enterprise, crear una suscripción a Azure con la [oferta Desarrollo/pruebas - Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p).

Para obtener más información acerca de los créditos de Azure para cada oferta de MSDN, consulte [Crédito mensual de Azure para suscriptores de Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/).


### <a name="how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab"></a>¿Cómo se automatiza el proceso de eliminación de todas las máquinas virtuales en mi laboratorio?
Como propietario de laboratorio, puede eliminar las máquinas virtuales en el laboratorio en Azure Portal. También puede eliminar todas las máquinas virtuales en el laboratorio mediante un script de PowerShell. En el ejemplo siguiente, en el comentario **Values to change** (Valores para cambiar), modifique los valores del parámetro. Puede recuperar los valores `subscriptionId`, `labResourceGroup` y `labName` del panel del laboratorio en Azure Portal.

```powershell
# Delete all the VMs in a lab.

# Values to change:
$subscriptionId = "<Enter Azure subscription ID here>"
$labResourceGroup = "<Enter lab's resource group here>"
$labName = "<Enter lab name here>"

# Sign in to your Azure account.
Connect-AzAccount

# Select the Azure subscription that has the lab. This step is optional
# if you have only one subscription.
Select-AzSubscription -SubscriptionId $subscriptionId

# Get the lab that has the VMs that you want to delete.
$lab = Get-AzResource -ResourceId ('subscriptions/' + $subscriptionId + '/resourceGroups/' + $labResourceGroup + '/providers/Microsoft.DevTestLab/labs/' + $labName)

# Get the VMs from that lab.
$labVMs = Get-AzResource | Where-Object {
          $_.ResourceType -eq 'microsoft.devtestlab/labs/virtualmachines' -and
          $_.Name -like "$($lab.Name)/*"}

# Delete the VMs.
foreach($labVM in $labVMs)
{
    Remove-AzResource -ResourceId $labVM.ResourceId -Force
}
```

## <a name="environments"></a>Entornos

### <a name="how-can-i-use-resource-manager-templates-in-my-devtest-labs-environment"></a>¿Cómo se pueden usar plantillas de Resource Manager en un entorno de DevTest Labs?
Para implementar las plantillas de Resource Manager en un entorno de DevTest Labs, aplique los pasos mencionados en el artículo sobre la [característica de entornos de DevTest Labs](devtest-lab-test-env.md). Básicamente, consulte las plantillas de Resource Manager en un repositorio de GIT (Azure Repos o GitHub) y agregue un [repositorio privado para las plantillas](devtest-lab-test-env.md) al laboratorio. Este escenario puede no ser útil si usa DevTest Labs para hospedar máquinas de desarrollo, pero sí puede serlo si va a crear un entorno de ensayo, que es representativo de producción.

También merece la pena tener en cuenta que el número de máquinas virtuales por laboratorio o por opción de usuario solo limita el número de máquinas creadas de forma nativa en el propio laboratorio y no según el entorno (plantillas de Resource Manager).

## <a name="custom-images"></a>Custom Images

### <a name="how-can-i-set-up-an-easily-repeatable-process-to-bring-my-custom-organizational-images-into-a-devtest-labs-environment"></a>¿Cómo se configura un proceso repetible fácilmente para incorporar imágenes de la organización personalizadas en un entorno de DevTest Labs?
Vea este [vídeo sobre el patrón de un generador de imágenes](https://sec.ch9.ms/ch9/8e8a/9ea0b8d4-b803-4f23-bca4-4808d9368e8a/dtlimagefactory_mid.mp4). Este escenario es un escenario avanzado, y los scripts se proporcionan solo como ejemplo. Si se requieren cambios, deberá administrar y mantener los scripts usados en su entorno.

Para obtener información detallada sobre cómo crear un generador de imágenes, consulte [Creación de un generador de imágenes personalizadas en Azure DevTest Labs](image-factory-create.md).

### <a name="what-is-the-difference-between-a-custom-image-and-a-formula"></a>¿Cuál es la diferencia entre una imagen personalizada y una fórmula?
Una imagen personalizada es una imagen administrada. Una fórmula es una imagen que puede configurar con opciones adicionales y, a continuación, guardar y reproducir. Si lo que quiere es crear rápidamente varios entornos con la misma imagen básica inmutable, puede ser preferible una imagen personalizada. En cambio, si desea reproducir la configuración de la máquina virtual con los últimos bits, como parte de una red o una subred virtual, o como máquina virtual de un tamaño específico, es mejor una fórmula. Para una explicación más detallada, consulte [Comparación de imágenes personalizadas y fórmulas en DevTest Labs](devtest-lab-comparing-vm-base-image-types.md).

### <a name="when-should-i-use-a-formula-vs-custom-image"></a>¿Cuándo debe usar una fórmula en lugar de una imagen personalizada?
Normalmente, los factores decisivos en este escenario son el costo y la reutilización. Si tiene un escenario donde muchos usuarios o laboratorios requieren una imagen con una gran cantidad de software sobre la imagen base, podría reducir los costos mediante la creación de una imagen personalizada. Esto significa que la imagen se crea una vez. Reduce el tiempo de configuración de la máquina virtual y el costo en que se incurre por la ejecución de la máquina virtual durante la configuración.

Sin embargo, otro factor que se debe tener en cuenta es la frecuencia de los cambios realizados en el paquete de software. Si ejecuta compilaciones diariamente y requiere que el software se encuentre en las máquinas virtuales de los usuarios, considere el uso de una fórmula en lugar de una imagen personalizada.

Para una explicación más detallada, consulte [Comparación de imágenes personalizadas y fórmulas](devtest-lab-comparing-vm-base-image-types.md) en DevTest Labs.

### <a name="how-do-i-automate-the-process-of-uploading-vhd-files-to-create-custom-images"></a>¿Cómo se puede automatizar el proceso de carga de archivos VHD para crear imágenes personalizadas?

Para automatizar la carga de archivos VHD para crear imágenes personalizadas, dispone de dos opciones:

- Usar [AzCopy](../storage/common/storage-use-azcopy-v10.md) para copiar o cargar archivos VHD en la cuenta de almacenamiento asociada al laboratorio.
- Usar el [Explorador de Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md). Explorador de Azure Storage es una aplicación independiente que se ejecuta en Windows, OS X y Linux.

Para buscar la cuenta de almacenamiento de destino asociada al laboratorio:

1.  Inicie sesión en [Azure Portal](https://portal.azure.com).
2.  En el menú de la izquierda, seleccione **Grupos de recursos**.
3.  Busque y seleccione el grupo de recursos asociado al laboratorio.
4.  En **Información general**, seleccione una de las cuentas de almacenamiento.
5.  Seleccione **Blobs**.
6.  Busque cargas en la lista. Si no existe ninguna, vuelva al paso 4 y pruebe con otra cuenta de almacenamiento.
7.  Use la **dirección URL** como destino del comando AzCopy.

¿Cuándo debo usar una imagen de Azure Marketplace en lugar de una imagen de la organización personalizada propia?

### <a name="when-should-i-use-an-azure-marketplace-image-vs-my-own-custom-organizational-image"></a>¿Cuándo debo usar una imagen de Azure Marketplace en lugar de una imagen de la organización personalizada propia?
Azure Marketplace se debe usar de forma predeterminada a menos que tenga requisitos organizativos o preocupaciones específicos. Algunos ejemplos frecuentes son:

- Instalación compleja del software que requiere que una aplicación se incluya como parte de la imagen base.
- La instalación y configuración de una aplicación podrían tardar muchas horas, lo que no constituye un uso eficaz del tiempo de proceso que se agregará a una imagen de Azure Marketplace.
- Los desarrolladores y evaluadores necesitan tener acceso a una máquina virtual rápidamente y desean minimizar el tiempo de configuración de una nueva máquina virtual.
- Existencia de condiciones de cumplimiento o normativas (por ejemplo, las directivas de seguridad) para todas las máquinas.
- El uso de imágenes personalizadas no se debe considerar a la ligera. Generan una complejidad adicional, ya que ahora debe administrar archivos de disco duro virtual para las imágenes base subyacentes. También debe revisar periódicamente esas imágenes base con las actualizaciones de software. Estas actualizaciones incluyen nuevas actualizaciones del sistema operativo (SO) y las actualizaciones o los cambios de configuración necesarios para el paquete de software.

## <a name="artifacts"></a>Artefactos

### <a name="what-are-artifacts"></a>¿Qué son los artefactos?
Los artefactos son elementos personalizables que puede usar para implementar los bits más recientes o herramientas de desarrollo en una máquina virtual. Asocie los artefactos a la máquina virtual al crearla. Después de aprovisionar la máquina virtual, los artefactos la implementan y configuran. Varios artefactos preexistentes están disponibles en nuestro [repositorio público de GitHub](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts). También puede [crear sus propios artefactos](devtest-lab-artifact-author.md).

### <a name="my-artifact-failed-during-vm-creation-how-do-i-troubleshoot-it"></a>Mi artefacto produjo errores durante la creación de la máquina virtual. ¿Cómo se soluciona este problema?
Para aprender a obtener registros de los artefactos con error, consulte [How to diagnose artifact failures in DevTest Labs](devtest-lab-troubleshoot-artifact-failure.md) (Diagnóstico de errores de artefactos en DevTest Labs).

### <a name="when-should-an-organization-use-a-public-artifact-repository-vs-private-artifact-repository-in-devtest-labs"></a>¿Cuándo debería una organización utilizar un repositorio de artefactos público frente a un repositorio de artefactos privado en DevTest Labs?
El [repositorio de artefactos público](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts) proporciona un conjunto inicial de paquetes de software que se utilizan con mayor frecuencia. Ayuda a una rápida implementación sin tener que invertir tiempo en reproducir herramientas y complementos comunes para desarrolladores. Puede elegir implementar su propio repositorio privado. Puede utilizar un repositorio público y uno privado en paralelo. También puede elegir deshabilitar el repositorio público. Los criterios para implementar un repositorio privado deben estar guiados por las siguientes preguntas y consideraciones:

- ¿Tiene la organización el requisito de tener un software corporativo con licencia como parte de su oferta de DevTest Labs? Si la respuesta es afirmativa, entonces se debe crear un repositorio privado.
- ¿Desarrolla la organización software personalizado que proporciona una operación específica, que se requiere como parte del proceso general de aprovisionamiento? Si la respuesta es afirmativa, se debe implementar un repositorio privado.
- Si la directiva de gobernanza de la organización requiere aislamiento y los repositorios externos no están bajo la administración directa de la configuración por parte de la organización, se debe implementar un repositorio de artefactos privado. Como parte de este proceso, una copia inicial del repositorio público se puede copiar e integrar con el repositorio privado. Después, el repositorio público se puede deshabilitar para que nadie dentro de la organización pueda acceder a él. Este enfoque obliga a todos los usuarios dentro de la organización a tener un único repositorio aprobado por ella y minimizar la deriva de la configuración.


### <a name="should-an-organization-plan-for-a-single-repository-or-allow-multiple-repositories"></a>¿Debería una organización planear para un único repositorio o permitir múltiples repositorios?
Como parte de la estrategia general de gobernanza y administración de configuración de la organización, le recomendamos que utilice un repositorio centralizado. Cuando utiliza varios repositorios, pueden convertirse en silos de software no administrado con el tiempo. Con un repositorio central, múltiples equipos pueden consumir artefactos de este repositorio para sus proyectos. Impone la estandarización, la seguridad, la facilidad de administración y elimina la duplicación de esfuerzos. Como parte de la centralización, las siguientes acciones son procedimientos recomendados para la administración a largo plazo y la sostenibilidad:

- Asocie los servicios de Azure Repos con el mismo inquilino de Azure Active Directory que la suscripción de Azure está utilizando para la autenticación y autorización.
- Cree un grupo llamado `All DevTest Labs Developers` en Azure Active Directory que se administre de forma centralizada. Cualquier desarrollador que contribuya al desarrollo de artefacto debe colocarse en este grupo.
- El mismo grupo de Azure Active Directory puede usarse para proporcionar acceso al repositorio de Azure Repos y al laboratorio.
- En Azure Repos, las ramificaciones o bifurcaciones deben utilizarse para separar un repositorio en desarrollo del repositorio de producción principal. El contenido solo se agrega a la rama maestra con una solicitud de incorporación de cambios después de una revisión apropiada del código. Una vez que el revisor de código aprueba el cambio, un desarrollador jefe, que es responsable del mantenimiento de la rama maestra, combina el código actualizado.

## <a name="cicd-integration"></a>Integración de CI/CD

### <a name="does-devtest-labs-integrate-with-my-cicd-toolchain"></a>¿Se integra DevTest Labs con mi cadena de herramientas de CI/CD?
Si va a utilizar Azure DevOps, puede usar una [extensión Tareas de DevTest Labs](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks) que le permite automatizar la canalización de versión de DevTest Labs. Algunas de las tareas que puede realizar con esta extensión incluyen:

- Crear e implementar una máquina virtual automáticamente. También puede configurar la máquina virtual con la compilación mas reciente mediante las tareas de PowerShell Azure DevOps Services o de Copia de archivos de Azure.
- Capturar automáticamente el estado de una máquina virtual después de intentar reproducir un error en la misma máquina virtual para investigarlo con más detalle.
- Eliminar la máquina virtual al final de la canalización de versión cuando ya no sea necesaria.

Las siguientes entradas de blog proporcionan orientación e información sobre el uso de la extensión Azure DevOps Services:

- [DevTest Labs y la extensión de Azure DevOps](integrate-environments-devops-pipeline.md)
- [Deploy a new VM in an existing DevTest Labs lab from Azure DevOps Services](https://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS) (Implementación de una nueva máquina virtual en un laboratorio de DevTest Labs existente desde Azure DevOps Services)
- [Using Azure DevOps Services release management for continuous deployments to Dev Test Labs](https://www.visualstudiogeeks.com/blog/DevOps/Use-VSTS-ReleaseManagement-to-Deploy-and-Test-in-AzureDevTestLabs) (Uso de la administración de versiones de Azure DevOps Services para implementaciones continuas en Dev Test Labs)

Para otras cadenas de herramientas de entrega continua (CD) o integración continua (CI), puede lograr los mismos escenarios implementando [plantillas de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) mediante [cmdlets de Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) y [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Management.DevTestLabs/). También puede usar las [API de REST para DevTest Labs](https://aka.ms/dtlrestapis) a fin de integrarlas con su cadena de herramientas.

## <a name="networking"></a>Redes

### <a name="when-should-i-create-a-new-virtual-network-for-my-devtest-labs-environment-vs-using-an-existing-virtual-network"></a>¿Cuándo se debe crear una red virtual para un entorno de DevTest Labs en lugar de usar una red virtual existente?
Si las máquinas virtuales necesitan interactuar con la infraestructura existente, considere la posibilidad de utilizar una red virtual existente dentro de su entorno de DevTest Labs. Si usa ExpressRoute, es posible que desee minimizar el número de redes virtuales y subredes para no fragmentar el espacio de direcciones IP que se asigna para su uso en las suscripciones.

Considere la posibilidad de usar aquí también el patrón de emparejamiento de la red virtual ([modelo tipo hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)). Este enfoque permite la comunicación de red virtual o subred entre suscripciones. En caso contrario, cada entorno de DevTest Labs podría tener su propia red virtual.

Hay [límites](../azure-resource-manager/management/azure-subscription-service-limits.md) en el número de redes virtuales por suscripción. La cantidad predeterminada es 50, aunque este límite puede aumentarse a 100.

### <a name="when-should-i-use-a-shared-ip-vs-public-ip-vs-private-ip"></a>¿Cuándo se debe usar una dirección IP compartida en lugar de una pública o privada?

Si usa una VPN de sitio a sitio o ExpressRoute, considere la posibilidad de usar direcciones IP privadas para que se pueda acceder a las máquinas a través de la red interna, pero no a través de Internet.

> [!NOTE]
> Los propietarios de laboratorios pueden cambiar esta directiva de subred para asegurarse de que nadie puede crear accidentalmente direcciones IP públicas para sus máquinas virtuales. El propietario de la suscripción debe crear una directiva de suscripción que impida la creación de direcciones IP públicas.

Si se usan direcciones IP compartidas, las máquinas virtuales del laboratorio comparten una dirección IP pública. Este enfoque puede resultar útil si necesita impedir que se traspasen los límites de las direcciones IP públicas de una suscripción concreta.

### <a name="how-do-i-ensure-that-development-and-test-virtual-machines-are-unable-to-reach-the-public-internet-are-there-any-recommended-patterns-to-set-up-network-configuration"></a>¿Cómo garantizar que las máquinas virtuales de desarrollo y pruebas no sean accesibles a través de la red pública de Internet? ¿Existen patrones recomendados para configurar la red?

Sí. Hay dos aspectos que se deben tener en cuenta: el tráfico entrante y saliente.

- **Tráfico entrante**: si la máquina virtual no tiene una dirección IP pública, no se puede acceder a ella a través de Internet. Un enfoque común consiste en asegurarse de que se establece una directiva de nivel de suscripción, para que ningún usuario pueda crear una dirección IP pública.
- **Tráfico saliente**: si desea evitar que las máquinas virtuales tengan acceso directamente a la red pública de Internet y forzar el tráfico a través de un firewall corporativo, puede enrutar el tráfico local a través de ExpressRoute o VPN, mediante el uso del enrutamiento forzado.

> [!NOTE]
> Si tiene un servidor proxy que bloquea el tráfico sin configuración del proxy, no olvide agregar excepciones a la cuenta de almacenamiento de artefactos del laboratorio.

También puede usar grupos de seguridad de red para máquinas virtuales o subredes. Este paso agrega una capa adicional de protección para permitir o bloquear el tráfico.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="why-isnt-my-existing-virtual-network-saving-properly"></a>¿Por qué mi máquina virtual existente no se guarda correctamente?
Una posibilidad es que el nombre de la red virtual contenga puntos. Si es así, intente quitar los puntos o reemplazarlos con guiones. A continuación, vuelva a intentar guardar la red virtual.

### <a name="why-do-i-get-a-parent-resource-not-found-error-when-i-provision-a-vm-from-powershell"></a>¿Por qué aparece el error "No se encuentra el recurso primario" al aprovisionar una máquina virtual desde PowerShell?
Cuando un recurso es un elemento primario de otro recurso, el primario debe existir antes de crear el secundario. Si el recurso primario no existe, verá un mensaje **ParentResourceNotFound**. Si no se especifica una dependencia del recurso primario, es posible que el recurso secundario se implemente antes que el primario.

Las máquinas virtuales son recursos secundarios en un laboratorio en un grupo de recursos. Cuando se usan plantillas de Resource Manager para la implementación de máquinas virtuales con PowerShell, el nombre del grupo de recursos proporcionado en el script de PowerShell debe ser el del grupo de recursos del laboratorio. Para más información, vea, [Solución de errores comunes de implementación de Azure](../azure-resource-manager/templates/common-deployment-errors.md).

### <a name="where-can-i-find-more-error-information-if-a-vm-deployment-fails"></a>¿Dónde puedo encontrar más información sobre errores relativos a la implementación de VM?
Los errores de implementación de máquinas virtuales se capturan en los registros de actividad. Puede encontrar los registros de actividad de las máquinas virtuales de laboratorio en **Registros de auditoría** o **Diagnóstico de máquina virtual** en el menú de recursos de la página de máquina virtual del laboratorio (la página se muestra una vez que selecciona la máquina virtual en la lista Mis máquinas virtuales).

En ocasiones, el error de implementación se produce antes de que comience la implementación de la máquina virtual. Un ejemplo es cuando se supera el límite de suscripciones para un recurso que se creó con la máquina virtual. En este caso, los detalles del error se capturan en los registros de actividad de nivel de laboratorio. Los registros de actividad se encuentran en la parte inferior de las opciones de **Configuración y directivas**. Para obtener más información sobre el uso de los registros de actividad en Azure, consulte [Visualización de registros de actividad para auditar las acciones sobre los recursos](../azure-resource-manager/management/view-activity-logs.md).

### <a name="why-do-i-get-location-is-not-available-for-resource-type-error-when-trying-to-create-a-lab"></a>¿Por qué aparece el error "la ubicación no está disponible para el tipo de recurso" al intentar crear un laboratorio?
Es posible que vea un mensaje de error similar al siguiente al intentar crear un laboratorio:

```
The provided location 'australiacentral' is not available for resource type 'Microsoft.KeyVault/vaults'. List of available regions for the resource type is 'northcentralus,eastus,northeurope,westeurope,eastasia,southeastasia,eastus2,centralus,southcentralus,westus,japaneast,japanwest,australiaeast,australiasoutheast,brazilsouth,centralindia,southindia,westindia,canadacentral,canadaeast,uksouth,ukwest,westcentralus,westus2,koreacentral,koreasouth,francecentral,southafricanorth
```

Puede resolver este error realizando uno de los siguientes pasos:

#### <a name="option-1"></a>Opción 1
Compruebe la disponibilidad del tipo de recurso en las regiones de Azure en la página de [productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Si el tipo de recurso no está disponible en una región determinada, DevTest Labs no admitirá la creación de un laboratorio en esa región. Seleccione otra región al crear el laboratorio.

#### <a name="option-2"></a>Opción 2
Si el tipo de recurso está disponible en su región, compruebe si está registrado con su suscripción. Se puede realizar en el nivel de propietario de la suscripción como se muestra en [este artículo](../azure-resource-manager/management/resource-providers-and-types.md).
