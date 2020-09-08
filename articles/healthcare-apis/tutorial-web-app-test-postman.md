---
title: 'Tutorial de la aplicación web: prueba de conexión a Azure API for FHIR'
description: En estos tutoriales se explica un ejemplo para implementar una aplicación web sencilla. En esta sección del tutorial se explican las pruebas de conexión al servidor de FHIR con Postman.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.openlocfilehash: 1c64468a2e420734ca51a5b9308bb52e13712c51
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852929"
---
# <a name="testing-the-fhir-api"></a>Prueba de la API de FHIR
En los dos pasos anteriores, se ha implementado Azure API for FHIR y se ha registrado la aplicación cliente. Ya está preparado para probar que su instancia de Azure API for FHIR está configurada con la aplicación cliente. 

## <a name="retrieve-capability-statement"></a>Recuperación de la instrucción de funcionalidad
En primer lugar, se obtendrá la instrucción de funcionalidad de Azure API for FHIR. 
1. Abra Postman.
1. Recupere la declaración de capacidad mediante GET https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com/metadata. En la imagen siguiente, el nombre del servidor de FHIR es **fhirserver**.

![Instrucción de funcionalidad](media/tutorial-web-app/postman-capability-statement.png)

A continuación, se intentará recuperar a un paciente. Para recuperar un paciente, escriba GET https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com/Patient. Aparecerá un error 401 de falta de autorización. Este error se debe a que no ha demostrado que puede acceder a los datos del paciente.

## <a name="get-patient-from-fhir-server"></a>Obtención de un paciente del servidor de FHIR
![Paciente con error](media/tutorial-web-app/postman-patient-authorization-failed.png)

Para acceder, necesita un token de acceso.
1. En Postman, seleccione **Authorization** (Autorización) y establezca el tipo en **OAuth2.0**.
1. Seleccione **Get New Access Token** (Obtener nuevo token de acceso).
1. Rellene los campos y seleccione **Request Token** (Solicitar token). A continuación puede ver los valores para cada campo de este tutorial.

|Campo                |Value                                                               |
|---------------------|--------------------------------------------------------------------|
|Nombre del token           |Nombre para el token                                               |
|Tipo de concesión           |Código de autorización                                                  |
|Dirección URL de devolución de llamadas         |https://www.getpostman.com/oauth2/callback                          |
|Dirección URL de autenticación             |https://login.microsoftonline.com/\<AZURE-AD-TENANT-ID> /oauth2/?resource=https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com|
|Dirección URL del token de acceso     |https://login.microsoftonline.com/\<AZURE-AD-TENANT-ID> /oauth2/token|
|Id. de cliente            |Identificador de cliente copiado en los pasos anteriores             |
|Secreto del cliente        |\<BLANK>                                                            |
|Ámbito                |\<BLANK>                                                            |
|State                |1234                                                                |
|Autenticación de clientes|Envío de credenciales de cliente en el cuerpo                                     |

4. Inicie sesión con sus credenciales y seleccione **Accept** (Aceptar).
1. Desplácese hacia abajo en el resultado y seleccione **Use Token** (Usar token).
1. Seleccione **Send** (Enviar) de nuevo en la parte superior y esta vez debería obtener un resultado de ![paciente correcto](media/tutorial-web-app/postman-patient-authorization-success.png).

## <a name="post-patient-into-fhir-server"></a>Publicación de un paciente en el servidor de FHIR
Ya tiene acceso y puede crear un nuevo paciente. Este es un ejemplo de un paciente único que puede agregar al servidor de FHIR. Escriba el código siguiente en la sección **Body** (Cuerpo) de Postman.

``` json
    {
        "resourceType": "Patient",
        "active": true,
        "name": [
            {
                "use": "official",
                "family": "Kirk",
                "given": [
                    "James",
                    "Tiberious"
                ]
            },
            {
                "use": "usual",
                "given": [
                    "Jim"
                ]
            }
        ],
        "gender": "male",
        "birthDate": "1960-12-25"
    }
```
En esta publicación se creará un nuevo paciente en el servidor de FHIR con el nombre James Tiberious Kirk.
![Publicación de un paciente](media/tutorial-web-app/postman-post-patient.png)

Si realiza el paso GET anterior para recuperar a un paciente, verá James Tiberious Kirk en la salida.

## <a name="troubleshooting-access-issues"></a>Solución de problemas de acceso
Si se han producido problemas durante alguno de estos pasos, revise los documentos que hemos reunido en Azure Active Directory y Azure API for FHIR. 

* [Azure AD y Azure API for FHIR](azure-ad-hcapi.md): en este documento se describen algunos de los principios básicos de Azure Active Directory y cómo interactúa con Azure API for FHIR.
* [Validación de tokens de acceso](azure-ad-hcapi-token-validation.md): esta guía de procedimientos proporciona detalles más específicos sobre la validación de tokens de acceso y los pasos que se deben llevar a cabo para resolver problemas de acceso.

## <a name="next-steps"></a>Pasos siguientes
Ahora que ya puede conectarse correctamente a la aplicación cliente, está preparado para escribir la aplicación web.

>[!div class="nextstepaction"]
>[Escribir una aplicación web](tutorial-web-app-write-web-app.md)



