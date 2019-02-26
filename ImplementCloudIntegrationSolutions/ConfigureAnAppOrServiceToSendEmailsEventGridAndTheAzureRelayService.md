finish

https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-hybrid-connections-dotnet-get-started

https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-hybrid-connections-http-requests-dotnet-get-started

https://docs.microsoft.com/en-us/azure/service-bus-relay/service-bus-dotnet-hybrid-app-using-service-bus-relay

https://docs.microsoft.com/en-us/azure/service-bus-relay/service-bus-relay-rest-tutorial

https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-authentication-and-authorization


# Configure an app or service to send emails, Event Grid, and the Azure Relay Service

## Reading material:
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it
https://docs.microsoft.com/en-us/dotnet/framework/wcf/whats-wcf
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-wcf-dotnet-get-started
https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-quotas
https://azure.microsoft.com/en-us/pricing/details/service-bus/
https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-hybrid-connections-protocol


## Videos

## What is Azure Relay?
The Azure Relay service enables you to securely expose services that run in your corporate network to the public cloud. You can do so without opening a port on your firewall, or making intrusive changes to your corporate network infrastructure. Relay enables you to securely control who can access these services at a fine-grained level. It provides a powerful and secure way to expose application functionality and data from your existing enterprise solutions and take advantage of it from the cloud.

## What is a Hybrid Connection?
Hybrid Connections enables bi-directional, binary stream communication and simple datagram flow between two networked applications. Either or both parties can reside behind NATs or firewalls. Hybrid Connections capability of Relay is a secure, open-protocol evolution based on HTTP and WebSockets. It supersedes the former, equally named BizTalk Services feature.

## Definitions
1. Hybrid Connection: a secure, and open-protocol evolution of the Relay features that existed earlier. You can use it on any platform and in any language. It's based on HTTP and WebSockets protocols.

2. WCF: Windows Communication Foundation (WCF) is a framework for building service-oriented applications. Using WCF, you can send data as asynchronous messages from one service endpoint to another. A service endpoint can be part of a continuously available service hosted by IIS, or it can be a service hosted in an application. An endpoint can be a client of a service that requests data from a service endpoint.

3. Relay Namespace: A namespace is a scoping container that you can use to address Relay resources within your application.

4. Messaging unit: The premium tier provides resource isolation at the CPU and memory level so that each workload runs in isolation. This resource container is called a messaging unit. A premium namespace has least one messaging unit. You can select 1, 2, or 4.

5. 

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

* Quotas

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

* Service Bus configurations can be done entirely in the app.config to keep it out of the code like

  `<client>
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
</behaviors>`
  
  or 

  `<behaviors>
    <endpointBehaviors>
        <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
                <tokenProvider>
                    <sharedAccessSignature keyName="RootManageSharedAccessKey" key="<yourKey>" />
                </tokenProvider>
            </transportClientEndpointBehavior>
        </behavior>
    </endpointBehaviors>
</behaviors>`