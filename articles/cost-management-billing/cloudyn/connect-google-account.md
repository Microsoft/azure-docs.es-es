---
title: Conexión de una cuenta de Google Cloud Platform a Cloudyn en Azure
description: Conecte una cuenta de Google Cloud Platform para ver los datos de uso y costo en los informes de Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 8d549a8fd7c8b03f98fe2b11c94531ed323e0e3a
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690075"
---
# <a name="connect-a-google-cloud-platform-account"></a>Conectar una cuenta de Google Cloud Platform

Puede conectar una cuenta existente de Google Cloud Platform a Cloudyn. Después de conectar una cuenta a Cloudyn, los datos de uso y costo estarán disponibles en los informes de Cloudyn. En este artículo verá cómo configurar y conectar su cuenta de Google con Cloudyn.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="collect-project-information"></a>Recopilación de información del proyecto

Debe comenzar por recopilar información acerca de su proyecto.

1. Inicie sesión en la consola de Google Cloud Platform en [https://console.cloud.google.com](https://console.cloud.google.com).
2. Revise la información del proyecto que quiera incorporar a Cloudyn y anote el **Nombre del proyecto** y el **Id. del proyecto**. Mantenga esta información a mano para los pasos posteriores.  
    ![Nombre del proyecto e identificador de proyecto que se muestra en la consola de Google Cloud Platform](./media/connect-google-account/gcp-console01.png)
3. Si la facturación no está habilitada ni vinculada a su proyecto, cree una cuenta de facturación. Para más información, consulte [Create a new billing account](https://cloud.google.com/billing/docs/how-to/manage-billing-account#create/_a/_new/_billing/_account) (Crear una nueva cuenta de facturación).

## <a name="enable-storage-bucket-billing-export"></a>Habilitar la exportación de facturación de depósitos de almacenamiento

Cloudyn recupera los datos de facturación de Google de un depósito de almacenamiento. Mantenga la información de **Nombre del depósito** y **Prefijo del informe** a mano para usarla más tarde durante el registro de Cloudyn.

Las tarifas del uso de Google Cloud Storage para almacenar informes de uso son muy reducidas. Para más información, consulte [Precios de Cloud Storage](https://cloud.google.com/storage/pricing).

1. Si no ha habilitado la exportación de facturas a un archivo, siga las instrucciones de [Habilitar la exportación de facturación a un archivo](https://cloud.google.com/billing/docs/how-to/export-data-file#how_to_enable_billing_export_to_a_file). Puede usar el formato de exportación de facturas JSON o CSV.
2. En caso contrario, en la consola de Google Cloud Platform, vaya a **Facturación** > **Exportación de la facturación**. Anote el **Nombre del depósito** y el **Prefijo del informe** de la facturación.  
    ![Información de exportación de facturación que se muestra en la página de exportación de facturación](./media/connect-google-account/billing-export.png)

## <a name="enable-google-cloud-platform-apis"></a>Habilite las API de Google Cloud Platform

Para recopilar información de uso y de recursos, Cloudyn necesita que las siguientes API de Google Cloud Platform estén habilitadas:

- BigQuery API
- Google Cloud SQL
- Google Cloud Datastore API
- Google Cloud Storage
- Google Cloud Storage JSON API
- Google Compute Engine API

### <a name="enable-or-verify-apis"></a>Habilitar o comprobar API

1. En la consola de Google Cloud Platform, seleccione el proyecto que quiera registrar en Cloudyn.
2. Vaya a **API y servicios** > **Biblioteca**.
3. Use la búsqueda para encontrar todas las API enumeradas anteriormente.
4. Para cada API, compruebe que se muestra **API habilitada**. De lo contrario, haga clic en **HABILITAR**.

## <a name="add-a-google-cloud-account-to-cloudyn"></a>Adición de una cuenta de Google Cloud a Cloudyn

1. Abra el portal de Cloudyn desde Azure Portal o vaya a [https://azure.cloudyn.com](https://azure.cloudyn.com/) e inicie sesión.
2. Haga clic en **Configuración** (símbolo de engranaje) y, después, seleccione **Cuentas de Cloud**.
3. En **Administración de cuentas**, seleccione la pestaña **Cuentas de Google** y, después, haga clic en **Agregar nuevo +** .
4. En **Nombre de la cuenta de Google**, escriba la dirección de correo electrónico de la cuenta de facturación y, después, haga clic en **Siguiente**.
5. En el cuadro de diálogo de autenticación de Google, seleccione o escriba una cuenta de Google y, después, haga clic en **PERMITIR** para que cloudyn.com pueda acceder a su cuenta.
6. Agregue la información del proyecto que anotó antes. Incluido el **Id. del proyecto**, el nombre del **Proyecto**, el nombre del depósito de **facturación** y el prefijo del informe del **archivo facturación** y, después, haga clic en  **Guardar**.  
    ![Agregar proyecto de Google a la cuenta de Cloudyn](./media/connect-google-account/add-project.png)

Su cuenta de Google aparece en la lista de cuentas y debería aparecer como **Autenticada**. Debajo de esta, el nombre del proyecto de Google y el identificador deberían aparecer y tener un símbolo de marca de verificación verde. El estado de la cuenta debe aparecer como **Completado**.

En unas horas, los informes de Cloudyn muestran información acerca del costo y del uso de Google.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de Cloudyn, consulte el tutorial [Revisión del uso y los costos](tutorial-review-usage.md) de Cloudyn.
