finish

https://docs.microsoft.com/en-us/azure/event-grid/custom-event-quickstart

https://docs.microsoft.com/en-us/azure/event-grid/custom-event-quickstart

https://docs.microsoft.com/en-us/azure/event-grid/custom-event-quickstart-portal

https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-quickstart-powershell?toc=%2fazure%2fevent-grid%2ftoc.json

https://docs.microsoft.com/en-us/azure/event-grid/resize-images-on-storage-blob-upload-event?tabs=dotnet

https://docs.microsoft.com/en-us/azure/event-grid/monitor-virtual-machine-changes-event-grid-logic-app

https://docs.microsoft.com/en-us/azure/event-grid/ensure-tags-exists-on-new-virtual-machines

https://docs.microsoft.com/en-us/azure/event-grid/overview


# Configure an app or service to send emails, Event Grid, and the Azure Relay Service

## Reading material:
### Azure Relay Service
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it
https://docs.microsoft.com/en-us/dotnet/framework/wcf/whats-wcf
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-wcf-dotnet-get-started
https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-quotas
https://azure.microsoft.com/en-us/pricing/details/service-bus/
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-hybrid-connections-protocol
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-hybrid-connections-dotnet-get-started
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-hybrid-connections-http-requests-dotnet-get-started
https://docs.microsoft.com/en-us/azure/service-bus-relay/service-bus-dotnet-hybrid-app-using-service-bus-relay
https://docs.microsoft.com/en-us/azure/service-bus-relay/service-bus-relay-rest-tutorial
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-authentication-and-authorization

### Sending emails
https://docs.microsoft.com/en-us/azure/sendgrid-dotnet-how-to-send-email
https://sendgrid.com/docs/API_Reference/Web_API_v3/Mail/index.html

### Event Grid
https://docs.microsoft.com/en-us/azure/event-grid/
https://docs.microsoft.com/en-us/azure/event-grid/compare-messaging-services

## Videos

## What is SendGrid?
SendGrid powers email delivery for your whole team. Integrate via API, SMTP, or build and send email campaigns using our online email marketing tools to get started sending quickly.

## What is Event Grid?
Azure Event Grid is a fully-managed intelligent event routing service that allows for uniform event consumption using a publish-subscribe model. Event Grid is an eventing backplane that enables event-driven, reactive programming. It uses a publish-subscribe model. Publishers emit events, but have no expectation about which events are handled. Subscribers decide which events they want to handle.

## What is Azure Relay?
The Azure Relay service enables you to securely expose services that run in your corporate network to the public cloud. You can do so without opening a port on your firewall, or making intrusive changes to your corporate network infrastructure. Relay enables you to securely control who can access these services at a fine-grained level. It provides a powerful and secure way to expose application functionality and data from your existing enterprise solutions and take advantage of it from the cloud.

## What is a Hybrid Connection?
Hybrid Connections enables bi-directional, binary stream communication and simple datagram flow between two networked applications. Either or both parties can reside behind NATs or firewalls. Hybrid Connections capability of Relay is a secure, open-protocol evolution based on HTTP and WebSockets. It supersedes the former, equally named BizTalk Services feature.

## Definitions
1. Hybrid Connection: a secure, and open-protocol evolution of the Relay features that existed earlier. You can use it on any platform and in any language. It's based on HTTP and WebSockets protocols.

2. WCF: Windows Communication Foundation (WCF) is a framework for building service-oriented applications. Using WCF, you can send data as asynchronous messages from one service endpoint to another. A service endpoint can be part of a continuously available service hosted by IIS, or it can be a service hosted in an application. An endpoint can be a client of a service that requests data from a service endpoint.

3. Relay Namespace: A namespace is a scoping container that you can use to address Relay resources within your application.

4. Messaging unit: The premium tier provides resource isolation at the CPU and memory level so that each workload runs in isolation. This resource container is called a messaging unit. A premium namespace has least one messaging unit. You can select 1, 2, or 4.

5. REST Service: REpresentational State Transfer Service. An architectural style for developing web services. It builds upon existing systems and features of the internet's HTTP in order to achieve its objectives. 

## Misc notes

### Azure Relay

#### Hybrid Connections

* Hybrid Connections uses WebSockets on port 443 with SSL as the underlying transport mechanism, which uses HTTPS only.

* When a listener connects out to azure, after the listen message is accepted by azure it is held open as the control channel for enabling all subsequent interactions.

* When a sender opens a new connection on the service, the service chooses and notifies one of the active listeners on the Hybrid Connection. This notification is sent to the listener over the open control channel as a JSON message. The message contains the URL of the WebSocket endpoint that the listener must connect to for accepting the connection.

* The request/response flow uses the control channel by default, but can be "upgraded" to a distinct rendezvous WebSocket whenever required. Distinct WebSocket connections improve throughput for each client conversation, but they burden the listener with more connections that need to be handled, which may not be desire able for lightweight clients.

* On the control channel, request and response bodies are limited to at most 64 kB in size. HTTP header metadata is limited to a total of 32 kB. If either the request or the response exceeds that threshold, the listener MUST upgrade to a rendezvous WebSocket using a gesture equivalent to handling the Accept.

* The owner of the Hybrid Connection can choose to allow anonymous senders.

* The sender client comes out of the handshake with a "clean" WebSocket, which is connected to a listener and that needs no further preambles or preparation. This model enables practically any existing WebSocket client implementation to readily take advantage of the Hybrid Connections service by supplying a correctly constructed URL into their WebSocket client layer.

* Server code example

```
private static async Task RunAsync()
{
    var cts = new CancellationTokenSource();

    var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(KeyName, Key);
    var listener = new HybridConnectionListener(new Uri(string.Format("sb://{0}/{1}", RelayNamespace, ConnectionName)), tokenProvider);

    // Subscribe to the status events.
    listener.Connecting += (o, e) => { Console.WriteLine("Connecting"); };
    listener.Offline += (o, e) => { Console.WriteLine("Offline"); };
    listener.Online += (o, e) => { Console.WriteLine("Online"); };

    // Opening the listener establishes the control channel to
    // the Azure Relay service. The control channel is continuously 
    // maintained, and is reestablished when connectivity is disrupted.
    await listener.OpenAsync(cts.Token);
    Console.WriteLine("Server listening");

    // Provide callback for the cancellation token that will close the listener.
    cts.Token.Register(() => listener.CloseAsync(CancellationToken.None));

    // Start a new thread that will continuously read the console.
    new Task(() => Console.In.ReadLineAsync().ContinueWith((s) => { cts.Cancel(); })).Start();

    // Accept the next available, pending connection request. 
    // Shutting down the listener allows a clean exit. 
    // This method returns null.
    while (true)
    {
        var relayConnection = await listener.AcceptConnectionAsync();
        if (relayConnection == null)
        {
            break;
        }

        ProcessMessagesOnConnection(relayConnection, cts);
    }

    // Close the listener after you exit the processing loop.
    await listener.CloseAsync(cts.Token);
}

private static async void ProcessMessagesOnConnection(HybridConnectionStream relayConnection, CancellationTokenSource cts)
{
    Console.WriteLine("New session");

    // The connection is a fully bidrectional stream. 
    // Put a stream reader and a stream writer over it.  
    // This allows you to read UTF-8 text that comes from 
    // the sender, and to write text replies back.
    var reader = new StreamReader(relayConnection);
    var writer = new StreamWriter(relayConnection) { AutoFlush = true };
    while (!cts.IsCancellationRequested)
    {
        try
        {
            // Read a line of input until a newline is encountered.
            var line = await reader.ReadLineAsync();

            if (string.IsNullOrEmpty(line))
            {
                // If there's no input data, signal that 
                // you will no longer send data on this connection.
                // Then, break out of the processing loop.
                await relayConnection.ShutdownAsync(cts.Token);
                break;
            }

            // Write the line on the console.
            Console.WriteLine(line);

            // Write the line back to the client, prepended with "Echo:"
            await writer.WriteLineAsync($"Echo: {line}");
        }
        catch (IOException)
        {
            // Catch an I/O exception. This likely occurred when
            // the client disconnected.
            Console.WriteLine("Client closed connection");
            break;
        }
    }

    Console.WriteLine("End session");

    // Close the connection.
    await relayConnection.CloseAsync(cts.Token);
}
```

* Client code example
```
private static async Task RunAsync()
{
    Console.WriteLine("Enter lines of text to send to the server with ENTER");

    // Create a new hybrid connection client.
    var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(KeyName, Key);
    var client = new HybridConnectionClient(new Uri(String.Format("sb://{0}/{1}", RelayNamespace, ConnectionName)), tokenProvider);

    // Initiate the connection.
    var relayConnection = await client.CreateConnectionAsync();

    // Run two concurrent loops on the connection. One 
    // reads input from the console and then writes it to the connection 
    // with a stream writer. The other reads lines of input from the 
    // connection with a stream reader and then writes them to the console. 
    // Entering a blank line shuts down the write task after 
    // sending it to the server. The server then cleanly shuts down
    // the connection, which terminates the read task.

    var reads = Task.Run(async () => {
        // Initialize the stream reader over the connection.
        var reader = new StreamReader(relayConnection);
        var writer = Console.Out;
        do
        {
            // Read a full line of UTF-8 text up to newline.
            string line = await reader.ReadLineAsync();
            // If the string is empty or null, you are done.
            if (String.IsNullOrEmpty(line))
                break;
            // Write to the console.
            await writer.WriteLineAsync(line);
        }
        while (true);
    });

    // Read from the console and write to the hybrid connection.
    var writes = Task.Run(async () => {
        var reader = Console.In;
        var writer = new StreamWriter(relayConnection) { AutoFlush = true };
        do
        {
            // Read a line from the console.
            string line = await reader.ReadLineAsync();
            // Write the line out, also when it's empty.
            await writer.WriteLineAsync(line);
            // Quit when the line is empty.
            if (String.IsNullOrEmpty(line))
                break;
        }
        while (true);
    });

    // Wait for both tasks to finish.
    await Task.WhenAll(reads, writes);
    await relayConnection.CloseAsync(CancellationToken.None);
}
```

* HTTP request server code example
```
private static async Task RunAsync()
{
    var cts = new CancellationTokenSource();

    var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(KeyName, Key);
    var listener = new HybridConnectionListener(new Uri(string.Format("sb://{0}/{1}", RelayNamespace, ConnectionName)), tokenProvider);

    // Subscribe to the status events.
    listener.Connecting += (o, e) => { Console.WriteLine("Connecting"); };
    listener.Offline += (o, e) => { Console.WriteLine("Offline"); };
    listener.Online += (o, e) => { Console.WriteLine("Online"); };

    // Provide an HTTP request handler
    listener.RequestHandler = (context) =>
    {
        // Do something with context.Request.Url, HttpMethod, Headers, InputStream...
        context.Response.StatusCode = HttpStatusCode.OK;
        context.Response.StatusDescription = "OK, This is pretty neat";
        using (var sw = new StreamWriter(context.Response.OutputStream))
        {
            sw.WriteLine("hello!");
        }

        // The context MUST be closed here
        context.Response.Close();
    };

    // Opening the listener establishes the control channel to
    // the Azure Relay service. The control channel is continuously 
    // maintained, and is reestablished when connectivity is disrupted.
    await listener.OpenAsync();
    Console.WriteLine("Server listening");

    // Start a new thread that will continuously read the console.
    await Console.In.ReadLineAsync();

    // Close the listener after you exit the processing loop.
    await listener.CloseAsync();
}
```

* HTTP Connection client code example
```
private static async Task RunAsync()
{
    var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
    KeyName, Key);
    var uri = new Uri(string.Format("https://{0}/{1}", RelayNamespace, ConnectionName));
    var token = (await tokenProvider.GetTokenAsync(uri.AbsoluteUri, TimeSpan.FromHours(1))).TokenString;
    var client = new HttpClient();
    var request = new HttpRequestMessage()
    {
        RequestUri = uri,
        Method = HttpMethod.Get,
    };
    request.Headers.Add("ServiceBusAuthorization", token);
    var response = await client.SendAsync(request);
    Console.WriteLine(await response.Content.ReadAsStringAsync());
}
```

#### Service Bus Relay

* Topics/subscriptions, Sessions, transactions and de-duplication are not supported in the Basic pricing tier

* resource isolation and geo disaster recovery are only premium tier

* The relay service supports the following scenarios between on-premises services and applications running in the cloud or in another on-premises environment.

  * Traditional one-way, request/response, and peer-to-peer communication
  * Event distribution at internet-scope to enable publish/subscribe scenarios
  * Bi-directional and unbuffered socket communication across network boundaries.

* Azure Relay differs from network-level integration technologies such as VPN. An Azure relay can be scoped to a single application endpoint on a single machine.

* In the relayed data transfer pattern, the basic steps involved are:

  * An on-premises service connects to the relay service through an outbound port.

  * It creates a bi-directional socket for communication tied to a particular address.

  * The client can then communicate with the on-premises service by sending traffic to the relay service targeting that address.

  * The relay service then relays data to the on-premises service through the bi-directional socket dedicated to the client. The client doesn't need a direct connection to the on-premises service. It doesn't need to know the location of the service. And, the on-premises service doesn't need any inbound ports open on the firewall.

* Azure Relay has two features:

  * Hybrid Connections - Uses the open standard web sockets enabling multi-platform scenarios.

  * WCF Relays - Uses Windows Communication Foundation (WCF) to enable remote procedure calls. WCF Relay is the legacy relay offering that many customers already use with their WCF programming models.

* If you require .Net Core / NodeJS / RPC, you have to use a hybrid connection.

* If you require WCF, then you'll hvae to use the WCF Relay

* Hybrid connections are not charged per message like WCF Relays. They are charged per listener and per gigabyte send.

* Using WCF Relay send / response messaging will be billed as 2 seperate messages, which is cheaper then using a que, which would be charged for a send que / deque and reply que / deque, which would be 4 messages. Although the que version would have better durability among other things.

* When using WCF Relay netTCPRelay binding, all data is treated as a stream for calculating billable messages. In this case, Service Bus calculates the total amount of data sent or received via each individual relay on a 5-minute basis. Then, it divides that total amount of data by 64 KB to determine the number of billable messages for that relay during that time period.

* Service Bus configurations can be done entirely in the app.config to keep it out of the code like

```
<client>
    <endpoint name="solver" contract="Service.IProblemSolver"
              binding="netTcpRelayBinding"
              address="sb://<namespaceName>.servicebus.windows.net/solver"
              behaviorConfiguration="sbTokenProvider"/>
</client>
<behaviors>
    <endpointBehaviors>
        <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
                <tokenProvider>
                    <sharedAccessSignature keyName="RootManageSharedAccessKey" key="<yourKey>" />
                </tokenProvider>
            </transportClientEndpointBehavior>
        </behavior>
    </endpointBehaviors>
</behaviors>
```
  
  or 

```
<behaviors>
    <endpointBehaviors>
        <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
                <tokenProvider>
                    <sharedAccessSignature keyName="RootManageSharedAccessKey" key="<yourKey>" />
                </tokenProvider>
            </transportClientEndpointBehavior>
        </behavior>
    </endpointBehaviors>
</behaviors>
```

#### WCF Notes

* When you create a WCF REST-style service, you must define the contract. The contract specifies what operations the host supports. Each method in the interface corresponds to a specific service operation. The ServiceContractAttribute attribute must be applied to each interface, and the OperationContractAttribute attribute must be applied to each operation. If a method in an interface that has the ServiceContractAttribute does not have the OperationContractAttribute, that method is not exposed.

* The primary difference between a WCF contract and a REST-style contract is the addition of a property to the OperationContractAttribute: WebGetAttribute. This property enables you to map a method in your interface to a method on the other side of the interface. This example uses the WebGetAttribute attribute to link a method to HTTP GET. This enables Service Bus to accurately retrieve and interpret commands sent to the interface.

#### Sendgrid Notes

* The total size of your email, including attachments, must be less than 30MB.

* The total number of recipients must be less than 1000. This includes all recipients defined within the to, cc, and bcc parameters, across each object that you include in the personalizations array.

* The total length of custom arguments must be less than 10000 bytes.

* Unicode encoding is not supported for the from field within the personalizations array.

* The to.name, cc.name, and bcc.name personalizations cannot include either the ; or , characters.

#### Event Grid Notes

* Event Grid supports dead-lettering for events that aren't delivered to an endpoint.




### Quotas

  * Concurrent listeners per Hybrid Connection: 25

  * basic / standard namespaces per subscription: 100

  * premium namespaces per subscription: 10

  * Concurrent listeners on a relay: 25

  * Concurrent relay connections per all relay endpoints in a service namespace: 5000

  * Relay endpoints per service namespace: 10,000

  * Message size for NetOnewayRelayBinding and NetEventRelayBinding relays: 64k

  * Message size for HttpRelayTransportBindingElement and NetTcpRelayBinding relays: none

  * Relay usage: 5 billion messages, 2 million relay hours

* A Relay namespace name must be between 6 and 50 characters in length.