---
title: Creación de una oferta de Dynamics 365 for Customer Engagement y PowerApps en el marketplace comercial de Microsoft
description: Creación de una nueva oferta de Dynamics 365 for Customer Engagement y PowerApps para mostrarla o venderla en Azure Marketplace, AppSource, o bien mediante el programa Proveedor de soluciones en la nube (CSP) del Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: navits09
ms.author: navits
ms.date: 06/17/2020
ms.openlocfilehash: 000413267d09f5f51ffdb50ccc5aeaab78a18b5e
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88036664"
---
# <a name="create-a-dynamics-365-for-customer-engagement--powerapps-offer"></a>Creación de una oferta de Dynamics 365 for Customer Engagement y PowerApps

En este tema se explica cómo crear una nueva oferta de Dynamics 365 for Customer Engagement y PowerApps. Todas las aplicaciones de Dynamics 365 for Customer Engagement (PowerApps, Sales, Service, Project Service y Field Service) deben someterse a nuestro proceso de certificación y admitir una experiencia de evaluación. El proceso de certificación comprueba su solución en busca de requisitos estándar, compatibilidad y procedimientos adecuados. La experiencia de evaluación permite a los usuarios implementar la solución en un entorno de Dynamics 365 en vivo.

Antes de comenzar, [cree una cuenta de marketplace comercial en el Centro de partners](create-account.md), si aún no lo ha hecho. Asegúrese de que la cuenta está inscrita en el programa Marketplace comercial.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página Información general, seleccione **+ Nueva oferta** > **Dynamics 365 for Customer Engagement y PowerApps**.

    ![Muestra el menú de navegación izquierdo.](./media/new-offer-dynamics-365-customer-engagement-powerapps.png)

> [!NOTE]
> Después de publicar una oferta, las modificaciones que se realicen en el Centro de partners solo aparecen en los escaparates cuando la oferta se vuelve a publicar. Asegúrese de volver a realizar la publicación siempre después de realizar cambios.

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
- Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
- No se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-setup"></a>Configuración de la oferta

Siga estos pasos para configurar la oferta.

### <a name="how-do-you-want-potential-customers-to-interact-with-this-listing-offer"></a>¿Cómo quiere que los clientes potenciales interactúen con esta oferta en la lista?

Seleccione la opción que quiere usar para esta oferta.

#### <a name="get-it-now-free"></a>Obténgalo ahora (de forma gratuita)

Presente su oferta a los clientes de forma gratuita proporcionándoles una dirección URL válida (que comienza por *http* o *https*) desde donde pueden acceder a su aplicación.  Por ejemplo, `https://contoso.com/my-app`.

#### <a name="free-trial-listing"></a>Evaluación gratuita (anuncio)

Para presentar la oferta a los clientes con un vínculo a una evaluación gratuita, proporcióneles una dirección URL válida (que empiece por `http` o `https`) desde la que puedan obtenerla.  Por ejemplo, `https://contoso.com/trial/my-app`. Las ofertas que incluyen evaluaciones gratuitas se crean, administran y configuran desde su servicio y no tienen suscripciones administradas por Microsoft.

> [!NOTE]
> Los tokens que la aplicación recibirá a través del vínculo de evaluación gratuita solo se pueden usar para obtener información de usuario mediante Azure Active Directory (Azure AD) para automatizar la creación de cuentas en la aplicación. No se admiten las cuentas de Microsoft para la autenticación con este token.

#### <a name="contact-me"></a>Ponerse en contacto conmigo

Recopile información de contacto de los clientes mediante la conexión de su sistema de Administración de relaciones con los clientes (CRM). El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre e identificador de la oferta y el origen de Marketplace donde la ha encontrado, se envían al sistema CRM que se ha configurado. Para obtener más información sobre cómo configurar CRM, consulte [Clientes potenciales](#customer-leads).

Seleccione **Guardar borrador** antes de continuar.

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

### <a name="applicable-dynamics-365-products"></a>Productos de Dynamics 365 a los que se aplica

Seleccione todos los productos de Dynamics 365 a los que se aplica esta oferta.

### <a name="app-version"></a>Versión de la aplicación

Escriba el número de versión de la oferta. Los clientes verán esta versión en la página de detalles de la oferta. Si solo va a actualizar el número de versión debido a cambios descriptivos o de marketing, active la casilla **Solo cambios de marketing**. Esta opción permite que la oferta pueda omitir las etapas de certificación y aprovisionamiento.

### <a name="terms-and-conditions"></a>Términos y condiciones

Proporcione sus propios términos y condiciones legales aquí. También puede proporcionar la dirección donde se pueden encontrar. Los clientes deberán aceptar estos términos para poder probar la oferta.

Seleccione **Guardar borrador** antes de continuar.

## <a name="offer-listing"></a>Descripción de la oferta

La página muestra los idiomas en los que se mostrará la oferta. La única opción disponible actualmente es **Inglés (Estados Unidos)** .

Tendrá que definir los detalles de Marketplace para cada idioma y mercado; por ejemplo, el nombre de la oferta, la descripción e imágenes. Seleccione el nombre de idioma o mercado para proporcionar esta información.

> [!NOTE]
> No es necesario que la oferta (por ejemplo, la descripción, los documentos, las capturas de pantallas y los términos de uso) esté en inglés si la descripción de la oferta comienza por la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es aceptable proporcionar una *dirección URL de vínculo útil* para ofrecer contenido en un idioma distinto del que se usa en el contenido del anuncio de la oferta.

A continuación se muestra un ejemplo de cómo aparece la información de la oferta en Microsoft AppSource (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/example-azure-marketplace-d365-customer-engagement.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
2. Productos
3. Categorías
4. Dirección de soporte técnico (vínculo)
5. Dirección de las Condiciones de uso (vínculo)
6. Nombre de la oferta
7. Descripción
8. Capturas de pantallas o vídeos

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

Escriba la dirección en la que los clientes pueden obtener más información sobre la oferta.

#### <a name="privacy-policy-url"></a>Dirección URL de la directiva de privacidad

Escriba la dirección de la directiva de privacidad de su organización. Usted es el responsable no solo de garantizar que la aplicación cumple la normativa y la legislación en lo tocante a la privacidad, sino también de proporcionar una directiva de privacidad válida.

### <a name="contacts"></a>Contactos

Indique el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes, pero estará disponible para Microsoft y puede proporcionarse a los asociados de CSP.

En la sección **Contacto de soporte técnico**, también debe proporcionar la **dirección URL de soporte técnico** en la que los asociados de CSP pueden encontrar soporte técnico para la oferta.

### <a name="supporting-documents"></a>Documentos relacionados

Proporcione aquí al menos un documento de marketing relacionado (y un máximo de tres), como notas del producto, folletos, listas de comprobación o presentaciones, en formato PDF.

### <a name="marketplace-images"></a>Imágenes de Marketplace

Proporcione logotipos e imágenes para la oferta. Todas las imágenes deben estar en formato PNG.

[!INCLUDE [logotips](../includes/graphics-suggestions.md)]

>[!NOTE]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione el logotipo de la oferta en tres tamaños de píxel:
- **Pequeño** (obligatorio; 48 x 48)
- **Grande** (obligatorio; 216 x 216)
- **Ancho** (opcional; 255 x 115)

[!INCLUDE [logos-appsource-only](../includes/logos-appsource-only.md)]

#### <a name="screenshots"></a>Capturas de pantalla

Agregue capturas de pantallas que muestren el funcionamiento de la oferta. Se requiere al menos una captura de pantalla y puede agregar un máximo de cinco. Todas las capturas de pantallas deben tener 1280 x 720 píxeles.

#### <a name="videos"></a>Vídeos

Opcionalmente, puede agregar hasta cuatro vídeos que muestren la oferta. Estos vídeos se deben hospedar en YouTube o Vimeo. En cada uno de ellos, escriba el nombre del vídeo, su dirección URL y una imagen en miniatura del vídeo (1280 x 720 píxeles).

#### <a name="additional-marketplace-listing-resources"></a>Recursos adicionales de la descripción de Marketplace

[Procedimientos recomendados la descripción de ofertas en Marketplace](../gtm-offer-listing-best-practices.md)

Seleccione **Guardar borrador** antes de continuar.

## <a name="availability"></a>Disponibilidad

Esta página le permite definir dónde y cómo hacer que la oferta esté disponible.

### <a name="markets"></a>Mercados

Aquí especifica los mercados en los que la oferta debe estar disponible. Para ello, seleccione **Editar mercados**, que mostrará la ventana emergente **Selección de mercado**.

De manera predeterminada, no hay ningún mercado seleccionado, pero debe seleccionar al menos uno para publicar la oferta. Elija **Seleccionar todo** para que su oferta esté disponible en cada mercado posible o seleccione los mercados específicos que quiera agregar.

Las selecciones aquí solo se aplican a las nuevas adquisiciones. Si algún usuario ya tiene su aplicación en un determinado mercado y posteriormente lo quita, las personas que ya tienen la oferta en dicho mercado podrán seguir utilizándola, pero ningún cliente nuevo en ese mercado podrá obtener su oferta.

> [!IMPORTANT]
> Es su responsabilidad cumplir los requisitos legales locales, incluso si esos requisitos no aparecen aquí ni en el Centro de partners. Incluso si selecciona todos los mercados, las leyes y restricciones locales u otros factores pueden impedir que se muestren determinadas ofertas en algunos países y regiones.

### <a name="preview-audience"></a>Público preliminar

Antes de publicar su oferta en la oferta de Marketplace más amplia, primero deberá ponerla a disposición de un **público preliminar** limitado. Escriba una **clave de ocultación** (cualquier cadena que use solo letras minúsculas o números) aquí. Los miembros del público preliminar pueden usar esta clave de ocultación como token para ver una versión preliminar de la oferta en Marketplace.

Después, cuando esté listo para que la oferta esté disponible y quite la restricción de versión preliminar, deberá quitar la **tecla de ocultación** y volver a publicarla.

Seleccione **Guardar borrador** antes de continuar.

## <a name="technical-configuration"></a>Configuración técnica

Esta página especifica los detalles técnicos usados para conectarse a la oferta. Esta conexión nos permite aprovisionar su oferta para el cliente final, en caso de que este elija adquirirla.

### <a name="base-license-model"></a>Modelo de licencia básico

El modelo de licencia base determina cómo se asignan los clientes a la aplicación en el centro de administración de CRM. Seleccione **recurso** para las licencias que se basan en instancias o **Usuario** si se asigna una licencia por cada inquilino.

### <a name="requires-s2s-outbound-and-crm-secure-store-access"></a>Se requiere un acceso saliente de S2S y acceso al almacenamiento seguro de CRM

Marque esta casilla para permitir la configuración del acceso al almacenamiento seguro de CRM o del acceso saliente de servidor a servidor (S2S). Esta característica requiere una atención especial por parte del equipo de Dynamics 365 durante la fase de certificación. Microsoft se pondrá en contacto con usted para completar los pasos adicionales para admitir esta característica.

### <a name="application-configuration-url"></a>Dirección URL de configuración de aplicación

Proporcione la dirección URL de la página web de configuración que permite al cliente configurar la aplicación.

### <a name="crm-package"></a>Paquete de CRM

En el campo**Dirección URL de la ubicación del paquete**, escriba la dirección URL de una cuenta de Azure Blob Storage que contenga el archivo .zip del paquete de CRM cargado. Incluya una clave SAS de solo lectura para permitir que Microsoft pueda recoger el paquete para su comprobación.

> [!IMPORTANT]
> Para evitar un bloque de publicación, asegúrese de que la fecha de expiración de la dirección URL del almacenamiento de blobs no haya expirado. Se puede revisar la fecha accediendo a la directiva. Se recomienda que el **Tiempo de expiración** sea al menos dentro de un mes.

Seleccione la casilla etiquetada **Hay más de un paquete de CRM en el archivo de paquete**, si procede. Si es así, asegúrese de incluir todos los paquetes en el archivo ZIP.

Para obtener información detallada sobre cómo compilar el paquete y actualizar su estructura, consulte [Paso 3: Crear un paquete AppSource para la aplicación](https://docs.microsoft.com/powerapps/developer/common-data-service/create-package-app-appsource).

### <a name="crm-package-availability"></a>Disponibilidad del paquete de CRM

En esta sección, seleccione **+ Agregar región** para especificar las regiones geográficas en las que el paquete de CRM estará disponible para los clientes. La implementación en las siguientes regiones soberanas requiere un permiso y una validación especiales durante el proceso de certificación: [Alemania](../../germany/index.yml), la [nube del US Gov](../../azure-government/documentation-government-welcome.md) y TIP.

De manera predeterminada, la **dirección URL de configuración de aplicación** que especificó anteriormente se usará para cada región. Si lo prefiere, puede especificar una dirección URL de configuración de aplicación independiente para una o varias regiones específicas. 

Seleccione **Guardar borrador** antes de continuar.

## <a name="test-drive-technical-configuration"></a>Configuración técnica de la versión de prueba

Esta página permite configurar una demostración («versión de prueba») que permite a los clientes probar la oferta antes de adquirirla. Más información en [Qué es la versión de prueba](../what-is-test-drive.md).

Para habilitar una versión de prueba, seleccione la pestaña**Habilitar una versión de prueba** y marque la casilla[Configuración de la oferta](#test-drive). Para eliminar la versión de prueba de la oferta, desactive esta casilla.

Cuando haya terminado de configurar la versión de prueba, seleccione **Guardar borrador** antes de continuar.

## <a name="supplemental-content"></a>Contenido complementario

Esta página le permite proporcionar información adicional sobre su oferta, lo que nos ayudará a validarla. Esta información no se muestra a los clientes ni se publica en Marketplace.

### <a name="key-usage-scenario"></a>Escenario de uso clave

Cargue un archivo PDF que indique los escenarios de uso claves de la oferta. Es posible que nuestro equipo de validación compruebe todos los escenarios que se enumeran aquí antes de aprobar la oferta para Marketplace.

Seleccione **Guardar borrador** antes de continuar.

## <a name="publish"></a>Publicar

### <a name="submit-offer-to-preview"></a>Envío de la oferta para tener una versión preliminar

Una vez que haya completado todas las secciones necesarias de la oferta, seleccione **Revisar y publicar** en la esquina superior derecha del portal.

Si es la primera vez que publica esta oferta, puede:

- Ver el estado de finalización de cada sección de la oferta.
    - **No iniciada**: la sección no se ha modificado y se debe completar.
    - **Incompleta**: la sección tiene errores que deben corregirse o se requiere más información. Vuelva a las secciones y actualícelas.
    - **Completa**: la sección está completa, se han proporcionado todos los datos necesarios y no hay ningún error. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- En la sección **Notas para la certificación**, envíe instrucciones para la realización de pruebas al equipo de certificación para asegurarse de que la aplicación se prueba correctamente, además de todas las notas adicionales que pueden resultar útiles para conocer la aplicación.
- Enviar la oferta para su publicación, para lo que debe seleccionar **Enviar**. Le enviaremos un correo electrónico que le avise cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación. Vuelva al Centro de partners y seleccione **Publicar** para la oferta que desea ofrecer al público.

## <a name="next-step"></a>Paso siguiente

- [Actualización de una oferta existente en Marketplace comercial](./update-existing-offer.md)
