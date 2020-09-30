---
title: Vinculación de una cuenta de Azure a un identificador de asociado
description: Controle las interacciones con clientes de Azure vinculando un Id. de partner a la cuenta de usuario que utiliza para administrar los recursos del cliente.
author: dhirajgandhi
ms.reviewer: dhgandhi
ms.author: banders
ms.date: 07/24/2020
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.openlocfilehash: dc4d319e0e6b55af8af460fa8a56b9ef24a53341
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89487358"
---
# <a name="link-a-partner-id-to-your-azure-accounts"></a>Vinculación de un Id. de partner a cuentas de Azure

Los asociados de Microsoft proporcionan servicios que ayudan a los clientes lograr los objetivos del negocio y la misión con productos de Microsoft. Cuando actúa en nombre del cliente que administra y configura los servicios de Azure y les da soporte, los usuarios asociados deberán acceder al entorno del cliente. Vínculo de administración de asociados (PAL) permite a los asociados asociar su identificador de red de asociado con las credenciales usadas para la entrega del servicio.

PAL permite a Microsoft identificar y reconocer a los asociados que impulsan el éxito de los clientes de Azure. Microsoft puede atribuir la influencia y los ingresos por consumo de Azure a su organización en función de los permisos de la cuenta (rol de Azure) y el ámbito (suscripción, grupo de recursos y recurso).

## <a name="get-access-from-your-customer"></a>Obtención de acceso del cliente

Antes de vincular su Id. de partner, el cliente debe concederle acceso a sus recursos de Azure mediante una de las siguientes opciones:

- **Usuario invitado**: el cliente puede agregarle como usuario invitado y asignarle roles de Azure. Para más información, consulte [Adición de usuarios invitados de otro directorio](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b).

- **Cuenta de directorio**: el cliente puede crear una cuenta de usuario automáticamente en su propio directorio y asignarle cualquier rol de Azure.

- **Entidad de servicio**: el cliente puede agregar una aplicación o un script de su organización en el directorio del cliente y asignarle cualquier rol de Azure. La identidad de la aplicación o el script se conoce como entidad de servicio.

- **Azure Lighthouse**: el cliente puede delegar una suscripción (o un grupo de recursos) para que los usuarios puedan trabajar en ella desde su inquilino. Para más información, consulte la [administración de recursos delegados de Azure](https://docs.microsoft.com/azure/lighthouse/concepts/azure-delegated-resource-management).

## <a name="link-to-a-partner-id"></a>Vinculación a un Id. de partner

Cuando acceda a los recursos del cliente, use Azure Portal, PowerShell o la CLI de Azure para vincular su identificador de Microsoft Partner Network (Id. de MPN) a su identificador de usuario o entidad de servicio. Vincule el Id. de partner en cada inquilino de cliente.

### <a name="use-the-azure-portal-to-link-to-a-new-partner-id"></a>Uso de Azure Portal para vincular a un nuevo Id. de partner

1. Vaya al [vínculo a un identificador de partner](https://portal.azure.com/#blade/Microsoft_Azure_Billing/managementpartnerblade) en Azure Portal.

2. Inicie sesión en Azure Portal.

3. Escriba el identificador de partner de Microsoft. El Id. de partner es el identificador de [Microsoft Partner Network](https://partner.microsoft.com/) de su organización.

   ![Captura de pantalla que muestra el vínculo a un identificador de partner](./media/link-partner-id/link-partner-id01.png)

4. Para vincular un identificador de partner con otro cliente, cambie el directorio. En **Cambiar directorio**, seleccione su directorio.

   ![Captura de pantalla que muestra la opción Cambiar directorio](./media/link-partner-id/directory-switcher.png)

### <a name="use-powershell-to-link-to-a-new-partner-id"></a>Uso de PowerShell para vincular a un nuevo Id. de partner

1. Instale el módulo de PowerShell [Az.ManagementPartner](https://www.powershellgallery.com/packages/Az.ManagementPartner/).

2. Inicie sesión en el inquilino de cliente con la cuenta de usuario o la entidad de servicio. Para obtener más información, consulte [Inicio de sesión con PowerShell](https://docs.microsoft.com/powershell/azure/authenticate-azureps).

   ```azurepowershell-interactive
    C:\> Connect-AzAccount -TenantId XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
   ```

3. Vincule el nuevo Id. de partner. El Id. de partner es el identificador de [Microsoft Partner Network](https://partner.microsoft.com/) de su organización.

    ```azurepowershell-interactive
    C:\> new-AzManagementPartner -PartnerId 12345
    ```

#### <a name="get-the-linked-partner-id"></a>Obtención del Id. de partner vinculado
```azurepowershell-interactive
C:\> get-AzManagementPartner
```

#### <a name="update-the-linked-partner-id"></a>Actualización del Id. de partner vinculado
```azurepowershell-interactive
C:\> Update-AzManagementPartner -PartnerId 12345
```
#### <a name="delete-the-linked-partner-id"></a>Eliminación del Id. de partner vinculado
```azurepowershell-interactive
C:\> remove-AzManagementPartner -PartnerId 12345
```

### <a name="use-the-azure-cli-to-link-to-a-new-partner-id"></a>Uso de la CLI de Azure para vincular un nuevo Id. de partner
1. Instale la extensión de la CLI de Azure.

    ```azurecli-interactive
    C:\ az extension add --name managementpartner
    ```

2. Inicie sesión en el inquilino de cliente con la cuenta de usuario o la entidad de servicio. Para obtener más información, consulte [Inicio de sesión con la CLI de Azure](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

    ```azurecli-interactive
    C:\ az login --tenant <tenant>
    ```

3. Vincule el nuevo Id. de partner. El Id. de partner es el identificador de [Microsoft Partner Network](https://partner.microsoft.com/) de su organización.

     ```azurecli-interactive
     C:\ az managementpartner create --partner-id 12345
      ```  

#### <a name="get-the-linked-partner-id"></a>Obtención del Id. de partner vinculado
```azurecli-interactive
C:\ az managementpartner show
```

#### <a name="update-the-linked-partner-id"></a>Actualización del Id. de partner vinculado
```azurecli-interactive
C:\ az managementpartner update --partner-id 12345
```

#### <a name="delete-the-linked-partner-id"></a>Eliminación del Id. de partner vinculado
```azurecli-interactive
C:\ az managementpartner delete --partner-id 12345
```

## <a name="next-steps"></a>Pasos siguientes

Únase al debate en la [comunidad de asociados de Microsoft ](https://aka.ms/PALdiscussion) para recibir actualizaciones o enviar comentarios.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**¿Quién puede vincular el Id. de partner?**

Cualquier usuario de la organización asociada que administre los recursos de Azure del cliente puede vincular el identificador de partner a la cuenta.

**¿Se puede cambiar un Id. de partner después de vincularlo?**

Sí. El Id. de partner vinculado se puede cambiar, agregar o quitar.

**¿Qué ocurre si un usuario tiene una cuenta en varios inquilinos de cliente?**

El vínculo entre el Id. de partner y la cuenta se realiza para cada inquilino de cliente. Vincule el Id. de partner en cada inquilino de cliente.

Sin embargo, si va a administrar los recursos del cliente a través de Azure Lighthouse, debe crear el vínculo en el inquilino del proveedor de servicios y usar una cuenta que tenga acceso a los recursos del cliente. Para más información, consulte el artículo sobre la [vinculación de un identificador de asociado para habilitar el crédito que ha obtenido un asociado en los recursos delegados](../../lighthouse/how-to/partner-earned-credit.md).

**¿Pueden otros partners o clientes editar o quitar el vínculo con el Id. de partner?**

El vínculo está asociado al nivel de cuenta del usuario. Solo usted puede editar o quitar el vínculo con el Id. de partner. Ni el cliente ni ningún otro partner podrán modificar el vínculo con el Id. de partner.

**¿Qué id. de MPN debo usar si mi compañía tiene varios?**

Las cuentas de ubicación de asociados y los identificadores MPN se deben usar para vincular el identificador de asociado.  Más información sobre las [cuentas de asociado](https://docs.microsoft.com/partner-center/account-structure)

**¿Dónde puedo encontrar informes de ingresos influenciados para el identificador de asociado vinculado?**

Los informes de rendimiento del producto en la nube están disponibles para los asociados en el centro de asociados en el panel [My Insights dashboard](https://partner.microsoft.com/membership/reports/myinsights) (Panel Mi información). Debe seleccionar Vínculo de administración de asociados como el tipo de asociación de asociados.

**¿Por qué no puedo ver mi cliente en los informes?**

No puede ver al cliente en los informes por los siguientes motivos

1. La cuenta de usuario vinculada no tiene el tipo de acceso [Acceso basado en rol](https://docs.microsoft.com/azure/role-based-access-control/overview) en ningún recurso o suscripción de Azure del cliente.

2. La suscripción de Azure donde el usuario tiene el tipo de acceso [Acceso basado en rol](https://docs.microsoft.com/azure/role-based-access-control/overview) no tiene ningún uso.

**¿Funciona el identificador de asociado del vínculo con Azure Stack?**

Sí, puede vincular su identificador de asociado para Azure Stack.

**¿Cómo se vincula el identificador de asociado si la empresa usa [Azure Lighthouse](../../lighthouse/overview.md) para acceder a los recursos del cliente?**

Si incorpora clientes a la administración de recursos delegados de Azure mediante la [publicación de una oferta de servicios administrados en Azure Marketplace](../../lighthouse/how-to/publish-managed-services-offers.md), el identificador de MPN se asociará automáticamente.

Si [incorpora clientes mediante la implementación de plantillas de Azure Resource Manager](../../lighthouse/how-to/onboard-customer.md), deberá asociar el identificador de MPN con al menos una cuenta de usuario que tenga acceso a cada una de las suscripciones incorporadas. Tenga en cuenta que esto lo deberá realizar en el inquilino del proveedor de servicios, no en el de cada cliente. Para simplificar, se recomienda crear una cuenta de entidad de servicio en el inquilino, asociarla con el identificador de MPN y, después, concederle acceso a todos los clientes que incorpore con un [rol integrado de Azure que sea apto para el crédito obtenido por el asociado](/partner-center/azure-roles-perms-pec). Para más información, consulte el artículo sobre la [vinculación de un identificador de asociado para habilitar el crédito que ha obtenido un asociado en los recursos delegados](../../lighthouse/how-to/partner-earned-credit.md).

**¿Cómo se explica Vínculo de administración de asociados (PAL) a los clientes?**

Vínculo de administración de asociados (PAL) permite a Microsoft identificar y reconocer a los asociados que ayudan a los clientes a lograr objetivos empresariales y a obtener valor en la nube. En primer lugar, los clientes deben proporcionar un acceso de asociado a su recurso de Azure. Una vez que se concede el acceso, se asocia el identificador de Microsoft Partner Network (MPN ID) del asociado. Esta asociación ayuda a Microsoft no solo a conocer el ecosistema de proveedores de servicios de TI, sino también a perfeccionar las herramientas y los programas necesarios para proporcionar mejor soporte técnico a nuestros clientes comunes.

**¿Qué datos recopila PAL?**

La asociación de PAL con las credenciales existentes no proporciona nuevos datos de los clientes a Microsoft. Solo proporciona a Microsoft una telemetría en la que un asociado está implicado activamente en el entorno de Azure de un cliente. Microsoft puede atribuir la influencia y los ingresos por consumo de Azure del entorno del cliente a una organización de asociados en función de los permisos de la cuenta (rol de Azure) y el ámbito (grupo de administración, suscripción, grupo de recursos, recurso) que el cliente proporciona al asociado. 

**¿Afecta esto a la seguridad del entorno de Azure de los clientes?**

La asociación de PAL solo agrega el identificador de MPN del asociado a la credencial ya aprovisionada, no modifica ningún permiso (rol de Azure) ni proporciona datos adicionales del servicio de Azure a ningún asociado ni a Microsoft.
