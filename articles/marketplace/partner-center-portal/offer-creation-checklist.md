---
title: Lista de comprobación para la creación de ofertas de SaaS en Microsoft Commercial Marketplace
description: Los detalles que puede incluir en el proceso de creación de ofertas de SaaS en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/08/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: 11c1c307d00b9347081a313308ad2467086ec208
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327402"
---
# <a name="saas-offer-creation-checklist-in-partner-center"></a>Lista de comprobación de creación de ofertas de SaaS

El proceso de creación de ofertas de SaaS le lleva a través de varias páginas.  Este artículo describe los detalles que puede proporcionar en cada una de ellas, con vínculos para obtener más información acerca de cada elemento.

> [!NOTE]
> Si va a crear una oferta de SaaS comercializable, asegúrese de implementar la integración con la [API de suministro de SaaS](./pc-saas-fulfillment-apis.md).  La integración con las API es la única manera de que la transacción de Marketplace funcione correctamente. También debe asegurarse de que la aplicación usa la autenticación de Azure AD con el inicio de sesión único (SSO). Consulte [Azure AD y ofertas de SaaS comercializable en el marketplace comercial](../azure-ad-saas.md).

Los elementos que tiene que proporcionar o especificar se detallan a continuación.  Algunas áreas son opcionales o tienen aplicados los valores proporcionados, que puede cambiar según desee.  No es necesario que trabaje en estas secciones en el orden que se muestra aquí.

| **Elemento**    | **Propósito**  |
| :---------- | :-------------------|
| [**Modal de nueva oferta**](#new-offer-modal) | Recopila información sobre la identidad de la oferta.  |
| [Página de configuración de la oferta](#offer-setup-page) | Le permite optar por usar las características clave y elegir cómo vender su oferta a través de Microsoft.  |
| [Página de propiedades](#properties-page) | Defina las categorías y los sectores que se usan para agrupar su oferta en los catálogos de soluciones, los contratos legales que dan soporte a su oferta y su versión de la aplicación. |
| [Página de descripción de la oferta](#offer-listing-page) | Defina los detalles de la oferta que se muestran en Marketplace, lo que incluye descripciones de la oferta y los recursos de marketing.|
| [Página de versión preliminar](#preview-page) | Defina un público preliminar limitado para publicar la oferta antes de ponerla a disposición de un público más amplio de Marketplace.|
| [Página de configuración técnica](#technical-configuration-page)  |  Solo está disponible si selecciona vender la oferta a través de Microsoft.  Defina los detalles técnicos (dirección URL de la página de aterrizaje, dirección URL del webhook de conexión, Id. de inquilino de Azure AD e Id. de la aplicación de Azure AD) que usa Marketplace para conectarse a la oferta.  Estos parámetros son necesarios para integrarse correctamente con la realización de SaaS y las API de facturación según el uso de Marketplace.|
| [**Modal de nuevo plan**](#plan-identity-modal) | Recopila información sobre la identidad del plan.  |
| [Página de descripción del plan](#plan-listing-page)  | Solo está disponible si selecciona vender la oferta a través de Microsoft. Defina los detalles usados para mostrar el plan en Marketplace.  |
| [Página de precios y disponibilidad del plan](#plan-pricing--availability-page)  | Solo está disponible si selecciona vender la oferta a través de Microsoft.  Recopila las características empresariales (modelo de precios), el público y la disponibilidad de mercado para cada plan (versión) de su oferta.  |
| [Página de descripción de la versión de prueba](#test-drive-listing-page)  | Solo disponible si selecciona ofrecer una versión de prueba de la oferta. Defina los detalles usados para mostrar la versión de prueba en Marketplace.  |
| Página de configuración técnica de la versión de prueba  | Solo disponible si selecciona ofrecer una versión de prueba de la oferta. Define los detalles técnicos de la demostración (o "versión de prueba") que permitirá a los clientes probar su oferta antes de comprarla.  |
| [Página de revisión y publicación](#review-and-publish-page)  | Seleccione los cambios que desee publicar, vea el estado de cada página y proporcione notas al equipo de certificación.  |

## <a name="new-offer-modal"></a>Modal de nueva oferta

Los primeros datos que tendrá que proporcionar son un identificador y un alias para la oferta.

| **Nombre del campo**    | **Notas**   |  
| :---------------- | :-----------| 
| Id. de oferta  | Requerido, no se puede cambiar después de la creación. Debe tener 50 caracteres como máximo y constar únicamente de caracteres alfanuméricos en minúscula, guiones o caracteres de subrayado. |
| Alias de la oferta  | Necesario. |

## <a name="offer-setup-page"></a>Página de configuración de la oferta

La página de configuración de la oferta es donde puede optar por diferentes canales y movimientos de ventas, así como declarar el uso de características clave, como la versión de prueba y los clientes potenciales. 

| **Nombre del campo**    | **Notas**   | 
| :---------------- | :-----------|  
| ¿Desea vender a través de Microsoft?  | Necesario. Valor predeterminado: Sí |
| ¿Cómo desea que los clientes potenciales interactúen con esta oferta publicada? (Llamada a la acción)  | Requerido si no se vende a través de Microsoft. Valor predeterminado: Evaluación gratuita, opciones: "Obtenerla ahora", "Evaluación gratuita", "Ponerse en contacto conmigo". |
| Dirección URL de la versión de evaluación  | Requerido si se selecciona "Evaluación gratuita" como el modo en que los clientes deben interactuar con la oferta publicada. |
| Dirección URL de la oferta  | Requerido si se selecciona "Obtenerla ahora" como el modo en que los clientes deben interactuar con la oferta publicada. |
| Canales  | Opcional. Valor predeterminado: No habilitado para el canal CSP (revendedor).  |
| Versión de prueba | Opcional. Valor predeterminado: No hay ninguna versión de prueba habilitada.  |
| Tipo de versión de prueba | Requerido si se habilita una versión de prueba. Valor predeterminado: Ninguno seleccionado. Opciones: Azure Resource Manager, Dynamics 365 for Business Central, Dynamics 365 for Customer Engagement, Dynamics 365 for Operations, Logic Apps, Power BI.  |
| Clientes potenciales: conexión a un sistema CRM | Requerido si se vende a través de Microsoft o si la oferta incluye "Ponerse en contacto conmigo". Valor predeterminado: ningún sistema CRM conectado. Opciones de CRM: tabla de Azure, blob de Azure, Dynamics CRM Online, punto de conexión HTTPS, Marketo, Salesforce  |

## <a name="properties-page"></a>Página de propiedades

La pestaña de propiedades es donde define las categorías y los sectores que se usan para agrupar su oferta en los catálogos de soluciones, los contratos legales que dan soporte a su oferta y su versión de la aplicación. Asegúrese de proporcionar detalles completos y precisos sobre la oferta en esta página para que se muestre correctamente y se ofrezca al conjunto correcto de clientes. 

| **Nombre del campo**    | **Notas**   | 
| :---------------- | :-----------|  
| Categoría y subcategoría | Requerido 1 y máximo 3. Valor predeterminado: Ninguno seleccionado. |
| Sectores y subsectores | Opcional. Máximo de 2 sectores L1 y máximo de 2 subsectores dentro de cada sector L1, valor predeterminado: Ninguno seleccionado |
| Versión de la aplicación  | Opcional. Valor predeterminado: Ninguno. |
| Usar el Contrato estándar  | Opcional. Valor predeterminado: no seleccionado.  | |
| Términos de uso  | Requerido si el Contrato estándar no está seleccionado.  |

## <a name="offer-listing-page"></a>Página de descripción de la oferta

La página de descripción es donde debe proporcionar el texto y las imágenes que los clientes ven cuando consultan la oferta en Marketplace. 

| **Nombre del campo**    | **Notas**   |
| :---------------- | :-----------| 
| Nombre  | Requerido, máx. 50 caracteres. |
| Resumen  | Requerido, máx. 100 caracteres. | 
| Descripción  | Requerido, máx. 3000 caracteres. |
| Instrucciones de introducción  | Requerido, máx. 3000 caracteres. |
| Instrucciones de introducción  | Requerido, máx. 3000 caracteres. |
| Palabras clave de búsqueda  | Opcional, recomendado, máx. 3 palabras clave. |
| Dirección URL de la directiva de privacidad  | Necesario. |
| Dirección URL de materiales de marketing del programa CSP  | Opcional. |
| Título + dirección URL de vínculos útiles  | Opcional. |
| Título + archivo de documentos de apoyo  | Requerido, mín. 1 y máx. 3. Debe estar en formato de archivo PDF. |
| Capturas de pantalla  | Requerido, mín. 1 captura de pantalla y máx. 5; se recomienda 4 o más. Debe ser 1280 x 720 en formato PNG. |
| Logotipos del almacén (pequeño, mediano, grande, ancho)  | Se requiere pequeño (48 x 48) y grande (216 x 216); otros tamaños opcionales pero recomendados: Mediano (90 x 90), ancho (255 x 115). Debe tener el formato .PNG. |
| Nombre + dirección URL + miniatura de vídeos  | Opcional, recomendado, máx. 4 vídeos. La miniatura debe tener el tamaño 1280 x 720 y estar en formato PNG. El vídeo debe estar hospedado en YouTube o Vimeo. |
| Contactos (programa CSP, ingeniería, soporte técnico)  | Se requiere el contacto de ingeniería y soporte técnico (nombre, correo electrónico y número de teléfono); el contacto del programa CSP es opcional pero está recomendado. |
| Dirección URL de soporte técnico  | Necesario. |

## <a name="preview-page"></a>Página de versión preliminar

La página de versión preliminar es donde se especifica el público que tendrá acceso a la versión preliminar de su oferta, para verificar que la oferta cumple con todos sus requisitos antes de que se publique. 

| **Nombre del campo**    | **Notas**   | 
| :---------------- | :-----------| 
| Correo electrónico + descripción de AAD/MSA | Requerido, mín. 1 y máx. 10 si se especifica manualmente, o hasta 20 si carga un archivo CSV. |

## <a name="technical-configuration-page"></a>Página de configuración técnica 

La página de configuración técnica es donde especifica los detalles técnicos usados por Microsoft para conectarse a su oferta. No verá esta página si ha decidido no vender a través de Microsoft.

> [!NOTE]
> En el caso de las ofertas comercializables, debe crear una página de aterrizaje y la aplicación debe usar la autenticación de Azure AD con el inicio de sesión único (SSO). Para más información, consulte [Azure AD y ofertas de SaaS comercializable en el marketplace comercial](../azure-ad-saas.md).

| **Nombre del campo**    | **Notas**   |  
| :---------------- | :-----------| 
| URL de página de aterrizaje | Requerido si se vende a través de Microsoft. |
| Webhook de conexión | Requerido si se vende a través de Microsoft. |
| Identificador de inquilino de Azure AD | Requerido si se vende a través de Microsoft. |
| Identificador de aplicación de Azure AD | Requerido si se vende a través de Microsoft. |

## <a name="plan-identity-modal"></a>Modal de identidad de plan

Los primeros datos que tendrá que proporcionar son un nombre y un identificador para el plan. No verá esta página si ha decidido no vender a través de Microsoft.

| **Nombre del campo**    | **Notas**   |  
| :---------------- | :-----------| 
| Identificador de plan  | Requerido si se vende a través de Microsoft. No se puede cambiar después de la creación. Debe tener 50 caracteres como máximo y constar únicamente de caracteres alfanuméricos en minúscula, guiones o caracteres de subrayado. |
| Plan Name  | Requerido si se vende a través de Microsoft. Debe ser único en todos los planes de la oferta. Máx. 50 caracteres. |

## <a name="plan-listing-page"></a>Página de descripción del plan

La página de descripción del plan es donde proporciona el texto que verán los clientes al consultar el plan en el Marketplace. No verá esta página si ha decidido no vender a través de Microsoft.

| **Nombre del campo**    | **Notas**   |  
| :---------------- | :-----------| 
| Description del plan   | Requerido si se vende a través de Microsoft. Max. 500 caracteres. | |

## <a name="plan-pricing--availability-page"></a>Página de precios y disponibilidad del plan

La página de precios y disponibilidad del plan es donde define las características empresariales, el público y la disponibilidad de mercado para cada plan (versión) de su oferta. No verá esta página si ha decidido no vender a través de Microsoft.

| **Nombre del campo**    | **Notas**   | 
| :---------------- | :-----------| 
| Disponibilidad de mercado  | Requerido, mín. 1 y máx. 141. |
| Modelo de precios  | Necesario. Valor predeterminado: Tarifa plana. Opciones: Tarifa plana, por usuario. |
| Puestos mínimo y máximo  | Opcional, solo disponible si se selecciona el modelo de precios basado en puestos. |
| Período de facturación  | Necesario. Valor predeterminado: Mensual. Opciones: Mensual, anual. |
| Price  | USD requeridos al mes, si se selecciona el período de facturación mensual; o USD al año si se selecciona el período de facturación anual. |
| Público del plan  | Opcional. Valor predeterminado: Plan público. Opciones: Público, privado por identificador de inquilino |
| Público del plan restringido (identificador de inquilino + descripción)  | Requerido si se selecciona un plan privado. Mín.1 y máx. 10 identificadores de inquilino si introducen manualmente. Máximo 20 000 si se importa un archivo .csv. |

## <a name="test-drive-listing-page"></a>Página de descripción de la versión de prueba

Solo disponible si selecciona ofrecer una versión de prueba de la oferta. Defina los detalles usados para mostrar la versión de prueba en Marketplace.

| **Nombre del campo**    | **Notas**   | 
| :---------------- | :-----------| 
| Descripción  | Necesario. |
| Nombre + archivo del manual del usuario  | Requerido, máx. 1 documento. Debe estar en formato PDF. |
| Nombre + dirección URL + miniatura de vídeo  | Opcional, recomendado. La miniatura debe tener el tamaño 533 x 324 y estar en formato JPGP o PNG. El vídeo debe estar hospedado en YouTube o Vimeo. |

## <a name="review-and-publish-page"></a>Página de revisión y publicación

| **Nombre del campo**    | **Notas**   | 
| :---------------- | :-----------| 
| Notas para certificación  | Opcional. |

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta SaaS](./create-new-saas-offer.md)
