---
title: Solución de problemas de la extensión del Panel de acceso de Azure para Internet Explorer | Microsoft Docs
description: Cómo usar la directiva de grupo para implementar el complemento de Internet Explorer para el portal de Mis aplicaciones.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/11/2019
ms.author: mimart
ms.reviewer: asteen
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: f08a02842c97b0f4076a1b311aa918df6d83c592
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824502"
---
# <a name="troubleshoot-the-access-panel-extension-for-internet-explorer"></a>Solución de problemas de la extensión del Panel de acceso para Internet Explorer

Este artículo le ayudará a solucionar los siguientes problemas:

* No puede tener acceso a sus aplicaciones a través del portal de Mis aplicaciones con Internet Explorer.
* Verá el mensaje "Instalar software" aunque ya haya instalado el software.

Si es un administrador, consulte [cómo implementar la extensión del Panel de acceso para Internet Explorer mediante la directiva de grupo](deploy-access-panel-browser-extension.md).

## <a name="run-the-diagnostic-tool"></a>Ejecute la herramienta de diagnóstico

Puede diagnosticar problemas de instalación con la extensión del Panel de acceso descargando y ejecutando la herramienta de diagnóstico del Panel de acceso. 

Para descargar e instalar la herramienta de diagnóstico:

1. [Seleccione este vínculo para descargar la herramienta de diagnóstico.](https://account.activedirectory.windowsazure.com/applications/AccessPanelExtensionDiagnosticTool/AccessPanelExtensionDiagnosticTool.zip)

2. Abra el archivo y extraiga el contenido en el equipo.
   
3. Para ejecutar la herramienta, haga clic en el archivo denominado *AccessPanelExtensionDiagnosticTool.js* y seleccione **abrir con** > **Microsoft Windows Script Host de** .
   
    ![Abrir con > Host de script basado en Microsoft Windows](./media/manage-access-panel-browser-extension/open-access-panel-extension-diagnostic-tool.png)

4. Revise los resultados de diagnóstico que aparecen y seleccione **Sí** para corregir los problemas. El **comprobar resultados** aparece el cuadro de diálogo con información sobre qué hacer si la extensión no funciona.  

5. Lea el mensaje y seleccione **Aceptar**.

## <a name="check-that-the-access-panel-extension-is-enabled"></a>Comprobar que la extensión del Panel de acceso está activada

Para comprobar que ha habilitado la extensión del Panel de acceso en Internet Explorer:

1. En Internet Explorer, seleccione el **icono de engranaje** en la esquina superior derecha de la ventana y seleccione **opciones de Internet**.
   
2. Vaya a la **programas** pestaña y seleccione **administrar complementos**.
   
3. Seleccione **extensión Access Panel** en el **Microsoft Corporation** sección y seleccione **habilitar**.
   
4. Para guardar los cambios, cierre todas las ventanas del explorador de Internet Explorer ha abierto. El cambio surte efecto la próxima vez que abra Internet Explorer.

## <a name="enable-extensions-for-inprivate-browsing"></a>Habilitar las extensiones para la exploración de InPrivate

Para habilitar las extensiones para la exploración de InPrivate:

1. En Internet Explorer, seleccione el **icono de engranaje** en la esquina superior derecha de la ventana y seleccione **opciones de Internet**.
   
2. Vaya a la **privacidad** ficha y compruebe que la **deshabilitar barras de herramientas y extensiones cuando se inicie la exploración InPrivate** casilla está desactivada.
   
3.  Para guardar los cambios, cierre todas las ventanas del explorador de Internet Explorer ha abierto. El cambio surte efecto la próxima vez que abra Internet Explorer.

## <a name="uninstall-the-access-panel-extension"></a>Desinstalar la extensión del Panel de acceso

Para desinstalar la extensión del Panel de acceso desde el equipo:

1. En el Panel de Control, busque *desinstalar*. 

2. En los resultados de búsqueda, seleccione **desinstalar un programa**.
   
    ![Busque el programa de desinstalación.](./media/manage-access-panel-browser-extension/uninstall-program-control-panel.png)

3. En la lista, seleccione **extensión Access Panel** y seleccione **desinstalar**.

    ![Desinstale la extensión del Panel de acceso.](./media/manage-access-panel-browser-extension/uninstall-access-panel-extension.png)
   
4. Después, puede intentar instalar la extensión de nuevo para ver si se ha resuelto el problema.

Si experimenta problemas al desinstalar la extensión, también puede quitar mediante el [Microsoft Fix It](https://go.microsoft.com/?linkid=9779673) herramienta.

## <a name="related-articles"></a>Artículos relacionados
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](what-is-single-sign-on.md)
* [Cómo implementar la extensión del Panel de acceso para Internet Explorer mediante la directiva de grupo](deploy-access-panel-browser-extension.md)

