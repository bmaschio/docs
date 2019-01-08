# Consume a SOAP WebSevice 
This page will walk you you through the process of consuming a SOAP-based web service with Jolie.

## Web services

Interacting with Web Services usually implies reading some WSDL \(Web Service Description Language\) document. The WSDL documents are written in XML and can result complex to read and decode. Like other languages Jolie has its own tool to import  the complex structures of a WSDL into a more friendly Jolie interface.

_wsdl2jolie_ \(whose executable is installed by default in Jolie standard trunk\) is a tool that takes a URL to a WSDL document and automatically downloads all the related files \(e.g., referred XML schemas\), parses them and outputs the corresponding Jolie port/interface/data type definitions.

## Using wsdl2jolie

_jolie2wsdl_ can be executed from console with the following syntax

```text
wsdl2jolie wsdl_uri
```

`wdsl_uri` can be a URL or a file path \(in case of local usage\).

The output \(on screen\)  the tool returns is a set of service declarations \(in Jolie\) needed for invoking the web service , that can be copied and pasted to any Jolie code. 

## Full example

Let us an example of a WSDL document for a service that validates the ISBN number, the WSDL URL is `http://webservices.daehosting.com/services/isbnservice.wso?WSDL`.

Now following the _wsdl2jolie_ syntax we can execute the following command

                `wsdl2jolie http://webservices.daehosting.com/services/isbnservice.wso?WSDL` 

```jolie
Retrieving document at 'http://webservices.daehosting.com/services/isbnservice.wso?WSDL'.
type NOTATIONType:any

type IsValidISBN13Response:void {
        .IsValidISBN13Result:bool
}

type IsValidISBN13:void {
        .sISBN:string
}

type IsValidISBN10Response:void {
        .IsValidISBN10Result:bool
}

type IsValidISBN10:void {
        .sISBN:string
}

interface ISBNServiceSoapType {
RequestResponse:
        IsValidISBN13(IsValidISBN13)(IsValidISBN13Response),
        IsValidISBN10(IsValidISBN10)(IsValidISBN10Response)
}

outputPort ISBNServiceSoap {
Location: "socket://webservices.daehosting.com:80/services/isbnservice.wso"
Protocol: soap {
        .wsdl = "http://webservices.daehosting.com/services/isbnservice.wso?WSDL";
        .wsdl.port = "ISBNServiceSoap"
}
Interfaces: ISBNServiceSoapType
}

outputPort ISBNServiceSoap12 {
Location: "socket://localhost:80/"
Protocol: soap
Interfaces: ISBNServiceSoapType
}

```

The resulting output  is the Jolie equivalent of the WSDL document. The WSDL to Jolie interface trasformation has the following mapping

| WSDL | Jolie |
| :--- | :--- |
| `<types>` | `type` |
| `<messages>` | `type` |
| `<portType>` | `interface` |
| `<binding>` | `outputPort:Protocol` |
| `<service:port>` | `outputPort` |
 
 

Those `.wsdl` and `.wsdl.port` parameters are improvement to the SOAP protocol: when the output port is used for the first time, Jolie will read the WSDL document for processing information about the correct configuration for interacting with the service instead of forcing the user to manually insert it.

Once our interface is created, we can store it into a file, e.g., PrimeNumbers.iol, and use the output ports we discovered from Jolie code. As in the following:

```jolie
include "PrimeNumbers.iol"
include "console.iol"

main
{
    request.max = 27;
    GetPrimeNumbers@PrimeNumbersSoap( request )( response );
    println@Console( response.GetPrimeNumbersResult )()
}
```

Our little program will output `1,3,5,7,11,13,17,19,23`.

The comprehensive code of this example can be downloaded from the link below:

[Web Services Code Example](https://github.com/jolie/docs/blob/master/files/web-services/code/wsdl2jolie_code.zip)

Remarkably, wsdl2jolie has two benefits: it acts as a useful tool that creates the typed interface of a Web Service from Jolie and creates a more human-readable form of a WSDL document \(i.e., its Jolie form\).

## The generated document

wdsl2jolie creates a document which contains:

* the types contained into \(or referred by\) the WSDL;
* the Jolie interface with all the operation declarations;
* the Jolie outputPort ports needed for the Web Service invocation.

### Mapping

In the following table we show the mapping between WSDL elements and Jolie elements:

| WSDL | Jolie |
| :--- | :--- |
| `<types>` | `type` |
| `<messages>` | `type` |
| `<portType>` | `interface` |
| `<binding>` | `outputPort:Protocol` |
| `<service:port>` | `outputPort` |

## SOAP outputPort

The SOAP outputPorts are generated with two parameters:

* `wsdl`, which sets the location of the WSDL document;
* `wsdl.port`, which sets the WSDL port related to the current outputPort.

Plus, another parameter can be added in order to display debug messages, which is `debug`; if set to 1, all the SOAP messages of the current outputPort are displayed on the standard output.

The `wsdl` and `wsdl.port` parameters are needed for formatting the messages to and from the web service in conformance with the WSDL document.

## Jolie Metaservice

Another feature that derives from the improvement of the SOAP protocol is that now Jolie standard library _MetaService_ can act as a transparent bridge between Web Services.

Once set the `addRedirection` operation with the right protocol configuration \(e.g., the `.wsdl` and `.wsdl.port` parameters\), MetaService automatically downloads the WSDL document - which is automatically cached -, and make it callable by clients.

Hence, it becomes really easy to use libraries such as QtJolie which requires only the location of the WSDL document to enable a client to call the Web Service of interest.

Plus, using wsdl2jolie combined with other tools, such as jolie2plasma, enables to use the aforementioned Jolie intermediate representation for transforming a Web Service interface definition into one compatible with a \(KDE\) Plasma::Service XML. In the same way, C++ generators can be written for QtJolie, introducing ease and type-safeness to Web Services invocations.

So far not all the WSDL and SOAP features are supported, which can raise compatibility problems when using them:

* SOAP 1.2, currently NOT supported;
* XML Schema Extended types, currently NOT supported;
* HTTP GET and HTTP POST, currently HALF supported as Web Service calls.

