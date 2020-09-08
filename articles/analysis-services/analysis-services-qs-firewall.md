---
title: 'Guía de inicio rápido: Configuración de un firewall de servidor de Azure Analysis Services | Microsoft Docs'
description: Esta inicio rápido le ayudará a configurar un firewall para el servidor de Azure Analysis Services mediante Azure Portal.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: quickstart
ms.date: 08/12/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: e4953137cf939c35c6ac73fe51ca43eca6e99edc
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192443"
---
# <a name="quickstart-configure-server-firewall---portal"></a>Inicio rápido: Configuración del firewall del servidor: Portal

Esta guía de inicio rápido le ayudará a configurar un firewall para el servidor de Azure Analysis Services. La habilitación de un firewall y la configuración de los intervalos de direcciones IP solo para los equipos que tienen acceso al servidor son una parte importante de la protección del servidor y los datos.

## <a name="prerequisites"></a>Prerrequisitos

- Un servidor de Analysis Services en la suscripción. Para más información, vea esta [Inicio rápido: Creación de un servidor (Portal)](analysis-services-create-server.md) o [Inicio rápido: Creación de un servidor (PowerShell)](analysis-services-create-powershell.md)
- Uno o más intervalos de direcciones IP para los equipos cliente (si es necesario).

> [!NOTE]
> La importación de datos (actualización) y las conexiones de informes paginados de Power BI Premium en Microsoft Cloud Germany no se admiten actualmente cuando está habilitado un firewall, incluso si la opción Allow access from Power BI (Permitir el acceso desde Power BI) está activada.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal 

[Iniciar sesión en el portal](https://portal.azure.com)

## <a name="configure-a-firewall"></a>Configuración de un firewall

1. Haga clic en el servidor para abrir la página Información general. 
2. En **Configuración** > **Firewall** > **Habilitar firewall**, seleccione **Activar**.
3. Para habilitar las conexiones desde Power BI y Power BI Premium, en **Allow access from Power BI** (Permitir el acceso desde Power BI), seleccione **Activar**.  
4. (Opcional) Especifique uno o más intervalos de direcciones IP. Escriba un nombre y una dirección IP inicial y final para cada intervalo. El nombre de la regla de firewall debe estar limitado a 128 caracteres y solo puede contener caracteres en mayúsculas, caracteres en minúsculas, números, caracteres de subrayado y guiones. No se permiten espacios en blanco ni otros caracteres especiales.
5. Haga clic en **Save**(Guardar).

     ![Configuración de firewall](./media/analysis-services-qs-firewall/aas-qs-firewall.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine intervalos de direcciones IP o deshabilite el firewall.

## <a name="next-steps"></a>Pasos siguientes
En esta guía de inicio rápido, ha aprendido a configurar un firewall para el servidor. Ahora que tiene el servidor y está protegido con un firewall, puede agregar un modelo de datos de ejemplo básico desde el portal. Tener un modelo de ejemplo es útil para aprender cómo configurar los roles de la base de datos del modelo y probar las conexiones de cliente. Para más información, puede seguir el tutorial para agregar un modelo de ejemplo.

> [!div class="nextstepaction"]
> [Tutorial: Incorporación de un modelo de ejemplo al servidor](analysis-services-create-sample-model.md)
