---
title: 'Creación de una oferta de contenedor de Azure: Azure Marketplace'
description: Aprenda a crear y publicar una oferta de contenedor para Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: keferna
ms.author: keferna
ms.date: 06/17/2020
ms.openlocfilehash: 12987ae687e9af59fdd71b3aa846612c947b9a90
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89649272"
---
# <a name="create-an-azure-container-offer-in-azure-marketplace"></a>Creación de una oferta de contenedor de Azure en Azure Marketplace

En este artículo se explica cómo crear y publicar una oferta de contenedor para Azure Marketplace. Antes de comenzar, debe [crear una cuenta de marketplace comercial en el Centro de partners](create-account.md), si aún no lo ha hecho. Asegúrese de que la cuenta está inscrita en el programa Marketplace comercial.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).

2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.

3. En la página Información general, seleccione **+ Nueva oferta** > **Azure Container**.

   ![Muestra el menú de navegación izquierdo.](./media/new-offer-azure-container.png)

> [!TIP]
> Después de publicar una oferta, las modificaciones que se realicen en el Centro de partners solo aparecen en las tiendas en línea cuando la oferta se vuelve a publicar. Asegúrese de volver a realizar la publicación siempre después de realizar cambios.

### <a name="offer-id-and-alias"></a>Identificador y alias de la oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
- Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
- No se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-overview"></a>Información general de la oferta

En la página **Información general de la oferta** se muestra una representación visual de los pasos necesarios para publicar esta oferta (tanto completados como siguientes), así como el tiempo necesario que debe tardar cada uno en completarse.

En esta página se muestran diferentes vínculos según el estado actual de la oferta. Por ejemplo:

- Si la oferta es un borrador: [Eliminación de una oferta borrador](update-existing-offer.md#delete-a-draft-offer)
- Si la oferta está activa: [Detención de la venta de una oferta](update-existing-offer.md#stop-selling-an-offer-or-plan)
- Si la oferta está en versión preliminar: [publicar](publishing-status.md#publisher-approval)
- Si no ha completado el cierre de sesión del anunciante: [Cancelación de la publicación](update-existing-offer.md#cancel-publishing)

## <a name="offer-setup"></a>Configuración de la oferta

Siga estos pasos para configurar la oferta.

### <a name="customer-leads--optional"></a>Clientes potenciales: opcional

Al publicar la oferta en el marketplace comercial con el Centro de partners, conéctela a un sistema de administración de relaciones con clientes (CRM). Esto le permite recibir información de contacto del cliente en cuanto alguien expresa interés en el producto o lo usa.

1. **Seleccione un destino de clientes potenciales al que quiera que enviemos los clientes potenciales**. El Centro de partners admite los siguientes sistemas CRM:

   - [Dynamics 365](commercial-marketplace-lead-management-instructions-dynamics.md) for Customer Engagement
   - [Marketo](commercial-marketplace-lead-management-instructions-marketo.md)
   - [Salesforce](commercial-marketplace-lead-management-instructions-salesforce.md)

   > [!NOTE]
   > Si su sistema CRM no aparece en la lista anterior, use [Tabla de Azure](commercial-marketplace-lead-management-instructions-azure-table.md) o [Punto de conexión HTTPS](commercial-marketplace-lead-management-instructions-https.md) para almacenar los datos del cliente potencial y, a continuación, exporte los datos al sistema CRM.

2. Conecte su oferta al destino de clientes potenciales cuando la publique en el Centro de partners.
3. Compruebe que la conexión al destino de clientes potenciales está configurada correctamente. Después de publicarla en el Centro de partners, se valida la conexión y se le envía un cliente potencial de prueba. Mientras obtiene una vista previa de la oferta antes de publicarla, también puede probar la conexión de los clientes potenciales tratando de adquirir la oferta en el entorno de versión preliminar.
4. Asegúrese de que la conexión con el destino de clientes potenciales permanece actualizada para no perder ningún cliente potencial.

Estos son algunos recursos adicionales de administración de clientes potenciales:

- [Introducción a la administración de clientes potenciales](commercial-marketplace-get-customer-leads.md)
- [Preguntas frecuentes de la administración de clientes potenciales](../lead-management-for-cloud-marketplace.md#frequently-asked-questions)
- [Errores comunes de la configuración de clientes potenciales](../lead-management-for-cloud-marketplace.md#publishing-config-errors)
- [Introducción a la administración de clientes potenciales](https://assetsprod.microsoft.com/mpn/cloud-marketplace-lead-management.pdf) en formato PDF (asegúrese de que el bloqueador de elementos emergentes está desactivado)

Seleccione **Guardar borrador** antes de continuar.

### <a name="properties"></a>Propiedades

Esta página le permite definir las categorías que se usan para agrupar su oferta en Marketplace y los contratos legales que dan soporte a su oferta.

#### <a name="category"></a>Category

Seleccione categorías y subcategorías para colocar la oferta en las áreas de búsqueda adecuadas del marketplace. En la descripción de la oferta, asegúrese de indicar la forma en que la oferta admite estas categorías. Seleccione:

- Al menos una y un máximo de dos categorías, incluidas una categoría principal y una secundaria (opcional).
- Hasta dos subcategorías para cada categoría principal o secundaria. Si no hay ninguna subcategoría aplicable a la oferta, seleccione **No aplicable**.

Vea la lista completa de categorías y subcategorías en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md). Los contenedores siempre aparecen en **Contenedores** y, después, en la categoría **Imágenes de contenedor**.

#### <a name="legal"></a>Información legal

Debe proporcionar los términos y condiciones de la oferta. Hay dos opciones:

- Usar el Contrato estándar para marketplace comercial de Microsoft.
- Proporcionar sus propios términos y condiciones.

#### <a name="standard-contract-for-the-microsoft-commercial-marketplace"></a>Contrato estándar para marketplace comercial de Microsoft

Ofrecemos una plantilla de contrato estándar para facilitar las transacciones en Marketplace comercial. Puede optar por ofrecer su solución con el contrato estándar, que los clientes solo deben consultar y aceptar una vez. Esta es una buena opción si no desea crear términos y condiciones personalizados.

Para más información sobre el Contrato estándar, consulte [Contrato estándar para marketplace comercial de Microsoft](../standard-contract.md). También puede descargar el [contrato estándar](https://go.microsoft.com/fwlink/?linkid=2041178) en formato PDF (asegúrese de que el bloqueador de elementos emergentes está desactivado).

Para usar el contrato estándar, active la casilla **Use the Standard Contract for Microsoft's commercial marketplace (Usar el contrato estándar para Marketplace comercial de Microsoft) (../standard-contract.md).

> [!NOTE]
> Después de publicar una oferta con el contrato estándar para Marketplace comercial de Microsoft, no puede usar sus propios términos y condiciones personalizados. Puede ofrecer su solución con el Contrato estándar o con sus propios términos y condiciones.

:::image type="content" source="media/azure-create-container-offer-images/azure-create-1-standard-contract.png" alt-text="Muestra el uso de la casilla del Contrato estándar para marketplace comercial de Microsoft.":::

##### <a name="your-own-terms-and-conditions"></a>Términos y condiciones propios

Si quiere proporcionar sus propios términos y condiciones personalizados, especifíquelos en el cuadro **Términos y condiciones**. Puede escribir un número ilimitado de caracteres de texto en este cuadro. Los clientes deben aceptar estos términos para poder probar la oferta.

Seleccione **Guardar borrador** antes de pasar a la siguiente sección, Descripción de la oferta.

## <a name="offer-listing"></a>Descripción de la oferta

Esta página le permite definir los detalles de la oferta que se muestran en el marketplace comercial. Esto incluye el nombre de la oferta, la descripción y las imágenes.

> [!NOTE]
> No es necesario que los detalles de la oferta estén en inglés si la descripción de la oferta comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es puede proporcionar un vínculo al contenido de la oferta en un idioma distinto al utilizado en la descripción de la oferta.

### <a name="name"></a>Nombre

El nombre que se escribe aquí se muestra como el título de la oferta. Este campo se rellena previamente con el texto que escribió en el cuadro **Alias de la oferta** al crear la oferta. Puede cambiar este nombre más adelante.

El nombre:

- Puede contener marcas comerciales (y puede incluir símbolos de marca comercial y copyright).
- No puede tener más de 50 caracteres.
- No puede incluir emojis.

### <a name="search-results-summary"></a>Resumen de los resultados de la búsqueda

Breve descripción de la oferta. Puede tener una longitud de hasta 100 caracteres y se usa en los resultados de la búsqueda de Marketplace.

### <a name="long-summary"></a>Resumen largo

Descripción más detallada de la oferta. Puede tener una longitud de hasta 256 caracteres y se usa en los resultados de la búsqueda de Marketplace.

### <a name="description"></a>Descripción

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Long description-3](./includes/long-description-3.md)]

#### <a name="privacy-policy-link"></a>Vínculo de la directiva de privacidad

Escriba la dirección web de la directiva de privacidad de su organización. Usted es responsable de garantizar que la oferta cumpla con las leyes y reglamentos de privacidad. También es responsable de publicar una directiva de privacidad válida en su sitio web.

#### <a name="useful-links"></a>Vínculos útiles

Proporcione documentos en línea complementarios sobre su oferta. Puede agregar hasta 25 vínculos. Para agregar un vínculo, seleccione **+ Agregar un vínculo** y, a continuación, rellene los campos siguientes:

- **Título**: los clientes lo verán en la página de detalles de la oferta.
- **Vínculo (URL)** : escriba un vínculo para que los clientes vean el documento en línea. El vínculo debe empezar por http:// o https://.

### <a name="contact-information"></a>Información de contacto

Debe proporcionar el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes, pero está disponible para Microsoft. También se puede proporcionar a los asociados del programa Proveedor de soluciones en la nube (CSP).

- Contacto de soporte técnico (se requiere): Para preguntas generales de soporte técnico.
- Contacto de ingeniería (se requiere): Para preguntas técnicas y problemas de certificación.
- Contacto del programa CSP (opcional): Para preguntas de revendedores relacionadas con el programa CSP.

En la sección **Contacto de soporte técnico**, proporcione el **Sitio web de soporte técnico**, donde los asociados pueden encontrar soporte técnico para la oferta según esté disponible en Azure global, Azure Government o ambos.

En la sección **Contacto del programa CSP**, proporcione el vínculo (**Materiales de marketing del programa CSP**), donde los asociados del programa CSP pueden encontrar materiales de marketing para la oferta.

#### <a name="additional-marketplace-listing-resources"></a>Recursos adicionales de la descripción de Marketplace

Para más información sobre la creación de listas de ofertas, consulte [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md).

### <a name="marketplace-images"></a>Imágenes de Marketplace

Proporcione logotipos e imágenes para usarlos con la oferta. Todas las imágenes deben estar en formato PNG. Las imágenes borrosas se rechazarán.

[!INCLUDE [logotips](../includes/graphics-suggestions.md)]

>[!Note]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear un logotipo **Pequeño** y un logotipo **Mediano**. Opcionalmente, puede reemplazarlos por imágenes diferentes más adelante.

- **Grande** (de 216 x 216 a 350 x 350 píxeles, obligatorio)
- **Mediano** (90 x 90 píxeles, opcional)
- **Pequeño** (48 x 48 píxeles, opcional)

Estos logotipos se usan en distintos lugares del anuncio:

[!INCLUDE [logos-azure-marketplace-only](../includes/logos-azure-marketplace-only.md)]

[!INCLUDE [Logo tips](../includes/graphics-suggestions.md)]

#### <a name="screenshots-optional"></a>Capturas de pantalla (opcional)

Agregue hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Cada una debe tener un tamaño de 1280 x 720 píxeles y el formato PNG.

#### <a name="videos-optional"></a>Vídeos (opcional)

Agregue hasta cinco vídeos que muestren la oferta. Escriba el nombre del vídeo, su dirección web y una imagen en miniatura en formato PNG del vídeo con un tamaño de 1280 x 720 píxeles.

#### <a name="offer-examples"></a>Ejemplos de oferta

En los siguientes ejemplos se muestra cómo aparecen los campos de las listas de ofertas en diferentes ubicaciones de la oferta.

Muestra la página **Descripción de la oferta** en Azure Marketplace:

:::image type="content" source="media/azure-create-container-offer-images/azure-create-6-offer-listing-mkt-plc.png" alt-text="Muestra la página Descripción de la oferta en Azure Marketplace." :::

Muestra los resultados de la búsqueda en Azure Marketplace:

:::image type="content" source="media/azure-create-container-offer-images/azure-create-7-search-results-mkt-plc.png" alt-text="Muestra los resultados de la búsqueda en Azure Marketplace.":::

Muestra la página **Descripción de la oferta** en Azure Portal:

:::image type="content" source="media/azure-create-container-offer-images/azure-create-8-offer-listing-portal.png" alt-text="Muestra la página Descripción de la oferta en Azure Portal.":::

Muestra los resultados de la búsqueda en Azure Portal:

:::image type="content" source="media/azure-create-container-offer-images/azure-create-9-search-results-portal.png" alt-text="Muestra los resultados de la búsqueda en Azure Portal.":::

## <a name="preview"></a>Versión preliminar

En la pestaña Versión preliminar puede elegir un **público preliminar** limitado para validar la oferta antes de su publicación.

> [!IMPORTANT]
> Después de ver la oferta en **Versión preliminar**, debe seleccionar **Publicar** para publicar la oferta al público.

Especifique el público de versión preliminar mediante los GUI de identificador de suscripción de Azure con una descripción adicional de cada uno. Los clientes no pueden ver ninguno de estos campos.

> [!NOTE]
> Puede encontrar el identificador de suscripción de Azure en la página Suscripciones de Azure Portal.

Agregue al menos un id. de suscripción de Azure, ya sea de forma individual (hasta 10) o mediante la carga de un archivo CSV (hasta 100). Al agregar estos identificadores de suscripción, puede determinar quién puede obtener una vista previa de la oferta antes de publicarla. Si la oferta ya está publicada, puede elegir un público preliminar para probar los cambios o las actualizaciones de la oferta.

Seleccione **Guardar borrador** antes de continuar.

## <a name="plan-overview"></a>Información general del plan

Esta pestaña le permite ofrecer diversas opciones de planes dentro de la misma oferta. Los planes (anteriormente denominados SKU) pueden diferir en cuanto a las nubes disponibles, como las nubes globales, las nubes gubernamentales y la imagen de referencia del plan. Para publicar la oferta en el marketplace comercial, debe configurar al menos un plan.

Puede crear hasta 100 planes para cada oferta, de los cuales hasta 45 pueden ser privados. Más información sobre los planes privados en [Ofertas privadas en el marketplace comercial de Microsoft](../private-offers.md).

Después de crear sus planes, la pestaña **Información general del plan** muestra lo siguiente:

- Nombres de los planes
- Modelo de precios
- Regiones de Azure (globales o gubernamentales)
- Estado de publicación actual
- Acciones disponibles

Las acciones disponibles en Información general del plan varían en función del estado actual del plan del plan. Incluyen:

- **Eliminar borrador**: si el estado del plan es un borrador.
- **Detener el plan de venta**: si el estado el plan es publicado.

### <a name="create-new-plan"></a>Creación de un nuevo plan

Seleccione **Crear nuevo plan**. Aparece el cuadro de diálogo **Nuevo plan**.

En el cuadro **Identificador de plan**, cree un identificador de plan único para cada plan de la oferta. Este identificador será visible para los clientes en la dirección web del producto. Use solo letras minúsculas y números, guiones o caracteres de subrayado, y un máximo de 50 caracteres.

> [!NOTE]
> El id. de oferta no se puede cambiar después de seleccionar **Crear**.

En el cuadro **Nombre del plan**, escriba un nombre para este plan. Los clientes verán este nombre al decidir que plan van a seleccionar en su oferta. Cree un nombre único para cada plan de esta oferta. Por ejemplo, puede usar el nombre de oferta **Windows Server** con los planes **Windows Server 2016** y **Windows Server 2019**.

### <a name="plan-setup"></a>Configuración del plan

Esta pestaña le permite elegir las nubes en las que está disponible el plan. Sus respuestas de esta pestaña afectan a los campos que se muestran en otras pestañas.

#### <a name="azure-regions"></a>Regiones de Azure

Todos los planes de las ofertas de Azure Container Service están disponibles automáticamente en **Azure Global**.  Los clientes pueden usar su plan en todas las regiones globales de Azure que usan el marketplace comercial. Para obtener más información, consulte [Disponibilidad geográfica y soporte técnico de moneda](../marketplace-geo-availability-currencies.md).

Seleccione la opción [Azure Government](../../azure-government/documentation-government-welcome.md) para que la solución aparezca aquí. Es una nube de la comunidad gubernamental con acceso controlado para los clientes de agencias gubernamentales tribales, locales, estatales o federales de Estados Unidos, así como para los asociados aptos para abastecerlos. Como publicador, es responsable de los controles de cumplimiento, las medidas de seguridad y los procedimientos recomendados para esta comunidad en la nube. Azure Government usa redes y centros de datos aislados físicamente (ubicados solo en Estados Unidos). Antes de la [publicación](../../azure-government/documentation-government-manage-marketplace-partners.md) en Azure Government, pruebe y valide la solución en esa área, ya que los resultados pueden ser diferentes. Para crear y probar la solución, solicite una cuenta de prueba desde [Prueba de Microsoft Azure Government](https://azure.microsoft.com/global-infrastructure/government/request/).

> [!NOTE]
> Una vez que el plan esté publicado y disponible en una región específica, no se puede quitar esa región.

#### <a name="azure-government-certifications"></a>Certificaciones de Azure Government

Esta opción es visible únicamente si **Azure Government** está seleccionada en **Regiones de Azure**.

Los servicios de Azure Government controlan datos que están sujetos a determinados reglamentos y requisitos gubernamentales. Por ejemplo, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 y CJIS.

Para mostrar sus certificaciones de estos programas, puede proporcionar hasta 100 vínculos que las describan. Pueden ser vínculos a sus listas en el programa directamente o a su propio sitio web. Estos vínculos solo son visibles para los clientes de Azure Government.

### <a name="plan-listing"></a>Lista del plan

Esta pestaña muestra información específica de cada plan de la oferta actual.

### <a name="plan-name"></a>Nombre del plan

Se rellena previamente con el nombre que asignó al plan cuando lo creó. Puede cambiar este nombre según sea necesario. Puede tener una longitud máxima de 50 caracteres. Este nombre aparece como el título de este plan en Azure Marketplace y Azure Portal. Se usa como el nombre de módulo predeterminado cuando el plan está listo para usarse.

### <a name="plan-summary"></a>Resumen del plan

Breve resumen del plan de software (no de la oferta). Este resumen aparece en los resultados de la búsqueda de Azure Marketplace y puede contener hasta 100 caracteres.

### <a name="plan-description"></a>Description del plan

Describa lo que hace que este plan sea único, así como las diferencias entre los planes de la oferta. No describa la oferta, solo el plan. Esta descripción aparecerá en Azure Marketplace y en Azure Portal en la página **Descripción de la oferta**. Puede ser el mismo contenido que proporcionó en el resumen del plan y contener hasta 2000 caracteres.

Después de completar estos campos seleccione **Guardar**.

#### <a name="plan-examples"></a>Ejemplos de plan

En los siguientes ejemplos se muestra cómo aparecen los campos de descripción de la oferta en las distintas vistas.

Estos son los campos de Azure Marketplace cuando se visualizan los detalles del plan:

:::image type="content" source="media/azure-create-container-offer-images/azure-create-10-plan-details-mtplc.png" alt-text="Muestra los campos que se ven al visualizar los detalles del plan en Azure Marketplace.":::

Estos son los detalles del plan en Azure Portal:

:::image type="content" source="media/azure-create-container-offer-images/azure-create-11-plan-details-portal.png" alt-text="Muestra los detalles del plan en Azure Portal.":::

### <a name="plan-availability"></a>Disponibilidad del plan

Si desea ocultar la oferta publicada para que los clientes no puedan buscarla, examinarla ni comprarla en el marketplace, seleccione la casilla **Ocultar plan** en la pestaña **Disponibilidad**.

Este campo se usa cuando:

- La oferta está pensada para usarse solo indirectamente cuando se hace referencia a esta desde otra aplicación.
- La oferta no debe adquirirse individualmente.
- El plan se usó para las pruebas iniciales y ya no es relevante.
- El plan se ha usado para las ofertas temporales o estacionales y ya no debe ofrecerse.

## <a name="technical-configuration"></a>Configuración técnica

Las imágenes de contenedor se deben hospedar en una instancia privada de [Azure Container Registry](https://azure.microsoft.com/services/container-registry/). En la pestaña **Configuración técnica**, proporcione información de referencia para el repositorio de imágenes de contenedor de Azure Container Registry.

Una vez publicada la oferta, la imagen de contenedor se copia en Azure Marketplace en un registro de contenedor público específico. Todas las solicitudes para usar la imagen de contenedor se atienden desde el registro de contenedor público de Azure Marketplace, no desde su registro privado. Para más información, consulte [Preparación de los recursos técnicos del contenedor de Azure](create-azure-container-technical-assets.md).

### <a name="image-repository-details"></a>Detalles del repositorio de imágenes

Proporcione la siguiente información en la pestaña **Detalles del repositorio de imágenes**.

**Identificador de suscripción de Azure**: proporcione el identificador de suscripción en el que se indica el uso de recursos y se facturan los servicios para la instancia de Azure Container Registry que incluye la imagen de contenedor. Puede encontrar este identificador en la [página Suscripciones](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) de Azure Portal.

**Nombre del grupo de recursos de Azure**: proporcione el nombre del [grupo de recursos](../../azure-resource-manager/management/manage-resource-groups-portal.md) que contiene la instancia de Azure Container Registry con la imagen de contenedor. El grupo de recursos debe ser accesible en el identificador de suscripción (anterior). Puede encontrar el nombre en la página [Grupos de recursos](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceGroups) en Azure Portal.

**Nombre de Azure Container Registry**: proporcione el nombre de la instancia de [Azure Container Registry](../../container-registry/container-registry-intro.md) que tiene la imagen de contenedor. El registro de contenedor debe estar en el grupo de recursos de Azure que proporcionó anteriormente. Incluya solo el nombre del registro, no el nombre completo del servidor de inicio de sesión. Asegúrese de omitir la parte **azurecr.io** del nombre. Puede encontrar el nombre del registro en la [página Registros de contenedor](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.ContainerRegistry%2Fregistries) en Azure Portal.

**Nombre de usuario del administrador de Azure Container Registry**: proporcione el [nombre de usuario del administrador](../../container-registry/container-registry-authentication.md#admin-account) asociado a la instancia de Azure Container Registry que tiene la imagen de contenedor. El nombre de usuario y la contraseña son necesarios para asegurarse de que la empresa tiene acceso al registro. Para obtener el nombre de usuario y la contraseña de administrador, establezca la propiedad **admin-enabled** en **True** mediante la Interfaz de la línea de comandos (CLI) de Azure. Opcionalmente, puede establecer **Usuario administrador** en **Habilitar** en Azure Portal.

 :::image type="content" source="media/azure-create-container-offer-images/azure-create-12-update-container-registry-edit.png" alt-text="Muestra el cuadro de diálogo Actualizar registro de contenedor.":::

**Contraseña para Azure Container Registry** – proporcione la contraseña del nombre de usuario administrador asociado a la instancia de Azure Container Registry que tiene la imagen de contenedor. El nombre de usuario y la contraseña son necesarios para asegurarse de que la empresa tiene acceso al registro. Puede obtener la contraseña en Azure Portal desde **Container Registry** > **Claves de acceso** o bien con la CLI de Azure con el [comando show](https://docs.microsoft.com/cli/azure/acr/credential?view=azure-cli-latest#az-acr-credential-show).

:::image type="content" source="media/azure-create-container-offer-images/azure-create-13-access-keys.png" alt-text="Muestra el menú de la clave de acceso.":::

**Nombre del repositorio en Azure Container Registry**. Proporcione el nombre del repositorio de Azure Container Registry que tiene su imagen. Incluya el nombre del repositorio cuando inserte la imagen en el registro. Para encontrar el nombre del repositorio, puede ir a la página [Container Registry](https://azure.microsoft.com/services/container-registry/) > **Repositorios**. Para más información, consulte [Visualización de los repositorios de registros de contenedor en Azure Portal](../../container-registry/container-registry-repositories.md).

> [!NOTE]
> Una vez establecido el nombre, no se puede cambiar. Use un nombre único para cada oferta de su cuenta.

### <a name="image-tags-for-new-versions-of-your-offer"></a>Etiquetas de imagen para las nuevas versiones de la oferta

Los clientes deben poder obtener actualizaciones automáticamente de Azure Marketplace cuando publique una actualización. Si no quieren realizar la actualización, deben poder mantener una versión específica de la imagen. Para hacerlo, puede agregar nuevas etiquetas de imagen cada vez que realice una actualización en la imagen.

### <a name="image-tag"></a>Etiqueta de imagen

Este campo debe incluir una etiqueta **más reciente** que apunte a la versión más reciente de la imagen en todas las plataformas admitidas. Debe incluir al menos una etiqueta de versión (por ejemplo, que empiece por xx.xx.xx, donde xx es un número). Los clientes deben usar [etiquetas de manifiesto](https://github.com/estesp/manifest-tool) para dirigirse a varias plataformas. Todas las etiquetas a las que hace referencia una etiqueta de manifiesto también se deben agregar para que podamos cargarlas.

Todas las etiquetas de manifiesto (excepto la etiqueta más reciente) deben comenzar por X.Y **-** o X.Y.Z-, donde X, Y, Z son números enteros. Por ejemplo, si una etiqueta **más reciente** apunta a 1.0.1-linux-x64, 1.0.1-linux-arm32 y 1.0.1-windows-arm32, estas seis etiquetas se deben agregar a este campo. Para más información, consulte [Preparación de los recursos técnicos del contenedor de Azure](create-azure-container-technical-assets.md).

> [!NOTE]
> No olvide agregar una etiqueta de prueba a la imagen para facilitar su identificación durante las pruebas.

## <a name="review-and-publish"></a>Revisión y publicación

Una vez que haya completado todas las secciones necesarias de la oferta, puede enviarla para su revisión y publicación.

En la esquina superior derecha del portal, seleccione **Revisar y** **publicar**.

En la página de revisión, puede:

- Ver el estado de finalización de cada sección de la oferta. No puede realizar la publicación hasta que todas las secciones de la oferta estén marcadas como completas.
  - **No iniciada**: la sección no se ha iniciado y se debe completar.
  - **Incompleta**: la sección tiene errores que se deben corregir o se requiere que proporcione más información. Consulte las secciones anteriores de este documento para obtener ayuda.
  - **Completar**: incluye todos los datos necesarios sin errores. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- Proporcione instrucciones de prueba al equipo de certificación para asegurarse de que la oferta se ha probado correctamente. Además, proporcione cualquier nota complementaria que resulte útil para entender su oferta.

Para enviar la oferta para su publicación, seleccione **Publicar**.

Le enviaremos un correo electrónico para avisarle cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación.

Para ofrecer la oferta al público, vaya al Centro de partners y seleccione **Publicar**.

## <a name="next-step"></a>Paso siguiente

- [Actualización de una oferta existente en Marketplace comercial](update-existing-offer.md)
