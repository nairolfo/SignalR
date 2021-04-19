# Centro Real-Time Events and Hubs

Centro provides real-time communication with [ASP.NET SignalR](https://docs.microsoft.com/en-us/aspnet/signalr/overview/getting-started/introduction-to-signalr#what-is-signalr). It supports WebSocket and fallback to other mechanisms when that technology isn't supported on the client or server.

Note: Centro uses ASP.NET SignalR, not ASP.NET Core SignalR.

The following sections describe the [SignalR Hubs](https://docs.microsoft.com/en-us/aspnet/signalr/overview/getting-started/introduction-to-signalr#connections-and-hubs) used by Centro for client/server communication.

## Centro Hubs

#### ResourceEventHub

Provides real-time events for Catalog Resources. Exposes the following server methods:

- `SubscribeToPart(string partKey)`

    Called by clients to subscribe and receive real-time events when Resources in the specified Catalog Part are updated. Possible updates are listed in [ResourceUpdateEventType.cs](./ResourceUpdateEventType.cs).

- `UnsubscribeFromPart(string partKey)`

    Called by clients to unsubscribe from the specified Catalog Part.

- `Update(ResourceUpdateModel update)`

    Called by the server to dispatch real-time event messages for Jobs updates to subscribed clients. Clients must register an event handler matching this method to receive messages.

    Details for the update model can be found in [ResourceUpdateModel.cs](./ResourceUpdateModel.cs).

#### JobEventHub

Provides real-time events for Pipeline Jobs. Exposes the following server methods:

- `SubscribeToPipeline(string pipeId)`

    Called by clients to subscribe and receive real-time events when Jobs in the specified Pipeline are updated.

- `UnsubscribeFromPipeline(string pipeId)`

    Called by clients to unsubscribe from the specified Pipeline.

- `Update(MiniJob message)`

    Called by the server to dispatch real-time event messages for Jobs updates to subscribed clients. Clients must register an event handler matching this method to receive messages.

    Details for the update model can be found in [MiniJob.cs](../../Core/Models/Job/MiniJob.cs).

## Usage

This section contains short examples showing how to use the Centro Hubs.

Centro Hubs are accessed at http://localhost/Centro/signalr and require authorization.

#### .NET Client (C#)

Connecting to a hub to receive real-time updates involves the following steps:

1. Create and setup a `HubConnection`
1. Create a proxy to communicate with the desired Hub
1. Register an event handler with the proxy
1. Start the connection

Instances of `HubConnection` and proxies are usually long-lived. You use the `HubConnection` instance to control the connection to the server and the proxy instances to interact with the Centro Hubs.
You dispose the instance of `HubConnection` when you don't need it anymore.

Example:

```csharp
var hub = new HubConnection("http://localhost/Centro", true))
// Dispose hub once you don't need it anymore

IHubProxy resourceHubProxy = hub.CreateHubProx("ResourceEventHub");

var handler = resourceHubProxy.On<object>("Update", (object updateContent) =>
{
    // Handle updates
});

hub.Headers.Add("Authorization", "<TOKEN_TYPE> <TOKEN>");

// Start the connection
try
{
    await hub.Start();
}
catch (Exception ex)
{
    // Cannot connect
}

// Subscribe to/Unsubscribe from Catalog Parts at any point
resourceHubProxy.Invoke("SubscribeToPart", "1882708");
resourceHubProxy.Invoke("UnsubscribeFromPart", "1882708");
```

Detailed documentation: [ASP.NET SignalR Hubs API Guide - .NET Client (C#)](https://docs.microsoft.com/en-us/aspnet/signalr/overview/guide-to-the-api/hubs-api-guide-net-client)
