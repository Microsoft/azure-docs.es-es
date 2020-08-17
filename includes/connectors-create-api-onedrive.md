---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 07/21/2020
ms.openlocfilehash: 27df83552b450b7657c0595d1d419001df98148c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87040243"
---
## <a name="prerequisites"></a>Prerequisites

* Una cuenta de [OneDrive](https://www.microsoft.com/store/apps/onedrive/9wzdncrfj1p3). 

Para poder usar la cuenta de OneDrive con Logic Apps, debe autorizar a Logic Apps a conectarse a la cuenta de OneDrive en Azure Portal.

Siga estos pasos para autorizar a Logic Apps a conectarse a la cuenta de OneDrive:  

1. Inicie sesión en Azure Portal. 

1. En **Servicios de Azure**, seleccione **Logic Apps**. Luego seleccione el nombre de la aplicación lógica en la lista.

1. En el menú de la aplicación lógica, seleccione **Diseñador de aplicación lógica** en **Herramientas de desarrollo**.

1. En el Diseñador de aplicación lógica, seleccione **Mostrar las API administradas por Microsoft** en la lista desplegable y escriba *OneDrive* en el cuadro de búsqueda. Seleccione el desencadenador o la acción que vaya a usar:

   ![Captura de pantalla del Diseñador de aplicación lógica con una lista de acciones de la API de OneDrive que se van a agregar.](./media/connectors-create-api-onedrive/onedrive-1.png)

2. Si no ha creado previamente ninguna conexión a OneDrive, siga la solicitud para iniciar sesión con las credenciales de OneDrive:  

   ![Captura de pantalla del Diseñador de aplicación lógica con la solicitud de inicio de sesión de la API de OneDrive.](./media/connectors-create-api-onedrive/onedrive-2.png)

3. Seleccione **Iniciar sesión** y escriba su nombre de usuario y contraseña. Seleccione **Iniciar sesión**: 

   ![Captura de pantalla de la página de inicio de sesión de la cuenta Microsoft para la autorización de la API de OneDrive.](./media/connectors-create-api-onedrive/onedrive-3.png)   

    Estas credenciales se usan para autorizar a la aplicación lógica a acceder a los datos de la cuenta de OneDrive. 

4. Seleccione **Sí** para autorizar a la aplicación lógica a usar la cuenta de OneDrive:  

   ![Captura de pantalla de autorización de cuenta Microsoft para Logic Apps con las acciones permitidas.](./media/connectors-create-api-onedrive/onedrive-4.png)   
   
5. La conexión ya se muestra en el paso. Seleccione Guardar y siga creando la aplicación lógica. 

   ![Captura de pantalla del Diseñador de aplicación lógica que muestra el editor de acción con la conexión de la API de OneDrive.](./media/connectors-create-api-onedrive/onedrive-5.png)
