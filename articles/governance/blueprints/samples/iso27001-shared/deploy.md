---
title: 'Ejemplo de plano técnico de servicios compartidos según la norma ISO 27001: Pasos de implementación'
description: Pasos de implementación del ejemplo de plano técnico de servicios compartidos según la norma ISO 27001.
author: DCtheGeek
ms.author: dacoulte
ms.date: 03/14/2019
ms.topic: sample
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: 88d5cfbbcb29cacc2e8c1c6a226367c5f23e8231
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2019
ms.locfileid: "64926300"
---
# <a name="deploy-the-iso-27001-shared-services-blueprint-sample"></a>Implementación del ejemplo de plano técnico de servicios compartidos según la norma ISO 27001

Para implementar el ejemplo de plano técnico de los servicios compartidos según la norma ISO 27001 de Azure Blueprints, debe seguir los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios**, busque la opción **Directiva** en el panel izquierdo y selecciónela. En la página **Directiva**, seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico de **servicios compartidos según la norma ISO 27001** en _Otros ejemplos_ y seleccione **Usar este ejemplo**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para su copia del ejemplo de plano técnico de servicios compartidos según la norma ISO 27001.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

## <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adecuarla a su entorno y necesidades, pero esa modificación puede apartarla de la norma ISO 27001.

1. Seleccione **Todos los servicios**, busque la opción **Directiva** en el panel izquierdo y selecciónela. En la página **Directiva**, seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. Escriba **Notas de cambios** como "Primera versión publicada del ejemplo de plano técnico según la norma ISO 27001". A continuación, seleccione **Publicar** en la parte inferior de la página.

## <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios**, busque la opción **Directiva** en el panel izquierdo y selecciónela. En la página **Directiva**, seleccione **Planos técnicos**.

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

   - Parámetros de plano técnico

     Muchos de los artefactos de la definición de plano técnico usan los parámetros definidos en esta sección para proporcionar coherencia.

     - **Nombre de la organización**: Escriba un nombre corto para su organización. Esta propiedad se utiliza principalmente para asignar nombres a los recursos.
     - **Prefijo de la dirección de subred de los servicios compartidos**: Proporcione el valor de la notación CIDR para conectar en red los recursos implementados.
     - **Ubicación de los servicios compartidos**: Determina en qué ubicación se implementan los artefactos. No todos los servicios están disponibles en todas las ubicaciones. Los artefactos que implementan tales servicios proporcionan una opción de parámetro para la ubicación en la que se va a implementar ese artefacto.
     - **Ubicación permitida (Directiva: Iniciativa de plano técnico para ISO 27001)**: Valor que indica las ubicaciones permitidas para los grupos de recursos y los recursos.
     - **Área de trabajo de Log Analytics para los agentes de máquina virtual (directiva: Iniciativa de plano técnico para ISO 27001)**: Especifica el identificador de recurso de un área de trabajo. Este parámetro utiliza una función `concat` para construir el identificador de recurso.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

## <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|DESCRIPCIÓN|
|-|-|-|-|
|\[Versión preliminar\]: Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Linux|Asignación de directiva|Opcional: Lista de imágenes de VM que han admitido el sistema operativo Linux que se agregarán al ámbito.|(Opcional) El valor predeterminado es _["none"]_.|
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Linux|Asignación de directiva|Opcional: Lista de imágenes de VM que han admitido el sistema operativo Linux que se agregarán al ámbito.|(Opcional) El valor predeterminado es _["none"]_.|
|\[Versión preliminar\]: Implementar el agente de Log Analytics para VM Scale Sets (VMSS) para Windows|Asignación de directiva|Opcional: Lista de imágenes de VM que han admitido el sistema operativo Windows que se agregarán al ámbito.|(Opcional) El valor predeterminado es _["none"]_.|
|\[Versión preliminar\]: Implementar el agente de Log Analytics en máquinas virtuales Windows|Asignación de directiva|Opcional: Lista de imágenes de VM que han admitido el sistema operativo Windows que se agregarán al ámbito.|(Opcional) El valor predeterminado es _["none"]_.|
|Tipos de recursos permitidos|Asignación de directiva|Tipos de recursos permitidos|Lista de tipos de recursos que se pueden implementar. Esta lista se compone de todos los tipos de recursos implementados en los servicios compartidos.|
|SKU de cuenta de almacenamiento permitida|Asignación de directiva|SKU de almacenamiento permitidas|Lista de SKU de cuenta de almacenamiento de registros de diagnóstico permitidas. El valor predeterminado es _["Standard_LRS"]_.|
|SKU de máquina virtual permitida|Asignación de directiva|Lista de SKU de máquinas virtuales que se pueden implementar. El valor predeterminado es _["Standard_DS1_v2", "Standard_DS2_v2"]_.|
|Iniciativa de plano técnico para ISO 27001|Asignación de directiva|Tipos de recursos para auditar registros de diagnóstico|Lista de tipos de recursos para auditar si la opción de registro de diagnóstico no está habilitada. Los valores aceptables se pueden encontrar en [esquemas de los registros de diagnóstico de Azure Monitor](../../../../azure-monitor/platform/diagnostic-logs-schema.md#supported-log-categories-per-resource-type).|
|Grupo de recursos de Log Analytics|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-sharedsvsc-log-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de Log Analytics|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de Log Analytics|Plantilla de Resource Manager|Nivel de servicio|Establece el nivel de servicio del área de trabajo de Log Analytics. El valor predeterminado es _PerNode_.|
|Plantilla de Log Analytics|Plantilla de Resource Manager|Retención del registro, en días|Retención de datos, en días. El valor predeterminado es _365_.|
|Plantilla de Log Analytics|Plantilla de Resource Manager|Ubicación|Región que se usa para crear el área de trabajo de Log Analytics. El valor predeterminado es _Oeste de EE. UU. 2_.|
|Grupo de recursos de red|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-sharedsvcs-net-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de red|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de Azure Firewall|Plantilla de Resource Manager|IP privada del firewall de Azure|Configura la dirección IP privada de [Azure Firewall](../../../../firewall/overview.md). Este valor también se usa como tabla de rutas predeterminadas en la subred de servicios compartidos. Debe ser parte de la notación CIDR definida en **Prefijo de la dirección de subred de Azure Firewall**. El valor predeterminado es _10.0.4.4_.|
|Plantilla de Azure Firewall|Plantilla de Resource Manager|Retención del registro, en días|Retención de datos, en días. El valor predeterminado es _365_.|
|Plantilla del grupo de seguridad de red|Plantilla de Resource Manager|Retención del registro, en días|Retención de datos, en días. El valor predeterminado es _365_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Prefijo de dirección de Virtual Network|La notación CIDR de la red virtual. El valor predeterminado es _10.0.0.0/16_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Habilitar protección contra DDoS en Virtual Network|Configura la protección contra DDoS de la red virtual. El valor predeterminado es _true_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Prefijo de la dirección de subred de los servicios compartidos|La notación CIDR para la subred de servicios compartidos. El valor predeterminado es _10.0.0.0/24_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Prefijo de la dirección de subred DMZ|La notación CIDR de la subred DMZ. El valor predeterminado es _10.0.1.0/24_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Prefijo de dirección de subred de Application Gateway|La notación CIDR de la subred de Application Gateway. El valor predeterminado es _10.0.2.0/24_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Prefijo de dirección de subred de la puerta de enlace de Virtual Network|La notación CIDR de la subred de la puerta de enlace de red virtual. El valor predeterminado es _10.0.3.0/24_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Prefijo de la dirección de subred de Azure Firewall|La notación CIDR de la subred de [Azure Firewall](../../../../firewall/overview.md). Debe incluir el parámetro de **dirección IP privada de Azure Firewall**.|
|Grupo de recursos de Key Vault|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-sharedsvcs-kv-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de Key Vault|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de Key Vault|Plantilla de Resource Manager|Nombre de usuario del administrador de jumpbox|Nombre de usuario de jumpbox. Debe coincidir con el mismo valor de propiedad de la **plantilla de jumpbox**. El valor predeterminado es _jb-admin-user_.|
|Plantilla de Key Vault|Plantilla de Resource Manager|Contraseña o clave SSH del administrador de jumpbox|Clave o contraseña de la cuenta en el jumpbox. Debe coincidir con el mismo valor de propiedad de la **plantilla de jumpbox**. No hay ningún valor predeterminado y no puede dejarse en blanco.|
|Plantilla de Key Vault|Plantilla de Resource Manager|Nombre de usuario del administrador de dominio|Nombre de usuario que se usa para acceder a la máquina virtual de Active Directory y unir otras máquinas virtuales a un dominio. Debe coincidir con el valor de propiedad del **usuario administrador de dominio** de la **plantilla de Active Directory Domain Services**. El valor predeterminado es _domain-admin-user_.|
|Plantilla de Key Vault|Plantilla de Resource Manager|Contraseña del administrador de dominio|Contraseña del usuario administrador de dominio. No hay ningún valor predeterminado y no puede dejarse en blanco.|
|Plantilla de Key Vault|Plantilla de Resource Manager|Identificador de objeto de AAD|El identificador de objeto de AAD de la cuenta que requiere acceso a la instancia de Key Vault. No hay ningún valor predeterminado y no se puede dejar en blanco. Para buscar este valor en Azure Portal, busque y seleccione "Usuarios" en _Servicios_. Use el cuadro _Nombre_ para filtrar por el nombre de cuenta y seleccione esa cuenta. En la página _Perfil de usuario_, seleccione el icono "Haga clic para copiar" que está situado junto al _identificador de objeto_.  |
|Plantilla de Key Vault|Plantilla de Resource Manager|Retención del registro, en días|Retención de datos, en días. El valor predeterminado es _365_.|
|Plantilla de Key Vault|Plantilla de Resource Manager|SKU de Key Vault|Especifica la SKU de Key Vault que se crea. El valor predeterminado es _Premium_.|
|Grupo de recursos de jumpbox|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-sharedsvcs-jb-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de jumpbox|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de jumpbox|Plantilla de Resource Manager|Nombre de usuario del administrador de jumpbox|Nombre de usuario que se usa para acceder a las máquinas virtuales de jumpbox. Debe coincidir con el mismo valor de propiedad de la **plantilla de Key Vault**. El valor predeterminado es _jb-admin-user_.|
|Plantilla de jumpbox|Plantilla de Resource Manager|Contraseña del administrador de Jumpbox (identificador de recurso de Key Vault)|Identificador de recurso de Key Vault. Use "/subscriptions/{subscriptionId}/resourceGroups/{orgName}-sharedsvcs-kv-rg/providers/Microsoft.KeyVault/vaults/{orgName}-sharedsvcs-kv" y sustituya `{subscriptionId}` por el identificador de la suscripción y `{orgName}` por el parámetro del plano técnico de **nombre de la organización**.|
|Plantilla de jumpbox|Plantilla de Resource Manager|Contraseña del administrador de jumpbox (nombre del secreto de Key Vault)|Nombre de usuario del administrador de jumpbox. El valor debe coincidir con el del **nombre de usuario del administrador de jumpbox** de la propiedad **plantilla de Key Vault**.|
|Plantilla de jumpbox|Plantilla de Resource Manager|Sistema operativo de jumpbox|Determina el sistema operativo de la máquina virtual de jumpbox. El valor predeterminado es _Windows_.|
|Grupo de recursos de Active Directory Domain Services|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-sharedsvcs-adds-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de Active Directory Domain Services|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de Active Directory Domain Services|Plantilla de Resource Manager|Nombre de usuario del administrador de dominio|Nombre de usuario de jumpbox de ADDS. Debe coincidir con el mismo valor de propiedad de la **plantilla de Key Vault**. El valor predeterminado es _adds-admin-user_.|
|Plantilla de Active Directory Domain Services|Plantilla de Resource Manager|Contraseña del administrador de dominio (identificador de recurso de Key Vault)|Identificador de recurso de Key Vault. Use "/subscriptions/{subscriptionId}/resourceGroups/{orgName}-sharedsvcs-kv-rg/providers/Microsoft.KeyVault/vaults/{orgName}-sharedsvcs-kv" y sustituya `{subscriptionId}` por el identificador de la suscripción y `{orgName}` por el parámetro del plano técnico de **nombre de la organización**.|
|Plantilla de Active Directory Domain Services|Plantilla de Resource Manager|Contraseña del administrador de dominio (nombre del secreto de Key Vault)|Nombre de usuario del administrador de dominio. El valor debe coincidir con el del **nombre de usuario del administrador de dominio** de la propiedad **plantilla de Key Vault**.|
|Plantilla de Active Directory Domain Services|Plantilla de Resource Manager|Nombre de dominio|Nombre de la instancia de Active Directory creada por el ejemplo. El valor predeterminado es _contoso.com_.|
|Plantilla de Active Directory Domain Services|Plantilla de Resource Manager|Usuario administrador del dominio|Nombre de usuario de la cuenta de AD del administrador y para unir dispositivos al dominio de AD. Debe coincidir con el valor de propiedad del **nombre de usuario del administrador de AD** en la **plantilla de Key Vault**. El valor predeterminado es _domain-admin-user_.|
|Plantilla de Active Directory Domain Services|Plantilla de Resource Manager|Contraseña del administrador de dominio|Establezca los detalles de Key Vault para almacenar la contraseña. No hay ningún valor predeterminado y no puede dejarse en blanco.|

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado los pasos para implementar el ejemplo de plano técnico de los servicios compartidos según la norma ISO 27001, visite los siguientes artículos para obtener información sobre la arquitectura y la asignación de controles:

> [!div class="nextstepaction"]
> [Plano técnico de servicios compartidos según la norma ISO 27001: Introducción](./index.md)
> [Plano técnico de servicios compartidos según la norma ISO 27001: Asignación de controles](./control-mapping.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Más información sobre el [ciclo de vida del plano técnico](../../concepts/lifecycle.md)
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
