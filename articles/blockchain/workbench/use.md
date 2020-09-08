---
title: Uso de aplicaciones en Azure Blockchain Workbench
description: Tutorial sobre cómo usar contratos de aplicación en la versión preliminar de Azure Blockchain Workbench.
ms.date: 10/14/2019
ms.topic: tutorial
ms.reviewer: brendal
ms.openlocfilehash: 5761bf1294691c2d50e0e389fe69ec286df4a06c
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "74324044"
---
# <a name="tutorial-using-applications-in-azure-blockchain-workbench"></a>Tutorial: Uso de aplicaciones en Azure Blockchain Workbench

Puede usar Blockchain Workbench para crear y realizar acciones en contratos. También puede ver detalles del contrato como el estado y el historial de transacciones.

Aprenderá a:

> [!div class="checklist"]
> * Crear un contenedor nuevo
> * Realizar una acción en un contrato

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una implementación de Blockchain Workbench. Para más información sobre la implementación, consulte [Implementación de Azure Blockchain Workbench](deploy.md).
* Una aplicación de cadena de bloques implementada en Blockchain Workbench. Consulte [Creación de una aplicación de cadena de bloques en Azure Blockchain Workbench](create-app.md)

[Abra Blockchain Workbench](deploy.md#blockchain-workbench-web-url) en el explorador.

![Blockchain Workbench](./media/use/workbench.png)

Debe iniciar sesión como miembro de Blockchain Workbench. Si no aparece ninguna aplicación, es un miembro de Blockchain Workbench, pero no es miembro de ninguna aplicación. El administrador de Blockchain Workbench puede asignar a miembros a las aplicaciones.

## <a name="create-new-contract"></a>Creación de un nuevo contrato

Para crear un contrato, debe ser un miembro especificado como un **iniciador** de contratos. Para obtener información sobre la definición de roles de aplicación e iniciadores para el contrato, vea los [flujos de trabajo de la información general de configuración](configuration.md#workflows). Para obtener información sobre la asignación de miembros a los roles de aplicación, vea cómo [agregar un miembro a la aplicación](manage-users.md#add-member-to-application).

1. En la sección de aplicaciones de Blockchain Workbench, seleccione el icono de la aplicación que contiene el contrato que desea crear. Se muestra una lista de contratos activos.

2. Para crear un nuevo contrato, seleccione **Nuevo contrato**.

    ![Botón de nuevo contrato](./media/use/contract-list.png)

3. Se muestra el panel **Nuevo contrato**. Especifique los valores de los parámetros iniciales. Seleccione **Crear**.

    ![Panel Nuevo contrato](./media/use/new-contract.png)

    El contrato recién creado se muestra en la lista con los otros contratos activos.

    ![Lista de contratos activos](./media/use/active-contracts.png)

## <a name="take-action-on-contract"></a>Realización de acciones en el contrato

Según el estado que tenga el contrato, los miembros pueden realizar acciones para realizar la transición al siguiente estado del contrato. Las acciones se definen como [transiciones](configuration.md#transitions) dentro de un [estado](configuration.md#states). Los miembros que pertenecen a un rol de aplicación o instancia permitido para la transición pueden realizar la acción. 

1. En la sección de aplicaciones de Blockchain Workbench, seleccione el icono de la aplicación que contiene el contrato en el que realizar la acción.
2. Seleccione el contrato en la lista. Los detalles sobre el contrato se muestran en secciones diferentes. 

    ![Detalles del contrato](./media/use/contract-details.png)

    | Sección  | Descripción  |
    |---------|---------|
    | Estado | Muestra el progreso actual dentro de las fases de contrato |
    | Detalles | Valores actuales del contrato |
    | Acción | Detalles sobre la última acción |
    | Actividad | Historial de transacciones del contrato |
    
3. En la sección **Acción**, seleccione **Realizar acción**.

4. Los detalles sobre el estado actual del contrato se muestran en un panel. Elija la acción que desea realizar en la lista desplegable. 

    ![Elegir acción](./media/use/choose-action.png)

5. Seleccione **Realizar acción** para iniciar la acción.
6. Si se requieren parámetros para la acción, especifique los valores pertinentes.

    ![Realizar acción](./media/use/take-action.png)

7. Seleccione **Realizar acción** para ejecutar la acción.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Versiones de aplicaciones de Azure Blockchain Workbench](version-app.md)
