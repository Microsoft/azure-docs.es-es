---
title: Creación de una oferta de Dynamics 365 for Operations en el portal de Marketplace comercial
description: Cómo crear una nueva oferta de Dynamics 365 for Operations para mostrarla o venderla en Azure Marketplace, AppSource, o bien a través del programa Proveedor de soluciones en la nube (CSP) mediante el portal de Marketplace comercial del Centro de partners de Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: navits09
ms.author: navits
ms.date: 06/17/2020
ms.openlocfilehash: 553fd4e49d274fcf4c25356764182d54b91149e4
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88036647"
---
# <a name="create-a-dynamics-365-for-operations-offer"></a>Creación de una oferta de Dynamics 365 for Operations

En este tema se explica cómo crear una nueva oferta de Dynamics 365 for Operations. [Microsoft Dynamics 365 for Finance and Operations](https://dynamics.microsoft.com/finance-and-operations) es un servicio de planeación de recursos empresariales (ERP) que admite finanzas avanzadas, operaciones, fabricación y administración de la cadena de suministro. Todas las ofertas de Dynamics 365 for Operations deben pasar por nuestro proceso de certificación.

Antes de comenzar, [cree una cuenta de Marketplace comercial en el Centro de partners](create-account.md), si aún no lo ha hecho. Asegúrese de que su cuenta está inscrita en el programa marketplace comercial.

>[!NOTE]
> Una vez publicada una oferta, las modificaciones de esta realizadas en el Centro de partners solo se actualizarán en el sistema y en los escaparates después de volver a publicarla. Asegúrese de enviar la oferta para su publicación después de realizar los cambios.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación, en la izquierda, seleccione **Marketplace comercial** > **Información general**.
3. En la página de Información general, seleccione **+ Nueva oferta** > **Dynamics 365 for Operations**.

    ![Muestra el menú de navegación izquierdo.](./media/new-offer-dynamics-365-operations.png)

> [!NOTE]
> Después de publicar una oferta, las modificaciones que se realicen en el Centro de partners solo aparecen en los escaparates cuando la oferta se vuelve a publicar. Asegúrese de volver a realizar la publicación siempre después de realizar cambios.

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
- Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-setup"></a>Configuración de la oferta

### <a name="how-do-you-want-potential-customers-to-interact-with-this-listing-offer"></a>¿Cómo quiere que los clientes potenciales interactúen con esta oferta en la lista?

Seleccione la opción que quiere usar para esta oferta.

#### <a name="get-it-now-free"></a>Obténgalo ahora (de forma gratuita)

Presente su oferta a los clientes de forma gratuita proporcionándoles una dirección URL válida (que comienza por *http* o *https*) desde donde pueden acceder a su aplicación.  Por ejemplo: `https://contoso.com/my-app`

#### <a name="free-trial-listing"></a>Evaluación gratuita (anuncio)

Para presentar la oferta a los clientes con un vínculo a una evaluación gratuita, proporcióneles una dirección URL válida (que empiece por `http` o `https`) desde la que puedan obtenerla. Por ejemplo, `https://contoso.com/trial/my-app`. Las ofertas que incluyen evaluaciones gratuitas se crean, administran y configuran desde su servicio y no tienen suscripciones administradas por Microsoft.

> [!NOTE]
> Los tokens que la aplicación recibirá a través del vínculo de evaluación gratuita solo se pueden usar para obtener información de usuario mediante Azure Active Directory (Azure AD) para automatizar la creación de cuentas en la aplicación. No se admiten las cuentas de Microsoft para la autenticación con este token.

#### <a name="contact-me"></a>Ponerse en contacto conmigo

Recopile información de contacto de los clientes mediante la conexión de su sistema de Administración de relaciones con los clientes (CRM). El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre e identificador de la oferta y el origen de Marketplace donde la ha encontrado, se envían al sistema CRM que se ha configurado. Para más información sobre cómo configurar el sistema CRM, consulte [Clientes potenciales](#customer-leads).

### <a name="test-drive"></a>Versión de prueba

Una versión de prueba es una excelente manera de presentar la oferta a clientes potenciales, ya que se les brinda la oportunidad de «probar antes de comprar», lo que aumenta la conversión y la generación de clientes potenciales altamente cualificados. Para obtener más información, comience con [Qué es la versión de prueba](../what-is-test-drive.md).

Para habilitar una versión de prueba durante un período de tiempo determinado, seleccione la casilla **Habilitar una versión de prueba**. Para eliminar la versión de prueba de la oferta, desactive esta casilla.

### <a name="customer-leads"></a>Clientes potenciales

[!INCLUDE [Connect lead management](./includes/connect-lead-management.md)]

Para obtener más información, consulte [Introducción a la administración de clientes potenciales](./commercial-marketplace-get-customer-leads.md).

Seleccione **Guardar borrador** antes de continuar.

## <a name="properties"></a>Propiedades

Esta página le permite definir las categorías y los sectores que se usan para agrupar su oferta en el marketplace, la versión de la aplicación y los contratos legales que dan soporte a su oferta.

### <a name="category"></a>Category

Seleccione categorías y subcategorías para colocar la oferta en las áreas de búsqueda adecuadas del marketplace. En la descripción de la oferta, asegúrese de indicar la forma en que la oferta admite estas categorías. Seleccione:

- Al menos una y un máximo de dos categorías, incluidas una categoría principal y una secundaria (opcional).
- Hasta dos subcategorías para cada categoría principal o secundaria. Si no hay ninguna subcategoría aplicable a la oferta, seleccione **No aplicable**.

Vea la lista completa de categorías y subcategorías en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md).

### <a name="industry"></a>Sector

[!INCLUDE [Industry Taxonomy](./includes/industry-taxonomy.md)]

### <a name="app-version"></a>Versión de la aplicación

Escriba el número de versión de la oferta. Los clientes verán esta versión en la página de detalles de la oferta.

### <a name="terms-and-conditions"></a>Términos y condiciones

Proporcione sus propios términos y condiciones legales en el campo **Términos y condiciones**. También puede proporcionar la dirección URL donde se pueden encontrar los términos y condiciones. Los clientes deberán aceptar estos términos para poder probar la oferta.

Seleccione **Guardar borrador** antes de continuar.

## <a name="offer-listing"></a>Descripción de la oferta

La página muestra los idiomas en los que se mostrará la oferta. La única opción disponible actualmente es **Inglés (Estados Unidos)** .

Tendrá que definir los detalles de Marketplace (nombre de la oferta, descripción, imágenes, etc.) para cada idioma y mercado. Seleccione el nombre de idioma o mercado para proporcionar esta información.

> [!NOTE]
> No es necesario que el contenido del anuncio de la oferta (por ejemplo, la descripción, los documentos, las capturas de pantallas, los términos de uso, etc.) esté en inglés siempre que la descripción de la oferta empiece por la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es aceptable proporcionar una *dirección URL de vínculo útil* para ofrecer contenido en un idioma distinto del que se usa en el contenido del anuncio de la oferta.

Este es un ejemplo de cómo aparece la información de la oferta en Microsoft AppSource:

:::image type="content" source="media/example-azure-marketplace-d365-operations.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
2. Productos
3. Categorías
4. Industrias
5. Dirección de soporte técnico (vínculo)
6. Términos de uso
7. Directiva de privacidad
8. Nombre de la oferta
9. Capturas de pantallas o vídeos
10. Descripción

### <a name="name"></a>Nombre

El nombre que escriba aquí se mostrará a los clientes como el título de la descripción de la oferta. Este campo se ha rellenado previamente con el texto introducido para **Alias de la oferta** cuando creó la oferta, pero puede cambiar este valor. Este nombre puede contener marcas comerciales (y puede incluir símbolos de marca comercial o copyright). El nombre no puede tener más de 50 caracteres ni incluir emojis.

### <a name="short-description"></a>Descripción breve

Proporcione una breve descripción de la oferta (hasta 100 caracteres). Esta descripción se puede usar en los resultados de la búsqueda de Marketplace.

### <a name="description"></a>Descripción

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Rich text editor](./includes/rich-text-editor.md)]

### <a name="search-keywords"></a>Palabras clave de búsqueda

Opcionalmente, puede escribir un máximo de tres palabras clave de búsqueda para ayudar a los clientes a buscar su oferta en Marketplace. Para obtener mejores resultados, intente usar también estas palabras clave en la descripción.

### <a name="products-your-app-works-with"></a>Productos con los que funciona su aplicación

Si quiere permitir que los clientes sepan que su aplicación funciona con productos específicos, escriba hasta tres nombres de productos aquí.

### <a name="support-urls"></a>Direcciones URL de soporte técnico

Esta sección le permite proporcionar vínculos para ayudar a los clientes a comprender mejor su oferta.

#### <a name="help-link"></a>Vínculo de ayuda

Escriba la dirección URL en la que los clientes pueden obtener más información sobre su oferta.

#### <a name="privacy-policy-url"></a>Dirección URL de la directiva de privacidad

Escriba la dirección URL a la directiva de privacidad de su organización. Usted es el responsable no solo de garantizar que la aplicación cumple la normativa y la legislación en lo tocante a la privacidad, sino también de proporcionar una directiva de privacidad válida.

### <a name="contacts"></a>Contactos

En esta sección, va a proporcionar el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes, pero estará disponible para Microsoft y puede proporcionarse a los asociados de CSP.

En la sección **Contacto de soporte técnico**, proporcione la **dirección URL de soporte técnico** en la que los asociados de CSP pueden encontrar soporte técnico para la oferta.

### <a name="supporting-documents"></a>Documentos relacionados

Proporcione al menos un documento de marketing relacionado (y un máximo de tres) aquí, como notas del producto, folletos, listas de comprobación o presentaciones. Estos documentos deben estar en formato PDF.

### <a name="marketplace-images"></a>Imágenes de Marketplace

En esta sección, puede proporcionar logotipos e imágenes que se usarán al mostrar la oferta al cliente. Todas las imágenes deben estar en formato .PNG.

[!INCLUDE [logotips](../includes/graphics-suggestions.md)]

>[!Note]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione el logotipo de la oferta en dos tamaños de píxel:

- **Pequeño (48 x 48)**
- **Grande (216 x 216)**

[!INCLUDE [logos-appsource-only](../includes/logos-appsource-only.md)]

#### <a name="screenshots"></a>Capturas de pantalla

Agregue capturas de pantallas que muestren el funcionamiento de la oferta. Se requiere al menos una captura de pantalla y puede agregar un máximo de cinco. Todas las capturas de pantallas deben tener 1280 x 720 píxeles.

#### <a name="videos"></a>Vídeos

Opcionalmente, puede agregar hasta cuatro vídeos que muestren la oferta. Estos vídeos se deben hospedar en YouTube o Vimeo. En cada uno de ellos, escriba el nombre del vídeo, su dirección URL y una imagen en miniatura del vídeo (1280 x 720 píxeles).

#### <a name="additional-marketplace-listing-resources"></a>Recursos adicionales de la descripción de Marketplace

[Procedimientos recomendados la descripción de ofertas en Marketplace](../gtm-offer-listing-best-practices.md)

Seleccione **Guardar borrador** antes de continuar.

## <a name="availability"></a>Disponibilidad

Esta página proporciona opciones sobre dónde y cómo hacer que la oferta esté disponible.

### <a name="markets"></a>Mercados

Esta sección le permite especificar los mercados en los que la oferta debe estar disponible. Para ello, seleccione **Editar mercados**, que mostrará la ventana emergente **Selección de mercado**.

De forma predeterminada, no se selecciona ningún mercado. Seleccione al menos un mercado para publicar la oferta. Haga clic en **Seleccionar todo** para que su oferta esté disponible en cada mercado posible o seleccione los mercados específicos que quiera agregar. Al acabar, seleccione **Guardar**.

Las selecciones aquí solo se aplican a las nuevas adquisiciones. Si algún usuario ya tiene su aplicación en un determinado mercado y posteriormente lo quita, las personas que ya tienen la oferta en dicho mercado podrán seguir utilizándola, pero ningún cliente nuevo en ese mercado podrá obtener su oferta.

> [!IMPORTANT]
> Es su responsabilidad cumplir los requisitos legales locales, incluso si esos requisitos no aparecen aquí ni en el Centro de partners.

Tenga en cuenta que, incluso si selecciona todos los mercados, las leyes y restricciones locales u otros factores pueden impedir que se muestren determinadas ofertas en algunos países y regiones.

### <a name="preview-audience"></a>Público preliminar

Antes de publicar su oferta en la oferta de Marketplace más amplia, primero deberá ponerla a disposición de un **público preliminar** limitado. Escriba una **clave de ocultación** (cualquier cadena que use solo letras minúsculas o números) aquí. Los miembros del público preliminar pueden usar esta clave de ocultación como token para ver una versión preliminar de la oferta en Marketplace.

Después, cuando esté listo para que la oferta esté disponible y quite la restricción de versión preliminar, deberá quitar la **tecla de ocultación** y volver a publicarla.

Seleccione **Guardar borrador** antes de continuar.

## <a name="technical-configuration"></a>Configuración técnica

Esta página especifica los detalles técnicos usados para conectarse a la oferta. Esta conexión nos permite aprovisionar su oferta para el cliente final, en caso de que este elija adquirirla.

### <a name="solution-identifier"></a>Identificador de la solución

Proporcione el identificador de la solución (GUID) para la solución.

Para encontrar el identificador de la solución:

1. En Microsoft Dynamics Lifecycle Services (LCS), seleccione **Administración de soluciones**.
2. Seleccione la solución y, a continuación, busque el **identificador de la solución** en la **información general del paquete**. Si el identificador está en blanco, seleccione **Editar**, vuelva a publicar el paquete e inténtelo de nuevo.

### <a name="release-version"></a>Versión de lanzamiento

Seleccione la versión de Dynamics 365 for Finance and Operations con la que funciona esta solución.

Seleccione **Guardar borrador** antes de continuar.

## <a name="test-drive-technical-configuration"></a>Configuración técnica de la versión de prueba

Esta página permite configurar una demostración («versión de prueba») que permite a los clientes probar la oferta antes de adquirirla. Más información en [Qué es la versión de prueba](../what-is-test-drive.md).

Para habilitar una versión de prueba, seleccione la pestaña**Habilitar una versión de prueba** y marque la casilla[Configuración de la oferta](#test-drive). Para eliminar la versión de prueba de la oferta, desactive esta casilla.

Cuando haya terminado de configurar la versión de prueba, seleccione **Guardar borrador** antes de continuar.

## <a name="supplemental-content"></a>Contenido complementario

Esta página le permite proporcionar información adicional sobre su oferta, lo que nos ayudará a validarla. Esta información no se muestra a los clientes ni se publica en Marketplace.

### <a name="validation-assets"></a>Recursos de validación

Debe cargar un [informe de análisis de personalización (CAR)](https://docs.microsoft.com/dynamics365/unified-operations/dev-itpro/dev-tools/customization-analysis-report) en esta sección. Este informe se genera mediante el análisis de los modelos de personalización y extensión, en función de un conjunto predefinido de reglas de procedimientos recomendados.

Este archivo debe estar en formato .xls o .xlsx. Si tiene más de un informe, puede cargar un archivo ZIP que contenga todos los informes.

### <a name="does-solution-include-localizations"></a>¿La solución incluye localizaciones?

Seleccione **Sí** si la solución permite el uso de estándares y directivas locales (por ejemplo, si se ajusta a las distintas reglas de nómina que requieren diferentes países o regiones). De lo contrario, seleccione **No**.

### <a name="does-solution-enable-translations"></a>Does solution enable translation(s)? (¿La solución permite traducciones?)

Responda **Sí** si el texto de la solución se puede traducir en otros idiomas. De lo contrario, seleccione **No**.

Seleccione **Guardar borrador** antes de continuar.

## <a name="publish"></a>Publicar

### <a name="submit-offer-to-preview"></a>Envío de la oferta para tener una versión preliminar

Una vez que haya completado todas las secciones necesarias de la oferta, seleccione **Revisar y publicar** en la esquina superior derecha del portal.

Si es la primera vez que publica esta oferta, puede:

- Ver el estado de finalización de cada sección de la oferta.
    - **No iniciada**: Significa que la sección no se ha modificado y se debe completar.
    - **Incompleta**: Significa que la sección tiene errores que deben corregirse o que requiere más información. Vuelva a las secciones y actualícelas.
    - **Completa**: Significa que la sección está completa, que se han proporcionado todos los datos necesarios y que no hay ningún error. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- En la sección **Notas para la certificación**, envíe instrucciones para la realización de pruebas al equipo de certificación para asegurarse de que la aplicación se prueba correctamente, además de todas las notas adicionales que pueden resultar útiles para conocer la aplicación. Para obtener más información sobre las instrucciones de prueba y la finalización de una primera publicación, vea [Validación funcional de Dynamics 365 Finance and Operations de AppSource](https://docs.microsoft.com/azure/marketplace/dynamics-365-finance-operations-functional-validation).
- Enviar la oferta para su publicación, para lo que debe seleccionar **Enviar**. Le enviaremos un correo electrónico que le avise cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación. Vuelva al Centro de partners y seleccione **Publicar** para ofrecer la oferta al público.

## <a name="next-step"></a>Paso siguiente

- [Actualización de una oferta existente en Marketplace comercial](./update-existing-offer.md)
