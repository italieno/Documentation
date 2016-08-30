## Deltatre Push Sdk

A Push Notifications Client Api to integrate easily with main push notification providers like [Urban Airship](https://www.urbanairship.com/) and Others

### Install Deltatre.Push via NuGet

If you want to include Deltatre.Push in your project, you can [install it directly from Deltatre NuGet](http://nuget.deltatre.it/packages/Deltatre.Push/)

To install Deltatre.Push run the following command in the Package Manager Console

```
PM> Install-Package Deltatre.Push
```

And if you are using Windsor Castle  as your preferred Inversion of Control container:

```
PM> Install-Package Deltatre.Push.Windsor
```

###Urban Airship Configuration

API requests are identified using HTTP basic authentication. The username portion is the application key. The password portion is either the application secret, or master secret. The application secret is restricted to certain low-security APIs, as it is intended to be included in the distributed application. The master secret is needed for sending messages and other high-value requests, and as such must be guarded carefully.

In your **app/web configuration file** please add the following sections

```xml
<configSections>
    <section name="UrbanAirship" type="Deltatre.Push.Infra.Configuration.UrbanAirshipSection, Deltatre.Push.Infra" />
</configSections>
<UrbanAirship applicationKey="[AppKey]" applicationSecret="[AppMasterSecret]" />
```
Key, Secret and Master Secret are available in the APIs & Integrations section of your application 

###Sending a Push Notification via Urban Airship

Simple Console Application reading user input and sending a message to a Tag

```csharp

namespace ConsoleApplication
{
    class Program
    {
        static void Main(string[] args)
        {
            MainAsync().Wait();
        }

        static async Task MainAsync()
        {
            Console.WriteLine("console application started");

			//Resolve all internal api dependencies
            var resolver = new UrbanAirshipApiWindsorResolver();
            var container = resolver.Resolve();
			
			//Instanciate a new api service
            var api = container.Resolve<IUrbanAirshipApi>();
            
            Console.WriteLine("write a message and press any key to send a push notification...");
            var message = Console.ReadLine();
            Console.WriteLine("sending...");

			//Set all push notification payload attributes
            var pushParameters = new PushParameters()
            {
                Message = message,
                DeviceTypes = new List<DeviceType>() { DeviceType.All },
                AudienceSelector = new AudienceSelector().AddTag("tag1", "grp1")
            };
			
			//send the push notification
            var operation = await api.PushAsync(pushParameters);

			//check result
            Console.WriteLine(operation.OperationStatus == OperationStatus.Succeeded ? "OK" : "Error");
            Console.ReadLine();
        }
    }
}

```
here under few test to understand better how an [audience selector](http://docs.urbanairship.com/api/ua.html#audience-selection) works


```csharp
[Test]
public void AndAudience_ShouldAddATagAndNotAnAlias()
{
    var tagSelector = new AudienceSelector().AddTag("Tag1");
    var notAliasSelector = new AudienceSelector().NotAudience(new AudienceSelector().AddAlias("NoTag"));
    var compound = _sut.AndAudience(new List<AudienceSelector>() { tagSelector, notAliasSelector });

    //check result serialization
    var result = JsonSerializer.Serialize(compound);
    Assert.AreEqual(@"{""AND"":[{""tag"":""Tag1""},{""NOT"":{""alias"":""NoTag""}}]}", result);
}

#region "Test created to simulate API WebSites Examples http://docs.urbanairship.com/api/ua.html#logical-expressions"

[Test]
public void OrAudience_ShouldAdd3TagsInOr()
{
    var selectorTag1 = new AudienceSelector().AddTag("apples");
    var selectorTag2 = new AudienceSelector().AddTag("oranges");
    var selectorTag3 = new AudienceSelector().AddTag("bananas");

    var compound = _sut.OrAudience(new List<AudienceSelector>(){ selectorTag1, selectorTag2, selectorTag3 });
    
    //check result serialization
    var result = JsonSerializer.Serialize(compound);
    Assert.AreEqual(@"{""OR"":[{""tag"":""apples""},{""tag"":""oranges""},{""tag"":""bananas""}]}", result);
}
```


