---
title: Atribución de uso de partners y clientes de marketplace comercial
description: Obtenga información general sobre el seguimiento del uso de clientes de las soluciones de Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: vikrambmsft
ms.author: vikramb
ms.date: 04/14/2020
ms.custom: devx-track-terraform
ms.openlocfilehash: ab729d34219c05ee76a2a14832f41342d29eab21
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87065797"
---
# <a name="commercial-marketplace-partner-and-customer-usage-attribution"></a>Atribución de uso de partners y clientes de marketplace comercial

La atribución del uso de clientes es un método para asociar los recursos de Azure que se ejecutan en las suscripciones de cliente, implementado para ejecutar la solución, con usted como partner. La formación de estas asociaciones en sistemas internos de Microsoft aporta mayor visibilidad de la superficie de Azure que ejecuta el software. Al adoptar esta capacidad de seguimiento, se alinea con los equipos de ventas de Microsoft y obtiene crédito para los programas para partners de Microsoft.

Puede formar la asociación a través de Azure Marketplace, el repositorio del Inicio rápido, los repositorios privados de GitHub y la involucración del cliente 1:1 que crea IP durables (como el desarrollo de una aplicación).

La atribución de uso del cliente admite tres opciones de implementación:

- Plantillas de Azure Resource Manager: los asociados pueden usar plantillas de Resource Manager para implementar los servicios de Azure para ejecutar su software. Los asociados pueden crear una plantilla de Resource Manager para definir la infraestructura y configuración de su solución de Azure. Una plantilla de Resource Manager les permite tanto a usted como a sus clientes implementar la solución a lo largo de su ciclo de vida. Puede estar seguro de que los recursos se implementan de forma consistente.
- API de Azure Resource Manager: los asociados pueden llamar a las API de Resource Manager directamente para implementar una plantilla de Azure Resource Manager o para generar las llamadas a API para aprovisionar directamente los servicios de Azure.
- Terraform: los asociados pueden usar Terraform para implementar una plantilla de Resource Manager o servicios de Azure directamente.

>[!IMPORTANT]
>- La atribución de uso de clientes no está pensada para realizar un seguimiento del trabajo de los integradores de sistemas, los proveedores de servicios administrados o las herramientas diseñadas para implementar y administrar el software que se ejecuta en Azure.
>
>- La atribución de uso de clientes es para implementaciones nuevas y NO admite el etiquetado de recursos existentes que ya se hayan implementado.
>
>- Se requiere la atribución de uso de clientes en las ofertas de [aplicación de Azure](./partner-center-portal/create-new-azure-apps-offer.md) publicadas en Azure Marketplace.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-guids"></a>Creación de los identificadores únicos globales

Un GUID es un identificador de referencia único que tiene 32 dígitos hexadecimales. Para crear identificadores únicos globales para el seguimiento, debe usar un generador de GUID. El equipo de Azure Storage ha creado un [formulario de generación de GUID](https://aka.ms/StoragePartners) que enviará por correo electrónico un GUID en el formato correcto. Este GUID se puede reutilizar en los distintos sistemas de seguimiento.

> [!NOTE]
> Es muy recomendable usar el [formulario del generador de GUID de Azure Storage](https://aka.ms/StoragePartners) para crear el identificador único global. Para más información, consulte las [P+F](#faq).

Se recomienda crear un GUID único para todos los canales de distribución y oferta de cada producto. Puede optar por usar un solo GUID para los múltiples canales de distribución del producto si no quiere que los informes se dividan.

Si implementa un producto mediante una plantilla y está disponible tanto en Azure Marketplace como en GitHub, puede crear y registrar dos GUID distintos:

- Producto A en Azure Marketplace
- Producto A en GitHub

La creación de informes se realiza mediante un id. y un GUID de Microsoft Partner Network.

También puede realizar un seguimiento del uso en un nivel más granular registrando GUID adicionales y cambiando los GUID entre planes, donde los planes son variantes de una oferta.

## <a name="register-guids"></a>Registro de GUID

Los GUID deben estar registrados en el Centro de partners para permitir la atribución de uso de clientes.

Después de agregar un GUID a la plantilla o en el agente de usuario, y registrarlo en el Centro de partners, se realizará un seguimiento de las futuras implementaciones.

> [!NOTE]
> Si va a publicar la oferta de [Aplicación de Azure](./partner-center-portal/create-new-azure-apps-offer.md) en Azure Marketplace a través del Centro de partners, cualquier GUID nuevo que se use dentro de la plantilla se registrará de forma automática en el perfil del Centro de partners al cargar la plantilla.  

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard).

1. Regístrese como [publicador comercial de Marketplace](https://aka.ms/JoinMarketplace).

   * Los partners deben [tener un perfil en el Centro de partners](become-publisher.md). Le recomendamos que vea la oferta en Azure Marketplace o AppSource.
   * Los asociados pueden registrar varios identificadores únicos globales.
   * Los partners pueden registrar los GUID para las plantillas de solución que no son de marketplace y las ofertas.

1. En la esquina superior derecha, seleccione el icono de engranaje de la configuración y, a continuación, **Configuración de desarrollador**.

1. En la **página de configuración de la cuenta**, seleccione **Add Tracking GUID** (Agregar GUID de seguimiento).

1. En el cuadro **GUID**, escriba su identificador único global de seguimiento. Escriba solo el GUID, sin el prefijo `pid-`. En el cuadro **Descripción**, escriba el nombre o la descripción de la oferta.

1. Para registrar varios identificadores únicos globales, vuelva a seleccionar **Add Tracking GUID** (Agregar GUID de seguimiento). Aparecen más cuadros en la página.

1. Seleccione **Guardar**.

## <a name="use-resource-manager-templates"></a>Uso de plantillas de Resource Manager
Muchas soluciones de partners se implementan mediante plantillas de Azure Resource Manager. Si tiene una plantilla de Resource Manager disponible en Azure Marketplace, en GitHub o como Inicio rápido, el proceso de modificación de esta para habilitar la atribución de uso de clientes debe ser sencillo.

> [!NOTE]
> Para más información sobre la creación y publicación de plantillas de solución, consulte
> * [Creación e implementación de plantillas de Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).
>* [Oferta de aplicación de Azure](./partner-center-portal/create-new-azure-apps-offer.md).
>* Vídeo: [Building Solution Templates, and Managed Applications for the Azure Marketplace](https://channel9.msdn.com/Events/Build/2018/BRK3603) (Creación de plantillas de soluciones y aplicaciones administradas para Azure Marketplace).


Para agregar un identificador único global (GUID), se realizas una modificación única en el archivo de plantilla principal:

1. [Cree un GUID](#create-guids) mediante el método sugerido y [registre el GUID](#register-guids).

1. Abra la plantilla de Resource Manager.

1. Agregue un nuevo recurso en el archivo de plantilla principal. El recurso solo debe estar en los archivos **mainTemplate.json** o **azuredeploy.json**, no en ninguna de las plantillas vinculadas o anidadas.

1. Escriba el valor del GUID después del prefijo `pid-` (por ejemplo, pid-eb7927c8-dd66-43e1-b0cf-c346a422063).

1. Compruebe si la plantilla tiene errores.

1. Vuelva a publicar la plantilla en los repositorios adecuados.

1. [Compruebe que el GUID es correcto en la implementación de plantillas](#verify-the-guid-deployment).

### <a name="sample-resource-manager-template-code"></a>Código de ejemplo de plantilla de Resource Manager

Para habilitar el seguimiento de recursos de la plantilla, deberá agregar los siguientes recursos adicionales en la sección de recursos. Asegúrese de modificar el siguiente código de ejemplo con sus propias entradas cuando lo agregue al archivo de plantilla principal.
El recurso solo se debe agregar al archivo **mainTemplate.json** o **azuredeploy.json**, y no a plantillas vinculadas o anidadas.

```
// Make sure to modify this sample code with your own inputs where applicable

{ // add this resource to the resources section in the mainTemplate.json (do not add the entire file)
    "apiVersion": "2018-02-01",
    "name": "pid-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX", // use your generated GUID here
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "resources": []
        }
    }
} // remove all comments from the file when complete
```

## <a name="use-the-resource-manager-apis"></a>Uso de las API de Resource Manager

En algunos casos, es posible que prefiera realizar llamadas directamente a las API REST de Resource Manager para implementar los servicios de Azure. [Azure admite varios SDK](https://docs.microsoft.com/azure/?pivot=sdkstools) para habilitar estas llamadas. Puede usar uno de los SDK, o llamar a las API REST directamente para implementar los recursos.

Si usa una plantilla de Resource Manager, para etiquetar la solución debe seguir las instrucciones que se han proporcionado anteriormente. Si no usa una plantilla de Resource Manager y realiza llamadas API directas, puede etiquetar la implementación para asociar el uso de los recursos de Azure.

### <a name="tag-a-deployment-with-the-resource-manager-apis"></a>Etiquetado de una implementación con las API de Resource Manager

Para habilitar la atribución de uso del cliente, al diseñar las llamadas API, se incluye un identificador único global en el encabezado del agente de usuario de la solicitud. Agregue el identificador único global a cada oferta o SKU. Dé formato a la cadena con el prefijo `pid-` e incluya el GUID generado por el asociado. Este es un ejemplo del formato del identificador único global para la inserción en el agente de usuario:

![Formato de GUID de ejemplo](media/marketplace-publishers-guide/tracking-sample-guid-for-lu-2.PNG)

> [!NOTE]
> El formato de la cadena es importante. Si no se incluye el prefijo `pid-`, no es posible consultar los datos. El seguimiento de los distintos SDK se hace de forma diferente. Para implementar este método, revise la compatibilidad y el enfoque del seguimiento de su SDK de Azure preferido.

#### <a name="example-the-python-sdk"></a>Ejemplo: SDK de Python

Para Python, use el atributo **config**. El atributo solo se puede agregar a un UserAgent. Este es un ejemplo:

![Agregar el atributo a un agente de usuario](media/marketplace-publishers-guide/python-for-lu.PNG)

> [!NOTE]
> Agregue el atributo a cada cliente. No hay ninguna configuración estática global. Puede etiquetar una fábrica de cliente para asegurarse de que todos los clientes realizan el seguimiento. Para más información, consulte este [ejemplo de fábrica de cliente en GitHub](https://github.com/Azure/azure-cli/blob/7402fb2c20be2cdbcaa7bdb2eeb72b7461fbcc30/src/azure-cli-core/azure/cli/core/commands/client_factory.py#L70-L79).

#### <a name="tag-a-deployment-by-using-the-azure-powershell"></a>Etiquetado de una implementación mediante Azure PowerShell

Si implementa recursos a través de Azure PowerShell, use el siguiente método para anexar el identificador único global:

```powershell
[Microsoft.Azure.Common.Authentication.AzureSession]::ClientFactory.AddUserAgent("pid-eb7927c8-dd66-43e1-b0cf-c346a422063")
```

#### <a name="tag-a-deployment-by-using-the-azure-cli"></a>Etiquetado de una implementación mediante la CLI de Azure

Cuando use la CLI de Azure para anexar un identificador único global, establezca la variable de entorno **AZURE_HTTP_USER_AGENT**. Dicha variable se puede establecer en el ámbito de un script. Pero también se puede establecer globalmente para el ámbito de una shell:

```
export AZURE_HTTP_USER_AGENT='pid-eb7927c8-dd66-43e1-b0cf-c346a422063'
```
Para más información, consulte [Azure SDK para Go](https://docs.microsoft.com/azure/developer/go/).

## <a name="use-terraform"></a>Uso de Terraform

La compatibilidad con Terraform está disponible a partir de la versión 1.21.0 del proveedor de Azure: [https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/CHANGELOG.md#1210-january-11-2019](https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/CHANGELOG.md#1210-january-11-2019).  Esta compatibilidad se aplica a todos los asociados que implementen su solución mediante Terraform y a todos los recursos que implemente y mida el proveedor de Azure (versión 1.21.0 o posteriores).

El proveedor de Azure para Terraform agrega un nuevo campo opcional denominado [*partner_id*](https://www.terraform.io/docs/providers/azurerm/#partner_id) que es donde se especifica el identificador único global de seguimiento que usa para la solución. El valor de este campo también puede proceder de la variable de entorno *ARM_PARTNER_ID*.

```
provider "azurerm" {
          subscription_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          client_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          ……
          # new stuff for ISV attribution
          partner_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"}
```
Los asociados que deseen que la implementación mediante Terraform tenga un seguimiento de la atribución de uso del cliente deben hacer lo siguiente:

* Crear un identificador único global (que se debe agregar para cada oferta o SKU).
* Actualizar el proveedor de Azure para establecer el valor de *partner_id* con el identificador único global (NO añadir el prefijo "pid-" al identificador único global, simplemente establecerlo con el valor real).


## <a name="verify-the-guid-deployment"></a>Comprobación de la implementación de GUID

Tras modificar la plantilla y realizar una implementación de prueba, use el siguiente script de PowerShell para recuperar los recursos que implementó y etiquetó.

Puede usar dicho script para comprobar que el GUID se ha agregado correctamente a la plantilla de Resource Manager. El script no se aplica a la implementación de la API de Resource Manager ni Terraform.

Inicie sesión en Azure. Antes de ejecutar el script seleccione la suscripción con la implementación que desea comprobar. Ejecute el script en el contexto de la suscripción de la implementación.

Tanto el **GUID** como el nombre **resourceGroup** son parámetros necesarios.

[El script original](https://gist.github.com/bmoore-msft/ae6b8226311014d6e7177c5127c7eba1#file-verify-deploymentguid-ps1) se puede obtener en GitHub.

```powershell
Param(
    [GUID][Parameter(Mandatory=$true)]$guid,
    [string][Parameter(Mandatory=$true)]$resourceGroupName
)

# Get the correlationId of the pid deployment

$correlationId = (Get-AzResourceGroupDeployment -ResourceGroupName
$resourceGroupName -Name "pid-$guid").correlationId

# Find all deployments with that correlationId

$deployments = Get-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName | Where-Object{$_.correlationId -eq $correlationId}

# Find all deploymentOperations in a deployment by name
# PowerShell doesn't surface outputResources on the deployment
# or correlationId on the deploymentOperation

foreach ($deployment in $deployments){

# Get deploymentOperations by deploymentName
# then the resourceId for any create operation

($deployment | Get-AzResourceGroupDeploymentOperation | Where-Object{$_.properties.provisioningOperation -eq "Create" -and $_.properties.targetResource.resourceType -ne "Microsoft.Resources/deployments"}).properties.targetResource.id

}
```

## <a name="report"></a>Informe

Puede encontrar el informe de atribución de uso del cliente en el panel del Centro de partners ([https://partner.microsoft.com/dashboard/mpn/analytics/CPP/MicrosoftAzure](https://partner.microsoft.com/dashboard/mpn/analytics/CPP/MicrosoftAzure)). Para ver el informe, tendrá que iniciar sesión con las credenciales del Centro de asociados. Si tiene problemas con los informes o el inicio de sesión, cree una solicitud de soporte técnico siguiendo las instrucciones de la sección de soporte técnico de Get.

Elija Tracked Templates (Plantillas en seguimiento) en la lista desplegable de Partner Association Type (Tipo de asociación del asociado) para ver el informe.

![Informe de la atribución de uso del cliente](media/marketplace-publishers-guide/customer-usage-attribution-report.png)

## <a name="notify-your-customers"></a>Notificación a los clientes

Los asociados deben informar a sus clientes de las implementaciones que usan atribución de uso del cliente. Microsoft notifica al asociado el uso de Azure que está asociado con estas implementaciones. Los ejemplos siguientes incluyen contenido que puede usar para notificar a los clientes estas implementaciones. En los ejemplos, reemplace \<PARTNER> por el nombre de la empresa. Los asociados deben asegurarse de que la notificación es compatible con sus directivas de privacidad y recopilación de datos, incluyendo las opciones que permiten excluir a los clientes del seguimiento.

### <a name="notification-for-resource-manager-template-deployments"></a>Notificación de implementaciones de plantillas de Resource Manager

Al implementar esta plantilla, Microsoft puede identificar la instalación del software de \<PARTNER> con los recursos de Azure que se implementan. Microsoft puede correlacionar los recursos de Azure que se usan para admitir el software. Microsoft recopila esta información para proporcionar las mejores experiencias con sus productos y conseguir que sus negocios funcionen. Los datos se recopilan en función de las directivas de privacidad de Microsoft, que se encuentran en https://www.microsoft.com/trustcenter, y se rigen por ellas.

### <a name="notification-for-sdk-or-api-deployments"></a>Notificación para las implementaciones de SDK o API

Al implementar el software de \<PARTNER>, Microsoft puede identificar la instalación del software de \<PARTNER> con los recursos de Azure que se implementan. Microsoft puede correlacionar los recursos de Azure que se usan para admitir el software. Microsoft recopila esta información para proporcionar las mejores experiencias con sus productos y conseguir que sus negocios funcionen. Los datos se recopilan en función de las directivas de privacidad de Microsoft, que se encuentran en https://www.microsoft.com/trustcenter, y se rigen por ellas.

## <a name="get-support"></a>Obtención de soporte técnico

Existen dos canales de soporte técnico según los problemas que se encuentre.

Si tiene algún problema en el Centro de asociados, como consultar un informe de atribución de uso del cliente o iniciar sesión, cree una solicitud de soporte técnico para el equipo de soporte técnico del Centro de asociados aquí: [https://partner.microsoft.com/support](https://partner.microsoft.com/support)

![Captura de pantalla de la página para obtener soporte técnico](./media/marketplace-publishers-guide/partner-center-log-in-support.png)

Si necesita ayuda para la atribución de uso de del cliente o la incorporación a Marketplace, por ejemplo, cómo configurar la primera, siga estos pasos:

1. Vaya a la [página de soporte técnico](https://go.microsoft.com/fwlink/?linkid=844975).

1. En **Problem type** (Tipo de problema), seleccione **Marketplace Onboarding** (Incorporación a Marketplace).

1. En **Category** (Categoría), elija la categoría del problema:

   - Para problemas de asociación de uso, seleccione **Other** (Otros).
   - Para problemas de acceso a Azure Marketplace, seleccione **Access Problem** (Problema de acceso).

     ![Elegir la categoría de problema](media/marketplace-publishers-guide/lu-article-incident.png)

1. Haga clic en **Start Request** (Iniciar solicitud).

1. En la página siguiente, especifique los valores necesarios. Seleccione **Continuar**.

1. En la página siguiente, especifique los valores necesarios.

   > [!IMPORTANT]
   > En el cuadro **Incident title** (Título de la incidencia), escriba **Seguimiento del uso de ISV**. Describa el problema de forma detallada.

   ![Escribir Seguimiento del uso de ISV como título de la incidencia](media/marketplace-publishers-guide/guid-dev-center-help-hd%201.png)

1. Complete el formulario y seleccione **Submit** (Enviar).

También puede recibir orientación técnica de un asesor técnico para asociados de Microsoft sobre escenarios técnicos antes de la venta, escenarios de implementación y escenarios de desarrollo de aplicaciones para entender e incorporar la atribución de uso del cliente.

### <a name="how-to-submit-a-technical-consultation-request"></a>Envío de una solicitud de consulta técnica

1. Visite [Servicios técnicos para socios](https://aka.ms/TechnicalJourney).
1. Seleccione Cloud infrastructure and management (Infraestructura en la nube y administración) y una nueva página se abrirá para que pueda ver el trayecto técnico.
1. En Deployment Services (Servicios de implementación), haga clic en el botón Submit a request (Enviar una solicitud).
1. Inicie sesión con su cuenta de servicio administrada (cuenta de MPN) o de AAD (cuenta de panel del asociado); se abrirá un formulario de solicitud en línea en función de sus credenciales de inicio de sesión:
    * Complete/revise la información de contacto.
    * Los detalles de la consulta pueden haberse rellenado previamente o quizá deba seleccionar de las listas desplegables.
    * Escriba un título y la descripción del problema (proporcione el máximo de detalle).
1. Haga clic en Enviar.

Vea instrucciones paso a paso con capturas de pantallas en [Uso de los servicios de preventas e implementación técnicos](https://aka.ms/TechConsultInstructions).

### <a name="whats-next"></a>Pasos siguientes

Un asesor técnico para los asociados de Microsoft le contactará para programar una llamada y determinar sus necesidades.

## <a name="faq"></a>Preguntas más frecuentes

**¿Cuál es la ventaja de agregar el identificador único global a la plantilla?**

Microsoft proporciona a sus asociados una vista de las implementaciones por parte de los clientes de sus soluciones, así como información detallada sobre su uso de influencia. Tanto Microsoft como el asociado pueden usar dicha información para impulsar una participación más estrecha entre los equipos de ventas. Tanto Microsoft como el asociado pueden usar los datos para obtener una vista más coherente del impacto que un asociado individual tiene en el crecimiento de Azure.

**Una vez que se ha agregado un identificador único global, ¿se puede cambiar?**

Sí, cualquier cliente o asociado de implementación puede personalizar la plantilla y cambiar o quitar el identificador único global. Se recomienda que los asociados describan de forma proactiva la función de los recursos y el identificador único global a sus clientes y asociados, para evitar la eliminación o modificación de este. El cambio del identificador único global solo afecta a las implementaciones y a los recursos nuevos, no a los existentes.

**¿Puedo realizar un seguimiento de las plantillas implementadas desde un repositorio que no sea de Microsoft, como GitHub?**

Sí, si el GUID está presente cuando se implemente la plantilla, se realiza el seguimiento. Los partners todavía deben registrar sus GUID.

**¿El cliente recibe también un informe?**

Los clientes pueden realizar el seguimiento del uso tanto de recursos individuales como de grupos de recursos definidos por los usuarios en Azure Portal. Los clientes no ven el uso desglosado por GUID.

**¿Es esta metodología similar al asociado digital de registro (DPOR)?**

Este nuevo método de conexión de la implementación y el uso en la solución de un asociado proporciona un mecanismo que permite vincular una solución de asociado al uso de Azure. DPOR está pensado para asociar un asociado de consultoría (integrador de sistemas) o de administración (proveedor de servicios administrados) con la suscripción de Azure de un cliente.

**¿Cuál es la ventaja de usar el formulario del generador de GUID de Azure Storage?**

El formulario del generador de GUID de Azure Storage garantiza la generación de un GUID del formato necesario. Además, si usa alguno de los métodos de seguimiento de planos de datos de Azure Storage, puede aprovechar el mismo GUID para el seguimiento de planos de control de Marketplace. Esto le permite aprovechar un único GUID unificado para la atribución de asociado sin tener que mantener distintos GUID.

**¿Puedo usar un disco duro virtual privado personalizado para una oferta de plantilla de solución en Azure Marketplace?**

Lamentablemente, no. La imagen de máquina virtual debe proceder de Azure Marketplace; consulte: [Guía de publicación de ofertas de máquinas virtuales en Azure Marketplace](marketplace-virtual-machines.md).

Puede crear una oferta de máquina virtual en Marketplace mediante el disco duro virtual personalizado y marcarla como privada para que nadie la vea y hacer referencia a esta máquina virtual en la plantilla de solución.

**¿No se pudo actualizar la propiedad *contentVersion* para la plantilla principal?**

A veces se puede producir un error al implementar la plantilla mediante TemplateLink desde otra plantilla que espere una versión de contentVersion anterior por alguna razón. La solución alternativa es usar la propiedad de metadatos:

```
"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "metadata": {
        "contentVersion": "1.0.1.0"
    },
    "parameters": {
```
