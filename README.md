# EViosLib

Library that enables quick start/access for comunicating with Eterprise Voice skills.

## Getting Started

These instructions will help you implement this library in few easy steps.


### Prerequisites

Use provided .podspec file and add it to your Podfile as follows:

```
...
pod 'EViosLib', :podspec => 'path to .podspec file'
...
```

Install pods using pod install command

### Init

First step is to create instance of EVClientData and populate it with keys you were provided:


```
let data = EVClientLoginData(awsRegion: .USEast1,
                          awsIdentityPoolId: "your identityPoolId",
                          awsUserPoolId: "your userPoolId",
                          lexBotName: "name of bot you will be using",
                          lexBotAlias: "alias of used bot",
                          applicationId: "application ID")
```

All responses are passed trough EVInteractionDelegate so we should should implement it at this point:

```
...
class TestViewController: UIViewController, EVInteractionDelegate {
...
```

Delegate implements two simple methods: onResponse(response: EVInteractorResponseModel) and onError(error: String).
As the names suggest onError provides string response when an error occurs and onResponse method provides bot response data on successful interaction but more abaut response format a bit later. We should first finish with initialization.

Main interaction interface is EVInteractionKit whitch we initialize by passing created EVClientData object together with user identification; user tokens OR by directly logging in the user. In this step we also need to set or viewController which comfors to EVInteractionDelegate protocol as delegate.

Exaple of passing tokens:

```
let interactionKit = EVInteractionKit(clientLoginData: data,
                                            accessToken: userAccessToken,
                                            idToken: userIdToken,
                                            interactionDelegate: self) 
```

Exaple of username/password:

```
let interactionKit = EVInteractionKit(clientLoginData: data,
                                    username: "",
                                    password: "",
                                    appClientId: "application client Id",
                                    interactionDelegate: self)
```

If any of values in data object are wrong or invalid you should get an error in onError delegate method.



## Usage

Now that our interaction kit is ready lets make a simple text request to our bot:

```
interactionKit.textRequest(message: "Help", sessionAttributes: ["testAttr": "testValue"])
```

In our request we pass a simple message and if we need to, we can add new key, value pair to session attributes.

If all goes well you should get a response in your onResponse method. Response data object is EVInteractorResponseModel with following keys:

 - responseType: it can be text or voice, depending on request type
 - sessionAttributes: key value pairs of attributes used in perserving interaction session, custom values form your bot can   be found here
 - outputText: textual response to our request
 - inputTranscript: in case of voice request, in this field you will find String representation of what bot understood
 - audioStream: in case of voice request, Data object containing voice response; it can be easily played using this snippet:
 
```
try! AVAudioSession.sharedInstance().setCategory(.playback, mode: .default, options: [])
player = try AVAudioPlayer(data: stream)
player?.delegate = self
player?.prepareToPlay()
player?.volume = 1.0
player?.play()
```
 
 Few more notes regarding playback:
  - set AVAudioSession to playback category (first line)
  - You sould set AVAudioPlayer delegate and continue interaction AFTER the playback is finished    (audioPlayerDidFinishPlaying method gets triggered)
  - player should be defined outside metods to avoid it beeing dealoccated when metod finishes
