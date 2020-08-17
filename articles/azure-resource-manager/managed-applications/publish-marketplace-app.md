---
title: Aplicaciones administradas en Marketplace
description: En este artículo se describen las aplicaciones administradas de Azure que están disponibles a través de Marketplace.
author: tfitzmac
ms.topic: tutorial
ms.date: 07/17/2019
ms.author: tomfitz
ms.openlocfilehash: a5e2a3569c70404d64d24ecfc35a8258ea864a4f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87480431"
---
# <a name="tutorial-publish-azure-managed-applications-in-the-marketplace"></a>Tutorial: Publicación de aplicaciones administradas de Azure en Marketplace

Los proveedores pueden usar las aplicaciones administradas de Azure para ofrecer sus soluciones a todos los clientes de Azure Marketplace. Entre estos proveedores se incluyen los proveedores de servicios administrados (MSP), los fabricantes de software independientes (ISV) y los integradores de sistemas (SIs). Las aplicaciones administradas reducen la sobrecarga de mantenimiento para los clientes. Los proveedores pueden vender la infraestructura y el software a través de Marketplace. Pueden asociar los servicios y el soporte técnico operativo a las aplicaciones administradas. Para más información, consulte [Información general sobre las aplicaciones administradas](overview.md).

En este artículo se explica cómo puede publicar una aplicación en Marketplace y ponerla a disposición de los clientes.

## <a name="prerequisites-for-publishing-a-managed-application"></a>Requisitos previos para publicar una aplicación administrada

Para completar este artículo, ya debe tener el archivo .zip de la definición de la aplicación administrada. Para más información, consulte [Creación de la aplicación de catálogo de servicios](publish-service-catalog-app.md).

Hay varios requisitos previos de tipo empresarial. Son las siguientes:

* Su empresa o sus subsidiarias deben encontrarse en uno de los países o las regiones donde las ventas se permiten en Marketplace.
* El producto debe contar con una licencia que sea compatible con los modelos de facturación admitidos por Marketplace.
* Ponga el soporte técnico a disposición de los clientes de forma comercialmente razonable. Puede ser gratuito, de pago o a través del soporte técnico de la comunidad.
* Debe tener licencias para el software y las dependencias de software de terceros.
* Proporcione contenido que cumpla los criterios para que su oferta se publique en Marketplace y en Azure Portal.
* Acepte los términos del acuerdo del publicador y de las directivas de participación de Azure Marketplace.
* Acepte los Términos de uso, la Declaración de privacidad de Microsoft y el Contrato del programa Microsoft Azure Certified.

También debe tener una cuenta de Marketplace. Para crearla, consulte [Creación de una cuenta de Marketplace comercial en el Centro de partners](../../marketplace/partner-center-portal/create-account.md).

## <a name="create-a-new-azure-application-offer"></a>Creación de una nueva oferta de aplicación de Azure

Después de crear la cuenta de Partner Portal, está listo para crear su oferta de aplicación administrada.

### <a name="set-up-an-offer"></a>Configuración de una oferta

Una oferta de aplicación administrada se corresponde con una clase de oferta de producto de un publicador. Si tiene un nuevo tipo de aplicación que quiere que esté disponible en Marketplace, puede configurarla como una nueva oferta. Una oferta es una colección de SKU. Cada oferta aparece como su propia entidad en Marketplace.

1. Inicie sesión en [Cloud Partner Portal](https://cloudpartner.azure.com/).

1. En el panel de navegación de la izquierda, seleccione **+ Oferta nueva** > **Aplicaciones de Azure**.

1. En la vista **Edición**, puede consultar los formularios necesarios. Cada uno de estos formularios se describirá más adelante en este artículo.

## <a name="offer-settings-form"></a>Formulario de configuración de oferta

Los campos del formulario **Configuración de la oferta** son:

* **Id. de oferta**: se trata del identificador único que identifica la oferta en un perfil del anunciante. Este identificador se muestra en las direcciones URL de producto, las plantillas de Resource Manager y los informes de facturación. Puede contener solo caracteres alfanuméricos en minúscula o guiones (-). El identificador no puede terminar con un guión. Está limitado a un máximo de 50 caracteres. En cuanto se lanza una oferta, este campo se bloquea.
* **Id. del anunciante**: use esta lista desplegable para elegir el perfil del anunciante en el que quiere que se publique esta oferta. En cuanto se lanza una oferta, este campo se bloquea.
* **Name**: este nombre para mostrar para su oferta aparece en Marketplace y en el portal. Puede tener un máximo de 50 caracteres. Incluya un nombre de marca que identifique el producto. No incluya aquí el nombre de su empresa a menos que sea así como se comercializa. Si comercializa esta oferta en su propio sitio web, asegúrese de que el nombre sea exactamente el mismo que aparece en él.

Al acabar, seleccione **Guardar** para guardar lo que ha hecho hasta ahora.

## <a name="skus-form"></a>Formulario de SKU

El siguiente paso consiste en agregar SKU para la oferta.

Una SKU es la unidad de compra más pequeña de una oferta. Puede usar una SKU dentro de la misma clase de producto (oferta) para diferenciar entre:

* Distintas características que se admiten.
* Si la oferta es administrada o no administrada.
* Los modelos de facturación que se admiten.

La SKU aparece debajo de la oferta principal en Marketplace. como una entidad de compra propia en Azure Portal.

1. Seleccione **SKU** > **SKU nueva**.

1. Escriba un **Identificador de SKU**. Se trata del identificador único de la SKU dentro de una oferta. Este identificador se muestra en las direcciones URL de producto, las plantillas de Resource Manager y los informes de facturación. Puede contener solo caracteres alfanuméricos en minúscula o guiones (-). El identificador, que tendrá 50 caracteres como máximo, no puede terminar con un guion. En cuanto se lanza una oferta, este campo se bloquea. Puede tener varias SKU dentro de una oferta. Necesita una SKU para cada imagen que pretenda publicar.

1. Rellene la sección de **detalles de la SKU** en el formulario siguiente:

   Rellene los siguientes campos:

   * **Título**: especifique un título para esta SKU. Este título aparece en la galería para este elemento.
   * **Resumen**: proporcione un breve resumen de esta SKU. Este texto aparece debajo del título.
   * **Descripción**: escriba una descripción detallada de la SKU.
   * **Tipo de SKU**: los valores permitidos son *Aplicación administrada* y *Plantillas de solución*. En este caso, seleccione *Aplicación administrada*.
   * **Country/Region availability** (Disponibilidad de país o región): seleccione los países o las regiones donde la aplicación administrada está disponible.
   * **Precios**: proporcione un precio para la administración de la aplicación. Seleccione los países o las regiones disponibles antes de establecer el precio.

1. Agregue un nuevo paquete. Rellene la sección de **detalles del paquete** en el formulario siguiente:

   Rellene los siguientes campos:

   * **Versión**: escriba una versión para el paquete que vaya a cargar. Debe tener este formato: `{number}.{number}.{number}{number}`.
   * **Package file (.zip)** (Archivo del paquete [.zip]): este paquete contiene dos archivos necesarios comprimidos en uno .zip. Un archivo es una plantilla de Resource Manager que define los recursos que va a implementar en la aplicación administrada. El otro archivo define la [interfaz de usuario](create-uidefinition-overview.md) para que los consumidores implementen la aplicación administrada mediante el portal. En la interfaz de usuario, puede especificar los elementos que permiten a los consumidores proporcionar los valores de los parámetros.
   * **Id. de inquilino**: El identificador del inquilino de la cuenta a la que se va a acceder.
   * **Enable JIT Access?** (¿Habilitar acceso JIT?): Seleccione **Sí** para habilitar el [control de acceso just-in-time](request-just-in-time-access.md) para la cuenta. Cuando se habilita, se solicita acceso a la cuenta del consumidor durante un período específico. Para exigir que los consumidores de la aplicación administrada concedan el acceso permanente a la cuenta, seleccione **No**.
   * **Customize allowed customer actions?** (¿Personalizar las acciones de cliente permitidas?): Seleccione **Sí** para especificar qué acciones pueden realizar los consumidores en los recursos administrados.
   * **Allowed customer actions** (Acciones de cliente permitidas): Si selecciona **Sí** en el valor anterior, puede especificar las acciones que se permiten a los consumidores mediante el uso de [asignaciones de denegación para recursos de Azure](../../role-based-access-control/deny-assignments.md).

     Puede consultar las acciones disponibles en [Operaciones del proveedor de recursos de Azure Resource Manager](../../role-based-access-control/resource-provider-operations.md). Por ejemplo, para permitir que los consumidores reinicien las máquinas virtuales, agregue `Microsoft.Compute/virtualMachines/restart/action` a las acciones permitidas. La acción `*/read` se permite automáticamente, por lo que no es necesario incluir ese valor.
   * **PrincipalId**: esta propiedad es el identificador de Azure Active Directory de un usuario, de un grupo de usuarios o de una aplicación a quienes se haya concedido acceso a los recursos de la suscripción de los clientes. La definición de rol describe los permisos.
   * **Role Definition**: esta propiedad es una lista de todos los roles integrados de Azure que proporciona Azure AD. Puede seleccionar el rol que resulte más adecuado para administrar los recursos en nombre del cliente.
   * **Configuración de directivas**: aplique una directiva de [Azure Policy](../../governance/policy/overview.md) a la aplicación administrada para especificar los requisitos de cumplimiento para las soluciones implementadas. Entre las opciones disponibles, seleccione las directivas que se van a aplicar. En **Parámetros de directiva**, proporcione una cadena JSON con los valores del parámetro. Para las definiciones de directiva y el formato de los valores de parámetro, consulte [ejemplos de Azure Policy](../../governance/policy/samples/index.md).

Puede agregar varias autorizaciones. Se recomienda que cree un grupo de usuarios de AD y especifique su identificador en **PrincipalId**. De esta manera, puede agregar más usuarios al grupo de usuarios sin necesidad de actualizar la SKU.

Para más información sobre RBAC, consulte [Introducción al control de acceso basado en roles en Azure Portal](../../role-based-access-control/overview.md).

## <a name="marketplace-form"></a>Formulario de Marketplace

En el formulario de Marketplace se solicitan campos que aparecen en [Azure Marketplace](https://azuremarketplace.microsoft.com) y en [Azure Portal](https://portal.azure.com/).

### <a name="preview-subscription-ids"></a>Id. de suscripción de versión preliminar

Escriba una lista de identificadores de suscripciones de Azure que puedan tener acceso a la oferta una vez publicada. Puede usar estas suscripciones de lista blanca para probar la versión preliminar de la oferta antes de activarla. Puede compilar una lista de permitidos de hasta 100 suscripciones en el portal de asociados.

### <a name="suggested-categories"></a>Categorías sugeridas

Seleccione hasta cinco categorías de la lista con las que la oferta pueda asociarse mejor. Estas categorías se usan para asignar la oferta a las categorías de productos disponibles en [Azure Marketplace](https://azuremarketplace.microsoft.com) y en [Azure Portal](https://portal.azure.com/).

#### <a name="azure-marketplace"></a>Azure Marketplace

En el resumen de la aplicación administrada, se muestran los campos siguientes:

![Resumen de Marketplace](./media/publish-marketplace-app/publishvm10.png)

La pestaña **Información general** de la aplicación administrada muestra los campos siguientes:

![Introducción a Marketplace](./media/publish-marketplace-app/publishvm11.png)

La pestaña **Planes y precios** de la aplicación administrada muestra los campos siguientes:

![Planes de Marketplace](./media/publish-marketplace-app/publishvm15.png)

#### <a name="azure-portal"></a>Portal de Azure

En el resumen de la aplicación administrada, se muestran los campos siguientes:

![Resumen del portal](./media/publish-marketplace-app/publishvm12.png)

La información general de la aplicación administrada muestra los campos siguientes:

![Información general de Azure Portal](./media/publish-marketplace-app/publishvm13.png)

#### <a name="logo-guidelines"></a>Directrices para logotipos

Siga estas directrices para cualquier logotipo que cargue en Cloud Partner Portal:

*   El diseño de Azure tiene una paleta de colores simple. Utilice pocos colores primarios y secundarios en el logotipo.
*   Los colores del tema del portal son el blanco y el negro. No use estos colores como fondo de los logotipos. Use un color que haga que su logotipo destaque en el portal. Nosotros recomendamos usar colores primarios simples. *Si usa un fondo transparente, asegúrese de que el logotipo y el texto no son blancos, negros ni azules.*
*   No utilice un fondo degradado en el logotipo.
*   No coloque texto en el logotipo, ni siquiera el nombre de su empresa o de la marca. La apariencia del logotipo debe ser homogénea y evitar degradados.
*   Asegúrese de que el logotipo no quede estirado.

#### <a name="hero-logo"></a>Logotipo de imagen prominente

El logotipo de imagen prominente es opcional. El publicador puede decidir no cargar un logotipo de imagen prominente. Una vez cargado el icono de imagen prominente, no se puede eliminar. A partir de ese momento, el asociado debe seguir las instrucciones de Marketplace para los iconos de imágenes prominentes.

Siga estas instrucciones para el icono de imagen prominente:

*   El nombre para mostrar del publicador, el título del plan y el resumen largo de la oferta se muestran en color blanco. Por lo tanto, no use un color claro para el fondo del icono de imagen prominente. Los fondos transparentes y de color negro o blanco no pueden utilizarse en los iconos de imagen prominente.
*   Una vez que aparece la oferta, los elementos se insertan mediante programación dentro del logotipo de imagen prominente. Los elementos insertados incluyen el nombre para mostrar del publicador, el título del plan, el resumen largo de la oferta y el botón **Crear**. Por lo tanto, no escriba ningún texto mientras diseña el logotipo de imagen prominente. Dado que el texto se incluye mediante programación en ese espacio, deje espacios en blanco a la derecha. El espacio en blanco para el texto debe ser de 415 x 100 píxeles en la derecha. Se ve compensado por 370 píxeles desde la izquierda.

    ![Ejemplo de logotipo de imagen prominente](./media/publish-marketplace-app/publishvm14.png)

## <a name="support-form"></a>Formulario de soporte técnico

Rellene el formulario **Soporte técnico** con los contactos de soporte técnico de su empresa. Esta información podría contener contactos de ingeniería y contactos de soporte técnico al cliente.

## <a name="publish-an-offer"></a>Publicación de una oferta

Después de rellenar todas las secciones, seleccione **Publicar** para comenzar el proceso que hace que la oferta esté a disposición de los clientes.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre lo que sucede después de hacer clic en **Publicar**, consulte [Publicación de una oferta de aplicación de Azure](../../marketplace/partner-center-portal/create-new-azure-apps-offer.md).
* Para una introducción a las aplicaciones administradas, consulte [Introducción a las aplicaciones administradas](overview.md).
* Para información sobre cómo publicar una aplicación administrada del catálogo de servicios, consulte [Creación y publicación de una aplicación administrada del catálogo de servicios](publish-service-catalog-app.md).
