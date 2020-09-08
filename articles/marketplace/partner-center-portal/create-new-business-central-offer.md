---
title: 'Creación de una oferta de Dynamics 365 Business Central: marketplace comercial de Microsoft'
description: Conozca los pasos y las consideraciones para crear una oferta de Dynamics 365 Business Central en el marketplace comercial de Microsoft del Centro de partners. Puede mostrar o vender su oferta en Azure Marketplace o a través del programa Proveedor de soluciones en la nube (CSP).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: navits09
ms.author: navits
ms.date: 08/07/2020
ms.openlocfilehash: 4226f9cc45589b8c555d3d54918e31a04acbfc6b
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88817058"
---
# <a name="create-a-dynamics-365-business-central-offer"></a>Creación de una oferta de Dynamics 365 Business Central

En este artículo se explica cómo crear una nueva oferta de Dynamics 365 Business Central. [Microsoft Dynamics 365 Business Central](https://dynamics.microsoft.com/business-central) es una sistema de planeación de recursos empresariales (ERP) que controla una amplia gama de procesos empresariales, incluidas las finanzas, las operaciones, la cadena de suministro, la administración de relaciones con el cliente, la administración de proyectos y el comercio electrónico. Los paquetes Premium también admiten el modelo de implementación clásico y la fabricación. Todas las ofertas de Dynamics 365 Business Central deben pasar por nuestro proceso de certificación.

Antes de comenzar, debe [crear una cuenta de marketplace comercial en el Centro de partners](create-account.md), si aún no lo ha hecho. Asegúrese de que la cuenta está inscrita en el programa Marketplace comercial.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página de Información general, seleccione **+ Nueva oferta** > **Dynamics 365 Business Central**.

    ![Muestra el menú de navegación izquierdo.](./media/new-offer-dynamics-365-business-central.png)

> [!NOTE]
> Después de publicar una oferta, las modificaciones que se realicen en el Centro de partners solo aparecen en las tiendas en línea cuando la oferta se vuelve a publicar. Asegúrese de volver a realizar la publicación siempre después de realizar cambios.

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
- Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
- El alias de la oferta no se puede cambiar después de seleccionar **Crear**.

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

Aquí es donde se definen los detalles de la oferta, como el nombre, la descripción y las imágenes.

> [!NOTE]
> Solo puede proporcionar los detalles de la descripción de la oferta en un idioma. No es necesario que los detalles de la oferta estén en inglés si la descripción de la oferta comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es aceptable proporcionar una *dirección URL de vínculo de ayuda* para ofrecer contenido en un idioma distinto del que se usa en el contenido del anuncio de la oferta.

A continuación se muestra un ejemplo de cómo aparece la información de la oferta en Microsoft AppSource (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/example-d365-business-central.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
2. Productos
3. Categorías
4. Dirección de soporte técnico (vínculo)
5. Términos de uso
6. Directiva de privacidad
7. Nombre de la oferta
8. Resumen
9. Descripción
10. Capturas de pantallas o vídeos

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

### <a name="helpprivacy-urls"></a>Direcciones URL de ayuda y privacidad

Esta sección le permite proporcionar vínculos para ayudar a los clientes a comprender mejor su oferta.

#### <a name="help-link"></a>Vínculo de ayuda

Escriba la dirección URL en la que los clientes pueden obtener más información sobre su oferta. El **vínculo de ayuda** no puede ser el mismo que la **dirección URL de soporte técnico** (se explica a continuación).

#### <a name="privacy-policy-link"></a>Vínculo de la directiva de privacidad

Escriba la dirección URL a la directiva de privacidad de su organización. Usted es el responsable no solo de garantizar que la aplicación cumple la normativa y la legislación en lo tocante a la privacidad, sino también de proporcionar una directiva de privacidad válida.

### <a name="contact-information"></a>Información de contacto

En esta sección, debe proporcionar el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes, pero estará disponible para Microsoft y puede proporcionarse a los asociados de CSP.

En la sección **Contacto de soporte técnico**, también debe proporcionar la **dirección URL de soporte técnico** en la que los asociados de CSP pueden encontrar soporte técnico para la oferta. La dirección URL de soporte técnico no puede ser la misma que el **vínculo de ayuda**.

### <a name="supporting-documents"></a>Documentos relacionados

Proporcione al menos un documento de marketing relacionado (y un máximo de tres) aquí, como notas del producto, folletos, listas de comprobación o presentaciones. Estos documentos deben estar en formato .PDF.

### <a name="marketplace-images"></a>Imágenes de Marketplace

Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear un logotipo **Pequeño**. Opcionalmente, puede reemplazarlo por una imagen diferente.

- **Grande** (de 216 x 216 a 350 x 350 píxeles, obligatorio)
- **Pequeño** (48 x 48 píxeles, opcional)

Estos logotipos se usan en distintos lugares de la publicación:

[!INCLUDE [logos-appsource-only](../includes/logos-appsource-only.md)]

[!INCLUDE [Logo tips](../includes/graphics-suggestions.md)]

>[!NOTE]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio `https://upload.xboxlive.com` que usa el Centro de partners.

#### <a name="screenshots"></a>Capturas de pantalla

Agregue capturas de pantallas que muestren el funcionamiento de la oferta. Son necesarios al menos tres capturas de pantalla y puede agregar hasta cinco. Todas las capturas de pantallas deben tener 1280 x 720 píxeles.

#### <a name="videos"></a>Vídeos

Opcionalmente, puede agregar hasta cinco vídeos que muestren la oferta. Estos vídeos se deben hospedar en YouTube o Vimeo. En cada uno de ellos, escriba el nombre del vídeo, su dirección URL y una imagen en miniatura del vídeo (1280 x 720 píxeles).

#### <a name="additional-marketplace-listing-resources"></a>Recursos adicionales de la descripción de Marketplace

[Procedimientos recomendados la descripción de ofertas en Marketplace](../gtm-offer-listing-best-practices.md)

Seleccione **Guardar borrador** antes de continuar.

## <a name="availability"></a>Disponibilidad

Esta página le proporciona opciones sobre dónde y cómo hacer que la oferta esté disponible.

### <a name="markets"></a>Mercados

Esta sección le permite especificar los mercados en los que la oferta debe estar disponible. Para ello, seleccione **Editar mercados**, que mostrará la ventana emergente **Selección de mercado**.

Seleccione al menos un mercado para publicar la oferta. Elija **Seleccionar todo** para que su oferta esté disponible en cada mercado posible o seleccione los mercados específicos que quiera agregar.

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

### <a name="package-type"></a>Tipo de paquete

Seleccione la opción que se aplica a su oferta:

* **Complemento**: una aplicación de complemento amplía la experiencia y la funcionalidad existente de Dynamics 365 Business Central. Para obtener más información, consulte [Aplicaciones de complemento](https://docs.microsoft.com/dynamics365/business-central/dev-itpro/developer/readiness/readiness-add-on-apps).
* **Connect**: se puede usar una aplicación de Connect en el escenario en el que debe establecerse una conexión punto a punto entre Dynamics 365 Business Central y una solución o servicio de terceros. Para obtener más información, consulte [Aplicaciones de Connect](https://docs.microsoft.com/dynamics365/business-central/dev-itpro/developer/readiness/readiness-connect-apps).

### <a name="file-upload"></a>Carga de archivos

Si seleccionó **Complemento** anteriormente, aquí se cargará el archivo de paquete de la oferta, junto con los archivos de paquete de cualquier extensión en la que tenga dependencias.

#### <a name="extensions-package-file"></a>Archivo de paquete de extensiones

Cargue el archivo de paquete de extensión (.app) de la oferta.

#### <a name="library-package-file"></a>Archivo de paquete de biblioteca

Se requiere si la oferta debe instalarse junto con otra extensión que no se publicará en Marketplace. Si es así, cargue aquí su archivo .app.

#### <a name="dependency-package-file"></a>Archivo de paquete de dependencia

Se requiere si la oferta debe instalarse junto con otra extensión que ya se ha publicado en Marketplace. Si es así, cargue aquí su archivo `.app` o `.zip`.

### <a name="url-to-app-installation"></a>Dirección URL a la instalación de la aplicación

Si seleccionó **Connect** anteriormente, proporcione la dirección de la instalación de la aplicación aquí. En el caso de los servicios conectados que no requieren instalación, proporcione la dirección de la página de aterrizaje del servicio o la página de registro.

Seleccione **Guardar borrador** antes de continuar.

## <a name="test-drive-technical-configuration"></a>Configuración técnica de la versión de prueba

Esta página permite configurar una demostración («versión de prueba») que permite a los clientes probar la oferta antes de adquirirla. Más información en [Qué es la versión de prueba](../what-is-test-drive.md).

Para habilitar una versión de prueba, seleccione la pestaña**Habilitar una versión de prueba** y marque la casilla[Configuración de la oferta](#test-drive). Para eliminar la versión de prueba de la oferta, desactive esta casilla.

Cuando haya terminado de configurar la versión de prueba, seleccione **Guardar borrador** antes de continuar.

## <a name="supplemental-content"></a>Contenido complementario

Esta página le permite proporcionar información adicional sobre su oferta, lo que nos ayudará a validarla. Esta información no se muestra a los clientes ni se publica en Marketplace.

### <a name="target-release"></a>Versión de destino

Indique qué versión de Microsoft Dynamics Business Central tiene como destino su solución: **Actual**, **Siguiente mayor** o **Siguiente menor**. Esta información nos permite probar la solución correctamente.

### <a name="supported-editions"></a>Ediciones compatibles

Si su oferta requiere la edición Premium de Microsoft Dynamics 365 Business Central, seleccione solo **Premium**. En caso contrario, seleccione **Essentials** y **Premium**.

### <a name="key-usage-scenario"></a>Escenario de uso clave

Debe cargar un archivo PDF que enumere los escenarios de uso claves de la oferta enumerados en un documento (formato .pdf). Nuestro equipo de validación puede comprobar todos los escenarios que se enumeran aquí antes de aprobar su oferta para Marketplace.

### <a name="app-tests-automation"></a>Automatización de pruebas de la aplicación

Si su oferta es una aplicación de complemento, debe cargar un archivo de **automatización de pruebas de la aplicación** (.app). Este archivo no es aplicable a las aplicaciones de Connect.

### <a name="test-accounts"></a>Cuentas de prueba

Si se necesita una cuenta de prueba para que nuestro equipo de certificación revise correctamente su oferta, cargue un archivo .pdf, .doc o .docx con la información de las **cuentas de prueba**.

## <a name="publish"></a>Publicar

### <a name="submit-offer-to-preview"></a>Envío de la oferta para tener una versión preliminar

Una vez que haya completado todas las secciones necesarias de la oferta, seleccione **Publicar** en la esquina superior derecha del portal. Se le redirigirá a la página **Review and publish** (Revisar y publicar). 

Si es la primera vez que publica esta oferta, puede:

- Ver el estado de finalización de cada sección de la oferta.
    - *No iniciada*: significa que la sección no se ha modificado y se debe completar.
    - *Incompleta*: significa que la sección tiene errores que deben corregirse o que requiere más información. Vuelva a las secciones y actualícelas.
    - *Completa*: significa que la sección está completa, que se han proporcionado todos los datos necesarios y que no hay ningún error. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- En la sección **Notas para la certificación**, envíe instrucciones para la realización de pruebas al equipo de certificación para asegurarse de que la aplicación se prueba correctamente, además de todas las notas adicionales que pueden resultar útiles para conocer la aplicación.
- Enviar la oferta para su publicación, para lo que debe seleccionar **Enviar**. Le enviaremos un correo electrónico cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación. Vuelva al Centro de partners y seleccione **Publicar** para ofrecer la oferta al público.

## <a name="next-steps"></a>Pasos siguientes

- [Actualización de una oferta existente en Marketplace comercial](./update-existing-offer.md)
