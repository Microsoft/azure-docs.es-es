---
title: Validación funcional de una oferta de AppSource Dynamics 365 Finance and Operations en Azure Marketplace.
description: Procedimientos para validar funcionalmente una oferta de Dynamics 365 Finance and Operations en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: emuench
ms.author: navits
ms.date: 07/17/2020
ms.openlocfilehash: 1b5c756fed5d172593b23c2a157c280caa614940
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095300"
---
# <a name="appsource-dynamics-365-finance-and-operations-functional-validation"></a>Validación funcional de AppSource Dynamics 365 Finance and Operations

Para completar una primera publicación en [Centro de partners](https://partner.microsoft.com/dashboard/home), las ofertas de Dynamics 365 Finance and Operations requieren dos validaciones funcionales:

- Carga de un vídeo de demostración del entorno de Dynamics 365 que muestre la funcionalidad básica.
- Presentación de capturas de pantalla que muestren el entorno de [Lifecycle Services](https://lcs.dynamics.com/) (LCS) de la solución.

> [!NOTE]
> Las publicaciones de recertificación posteriores no requieren demostración. Para más información, consulte el [documento de directivas de AppSource](https://docs.microsoft.com/legal/marketplace/certification-policies#1440-dynamics-365-finance-ops).

## <a name="how-to-validate"></a>Validación

Hay dos opciones para la validación funcional:

- Mantener una conferencia telefónica de 30 minutos con nosotros en horario comercial de la hora estándar del Pacífico (PST) para demostrar y grabar el entorno [LCS](https://lcs.dynamics.com/) y la solución, o bien
- En el Centro de partners, ir a [Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) > **Introducción** y cargar una dirección URL de un vídeo de demostración y capturas de pantalla de LCS en la pestaña Contenido adicional de la oferta.

El equipo de certificación de Microsoft revisa el vídeo y los archivos, y aprueba la solución o le envía un correo electrónico con los pasos a seguir.

### <a name="option-1-30-minute-conference-call"></a>Opción 1: conferencia telefónica de 30 minutos

Para programar una llamada de revisión final, póngase en contacto con [appsourceCRM@microsoft.com](mailto:appsourceCRM@microsoft.com) incluyendo el nombre de la oferta y algunas posibles franjas horarias entre las 8 a.m. y las 5 p.m. hora del Pacífico.

### <a name="option-2-upload-a-demo-video-and-lcs-screenshots"></a>Opción 2: carga de un vídeo de demostración y capturas de pantalla de LCS

1. Grabe un vídeo y cargue la dirección en el sitio de hospedaje de su elección. Siga estas instrucciones:

    - Que sea visible para el equipo de certificación de Microsoft.
    - Que dure menos de 20 minutos.
    - Que incluya hasta tres aspectos destacados de la funcionalidad principal de la solución en el entorno de Dynamics 365.

    > [!NOTE]
    > Puede usar un vídeo de marketing existente si cumple con las directrices.

2. Realice las siguientes capturas de pantalla del entorno [LCS](https://lcs.dynamics.com/) que coincida con la oferta o solución que desee publicar. Tienen que ser suficientemente claras para que el equipo de certificación pueda leer el texto. Guárdelas como archivos JPG. En lugar de proporcionar capturas de pantalla, también puede proporcionar un permiso a [appSourceCRM@microsoft.com](mailto:appSourceCRM@microsoft.com) para acceder a su entorno de LCS y comprobar la configuración.

    1. Vaya a **LCS** > **Modelador de procesos empresariales** > **Project library** (Biblioteca de proyecto). Realice capturas de pantalla de todos los pasos del proceso. Incluya las columnas **Diagramas** y **Revisada**, como se muestra aquí:

       :::image type="content" source="media/dynamics-365-finance-operations/project-library.png" alt-text="Muestra la ventana biblioteca de proyectos.":::

      2. Vaya a **LCS** > **Administración de soluciones** > **Test Solution Package** (Paquete de solución de prueba). Realice capturas de pantallas que incluyan la información general del paquete y el contenido que se muestra en estos ejemplos:

    | Campo | Imagen <img src="" width="400px">|
    | --- | --- |
    | Introducción al paquete | [![Pantalla de información general del paquete](media/dynamics-365-finance-operations/package-overview-45.png)](media/dynamics-365-finance-operations/package-overview.png#lightbox) |
    | <ul><li>Aprobadores de soluciones</li></ul> | [![Pantalla de información general del paquete](media/dynamics-365-finance-operations/solution-approvers-45.png)](media/dynamics-365-finance-operations/solution-approvers.png#lightbox) |
    | Contenido del paquete<ul><li>Modelo</li><li>Paquete de software implementable</li></ul> | [![Pantalla uno de contenido del paquete](media/dynamics-365-finance-operations/package-contents-1-45.png)](media/dynamics-365-finance-operations/package-contents-1.png#lightbox) |
    | <ul><li>Configuración de GER</li><li>Copia de seguridad completa de base de datos</li></ul><br>No se necesitan artefactos en la sección de **configuración de GER**. | [![Pantalla dos de contenido del paquete](media/dynamics-365-finance-operations/package-contents-2-45.png)](media/dynamics-365-finance-operations/package-contents-2.png#lightbox) |
    | <ul><li>Modelo de informe de Power BI</li><li>Artefacto de BPM</li></ul><br>Los artefactos no son necesarios en la sección **Power BI**. | [![Pantalla tres de contenido del paquete](media/dynamics-365-finance-operations/package-contents-3-45.png)](media/dynamics-365-finance-operations/package-contents-3.png#lightbox) |
    | <ul><li>Paquete de datos de proceso</li><li>Contrato de licencia de la solución y directiva de privacidad</li></ul><br>Las secciones **Configuración de GER** y **Modelo de informe de Power BI** son opcionales para incluir en las ofertas de Finance and Operations. | [![Pantalla cuatro de contenido del paquete ](media/dynamics-365-finance-operations/package-contents-4-45.png)](media/dynamics-365-finance-operations/package-contents-4.png#lightbox) |

    Para más información acerca de cada sección del portal de LCS, consulte el [manual del usuario de LCS](https://docs.microsoft.com/dynamics365/fin-ops-core/dev-itpro/lifecycle-services/lcs-user-guide).

3. Realice la carga en el Centro de partners.

    1. Cree un documento de texto que incluya la dirección del vídeo de demostración y las capturas de pantalla, o guarde las capturas de pantalla como archivos JPG independientes.
    2. Agregue el texto y los archivos JPG a un archivo .zip en [Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) del Centro de partners en la pestaña **Contenido adicional** de la oferta de Finance and Operations.

    [![Muestra la ventana de biblioteca del proyecto](media/dynamics-365-finance-operations/supplemental-content.png)](media/dynamics-365-finance-operations/supplemental-content.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de cómo crear una oferta, consulte: [Creación de una oferta de Dynamics 365 for Operations](https://docs.microsoft.com/azure/marketplace/partner-center-portal/create-new-operations-offer).
