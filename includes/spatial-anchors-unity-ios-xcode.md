---
author: ramonarguelles
ms.service: azure-spatial-anchors
ms.topic: include
ms.date: 08/14/2020
ms.author: rgarcia
ms.openlocfilehash: b93243a537fafce6d865ec207b12dc2654cafd20
ms.sourcegitcommit: c52e50ea04dfb8d4da0e18735477b80cafccc2cf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89536280"
---
Seleccione **Build** (Compilar). En el cuadro de diálogo que aparece, seleccione la carpeta a la que va a exportar el proyecto Xcode.

Cuando se complete la exportación, aparecerá una carpeta que contiene el proyecto Xcode exportado.

> [!NOTE]
> Si aparece una ventana en la que se le pregunta si desea reemplazar o anexar, es aconsejable que seleccione **Append** (Anexar), porque es más rápido. Solo sería necesario seleccionar **Reemplazar** si va a cambiar los recursos de la escena. (Por ejemplo, si va a agregar, quitar, o cambiar las relaciones de elementos primarios y secundarios, o si va a agregar, quitar o cambiar las propiedades.) Si solo va a realizar cambios en el código fuente, **Anexar** debería ser suficiente.

## <a name="open-the-xcode-project"></a>Apertura del proyecto de Xcode

Ahora puede abrir `Unity-iPhone.xcodeproj` en Xcode. Puede iniciar Xcode y abrir el proyecto de `Unity-iPhone.xcodeproj` exportado, o bien iniciar el proyecto en Xcode mediante la ejecución del siguiente comando desde la ubicación de la que exportó el proyecto:

```bash
open ./Unity-iPhone.xcodeproj
```

Seleccione el nodo raíz **Unity-iPhone** para ver la configuración del proyecto y, después, seleccione la pestaña **General**.

En **Firma**, asegúrese de que **Automatically manage signing** (Administrar automáticamente la firma) está habilitado. Si no lo está, habilítelo y seleccione **Enable Automatic** (Habilitar automático) en el cuadro de diálogo que aparece para restablecer la configuración de compilación.

En **Deployment Info** (Información de implementación), asegúrese de que **Deployment Target** (Destino de la implementación) esté establecido en `11.0`.

## <a name="deploy-the-app-to-your-ios-device"></a>Implementación de la aplicación en el dispositivo iOS

Conecte el dispositivo iOS al equipo Mac y establezca el **esquema activo** en su dispositivo iOS.

![Selección del dispositivo](./media/spatial-anchors-unity/select-device.png)

Seleccione **Build and then run the current scheme** (Compilar y luego ejecutar el esquema actual).

![Implementación y ejecución](./media/spatial-anchors-unity/deploy-run.png)
