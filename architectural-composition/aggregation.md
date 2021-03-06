# Aggregation

## Aggregation

Aggregation is a generalisation of network proxies that allow a service to expose operations without implementing them in its behaviour, but delegating them to other services. Aggregation can also be used for programming various architectural patterns, such as load balancers, reverse proxies, and adapters.

The syntax for aggregation extends that given for input ports.

```text

inputPort id {
    Location: URI
    Protocol: p
    Interfaces: iface_1, ..., iface_n
    [ Aggregates: outputPort_1, outputPort_2, ... ]
}
```

Where the `Aggregates` primitive expects a list of output port names.

We can now define how aggregation works. Given IP as an input port, whenever a message for operation OP is received through IP, we have three scenarios:

* OP is an operation declared in one of the interfaces of IP. In this case, the message is normally received by the program.
* OP is not declared in one of the interfaces of IP and is declared in the interface of an output port \(OP\_port\) aggregated by IP. In this case the message is forwarded to OP\_port port as an output from the aggregator.
* OP is not declared in any interface of IP or of its aggregated output ports. Then, the message is rejected and an `IOException` fault is sent to the caller.

We can observe that in the second scenario aggregation _merges_ the interfaces of the aggregated output ports and makes them accessible through a single input port. Thus, an invoker would see all the aggregated services as a single one.

Remarkably, aggregation handles the request-response pattern seamlessly: when forwarding a request-response invocation to an aggregated service, the aggregator will automatically take care of relaying the response to the original invoker.

## The forwarder, an Aggregation example

Aggregation can be used for system integration, e.g., bridging services that use different communication technologies or protocols. The deployment snippet below creates a service that forwards incoming SODEP calls on TCP port 8000 to the output port `MyOP`, converting the received message to SOAP.

```text
outputPort MyOP {
    Location: "socket://someurl.ex:80"
    Protocol: soap
    Interfaces: MyInterface
}

inputPort MyInput {
    Location: "socket://localhost:8000"
    Protocol: sodep
    Aggregates: MyOP
}
```

## Aggregation and embedding

We give an example where three services - `A`, `B`, and `C` - are aggregated by a service `M`, which also embeds `C`.

```text
outputPort A {
    Location: "socket://someurlA.com:80/"
    Protocol: soap
    Interfaces: InterfaceA
}

outputPort B {
    Location: "socket://someurlB.com:80/"
    Protocol: xmlrpc
    Interfaces: InterfaceB
}

outputPort C {
    Interfaces: InterfaceC
}

embedded {
    Java: "example.serviceC" in C
}

inputPort M {
    Location: "socket://urlM.com:8000/"
    Protocol: sodep
    Aggregates: A, B, C
}
```

The code for aggregating services abstracts their actual deployment and remains the same either it is an embedded service \(C\) or an "external" one \(A,B\); this abstraction is achieved by setting the aggregation definition on output ports, which uncouples from it both the implementation and the location of the target service.

![](../.gitbook/assets/aggregation_1.png)

**Fig.1** The aggregator `M` exposes the union of all the interfaces of the services it aggregates \(`A`, `B`, `C`\). Service `C` executes inside the virtual machine of `M`, by embedding. Interfaces are represented with dotted rectangles

The obtained architectures is graphically represented in Fig.1, where we assume that the aggregated interfaces are singletons.

The grey arrows represent how messages will be forwarded. E.g., an incoming message for operation `op3` will be forwarded to the embedded service `C`.

