---
title: Microsoft Threat Modeling Tool, versión 9/12/2018
titleSuffix: Azure
description: Lea las notas de la versión de Microsoft Threat Modeling Tool publicadas el 12/09/2018. Las notas incluyen cambios de características y correcciones de errores.
author: jegeib
ms.author: jegeib
ms.service: security
ms.subservice: security-develop
ms.topic: article
ms.date: 01/15/2019
ms.openlocfilehash: 780ddc614feef492dafa54862abbea9552496274
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87539822"
---
# <a name="threat-modeling-tool-ga-release-71509112---9122018"></a>Threat Modeling Tool, versión de disponibilidad general 7.1.50911.2: 12 de septiembre de 2018

Nos complace anunciar que Microsoft Threat Modeling Tool ya está disponible para descargar como versión admitida disponible con carácter general (GA). Esta versión contiene importantes actualizaciones de privacidad y seguridad, así como correcciones de errores, actualizaciones de características y mejoras de estabilidad. Los usuarios existentes de la versión preliminar de 2017 recibirán la solicitud de actualizarse a la versión más reciente mediante la tecnología ClickOnce al abrir al cliente. Para los nuevos usuarios de la herramienta, puede [descargar el cliente](https://aka.ms/threatmodelingtool).

Con esta versión, se da por finalizado el soporte técnico de la versión preliminar de 2017 y se recomienda a todos los usuarios de dicha versión actualizarse a la versión de disponibilidad general (GA). El 15 de octubre, o con posterioridad a esta fecha, estableceremos la versión mínima de ClickOnce necesaria a partir de la cual Threat Modeling Tool y todos los clientes de la versión preliminar deberán actualizarse.

Microsoft Threat Modeling Tool 2016, que está disponible en el [Centro de descarga de Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=49168), seguirá recibiendo soporte técnico hasta el 1 de octubre de 2019 solo en lo que respecta a errores importantes relacionados con la seguridad.

## <a name="feature-changes"></a>Cambios de características

### <a name="azure-stencil-updates"></a>Actualizaciones de la galería de símbolos de Azure

Se han agregado galerías de símbolos de Azure adicionales, y sus amenazas y mitigaciones asociadas, al conjunto de galerías de símbolos que acompañan esta versión. Se realizaron cambios importantes en las áreas de enfoque de "Azure App Services", "Ofertas de base de datos de Azure" y "Azure Storage".

![Actualizaciones de la galería de símbolos de Azure](./media/threat-modeling-tool-releases-71509112/tmt_azure_stencil_update-300x70.png)

### <a name="onedrive-integration-feature-removed"></a>Eliminada la característica de integración de OneDrive

Se han quitado las características "Guardar en OneDrive", "Abrir desde OneDrive" y "Compartir un vínculo" de la versión preliminar. Se recomienda a los usuarios de OneDrive usar el cliente [OneDrive para Windows](https://onedrive.live.com/about/en-us/download/) de Microsoft para acceder a sus archivos almacenados en OneDrive mediante los cuadros de diálogo estándar para guardar y abrir archivos.

## <a name="notable-fixed-bugs-reported-by-customers"></a>Corrección de errores destacados notificados por los clientes

### <a name="in-tmt-preview-the-tool-crashes-when-using-the-standard-template"></a>En la versión preliminar de TMT, la herramienta se bloquea cuando se usa la plantilla estándar

- Cuando se agrega una galería de símbolos genérica (por ejemplo, "Generic Data Flow" [Flujo de datos genérico]) a la superficie de dibujo y se generan amenazas, la herramienta puede bloquearse. Ahora se ha corregido.

### <a name="in-tmt-preview-when-i-save-a-report-or-copy-the-threats-the-risk-levels-are-incorrect"></a>En la versión preliminar de TMT, al guardar un informe o copiar las amenazas, los niveles de riesgo son incorrectos

- Si un usuario modifica el nivel de riesgo de amenazas específicas y, luego, guarda un informe o copia los riesgos, el nivel de riesgo puede revertirse a "High" (Alto). Ahora se ha corregido.

## <a name="known-issues-and-faq"></a>Problemas conocidos y preguntas más frecuentes

### <a name="users-of-high-resolution-screens-may-experience-small-text-in-the-threat-properties"></a>Los usuarios de pantallas de alta resolución pueden experimentar un tamaño de texto pequeño en las propiedades de la amenaza

#### <a name="issue"></a>Problema

En la vista de análisis de la herramienta, si el usuario tiene una pantalla de alta resolución que está establecida de forma predeterminada para aumentar el tamaño de la letra para mejorar la legibilidad en Windows, la sección "Possible Mitigation(s)" (Mitigaciones posibles) de una amenaza puede aparecer con texto pequeño.

![Problema conocido con pantallas de alta resolución](./media/threat-modeling-tool-releases-71509112/tmt_screen_resolution-300x153.png)

#### <a name="workaround"></a>Solución alternativa

El usuario puede hacer clic en el texto de mitigación y usar el control de zoom estándar de Windows (Crtl-Rueda arriba del mouse) para ampliar esa sección.

### <a name="files-in-the-recently-opened-models-section-of-the-main-window-may-fail-to-open"></a>Los archivos de la sección "Recently Opened Models" (Modelos abiertos recientemente) de la ventana principal tienen problemas para abrirse

#### <a name="issue"></a>Problema

Se quitó la característica "Abrir desde OneDrive" de la versión preliminar. Los usuarios con "Recently Opened Models" (Modelos abiertos recientemente) que se guardaron en OneDrive recibirán el siguiente error.

![Eliminada característica de OneDrive](./media/threat-modeling-tool-releases-71509112/tmt_save_error-300x131.png)

#### <a name="workaround"></a>Solución alternativa

Se recomienda a los usuarios de OneDrive usar el cliente [OneDrive para Windows](https://onedrive.live.com/about/en-us/download/) de Microsoft para acceder a sus archivos almacenados en OneDrive mediante el cuadro de diálogo estándar para abrir un modelo.

![Eliminada característica de OneDrive](./media/threat-modeling-tool-releases-71509112/tmt_save_onedrive-300x149.png)

### <a name="my-organization-uses-the-2016-version-of-the-tool-can-i-use-the-azure-stencil-set"></a>Mi organización usa la versión 2016 de la herramienta, ¿puedo usar el conjunto de galerías de símbolos de Azure?

Sí, puede hacerlo. El [conjunto de galerías de símbolos de Azure está disponible en GitHub](https://github.com/Microsoft/threat-modeling-templates/) y se puede cargar en la versión 2016 de la herramienta. Para crear un nuevo modelo con el conjunto de galerías de símbolos de Azure, use el cuadro de diálogo "Template For New Models" (Plantilla para nuevos modelos) en la pantalla del menú principal. TMT 2016 no puede representar los vínculos que se encuentra en los campos de posibles mitigaciones del conjunto de galerías de símbolos de Azure; por lo tanto, puede que vea que se muestran como etiquetas HTML.

![Actualizaciones de la galería de símbolos de Azure en el cliente de 2016](./media/threat-modeling-tool-releases-71509112/tmt_azure_stencils-300x212.png)

## <a name="system-requirements"></a>Requisitos del sistema

- Sistemas operativos compatibles
  - Microsoft Windows 10
- Versión de .NET necesaria
  - .NET 3.5.2
- Requisitos adicionales
  - Se necesita una conexión a Internet para recibir actualizaciones de la herramienta, así como plantillas.

## <a name="documentation-and-feedback"></a>Documentación y comentarios

- La documentación de Threat Modeling Tool se encuentra en [docs.microsoft.com](threat-modeling-tool.md) e incluye información [sobre el uso de la herramienta](threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Pasos siguientes

Descargue la versión más reciente de [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).
