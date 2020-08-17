---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 07/21/2020
ms.openlocfilehash: 06a503e800309d8689735417931820d9f16a6e75
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87102998"
---
## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de [OneDrive for Business](https://OneDrive.com) 

Para poder usar la cuenta de OneDrive para la Empresa con Logic Apps, debe autorizar a Logic Apps a conectarse a dicha cuenta en Azure Portal.

Siga estos pasos para autorizar a Logic Apps a conectarse a la cuenta de OneDrive para la Empresa:  

1. Inicie sesión en Azure Portal. 

1. En **Servicios de Azure**, seleccione **Logic Apps**. Luego seleccione el nombre de la aplicación lógica en la lista.

1. En el menú de la aplicación lógica, seleccione **Diseñador de aplicación lógica** en **Herramientas de desarrollo**.

1. En el Diseñador de aplicaciones lógicas, seleccione **Mostrar las API administradas por Microsoft** en la lista desplegable y escriba *OneDrive para la Empresa* en el cuadro de búsqueda. Seleccione el desencadenador o la acción que vaya a usar:  

   ![Captura de pantalla del Diseñador de aplicaciones lógicas, que muestra el desencadenador de periodicidad con las acciones de API de OneDrive para la Empresa.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-1.png)

2. Si no ha creado ninguna conexión a OneDrive para la Empresa antes, siga la solicitud para proporcionar sus credenciales de OneDrive para la Empresa. Estas credenciales se usan para autorizar a la aplicación lógica a acceder a los datos de su cuenta de OneDrive para la Empresa:  

   ![Captura de pantalla del Diseñador de aplicaciones lógicas, que muestra la solicitud de inicio de sesión de OneDrive para la Empresa.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-2.png)

3. Indique su nombre de usuario y contraseña de OneDrive para la Empresa para autorizar a la aplicación lógica:  

   ![Captura de pantalla de la página de inicio de sesión de OneDrive para la Empresa, que muestra la solicitud de inicio de sesión.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-3.png)   

4. La conexión ya se muestra en el paso. Seleccione Guardar y siga creando la aplicación lógica. 

   ![Captura de pantalla del Diseñador de aplicaciones lógicas, que muestra el desencadenador con la conexión de OneDrive para la Empresa enumerada.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-4.png)   
