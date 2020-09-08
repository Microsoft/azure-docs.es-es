---
title: 'Creación de una oferta de aplicación de Azure: Marketplace comercial de Microsoft'
description: Conozca los pasos y las consideraciones para crear una oferta de aplicación de Azure en el portal de Marketplace comercial del Centro de partners. Puede mostrar o vender su oferta de aplicación de Azure en Azure Marketplace o a través del programa Proveedor de soluciones en la nube (CSP).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: AarathiN
ms.author: aarathin
ms.date: 07/14/2020
ms.openlocfilehash: 071d3fc972e99d61a226492350b7eee146584df6
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815679"
---
# <a name="create-an-azure-application-offer"></a>Creación de una oferta de aplicación de Azure

En este artículo se explican los pasos y las consideraciones para crear una oferta de aplicación de Azure en Marketplace comercial. Antes de crear una oferta de aplicación de Azure, debe estar familiarizado con estos conceptos.

Antes de publicar una nueva oferta de aplicación de Azure, [cree una cuenta de Marketplace comercial en el Centro de partners](create-account.md) y asegúrese de que su cuenta esté inscrita en el programa de Marketplace comercial.

## <a name="before-you-begin"></a>Antes de empezar

Para diseñar, compilar y probar ofertas de aplicaciones de Azure, se necesitan conocimientos técnicos de la plataforma de Azure y de las tecnologías que se usan para crear la oferta. El equipo de ingeniería debe tener conocimientos sobre las tecnologías de Microsoft siguientes:

* Conocimientos básicos de los [Servicios de Azure](https://azure.microsoft.com/services/).
* Cómo [diseñar y estructurar las aplicaciones de Azure](https://azure.microsoft.com/solutions/architecture/).
* Conocimientos prácticos de [Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/), [Azure Storage](https://azure.microsoft.com/services/?filter=storage#storage) y [redes de Azure](https://azure.microsoft.com/services/?filter=networking#networking).
* Conocimientos prácticos de [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/).
* Conocimientos prácticos de [JSON](https://www.json.org/).

### <a name="technical-documentation-and-resources"></a>Documentación y recursos técnicos

Revise los siguientes recursos mientras prepara su oferta de aplicación de Azure para Marketplace comercial.

* [Nociones sobre las plantillas de Azure Resource Manager](../../azure-resource-manager/resource-group-authoring-templates.md)

* Guías de inicio rápido:

    * [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/)
    * [Guía de procedimientos recomendados para plantillas de Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md)
    * [Publicación de definición de aplicación](../../managed-applications/publish-service-catalog-app.md)
    * [Implementación de la aplicación de catálogo de servicios](../../managed-applications/deploy-service-catalog-quickstart.md)

* Tutoriales:

    * [Creación de archivos de definición](../../managed-applications/publish-service-catalog-app.md)
    * [Publicación de la aplicación Marketplace](../../managed-applications/publish-marketplace-app.md)

* Ejemplos:

    * [CLI de Azure](../../managed-applications/cli-samples.md)
    * [Azure PowerShell](../../managed-applications/powershell-samples.md)
    * [Soluciones para aplicaciones administradas](../../managed-applications/sample-projects.md)

En el vídeo [Creación de plantillas de soluciones y aplicaciones administradas para Azure Marketplace](https://channel9.msdn.com/Events/Build/2018/BRK3603) se proporciona una completa introducción al tipo de oferta de aplicación de Azure:

* Qué tipos de oferta están disponibles
* Qué recursos técnicos se necesitan
* Cómo crear una plantilla de Azure Resource Manager
* Desarrollo y prueba de la interfaz de usuario de la aplicación
* Cómo publicar la oferta de la aplicación
* El proceso de revisión de la aplicación

### <a name="suggested-tools"></a>Herramientas sugeridas

Elija uno o ambos de los entornos de scripting siguientes para ayudar a administrar la aplicación de Azure:

* [Azure PowerShell](https://docs.microsoft.com/powershell/azure/)
* [CLI de Azure](https://docs.microsoft.com/cli/azure)

Se recomienda agregar las herramientas siguientes al entorno de desarrollo:

* [Explorador de Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md)
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * Extensión: [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
    * Extensión: [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
    * Extensión: [Prettify JSON](https://marketplace.visualstudio.com/items?itemName=mohsen1.prettify-json)

Puede revisar las herramientas disponibles en la página [Herramientas de desarrollo de Azure](https://azure.microsoft.com/tools/). Asimismo, si usa Visual Studio, [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

## <a name="types-of-azure-application-plans"></a>Tipos de planes de la aplicación de Azure

Hay dos tipos de planes de la aplicación de Azure: plantillas de solución y aplicaciones administradas.

* La **plantilla de solución** es uno de los principales mecanismos para publicar una solución en Marketplace. Use este tipo de plan cuando la solución requiera automatizar además la implementación y la configuración para más de una sola máquina virtual. Con una plantilla de solución, puede automatizar el suministro de más de un recurso, incluyendo las máquinas virtuales, las redes y los recursos de almacenamiento para proporcionar soluciones IaaS complejas.  Para más información sobre la creación de plantillas de solución, consulte [Azure Resource Manager](../../azure-resource-manager/resource-group-overview.md).

* La **aplicación administrada** es similar a las plantillas de solución, aunque hay una diferencia importante. En una aplicación administrada, los recursos se implementan en un grupo de recursos que está administrado por el editor de la aplicación. El grupo de recursos está presente en la suscripción del consumidor, pero una identidad en el inquilino del editor tiene acceso al grupo de recursos. Como editor, puede especificar el costo del soporte técnico continuado de la solución. Use aplicaciones administradas para crear y entregar con facilidad a los clientes aplicaciones llave en mano totalmente administradas.  Para más información sobre las ventajas y los tipos de aplicaciones administradas, consulte [Introducción a las aplicaciones administradas de Azure](../../managed-applications/overview.md).

## <a name="technical-requirements"></a>Requisitos técnicos

Todas las aplicaciones de Azure incluyen al menos dos archivos en la carpeta raíz de un archivo `.zip`:

* Un archivo de plantilla de Resource Manager llamado [mainTemplate.json](../../azure-resource-manager/resource-group-overview.md).  Esta plantilla define los recursos que se van a implementar en la suscripción de Azure del cliente. Para ver ejemplos de plantillas de Resource Manager, consulte la [galería de plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/) o el repositorio [GitHub: Plantillas de inicio rápido de Azure Resource Manager](https://github.com/azure/azure-quickstart-templates) correspondiente.

* Una definición de interfaz de usuario para la experiencia de creación de aplicaciones de Azure llamada [createUiDefinition.json](../../managed-applications/create-uidefinition-overview.md).  En la interfaz de usuario, puede especificar los elementos que permiten a los consumidores proporcionar los valores de los parámetros.

Todas las nuevas ofertas de la aplicación de Azure deben incluir un [GUID de atribución de uso del cliente para asociados de Azure](../azure-partner-customer-usage-attribution.md). 

Para información sobre los requisitos de publicación de cada plan de aplicación, consulte [Requisitos de publicación de ofertas de plantillas de solución](../marketplace-solution-templates.md) y [Requisitos de publicación de ofertas de aplicaciones administradas](../marketplace-managed-apps.md).

## <a name="create-a-new-offer"></a>Crear una nueva oferta

>[!NOTE]
>Después de publicar una oferta, las modificaciones que realice en ella en el Centro de partners no aparecerán en las tiendas en línea hasta que vuelva a publicarla. Tras realizar los cambios, asegúrese siempre de volver a publicar la oferta.

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).

1. En el menú izquierdo, seleccione **Marketplace comercial** > **Información general**.

1. En la página Información general, seleccione **+ Nueva oferta** > **Aplicación de Azure**.

    ![Muestra el menú de navegación izquierdo.](./media/new-offer-azure-app.png)

1. En la página **Nueva oferta**, escriba un **identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

     * Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
     * Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
     * El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

1. Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

     * Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
     * El alias de la oferta no se puede cambiar después de seleccionar **Crear**.

1. Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-setup"></a>Configuración de la oferta

La página **Offer setup** (Configuración de la oferta) es donde puede configurar una versión de prueba y la administración de clientes potenciales de la oferta.

### <a name="test-drive"></a>Versión de prueba

Una versión de prueba es una excelente manera de presentar la oferta a clientes potenciales, ya que se les brinda la oportunidad de «probar antes de comprar», lo que aumenta la conversión y la generación de clientes potenciales altamente cualificados. [Más información sobre las versiones de prueba](../what-is-test-drive.md).

Para habilitar una versión de prueba durante un período de tiempo determinado, seleccione la casilla **Habilitar una versión de prueba**. Para eliminar la versión de prueba de la oferta, desactive esta casilla. Configure el entorno de la versión de prueba en la sección [Configuración técnica de la versión de prueba](#test-drive-technical-configuration) más adelante en este tema.

Para más información, consulte [Versión de prueba de la oferta en Marketplace comercial](test-drive.md). También puede leer sobre los [procedimientos recomendados de la versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices) y descargar el [PDF de información general de las versiones de prueba ](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf) (asegúrese de que el bloqueador de elementos emergentes está desactivado).

>[!Note]
>Dado que todas las aplicaciones de Azure se implementan mediante una plantilla de Azure Resource Manager, el único tipo de versión de prueba disponible para una aplicación de Azure es una [versión de prueba basada en Azure Resource Manager](../azure-resource-manager-test-drive.md).

### <a name="customer-leads"></a>Clientes potenciales

[!INCLUDE [Connect lead management](./includes/connect-lead-management.md)]

Para obtener más información, consulte [Introducción a la administración de clientes potenciales](./commercial-marketplace-get-customer-leads.md).

Seleccione **Guardar borrador** antes de continuar.

## <a name="properties"></a>Propiedades

La página de **Propiedades** le permite definir las categorías que se usan para agrupar su oferta en Marketplace, la versión de la aplicación y los contratos legales que respaldan su oferta.

### <a name="category"></a>Category

Seleccione categorías y subcategorías para colocar la oferta en las áreas de búsqueda adecuadas del marketplace. En la descripción de la oferta, asegúrese de indicar la forma en que la oferta admite estas categorías. Seleccione:

- Al menos una y un máximo de dos categorías, incluidas una categoría principal y una secundaria (opcional).
- Hasta dos subcategorías para cada categoría principal o secundaria. Si no hay ninguna subcategoría aplicable a la oferta, seleccione **No aplicable**.

Vea la lista completa de categorías y subcategorías en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md).

### <a name="legal"></a>Información legal

[!INCLUDE [Commercial marketplace Standard Contract](./includes/marketplace-contract-text.md)]

## <a name="offer-listing"></a>Descripción de la oferta

En esta página se administran la copia y las imágenes de la oferta de Marketplace comercial.

### <a name="marketplace-details"></a>Detalles del marketplace

> [!NOTE]
> No es necesario que la oferta (por ejemplo, la descripción, los documentos, las capturas de pantallas y los términos de uso) esté en inglés si la descripción de la oferta comienza por la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es aceptable proporcionar una *dirección URL de vínculo útil* para ofrecer contenido en un idioma distinto del que se usa en el contenido del anuncio de la oferta.

A continuación se muestra un ejemplo de cómo aparece la información en Azure Marketplace (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/example-azure-marketplace-app.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
2. Categorías
3. Dirección de soporte técnico (vínculo)
4. Términos de uso
5. Dirección de la directiva de privacidad (vínculo)
6. Nombre de la oferta
7. Resumen
8. Descripción
9. Capturas de pantallas o vídeos

<br>Este es un ejemplo de cómo aparece la información de la oferta en Azure Portal:

:::image type="content" source="media/example-virtual-machine-container-iot-edge-saas.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Título
2. Descripción
3. Vínculos útiles
4. Capturas de pantalla

#### <a name="name"></a>Nombre

El nombre que escriba aquí se mostrará a los clientes como el título de la descripción de la oferta. Este campo se ha rellenado previamente con el texto introducido para **Alias de la oferta** cuando creó la oferta, pero puede cambiar este valor. Este nombre puede contener marcas comerciales (y puede incluir símbolos de marca comercial o copyright). El nombre no puede tener más de 50 caracteres ni incluir emojis.

#### <a name="search-results-summary"></a>Resumen de los resultados de la búsqueda

Proporcione una breve descripción de la oferta (hasta 100 caracteres). Esta descripción se puede usar en los resultados de búsqueda.

#### <a name="long-summary"></a>Resumen largo

Proporcione una descripción más larga de la oferta, hasta 256 caracteres. Esta descripción se puede usar en los resultados de búsqueda.

#### <a name="description"></a>Descripción

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Rich text editor](./includes/rich-text-editor.md)]

#### <a name="search-keywords"></a>Palabras clave de búsqueda

Opcionalmente, puede escribir un máximo de tres palabras clave de búsqueda para ayudar a los clientes a buscar su oferta en Marketplace. Para obtener mejores resultados, intente usar también estas palabras clave en la descripción.

#### <a name="privacy-policy-link"></a>Vínculo de la directiva de privacidad

Escriba la dirección URL a la directiva de privacidad de su organización. Usted es el responsable no solo de garantizar que la aplicación cumple la normativa y la legislación en lo tocante a la privacidad, sino también de proporcionar una directiva de privacidad válida.

### <a name="useful-links"></a>Vínculos útiles

Puede agregar vínculos a documentos en línea complementarios si selecciona **+ Agregar un vínculo**.

### <a name="contact-information"></a>Información de contacto

Proporcione el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico**, un **contacto de ingeniería** o un **contacto del Programa de CSP**. Esta información no se muestra a los clientes, pero estará disponible para Microsoft y puede proporcionarse a los asociados de CSP. Algunos contactos pueden necesitar información adicional.

### <a name="marketplace-media"></a>Elementos multimedia del marketplace

Proporcione logotipos e imágenes para usarlos con la oferta. Todas las imágenes deben estar en formato PNG. Las imágenes borrosas harán que se rechace el envío.

[!INCLUDE [logotips](../includes/graphics-suggestions.md)]

>[!Note]
>Si tiene problemas al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear un logotipo **Pequeño** y un logotipo **Mediano**. Opcionalmente, puede reemplazarlos por imágenes diferentes más adelante.

- **Grande** (de 216 x 216 a 350 x 350 píxeles, obligatorio)
- **Mediano** (90 x 90 píxeles, opcional)
- **Pequeño** (48 x 48 píxeles, opcional)

Estos logotipos se usan en distintos lugares de la publicación:

[!INCLUDE [logos-azure-marketplace-only](../includes/logos-azure-marketplace-only.md)]

[!INCLUDE [Logo tips](../includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Capturas de pantalla

Agregue hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Cada captura de pantalla debe tener un tamaño de 1280 x 720 píxeles y el formato PNG. Cada captura de pantalla debe incluir un título.

#### <a name="videos"></a>Vídeos

Agregue hasta cinco vídeos que muestren la oferta. Se deben hospedar en un servicio de vídeo externo. Escriba el nombre de cada vídeo, su dirección web y una imagen en miniatura en formato PNG del vídeo de 1280 x 720 píxeles.

#### <a name="additional-marketplace-listing-resources"></a>Recursos adicionales de la descripción de Marketplace

- [Procedimientos recomendados la descripción de ofertas en Marketplace](../gtm-offer-listing-best-practices.md)

Seleccione **Guardar borrador** antes de continuar.

## <a name="preview-audience"></a>Público preliminar

En la pestaña Versión preliminar, elija un **público preliminar** limitado para validar la oferta antes de ponerla a disposición de un público más amplio del marketplace.

> [!IMPORTANT]
> Después de revisar la oferta en versión preliminar, seleccione **Publicar** para incluir la oferta como pública en el marketplace comercial.

Los GUID de identificador de suscripción de Azure, junto con una descripción adicional de cada uno, identifican el público preliminar. Los clientes no pueden ver ninguno de estos campos. Puede encontrar el identificador de suscripción de Azure en la página **Suscripciones** de Azure Portal.

Agregue al menos un id. de suscripción de Azure, ya sea de forma individual (hasta 10) o mediante la carga de un archivo CSV (hasta 100). Al agregar estos identificadores de suscripción, puede definir quién puede obtener una vista previa de la oferta antes de publicarla. Si la oferta ya está publicada, todavía puede definir un público preliminar para probar los cambios o las actualizaciones de la oferta.

> [!NOTE]
> Un público preliminar no es el mismo que un público privado. A una audiencia preliminar se le permite acceder a la oferta *antes* de que se publique en el marketplace. Podrá ver y validar todos los planes, incluidos aquellos que estarán disponibles solo para un público privado después de que su oferta se publique completamente en el marketplace. Un público privado (definido en la pestaña **Precios y disponibilidad** del plan) tiene acceso exclusivo a un plan determinado.

Seleccione **Guardar borrador** antes de continuar.

## <a name="technical-configuration"></a>Configuración técnica

Rellene esta sección solo si la oferta incluye una aplicación administrada que emitirá eventos de medición mediante la API del servicio de medición de Marketplace. Escriba los valores de **Id. de inquilino de Azure Active Directory** e **Identificador de la aplicación de Azure Active Directory** que usará el servicio al emitir eventos de medición.

Seleccione **Guardar borrador** antes de continuar.

## <a name="technical-configuration-offer-level"></a>Configuración técnica (nivel de oferta)

>[!Note]
>Los detalles técnicos de nivel de oferta son opcionales.  Solo tiene que configurar estos detalles si va a publicar una aplicación administrada con facturación de uso medido y tiene un servicio que se autenticará con un token de seguridad de Azure AD. Para más información, consulte las [estrategias de autenticación](./marketplace-metering-service-authentication.md) en las distintas opciones de autenticación.

La configuración técnica define los detalles (identificador de inquilino e identificador de aplicación) que se usan para identificar el servicio, que emitirá eventos de medición de una aplicación administrada mediante las [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md).  Escriba la identidad que usará el servicio al emitir eventos de medición.

* **Identificador de inquilino de Azure AD** (se requiere): En Azure Portal, debe [crear una aplicación de Azure Active Directory](../../active-directory/develop/howto-create-service-principal-portal.md) para que podamos validar que la conexión entre nuestros dos servicios está detrás de una comunicación autenticada. Para buscar el [identificador de inquilino](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in), vaya a Azure Active Directory, seleccione **Propiedades** y, después, busque el número de **Id. de directorio** que aparece (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e).
* **Identificador de aplicación de Azure AD** (se requiere): También necesita su [identificador de la aplicación](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) y una clave de autenticación. Para obtener esos valores, vaya a Azure Active Directory, seleccione **Registros de aplicaciones** y, después, busque el número de **Id. de la aplicación** que se muestra (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e). Para buscar la clave de autenticación, vaya a **Configuración** y seleccione **Claves**. Deberá proporcionar una descripción y una duración y, después, se proporcionara un valor numérico.

>[!Note]
>El identificador de la aplicación de Azure se asociará a su identificador de anunciante y solo se podrá volver a usar en la cuenta de este anunciante.

>[!Note]
>Esta configuración es necesaria si quiere usar [eventos de uso de Batch](marketplace-metering-service-apis.md#metered-billing-batch-usage-event).  En caso de que quiera enviar [eventos de uso](marketplace-metering-service-apis.md#metered-billing-single-usage-event), también puede usar el [servicio de metadatos de instancia](../../active-directory/managed-identities-azure-resources/overview.md) para obtener el [token de portador de JSON Web Token (JWT)](pc-saas-registration.md#how-to-get-the-publishers-authorization-token).

## <a name="plan-overview"></a>Información general del plan

Esta pestaña le permite ofrecer diversas opciones de planes dentro de la misma oferta. Estos planes (anteriormente llamados SKU) pueden diferir en términos de tipo de plan (plantilla de solución frente a aplicación administrada), monetización o público. Configure al menos un plan para mostrar su oferta en Marketplace.

Puede crear hasta 100 planes para cada oferta, de los cuales hasta 45 pueden ser privados. Más información sobre los planes privados en [Ofertas privadas en el marketplace comercial de Microsoft](../private-offers.md).

Una vez creados, verá los nombres de los planes, los identificadores, la disponibilidad (pública o privada), el estado actual de la publicación y las acciones disponibles en esta pestaña.

Las **acciones** disponibles en **Información general del plan** varían en función del estado actual del plan y pueden incluir:

* Si el estado del plan es **Borrador**: eliminar borrador.
* Si el estado del plan es **Activo**: detener venta del plan o sincronizar público privado.

### <a name="create-new-plan"></a>Creación de un nuevo plan

***Identificador de plan***: cree un identificador de plan único para cada plan de esta oferta. Este identificador será visible para los clientes en la dirección URL del producto.  Use solo caracteres alfanuméricos en minúscula, guiones o caracteres de subrayado. El identificador de este plan no puede tener más de 50 caracteres. Este identificador no se puede modificar después de seleccionar que se cree.

***Nombre del plan***: los clientes verán este nombre al decidir qué plan van a seleccionar en su oferta. Cree un nombre de oferta único para cada uno de los planes de esta oferta. El nombre del plan se usa para diferenciar planes de software que pueden formar parte de la misma oferta (por ejemplo, nombre de la oferta: Windows Server; planes: Windows Server 2016, Windows Server 2019).

### <a name="plan-setup"></a>Configuración del plan

Esta pestaña permite establecer la configuración de alto nivel del tipo de plan, aunque reutilice paquetes de otro plan, así como las nubes donde debe estar disponible el plan. Sus respuestas de esta pestaña afectarán a los campos mostrados en otras pestañas para el mismo plan.

#### <a name="plan-type"></a>Tipo de plan
Seleccione el tipo de plan de la oferta. El cliente administra completamente los planes de **plantillas de soluciones**. Un plan de **aplicación administrada** permite que los anunciantes administren la aplicación en nombre del cliente. Para más información, consulte [Tipos de planes de aplicación de Azure](#types-of-azure-application-plans).

#### <a name="re-use-technical-configuration"></a>Reutilización de la configuración técnica

Si tiene más de un plan del mismo tipo y los paquetes son idénticos entre ellos, puede seleccionar **this plan reuses packages from another plan** (este plan reutiliza paquetes de otro plan).  Al seleccionar esta opción, podrá seleccionar uno de los otros planes del mismo tipo del que esta oferta pueda reutilizar paquetes.

>[!Note]
>Cuando reutilice paquetes de otro plan, toda la pestaña Configuración técnica desaparecerá de este plan. Los detalles de configuración técnica del otro plan, incluidas las actualizaciones que realice en el futuro, se usarán también para este plan.<br><br>Esta configuración no se puede modificar una vez publicado este plan.

#### <a name="azure-regions-cloud"></a>Regiones de Azure (nube)

El plan debe estar disponible al menos en una región de Azure.

Seleccione la opción **Azure global** para que el plan esté disponible para los clientes de todas las regiones globales de Azure que tengan la integración de marketplace comercial. Para obtener más información, consulte [Disponibilidad geográfica y soporte técnico de moneda](../marketplace-geo-availability-currencies.md).

Seleccione la opción **Azure Government** para que el plan esté disponible en la región de [Azure Government](../../azure-government/documentation-government-welcome.md). Esta región proporciona acceso controlado para los clientes de entidades tribales, locales, estatales o federales de Estados Unidos, así como para los asociados aptos para abastecerlas. Como publicador, es responsable de los controles de cumplimiento, las medidas de seguridad y los procedimientos recomendados. Azure Government usa redes y centros de datos aislados físicamente (ubicados solo en Estados Unidos).

Antes de publicar en [Azure Government](../../azure-government/documentation-government-manage-marketplace-partners.md), pruebe y valide el plan en el entorno, ya que algunos puntos de conexión pueden ser diferentes. Para configurar y probar el plan, solicite una cuenta de prueba desde [Prueba de Microsoft Azure Government](https://azure.microsoft.com/global-infrastructure/government/request/).

>[!NOTE]
>Una vez que el plan esté publicado y disponible en una región de Azure específica, no se puede quitar esa región.

#### <a name="azure-government-certifications"></a>Certificaciones de Azure Government

Esta opción solo está visible si ha seleccionado **Azure Government**.

Los servicios de Azure Government controlan datos que están sujetos a determinados reglamentos y requisitos gubernamentales. Por ejemplo, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 y CJIS. Para dar a conocer sus certificaciones de estos programas, puede proporcionar hasta 100 vínculos que las describan. Pueden ser vínculos a su anuncio en el programa directamente o a descripciones de su cumplimiento de estos en sus propios sitios web. Estos vínculos solo son visibles para los clientes de Azure Government.

Seleccione **Guardar borrador** antes de continuar.

### <a name="plan-listing"></a>Lista del plan

Aquí es donde se configuran los detalles de la lista del plan. Esta pestaña muestra información específica que puede variar entre los planes de la misma oferta.

#### <a name="plan-name"></a>Nombre del plan

Se rellena previamente con el nombre que asignó al plan cuando lo creó. Este nombre aparece en el marketplace como el título de este plan y está limitado a 100 caracteres.

#### <a name="plan-summary"></a>Resumen del plan

Proporcione un breve resumen del plan (no de la oferta). Este resumen se limita a 100 caracteres.

#### <a name="plan-description"></a>Description del plan

Describa lo que hace que este plan sea único, así como las diferencias entre los planes de la oferta. No describa la oferta, solo el plan. La descripción del plan puede contener hasta 2 000 caracteres.

Seleccione **Guardar borrador** antes de continuar.

### <a name="availability-solution-template-plans-only"></a>Disponibilidad (solo planes de plantillas de soluciones)

Puede hacer que el plan sea visible para todo el mundo, solo para clientes específicos (un público privado) y ocultarlo para que únicamente lo usen otras plantillas de solución o aplicaciones administradas.

#### <a name="plan-visibility"></a>Visibilidad del plan

Tiene la opción de configurar cada plan para que lo pueda ver todo el mundo o solo un público concreto que elija. Para asignar la pertenencia a este público utilice los identificadores de suscripción de Azure.

Seleccione **This is a private plan** (Este es un plan privado) para que el plan sea privado y solo lo pueda ver el público que elija. Una vez que se haya publicado como un plan privado, es posible actualizar el público o elegir que el plan esté disponible para todos los usuarios. Una vez que se publica un plan como visible para todos los usuarios, debe permanecer así; no se puede volver a configurar como un plan privado.

Si convierte el plan en privado, escriba los **identificadores de suscripción de Azure** y su descripción. Cada identificador es un público que tendrá acceso a este plan privado. El acceso se asigna mediante identificadores de suscripción de Azure y existe la opción de incluir una descripción de cada identificador de suscripción de Azure asignado. Puede agregar hasta 10 identificadores de suscripción de cliente de forma individual o 20 000 si los importa en un archivo CSV. Los identificadores de suscripción de Azure se representan como GUID y las letras deben estar en minúsculas.

>[!Note]
>Una audiencia privada o restringida es diferente del público preliminar que definió en la pestaña **Versión preliminar**. Una audiencia preliminar puede acceder a la oferta _antes_ de que se publique en el marketplace. En tanto que la designación de una audiencia privada solo se aplica a un plan específico, el público preliminar puede ver todos los planes (privados o no) con fines de validación.

#### <a name="hide-plan"></a>Ocultar plan

Si su plantilla de solución está diseñada para implementarse solo indirectamente cuando se hace referencia a ella a través de otra plantilla de solución o aplicación administrada, active esta casilla para publicar su plantilla de solución, pero ocúltela para los clientes que la buscan directamente.

Seleccione **Guardar borrador** antes de continuar.

### <a name="pricing-and-availability-managed-application-plans-only"></a>Precios y disponibilidad (solo planes de aplicaciones administradas)

Configure los **mercados** en los que está disponible este plan, el **precio** al mes de la administración de la solución y la **visibilidad del plan** si solo deben verlo clientes específicos (un público privado).

Seleccione **Guardar borrador** antes de continuar.

#### <a name="markets"></a>Mercados

Todos los planes debe estar disponible al menos en un mercado. Seleccione la casilla de la ubicación del mercado en la que desea que este plan esté disponible. Se incluyen un cuadro de búsqueda y un botón para seleccionar los países o regiones con "Impuesto remitido", en los que Microsoft remite de los impuestos de uso y ventas en su nombre, que sirven de ayuda.

Si ya se ha establecido los precios del plan en dólares de Estados Unidos (USD) y agrega otra ubicación del mercado, el precio del nuevo mercado se calculará en función de los tipos de cambio actuales. Revise siempre el precio de cada mercado antes de la publicación. Los precios se pueden consultar mediante el vínculo "Exportar precios (xlsx)" después de guardar los cambios.

#### <a name="pricing"></a>Precios

Proporcione el precio mensual de este plan.  El precio se suma a cualquier infraestructura de Azure o costos de software de pago por uso en los que incurren los recursos implementados por esta solución.

Además del precio por mes, también puede establecer precios por el consumo de unidades no estándar mediante la [facturación de uso medido](./azure-app-metered-billing.md).  Si lo desea, puede establecer el precio por mes en cero y cobrar exclusivamente con la facturación de uso medido.

Los precios establecidos en USD (USD = dólares de Estados Unidos) se convierten en la moneda local de todos los mercados seleccionados con los tipos de cambio actuales en el momento de guardarlos. Valide estos precios antes de realizar la publicación. Para ello, debe exportar la hoja de cálculo de precios y examinar el precio de cada mercado. Si desea establecer precios personalizados en un mercado individual, modifique e importe la hoja de cálculo de precios.

>[!Note]
>Primero debe guardar los cambios de precios para habilitar la exportación de los datos de precios.

Revise los precios cuidadosamente antes de la publicación, ya que hay algunas restricciones en lo que se puede cambiar una vez publicado un plan.  

>[!Note]
>Una vez que se publica un precio para un mercado de su plan, no puede cambiarse más adelante.

#### <a name="plan-visibility"></a>Visibilidad del plan

Tiene la opción de configurar cada plan para que lo pueda ver todo el mundo o solo un público concreto que elija. Para asignar la pertenencia a este público utilice los identificadores de suscripción de Azure.

Seleccione **This is a private plan** (Este es un plan privado) para que el plan sea privado y solo lo pueda ver el público que elija. Una vez que se haya publicado como un plan privado, es posible actualizar el público o elegir que el plan esté disponible para todos los usuarios. Una vez que se publica un plan como visible para todos los usuarios, debe permanecer así; no se puede volver a configurar como un plan privado.

>[!Note]
>Una audiencia privada o restringida es diferente del público preliminar que definió en la pestaña **Versión preliminar**. Una audiencia preliminar puede acceder a la oferta _antes_ de que se publique en el marketplace. En tanto que la designación de una audiencia privada solo se aplica a un plan específico, el público preliminar puede ver todos los planes (privados o no) con fines de validación.

Si convierte el plan en privado, escriba los **identificadores de suscripción de Azure** y su descripción. Cada identificador es un público que tendrá acceso a este plan privado. El acceso se asigna mediante identificadores de suscripción de Azure y existe la opción de incluir una descripción de cada identificador de suscripción de Azure asignado. Puede agregar hasta 10 identificadores de suscripción de cliente de forma individual o 20 000 si los importa en un archivo CSV. Los identificadores de suscripción de Azure se representan como GUID y las letras deben estar en minúsculas.

>[!Note]
>Las ofertas privadas no son compatibles con las suscripciones de Azure que se establecen a través de un revendedor del programa Proveedor de soluciones en la nube (CSP).

### <a name="technical-configuration"></a>Configuración técnica

Esta pestaña le permite cargar el paquete de implementación que posibilitará que los clientes implementen el plan.

>[!Note]
>No se verá esta pestaña si configuró este plan para reutilizar paquetes de otro plan en la pestaña **Configuración del plan**.

#### <a name="package-details"></a>Detalles del paquete

Esta pestaña le permite editar la versión de borrador de la configuración técnica.

**Versión**: asigne la versión actual de la configuración técnica.  Incremente esta versión cada vez que publique un cambio en esta página. La versión debe tener el formato `{integer}.{integer}.{integer}`.

**Archivo de paquete** (.zip): este paquete contiene todos los archivos de plantilla necesarios para este plan, así como recursos adicionales, empaquetados como un archivo `.zip`.

Todos los paquetes del plan de la aplicación de Azure deben incluir estos dos archivos en la carpeta raíz de un archivo `.zip`:

* Un archivo de plantilla de Resource Manager llamado [mainTemplate.json](../../azure-resource-manager/resource-group-overview.md). Esta plantilla automatiza la implementación de recursos en la suscripción de Azure de clientes.  Para ver ejemplos de plantillas de Resource Manager, consulte la [galería de plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/) o el repositorio [GitHub: Plantillas de inicio rápido de Azure Resource Manager](https://github.com/azure/azure-quickstart-templates) correspondiente.
* Una definición de interfaz de usuario para la experiencia de creación de aplicaciones de Azure llamada [createUiDefinition.json](../../azure-resource-manager/managed-application-createuidefinition-overview.md).

Los tamaños de archivo máximos admitidos son:

* Hasta 1 GB en el tamaño del archivo `.zip` comprimido total.
* Hasta 1 GB para cada archivo individual descomprimido dentro del archivo `.zip`.  

Todas las nuevas ofertas de la aplicación de Azure deben incluir un GUID de [atribución de uso del cliente para asociados de Azure](../azure-partner-customer-usage-attribution.md).

>[!Note]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

Los planes de aplicaciones administradas requieren información adicional en esta pestaña.

#### <a name="previously-published-packages"></a>Paquetes publicados anteriormente

La subpestaña **Paquetes publicados anteriormente** permite ver todas las versiones publicadas de la configuración técnica.

#### <a name="enable-just-in-time-jit-access"></a>Habilitar acceso Just-In-Time (JIT)

Seleccione esta opción para habilitar acceso Just-In-Time (JIT) para este plan.  El acceso JIT permite solicitar acceso con privilegios elevados a recursos de la aplicación administrada con fines de solución de problemas o mantenimiento. Siempre tendrá acceso de solo lectura a los recursos, pero puede tener mayor acceso durante un período de tiempo específico.  Para obtener más información, consulte [Habilitación y solicitud de acceso Just-In-Time para Azure Managed Applications](../../managed-applications/request-just-in-time-access.md).  Para exigir que los consumidores de la aplicación administrada concedan el acceso permanente a la cuenta, deje esta opción desactivada.

>[!Note]
>Asegúrese de actualizar su archivo `createUiDefinition.json` para admitir esta característica.  

#### <a name="deployment-mode"></a>Modo de implementación

Seleccione si desea configurar el **modo de implementación incremental** o **completo** al implementar este plan: 

* En el **modo completo**, una reimplementación de la aplicación del cliente provocará la eliminación de recursos en el grupo de recursos administrados si los recursos no se definen en `mainTemplate.json`. 
* En el **modo incremental**, una reimplementación de la aplicación deja los recursos existentes sin cambios.

Para obtener más información sobre los modos de implementación, consulte [Modos de implementación de Azure Resource Manager](../../azure-resource-manager/deployment-modes.md).

#### <a name="notification-endpoint-url"></a>Dirección URL del punto de conexión de notificaciones

Especifique un punto de conexión de webhook de HTTPS para recibir notificaciones sobre todas las operaciones CRUD que están en las instancias de aplicación administradas de esta versión del plan.

#### <a name="customize-allowed-customer-actions"></a>Personalizar acciones de cliente permitidas

Seleccione esta opción para especificar qué acciones pueden realizar los clientes en los recursos administrados, además de las acciones "`*/read`" que están disponibles de forma predeterminada.

Enumere las acciones adicionales que desea permitir a su cliente realizar aquí, separadas por signos de punto y coma.  Para más información, consulte [Descripción de las asignaciones de denegación para recursos de Azure](../../role-based-access-control/deny-assignments.md). Puede consultar las acciones disponibles en [Operaciones del proveedor de recursos de Azure Resource Manager](../../role-based-access-control/resource-provider-operations.md). Por ejemplo, para permitir que los consumidores reinicien las máquinas virtuales, agregue `Microsoft.Compute/virtualMachines/restart/action` a las acciones permitidas.

#### <a name="global-azure--azure-government-cloud"></a>Azure global o nube de Azure Government

Indique quién debe tener acceso de administración a esta aplicación administrada en cada nube compatible. Los usuarios, grupos o aplicaciones a los que quiere conceder permiso para el grupo de recursos administrados se identifican mediante identidades de Azure Active Directory (AAD).

**Id. de inquilino de Azure Active Directory**: el identificador de inquilino de Azure AD (también conocido como identificador de directorio) que contiene las identidades de los usuarios, grupos o aplicaciones a los que quiere conceder permisos. Encontrará su identificador de inquilino de Azure AD en la página [Propiedades de Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties), en Azure Portal.

**Autorizaciones**: agregue el identificador de objeto de Azure Active Directory del usuario, grupo o aplicación a los que quiere conceder permiso para el grupo de recursos administrados. Identifique al usuario por su identificador de entidad de seguridad, que encontrará en la [hoja Usuarios de Azure Active Directory, en Azure Portal](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers).

Para cada entidad de seguridad, seleccione uno de los roles integrados de Azure AD de la lista (Propietario o Colaborador). El rol que seleccione describirá los permisos que tendrá la entidad de seguridad en los recursos de la suscripción del cliente. Para más información, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md). Para obtener más información sobre el control de acceso basado en rol (RBAC), consulte [Introducción al control de acceso basado en rol en Azure Portal](../../role-based-access-control/overview.md).

>[!Note]
>Aunque puede agregar hasta 100 autorizaciones por nube, suele ser más fácil crear un grupo de usuarios de Active Directory y especificar su id. en "ID de la entidad de seguridad". Esta opción le permitirá agregar más usuarios al grupo de administración una vez implementado el plan y reducir la necesidad de actualizar el plan solo para agregar más autorizaciones.

#### <a name="policy-settings"></a>Configuración de directiva

Aplique [directivas de Azure](../../governance/policy/overview.md) a la aplicación administrada para especificar los requisitos de cumplimiento para la solución implementada. Para las definiciones de directiva y el formato de los valores de parámetro, consulte [ejemplos de Azure Policy](../../governance/policy/samples/index.md). Puede configurar un máximo de cinco directivas y solo una instancia de cada opción Directivas. Algunas directivas requieren parámetros adicionales. La SKU Estándar es obligatoria para las directivas de auditoría. El nombre de la directiva está limitado a 50 caracteres.

Seleccione **Guardar borrador** antes de continuar.

## <a name="co-sell-with-microsoft"></a>Venta conjunta con Microsoft

Proporcionar información en la pestaña Venta conjunta es totalmente opcional para publicar su oferta. Es necesario para lograr los estados Co-sell Ready e IP Co-sell Ready. Los equipos de ventas de Microsoft usarán la información que proporciona para obtener más información sobre su solución al evaluar su idoneidad para las necesidades de los clientes. No está disponible directamente para los clientes.

Para más información sobre esta pestaña, consulte [Opción de venta conjunta en el Centro de partners](commercial-marketplace-co-sell.md).

## <a name="resell-through-csps"></a>Revender mediante los CSP

Amplíe el alcance de la oferta haciendo que esté disponible para los asociados en el programa [Proveedores de soluciones en la nube](https://azure.microsoft.com/offers/ms-azr-0145p/) (CSP). De esta forma, los distribuidores pueden vender su oferta a sus clientes y crear soluciones agrupadas.

Seleccione **Guardar borrador** antes de continuar.

## <a name="test-drive"></a>Versión de prueba

Configure una demostración (versión de prueba) que permita a los clientes probar la oferta antes de adquirirla. Para crear un entorno de demostración que permita a los clientes probar la oferta durante un período de tiempo determinado, consulte [Permitir a los clientes probar una versión de la oferta](test-drive.md).

Para habilitar una versión de prueba, seleccione la casilla **Habilitación de una versión de prueba** en la pestaña [Offer setup](#test-drive) (Configuración de la oferta). Para eliminar la versión de prueba de la oferta, desactive esta casilla.

### <a name="test-drive-technical-configuration"></a>Configuración técnica de la versión de prueba

- **Identificador de aplicación de Azure AD** (se requiere): Escriba el [identificador de la aplicación](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de Azure AD. Para encontrarlo, inicie sesión en [Azure Portal](https://portal.azure.com/), seleccione la pestaña Active Directory en el menú de la izquierda, seleccione **Registros de aplicaciones** y busque el número de **Id. de aplicación** que aparece (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e).

#### <a name="deployment-subscription-details"></a>Detalles de suscripción de implementación

Para implementar la versión de prueba en su nombre, cree y proporcione una suscripción de Azure independiente y única (no se requiere para las versiones de prueba de Power BI).

* **Id. de suscripción de Azure** (obligatorio para Azure Resource Manager y aplicaciones lógicas): escriba el identificador de la suscripción para conceder acceso a los servicios de la cuenta de Azure para la generación de informes y la facturación del uso de recursos. Se recomienda considerar la posibilidad de [crear una suscripción de Azure independiente](../../billing/billing-create-subscription.md) que se use para las versiones de prueba, en caso de que no se tenga ninguna. Para buscar su identificador de la suscripción de Azure inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya a la pestaña **Suscripciones** del menú izquierdo. Al seleccionar la pestaña se mostrará el identificador de la suscripción (como, por ejemplo, "a83645ac-1234-5ab6-6789-1h234g764ghty").
* **Id. de inquilino de Azure AD** (obligatorio): escriba el [identificador de inquilino](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de Azure Active Directory (AD). Para encontrarlo, inicie sesión en [Azure Portal](https://portal.azure.com/), seleccione la pestaña Active Directory en el menú de la izquierda, seleccione **Propiedades** y busque el número de **Id. de directorio** que aparece (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e). El id. de inquilino de su organización también lo puede buscar mediante la dirección URL de su nombre de dominio en: [https://www.whatismytenantid.com](https://www.whatismytenantid.com).
* **Nombre del inquilino de Azure AD** (obligatorio para Dynamic 365): escriba su nombre en Azure Active Directory (AD). Para buscar este nombre, inicie sesión en [Azure Portal](https://portal.azure.com/) y en la esquina superior derecha se mostrará su nombre del inquilino, debajo de su nombre de cuenta.
* **Identificador de aplicación de Azure AD** (obligatorio): escriba el [identificador de la aplicación](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de Azure Active Directory (AD). Para encontrarlo, inicie sesión en [Azure Portal](https://portal.azure.com/), seleccione la pestaña Active Directory en el menú de la izquierda, seleccione **Registros de aplicaciones** y busque el número de **Id. de aplicación** que aparece (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e).
* **Secreto de cliente de la aplicación de Azure Active Directory** (obligatorio): escriba el [secreto de cliente](../../active-directory/develop/howto-create-service-principal-portal.md#create-a-new-application-secret) de la aplicación de Azure AD. Para encontrar este valor, inicie sesión en [Azure Portal](https://portal.azure.com/). Seleccione la pestaña **Azure Active Directory** en el menú izquierdo, seleccione **Registros de aplicaciones** y, después, seleccione la versión de prueba de su aplicación. A continuación, seleccione **Certificados y secretos**, seleccione **Nuevo secreto de cliente**, escriba una descripción, seleccione **Nunca** en **Expira** y, después, elija **Agregar**. Asegúrese de copiar el valor antes de abandonar esta página.

Seleccione **Guardar borrador** antes de continuar.

### <a name="marketplace-listing-optional"></a>Lista de Marketplace (opcional)

Describa la experiencia de la versión de prueba.

<!-- The **Test Drive listings** option found under the **Test drive** tab displays the languages (and markets) where your test drive is available (currently English (United States) is the only location available). Additionally, this page displays the status of the language-specific listing and the date/time that it was added. You will need to define the test drive details (description, user manual, videos, etc.) for each language/market. -->

* **Descripción** (obligatoria): describa la versión de prueba, lo que se demostrará, los objetivos con los que experimentará el usuario, las características que se van a explorar y cualquier información de interés que ayude al usuario a determinar si debe adquirir la oferta. En este campo se puede escribir hasta 3000 caracteres de texto. 
* **Información de acceso** (obligatorio para las versiones de prueba de Azure Resource Manager y aplicaciones lógicas): explique lo que un cliente necesita saber para acceder a esta versión de prueba y usarla. se muestra un escenario para usar la oferta y exactamente lo que el cliente debe saber para acceder a las características en la versión de prueba. En este campo se puede escribir hasta 10 000 caracteres de texto.
* **Manual del usuario** (obligatorio): tutorial detallado de la experiencia con la versión de prueba. Debe abarcar exactamente lo que desee que el cliente aprenda al usar la versión de prueba y puede servir como referencia para las preguntas que puedan surgirle. El archivo debe estar en formato PDF y hay que asignarle un nombre (255 caracteres como máximo) después de cargarlo.
* **Vídeos: Agregar vídeos** (opcional): se pueden cargar vídeos en YouTube o Vimeo y hacer referencia a ellos aquí con un vínculo o una imagen en miniatura (533 × 324 píxeles) para que los clientes puedan ver toda la información que pueda ayudarles a conocer mejor la versión de prueba, incluida la forma de usar correctamente las características de la oferta y comprender los escenarios que destacan sus ventajas.
  * **Nombre** (se requiere)
  * **Dirección** (solo YouTube o Vimeo; obligatorio).
  * **Miniatura** (la imagen debe estar en formato PNG y tener un tamaño de 533 x 324 píxeles).

Seleccione **Guardar borrador** antes de continuar.

## <a name="publish"></a>Publicar

Cuando haya completado todas las secciones necesarias de la oferta, seleccione **Revisar y publicar** en la esquina superior derecha del portal.

Revise el estado de finalización de cada sección de la oferta.
    - *No iniciada*: significa que la sección no se ha modificado y se debe completar.
    - *Incompleta*: significa que la sección tiene errores que deben corregirse o que requiere más información. Vuelva a las secciones y actualícelas.
    - *Completa*: significa que la sección está completa, que se ha proporcionado todos los datos necesarios y que no hay ningún error. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.

Si esta es la primera vez que publica esta oferta, puede proporcionar instrucciones de prueba al equipo de certificación para asegurarse de que la aplicación se prueba correctamente, además de todas las notas adicionales que pueden resultar útiles para conocer la aplicación.

Seleccione **Enviar** para enviar la oferta y que se publique. Le enviaremos un correo electrónico que le avise cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación.

Vuelva al Centro de partners y seleccionar **Transmitir** la oferta para publicar la oferta para el público (o si es una oferta privada, al público privado).

### <a name="errors-and-review-feedback"></a>Errores y comentarios de revisión

El paso de **validación manual** del proceso de publicación representa una revisión extensiva de la oferta y sus recursos técnicos asociados (especialmente la plantilla de Azure Resource Manager), los problemas normalmente aparecen como vínculos de solicitud de incorporación de cambios. Para obtener una explicación sobre cómo ver y responder a estas solicitudes de incorporación de cambios, consulte [Administración de la revisión de comentarios](./azure-apps-review-feedback.md).

Si tiene errores en uno o varios de los pasos de publicación, corríjalos antes de volver a publicar la oferta.

## <a name="next-steps"></a>Pasos siguientes

* [Actualización de una oferta existente en Marketplace comercial](./update-existing-offer.md)
