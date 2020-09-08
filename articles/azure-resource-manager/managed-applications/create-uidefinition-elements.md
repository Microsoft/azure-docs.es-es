---
title: Creación de elementos de definición de interfaz de usuario
description: Describe los elementos que se usarán al crear definiciones de interfaz de usuario para Azure Portal.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 8b6c6e78c64f83ca39a29f319a103e893d8d8b38
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923728"
---
# <a name="createuidefinition-elements"></a>Elementos CreateUiDefinition

En este artículo se describen el esquema y las propiedades de todos los elementos compatibles de CreateUiDefinition. 

## <a name="schema"></a>Schema

El esquema de la mayoría de los elementos es el siguiente:

```json
{
  "name": "element1",
  "type": "Microsoft.Common.TextBox",
  "label": "Some text box",
  "defaultValue": "my value",
  "toolTip": "Provide a descriptive name.",
  "constraints": {},
  "options": {},
  "visible": true
}
```

| Propiedad | Obligatorio | Descripción |
| -------- | -------- | ----------- |
| name | Sí | Un identificador interno para hacer referencia a una instancia específica de un elemento. El uso más común del nombre del elemento es en `outputs`, donde se asignan los valores de salida de los elementos especificados a los parámetros de la plantilla. También se puede usar para enlazar el valor de salida de un elemento a `defaultValue` de otro elemento. |
| type | Sí | El control de interfaz de usuario que se representa para el elemento. Para ver una lista de los tipos admitidos, consulte [Elementos](#elements). |
| etiqueta | Sí | Texto para mostrar del elemento. Algunos tipos de elementos contienen varias etiquetas, por lo que el valor podría ser un objeto que contiene varias cadenas. |
| defaultValue | No | Valor predeterminado de la propiedad. Algunos tipos de elemento admiten valores predeterminados complejos, por lo que el valor podría ser un objeto. |
| toolTip | No | El texto que se muestra en la información sobre herramientas del elemento. Similar a `label`, algunos elementos admiten varias cadenas de información sobre herramientas. Se pueden insertar vínculos en línea con la sintaxis de Markdown.
| constraints | No | Una o varias propiedades que se utilizan para personalizar el comportamiento de validación del elemento. Las propiedades que se admiten para las restricciones varían según el tipo de elemento. Algunos tipos de elemento no permiten personalizar el comportamiento de validación y, por tanto, no tienen ninguna propiedad constraints. |
| opciones | No | Propiedades adicionales que personalizan el comportamiento del elemento. De forma similar a `constraints`, las propiedades que se admiten varían según el tipo de elemento. |
| visible | No | Indica si se muestra el elemento. Si `true`, se muestran el elemento y los elementos secundarios aplicables. El valor predeterminado es `true`. Use [funciones lógicas](create-uidefinition-functions.md#logical-functions) para controlar dinámicamente el valor de esta propiedad.

## <a name="elements"></a>Elementos

La documentación de cada elemento contiene un ejemplo de interfaz de usuario, un esquema, comentarios sobre el comportamiento del elemento (normalmente sobre la validación y personalización compatibles) y una salida de ejemplo.

- [Microsoft.Common.CheckBox](microsoft-common-checkbox.md)
- [Microsoft.Common.DropDown](microsoft-common-dropdown.md)
- [Microsoft.Common.EditableGrid](microsoft-common-editablegrid.md)
- [Microsoft.Common.FileUpload](microsoft-common-fileupload.md)
- [Microsoft.Common.InfoBox](microsoft-common-infobox.md)
- [Microsoft.Common.OptionsGroup](microsoft-common-optionsgroup.md)
- [Microsoft.Common.PasswordBox](microsoft-common-passwordbox.md)
- [Microsoft.Common.Section](microsoft-common-section.md)
- [Microsoft.Common.Slider](microsoft-common-slider.md)
- [Microsoft.Common.TagsByResource](microsoft-common-tagsbyresource.md)
- [Microsoft.Common.TextBlock](microsoft-common-textblock.md)
- [Microsoft.Common.TextBox](microsoft-common-textbox.md)
- [Microsoft.Compute.CredentialsCombo](microsoft-compute-credentialscombo.md)
- [Microsoft.Compute.SizeSelector](microsoft-compute-sizeselector.md)
- [Microsoft.Compute.UserNameTextBox](microsoft-compute-usernametextbox.md)
- [Microsoft.ManagedIdentity.IdentitySelector](microsoft-managedidentity-identityselector.md)
- [Microsoft.Network.PublicIpAddressCombo](microsoft-network-publicipaddresscombo.md)
- [Microsoft.Network.VirtualNetworkCombo](microsoft-network-virtualnetworkcombo.md)
- [Microsoft.Solutions.ArmApiControl](microsoft-solutions-armapicontrol.md)
- [Microsoft.Solutions.ResourceSelector](microsoft-solutions-resourceselector.md)
- [Microsoft.Storage.MultiStorageAccountCombo](microsoft-storage-multistorageaccountcombo.md)
- [Microsoft.Storage.StorageAccountSelector](microsoft-storage-storageaccountselector.md)

## <a name="next-steps"></a>Pasos siguientes

Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
