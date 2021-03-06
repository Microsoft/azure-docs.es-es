---
title: Uso de la inserción de Teams de Azure Communication Services para iOS
description: En esta introducción, aprenderá a usar la biblioteca de inserción de Teams de Azure Communication Services para iOS.
author: palatter
ms.author: palatter
ms.date: 24/02/2021
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: b6b3a2b45c3013170e8341228dd7b78b46881015
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107925379"
---
## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- `User Access Token` para habilitar el cliente de llamada. Para más información sobre [cómo obtener `User Access Token`](../../access-tokens.md)
- Complete el inicio rápido para [empezar a agregar la inserción de Teams a la aplicación](../getting-started-with-teams-embed.md).

## <a name="teams-embed-events"></a>Eventos de inserción de Teams

Agregue `MeetingUIClientDelegate` a la clase.

```swift
class ViewController: UIViewController, MeetingUIClientDelegate {

    private var meetingUIClient: MeetingUIClient?
```

Establecer `meetingUIClientDelegate` en `self`.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    meetingUIClient?.meetingUIClientDelegate = self
}
```

Implemente las funciones `didUpdateCallState` y `didUpdateRemoteParticipantCount`.

```swift
    func meetingUIClient(didUpdateCallState callState: MeetingUIClientCallState) {
        switch callState {
        case .connecting:
            print("Call state has changed to 'Connecting'")
        case .connected:
            print("Call state has changed to 'Connected'")
        case .waitingInLobby:
            print("Call state has changed to 'Waiting in Lobby'")
        case .ended:
            print("Call state has changed to 'Ended'")
        }
    }
    
    func meetingUIClient(didUpdateRemoteParticipantCount remoteParticpantCount: UInt) {
        print("Remote participant count has changed to: \(remoteParticpantCount)")
    }
```

## <a name="assigning-avatars-for-users"></a>Asignación de avatares para usuarios

Agregue `MeetingUIClientIdentityProviderDelegate` a la clase.

```swift
class ViewController: UIViewController, MeetingUIClientIdentityProviderDelegate {

    private var meetingUIClient: MeetingUIClient?
```

Establezca `MeetingUIClientIdentityProviderDelegate` en `self` antes de unirse a la reunión.

```swift
private func joinMeeting() {
    meetingUIClient?.meetingUIClientIdentityProviderDelegate = self
    let meetingJoinOptions = MeetingUIClientMeetingJoinOptions(displayName: "John Smith", enablePhotoSharing: true, enableNamePlateOptionsClickDelegate: true)
    let meetingLocator = MeetingUIClientTeamsMeetingLinkLocator(meetingLink: <MEETING_URL>)
    meetingUIClient?.join(meetingLocator: meetingLocator, joinCallOptions: meetingJoinOptions, completionHandler: { (error: Error?) in
        if (error != nil) {
            print("Join meeting failed: \(error!)")
        }
    })
}
```

Agregue e implemente `avatarFor` y asigne cada `userMri` con el avatar correspondiente.

```swift
    func avatarFor(userIdentifier: String, completionHandler: @escaping (UIImage?) -> Void) {
        if (userIdentifier.starts(with: "8:teamsvisitor:")) {
            // Anonymous teams user will start with prefix 8:teamsvistor:
            let image = UIImage (named: "avatarPink")
            completionHandler(image!)
        }
        else if (userIdentifier.starts(with: "8:orgid:")) {
            // OrgID user will start with prefix 8:orgid:
            let image = UIImage (named: "avatarDoctor")
            completionHandler(image!)
        }
        else if (userIdentifier.starts(with: "8:acs:")) {
            // ACS user will start with prefix 8:acs:
            let image = UIImage (named: "avatarGreen")
            completionHandler(image!)
        }
        else {
            completionHandler(nil)
        }
}

```

Agregue otros métodos de protocolo MeetingUIClientIdentityProviderDelegate obligatorios a la clase y es posible que se queden con una implementación vacía.
```swift
    func displayNameFor(userIdentifier: String, completionHandler: @escaping (String?) -> Void) {
    }
    
    func subTitleFor(userIdentifier: String, completionHandler: @escaping (String?) -> Void) {
    }
```
