# Building real-time applications with iOS, GraphQL & AWS AppSync

In this workshop we'll learn how to build cloud-enabled native iOS Swift apps with [AWS Amplify](https://aws-amplify.github.io/).

![](https://imgur.com/IPnnJyf.jpg)

### Topics we'll be covering:

- [GraphQL API with AWS AppSync](https://github.com/dennisAWS/aws-appsync-ios-workshop#getting-started---create-an-xcode-project)
- [Authentication](https://github.com/dennisAWS/aws-appsync-ios-workshop#adding-authentication)
- [Adding Authorization to the AWS AppSync API](https://github.com/dennisAWS/aws-appsync-ios-workshop#adding-authorization-to-the-graphql-api)
- [Creating & working with multiple serverless environments](https://github.com/dennisAWS/aws-appsync-ios-workshop#multiple-serverless-environments)
- [Deleting the resources](https://github.com/dennisAWS/aws-appsync-ios-workshop#removing-services)


## Getting Started - Create an Xcode project

To get started, create a new Xcode project & save as: `ios-amplify-app`

From Terminal, change into the new app directory & install the Amplify CLI.

### Install the Amplify CLI

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```

After installation, configure the CLI with your developer credentials:

Note: If you already have the AWS CLI installed and use a named profile, you can skip the `amplify configure` step.

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __us-east-1__
- Specify the username of the new IAM user: __amplify-workshop-user__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
? accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
? secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __amplify-workshop-user__

### Initializing A New Amplify Project
From the root of your Xcode project folder:
```bash
amplify init
```

- Enter a name for the project: __iosamplifyapp__
- Enter a name for the environment: __master__
- Choose your default editor: __Visual Studio Code (or your default editor)__   
- Please choose the type of app that you're building __ios__     
- Do you want to use an AWS profile? __Y__
- Please choose the profile you want to use: __amplify-workshop-user__

AWS Amplify CLI will iniatilize a new project & you will see a new folder: __amplify__ & a new file called `awsconfiguration.json` in the root directory. These files holds your Amplify project configuration.

To view the status of the amplify project at any time, you can run the Amplify `status` command:

```sh
amplify status
```

## Adding a GraphQL API

To add a GraphQL API, we can use the following command:

```sh
amplify add api
```

Answer the following questions

- Please select from one of the above mentioned services __GraphQL__   
- Provide API name: __ConferenceAPI__   
- Choose an authorization type for the API __API key__   
- Do you have an annotated GraphQL schema? __N__   
- Do you want a guided schema creation? __Y__   
- What best describes your project: __Single object with fields (e.g. “Todo” with ID, name, description)__   
- Do you want to edit the schema now? (Y/n) __Y__   

When prompted, update the schema to the following:   

```graphql
type Talk @model {
  id: ID!
  clientId: ID
  name: String!
  description: String!
  speakerName: String!
  speakerBio: String!
}
```

Next, let's deploy the GraphQL API into our account:

```bash
amplify push
```

- Do you want to generate code for your newly created GraphQL API __Y__
- Enter the file name pattern of graphql queries, mutations and subscriptions: __(graphql/**/*.graphql)__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions? __Y__
- Enter maximum statement depth [increase from default if your schema is deeply nested] __2__
- Enter the file name for the generated code __API.swift__

> To view the new AWS AppSync API at any time after its creation, go to the dashboard at [https://console.aws.amazon.com/appsync](https://console.aws.amazon.com/appsync). Also be sure that your region is set correctly.

### Performing mutations from within the AWS AppSync Console

In the AWS AppSync console, open your API & then click on Queries.

Execute the following mutation to create a new talk in the API:

```bash
mutation createTalk {
  createTalk(input: {
    name: "Monetize your Mobile Apps"
    description: "4 ways to make money as a mobile app developer"
    speakerName: "Dennis"
    speakerBio: "Mobile Quickie Developer"
  }) 
  {
      id 
      name 
      description 
      speakerName 
      speakerBio
  }
}
```

Now, let's query for the talk:

```bash
query listTalks {
  listTalks {
    items {
      id
      name
      description
      speakerName
      speakerBio
    }
  }
}
```

We can even add search / filter capabilities when querying:

```bash
query listTalks {
  listTalks(filter: {
    description: {
      contains: "money"
    }
  }) {
    items {
      id
      name
      description
      speakerName
      speakerBio
    }
  }
}
```

### Configuring the iOS applicaion - AppSync iOS Client SDK

Our backend resources have been created and we just verified mutations and queries in the AppSync Console. Let's move onto the mobile client!

To configure the app, we'll use [Cocoapods](https://cocoapods.org/) to install the AWS SDK for iOS and AWS AppSync Client dependencies.
In the root project folder, run the following command to initialize Cocoapods.

```js
pod init
```

This will create a new `Podfile`. Open up the `Podfile` in your favorite editor and add the following dependency for adding the AWSAppSync SDK to your app:

```swift
target 'ios-amplify-app' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for ios-amplify-app
  pod 'AWSAppSync', ' ~> 2.10.0'
  
end
```

Install the AppSync iOS SDK by running:
```js
pod install --repo-update
```

## Add the `awsconfiguration.json` and `API.swift` files to your Xcode project

We need to configure our iOS Swift application to be aware of our new AWS Amplify project. We do this by referencing the auto-generated `awsconfiguration.json` and `API.Swift` files in the root of your Xcode project folder.

Launch Xcode using the .xcworkspace from now on as we are using Cocoapods.
```js
$ open ios-amplify-app.xcworkspace/
```

In Xcode, right-click on the project folder and choose `"Add Files to ..."` and add the `awsconfiguration.json` and the `API.Swift` files to your project. When the Options dialog box that appears, do the following:

* Clear the Copy items if needed check box.
* Choose Create groups, and then choose Next.

Build the project (Command-B) to make sure we don't have any compile errors.

## Integrate AppSync iOS Client into your app
#### Update AppDelegate.swift
Add the folowing three numbered code snippets to your `AppDelegate.swift` class:

```swift
import UIKit
import AWSAppSync // #1

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    
    var appSyncClient: AWSAppSyncClient? // #2

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // BEGIN #3 AppSync cliet when using Authorization Type: API key
        do {
            // Directory in which AppSync stores its persistent cache databases
            let cacheConfiguration = try AWSAppSyncCacheConfiguration()
            
            // AppSync configuration & client initialization
            let appSyncServiceConfig = try AWSAppSyncServiceConfig()
            let appSyncConfig = try AWSAppSyncClientConfiguration(appSyncServiceConfig: appSyncServiceConfig, cacheConfiguration: cacheConfiguration)
                appSyncClient = try AWSAppSyncClient(appSyncConfig: appSyncConfig)
        } catch {
            print("Error initializing appsync client. \(error)")
        }
        // END #3 AppSync
        
        return true
    }

    //...
}
```

#### Update ViewController.swift

Add the following three items to your `ViewController.swift` class:
```swift
import UIKit
import AWSAppSync // #1

class ViewController: UIViewController {

    // Reference AppSync client
    var appSyncClient: AWSAppSyncClient? // #2
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // #3
        let appDelegate = UIApplication.shared.delegate as! AppDelegate
                appSyncClient = appDelegate.appSyncClient
    }
}
```

## Add GraphQL Mutation

Add the following mutation function to your `ViewController.swift` class:

```swift
// GraphQL Mutation - Add a new talk
func createTalkMutation(){
    let conferenceInput = CreateTalkInput(name: "Monetize your iOS app", description: "How to make dough as an iOS developer", speakerName: "Steve Jobs", speakerBio: "I do cool stuff at Apple")
    appSyncClient?.perform(mutation: CreateTalkMutation(input: conferenceInput))
    { (result, error) in
        if let error = error as? AWSAppSyncClientError {
            print("Error occurred: \(error.localizedDescription )")
        }
        if let resultError = result?.errors {
            print("Error saving conf talk: \(resultError)")
            return
        }
        
        guard let result = result?.data else { return }
        
        print("Talk created: \(String(describing: result.createTalk?.id))")
    }
}
```

## Add GraphQL Query

Add the following query function to your `ViewController.swift` class:

```swift
// GraphQL Query - List all talks
func getTalksQuery(){
    appSyncClient?.fetch(query: ListTalksQuery(), cachePolicy: .returnCacheDataAndFetch) { (result, error) in
        if error != nil {
            print(error?.localizedDescription ?? "")
            return
        }
        
        guard let talks = result?.data?.listTalks?.items else { return }
        talks.forEach{ print(("Title: " + ($0?.name)!) + "\nSpeaker: " + (($0?.speakerName)! + "\n")) }
    }
}
```

### Run App and Invoke Mutation and Query Functions

In order to execute the mutation and/or query functions above, we can invoke those functions from `ViewDidLoad()` in the `ViewController.swift` class:
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    //...

    createTalkMutation()
    getTalksQuery()

    //...
}
```
Build (Command-B) and run your app.

### Add GraphQL Subscriptions

GraphQL subscriptions give us the real-time updates when data changes in our API. 
First, set the discard variable at the `ViewController.swift` class level:

```swift
// Set a discard variable at the class level
var discard: Cancellable?
```

Now add this new `subscribeToTalks()` function to your `ViewController.swift` class: 

```swift
func subscribeToTalks() {
    do {
        discard = try appSyncClient?.subscribe(subscription: OnCreateTalkSubscription(), resultHandler: { (result, transaction, error) in
            if let result = result {
                print("Subscription triggered! " + result.data!.onCreateTalk!.name + " " + result.data!.onCreateTalk!.speakerName)
            } else if let error = error {
                print(error.localizedDescription)
            }
        })
    } catch {
        print("Error starting subscription.")
    }
}
```

Finally, call the `subscribeToTalks()` from `ViewDidLoad()` in your `ViewController.swift` class:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // ...
    
    // invoke to subscribe to newly created talks
    subscribeToTalks()
}
```
> Don't forget to comment out the calls to createTalkMutation() getTalksQuery() in the ViewDidLoad() or the app will create a new talk each time it loads.

Run the mobile app and it'll then subscribe to any new talk creation. To test the real-time subscription: Leave the app running and then create a new talk via the AppSync Console through a Mutation and you should see the iOS app log the new talk via a subscription!

## Adding Authentication to Your iOS App
#### Add Cognito User Pools (Basic Auth)

Up to this point, we used API key as authorization to calling our GraphQL API. This is good for testing but we need to add authentication to provide controlled (secure) access to our AppSync GraphQL API. Back at the project folder in Terminal, let's add authentication to our project via the Amplify CLI.

```sh
amplify add auth
```

- Do you want to use default authentication and security configuration? __Default configuration__   
- How do you want users to be able to sign in when using your Cognito User Pool? __Username__   
- What attributes are required for signing up? __Email__ (keep default)

Now run the `amplify push` command and the cloud resources will be created in your AWS account.

```bash
amplify push
```

> To view the new Cognito authentication service at any time after its creation, go to the dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

### Configure iOS App - AWS Auth iOS SDK

Open up your Podfile and add the following dependencies:
```swift
target 'ios-amplify-app' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for ios-amplify-app
  pod 'AWSAppSync', ' ~> 2.10.0'         # Added previously
  pod 'AWSMobileClient', '~> 2.9.0'      # Required dependency for auth
  pod 'AWSAuthUI', '~> 2.9.0'            # Optional for drop-in UI
  pod 'AWSUserPoolsSignIn', '~> 2.9.0'   # Optional for drop-in UI
end
```

Now run a Pod install to add the new dependencies for auth:

```bash
pod install
```

### Add the AWSMobileClient
AWSMobileClient itself is a credentials provider now and will be used for all authentication needs from federating logins to issuing AWS credentials via Cognito Identity.
To add AWSMobileClient to our iOS app, we'll go into __ViewController.swift__ and first import the `AWSMobileClient`:

```swift
import AWSMobileClient
```

Next, we'll initialize the `AWSMobileClient` in the ViewDidLoad() function in the `ViewController.swift` file:

```swift
AWSMobileClient.sharedInstance().initialize { (userState, error) in
    if let userState = userState {
            switch(userState){
            case .signedIn: // is Signed IN
                print("Logged In")
                print("Cognito Identity Id (authenticated): \(String(describing: AWSMobileClient.sharedInstance().identityId))")
            case .signedOut: // is Signed OUT
                print("Logged Out")
                print("Cognito Identity Id (unauthenticated): \(String(describing: AWSMobileClient.sharedInstance().identityId))")
                DispatchQueue.main.async {
                    self.showSignIn()
                }
            default:
                AWSMobileClient.sharedInstance().signOut()
            }
        } else if let error = error {
            print(error.localizedDescription)
        }
}
```

Last, we'll add the iOS SDK drop-in UI code in the `ViewController.swift` class to show our login UI if the user is not authenticated:

```swift
// Use the iOS SDK Auth UI to show login options to user (Basic auth, Google, or Facebook)
func showSignIn() {
    AWSMobileClient.sharedInstance().showSignIn(navigationController: self.navigationController!, {
        (userState, error) in
        if(error == nil){   // Successful signin
            DispatchQueue.main.async {
                print("User successfully logged in")
            }
        }
    })
}
```

>IMPORTANT: The drop-in UI requires the use of a navigation controller to anchor the view controller. Please make sure the app has an active navigation controller which is passed to the navigationController parameter. To add a Navigation Controller, select the Main.storyboard, select the yellow button at the top bar of the View Controller UI, then select 'Editor > Embed In > Navigation Controller.

Now, we can run the app and see the Auth Drop in UI in action. This drop in UI provides a built-in UI giving users the ability to sign up, and sign in via Amazon Cognito User Pools.

You now have authentication setup in your app! Go ahead and build (Command-B) and run the app. Based on the sample code above, the first time you launch the app, you should see the UI prompt for username, email, and password. Feel free to create an account and test it out. You'll know if you have successfully logged in when the log shows successful and you'll see the default white screen.

Congratulations! You can now authenticate users.

## Update AppSync iOS SDK Client
Now that we have our user authenticating via Cognito User Pools, we need to update the AppSync client configuarion in our iOS app.



## Update Authorization Type for GraphQL API

Next, we need to update the AppSync API to use Cognito User Pools as the authorization type. Remember, we previously setup authorization via an API key for testing purposes.

To switch our GraphQL API from API key to Cognito User Pools, we'll reconfigure the existing GraphQL API using the Amplify CLI by running this command from our Xcode project folder:

```sh
amplify configure api
```
Please select from one of the below mentioned services: __GraphQL__   
Choose an authorization type for the API: __Amazon Cognito User Pool__

Next, run `amplify push` to build out the new resources:

```sh
amplify push
```

Once deployed, the AppSync GraphQL API can only be accessed via an authenticated User Pool user.

## Update the AppSync iOS Client for User Pools
Now that we are using User Pools authorization for our GraphQL API, we need to modify the AppSync client configuration.

REPLACE the existing `initializeAppSync()` function in the `ViewController` class with the following AppSync client configuration for User Pools authorization:

```swift
// Use this AppSync client configuration when using Authorization Type: User Pools
func initializeAppSync() {
  do {
      // You can choose the directory in which AppSync stores its persistent cache databases
      let cacheConfiguration = try AWSAppSyncCacheConfiguration()

      // Initialize the AWS AppSync configuration
      let appSyncConfig = try AWSAppSyncClientConfiguration(appSyncServiceConfig: AWSAppSyncServiceConfig(),
        userPoolsAuthProvider: {
          class MyCognitoUserPoolsAuthProvider : AWSCognitoUserPoolsAuthProviderAsync {
              func getLatestAuthToken(_ callback: @escaping (String?, Error?) -> Void) {
                  AWSMobileClient.sharedInstance().getTokens { (tokens, error) in
                      if error != nil {
                          callback(nil, error)
                      } else {
                          callback(tokens?.idToken?.tokenString, nil)
                      }
                  }
              }
          }
          return MyCognitoUserPoolsAuthProvider()
          }(), cacheConfiguration: cacheConfiguration)

      // Initialize the AWS AppSync client
      appSyncClient = try AWSAppSyncClient(appSyncConfig: appSyncConfig)
  } catch {
      print("Error initializing appsync client. \(error)")
  }
}
```

## Advanced Use Case for Authenticated Users

Next, let's look at how to use the identity of the user to associate items created in the database with the logged in user & then query the database using these credentials. We'll store the user's identity in the database table as userId & add a new index on the DynamoDB table to optimize our query by userId.

### Adding a New Index to the DynamoDB Table

Next, we'll want to add a new GSI (global secondary index) in the table. We do this so we can query on the index to gain new data access pattern.

To add the index, open the [AppSync Console](https://console.aws.amazon.com/appsync/home), choose your API & click on __Data Sources__. Next, click on the data source link.

From here, click on the __Indexes__ tab & click __Create index__.

For the __partition key__, input `userId` to create a `userId-index` Index name & click __Create index__.

Next, we'll update the resolver for adding talks & querying for talks.

### Updating the AppSync Resolvers for Handling Auth Users
To start passing in the userId into newly crated talks, we need to create two new request templates (resolvers) to handling the passing of the userId when adding new talks and used to retrieve talks only created by the UserId.

In your Xcode project folder __amplify/backend/api/ConferenceAPI/resolvers__, create the following two NEW resolvers:

- __Mutation.createTalk.req.vtl__

- __Query.listTalks.req.vtl__

Here's the __Mutation.createTalk.req.vtl__ code:

```vtl
$util.qr($context.args.input.put("createdAt", $util.time.nowISO8601()))
$util.qr($context.args.input.put("updatedAt", $util.time.nowISO8601()))
$util.qr($context.args.input.put("__typename", "Talk"))
$util.qr($context.args.input.put("userId", $ctx.identity.sub))

{
  "version": "2017-02-28",
  "operation": "PutItem",
  "key": {
      "id":     $util.dynamodb.toDynamoDBJson($util.defaultIfNullOrBlank($ctx.args.input.id, $util.autoId()))
  },
  "attributeValues": $util.dynamodb.toMapValuesJson($context.args.input),
  "condition": {
      "expression": "attribute_not_exists(#id)",
      "expressionNames": {
          "#id": "id"
    }
  }
}
```

Here's the __Query.listTalks.req.vtl__ code:

```vtl
{
    "version" : "2017-02-28",
    "operation" : "Query",
    "index" : "userId-index",
    "query" : {
        "expression": "userId = :userId",
        "expressionValues" : {
            ":userId" : $util.dynamodb.toDynamoDBJson($ctx.identity.sub)
        }
    }
}
```

Next, run the `amplify push` command to update the GraphQL API:

```sh
amplify push
```

> Now that we've added authentication via Cognito User Pools to the GraphQL API, the users will need to log into the app (via basic auth) in order to perform mutations or queries. 

### Testing the New Authentication in the AppSync Console
For testing in the query feature of the AppSync Console, we'll need to authenticate a User Pool user via the `Login with User Pools` button in the queries section. The auth form requires a Client Id and basic auth credentials. You can find the App client Id `_clientWeb` under `App clients` of your User Pools settings in the [Cognito User Pools Console](https://console.aws.amazon.com/cognito/users). In the AppSync Console Query section, select and paste in the `_clientWeb` value into the ClienId field and then type your User Pool `username` & `password` you created previously. If successfully logged in, you can now test mutations and queries directly from the AppSync Console!

From now on, when new talks are created, the `userId` field will be populated with the `userId` of the authenticated (logged in) user.

When we query for talks in the Console or mobile app, we will only receive the talk data for the items that were created by the authenticated user.

```graphql
query listTalks {
  listTalks {
    items {
      id
      name
      description
      speakerName
      speakerBio
    }
  }
}
```

## Multiple Amplify Project Environments

Now that we have our GraphQL API up & running with authentication, what if we wanted to update our API to test it out without it affecting our existing envrionment?

To do so, we can create a clone of our existing environment, test it out, & then deploy & test the new resources.

Once we are happy with the new feature, we can then merge it back into our main environment. Let's see how to do this!

## Removing Categories from Amplify Project

If at any time, or at the end of this workshop, you would like to delete a category from your project & your account, you can do this by running the `amplify remove` command:

```sh
amplify remove auth

amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.

## Deleting the Amplify Project

```sh
amplify delete
```
