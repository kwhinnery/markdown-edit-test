
Twilio Mini Hack
===
Welcome fine adventurer to the Twilio Mini Hack.  This simple hack will introduce you to Twilio Client which allow you to easily embed Voice over IP communication directly into your native iOS and Android applications.  

Completing this hack means you will have built a simple app that will make a VoIP phone call from inside of your your iOS or Android application to a PSTN  phone like your cell phone.

Sounds amazing doesn't it?  We think so too!  

We think you'll be able to complete this mini hack in 20 minutes.  If you get stuck or have any questions, no problem.  Head over to the Twilio booth and we'll be happy to walk through some code with you.

Alright.  With the intro out of the way, lets gets get building!

Getting Started
===
To get started you'll need to set up a bit of infrastructure.

As you might expect, the first thing you'll need is a Twilio account.  Don't worry, trial accounts are free so if you don't already have one, [head on over to the Twilio website]() and sign up.  I'll wait right here while you do it.

Your back! Fantastic. Lets move on to the second thing you'll need.  

Now, we know you can't wait to make your phone ring (we can't either!) so to get you moving fast we've set up a really simple starter solution for you.  If you're reading this that means you've already found the Github repo, so go ahead and clone it to your machine or download the ZIP.

In the solution you'll find three projects:

- **TwilioMiniHackStarter.Web** - An ASP.NET website which you will use to generate a Capability Token (we'll talk more about that in a minute)
- **TwilioMiniHackStarter.iOS** - A standard iPhone Single View application project
- **TwilioMiniHackStarter.Android** - A standard Android Application project

You'll use at least two of these projects for the hack, maybe all three if you're feeling especially adventurous.


TwiML and TwiML Applications
===
Brilliant.  Now that you've got the infrastructure for this hack in place, you're just about ready to make your first phone VoIP call to Twilio.  Lets take a minute to understand what happens when you make an inbound call to Twilio.

For all inbound communication to Twilio, whether voice, VoIP or Messaging, Twilio uses a set of simple XML commands called [TwiML](https://www.twilio.com/docs/api/twiml) to provide an experience to the inbound caller, or respond to the inbound message.

We get that TwiML by making an HTTP request to a URL that you have associated with either your [Twilio phone number](https://www.twilio.com/user/account/phone-numbers/incoming), or in the case of a VoIP call using Twilio Client (which does not use phone numbers), a [TwiML application](https://www.twilio.com/user/account/apps).

![Twilio Request](http://i.imgur.com/vLo21E8.png)

![Twilio Response](http://i.imgur.com/KpBHXp8.png)

Awesome.  Now that you've got a basic understanding of how Twilio works, lets get to the code.

Configuring Twilio Capabilities
===
A Twilio Client application actually starts with a bit of server-side code that generates a [Capability Token](https://www.twilio.com/docs/client/capability-tokens).  

A Capability Token is a simple encrypted string that lets an instance of Twilio Client authenticate with Twilio and tell Twilio if its allowed to make outbound or receive inbound calls.

To generate the capability token you'll use the [ASP.NET Website project](http://link to github) that is included in the starter solution:

![ASP.NET Website Project](http://i.imgur.com/yMA6ZLY.png)

In that project, open up the `ClientController.cs` file.  You'll see we've already added the code needed for generating a Capability Token using the [Twilio Client helper library](https://www.nuget.org/packages/Twilio.Client/) which we added via NuGet.  

    public ActionResult Token()
    {
		var capabilities = new TwilioCapability (
			                   "[REPLACE_WITH_YOUR_TWILIO_ACCOUNT_SID]", 
			                   "[REPLACE_WITH_YOUR_TWILIO_AUTH_TOKEN]");

		capabilities.AllowClientIncoming ("[REPLACE_WITH_A_CLIENT_NAME]");
		capabilities.AllowClientOutgoing ("[REPLACE_WITH_YOUR_TWIML_APPLICATION_SID]");
		var token = capabilities.GenerateToken ();

        return Content (token);
    }
        
As you can see there are some strings you'll need to replace.

You can find your Twilio Account Sid and Auth Token in your [Twilio dashboard](https://www.twilio.com/user/account):

![Twilio Dashboard](http://i.imgur.com/GRXNNg5.png)

Give your Twilio client a name by replacing the string parameter in the `AllowIncomingCalls` method:

    capabilities.AllowClientIncoming ("JohnSmith");

Finally, drop in a TwiML Application Sid in the `AllowOutgoingCalls` method.  

You can create a new TwiML Application Sid by heading over to the [TwiML Application tab in the Twilio Dashboard](https://www.twilio.com/user/account/apps).  Configure your TwiML Applications Voice Request URL with the following:

`http://twiliominihack.azurewebsites.net/Client/Bridge`

Loading this URL in a browser lets you see the TwiML its generating.  

    <Response>
        <Dial callerId="[SOURCE]">[TARGET]</Dial>
    </Response>
    
This TwiML uses the [`<Dial>`](https://www.twilio.com/docs/api/twiml/dial) verb to tell Twilio to connect the incoming VoIP call from your mobile application to an outbound phone call made to another phone number, like your mobile phone.

Right now the phone number to dial is represented by the `[TARGET]` placeholder.  In the next section, you'll learn how to pass that phone number to Twilio dynamically from your mobile app.

Once you've saved your TwiML application, the Sid will be displayed:

![TwiML Application](http://i.imgur.com/jaxsdVD.png)

Awesome!  Thats all there is to generating a capability token.

Go ahead and click the `Debug` button to run the website project from Xamarin Studio.  Once the browser loads you should see the capability token being output in the browser: 

![Capability Token loaded in the browser](http://i.imgur.com/6u2etir.png)

Once its working for you, leave the website running with the debugger and move on to the next section of the hack where you will build your mobile app.

Building a VoIP App
===
Now that you're generating a capability token, you're ready to start building a mobile application that can make an outbound VoIP call.

Because that application needs the Capability Token, you will need to keep the website running in order for the application to request it.  There are a several of options for doing this including:

1. Deploy the site to a local web server or a remote web host
2. Using Xamarun Studio's built in web server.  Using this option will require you to open a second instance of Xamarin studio which you can do using this script, or from the Terminal:

        open -n /Applications/Xamarin\ Studio.app

Twilio provides Twilio Client components for iOS and Android applications, so jump to your favorite platform to get started coding:

- [iOS](#ios)
- [Android](#android)

iOS
====
<a name="ios"></a>Alright iOS ninja, are you ready to build a VoIP enable application?  Fantastic!

Start by finding the iOS project we've provided in the hack solution:

![iOS project highlight](http://i.imgur.com/Vt6jZve.png)

There is nothing special about this project (yet), its literally just the default project template.  Start to make it more awesome by adding the Twilio Client for iOS component.  Check out [this blog post for step-by-step installation instructions](https://www.twilio.com/blog/2014/08/twilio-client-for-xamarin-part-1-introduction.html).

Once you add the component open up the `TwilioMiniHackStarter.iPhoneViewController.cs` and add a new using statement:

    using TwilioClient.iOS;

and a couple of class-level variables:  

	TCDevice _device;
	TCConnection _connection;

Now you are ready to initialize your instance of Twilio Client by create a new instance of the `TCDevice` object, passing into its constructor a Capability Token retrieved from the web site created earlier:

	public async override void ViewDidLoad ()
	{
		base.ViewDidLoad ();
			
		// Create an HTTPClient object and use it to fetch
		// a capability token from our website. 
		var client = new HttpClient ();
		var token = await client.GetStringAsync("http://localhost:8080/Client/Token");

		// Create a new TCDevice object passing in the token.
		_device = new TCDevice (token, null);
	}

Once you initialize Twilio Client, its ready to make an outbound call to Twilio.  

Add a button to the application UI and in its TouchUpInside event call the `TCDevice` instances `Connect` method.  The `Connect` method accepts a dictionary of parameters which you can use to pass arbitrary parameters to your TwiML Application.  

	partial void Call_TouchUpInside (UIButton sender)
	{
		var parameters = NSDictionary.FromObjectsAndKeys(
			new object[] { "[REPLACE_WITH_YOUR_MOBILE_PHONE_NUMBER]", "Target" },
			new object[] { "[REPLACE_WITH_YOUR_TWILIO_PHONE_NUMBER]", "Source"}			);

		if (_device != null && _device.State == TCDeviceState.Ready)
		{
			_connection = _device.Connect(parameters, null);
		}
	}
		
The snippet above passes in two parameters, `Target` and `Source`.  `Target` represents the phone number you want Twilio to connect this instance of Twilio Client to, for example your mobile phone number.  `Source` represents the the phone number that we want Twilio to use as the Caller ID when it calls the Target.

Now that your app is calling the `Connect` method, it's a great time to give it a test.  Launch the app in the iOS simulator and give that button a tap.  You should hear Twilio Client connect to Twilio and, if you are using an upgraded Twilio account, your mobile phone should start to ring.  Go ahead and answer that call and enjoy knowing you've made your first VoIP to Voice phone call using Twilio Client.

If, however, you are using a Twilio trial account, Twilio will prompt you to press a key to continue the call.  Obviously since this is a custom application there are no keys to press, so you need to make one, which is pretty easy to do.

Add another button to your application UI and in its `TouchUpInside` event, call the `Connection` instances `SendDigits` method, passing a string containing a number from 1 to 9:

	partial void SendDtmf_TouchUpInside (UIButton sender)
	{
		if (_connection != null && _connection.State == TCConnectionState.Connected) {
			_connection.SendDigits("1");
		}
	}

`SendDigits` lets you tell Twilio client to simulate pressing keys on a phones dial pad.  

Go ahead and give your app a second test run, and this time when Twilio prompts you to press a key, tap that second button and wait for your mobile phone to ring.

Amazing.  In just a few minutes you've used Twilio and Twilio Client to build a custom iOS application with embedded VoIP capabilities.  

Head over to the Twilio booth and give us a show.  Make our cell phones ring and we'll gladly hand over.

Android
====
<a name="android"></a>OK Android warrior, lets build a VoIP enabled app!

Start by finding the Android project we've provided in the hack solution:

![Android project highlight](http://i.imgur.com/htEztPp.png)

There is nothing special about this project (yet), it's literally just the default project template.  Start to make it more awesome by adding the Twilio Client for Android component.  Check out [this blog post for step-by-step installation instructions](https://www.twilio.com/blog/2014/08/twilio-client-for-xamarin-part-1-introduction.html).

Once you add the component open up the `MainActivity.cs` and add a new using statement:

    using TwilioClient.Android;

and a couple of class-level variables:  

	private Device _device;
	private IConnection _connection;

Now you are ready to initialize your instance of Twilio Client which you can do by calling the static `Initialize` method on the `Twilio` class.

    protected override void OnCreate (Bundle bundle)
	{
		base.OnCreate (bundle);

		// Set our view from the "main" layout resource
		SetContentView (Resource.Layout.Main);

		Twilio.Initialize (this.ApplicationContext, this);
	}

Now override the `OnInitialized` method which is called by the Initialize method.  In `OnInitialized` you can get a Capability Token from the website you created earlier and then create a new `TCDevice` object:

	public async void OnInitialized ()
	{
		try {
			var client = new HttpClient ();
			var token = await client.GetStringAsync ("http://localhost:8080/Client/Token");

			_device = Twilio.CreateDevice(token, null);
			
		} catch (Exception ex) {
			Log.Info(TAG, "Error: " + ex.Message);
		}
	}

Once you initialize Twilio Client, its ready to make an outbound call to Twilio.  

Add a button to the application UI and in its Click event call the `TCDevice` instances `Connect` method.  The Connect method accepts a dictionary of parameters which you can use to pass arbitrary parameters to your TwiML Application.  

	// Get our button from the layout resource,
	// and attach an event to it
	Button call = FindViewById<Button> (Resource.Id.callButton);
	call.Click += delegate {
		
		if(_device != null && _device.GetState() == Device.State.Ready) 
		{
			var parameters = new Dictionary<string, string> () {
				{ "Target", "[REPLACE_WITH_YOUR_MOBILE_PHONE_NUMBER]"},
				{ "Source", "[REPLACE_WITH_YOUR_TWILIO_PHONE_NUMBER]"}			};
				
			_connection = _device.Connect (parameters, null);
		}
	};	
		
The snippet above passes in two parameters, `Target` and `Source`.  `Target` represents the phone number you want to Twilio to connect this instance of Twilio Client to, for example your mobile phone number.  `Source` represents the the phone number that we want Twilio to use as the Caller ID when it calls the Target.

Now that your app is calling the `Connect` method, its a great time to give it a test.  Launch the app in the Android simulator and give that button a tap.  You should hear Twilio Client connect to Twilio and, if you are using an upgraded Twilio account, your mobile phone should start to ring.  Go ahead and answer that call and enjoy knowing you've made your first VoIP to Voice phone call using Twilio Client.

If, however, you are using a Twilio trial account, Twilio will prompt you to press a key to continue the call.  Obviously since this is a custom application there are no keys to press, so you need to make one, which is pretty easy to do.

Add another button to your application UI and in its `Click` event, call the `Connection` instances `SendDigits` method, passing a string containing a number from 1 to 9:

	Button sendDtmf = FindViewById<Button> (Resource.Id.sendDtmf);
	sendDtmf.Click += delegate {
		if (_connection != null && _connection.State == ConnectionState.Connected)
		{
			_connection.SendDigits("1");
		}
	};

`SendDigits` lets you tell Twilio client to simulate pressing keys on a phones dial pad.  

Go ahead and give your app a second test run, and this time when Twilio prompts you to press a key, tap that second button and wait for your mobile phone to ring.

Amazing.  In just a few minutes you've used Twilio and Twilio Client to build a custom Android application with embedded VoIP capabilities.  

Head over to the Twilio booth and give us a show.  Make our cell phones ring and we'll gladly hand over.
