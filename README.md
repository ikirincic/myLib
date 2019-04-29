# EViosLib 
  

Library that enables quick start/access for communicating with Enterprise Voice skills.  

  
## Getting Started 

These instructions will help you implement this library in few easy steps. 

  
### Prerequisites 

Use provided .podspec file and add it to your Podfile as follows: 

``` 
... 
pod 'EViosLib', :podspec => 'path to .podspec file' 
... 
``` 
  
Install pods using __pod install__ command. No other pods are needed. All AWS dependencies are installed with EV pod. 

  
### Init 

First step is to create an instance of EVClientData and populate it with keys you were provided. For demo purposes we will create it in 'viewDidLoad()' methot of the view controller but you should cretate it where it suits your architecture and when you get keys via API call or some other method.

``` 
let data = EVClientLoginData(awsRegion: .USEast1, 
                          awsIdentityPoolId: "your identityPoolId", 
                          awsUserPoolId: "your userPoolId", 
                          lexBotName: "name of a bot you will be using", 
                          lexBotAlias: "alias of used bot", 
                          applicationId: "application ID") 
``` 
  
All responses from library are passed through EVInteractionDelegate. Again, for demo purposes, we will use the view controller in which we are doing the test implementation. You should use the class that best suits your needs.

``` 
... 
class TestViewController: UIViewController, EVInteractionDelegate { 
... 
``` 

Delegate implements two simple methods: 'onResponse(response: EVInteractorResponseModel)' and 'onError(error: String)'. 

As the names suggest 'onError' provides string response when an error occurs and 'onResponse' method provides bot response data on successful interaction but more about response format a bit later. We should first finish with initialization. 

Main interaction interface is EVInteractionKit which we initialize by passing created EVClientData object together with user identification; user tokens OR by directly logging in the user. In this step we also need to set the class which comforts to EVInteractionDelegate protocol as a delegate. 

Example of passing tokens: 

``` 
let interactionKit = EVInteractionKit(clientLoginData: data, 
                                            accessToken: userAccessToken, 
                                            idToken: userIdToken, 
                                            interactionDelegate: self)
``` 

Example of username/password: 

``` 
let interactionKit = EVInteractionKit(clientLoginData: data, 
                                    username: "", 
                                    password: "", 
                                    appClientId: "application client Id", 
                                    interactionDelegate: self) 
``` 

If any of values in the data object are wrong or invalid, you should get an error in 'onError' delegate method. 


## Usage 

Now that the interaction kit is ready let's make a simple text request to the bot: 

``` 
interactionKit.textRequest(message: "Help", sessionAttributes: ["testAttr": "testValue"]) 
``` 

In the request we pass a simple message and if we need to, we can add new key & value pair to session attributes.   

If all goes well, you should get a response in your delegates 'onResponse' method. Response data object is EVInteractorResponseModel with following keys: 

- __responseType__: it can be text or voice, depending on request type 
- __sessionAttributes__: key value pairs of attributes used in preserving interaction session; custom values form your bot can be found here 
- __outputText__: textual response to the request 
- __inputTranscript__: in the case of voice request, in this field you will find String representation of what the bot understood 
- __audioStream__: in the case of voice request, Data object containing voice response; it can be easily played using this code snippet: 

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
  - You should set AVAudioPlayer delegate and continue the interaction AFTER the playback is finished    ('audioPlayerDidFinishPlaying' method gets triggered) 
  - player should be defined outside methods to avoid it being deallocated when the method finishes 
