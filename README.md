### Soap Client in go

Step 1: 

Create a python app to connect to the SOAP end point

This python app must be in python3
Install this requirement
```
suds-py3==1.4.1.0
```

Suds is a lightweight SOAP python client that provides a service proxy for Web Services.
Basic Usage
The ‘suds’ [suds.client.Client-class.html Client] class provides a consolidated API for consuming web services. The object contains (2) sub-namespaces:

service:
```
The [suds.client.Service-class.html service] namespace provides a proxy for the consumed service. This object is used to invoke operations (methods) provided by the service endpoint.
```
factory:
```
The [suds.client.Factory-class.html factory] namespace provides a factory that may be used to create instances of objects and types defined in the WSDL.
You will need to know the url for WSDL for each service used. Simply create a client for that service as follows:
```

Example:
```
from suds.client import Client
from suds.plugin import MessagePlugin

url="http://gps.rcontrol.com.mx/Tracking/wcf/RCService.svc?wsdl"
class LogPlugin(MessagePlugin):
  def sending(self, context):
    print(str(context.envelope))
  def received(self, context):
    print(str(context.reply))
client = Client(url, plugins=[LogPlugin()])
print(client)
```

Response: (The response tell us the Methods (functions to send or receive data) and the types (objects that we need to create for the methods using the suds factory)

```

Suds ( https://fedorahosted.org/suds/ )  version: 0.4 GA  build: R699-20100913
Service ( RCService ) tns="http://tempuri.org/"
   Prefixes (4)
      ns0 = "http://schemas.datacontract.org/2004/07/IronTracking"
      ns1 = "http://schemas.microsoft.com/2003/10/Serialization/"
      ns2 = "http://schemas.microsoft.com/2003/10/Serialization/Arrays"
      ns3 = "http://tempuri.org/"
   Ports (1):
      (BasicHttpBinding_IRCService)
         Methods (2):
            GPSAssetTracking(xs:string token, ns0:ArrayOfEvent events, )   <---- This method needs a token and a response called events type ns0:Arrayod Event.
            GetUserToken(xs:string userId, xs:string password, )
         Types (10):   <---- List of types
            ns0:AppointResult
            ns0:ArrayOfAppointResult
            ns0:ArrayOfEvent
            ns2:ArrayOfKeyValueOfstringstring
            ns0:Customer
            ns0:Event          <---- We need this type to make a respose to the event, for the method GPSAssetTracking
            ns0:UserTokenResult
            ns1:char
            ns1:duration
            ns1:guid

```

In this example there are 2 methods

The first method is pretty simple to obtain the token. The type to pass are strings:
```
response=client.service.GetUserToken("demo_cedis_corp","xxxxxxxx")
token=response['token']
print(token)
```

The second method is complex because we have an string an Event type object

We must create an object with the type ns0:Event
That can be acomplished using the factory from suds
```
myevent=client.factory.create('ns0:Event')
myevent.altitude=0
myevent.asset="123ABC"
myevent.battery=0
myevent.code=1
myevent.course=0
myevent.customer.id=0
myevent.customer.name=0
myevent.date="2020-02-20T13:15:22"
myevent.direction="Tlalnepantla de baz"
myevent.ignition=0
myevent.latitude="28.2882741"
myevent.longitude="-105.5069005"
myevent.odometer=0
arrayevent=client.factory.create('events')
arrayevent.Event.append(myevent)
myevent.serialNumber=0
myevent.shipment=0
myevent.speed=170
```
The reponse has an array (ArrayOfEvent)
Using the factory must create an object array of events
```
aevent=client.factory.create('ns3:events')
```

This object has a method called append to add to the array the other object
```
aevent.Event.append(myevent)
```

Now we are ready to send data we have the array 
```
response=client.service.GPSAssetTracking(token,aevent)
```

The class logplugin will provide the Input and the response

```
<?xml version="1.0" encoding="UTF-8"?><SOAP-ENV:Envelope xmlns:ns0="http://tempuri.org/" xmlns:ns1="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"><SOAP-ENV:Header/><ns1:Body><ns0:GetUserToken><ns0:userId>demo_cedis_corp</ns0:userId><ns0:password>xxxxxxxx</ns0:password></ns0:GetUserToken></ns1:Body></SOAP-ENV:Envelope>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/"><s:Body><GetUserTokenResponse xmlns="http://tempuri.org/"><GetUserTokenResult xmlns:a="http://schemas.datacontract.org/2004/07/IronTracking" xmlns:i="http://www.w3.org/2001/XMLSchema-instance"><a:exception i:nil="true" xmlns:b="http://schemas.microsoft.com/2003/10/Serialization/Arrays"/><a:token>bfe1d5d9-877d-8173-4c4f-d1589c5c896a</a:token></GetUserTokenResult></GetUserTokenResponse></s:Body></s:Envelope>
```

The response:
```
<?xml version="1.0" encoding="UTF-8"?><SOAP-ENV:Envelope xmlns:ns0="http://tempuri.org/" xmlns:ns1="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns2="http://schemas.datacontract.org/2004/07/IronTracking" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"><SOAP-ENV:Header/><ns1:Body><ns0:GPSAssetTracking><ns0:token>bfe1d5d9-877d-8173-4c4f-d1589c5c896a</ns0:token><ns0:events><ns2:Event><ns2:altitude>0</ns2:altitude><ns2:asset>123ABC</ns2:asset><ns2:battery>0</ns2:battery><ns2:code>1</ns2:code><ns2:course>0</ns2:course><ns2:customer><ns2:id>0</ns2:id><ns2:name>0</ns2:name></ns2:customer><ns2:date>2020-02-20T13:15:22</ns2:date><ns2:direction>Tlalnepantla de baz</ns2:direction><ns2:ignition>0</ns2:ignition><ns2:latitude>28.2882741</ns2:latitude><ns2:longitude>-105.5069005</ns2:longitude><ns2:odometer>0</ns2:odometer><ns2:serialNumber>0</ns2:serialNumber><ns2:shipment>0</ns2:shipment><ns2:speed>170</ns2:speed></ns2:Event></ns0:events></ns0:GPSAssetTracking></ns1:Body></SOAP-ENV:Envelope>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/"><s:Body><GPSAssetTrackingResponse xmlns="http://tempuri.org/"><GPSAssetTrackingResult xmlns:a="http://schemas.datacontract.org/2004/07/IronTracking" xmlns:i="http://www.w3.org/2001/XMLSchema-instance"><a:AppointResult><a:exception i:nil="true" xmlns:b="http://schemas.microsoft.com/2003/10/Serialization/Arrays"/><a:idJob>186283646</a:idJob></a:AppointResult></GPSAssetTrackingResult></GPSAssetTrackingResponse></s:Body></s:Envelope>

```
