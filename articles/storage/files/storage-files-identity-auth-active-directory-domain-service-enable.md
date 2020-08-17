---
title: Uso de Azure AD Domain Services para autorizar el acceso a los datos de archivo a través de SMB
description: Aprenda a habilitar la autenticación basada en identidades a través del Bloque de mensajes del servidor (SMB) para Azure Files mediante Azure Active Directory Domain Services. Las máquinas virtuales Windows unidas a un dominio pueden acceder entonces a los recursos compartidos de archivos de Azure con las credenciales de Azure AD.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 04/21/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: fe2bbc1d6f42819354f48812a34371a49e5acbac
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86999639"
---
# <a name="enable-azure-active-directory-domain-services-authentication-on-azure-files"></a>Habilitación de la autenticación de Azure Active Directory Domain Services en Azure Files

[Azure Files](storage-files-introduction.md) admite la autenticación basada en la identidad a través del protocolo Bloque de mensajes del servidor (SMB) mediante dos tipos de servicios de dominio: Active Directory Domain Services local (AD DS) y Azure Active Directory Domain Services (Azure AD DS). Le recomendamos encarecidamente que revise la sección [Funcionamiento](https://docs.microsoft.com/azure/storage/files/storage-files-active-directory-overview#how-it-works) para seleccionar el servicio de dominio adecuado para la autenticación. La instalación es diferente en función del servicio de dominio que elija. Este artículo se centra en la habilitación y configuración de Azure AD DS local para la autenticación con recursos compartidos de archivos de Azure.

Si no está familiarizado con los recursos compartidos de archivos de Azure, se recomienda leer la [guía de plan](storage-files-planning.md) antes de leer la siguiente serie de artículos.

> [!NOTE]
> Azure Files admite la autenticación Kerberos con Azure AD DS con cifrado RC4-HMAC. Todavía no se admite el cifrado de Kerberos con AES.
> Azure Files admite la autenticación de Azure AD DS con sincronización completa con Azure AD. Si se ha habilitado la sincronización de ámbito en Azure AD DS, que solo sincroniza un conjunto limitado de identidades de Azure AD, no se admite la autenticación ni la autorización.

## <a name="prerequisites"></a>Requisitos previos

Antes de habilitar Azure AD a través de SMB para recursos compartido de archivos para recursos compartido de archivos de Azure, asegúrese de haber completado los requisitos previos siguientes:

1.  **Seleccione o cree un inquilino de Azure AD.**

    Puede usar un inquilino nuevo o uno existente para la autenticación de Azure AD a través de SMB. El inquilino y el recurso compartido de archivos a los que desea acceder deben asociarse con la misma suscripción.

    Para crear un nuevo inquilino de Azure AD, puede [agregar un inquilino de Azure AD y una suscripción de Azure AD](https://docs.microsoft.com/windows/client-management/mdm/add-an-azure-ad-tenant-and-azure-ad-subscription). Si ya tiene un inquilino de Azure AD, pero desea crear otro para usarlo con los recursos compartido de archivos de Azure, consulte [Creación de un inquilino de Azure Active Directory](https://docs.microsoft.com/rest/api/datacatalog/create-an-azure-active-directory-tenant).

1.  **Habilite Azure AD Domain Services en el inquilino de Azure AD.**

    Para admitir la autenticación con las credenciales de Azure AD, debe habilitar Azure AD Domain Services para su inquilino de Azure AD. Si no es el administrador del inquilino de Azure AD, póngase en contacto con el administrador y siga las instrucciones paso a paso para [habilitar Azure Active Directory Domain Services mediante Azure Portal](../../active-directory-domain-services/tutorial-create-instance.md).

    Una implementación de Azure AD DS suele tardar unos 15 minutos en completarse. Confirme que el estado de mantenimiento de Azure AD muestra **En ejecución**, con la sincronización de hash de contraseña habilitada, antes de continuar con el paso siguiente.

1.  **Unión al dominio de una máquina virtual de Azure con Azure AD DS.**

    Para acceder a un recurso compartido de archivos mediante las credenciales de Azure AD desde una máquina virtual, esta debe unirse a un dominio con Azure AD DS. Para más información sobre cómo unir a un dominio una máquina virtual, consulte [Unión de una máquina virtual de Windows Server a un dominio administrado](../../active-directory-domain-services/join-windows-vm.md).

    > [!NOTE]
    > La autenticación de Azure AD DS a través de SMB con recursos compartido de archivos de Azure solo se admite en máquinas virtuales de Azure que se ejecutan en versiones del sistema operativo posteriores a Windows 7 o Windows Server 2008 R2.

1.  **Seleccione o cree un recurso compartido de archivos**.

    Seleccione un recurso compartido de archivos nuevo o existente que esté asociado con la misma suscripción que el inquilino de Azure AD. Para información acerca de cómo crear un nuevo recurso compartido de archivos, consulte [Creación de un recurso compartido de archivos en Azure Files](storage-how-to-create-file-share.md).
    Para obtener un rendimiento óptimo, se recomienda que el recurso compartido de archivos esté en la misma región que la máquina virtual desde la que vaya a acceder al recurso compartido.

1.  **Compruebe la conectividad de Azure Files; para ello, monte los recursos compartidos de archivos de Azure con la clave de su cuenta de almacenamiento.**

    Para comprobar que el recurso compartido de archivos y de la máquina virtual están configurados correctamente, intente montar el recurso compartido de archivos con la clave de la cuenta de almacenamiento. Para más información, consulte [Montaje de un recurso compartido de archivos de Azure y acceso al recurso compartido en Windows](storage-how-to-use-files-windows.md).

## <a name="regional-availability"></a>Disponibilidad regional

La autenticación de Azure Files con Azure AD DS está disponible en [todas las regiones públicas de Azure](https://azure.microsoft.com/global-infrastructure/locations/).

## <a name="overview-of-the-workflow"></a>Información general del flujo de trabajo

Antes de habilitar la autenticación de Azure AD DS a través de SMB para los recursos compartido de archivos de Files, compruebe que los entornos de Azure Storage y Azure AD se han configurado correctamente. Se recomienda que revise los [requisitos previos](#prerequisites) para asegurarse de que ha completado todos los pasos necesarios.

Luego, realice estas acciones para conceder acceso a los recursos de Azure Files con las credenciales de Azure AD:

1. Habilite la autenticación de Azure AD DS a través de SMB en la cuenta de almacenamiento a fin de registrarla con la implementación de Azure AD DS asociada.
2. Asigne permisos de acceso para un recurso compartido a una identidad de Azure AD (usuario, grupo o entidad de servicio).
3. Configure los permisos NTFS a través de SMB para archivos y directorios.
4. Monte un recurso compartido de archivos de Azure desde una máquina virtual unida al dominio.

En el diagrama siguiente se ilustra el flujo de trabajo de un extremo a otro para habilitar la autenticación de Azure AD DS a través de SMB para Azure Files.

![Diagrama que muestra Azure AD a través de SMB para el flujo de trabajo de Azure Files](media/storage-files-active-directory-enable/azure-active-directory-over-smb-workflow.png)

## <a name="1-enable-azure-ad-ds-authentication-for-your-account"></a>1. Habilitación de la autenticación de Azure AD DS para la cuenta

Para habilitar la autenticación de Azure AD DS a través de SMB para Azure Files, puede establecer una propiedad en las cuentas de almacenamiento mediante Azure Portal, Azure PowerShell o la CLI de Azure. Al establecer esta propiedad "une a un dominio" implícitamente la cuenta de almacenamiento con la implementación de Azure AD DS asociada. La autenticación de Azure AD DS a través de SMB se habilita entonces para todos los recursos compartidos de archivos nuevos y existentes de la cuenta de almacenamiento.

Tenga en cuenta que solo puede habilitar la autenticación de Azure AD DS a través de SMB después de haber implementado correctamente Azure AD DS en el inquilino de Azure AD. Para más información, consulte la sección los [requisitos previos](#prerequisites).

### <a name="azure-portal"></a>Azure portal

Para habilitar la autenticación de Azure AD DS a través de SMB mediante [Azure Portal](https://portal.azure.com), siga estos pasos:

1. En Azure Portal, vaya a la cuenta de almacenamiento existente o [cree una](../common/storage-account-create.md).
1. En la sección **Configuración**, haga clic en **Configuración**.
1. En **Identity-based access for file shares** (Acceso basado en identidad) cambie el elemento de alternancia de **Azure Active Directory Domain Service (AAD DS)** a **Enabled** (Habilitado).
1. Seleccione **Guardar**.

En la imagen siguiente se muestra cómo habilitar la autenticación de Azure AD DS a través de SMB para la cuenta de almacenamiento.

![Habilitación de la autenticación de Azure AD DS a través de SMB en Azure Portal](media/storage-files-active-directory-enable/portal-enable-active-directory-over-smb.png)

### <a name="powershell"></a>PowerShell  

Para habilitar la autenticación de Azure AD DS a través de SMB con Azure PowerShell, instale el módulo Az más reciente (2.4 o posterior) o el módulo Az.Storage (1.5 o posterior). Para más información sobre cómo instalar PowerShell, consulte [Instalación de Azure PowerShell en Windows con PowerShellGet](https://docs.microsoft.com/powershell/azure/install-Az-ps):

Para crear una cuenta de almacenamiento, llame a [New-AzStorageAccount](https://docs.microsoft.com/powershell/module/az.storage/New-azStorageAccount?view=azps-2.5.0) y establezca el parámetro **EnableAzureActiveDirectoryDomainServicesForFile** en **true**. En el ejemplo siguiente, no olvide reemplazar los valores de marcador de posición por los suyos propios. (Si ha estado usando el módulo de versión preliminar anterior, el parámetro para la habilitación de características es **EnableAzureFilesAadIntegrationForSMB**).

```powershell
# Create a new storage account
New-AzStorageAccount -ResourceGroupName "<resource-group-name>" `
    -Name "<storage-account-name>" `
    -Location "<azure-region>" `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableAzureActiveDirectoryDomainServicesForFile $true
```

Para habilitar esta característica en cuentas de almacenamiento existentes, use el siguiente comando:

```powershell
# Update a storage account
Set-AzStorageAccount -ResourceGroupName "<resource-group-name>" `
    -Name "<storage-account-name>" `
    -EnableAzureActiveDirectoryDomainServicesForFile $true
```


### <a name="azure-cli"></a>Azure CLI

Para habilitar la autenticación de Azure AD mediante SMB con la CLI de Azure, instale la versión más reciente de la CLI (versión 2.0.70 o posterior). Para más información sobre cómo instalar la CLI de Azure, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

Para crear una cuenta de almacenamiento nueva, llame a [az storage account create](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create) y establezca la propiedad `--enable-files-aadds` en **true**. En el ejemplo siguiente, no olvide reemplazar los valores de marcador de posición por los suyos propios. (Si ha estado usando el módulo de versión preliminar anterior, el parámetro para la habilitación de características es **file-aad**).

```azurecli-interactive
# Create a new storage account
az storage account create -n <storage-account-name> -g <resource-group-name> --enable-files-aadds $true
```

Para habilitar esta característica en cuentas de almacenamiento existentes, use el siguiente comando:

```azurecli-interactive
# Update a new storage account
az storage account update -n <storage-account-name> -g <resource-group-name> --enable-files-aadds $true
```

[!INCLUDE [storage-files-aad-permissions-and-mounting](../../../includes/storage-files-aad-permissions-and-mounting.md)]

Ya ha habilitado correctamente la autenticación de Azure AD DS a través de SMB y ha asignado un rol personalizado que proporciona acceso a un recurso compartido de archivos de Azure con una identidad de Azure AD. Para conceder a otros usuarios acceso al recurso compartido de archivos, siga las instrucciones de las secciones [Asignación de permisos de acceso](#2-assign-access-permissions-to-an-identity) para usar una identidad y [Configuración de los permisos NTFS en secciones de SMB](#3-configure-ntfs-permissions-over-smb).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Files y el uso de Azure AD a través de SMB, consulte estos recursos:

- [Introducción a la compatibilidad de la autenticación basada en la identidad de Azure Files con el acceso SMB](storage-files-active-directory-overview.md)
- [P+F](storage-files-faq.md)
