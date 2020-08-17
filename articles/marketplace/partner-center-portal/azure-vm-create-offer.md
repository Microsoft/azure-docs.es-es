---
title: Creación de una oferta de máquina virtual de Azure en Azure Marketplace
description: Obtenga información sobre cómo publicar una oferta de máquina virtual en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: emuench
ms.author: mingshen
ms.date: 07/28/2020
ms.openlocfilehash: 0adffbb9058141ea89daefc292b937841659c879
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798076"
---
# <a name="create-an-azure-virtual-machine-offer-on-azure-marketplace"></a>Creación de una oferta de máquina virtual de Azure en Azure Marketplace

En este artículo se explica cómo crear y publicar una oferta de máquina virtual de Azure para [Azure Marketplace](https://azuremarketplace.microsoft.com/). Se dirige a máquinas virtuales basadas en Windows y en Linux que contienen un sistema operativo, un disco duro virtual (VHD) y hasta 16 discos de datos. 

Antes de comenzar, debe [crear una cuenta de marketplace comercial en el Centro de partners](create-account.md). Asegúrese de que la cuenta esté inscrita en el programa Marketplace comercial.

## <a name="introduction"></a>Introducción

### <a name="the-benefits-of-publishing-to-azure-marketplace"></a>Ventajas de publicar en Azure Marketplace

Al publicar las ofertas en Azure Marketplace, puede:

- Promocionar la empresa con ayuda de la marca de Microsoft.
- Llegar a más de 100 millones de usuarios de Office 365 y Dynamics 365, y a más de 200 000 organizaciones.
- Obtenga clientes potenciales de alta calidad de estos marketplaces.
- Promocionar sus servicios por parte de los equipos de ventas de campo y televenta de Microsoft.

### <a name="before-you-begin"></a>Antes de empezar

Si todavía no lo ha hecho, consulte [Guía de publicación de ofertas de máquinas virtuales](../marketplace-virtual-machines.md) y este material sobre máquinas virtuales de Azure:

- Guías de inicio rápido
  - [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/)
  - [Plantillas de inicio rápido de Azure en GitHub](https://github.com/azure/azure-quickstart-templates)
- Tutoriales
  - [Máquinas virtuales Linux](../../virtual-machines/linux/tutorial-manage-vm.md)
  - [Máquinas virtuales Windows](../../virtual-machines/windows/tutorial-manage-vm.md)
- Ejemplos
  - [Ejemplos de la CLI de Azure para máquinas virtuales Linux](../../virtual-machines/linux/cli-samples.md)
  - [Azure PowerShell para máquinas virtuales Linux](../../virtual-machines/linux/powershell-samples.md)
  - [Ejemplos de la CLI de Azure para máquinas virtuales Windows](../../virtual-machines/windows/cli-samples.md)
  - [Azure PowerShell para máquinas virtuales Windows](../../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm-quick.md)

### <a name="fundamentals-in-technical-knowledge"></a>Aspectos básicos de los conocimientos técnicos

El proceso de diseño, compilación y prueba de estos recursos lleva tiempo y requiere experiencia tanto en la plataforma de Azure como en las tecnologías que se usan para crear la oferta.

El equipo de ingeniería debe tener unos conocimientos básicos y prácticos sobre las siguientes tecnologías de Microsoft:

- [Servicios de Azure](https://azure.microsoft.com/services/)
- [Diseño y arquitectura de las aplicaciones de Azure](https://azure.microsoft.com/solutions/architecture/)
- [Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/), [Azure Storage](https://azure.microsoft.com/services/?filter=storage#storage) y [Redes de Azure](https://azure.microsoft.com/services/?filter=networking#networking)

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el panel izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página **Información general**, seleccione **Nueva oferta** > **Máquina virtual de Azure**.

    ![Captura de pantalla que muestra las opciones de menú del panel izquierdo y el botón "Nueva oferta".](./media/new-offer-azure-virtual-machine.png)

> [!NOTE]
> Una vez publicada la oferta, las modificaciones que realice en el Centro de partners aparecerán en Azure Marketplace solo después de volver a publicarla. Tras realizar cambios en la oferta, asegúrese siempre de volver a publicarla.

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Este identificador se muestra a los clientes en la dirección web de la oferta de Azure Marketplace y en Azure PowerShell y la CLI de Azure, si procede.
- Use solo letras minúsculas y números. El identificador puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. El alias de la oferta es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se utiliza en Azure Marketplace. Es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-setup"></a>Configuración de la oferta

### <a name="test-drive"></a>Versión de prueba

Una *versión de prueba* es una excelente manera de presentar la oferta a los clientes potenciales. Le da la opción de "probar antes de comprar", lo que puede ayudar a aumentar las conversiones y a generar clientes potenciales muy competentes. Para más información, consulte [¿Qué es la versión de prueba?](../what-is-test-drive.md)

Para habilitar una versión de prueba durante un período de tiempo determinado, seleccione la casilla **Enable a test drive** (Habilitar una versión de prueba). Para eliminar la versión de prueba de la oferta, desactive la casilla.

Recursos adicionales de la versión de prueba:

- [Procedimientos técnicos recomendados](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices)
- [Procedimientos recomendados de marketing](../what-is-test-drive.md)
- [Descargue el archivo PDF de información general de la versión de prueba](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf) (asegúrese de que el bloqueador de elementos emergentes esté desactivado).

### <a name="customer-leads"></a>Clientes potenciales

Al publicar la oferta en el marketplace comercial con el Centro de partners, conéctela a un sistema de administración de relaciones con clientes (CRM). Esto le permite recibir información de contacto del cliente en cuanto alguien expresa interés en el producto o lo usa. Es necesario conectarse a un CRM si desea habilitar una versión de prueba (consulte la sección anterior). De lo contrario, la conexión a CRM es opcional.

1. Seleccione un destino de clientes potenciales al que quiera que enviemos los clientes potenciales. El Centro de partners admite los siguientes sistemas CRM:
    - [Dynamics 365](commercial-marketplace-lead-management-instructions-dynamics.md) para la involucración del cliente
    - [Marketo](commercial-marketplace-lead-management-instructions-marketo.md)
    - [Salesforce](commercial-marketplace-lead-management-instructions-salesforce.md)

    > [!NOTE]
    > Si su sistema CRM no aparece en la lista anterior, use el [almacenamiento de tablas de Azure](commercial-marketplace-lead-management-instructions-azure-table.md) o el [punto de conexión HTTPS](commercial-marketplace-lead-management-instructions-https.md) para almacenar los datos del clientes potencial. A continuación, exporte los datos al sistema CRM.

1. Conecte su oferta al destino de clientes potenciales cuando la publique en el Centro de partners.
1. Compruebe que la conexión al destino de clientes potenciales está configurada correctamente. Después de publicarla en el Centro de partners, Microsoft valida la conexión y le envía un cliente potencial de prueba. Mientras obtiene una vista previa de la oferta antes de publicarla, también puede probar la conexión de los clientes potenciales tratando de implementar la oferta en el entorno de versión preliminar.
1. Asegúrese de que la conexión con el destino de clientes potenciales permanezca actualizada para no perder ningún cliente potencial.

1. Seleccione **Guardar borrador** antes de continuar.

## <a name="properties"></a>Propiedades

En la página **Propiedades**, se definen las categorías que se usan para agrupar la oferta en Azure Marketplace, la versión de la aplicación y los contratos legales que respaldan la oferta.

### <a name="category"></a>Category

Seleccione categorías y subcategorías para colocar la oferta en las áreas de búsqueda adecuadas del marketplace. En la descripción de la oferta, asegúrese de indicar la forma en que la oferta admite estas categorías. Seleccione:

- Al menos una y un máximo de dos categorías, incluidas una categoría principal y una secundaria (opcional).
- Hasta dos subcategorías para cada categoría principal o secundaria. Si no hay ninguna subcategoría aplicable a la oferta, seleccione **No aplicable**.

Vea la lista completa de categorías y subcategorías en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md). Las ofertas de máquina virtual siempre aparecen en la categoría **Proceso** de Azure Marketplace.

### <a name="legal"></a>Información legal

Debe proporcionar los términos y condiciones de la oferta a los clientes. Tiene dos opciones:

- **Uso de términos y condiciones propios**

   Si quiere proporcionar sus propios términos y condiciones personalizados, escriba hasta 10 000 caracteres de texto en el cuadro **Términos y condiciones**. Si necesita una descripción más larga, escriba una única dirección web para los términos y condiciones. A los clientes les aparecerá como un vínculo activo.

   Sus clientes deben aceptar estos términos para poder probar la oferta.

- **Uso del Contrato estándar para el marketplace comercial de Microsoft**

   Para simplificar el proceso de adquisición para los clientes y reducir la complejidad legal para los proveedores de software, Microsoft ofrece un Contrato estándar para marketplace comercial. Al ofrecer el software con el Contrato estándar, los clientes tienen que leerlo y aceptarlo solo una vez, y no necesitan crear términos y condiciones personalizados.

   Para usar el Contrato estándar, seleccione la casilla **Use the Standard Contract for Microsoft's commercial marketplace** (¿Quiere usar el Contrato estándar para el marketplace comercial de Microsoft?) y, a continuación, en la ventana emergente, seleccione **Aceptar** (es posible que tenga que desplazarse hacia arriba para verlo).

   ![Captura de pantalla que muestra el panel Legal del Centro de partners con la casilla "Use the Standard Contract for Microsoft's commercial marketplace" (¿Quiere usar el Contrato estándar para el marketplace comercial de Microsoft?).](media/use-standard-contract.png)

  > [!NOTE]
  > Después de publicar una oferta con el Contrato estándar para el marketplace comercial, no puede usar sus propios términos y condiciones personalizados. Puede ofrecer su solución con el Contrato estándar o con sus propios términos y condiciones.

  Para más información, consulte [Contrato estándar para el marketplace comercial de Microsoft](../standard-contract.md). Descargue el archivo PDF [Contrato estándar](https://go.microsoft.com/fwlink/?linkid=2041178) (asegúrese de que el bloqueador de elementos emergentes esté desactivado).

  **Modificaciones del Contrato estándar**

  Las modificaciones del Contrato estándar le permiten seleccionar los términos del Contrato estándar por motivos de simplicidad y crear los términos para su producto o empresa. Los clientes necesitan revisar las modificaciones del contrato solo si ya han revisado y aceptado el Contrato estándar de Microsoft. Existen dos tipos de modificaciones:

  * **Modificaciones universales**: Estas modificaciones se aplican universalmente al Contrato estándar para todos los clientes. Se muestran a cada cliente de la oferta en el flujo de compra. Los clientes deben aceptar los términos del Contrato estándar y las modificaciones antes de que puedan usar su oferta. Puede proporcionar una única modificación universal por oferta. Puede escribir un número ilimitado de caracteres en este cuadro. Estos términos se muestran a los clientes en AppSource, Azure Marketplace o Azure Portal durante el flujo de detección y compra.

  * **Modificaciones personalizadas**: son modificaciones especiales del Contrato estándar que están destinadas a clientes específicos mediante identificadores de inquilino de Azure. Puede elegir el inquilino al que desea dirigirse. Solo los clientes del inquilino reciben los términos con modificaciones personalizadas en el flujo de compra de la oferta. Los clientes deben aceptar los términos del Contrato estándar y las modificaciones antes de que puedan usar su oferta.

    1. Seleccione **Agregar términos de modificación personalizados (10 como máximo)** . Puede proporcionar hasta diez términos de modificación personalizados por oferta. Haga lo siguiente:

       a. Escriba sus propios términos de modificación en el cuadro **Custom amendment terms** (Términos de modificación personalizados). Puede escribir un número ilimitado de caracteres. Solo los clientes de los identificadores de inquilino que se especifiquen para estos términos personalizados los verán en el flujo de compra de la oferta en Azure Portal.

       b. (Obligatorio) Proporcione los **identificador de inquilino**. Cada modificación personalizada puede tener como destino hasta 20 identificadores de inquilino. Si agrega una modificación personalizada, debe proporcionar al menos un identificador de inquilino, que identifica al cliente en Azure. El cliente puede encontrarlo para usted en Azure seleccionando **Azure Active Directory** > **Propiedades**. El valor del identificador de directorio es el identificador de inquilino (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e). También puede buscar el identificador de inquilino de la organización del cliente mediante su dirección web de nombre de dominio en [What is my Microsoft Azure and Office 365 tenant ID?](https://www.whatismytenantid.com/) (¿Cuál es mi identificador de inquilino de Microsoft Azure y Office 365?).

       c. (Opcional) Proporcione una **Descripción** detallada del identificador de inquilino que le ayude a identificar al cliente al que está destinada la modificación.

        > [!NOTE]
        > Estos dos tipos de modificaciones están emparejados entre sí. Los clientes específicos con modificaciones personalizadas también obtendrán la modificación universal al Contrato estándar durante la compra.

    1. Seleccione **Guardar borrador** antes de continuar.

## <a name="offer-listing"></a>Descripción de la oferta

En la página **Offer listing** (Descripción de la oferta), define los detalles de la oferta, como el nombre, la descripción, los vínculos y los contactos.

> [!NOTE]
> No es necesario que el contenido de la descripción de la oferta, por ejemplo, la descripción, los documentos, las capturas de pantallas y los términos de uso, esté en inglés si la descripción de la oferta comienza por la frase "Esta aplicación solo está disponible en \<non-English language>." También puede proporcionar una dirección URL que vincule a un sitio que ofrezca contenido en un idioma distinto del que se usa en el contenido de la descripción de la oferta.

### <a name="marketplace-details"></a>Detalles del marketplace

#### <a name="name"></a>Nombre

El nombre que escriba aquí se muestra a los clientes como el título de la descripción de la oferta. Este campo se rellena automáticamente con el texto que escribió en el cuadro **Offer alias** (Alias de la oferta) al crear la oferta. Puede cambiar este nombre más adelante. El nombre:

- Puede ser una marca comercial. Puede incluir símbolos de marca comercial y copyright.
- No puede contener más de 50 caracteres.
- No puede incluir emojis.

#### <a name="search-results-summary"></a>Resumen de los resultados de la búsqueda

Especifique una descripción breve de la oferta que se mostrará en los resultados de búsqueda de Azure Marketplace. Puede contener un máximo de 100 caracteres.

#### <a name="long-summary"></a>Resumen largo

Especifique una descripción larga de la oferta que se va a mostrar en los resultados de búsqueda de Azure Marketplace. Puede contener un máximo de 256 caracteres.

#### <a name="description"></a>Descripción

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Long description-3](./includes/long-description-3.md)]

#### <a name="privacy-policy-link"></a>Vínculo de la directiva de privacidad

Escriba la dirección web (URL) de la directiva de privacidad de la organización. Asegúrese de que la oferta cumpla con las leyes y regulaciones de privacidad. También debe publicar una directiva de privacidad válida en su sitio web.

### <a name="useful-links"></a>Vínculos útiles

Proporcione documentos en línea complementarios sobre su oferta. Para agregar un vínculo, seleccione **Add a link** (Agregar un vínculo) y, a continuación, rellene los campos siguientes:

- **Name**: los clientes verán el nombre en la página de detalles.
- **Dirección URL del vínculo**: escriba un vínculo para que los clientes vean el documento en línea.

### <a name="customer-support-links"></a>Vínculos de soporte al cliente

Proporcione el sitio web de soporte técnico en el que los clientes pueden ponerse en contacto con el equipo de soporte técnico.

- Sitio web de soporte técnico global de Azure
- Sitio web de soporte técnico de Azure Government

### <a name="partner-support-contact"></a>Contacto de soporte técnico del asociado

Proporcione información de contacto para que los asociados de Microsoft la usen cuando los clientes abran una incidencia de soporte técnico. Esta información no aparece en Azure Marketplace.

- Nombre
- Email
- Teléfono

### <a name="engineering-contact"></a>Contacto de ingeniería

Proporcione información de contacto para que Microsoft la use en caso de problemas con la oferta, incluidos los problemas de certificación. Esta información no aparece en Azure Marketplace.

- Nombre
- Email
- Teléfono

### <a name="azure-marketplace-media"></a>Elementos multimedia de Azure Marketplace

Proporcione logotipos e imágenes para usarlos con la oferta. Todas las imágenes deben estar en formato PNG. Las imágenes borrosas harán que se rechace el envío.

>[!NOTE]
>Si tiene problemas al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione un archivo PNG para el logotipo de tamaño **grande** (entre 216 x 216 y 350 x 350 píxeles). El Centro de partners lo utilizará para crear un logotipo de tamaño **pequeño** (48 x 48 píxeles) y un logotipo de tamaño **medio** (90 x 90 píxeles). Opcionalmente, puede reemplazarlos por imágenes diferentes.

Se requieren los tres tamaños de logotipo para su uso en distintos lugares de la lista:

- **Pequeño (48 x 48)**
- **Medio (90 x 90)**
- **Grande** (entre 216 x 216 y 350 x 350)

[!INCLUDE [Logo suggestions](./includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Capturas de pantalla

Agregue hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Cada captura de pantalla debe tener un tamaño de 1280 &times; 720 píxeles y el formato PNG. Cada captura de pantalla debe incluir un título.

#### <a name="videos"></a>Vídeos

Agregue hasta cinco vídeos que muestren la oferta. Los vídeos se deben hospedar en un servicio de vídeo externo. Escriba el nombre de cada vídeo, su dirección web y una imagen en miniatura en formato PNG del vídeo de 1280 &times; 720 píxeles.

Para ver más recursos sobre las listas de marketplace, consulte [Procedimientos recomendados para las listas de ofertas de marketplace](../gtm-offer-listing-best-practices.md).

Seleccione **Guardar borrador** antes de continuar.

## <a name="preview"></a>Versión preliminar

En la pestaña **Versión preliminar**, elija una **audiencia preliminar** limitada para validar la oferta antes de ponerla a disposición de una audiencia más amplia del marketplace comercial.

> [!IMPORTANT]
> Después de revisar la oferta en el panel **Versión preliminar**, seleccione **Publicar** a fin de incluir la oferta para la audiencia pública en el marketplace comercial.

Los GUID de identificador de suscripción de Azure, junto con una descripción adicional de cada uno, identifican el público preliminar. Los clientes no pueden ver ninguno de estos campos. Puede encontrar el identificador de suscripción de Azure en la página **Suscripciones** de Azure Portal.

Agregue al menos un identificador de suscripción de Azure, ya sea de forma individual (hasta 10 identificadores) o mediante la carga de un archivo CSV (hasta 100 identificadores). Al agregar estos identificadores de suscripción, puede definir quién puede obtener una vista previa de la oferta antes de publicarla. Si la oferta ya está publicada, todavía puede definir una audiencia preliminar para probar los cambios o las actualizaciones de la oferta.

> [!NOTE]
> Un público preliminar no es el mismo que un público privado. A una audiencia preliminar se le permite acceder a la oferta _antes_ de que se publique en Azure Marketplace. La audiencia privada podrá ver y validar todos los planes, incluidos aquellos que estén disponibles solo para una audiencia privada, después de que su oferta se publique por completo en Azure Marketplace. Una audiencia privada (definida en el panel **Precios y disponibilidad** del plan) tiene acceso exclusivo a un plan determinado.

Seleccione **Guardar borrador** antes de continuar con la siguiente sección.

## <a name="plan-overview"></a>Información general del plan

Puede proporcionar una serie de opciones de planes dentro de la misma oferta en el Centro de partners. Una oferta requiere al menos un plan (anteriormente denominada una SKU), que puede variar en términos de audiencia de monetización, regiones de Azure, características o imágenes de máquina virtual.

Después de crear los planes, la pestaña **Plan overview** (Información general del plan) muestra lo siguiente:

- Nombres de los planes
- Modelos de licencia
- Audiencia (pública o privada)
- Estado de publicación actual
- Acciones disponibles

Las acciones que están disponibles en el panel **Plan overview** (Información general del plan) varían en función del estado actual del plan.

- Si el estado del plan es un borrador, seleccione **Eliminar borrador**.
- Si el estado del plan se publica en directo, seleccione **Stop sell plan** (Detener el plan de venta) o **Sync private audience** (Sincronizar audiencia privada).

### <a name="create-a-new-plan"></a>Creación de un nuevo plan

Seleccione **Crear nuevo plan** en la parte superior. Aparece el cuadro de diálogo **Nuevo plan**.

En el cuadro **Id. de plan**, cree un identificador de plan único para cada plan de esta oferta. Este identificador será visible para los clientes en la dirección web del producto. Use solo letras minúsculas y números, guiones o caracteres de subrayado, y un máximo de 50 caracteres.

> [!NOTE]
> El id. de oferta no se puede cambiar después de seleccionar **Crear**.

En el cuadro **Nombre del plan**, escriba un nombre para este plan. Los clientes verán este nombre al decidir qué plan van a seleccionar en su oferta. Cree un nombre único que destaque claramente las diferencias entre los planes. Por ejemplo, puede especificar **Windows Server** con los planes *Pago por uso*, *BYOL*, *Avanzado* y *Enterprise*.

Seleccione **Crear**.

### <a name="plan-setup"></a>Configuración del plan

Establezca la configuración de alto nivel para el tipo de plan, especifique si reutiliza la configuración técnica de otro plan e identifique las regiones de Azure en las que debe estar disponible el plan. Las selecciones que se realicen aquí afectarán a los campos mostrados en otros paneles para el mismo plan.

#### <a name="reuse-a-technical-configuration"></a>Reutilización de una configuración técnica

Si tiene más de un plan del mismo tipo y los paquetes son idénticos entre ellos, puede seleccionar **This plan reuses technical configuration from another plan** (Este plan reutiliza la configuración técnica de otro plan). Esta opción le permite seleccionar uno de los otros planes del mismo tipo para esta oferta y reutilizar su configuración técnica.

> [!NOTE]
> Cuando reutilice la configuración técnica de otro plan, toda la pestaña **Configuración técnica** desaparecerá de este plan. Los detalles de configuración técnica del otro plan, incluidas las actualizaciones que realice en el futuro, se usarán también para este plan. Esta configuración no se puede modificar una vez publicado el plan.

#### <a name="azure-regions"></a>Regiones de Azure

El plan debe estar disponible al menos en una región de Azure.

Seleccione la opción **Azure global** para que el plan esté disponible para los clientes de todas las regiones globales de Azure que tengan la integración de marketplace comercial. Para más información, consulte [Disponibilidad geográfica y compatibilidad con monedas para Marketplace comercial](../marketplace-geo-availability-currencies.md).

Seleccione la opción **Azure Government** para que el plan esté disponible en la región de [Azure Government](../../azure-government/documentation-government-welcome.md). Esta región proporciona acceso controlado para los clientes de entidades tribales, locales, estatales o federales de Estados Unidos, así como para los asociados aptos para atenderlas. Como publicador, es responsable de los controles de cumplimiento, las medidas de seguridad y los procedimientos recomendados. Azure Government usa redes y centros de datos aislados físicamente (ubicados solo en Estados Unidos).

Antes de publicar en [Azure Government](../../azure-government/documentation-government-manage-marketplace-partners.md), pruebe y valide el plan en el entorno, ya que algunos puntos de conexión pueden ser diferentes. Para configurar y probar el plan, solicite una cuenta de prueba desde la página [Prueba de Microsoft Azure Government](https://azure.microsoft.com/global-infrastructure/government/request/).

> [!NOTE]
> Una vez que el plan esté publicado y disponible en una región de Azure específica, no se puede quitar esa región.

#### <a name="azure-government-certifications"></a>Certificaciones de Azure Government

Esta opción solo está visible si ha seleccionado **Azure Government** como la región de Azure en la sección anterior.

Los servicios de Azure Government controlan datos que están sujetos a determinados reglamentos y requisitos gubernamentales. Por ejemplo, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 y CJIS. Para dar a conocer sus certificaciones de estos programas, puede proporcionar hasta 100 vínculos que las describan. Pueden ser vínculos a su anuncio en el programa directamente o a descripciones de su cumplimiento de estos en sus propios sitios web. Estos vínculos solo son visibles para los clientes de Azure Government.

Seleccione **Guardar borrador** antes de continuar.

### <a name="plan-listing"></a>Lista del plan

En esta sección se configuran los detalles de la lista del plan. Este panel muestra información específica que puede variar de otros planes de la misma oferta.

#### <a name="plan-name"></a>Nombre del plan

Este campo se rellena automáticamente con el nombre que asignó al plan cuando lo creó. Este nombre aparece en Azure Marketplace como el título de este plan. Tiene un límite de 100 caracteres.

#### <a name="plan-summary"></a>Resumen del plan

Proporcione un breve resumen del plan, no de la oferta. Este resumen se limita a 100 caracteres.

#### <a name="plan-description"></a>Description del plan

Describa lo que hace que este plan sea único, así como las diferencias entre los planes de la oferta. Describa solo el plan, no la oferta. La descripción del plan puede contener hasta 2 000 caracteres.

Seleccione **Guardar borrador** antes de continuar.

### <a name="pricing-and-availability"></a>Precios y disponibilidad

En este panel, configura:

- Mercados en los que este plan está disponible. Todos los planes debe estar disponible al menos en un [mercado](../marketplace-geo-availability-currencies.md).
- Precio por hora.
- Si desea que el plan sea visible para todos los usuarios o solo para clientes específicos (audiencia privada).

#### <a name="markets"></a>Mercados

Todos los planes debe estar disponible al menos en un mercado. Seleccione la casilla de cada ubicación de mercado en la que este plan debe estar disponible para comprarlo. (Los usuarios de estos mercados pueden seguir implementando la oferta en todas las regiones de Azure seleccionadas en la sección ["Plan setup"](#plan-setup) [Configuración del plan].) El botón **Tax Remitted** (Impuestos remitidos) muestra los países y regiones en los que Microsoft lleva a cabo las ventas y la retención de impuestos en su nombre. La publicación en China se limita a planes que sean *Gratis* o *Traiga su propia licencia* (BYOL).

Si ya se han establecido los precios del plan en dólares de Estados Unidos (USD) y agrega otra ubicación de mercado, el precio del nuevo mercado se calcula en función de los tipos de cambio actuales. Revise siempre el precio de cada mercado antes de la publicación. Revise los precios seleccionando **Export prices (xlsx)** [Exportar precios (xlsx)] después de guardar los cambios.

Al eliminar un mercado, los clientes de ese mercado que usan implementaciones activas no podrán crear nuevas implementaciones ni escalar verticalmente las implementaciones existentes. Esto no afecta a las implementaciones existentes.

#### <a name="pricing"></a>Precios

Para **License model** (Modelo de licencia), seleccione **Usage-based monthly billed plan** (Plan facturado mensual basado en el uso) para configurar los precios de este plan o elija **Traiga su propia licencia** para que los clientes puedan usar este plan con su licencia existente.

Para un plan de facturación mensual en función del uso, utilice una de las tres opciones de entrada de precios siguientes:

- **Por núcleo**: proporcione precios por núcleo en dólares USD. Microsoft calcula los precios por tamaño de núcleo y los convierte a las monedas locales con la tasa de cambio actual.
- **Por tamaño de núcleo**: proporcione precios por tamaño de núcleo en dólares USD. Microsoft calcula los precios y los convierte a las monedas locales con la tasa de cambio actual.
- **Por mercado y tamaño de núcleo**: proporcione precios para cada tamaño de núcleo para todos los mercados. Puede importar los precios desde una hoja de cálculo.

> [!NOTE]
> Guarde los cambios de los precios para habilitar la exportación de los datos de precios. Después de que se publica un precio para un mercado en el plan, no se puede cambiar más adelante. Asegúrese de que los precios sean correctos antes de publicarlos; para ello, exporte la hoja de cálculo de precios y revise los precios en cada mercado.

#### <a name="free-trial"></a>Versión de prueba gratuita

Puede ofrecer una *evaluación gratuita* de un mes o de tres meses a los clientes.

#### <a name="visibility"></a>Visibilidad

Puede diseñar cada plan para que sea visible para todos los usuarios o solo para una audiencia preseleccionada. Asigne la pertenencia a esta audiencia restringida mediante los identificadores de suscripción de Azure.

**Pública**: todos los usuarios pueden ver el plan.

**Audiencia privada**: haga que el plan sea visible solo para una audiencia preseleccionada. Una vez publicado como un plan privado, puede actualizar la audiencia o cambiarlo a público. Después de hacer que un plan sea público, debe seguir siendo público. No se puede volver a cambiar a un plan privado.

> [!NOTE]
> Una audiencia privada o restringida es diferente de la audiencia preliminar que definió en la pestaña **Versión preliminar**. A una audiencia preliminar se le permite acceder a la oferta _antes_ de que se publique en Azure Marketplace. Mientras que la designación de una audiencia privada solo se aplica a un plan específico, la audiencia preliminar puede ver todos los planes privados y públicos con fines de validación.

**Audiencia restringida (identificadores de suscripción de Azure)** : asigne la audiencia que tendrá acceso a este plan privado mediante los identificadores de suscripción de Azure. Opcionalmente, incluya una descripción de cada identificador de suscripción de Azure que haya asignado. Agregue hasta 10 identificadores de suscripción manualmente o hasta 20 000 si importa una hoja de cálculo CSV. Los identificadores de suscripción de Azure se representan como GUID y todas las letras deben estar en minúsculas.

>[!Note]
>Las ofertas privadas no son compatibles con las suscripciones de Azure que se establecen a través de un revendedor del programa Proveedor de soluciones en la nube (CSP).

#### <a name="hide-a-plan"></a>Ocultar un plan

Si la máquina virtual está diseñada para utilizarla solo indirectamente cuando se hace referencia a ella desde otra plantilla de solución o aplicación administrada, seleccione esta casilla para publicar la máquina virtual, pero ocultarla para los clientes que puedan buscarla directamente.

> [!NOTE]
> Los planes ocultos no admiten vínculos de versión preliminar.

Seleccione **Guardar borrador** antes de continuar.

### <a name="technical-configuration"></a>Configuración técnica

Proporcione las imágenes y otras propiedades técnicas que están asociadas al plan. Para más información, consulte el artículo sobre [creación de recursos técnicos para máquinas virtuales de Azure](create-azure-container-technical-assets.md).

> [!NOTE]
> La pestaña **Technical configuration** (Configuración técnica) no se muestra si ha configurado este plan para reutilizar paquetes de otro plan en la pestaña **Plan setup** (Configuración del plan).

#### <a name="operating-system"></a>Sistema operativo

En el panel **Sistema operativo**, haga lo siguiente:

- En **Familia del sistema operativo**, seleccione entre el sistema operativo **Windows** o **Linux**.
- En **Versión** o **Distribuidor**, seleccione la versión de Windows o el distribuidor de Linux.
- En **Nombre descriptivo del sistema operativo**, especifique un nombre descriptivo para el sistema operativo. Este nombre es visible para los clientes.

#### <a name="recommended-vm-sizes"></a>Recommended VM Sizes (Tamaños de máquina virtual recomendados)

Seleccione hasta seis tamaños de máquina virtual recomendados para que se muestren en Azure Marketplace.

#### <a name="open-ports"></a>Abrir puertos

Puertos públicos o privados abiertos en una máquina virtual implementada.

#### <a name="storage-option-for-deployment"></a>Opción de almacenamiento para la implementación

En **Disk deployment option** (Opción de implementación de disco), seleccione el tipo de implementación de disco que pueden utilizar los clientes para la máquina virtual. Microsoft recomienda limitar la implementación solo a la **implementación de discos administrados**.

#### <a name="properties"></a>Propiedades

En **Support Accelerated Networking** (Compatibilidad con redes aceleradas), seleccione esta opción si la máquina virtual es compatible con [redes aceleradas](https://go.microsoft.com/fwlink/?linkid=2124513).

#### <a name="vm-images"></a>Imágenes de VM

Proporcione una versión de disco y el identificador URI de firma de acceso compartido (SAS) para las imágenes de máquina virtual. Agregue hasta 16 discos de datos para cada imagen de máquina virtual. Proporcione solo una nueva versión de imagen por plan en un envío especificado. Una vez publicada una imagen no se puede editar, pero se puede eliminar. La eliminación de una versión impide que los usuarios nuevos y existentes implementen una nueva instancia de la versión eliminada.

- **Versión del disco**: es la versión de la imagen que se va a proporcionar.
- **Identificador URI de SAS**: es la ubicación en la cuenta de Azure Storage en la que ha almacenado el disco duro virtual del sistema operativo. Para saber cómo obtener el URI de firma de acceso compartido, vea [Obtención del URI de firma de acceso compartido para la imagen de máquina virtual](get-sas-uri.md).
- Las imágenes de discos de datos también son identificadores URI de firma de acceso compartido de discos duros virtuales que se almacenan en sus cuentas de Azure Storage.
- Agregue solo una imagen por envío en un plan.

Independientemente del sistema operativo que use, agregue solo el número mínimo de discos de datos que necesite la solución. Durante la implementación, los usuarios no pueden eliminar los discos que forman parte de una imagen, pero siempre pueden agregar discos durante o después de la implementación.

Seleccione **Guardar borrador** antes de continuar y volver a **Plan overview** (Información general del plan).

## <a name="resell-through-csps"></a>Revender mediante los CSP

Amplíe el alcance de la oferta haciendo que esté disponible para los asociados en el programa [Proveedores de soluciones en la nube](https://azure.microsoft.com/offers/ms-azr-0145p/) (CSP). Todos los planes Traiga su propia licencia (BYOL) se incorporan automáticamente al programa. También puede optar por participar en los planes que no sean BYOL.

Seleccione **Guardar borrador** antes de continuar.

## <a name="test-drive"></a>Versión de prueba

Configure una demostración, o *versión de prueba*, que permita a los clientes probar su oferta durante un período de tiempo determinado antes de comprarla. Para crear un entorno de demostración para los clientes, consulte el artículo sobre [ofertas de versión de prueba en el marketplace comercial](test-drive.md).

Para habilitar una versión de prueba, seleccione la casilla **Enable a test drive** (Habilitar una versión de prueba) en el panel **Offer setup** (Configuración de la oferta). Para eliminar la versión de prueba de la oferta, desactive la casilla.

Recursos adicionales de la versión de prueba:

- [Procedimientos recomendados de marketing](../what-is-test-drive.md)
- [Procedimientos técnicos recomendados](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices)
- [Archivo PDF sobre información general de la versión de prueba](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf) (asegúrese de que el bloqueador de elementos emergentes esté desactivado).

Seleccione **Guardar borrador** antes de continuar.

## <a name="review-and-publish"></a>Revisión y publicación

Una vez que haya completado todas las secciones necesarias de la oferta, puede enviarla para su revisión y publicación.

En la esquina superior derecha, seleccione **Review and publish** (Examinar y publicar).

En esta página puede realizar las siguientes acciones:

- Vea el estado de finalización de cada sección de la oferta:

  - **No iniciado**: la sección no se ha iniciado y se debe completar.
  - **Incompletos**: la sección tiene errores que se deben corregir o se requiere que proporcione más información. Para ver instrucciones sobre cómo actualizar la información incompleta, consulte las secciones anteriores de este artículo.
  - **Completo**: la sección está completa y no contiene errores. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- Proporcione **Notes for certification** (Notas para la certificación) al equipo de certificación para asegurarse de que la aplicación se ha probado correctamente. Incluya instrucciones de prueba y cualquier nota complementaria que pueda ayudar al equipo a comprender su aplicación.

Para enviar la oferta para su publicación, seleccione **Revisar y publicar**.

Microsoft le enviará un correo electrónico para avisarle cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación. Para publicar la oferta para la audiencia (o, si es una oferta privada, para una audiencia privada), vaya al Centro de partners, a la página **Información general** de la oferta y seleccione **Publicar**. El estado de la oferta aparece en la parte superior de la página cuando la publicación esté en curso.

### <a name="errors-and-review-feedback"></a>Errores y comentarios de revisión

El paso **Validación manual** del proceso de publicación representa una revisión exhaustiva de la oferta y los recursos técnicos asociados. Si este proceso detecta errores en la oferta, recibirá un informe de certificación que detalla los problemas. Simplemente realice las correcciones necesarias y vuelva a publicar la oferta.

## <a name="offer-overview"></a>Información general de la oferta

En la página **Offer overview** (Información general de la oferta) se muestra una representación visual de los pasos, tanto completados como en curso, necesarios para publicar la oferta, así como el tiempo necesario que debería tardar cada uno en completarse.

Esta página también incluye vínculos para ayudarle a trabajar con la oferta, en función de su estado:

- Si la oferta es un borrador: [Eliminación de una oferta borrador](update-existing-offer.md#delete-a-draft-offer)
- Si la oferta está activa: [Detención de la venta de una oferta o un plan](update-existing-offer.md#stop-selling-an-offer-or-plan)
- Si la oferta está en versión preliminar: [Publicar](publishing-status.md#publisher-approval)
- Si no ha completado el cierre de sesión del anunciante: [Cancelación de la publicación](update-existing-offer.md#cancel-publishing)

## <a name="marketplace-examples"></a>Ejemplos de marketplace

Este es un ejemplo de cómo aparece la información de la oferta en Azure Marketplace:

:::image type="content" source="media/example-azure-marketplace-virtual-machine-offer.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo grande
2. Price
3. Categorías
4. Términos y condiciones
5. Dirección de la directiva de privacidad (vínculo)
6. Nombre de la oferta
7. Descripción
8. Vínculos útiles
9. Capturas de pantallas o vídeos

<br>Este es un ejemplo de cómo aparece la información de la oferta en los resultados de la búsqueda de Azure Marketplace:

:::image type="content" source="media/example-azure-marketplace-virtual-machine-search-results.png" alt-text="Ilustración de cómo aparece esta oferta en los resultados de la búsqueda de Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo pequeño
2. Nombre de la oferta
3. Resumen de los resultados de la búsqueda
4. Versión de prueba

<br>Este es un ejemplo de los detalles del plan de Azure Marketplace:

:::image type="content" source="media/example-azure-marketplace-virtual-machine-plan-details.png" alt-text="Ilustración de los detalles del plan de Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Nombre y resumen del plan
2. Tamaños de máquina virtual recomendados
3. Precios del plan

<br>Este es un ejemplo de cómo aparece la información de la oferta en Azure Portal:

:::image type="content" source="media/example-azure-portal-virtual-machine-offer.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Nombre
2. Descripción
3. Vínculos útiles
4. Capturas de pantallas o vídeos

<br>Este es un ejemplo de cómo aparece la información de la oferta en los resultados de la búsqueda de Azure Portal:

:::image type="content" source="media/example-azure-portal-virtual-machine-search-results.png" alt-text="Ilustración de cómo aparece esta oferta en los resultados de la búsqueda de Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo pequeño
2. Nombre de la oferta
3. Resumen de los resultados de la búsqueda

<br>Este es un ejemplo de los detalles del plan en Azure Portal:

:::image type="content" source="media/example-azure-portal-virtual-machine-plan-details.png" alt-text="Ilustración de los detalles del plan en Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Nombre del plan
2. Description del plan

## <a name="next-step"></a>Paso siguiente

- [Actualización de una oferta existente en Marketplace comercial](update-existing-offer.md)
