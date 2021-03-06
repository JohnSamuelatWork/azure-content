<properties
   pageTitle="Get started with Reliable Actors | Microsoft Azure"
   description="This tutorial walks you through the steps of creating, debugging, and deploying a canonical HelloWorld service using Service Fabric Reliable Actors."
   services="service-fabric"
   documentationCenter=".net"
   authors="vturecek"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="11/13/2015"
   ms.author="vturecek"/>

# Reliable Actors: The canonical HelloWorld walk-through scenario
This article explains the basics of Azure Service Fabric Reliable Actors and walks you through creating, debugging, and deploying a simple HelloWorld application in Visual Studio.

## Installation and setup
Before you start, ensure that you have the Service Fabric development environment set up on your machine.
If you need to set it up, see detailed instructions on [how to set up the development environment](service-fabric-get-started.md).

## Basic concepts
To get started with Reliable Actors, you need to understand just four basic concepts:

* **Actor service**. Reliable Actors are packaged in services that can be deployed in the Service Fabric infrastructure. A service can host one or more actors. We will go into more detail about the trade-offs of one actor versus multiple actors per service below. For now, let's assume that we need to implement only one actor.
* **Actor interface**. The actor interface is used to define the public interface of an actor. In the Reliable Actor model terminology, the actor interface defines the types of messages that the actor can understand and process. The actor interface is used by other actors and client applications to "send" (asynchronously) messages to the actor. Reliable Actors can implement multiple interfaces. As we will see, a HelloWorld actor can implement the IHelloWorld interface, but it can also implement an ILogging interface that defines different messages and/or functionalities.
* **Actor registration**. In the Reliable Actors service, the actor type needs to be registered. This way, Service Fabric is aware of the new type and can use it to create new actors.
* **ActorProxy class**. The ActorProxy class is used to bind to an actor and invoke the methods exposed through its interfaces. The ActorProxy class provides two important functionalities:
	* It resolves names. It is able to locate the actor in the cluster (find the node of the cluster where it is hosted).
	* It handles failures. It can retry method invocations and re-determine the actor location after, for example, a failure that requires the actor to be relocated to another node in the cluster.

## Create a new project in Visual Studio
After you have installed the Service Fabric tools for Visual Studio, you can create new project types. The new project types are under the **Cloud** category of the **New Project** dialog box.


![Service Fabric tools for Visual Studio - new project][1]

In the next dialog box, you can choose the type of project that you want to create.

![Service Fabric project templates][5]

For the HelloWorld project, let's use the Service Fabric Reliable Actors service.

After you have created the solution, you should see the following structure:

![Service Fabric project structure][2]

## Reliable Actors basic building blocks

A typical Reliable Actors solution is composed of three projects:

* **The application project (HelloWorldApplication)**. This is the project that packages all of the services together for deployment. It contains the **ApplicationManifest.xml** and PowerShell scripts for managing the application.

* **The interface project (HelloWorld.Interfaces)**. This is the project that contains the interface definition for the actor. In the HelloWorld.Interfaces project, you can define the interfaces that will be used by the actors in the solution.

```csharp

namespace MyActor.Interfaces
{
    using System.Threading.Tasks;
    using Microsoft.ServiceFabric.Actors;

    public interface IMyActor : IActor
    {
        Task<string> HelloWorld();
    }
}

```

* **The service project (HelloWorld)**. This is the project used to define the Service Fabric service that is going to host the actor. It contains some boilerplate code that does not need to be edited in most cases (ServiceHost.cs), as well as the implementation of the actor. The implementation of the actor involves implementing a class that derives from a base type (Actor). It also implements the interface(s) that are defined in the HelloWorld.Interfaces project.

```csharp

namespace MyActor
{
    using System;
    using System.Threading.Tasks;
    using Interfaces;
    using Microsoft.ServiceFabric.Actors;

    internal class MyActor : StatelessActor, IMyActor
    {
        public Task<string> HelloWorld()
        {
            throw new NotImplementedException();
        }
    }
}

```

The Reliable Actors service project contains the code to create a Service Fabric service. In the service definition, the actor type or types are registered, so that they can be used to instantiate new actors.

```csharp

namespace MyActor
{
    using System;
    using System.Fabric;
    using System.Threading;
    using Microsoft.ServiceFabric.Actors;

    internal static class Program
    {
        private static void Main()
        {
            try
            {
                using (FabricRuntime fabricRuntime = FabricRuntime.Create())
                {
                    fabricRuntime.RegisterActor<MyActor>();

                    Thread.Sleep(Timeout.Infinite);  // Prevents this host process from terminating so services keeps running.
                }
            }
            catch (Exception e)
            {
                ActorEventSource.Current.ActorHostInitializationFailed(e.ToString());
                throw;
            }
        }
    }
}

```

If you start from a new project in Visual Studio and you have only one actor definition, the registration is included by default in the code that Visual Studio generates. If you define other actors in the service, you need to add the actor registration by using:

```csharp

fabricRuntime.RegisterActor<MyActor>();


```

## Debugging

The Service Fabric tools for Visual Studio support debugging on your local machine. You can start a debugging session by hitting the F5 key. Visual Studio builds (if necessary) packages. It also deploys the application on the local Service Fabric cluster and attaches the debugger. The experience is similar to debugging an ASP.NET application.

During the deployment process, you can see the progress in the **Output** window.

![Service Fabric debugging output window][3]


## Next steps

- [Introduction to Service Fabric Reliable Actors](service-fabric-reliable-actors-introduction.md)
- [Actors APIs reference documentation](https://msdn.microsoft.com/library/azure/dn971626.aspx)
- [Sample code](https://github.com/Azure/servicefabric-samples)


<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject.PNG
[2]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-projectstructure.PNG
[3]: ./media/service-fabric-reliable-actors-get-started/debugging-output.PNG
[4]: ./media/service-fabric-reliable-actors-get-started/vs-context-menu.png
[5]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject1.PNG
