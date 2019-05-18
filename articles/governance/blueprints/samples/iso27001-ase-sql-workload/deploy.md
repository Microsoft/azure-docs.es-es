---
title: 'Ejemplo: plano técnico de cargas de trabajo de ASE o SQL según la norma ISO 27001 (pasos de implementación)'
description: Pasos de la implementación del ejemplo de plano técnico de cargas de trabajo de App Service Environment y SQL Database según la norma ISO 27001.
author: DCtheGeek
ms.author: dacoulte
ms.date: 03/14/2019
ms.topic: sample
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: 78f608aedd53aa1071eaf88864f5a63f8f9e6072
ms.sourcegitcommit: c174d408a5522b58160e17a87d2b6ef4482a6694
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2019
ms.locfileid: "59791018"
---
# <a name="deploy-the-iso-27001-app-service-environmentsql-database-workload-blueprint-sample"></a>Implementación del ejemplo de plano técnico de cargas de trabajo de App Service Environment y SQL Database según la norma ISO 27001

Para implementar el ejemplo de plano técnico para cargas de trabajo de App Service Environment y SQL Database según la norma ISO 27001 para Azure Blueprints, es preciso seguir estos pasos:

> [!div class="checklist"]
> - Implementar el ejemplo de plano técnico de [servicios compartidos según la norma ISO 27001](../iso27001-shared/index.md)
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="deploy-the-iso-27001-shared-services-blueprint-sample"></a>Implementación del ejemplo de plano técnico de servicios compartidos según la norma ISO 27001

Para poder implementar este ejemplo de plano técnico, es preciso implementar el ejemplo de plano técnico de [servicios compartidos según la norma ISO 27001](../iso27001-shared/index.md) en la suscripción de destino. Sin una implementación correcta de este último implementación correcta del muestra de plano técnico de servicios compartidos según la norma ISO 27001, al primero le faltarán dependencias de la infraestructura y se producirán error durante la implementación.

> [!IMPORTANT]
> Este ejemplo de plano técnico debe estar asignado a la misma suscripción que el ejemplo de plano técnico de [servicios compartidos según la norma ISO 27001](../iso27001-shared/index.md).

## <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios**, busque la opción **Directiva** en el panel izquierdo y selecciónela. En la página **Directiva**, seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico de **carga de trabajo de ASE/SQL según la norma ISO 27001** en _Otros ejemplos_ y seleccione **Usar este ejemplo**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: especifique un nombre para su copia del ejemplo de plano técnico de la carga de trabajo de ASE/SQL según la norma ISO 27001.
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
     - **Id. de suscripción de servicios compartidos**: identificador de la suscripción en la que se asigna el ejemplo de plano técnico de [servicios compartidos según la norma ISO 27001](../iso27001-shared/index.md).
     - **Prefijo de la dirección de la subred predeterminada**: la notación CIDR de la subred predeterminada de la red virtual.
       El valor predeterminado es _10.1.0.0/16_.
     - **Ubicación de la carga de trabajo**: determina en qué ubicación se implementan los artefactos. No todos los servicios están disponibles en todas las ubicaciones. Los artefactos que implementan tales servicios proporcionan una opción de parámetro para la ubicación en la que se va a implementar ese artefacto.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

## <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|DESCRIPCIÓN|
|-|-|-|-|
|Grupo de recursos de Log Analytics|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-workload-log-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de Log Analytics|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de Log Analytics|Plantilla de Resource Manager|Nivel de servicio|Establece el nivel de servicio del área de trabajo de Log Analytics. El valor predeterminado es _PerNode_.|
|Plantilla de Log Analytics|Plantilla de Resource Manager|Retención del registro, en días|Retención de datos, en días. El valor predeterminado es _365_.|
|Plantilla de Log Analytics|Plantilla de Resource Manager|Ubicación|Región que se usa para crear el área de trabajo de Log Analytics. El valor predeterminado es _Oeste de EE. UU. 2_.|
|Grupo de recursos de red|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-workload-net-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de red|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla del grupo de seguridad de red|Plantilla de Resource Manager|Retención del registro, en días|Retención de datos, en días. El valor predeterminado es _365_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|IP privada del firewall de Azure|Configura la dirección IP privada del [firewall de Azure](../../../../firewall/overview.md). Debe formar parte de la notación CIDR definida en el parámetro _Prefijo de la dirección de subred de Azure Firewall_ del artefacto de **ISO 27001: servicios compartidos.** El valor predeterminado es: _10.0.4.4_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Identificador de suscripción de servicios compartidos|Valor utilizado para habilitar el emparejamiento de VNET entre una carga de trabajo y los servicios compartidos.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Prefijo de dirección de Virtual Network|La notación CIDR de la red virtual. El valor predeterminado es _10.1.0.0/16_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Prefijo de la dirección de la subred predeterminada|La notación CIDR de la subred predeterminada de la red virtual. El valor predeterminado es _10.1.0.0/16_.|
|Plantilla de Virtual Network y de la tabla de rutas|Plantilla de Resource Manager|Dirección IP de AD DS|Dirección IP de la primera máquina virtual de AD DS. Este valor se utiliza como DNS de red virtual personalizado.|
|Grupo de recursos de Key Vault|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-workload-kv-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de Key Vault|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de Key Vault|Plantilla de Resource Manager|Identificador de objeto de AAD|El identificador de objeto de AAD de la cuenta que requiere acceso a la instancia de Key Vault. No hay ningún valor predeterminado y no se puede dejar en blanco. Para buscar este valor en Azure Portal, busque y seleccione "Usuarios" en _Servicios_. Use el cuadro _Nombre_ para filtrar por el nombre de cuenta y seleccione esa cuenta. En la página _Perfil de usuario_, seleccione el icono "Haga clic para copiar" que está situado junto al _identificador de objeto_.|
|Plantilla de Key Vault|Plantilla de Resource Manager|Retención del registro, en días|Retención de datos, en días. El valor predeterminado es _365_.|
|Plantilla de Key Vault|Plantilla de Resource Manager|SKU de Key Vault|Especifica la SKU de Key Vault que se crea. El valor predeterminado es _Premium_.|
|Plantilla de Key Vault|Plantilla de Resource Manager|Nombre de usuario de administrador de Azure SQL Server|Nombre de usuario que se utiliza para acceder a Azure SQL Server. Debe coincidir con el mismo valor de propiedad de **plantilla de Azure SQL Database**. El valor predeterminado es _sql-admin-user_.|
|Grupo de recursos de Azure SQL Database|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-workload-azsql-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de Azure SQL Database|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de Azure SQL Database|Plantilla de Resource Manager|Nombre de usuario de administrador de Azure SQL Server|Nombre de usuario de Azure SQL Server. Debe coincidir con el mismo valor de propiedad de la **plantilla de Key Vault**. El valor predeterminado es _sql-admin-user_.|
|Plantilla de Azure SQL Database|Plantilla de Resource Manager|Contraseña del administrador de Azure SQL Server (identificador de recurso de Key Vault)|Identificador de recurso de Key Vault. Use "/subscription/{subscriptionId}/resourceGroups/{orgName}-workload-kv/providers/Microsoft.KeyVault/vaults/{orgName}-workload-kv" y sustituya `{subscriptionId}` por el identificador de la suscripción y `{orgName}` por el parámetro del plano técnico de **nombre de la organización**.|
|Plantilla de Azure SQL Database|Plantilla de Resource Manager|Contraseña del administrador de Azure SQL Server (nombre del secreto de Key Vault)|Nombre de usuario del administrador de Azure SQL Server. El valor debe coincidir con el del **nombre de usuario del administrador de Azure SQL Server** de la propiedad **plantilla de Key Vault**.|
|Plantilla de Azure SQL Database|Plantilla de Resource Manager|Retención del registro, en días|Retención de datos, en días. El valor predeterminado es _365_.|
|Plantilla de Azure SQL Database|Plantilla de Resource Manager|Identificador de objeto de administrador de AAD|Identificador de objeto de AAD del usuario que se asignará como administrador de Active Directory. No hay ningún valor predeterminado y no se puede dejar en blanco. Para buscar este valor en Azure Portal, busque y seleccione "Usuarios" en _Servicios_. Use el cuadro _Nombre_ para filtrar por el nombre de cuenta y seleccione esa cuenta. En la página _Perfil de usuario_, seleccione el icono "Haga clic para copiar" que está situado junto al _identificador de objeto_.|
|Plantilla de Azure SQL Database|Plantilla de Resource Manager|Inicio de sesión de administrador de AAD|Actualmente no es posible establecer cuentas Microsoft (como live.com o outlook.com) como administrador. Solo los usuarios y grupos de seguridad de su organización pueden establecerse como administrador. No hay ningún valor predeterminado y no se puede dejar en blanco. Para buscar este valor en Azure Portal, busque y seleccione "Usuarios" en _Servicios_. Use el cuadro _Nombre_ para filtrar por el nombre de cuenta y seleccione esa cuenta. En la página _Perfil de usuario_, copie el valor de _Nombre de usuario_.|
|Grupo de recursos de App Service Environment|Grupos de recursos|NOMBRE|**Bloqueado**: concatena el **nombre de la organización** con `-workload-ase-rg` para que el grupo de recursos sea único.|
|Grupo de recursos de App Service Environment|Grupos de recursos|Ubicación|**Bloqueado**: utiliza el parámetro del plano técnico.|
|Plantilla de App Service Environment|Plantilla de Resource Manager|Nombre de dominio|Nombre de la instancia de Active Directory creada por el ejemplo. El valor predeterminado es _contoso.com_.|
|Plantilla de App Service Environment|Plantilla de Resource Manager|Ubicación de ASE|Ubicación de App Service Environment. El valor predeterminado es _Oeste de EE. UU. 2_.|
|Plantilla de App Service Environment|Plantilla de Resource Manager|Retención del registro de Application Gateway, en días|Retención de datos, en días. El valor predeterminado es _365_.|

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado los pasos para implementar el ejemplo de plano técnico de cargas de trabajo de App Service Environment y SQL Database según la norma ISO 27001, lea los siguientes artículos para obtener información sobre la arquitectura y la asignación de controles:

> [!div class="nextstepaction"]
> [Plano técnico para cargas de trabajo de App Service Environment y SQL Database compatibles con la norma ISO 27001: introducción](./index.md)
> [Plano técnico para cargas de trabajo de App Service Environment y SQL Database compatibles con la norma ISO 27001: asignación de controles](./control-mapping.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Más información sobre el [ciclo de vida del plano técnico](../../concepts/lifecycle.md)
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).