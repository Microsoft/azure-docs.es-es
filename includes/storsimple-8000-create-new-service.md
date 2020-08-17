---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 88f275016bad72122ebd195449b9af88086bdc7b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87507278"
---
#### <a name="to-create-a-new-service"></a>Para crear un nuevo servicio

1. Use las credenciales de la cuenta Microsoft para iniciar sesión en [Azure Portal](https://portal.azure.com/).

2. En Azure Portal, haga clic en **Crear un recurso** y, una vez en Marketplace, haga clic en **Ver todo**.

    ![Creación de un administrador de dispositivos de StorSimple](./media/storsimple-8000-create-new-service/createssdevman1.png)

    Busque el dispositivo _físico de StorSimple_. Seleccione y haga clic en **Serie física de dispositivos de StorSimple** y, a continuación, haga clic en **Crear**. O bien, en Azure Portal, haga clic en **+** y, una vez en **Almacenamiento**, haga clic en **Serie física de dispositivos de StorSimple**.

    ![Creación de un administrador de dispositivos de StorSimple](./media/storsimple-8000-create-new-service/createssdevman11.png)

3. En la hoja **Administrador de dispositivos de StorSimple**, realice los siguientes pasos:

   1. Proporcione un valor único en **Nombre de recurso** para el servicio. Este nombre es un nombre descriptivo que sirve para identificar el servicio. El nombre puede tener entre 2 y 50 caracteres que pueden ser letras, números y guiones. El nombre debe empezar y terminar con una letra o un número.

   2. Elija una **Suscripción** en la lista desplegable. La suscripción está vinculada a la cuenta de facturación. Este campo no está presente si tiene una sola suscripción.

   3. Como **Grupo de recursos**, elija la opción **Usar existente** o **Crear nuevo**. Para más información, consulte [Grupos de recursos en Azure](/azure/azure-resource-manager/management/manage-resource-groups-portal).

   4. Proporcione una **Ubicación** para el servicio. En general, elija la ubicación más cercana a la región geográfica donde quiera implementar el dispositivo. También es posible que quiera tener en cuenta las siguientes consideraciones:

      * Si ya tiene cargas de trabajo en Azure que se van a implementar con el dispositivo StorSimple, debe usar este centro de datos.
      * El servicio Administrador de dispositivos de StorSimple y Azure Storage pueden estar en dos ubicaciones independientes. En ese caso, se le pedirá que cree la cuenta del Administrador de dispositivos de StorSimple y la de almacenamiento de Azure por separado. Para crear una cuenta de almacenamiento de Azure, vaya al servicio Azure Storage en Azure Portal y siga los pasos indicados en la sección [Crear una cuenta de almacenamiento](../articles/storage/common/storage-account-create.md). Después de crear esta cuenta, agréguela al servicio Administrador de dispositivos de StorSimple mediante los pasos indicados en la sección [Configurar una nueva cuenta de almacenamiento para el servicio](../articles/storsimple/storsimple-8000-deployment-walkthrough-u2.md#configure-a-new-storage-account-for-the-service).

   5. Seleccione **Crear una cuenta de almacenamiento nueva** para crear automáticamente una cuenta de almacenamiento con el servicio. Especifique un nombre para esta cuenta de almacenamiento. Si necesitas tener tus datos en una ubicación diferente, desactiva esta casilla.

   6. Active **Anclar al panel** si desea un vínculo rápido a este servicio en el panel.

   7. Haga clic en **Crear** para crear el Administrador de dispositivos de StorSimple.

       ![Creación de un administrador de dispositivos de StorSimple](./media/storsimple-8000-create-new-service/createssdevman2.png)

Se tarda unos minutos en crear el servicio. Después de que el servicio se crea correctamente, verá una notificación y se abrirá la hoja del nuevo servicio.

![Creación de un administrador de dispositivos de StorSimple](./media/storsimple-8000-create-new-service/createssdevman5.png)
