---
title: Creación e implementación de un servicio en la nube | Microsoft Docs
description: Aprenda cómo usar el método Creación rápida para crear un servicio en la nube y usar Cargar para cargar e implementar un paquete de servicios en la nube en Azure.
services: cloud-services
documentationcenter: ''
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 05/18/2017
ms.author: tagore
ms.openlocfilehash: fca14ed4e3b338f9b49f949af7498dfd49990d3b
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142390"
---
# <a name="how-to-create-and-deploy-a-cloud-service"></a>Creación e implementación de un servicio en la nube
Azure Portal le ofrece dos formas de crear e implementar un servicio en la nube: *Creación rápida* y *Creación personalizada*.

En este artículo se explica cómo usar el método Creación rápida para crear un nuevo servicio en la nube y, a continuación, cómo usar **Cargar** para cargar e implementar un paquete de servicios en la nube en Azure. Cuando usa este método, el Portal de Azure pone a su disposición los vínculos pertinentes para completar todos los requisitos que vaya necesitando sobre la marcha. Si está listo para implementar su servicio en la nube una vez creado, puede hacer las dos cosas a la vez usando Creación personalizada.

> [!NOTE]
> Si tiene pensado publicar su servicio en la nube desde Azure DevOps, use Creación rápida y después configure la publicación Azure DevOps desde Creación rápida de Azure o en el panel. Para más información, consulte [Entrega continua en Azure con Azure DevOps][TFSTutorialForCloudService] o la ayuda de la página **Inicio rápido**.
>
>

## <a name="concepts"></a>Conceptos
Necesita tres componentes para implementar una aplicación como servicio en la nube en Azure:

* **Definición de servicio**  
  El archivo de definición de servicio en la nube (.csdef) define el modelo de servicio, incluyendo el número de roles.
* **Configuración de servicio**  
  El archivo de configuración de servicio en la nube (.cscfg) proporciona opciones de configuración para los roles de servicio en la nube e individuales, incluyendo el número de instancias de rol.
* **Paquete de servicio**  
  El paquete de servicio (.cspkg) contiene el código y las configuraciones de la aplicación y el archivo de definición de servicio.

Puede obtener más información acerca de estas y cómo crear un paquete [aquí](cloud-services-model-and-package.md).

## <a name="prepare-your-app"></a>Preparación de la aplicación
Antes de implementar un servicio en la nube, debe crear el paquete de servicio en la nube (.cspkg) desde su código de aplicación y un archivo de configuración de servicio en la nube (.cscfg). El SDK de Azure proporciona herramientas para preparar estos archivos de implementación necesarios. Puede instalar el SDK desde la página [Descargas de Azure](https://azure.microsoft.com/downloads/) , en el idioma en que prefiera implementar su código de aplicación.

Hay tres características del servicio en la nube que requieren configuraciones especiales antes de exportar un paquete de servicio:

* Si desea implementar un servicio en la nube que utilice Seguridad de la capa de transporte (TLS), anteriormente conocido como Capa de sockets seguros (SSL) para el cifrado de datos, [configure la aplicación](cloud-services-configure-ssl-certificate-portal.md#modify) para TLS.
* Si desea configurar una conexión de Escritorio remoto a instancias de rol, [configure los roles](cloud-services-role-enable-remote-desktop-new-portal.md) para Escritorio remoto.
* Si desea configurar una supervisión exhaustiva para su servicio en la nube, habilite Diagnósticos de Azure para el servicio en la nube. *Supervisión mínima* (nivel predeterminado de supervisión) usa contadores de rendimiento recopilados de los sistemas operativos host para las instancias de rol (máquinas virtuales). *supervisión detallada* recopila métricas adicionales basadas en los datos del rendimiento dentro de las instancias de rol para permitir un análisis más profundo de los problemas que se producen durante el procesamiento de la aplicación. Para obtener información acerca de cómo habilitar Diagnósticos de Azure, consulte [Habilitación de Diagnósticos en Azure](cloud-services-dotnet-diagnostics.md).

Para crear un servicio en la nube con implementaciones de roles web o de trabajo, debe [crear el paquete de servicio](cloud-services-model-and-package.md#servicepackagecspkg).

## <a name="before-you-begin"></a>Antes de empezar
* Si no ha instalado el SDK de Azure, haga clic en **Instalación de SDK de Azure** para abrir la [página Descargas de Azure](https://azure.microsoft.com/downloads/)y, a continuación, descargue el SDK en el idioma en que prefiera desarrollar el código. (Tendrá la oportunidad de hacer esto más tarde).
* Si hay instancias de rol que necesitan certificados, créelos. Los servicios en la nube requieren un archivo .pfx con una clave privada. Puede cargar los certificados en Azure al crear e implementar el servicio en la nube.

## <a name="create-and-deploy"></a>Creación e implementación
1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Haga clic en **Crear un recurso > Proceso**, luego desplácese hacia abajo y haga clic en **Servicio en la nube**.

    ![Publicación del servicio en la nube](media/cloud-services-how-to-create-deploy-portal/create-cloud-service.png)
3. En la nuevo panel **Servicio en la nube**, escriba un valor para **Nombre DNS**.
4. Cree un nuevo **Grupo de recursos** o seleccione uno existente.
5. Seleccione una **ubicación**.
6. Haga clic en **Paquete**. Se abre el panel **Cargar un paquete**. Rellene todos los campos obligatorios. Si cualquiera de los roles contiene una sola instancia, asegúrese de que la casilla **Implementar aunque uno o varios roles contengan una sola instancia** esté seleccionada.
7. Asegúrese de que la opción **Iniciar implementación** esté seleccionada.
8. Haga clic en **Aceptar** para cerrar el panel **Cargar un paquete**.
9. Si no tiene ningún certificado para agregar, haga clic en **Crear**.

    ![Publicación del servicio en la nube](media/cloud-services-how-to-create-deploy-portal/select-package.png)

## <a name="upload-a-certificate"></a>Carga de un certificado
Si el paquete de implementación se [configuró para usar certificados](cloud-services-configure-ssl-certificate-portal.md#modify), puede cargar el certificado ahora.

1. Seleccione **Certificados** y, en el panel **Agregar certificados**, seleccione el archivo .pfx del certificado TLS/SSL y especifique la **contraseña** del certificado,
2. Haga clic en **Adjuntar certificado** y luego en **Aceptar** en el panel **Agregar certificados**.
3. Haga clic en **Crear** en el panel **Servicio en la nube**. Cuando la implementación haya llegado al estado **Listo** , puede continuar con los pasos siguientes.

    ![Publicación del servicio en la nube](media/cloud-services-how-to-create-deploy-portal/attach-cert.png)

## <a name="verify-your-deployment-completed-successfully"></a>Compruebe que la implementación se haya completado correctamente.
1. Haga clic en la instancia de servicio en la nube.

    El estado debería mostrar que el servicio está **En ejecución**.
2. En **Essentials**, haga clic en la **URL del sitio** para abrir el servicio en la nube en un explorador web.

    ![CloudServices_QuickGlance](./media/cloud-services-how-to-create-deploy-portal/running.png)

[TFSTutorialForCloudService]: https://go.microsoft.com/fwlink/?LinkID=251796

## <a name="next-steps"></a>Pasos siguientes
* [Configuración general de su servicio en la nube](cloud-services-how-to-configure-portal.md).
* Configuración de un [nombre de dominio personalizado](cloud-services-custom-domain-name-portal.md).
* [Administración del servicio en la nube](cloud-services-how-to-manage-portal.md).
* Configure los [certificados TLS/SSL](cloud-services-configure-ssl-certificate-portal.md).



