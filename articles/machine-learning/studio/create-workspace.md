---
title: 'ML Studio (clásico): creación de un área de trabajo en Azure'
description: Para usar Azure Machine Learning Studio (clásico), debe tener un área de trabajo de Machine Learning Studio. Esta área de trabajo contiene las herramientas que necesita para crear, administrar y publicar experimentos.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 12/07/2017
ms.openlocfilehash: 89c79f668da3ff924dbb7f27574260118003655f
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87433248"
---
# <a name="create-and-share-an-machine-learning-studio-classic-workspace"></a>Creación y uso compartido de un área de trabajo de Machine Learning Studio (clásico)

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico) ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)

Para usar Azure Machine Learning Studio (clásico), debe tener un área de trabajo de Machine Learning Studio. Esta área de trabajo contiene las herramientas que necesita para crear, administrar y publicar experimentos.

## <a name="create-a-studio-classic-workspace"></a>Creación de un área de trabajo de Studio (clásico)

Para abrir un área de trabajo en Machine Learning Studio (clásico), debe iniciar sesión en la cuenta Microsoft que utilizó para crear el área de trabajo, o bien, puede ser necesario que reciba una invitación del propietario para unirse al área de trabajo. Desde Azure Portal puede administrar el área de trabajo, lo que incluye la capacidad de configurar el acceso.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/)

    > [!NOTE]
    > Para iniciar sesión y crear un área de trabajo de Studio (clásico), debe ser un administrador de suscripciones de Azure. 
    >
    > 

2. Haga clic en **+ Nuevo**.

3. En el cuadro de búsqueda, escriba **Área de trabajo de Machine Learning Studio (clásico)** y seleccione el elemento coincidente. A continuación, en la parte inferior de la página, haga clic en **Crear**.

4. Escriba la información de su área de trabajo:

   - El *nombre de área de trabajo* puede tener hasta 260 caracteres, pero no puede terminar con un espacio. El nombre no puede incluir estos caracteres: `< > * % & : \ ? + /`
   - El *plan de servicio web* elegido (o creado), junto con el *plan de tarifa* asociado seleccionado, se usa si implementa servicios web desde esta área de trabajo.

     ![Creación de un área de trabajo de Studio (clásico)](./media/create-workspace/create-new-workspace.png)

5. Haga clic en **Crear**.

   Machine Learning se encuentra actualmente disponible en un número limitado de regiones. Si su suscripción no incluye una de estas regiones, es posible que vea el mensaje de error "No tiene subscripciones en las regiones permitidas".  Para solicitar que una región se agregue a su suscripción, cree una nueva solicitud de soporte técnico de Microsoft desde Azure Portal, elija **Facturación** como el tipo de problema y siga las indicaciones para enviar su solicitud.


> [!NOTE]
> Machine Learning Studio (clásico) depende de la cuenta de Azure Storage que suministre para guardar los datos intermedios al ejecutar el flujo de trabajo. Una vez que se crea el área de trabajo, si se elimina la cuenta de almacenamiento y se cambian las claves de acceso, el área de trabajo dejará de funcionar y todos los experimentos que haya en ella darán error.
Si ha eliminado accidentalmente la cuenta, vuelva a crearla con el mismo nombre y en la misma región que la eliminada, y vuelva a sincronizar la clave de acceso. Si ha cambiado las claves de acceso de la cuenta de almacenamiento, vuelva a sincronizarlas en el área de trabajo mediante Azure Portal.

Una vez implementado el área de trabajo, puede abrirlo en Machine Learning Studio (clásico).

1. Vaya a Machine Learning Studio (clásico) en [https://studio.azureml.net/](https://studio.azureml.net/).

2. Seleccione el área de trabajo en la esquina superior derecha.

    ![Selección del área de trabajo](./media/create-workspace/open-workspace.png)

3. Haga clic en **my experiments** (Mis experimentos).

    ![Abrir experimentos](./media/create-workspace/my-experiments.png)

Para más información sobre cómo administrar un área de trabajo de Studio (clásico), consulte [Administración de un área de trabajo de Azure Machine Learning Studio (clásico)](manage-workspace.md).
Si tiene algún problema al crear el área de trabajo, consulte [Guía de solución de problemas: Creación y conexión a un área de trabajo de Machine Learning Studio (clásico)](troubleshooting-creating-ml-workspace.md).


## <a name="share-an-azure-machine-learning-studio-classic-workspace"></a>Uso compartido de un área de trabajo de Azure Machine Learning Studio (clásico)
Tras crear un área de trabajo de Machine Learning Studio (clásico), puede invitar usuarios y compartir el acceso a dicha área y a todos sus experimentos, conjuntos de datos, cuadernos, etc. Puede agregar usuarios en uno de estos roles:

* **Usuario**: los usuarios de un área de trabajo pueden crear, abrir, modificar y eliminar experimentos, conjuntos de datos y otros elementos en el área de trabajo.
* **Propietario**: un propietario puede invitar y quitar usuarios en el área de trabajo, además de lo que pueden hacer los usuarios.

> [!NOTE]
> La cuenta de administrador que crea el área de trabajo se agrega automáticamente al área de trabajo como propietario de dicha área. No obstante, a los otros administradores o usuarios de dicha suscripción no se les concede acceso automáticamente al área de trabajo; debe invitarlos de manera explícita.
> 
> 

### <a name="to-share-a-studio-classic-workspace"></a>Para crear un área de trabajo de Studio (clásico)

1. Inicie sesión en Machine Learning Studio (clásico) en [https://studio.azureml.net/Home](https://studio.azureml.net/Home).

2. En el panel izquierdo, haga clic en **SETTINGS** (Configuración).

3. Haga clic en la pestaña **USERS** (Usuarios).

4. Haga clic en **INVITE MORE USERS** (Invitar más usuarios) en la parte inferior de la página.

    ![Configuración de Studio](./media/create-workspace/settings.png)

5. Escriba una o varias direcciones de correo electrónico. Los usuarios necesitan una cuenta Microsoft o una cuenta de organización (de Azure Active Directory) válidas.

6. Seleccione si desea agregar los usuarios como propietario o usuario.

7. Haga clic en el botón de marca de verificación **OK** (Aceptar).

Cada usuario agregado recibirá un correo electrónico con instrucciones sobre cómo iniciar sesión en el área de trabajo compartida.

> [!NOTE]
> Para que los usuarios puedan implementar o administrar servicios web en esta área de trabajo, deben ser colaborador o administrador de la suscripción de Azure. 

## <a name="troubleshoot-storage-accounts"></a>Solución de problemas de cuentas de almacenamiento


El servicio Machine Learning necesita una cuenta de almacenamiento para almacenar los datos. Puede utilizar una cuenta de almacenamiento existente o crear una nueva cuenta de almacenamiento al crear la nueva área de trabajo de Machine Learning Studio (clásico) (si dispone de cuota para crear una nueva cuenta de almacenamiento).

Tras crear la nueva área de trabajo de Machine Learning Studio (clásico), puede iniciar sesión en Machine Learning Studio (clásico) con la cuenta de Microsoft que usó para crear el área de trabajo. Si encuentra un mensaje de error que indica que no se ha encontrado el área de trabajo (parecido a la captura de pantalla siguiente), use estos pasos para eliminar las cookies del explorador.

![Espacio de trabajo no encontrado](media/troubleshooting-creating-ml-workspace/screen3.png)

**Para eliminar las cookies del explorador**

1. Si usa Internet Explorer, haga clic en el botón **Herramientas** en la esquina superior derecha y seleccione **Opciones de Internet**.  

   ![Opciones de Internet](media/troubleshooting-creating-ml-workspace/screen4.png)

2. En la pestaña **General**, haga clic en **Eliminar…**

   ![Pestaña General](media/troubleshooting-creating-ml-workspace/screen5.png)

3. En el cuadro de diálogo **Eliminar el historial de exploración**, asegúrese de que **Cookies y datos del sitio web** está seleccionado y haga clic en **Eliminar**.

   ![Eliminación de cookies](media/troubleshooting-creating-ml-workspace/screen6.png)

Una vez eliminadas las cookies, reinicie el explorador y vaya a la página [Microsoft Azure Machine Learning Studio (clásico)](https://studio.azureml.net). Cuando se le solicite un nombre de usuario y contraseña, especifique la misma cuenta Microsoft que usó para crear el área de trabajo.


## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de cómo administrar un área de trabajo, consulte [Administración de un área de trabajo de Azure Machine Learning Studio (clásico)](manage-workspace.md).
