### Table of contents

* [Principle of Operation](REST-Gateway-0.0.4#principle-of-operation) 
  * [Consumer Gateway](REST-Gateway-0.0.4#consumer-gateway)
  * [Provider Gateway](REST-Gateway-0.0.4#provider-gateway)
* [Software Requirements](REST-Gateway-0.0.4#software-requirements)
* [Installation](REST-Gateway-0.0.4#installation)
  * [Consumer Gateway](REST-Gateway-0.0.4#consumer-gateway-1)
  * [Provider Gateway](REST-Gateway-0.0.4#provider-gateway-1)
* [Consumer Gateway Configuration](REST-Gateway-0.0.4#consumer-gateway-configuration)
* [Provider Gateway Configuration](REST-Gateway-0.0.4#provider-gateway-configuration)
* [Usage Examples](REST-Gateway-0.0.4#usage)
  * [REST API for City of Helsinki Service Map - List of Organizations](REST-Gateway-0.0.4#rest-api-for-city-of-helsinki-service-map---list-of-organizations)
    * [Consumer Gateway](REST-Gateway-0.0.4#consumer-gateway-2)
    * [Provider Gateway](REST-Gateway-0.0.4#provider-gateway-2)
  * [REST API for City of Helsinki Service Map - Single Organization](REST-Gateway-0.0.4#rest-api-for-city-of-helsinki-service-map---single-organization)
    * [Consumer Gateway](REST-Gateway-0.0.4#consumer-gateway-3)
    * [Provider Gateway](REST-Gateway-0.0.4#provider-gateway-3)
  * [Open Weather Map](REST-Gateway-0.0.4#open-weather-map)
    * [Consumer Gateway](REST-Gateway-0.0.4#consumer-gateway-4)
    * [Provider Gateway](REST-Gateway-0.0.4#provider-gateway-4)
  * [Finnish Patent and Registration Office - Business Information Search](REST-Gateway-0.0.4#finnish-patent-and-registration-office---business-information-search)
    * [Consumer Gateway](REST-Gateway-0.0.4#consumer-gateway-5)
    * [Provider Gateway](REST-Gateway-0.0.4#provider-gateway-5)
  * [Finnish Patent and Registration Office - Get Company](REST-Gateway-0.0.4#finnish-patent-and-registration-office---get-company)
    * [Consumer Gateway](REST-Gateway-0.0.4#consumer-gateway-6)
    * [Provider Gateway](REST-Gateway-0.0.4#provider-gateway-6)
  * [Finto : Finnish Thesaurus and Ontology Service - Search](REST-Gateway-0.0.4#finto--finnish-thesaurus-and-ontology-service---search)
    * [Consumer Gateway](REST-Gateway-0.0.4#consumer-gateway-6)
    * [Provider Gateway](REST-Gateway-0.0.4#provider-gateway-6)

### Principle of Operation

This is the fourth version of REST Gateway component that sits between X-Road security server and a REST service ([diagram](https://raw.githubusercontent.com/educloudalliance/xroad-rest-gateway/master/images/message-sequence_rest-gateway-0.0.4.png)). The component implements X-Road v6 [SOAP profile](https://confluence.csc.fi/download/attachments/47580926/xroad_profile_of_soap_messages_0%205.pdf?version=1&modificationDate=1415865090158&api=v2) and is tested with X-Road v6 pre-beta. The component includes the following features:

* **Provider Gateway** : access REST services (JSON, XML) via WSDL defined X-Road services
  * supported HTTP verbs: ```GET```, ```POST```, ```PUT``` and ```DELETE```
  * WSDL must be created manually
  * REST response can be wrapped in SOAP body or SOAP attachment
  * X-Road SOAP-headers are passed via HTTP headers (X-XRd-Client, X-XRd-Service, X-XRd-UserId, X-XRd-MessageId)
* **Consumer Gateway** : access WSDL defined X-Road services in a RESTful manner
  * full support for services published through Provider Gateway
  * limited support for legacy services (only services which requests don't contain nested elements - all the request parameters are ```request``` element's children)
  * supported HTTP verbs: ```GET```, ```POST```, ```PUT``` and ```DELETE```
  * reformatting of resource links
  * response's content type is given using Accept header (```text/xml```, ```application/json```)
    * if Provider Gateway returns the response as SOAP attachment, the value of HTTP Accept header is ignored and the response is returned in the original format
  * X-Road SOAP headers are passed via HTTP headers (X-XRd-UserId, X-XRd-MessageId)
  * support for browser-based access (**N.B.** reformatting of resource links not supported)
    * Service id and X-Road SOAP headers can be given as URL parameters
* automatic conversions: JSON -> XML, XML -> JSON
* new REST services can be added through configuration - no coding needed

#### Consumer Gateway

Consumer Gateway accepts HTTP GET, POST, PUT and DELETE requests, and it translates them to SOAP messages following the X-Road v6 SOAP profile. For example:

```
[GET] http://www.example.com/rest-gateway-0.0.4/Consumer/www.restservice.com/id?param1=value1&param2=value2
```
```
<request>
    <resourceId>id</resourceId>
    <param1>value1</param1>
    <param2>value2</param2>
</request>
```

If HTTP request (POST / PUT / DELETE) contains a request body, the body is passed as a SOAP attachment using the format in which it is received from the client. No conversions are applied to the request body, which means that the client must send the body in the format required by the service provider. An empty ```RESTGatewayRequestBody``` element is added to the SOAP request to indicate that the request body is passed as an attachment. For example:

```
Content-Type: application/json
[POST] http://www.example.com/rest-gateway-0.0.4/Consumer/www.restservice.com/id?param1=value1&param2=value2
Request body: {"id":1,"name":"Test name"}
```
```
.
.
.
<request>
    <resourceId>id</resourceId>
    <param1>value1</param1>
    <param2>value2</param2>
    <RESTGatewayRequestBody href="RESTGatewayRequestBody" />
</request>
.
.
.
------=_Part_1_1325547227.1416893406358
Content-Type: application/json
Content-ID: RESTGatewayRequestBody

{"id":1,"name":"Test name"}
------=_Part_1_1325547227.1416893406358-- 
```

 Response messages's content type is given using Accept header. Supported values are ```text/xml``` and ```application/json```. However, if Provider Gateway returns the response as SOAP attachment, the value of HTTP Accept header is ignored and the response is returned in its original format.

Consumer Gateway receives HTTP GET, POST, PUT and DELETE request from information system, converts the request to SOAP message, sends the SOAP message to the Security Server, receives the SOAP response from the security server, converts the response according to the value of the request's Accept header and returns the response to the information system. User id and message id that are required by X-Road can passed via HTTP headers (X-XRd-UserId, X-XRd-MessageId) or URL parameters. If X-XRd-UserId header is missing from the request, "anonymous" is set as user id. If X-XRd-MessageId is missing from the request, unique id is generated automatically. Both headers are included in the response message.

Browser-based access makes it possible to access services that are not configured in Consumer Gateway. The identifier of the service to be called is given as resource path. X-Road message headers and other request parameters are defined as URL parameters. It's enough to know that a service with the given identifier exists in X-Road and the X-Road client identifier that's used for making the service call is authorized to call the service. However, reformatting of resource links doesn't work when using browser-based access. For example:

```
http://localhost:8080/rest-gateway-0.0.4/Consumer/FI_PILOT.GOV.1019125-0.Demo2Service.getOrganizationList.v1/49?X-XRd-UserId=test&X-XRd-MessageId=1&X-XRd-NamespaceSerialize=http://vrk-test.x-road.fi/producer&X-XRd-NamespacePrefixSerialize=ks&Accept=application/json
```

#### Provider Gateway

Provider Gateway accepts SOAP messages following the X-Road v6 SOAP profile and translates them to REST service requests. XML request parameters are translated to REST service request URI. For example:

```
<request>
    <resourceId>id</resourceId>
    <param1>value1</param1>
    <param2>value2</param2>
</request>
```
```
[GET] http://www.restservice.com/id?param1=value1&param2=value2
```

If HTTP request body is required in the request that is sent to the service provider, the request body must be passed as a SOAP attachment. In this case SOAP request must contain an empty ```RESTGatewayRequestBody``` element, based on which Provider Gateway reads the first attachment and uses it as a HTTP request body when sending a request to a service provider. The response received from the service provider is wrapped in SOAP body or SOAP attachment according to Provider Gateway's configuration. For example:

```
.
.
.
<request>
    <resourceId>id</resourceId>
    <param1>value1</param1>
    <param2>value2</param2>
    <RESTGatewayRequestBody href="RESTGatewayRequestBody" />
</request>
.
.
.
------=_Part_1_1325547227.1416893406358
Content-Type: application/json
Content-ID: RESTGatewayRequestBody

{"id":1,"name":"Test name"}
------=_Part_1_1325547227.1416893406358-- 
```

```
Content-Type: application/json
[POST] http://www.restservice.com/id?param1=value1&param2=value2
Request body: {"id":1,"name":"Test name"}
```

Provider Gateway receives SOAP request from Security Server, translates the request to REST service request, sends the request to a REST service, converts the response to XML (if needed), wraps the response in SOAP message and returns the SOAP response to the security server. X-Road SOAP-headers are passed via HTTP headers (X-XRd-Client, X-XRd-Service, X-XRd-UserId, X-XRd-MessageId).

### Software Requirements

* Java 6 or later
* Tomcat 6 or later

### Installation

* Download the ```rest-gateway-0.0.4.war``` file.
* Copy the file ```tomcat.home/webapps``` folder.
* Start/restart Tomcat. The application is now accessible at:

#### Consumer Gateway

```
http://localhost:8080/rest-gateway-0.0.4/Consumer
```

#### Provider Gateway

```
http://localhost:8080/rest-gateway-0.0.4/Provider
```

The WSDL description is accessible at:

```
http://localhost:8080/rest-gateway-0.0.4/Provider?wsdl
```
### Consumer Gateway Configuration

General settings are configured through ```WEB-INF/classes/consumer-gateway.properties``` configuration file. All the general properties are mandatory.

<table>
          <tbody>
            <tr>
              <th>Property</th>
              <th>Required</th>
              <th>Default</th>
              <th>Description</th>
            </tr>
            <tr>
              <td>id.client</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Identifier of the X-Road client that initiates the service call: instance.memberClass.memberId.subsystem <br />Can be overridden for each service.</td>
            </tr>
            <tr>
              <td>ss.url</td>
              <td>&#42;</td>
              <td>-</td>
              <td>URL or IP address of the security server. The URL of Provider Gateway can be used for testing purposes.</td>
            </tr>
			<tr>
              <td>namespace.deserialize</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Namespace that's used for serializing incoming SOAP responses. Can be overridden for each service.</td>
            </tr>			
            <tr>
              <td>namespace.serialize</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Namespace that's used for serializing outgoing SOAP requests. Can be overridden for each service.</td>
            </tr>
            <tr>
              <td>namespace.prefix.serialize</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Namespace prefix that's used for serializing outgoing SOAP requests. Can be overridden for each service.</td>
            </tr>
</tbody>
</table>

Individual services are configured through ```WEB-INF/classes/consumers.properties``` configuration file. Each service has 8 properties of which 3 are mandatory. Each property must be prefixed with the number of the service, e.g. ```0.id```, ```0.path```, ```0.verb```. The numbering starts from zero.

<table>
          <tbody>
            <tr>
              <th>Property</th>
              <th>Required</th>
              <th>Default</th>
              <th>Description</th>
            </tr>
            <tr>
              <td>id</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Identifier of the X-Road service that's called : instance.memberClass.memberId.subsystem.service.version</td>
            </tr>
            <tr>
              <td>path</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Path is the part after Consumer Gateway base URL that identifies a single service. Path may contain a resource id's that's marked with {resourceId}. If the service response contains links to other resources it's recommended that the path matches with the real URL of the service.</td>
            </tr>
            <tr>
              <td>verb</td>
              <td>&#42;</td>
              <td>-</td>
              <td>HTTP verb that is supported by the service. The service identifier is a combination of path and verb properties, so the same path can be used multiple times with different verbs.</td>
            </tr>
            <tr>
              <td>id.client</td>
              <td></td>
              <td>-</td>
              <td>Identifier of the X-Road client that initiates the service call: instance.memberClass.memberId.subsystem <br />If not defined, default value from consumser-gateway.propertiesis used.</td>
            </tr>
            <tr>
              <td>response.modurl</td>
              <td></td>
              <td>false</td>
              <td>Boolean value that indicates if Consumer Gateway should try to reformat links to other resources found in the response message. It's recommended to keep this property as false, if the response message doesn't contain links to other resources.</td>
            </tr>
            <tr>
              <td>namespace.deserialize</td>
              <td></td>
              <td>-</td>
              <td>Namespace that's used for deserializing incoming SOAP responses. If not defined, default value from consumser-gateway.properties is used.</td>
            </tr>
            <tr>
              <td>namespace.serialize</td>
              <td></td>
              <td>-</td>
              <td>Namespace that's used for serializing outgoing SOAP requests. If not defined, default value from consumser-gateway.properties is used.</td>
            </tr>
            <tr>
              <td>namespace.prefix.serialize</td>
              <td></td>
              <td>-</td>
              <td>Namespace prefix that's used for serializing outgoing SOAP requests. If not defined, default value from consumser-gateway.properties is used.</td>
            </tr>
</tbody>
</table>

If the response message contains links to other resources, the links are reformatted to point the consumer gateway. However, only links which beginning matches with the resource path used in Consumer Gateway are reformatted. For example:

Resource path:
```
/avoindata.prh.fi/opendata/bis/v1/
```

Full URL of the resource path on Consumer Gateway:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/
```

Link to another resource:
```
http://avoindata.prh.fi/opendata/bis/v1/2659636-7
```

Reformatted link:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2659636-7
```

This link wouldn't be reformatted as its beginning doesn't match with the resource path:

```
http://anotherapi.prh.fi/opendata/bis/v1/2659636-7
```

### Provider Gateway Configuration

General settings are configured through ```WEB-INF/classes/provider-gateway.properties``` configuration file. All the general properties are mandatory.

<table>
          <tbody>
            <tr>
              <th>Property</th>
              <th>Required</th>
              <th>Default</th>
              <th>Description</th>
            </tr>
            <tr>
              <td>wsdl.path</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Path or filename of the WSDL file. If only filename is given, the file must be located in WEB-INF/classes folder.</td>
            </tr>
            <tr>
              <td>namespace.deserialize</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Namespace that's used for deserializing incoming SOAP requests. Can be overridden for each service.</td>
            </tr>
            <tr>
              <td>namespace.serialize</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Namespace that's used for serializing outgoing SOAP responses. Can be overridden for each service.</td>
            </tr>
            <tr>
              <td>namespace.prefix.serialize</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Namespace prefix that's used for serializing outgoing SOAP responses. Can be overridden for each service.</td>
            </tr>
</tbody>
</table>


REST services are configured through ```WEB-INF/classes/providers.properties``` configuration file. Each service has 12 properties of which 2 are mandatory. Each property must be prefixed with the number of the service, e.g. ```0.id```, ```0.url```. The numbering starts from zero.

<table>
          <tbody>
            <tr>
              <th>Property</th>
              <th>Required</th>
              <th>Default</th>
              <th>Description</th>
            </tr>
            <tr>
              <td>id</td>
              <td>&#42;</td>
              <td>-</td>
              <td>Identifier of the X-Road service : instance.memberClass.memberId.subsystem.service.version</td>
            </tr>
            <tr>
              <td>url</td>
              <td>*</td>
              <td>-</td>
              <td>URL of the REST service.</td>
            </tr>
            <tr>
              <td>verb</td>
              <td></td>
              <td>GET</td>
              <td>HTTP verb that's used in the service call. Supported values: GET, POST, PUT and DELETE.</td>
            </tr>
            <tr>
              <td>contenttype</td>
              <td></td>
              <td>-</td>
              <td>Content-type HTTP header that's used in the service call. Required if HTTP verb is POST, PUT or DELETE and HTTP request contains a request body.</td>
            </tr>
            <tr>
              <td>accept</td>
              <td></td>
              <td>-</td>
              <td>Accept HTTP header that's used in the service call.</td>
            </tr>
            <tr>
              <td>objecttag</td>
              <td></td>
              <td>result</td>
              <td>Wrapped element for a single JSON object.</td>
            </tr>
            <tr>
              <td>arraytag</td>
              <td></td>
              <td>results</td>
              <td>Wrapped element for a JSON array.</td>
            </tr>
            <tr>
              <td>response.attachment</td>
              <td></td>
              <td>false</td>
              <td>Return REST response as SOAP attachment. If response is returned as SOAP attachment, Consumer Gateway ignores the value of HTTP Accept header and returns the response in the original format.</td>
            </tr>
            <tr>
              <td>request.xrdheaders</td>
              <td></td>
              <td>true</td>
              <td>Pass X-Road SOAP headers to REST service via HTTP headers.</td>
            </tr>
            <tr>
              <td>namespace.deserialize</td>
              <td></td>
              <td>-</td>
              <td>Namespace that's used for deserializing incoming SOAP requests. If not defined, default value from provider-gateway.properties is used.</td>
            </tr>
            <tr>
              <td>namespace.serialize</td>
              <td></td>
              <td>-</td>
              <td>Namespace that's used for serializing outgoing SOAP responses. If not defined, default value from provider-gateway.properties is used.</td>
            </tr>
            <tr>
              <td>namespace.prefix.serialize</td>
              <td></td>
              <td>-</td>
              <td>Namespace prefix that's used for serializing outgoing SOAP responses. If not defined, default value from provider-gateway.properties is used.</td>
            </tr>
</tbody>
</table>

### Usage

REST Gateway 0.0.4 is shipped with configuration that includes 6 ready-to-use REST services. By default Consumer Gateway is configured to call Provider Gateway directly, and both Gateways have the same services configured ([diagram](https://raw.githubusercontent.com/educloudalliance/xroad-rest-gateway/master/images/default_configuration-rest-gateway-0.0.4.png)). In this way it's possible to test both Gateways without access to X-Road. In addition, it's possible to call Provider Gateway directly using SOAP without Consumer Gateway in the middle ([diagram](https://raw.githubusercontent.com/educloudalliance/xroad-rest-gateway/master/images/provider_call-rest-gateway-0.0.4.png)).

The preconfigured Consumer Gateway services must be called using HTTP GET and Accept header can be set to ```text/xml``` or ```application/json```. Provider Gateway services must be called using HTTP POST and Content-Type must be set to ```text/xml```.[REST Client plugin](https://addons.mozilla.org/fi/firefox/addon/restclient/) for Firefox can be used for testing purposes.

#### REST API for City of Helsinki Service Map - List of Organizations

API documentation: http://www.hel.fi/palvelukarttaws/rest/ver2_en.html

#### Consumer Gateway

Configuration:
```
0.id=FI_PILOT.GOV.1019125-0.Demo2Service.getOrganizationList.v1
0.path=/www.hel.fi/palvelukarttaws/rest/v2/organization/
0.verb=get
```

Service request:

* Method: ```GET```
* Accept: ```text/xml``` or ```application/json```
* X-XRd-MessageId: ```1```
* X-XRd-UserId: ```test```

URL:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/www.hel.fi/palvelukarttaws/rest/v2/organization/
```

Browser-based access:

```
http://localhost:8080/rest-gateway-0.0.4/Consumer/FI_PILOT.GOV.1019125-0.Demo2Service.getOrganizationList.v1/?X-XRd-UserId=test&X-XRd-MessageId=1&X-XRd-NamespaceSerialize=http://vrk-test.x-road.fi/producer&X-XRd-NamespacePrefixSerialize=ks&Accept=application/json
```

Consumer Gateway response:

```
{
    "results": {
        "result": [
            {
                "id": 49,
                "name_en": "City of Espoo",
                "name_sv": "Esbo stad",
                "data_source_url": "www.espoo.fi",
                "name_fi": "Espoon kaupunki"
            },
            {
                "id": 91,
                "name_en": "City of Helsinki",
                "name_sv": "Helsingfors stad",
                "data_source_url": "www.hel.fi",
                "name_fi": "Helsingin kaupunki"
            },
            {
                "id": 92,
                "name_en": "City of Vantaa",
                "name_sv": "Vanda stad",
                "data_source_url": "www.vantaa.fi",
                "name_fi": "Vantaan kaupunki"
            },
            {
                "id": 235,
                "name_en": "City of Kauniainen",
                "name_sv": "Grankulla stad",
                "data_source_url": "www.kauniainen.fi",
                "name_fi": "Kauniaisten kaupunki"
            },
            {
                "id": 1000,
                "name_en": "State IT Service Centre, Suomi.fi editorial team",
                "name_sv": "Statens IT-servicecentral, Suomi.fi-redaktionen",
                "data_source_url": "www.suomi.fi",
                "name_fi": "Valtion IT-palvelukeskus, Suomi.fi-toimitus"
            },
            {
                "id": 1001,
                "name_en": "HUS Hospital District",
                "name_sv": "Samkommunen HNS",
                "data_source_url": "www.hus.fi",
                "name_fi": "HUS-kuntayhtymä"
            },
            {
                "id": 1002,
                "name_en": "Helsinki Marketing Ltd",
                "name_sv": "Helsinki Marketing Ltd",
                "data_source_url": "www.visithelsinki.fi",
                "name_fi": "Helsingin markkinointi Oy"
            },
            {
                "id": 1003,
                "name_en": "Ministry of Justice, Election Unit",
                "name_sv": "Justitieministeriet, den högsta valmyndigheten",
                "data_source_url": "www.vaalit.fi",
                "name_fi": "Oikeusministeriö, ylin vaaliviranomainen"
            },
            {
                "id": 1004,
                "name_en": "Helsinki Region Environmental Services Authority HSY",
                "name_sv": "Helsingforsregionens miljötjänster HRM",
                "data_source_url": "www.hsy.fi",
                "name_fi": "Helsingin seudun ympäristöpalvelut HSY"
            },
            {
                "id": 1005,
                "name_en": "Service Map editorial team",
                "name_sv": "Servicekartans redaktion",
                "data_source_url": "www.hel.fi/palvelukartta",
                "name_fi": "Palvelukartan toimitus"
            },
            {
                "id": 1007,
                "name_en": "JLY - Finnish Solid Waste Association",
                "name_sv": "JLY - Avfallsverksföreningen rf",
                "data_source_url": "www.jly.fi",
                "name_fi": "JLY Jätelaitosyhdistys ry"
            },
            {
                "id": 1008,
                "name_en": "The Norwegian Electric Vehicle Association",
                "name_sv": "Norsk Elbilforening",
                "data_source_url": "NOBIL.no",
                "name_fi": "Norsk Elbilforening, sähköautojen latauspisteet"
            },
            {
                "id": 1009,
                "name_en": "External service point register user society",
                "name_sv": "Gemenskapen bakom externa serviceregister",
                "data_source_url": "asiointi.hel.fi/tprulkoinen",
                "name_fi": "Ulkoisen toimipisterekisterin käyttäjäyhteisö"
            },
            {
                "id": 1010,
                "name_en": "University of Jyväskylä, LIPAS Liikuntapaikat.fi",
                "name_sv": "Jyväskylä universitet, LIPAS Liikuntapaikat.fi",
                "data_source_url": "www.liikuntapaikat.fi",
                "name_fi": "Jyväskylän yliopisto, LIPAS Liikuntapaikat.fi"
            }
        ]
    }
}
```

#### Provider Gateway
Configuration:
```
0.id=FI_PILOT.GOV.1019125-0.Demo2Service.getOrganizationList.v1
0.url=http://www.hel.fi/palvelukarttaws/rest/v2/organization/
```
Service request:

* Method: ```POST```
* Content-Type: ```text/xml```

```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>getOrganizationList</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <test:getOrganizationList xmlns:test="http://vrk-test.x-road.fi/producer">
            <test:request/>
        </test:getOrganizationList>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```
API response URL: http://www.hel.fi/palvelukarttaws/rest/v2/organization/

REST Gateway response:
```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>getOrganizationList</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <ts1:getOrganizationListResponse xmlns:ts1="http://vrk-test.x-road.fi/producer">
            <ts1:request/>
            <ts1:response>
                <ts1:results>
                    <ts1:result>
                        <ts1:id>49</ts1:id>
                        <ts1:name_en>City of Espoo</ts1:name_en>
                        <ts1:name_sv>Esbo stad</ts1:name_sv>
                        <ts1:data_source_url>www.espoo.fi</ts1:data_source_url>
                        <ts1:name_fi>Espoon kaupunki</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>91</ts1:id>
                        <ts1:name_en>City of Helsinki</ts1:name_en>
                        <ts1:name_sv>Helsingfors stad</ts1:name_sv>
                        <ts1:data_source_url>www.hel.fi</ts1:data_source_url>
                        <ts1:name_fi>Helsingin kaupunki</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>92</ts1:id>
                        <ts1:name_en>City of Vantaa</ts1:name_en>
                        <ts1:name_sv>Vanda stad</ts1:name_sv>
                        <ts1:data_source_url>www.vantaa.fi</ts1:data_source_url>
                        <ts1:name_fi>Vantaan kaupunki</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>235</ts1:id>
                        <ts1:name_en>City of Kauniainen</ts1:name_en>
                        <ts1:name_sv>Grankulla stad</ts1:name_sv>
                        <ts1:data_source_url>www.kauniainen.fi</ts1:data_source_url>
                        <ts1:name_fi>Kauniaisten kaupunki</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1000</ts1:id>
                        <ts1:name_en>State IT Service Centre, Suomi.fi editorial team</ts1:name_en>
                        <ts1:name_sv>Statens IT-servicecentral, Suomi.fi-redaktionen</ts1:name_sv>
                        <ts1:data_source_url>www.suomi.fi</ts1:data_source_url>
                        <ts1:name_fi>Valtion IT-palvelukeskus, Suomi.fi-toimitus</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1001</ts1:id>
                        <ts1:name_en>HUS Hospital District</ts1:name_en>
                        <ts1:name_sv>Samkommunen HNS</ts1:name_sv>
                        <ts1:data_source_url>www.hus.fi</ts1:data_source_url>
                        <ts1:name_fi>HUS-kuntayhtymä</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1002</ts1:id>
                        <ts1:name_en>Helsinki Marketing Ltd</ts1:name_en>
                        <ts1:name_sv>Helsinki Marketing Ltd</ts1:name_sv>
                        <ts1:data_source_url>www.visithelsinki.fi</ts1:data_source_url>
                        <ts1:name_fi>Helsingin markkinointi Oy</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1004</ts1:id>
                        <ts1:name_en>Helsinki Region Environmental Services Authority HSY</ts1:name_en>
                        <ts1:name_sv>Helsingforsregionens miljötjänster HRM</ts1:name_sv>
                        <ts1:data_source_url>www.hsy.fi</ts1:data_source_url>
                        <ts1:name_fi>Helsingin seudun ympäristöpalvelut HSY</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1005</ts1:id>
                        <ts1:name_en>Service Map editorial team</ts1:name_en>
                        <ts1:name_sv>Servicekartans redaktion</ts1:name_sv>
                        <ts1:data_source_url>www.hel.fi/palvelukartta</ts1:data_source_url>
                        <ts1:name_fi>Palvelukartan toimitus</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1007</ts1:id>
                        <ts1:name_en>JLY - Finnish Solid Waste Association</ts1:name_en>
                        <ts1:name_sv>JLY - Avfallsverksföreningen rf</ts1:name_sv>
                        <ts1:data_source_url>www.jly.fi</ts1:data_source_url>
                        <ts1:name_fi>JLY Jätelaitosyhdistys ry</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1008</ts1:id>
                        <ts1:name_en>The Norwegian Electric Vehicle Association</ts1:name_en>
                        <ts1:name_sv>Norsk Elbilforening</ts1:name_sv>
                        <ts1:data_source_url>NOBIL.no</ts1:data_source_url>
                        <ts1:name_fi>Norsk Elbilforening, sähköautojen latauspisteet</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1009</ts1:id>
                        <ts1:name_en>External service point register user society</ts1:name_en>
                        <ts1:name_sv>Gemenskapen bakom externa serviceregister</ts1:name_sv>
                        <ts1:data_source_url>asiointi.hel.fi/tprulkoinen</ts1:data_source_url>
                        <ts1:name_fi>Ulkoisen toimipisterekisterin käyttäjäyhteisö</ts1:name_fi>
                    </ts1:result>
                    <ts1:result>
                        <ts1:id>1010</ts1:id>
                        <ts1:name_en>University of Jyväskylä, LIPAS Liikuntapaikat.fi</ts1:name_en>
                        <ts1:name_sv>Jyväskylä universitet, LIPAS Liikuntapaikat.fi</ts1:name_sv>
                        <ts1:data_source_url>www.liikuntapaikat.fi</ts1:data_source_url>
                        <ts1:name_fi>Jyväskylän yliopisto, LIPAS Liikuntapaikat.fi</ts1:name_fi>
                    </ts1:result>
                </ts1:results>
            </ts1:response>
        </ts1:getOrganizationListResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

#### REST API for City of Helsinki Service Map - Single Organization

API documentation: http://www.hel.fi/palvelukarttaws/rest/ver2_en.html

#### Consumer Gateway

Configuration:
```
1.id=FI_PILOT.GOV.1019125-0.Demo2Service.getOrganization.v1
1.path=/www.hel.fi/palvelukarttaws/rest/v2/organization/{resourceId}
1.verb=get
```

Service request:

* Method: ```GET```
* Accept: ```text/xml``` or ```application/json```
* X-XRd-MessageId: ```1```
* X-XRd-UserId: ```test```

URL:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/www.hel.fi/palvelukarttaws/rest/v2/organization/49
```

Browser-based access:

```
http://localhost:8080/rest-gateway-0.0.4/Consumer/FI_PILOT.GOV.1019125-0.Demo2Service.getOrganizationList.v1/49?X-XRd-UserId=test&X-XRd-MessageId=1&X-XRd-NamespaceSerialize=http://vrk-test.x-road.fi/producer&X-XRd-NamespacePrefixSerialize=ks&Accept=application/json
```

Consumer Gateway response:

```
{
    "result": {
        "id": 49,
        "name_en": "City of Espoo",
        "name_sv": "Esbo stad",
        "data_source_url": "www.espoo.fi",
        "name_fi": "Espoon kaupunki"
    }
}
```

#### Provider Gateway
Configuration:
```
1.id=FI_PILOT.GOV.1019125-0.Demo2Service.getOrganization.v1
1.url=http://www.hel.fi/palvelukarttaws/rest/v2/organization/
```
Service request:

* Method: ```POST```
* Content-Type: ```text/xml```

```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>getOrganization</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <test:getOrganization xmlns:test="http://vrk-test.x-road.fi/producer">
            <test:request>
                <resourceId>49</resourceId>
            </test:request>
        </test:getOrganization>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

API response URL: http://www.hel.fi/palvelukarttaws/rest/v2/organization/49

REST Gateway response:
```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>getOrganization</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <ts1:getOrganizationResponse xmlns:ts1="http://vrk-test.x-road.fi/producer">
            <ts1:request>
                <ts1:resourceId>49</ts1:resourceId>
            </ts1:request>
            <ts1:response>
                <ts1:result>
                    <ts1:id>49</ts1:id>
                    <ts1:name_en>City of Espoo</ts1:name_en>
                    <ts1:name_sv>Esbo stad</ts1:name_sv>
                    <ts1:data_source_url>www.espoo.fi</ts1:data_source_url>
                    <ts1:name_fi>Espoon kaupunki</ts1:name_fi>
                </ts1:result>
            </ts1:response>
        </ts1:getOrganizationResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

#### Open Weather Map

API documentation: http://openweathermap.org/current

#### Consumer Gateway

Configuration:
```
2.id=FI_PILOT.GOV.1019125-0.Demo2Service.getWeather.v1
2.path=/api.openweathermap.org/data/2.5/weather/
2.verb=get
```

Service request:

* Method: ```GET```
* Accept: ```text/xml``` or ```application/json```
* X-XRd-MessageId: ```1```
* X-XRd-UserId: ```test```

URL:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/api.openweathermap.org/data/2.5/weather/?q=Helsinki
```

Browser-based access:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/FI_PILOT.GOV.1019125-0.Demo2Service.getWeather.v1/?q=Helsinki&X-XRd-UserId=test&X-XRd-MessageId=1&X-XRd-NamespaceSerialize=http://vrk-test.x-road.fi/producer&X-XRd-NamespacePrefixSerialize=ks&Accept=application/json
```

Consumer Gateway response:

```
{
    "current": {
        "clouds": {
            "name": "overcast clouds",
            "value": 92
        },
        "lastupdate": {
            "value": "2015-02-22T06:34:36"
        },
        "wind": {
            "speed": {
                "name": "Light breeze",
                "value": 3.1
            },
            "direction": {
                "name": "South-southwest",
                "value": 202,
                "code": "SSW"
            }
        },
        "humidity": {
            "unit": "%",
            "value": 93
        },
        "pressure": {
            "unit": "hPa",
            "value": 988.543
        },
        "visibility": "",
        "precipitation": {
            "unit": "3h",
            "value": 0.6,
            "mode": "rain"
        },
        "weather": {
            "icon": "10d",
            "value": "light rain",
            "number": 500
        },
        "temperature": {
            "min": 274.85,
            "unit": "kelvin",
            "max": 274.85,
            "value": 274.85
        },
        "city": {
            "coord": {
                "lon": 24.94,
                "lat": 60.17
            },
            "id": 658225,
            "name": "Helsinki",
            "sun": {
                "set": "2015-02-22T15:28:58",
                "rise": "2015-02-22T05:38:25"
            },
            "country": "FI"
        }
    }
}
```

#### Provider Gateway

Configuration:
```
2.id=FI_PILOT.GOV.1019125-0.Demo2Service.getWeather.v1
2.url=http://api.openweathermap.org/data/2.5/weather?mode=xml
2.response.convert=false
```
Service request:

Method: ```POST```
Content-Type: ```text/xml```

```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>getWeather</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <test:getWeather xmlns:test="http://vrk-test.x-road.fi/producer">
            <test:request>
                <test:q>Helsinki</test:q>
            </test:request>
        </test:getWeather>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

API response URL: http://api.openweathermap.org/data/2.5/weather?mode=xml&q=Helsinki

REST Gateway response:
```

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>getWeather</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <ts1:getWeatherResponse xmlns:ts1="http://vrk-test.x-road.fi/producer">
            <ts1:request>
                <ts1:q>Helsinki</ts1:q>
            </ts1:request>
            <ts1:response>
                <ts1:current> 
                    <ts1:city id="658225" name="Helsinki"> 
                        <ts1:coord lat="60.17" lon="24.94"/> 
                        <ts1:country>FI</ts1:country> 
                        <ts1:sun rise="2015-01-10T07:15:39" set="2015-01-10T13:40:00"/> 
                    </ts1:city> 
                    <ts1:temperature max="270.55" min="270.55" unit="kelvin" value="270.55"/> 
                    <ts1:humidity unit="%" value="94"/> 
                    <ts1:pressure unit="hPa" value="988.069"/> 
                    <ts1:wind> 
                        <ts1:speed name="Moderate breeze" value="6.71"/> 
                        <ts1:direction code="NW" name="Northwest" value="304"/> 
                    </ts1:wind> 
                    <ts1:clouds name="scattered clouds" value="32"/> 
                    <ts1:visibility/> 
                    <ts1:precipitation mode="no"/> 
                    <ts1:weather icon="03d" number="802" value="scattered clouds"/> 
                    <ts1:lastupdate value="2015-01-10T08:05:03"/>
                </ts1:current>
            </ts1:response>
        </ts1:getWeatherResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```
#### Finnish Patent and Registration Office - Business Information Search

API documentation: http://avoindata.prh.fi/ytj_en.html

#### Consumer Gateway

Configuration:
```
3.id=FI_PILOT.GOV.1019125-0.Demo2Service.searchCompany.v1
3.path=/avoindata.prh.fi/opendata/bis/v1/
3.verb=get
3.response.modurl=true
```

Service request:

* Method: ```GET```
* Accept: ```text/xml``` or ```application/json```
* X-XRd-MessageId: ```1```
* X-XRd-UserId: ```test```

URL:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/?totalResults=false&resultsFrom=0&name=asunto&companyRegistrationFrom=2014-02-28
```

Browser-based access:

```
http://localhost:8080/rest-gateway-0.0.4/Consumer/FI_PILOT.GOV.1019125-0.Demo2Service.searchCompany.v1/?totalResults=false&resultsFrom=0&name=asunto&companyRegistrationFrom=2014-02-28&X-XRd-UserId=test&X-XRd-MessageId=1&X-XRd-NamespaceSerialize=http://vrk-test.x-road.fi/producer&X-XRd-NamespacePrefixSerialize=ks&Accept=application/json
```

Consumer Gateway response:

```
{
    "responseData": {
        "exceptionNoticeUri": null,
        "nextResultsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1?companyRegistrationFrom=2014-02-28&resultsFrom=10&totalResults=false&name=asunto",
        "resultsFrom": 0,
        "results": [
            {
                "registrationDate": "2014-03-03",
                "name": "Asunto Oy Akaan Nelikko",
                "businessId": "2608731-9",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2608731-9",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-03",
                "name": "Asunto Oy Lempäälän Onni",
                "businessId": "2608732-7",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2608732-7",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-03",
                "name": "Asunto-osakeyhtiö Jokioisten Kartano",
                "businessId": "2608782-9",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2608782-9",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-03",
                "name": "Asunto Oy Janakkalan Mäntyniemenherra",
                "businessId": "2608809-8",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2608809-8",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-03",
                "name": "Asunto Oy Kokkolan Lepolanpelto",
                "businessId": "2608873-3",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2608873-3",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-04",
                "name": "Asunto Oy Kokkolan Aurinkopiha",
                "businessId": "2608876-8",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2608876-8",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-05",
                "name": "Asunto Oy Kuusamon Samostenpuisto",
                "businessId": "2609161-7",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2609161-7",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-05",
                "name": "Asunto Oy Vaasan Klemettilän puisto III",
                "businessId": "2609164-1",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2609164-1",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-05",
                "name": "Asunto Oy Vaasan Klemettilän puisto I",
                "businessId": "2609171-3",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2609171-3",
                "companyForm": "AOY"
            },
            {
                "registrationDate": "2014-03-05",
                "name": "Asunto Oy Vaasan Klemettilän puisto II",
                "businessId": "2609177-2",
                "detailsUri": "http://localhost:80/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2609177-2",
                "companyForm": "AOY"
            }
        ],
        "totalResults": -1,
        "previousResultsUri": null,
        "type": "fi.prh.opendata.bis",
        "version": 1
    }
}
```

#### Provider Gateway

Configuration:
```
3.id=FI_PILOT.GOV.1019125-0.Demo2Service.searchCompany.v1
3.url=http://avoindata.prh.fi/bis/v1
3.objecttag=responseData
```
Service request:

* Method: ```POST```
* Content-Type: ```text/xml```

```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>searchCompany</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <test:searchCompany xmlns:test="http://vrk-test.x-road.fi/producer">
            <test:request>
                <totalResults>false</totalResults>
                <maxResults>10</maxResults>
                <resultsFrom>0</resultsFrom>
                <name>asunto</name>
                <companyRegistrationFrom>2014-02-28</companyRegistrationFrom>
            </test:request>
        </test:searchCompany>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

API results URL: http://avoindata.prh.fi/bis/v1?totalResults=false&maxResults=10&resultsFrom=0&name=asunto&companyRegistrationFrom=2014-02-28

REST Gateway response:
```

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>searchCompany</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <ts1:searchCompanyResponse xmlns:ts1="http://vrk-test.x-road.fi/producer">
            <ts1:request>
                <ts1:totalResults>false</ts1:totalResults>
                <ts1:maxResults>10</ts1:maxResults>
                <ts1:resultsFrom>0</ts1:resultsFrom>
                <ts1:name>asunto</ts1:name>
                <ts1:companyRegistrationFrom>2014-02-28</ts1:companyRegistrationFrom>
            </ts1:request>
            <ts1:response>
                <ts1:responseData>
                    <ts1:exceptionNoticeUri>http://avoindata.prh.fi/bis-exception.txt</ts1:exceptionNoticeUri>
                    <ts1:nextResultsUri>http://avoindata.prh.fi/opendata/bis/v1?companyRegistrationFrom=2014-02-28&amp;resultsFrom=10&amp;totalResults=false&amp;name=asunto&amp;maxResults=10</ts1:nextResultsUri>
                    <ts1:results>
                        <ts1:registrationDate>2015-01-08</ts1:registrationDate>
                        <ts1:name>Asunto Oy Kangasalan Kauppakulma</ts1:name>
                        <ts1:businessId>2664443-7</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2664443-7</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2015-01-05</ts1:registrationDate>
                        <ts1:name>Asunto Oy Mikkelin Sinfonia</ts1:name>
                        <ts1:businessId>2664370-9</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2664370-9</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2015-01-02</ts1:registrationDate>
                        <ts1:name>Asunto Oy Tampereen Ryydynkatu 64</ts1:name>
                        <ts1:businessId>2663825-6</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2663825-6</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                        <ts1:name>Asunto Osakeyhtiö Teijon Metsänvartija</ts1:name>
                        <ts1:businessId>2663307-6</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2663307-6</ts1:detailsUri>
                        <ts1:companyForm>OY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2014-12-29</ts1:registrationDate>
                        <ts1:name>Asunto Oy Hämeenlinnan Hongiston Kartano II</ts1:name>
                        <ts1:businessId>2662966-7</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2662966-7</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2014-12-29</ts1:registrationDate>
                        <ts1:name>Asunto Oy Hämeenlinnan Hongiston Pehtoori</ts1:name>
                        <ts1:businessId>2662967-5</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2662967-5</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2014-12-29</ts1:registrationDate>
                        <ts1:name>Asunto Oy Hämeenlinnan Hongiston Kartano III</ts1:name>
                        <ts1:businessId>2662965-9</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2662965-9</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2014-12-23</ts1:registrationDate>
                        <ts1:name>Asunto Oy Huittisten Satulatie 5</ts1:name>
                        <ts1:businessId>2662624-7</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2662624-7</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2014-12-23</ts1:registrationDate>
                        <ts1:name>Asunto Oy Lapuan Eeronpuisto</ts1:name>
                        <ts1:businessId>2663340-4</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2663340-4</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:results>
                        <ts1:registrationDate>2014-12-23</ts1:registrationDate>
                        <ts1:name>Asunto Oy Lapuan Mikonpuisto</ts1:name>
                        <ts1:businessId>2663388-5</ts1:businessId>
                        <ts1:detailsUri>http://avoindata.prh.fi/opendata/bis/v1/2663388-5</ts1:detailsUri>
                        <ts1:companyForm>AOY</ts1:companyForm>
                    </ts1:results>
                    <ts1:resultsFrom>0</ts1:resultsFrom>
                    <ts1:totalResults>-1</ts1:totalResults>
                    <ts1:previousResultsUri>null</ts1:previousResultsUri>
                    <ts1:type>fi.prh.opendata.bis</ts1:type>
                    <ts1:version>1</ts1:version>
                </ts1:responseData>
            </ts1:response>
        </ts1:searchCompanyResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

#### Finnish Patent and Registration Office - Get Company

API documentation: http://avoindata.prh.fi/ytj_en.html

#### Consumer Gateway

Configuration:
```
4.id=FI_PILOT.GOV.1019125-0.Demo2Service.getCompany.v1
4.path=/avoindata.prh.fi/opendata/bis/v1/{resourceId}
4.verb=get
4.response.modurl=true
```

Service request:

* Method: ```GET```
* Accept: ```text/xml``` or ```application/json```
* X-XRd-MessageId: ```1```
* X-XRd-UserId: ```test```

URL:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/avoindata.prh.fi/opendata/bis/v1/2663307-6
```

Browser-based access:

```
http://localhost:8080/rest-gateway-0.0.4/Consumer/FI_PILOT.GOV.1019125-0.Demo2Service.getCompany.v1/2663307-6/?X-XRd-UserId=test&X-XRd-MessageId=1&X-XRd-NamespaceSerialize=http://vrk-test.x-road.fi/producer&X-XRd-NamespacePrefixSerialize=ks&Accept=application/json
```

Consumer Gateway response:

```
{
    "responseData": {
        "exceptionNoticeUri": null,
        "nextResultsUri": null,
        "resultsFrom": 0,
        "results": {
            "registrationDate": "2014-12-30",
            "names": {
                "registrationDate": "2014-12-30",
                "order": 0,
                "source": 3,
                "name": "Asunto Osakeyhtiö Teijon Metsänvartija",
                "endDate": null,
                "version": 1
            },
            "name": "Asunto Osakeyhtiö Teijon Metsänvartija",
            "liquidations": {
                "registrationDate": "2014-12-30",
                "source": 3,
                "description": null,
                "endDate": null,
                "language": null,
                "type": "TP",
                "version": 1
            },
            "registedOffices": [
                {
                    "registrationDate": "2014-12-30",
                    "order": 0,
                    "source": 0,
                    "name": "SALO",
                    "endDate": null,
                    "language": "SE",
                    "version": 1
                },
                {
                    "registrationDate": "2014-12-30",
                    "order": 0,
                    "source": 0,
                    "name": "SALO",
                    "endDate": null,
                    "language": "EN",
                    "version": 1
                },
                {
                    "registrationDate": "2014-12-30",
                    "order": 0,
                    "source": 0,
                    "name": "SALO",
                    "endDate": null,
                    "language": "FI",
                    "version": 1
                }
            ],
            "registeredEntries": {
                "register": 1,
                "authority": 2,
                "registrationDate": "2014-12-30",
                "status": 2,
                "description": null,
                "endDate": null,
                "language": null,
                "statusDate": "2014-12-30"
            },
            "businessId": "2663307-6",
            "contactDetails": [
                {
                    "registrationDate": "2014-12-30",
                    "source": 0,
                    "value": "0400815565",
                    "endDate": null,
                    "language": "FI",
                    "type": "Matkapuhelin",
                    "version": 1
                },
                {
                    "registrationDate": "2014-12-30",
                    "source": 0,
                    "value": "0400815565",
                    "endDate": null,
                    "language": "SE",
                    "type": "Mobiltelefon",
                    "version": 1
                }
            ],
            "addresses": [
                {
                    "careOf": null,
                    "registrationDate": "2014-12-30",
                    "source": 0,
                    "street": "Teijontie 23",
                    "postCode": 25570,
                    "endDate": null,
                    "language": "FI",
                    "type": 2,
                    "version": 1,
                    "city": "TEIJO",
                    "country": null
                },
                {
                    "careOf": null,
                    "registrationDate": "2014-12-30",
                    "source": 0,
                    "street": "Teijontie 23",
                    "postCode": 25570,
                    "endDate": null,
                    "language": "FI",
                    "type": 1,
                    "version": 1,
                    "city": "TEIJO",
                    "country": null
                }
            ],
            "companyForms": [
                {
                    "registrationDate": "2014-12-30",
                    "source": 3,
                    "name": "Osakeyhtiö",
                    "endDate": null,
                    "language": "FI",
                    "type": "OY",
                    "version": 1
                },
                {
                    "registrationDate": "2014-12-30",
                    "source": 3,
                    "name": "Aktiebolag",
                    "endDate": null,
                    "language": "SE",
                    "type": "AB",
                    "version": 1
                },
                {
                    "registrationDate": "2014-12-30",
                    "source": 3,
                    "name": "Limited company",
                    "endDate": null,
                    "language": "EN",
                    "type": null,
                    "version": 1
                }
            ],
            "detailsUri": null,
            "companyForm": "OY"
        },
        "totalResults": -1,
        "previousResultsUri": null,
        "type": "fi.prh.opendata.bis",
        "version": 1
    }
}
```

#### Provider Gateway

Configuration:
```
4.id=FI_PILOT.GOV.1019125-0.Demo2Service.getCompany.v1
4.url=http://avoindata.prh.fi/bis/v1
4.objecttag=responseData
```
Service request:

* Method: ```POST```
* Content-Type: ```text/xml```

```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>getCompany</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <test:getCompany xmlns:test="http://vrk-test.x-road.fi/producer">
            <test:request>
                <resourceId>2663307-6</resourceId>
            </test:request>
        </test:getCompany>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

API response URL: http://avoindata.prh.fi/bis/v1/2663307-6

REST Gateway response:
```

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>getCompany</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <ts1:getCompanyResponse xmlns:ts1="http://vrk-test.x-road.fi/producer">
            <ts1:request>
                <ts1:resourceId>2663307-6</ts1:resourceId>
            </ts1:request>
            <ts1:response>
                <ts1:responseData>
                    <ts1:exceptionNoticeUri>http://avoindata.prh.fi/bis-exception.txt</ts1:exceptionNoticeUri>
                    <ts1:nextResultsUri>null</ts1:nextResultsUri>
                    <ts1:results>
                        <ts1:registedOffices>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>0</ts1:source>
                            <ts1:order>0</ts1:order>
                            <ts1:name>SALO</ts1:name>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>SE</ts1:language>
                            <ts1:version>1</ts1:version>
                        </ts1:registedOffices>
                        <ts1:registedOffices>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>0</ts1:source>
                            <ts1:order>0</ts1:order>
                            <ts1:name>SALO</ts1:name>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>EN</ts1:language>
                            <ts1:version>1</ts1:version>
                        </ts1:registedOffices>
                        <ts1:registedOffices>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>0</ts1:source>
                            <ts1:order>0</ts1:order>
                            <ts1:name>SALO</ts1:name>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>FI</ts1:language>
                            <ts1:version>1</ts1:version>
                        </ts1:registedOffices>
                        <ts1:companyForms>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>3</ts1:source>
                            <ts1:name>Osakeyhtiö</ts1:name>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>FI</ts1:language>
                            <ts1:type>OY</ts1:type>
                            <ts1:version>1</ts1:version>
                        </ts1:companyForms>
                        <ts1:companyForms>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>3</ts1:source>
                            <ts1:name>Aktiebolag</ts1:name>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>SE</ts1:language>
                            <ts1:type>AB</ts1:type>
                            <ts1:version>1</ts1:version>
                        </ts1:companyForms>
                        <ts1:companyForms>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>3</ts1:source>
                            <ts1:name>Limited company</ts1:name>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>EN</ts1:language>
                            <ts1:type>null</ts1:type>
                            <ts1:version>1</ts1:version>
                        </ts1:companyForms>
                        <ts1:detailsUri>null</ts1:detailsUri>
                        <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                        <ts1:names>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>3</ts1:source>
                            <ts1:order>0</ts1:order>
                            <ts1:name>Asunto Osakeyhtiö Teijon Metsänvartija</ts1:name>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:version>1</ts1:version>
                        </ts1:names>
                        <ts1:name>Asunto Osakeyhtiö Teijon Metsänvartija</ts1:name>
                        <ts1:liquidations>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>3</ts1:source>
                            <ts1:description>null</ts1:description>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>null</ts1:language>
                            <ts1:type>TP</ts1:type>
                            <ts1:version>1</ts1:version>
                        </ts1:liquidations>
                        <ts1:registeredEntries>
                            <ts1:register>1</ts1:register>
                            <ts1:authority>2</ts1:authority>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:status>2</ts1:status>
                            <ts1:description>null</ts1:description>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>null</ts1:language>
                            <ts1:statusDate>2014-12-30</ts1:statusDate>
                        </ts1:registeredEntries>
                        <ts1:contactDetails>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>0</ts1:source>
                            <ts1:value>0400815565</ts1:value>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>FI</ts1:language>
                            <ts1:type>Matkapuhelin</ts1:type>
                            <ts1:version>1</ts1:version>
                        </ts1:contactDetails>
                        <ts1:contactDetails>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>0</ts1:source>
                            <ts1:value>0400815565</ts1:value>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>SE</ts1:language>
                            <ts1:type>Mobiltelefon</ts1:type>
                            <ts1:version>1</ts1:version>
                        </ts1:contactDetails>
                        <ts1:businessId>2663307-6</ts1:businessId>
                        <ts1:addresses>
                            <ts1:careOf>null</ts1:careOf>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>0</ts1:source>
                            <ts1:street>Teijontie 23</ts1:street>
                            <ts1:postCode>25570</ts1:postCode>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>FI</ts1:language>
                            <ts1:type>2</ts1:type>
                            <ts1:country>null</ts1:country>
                            <ts1:city>TEIJO</ts1:city>
                            <ts1:version>1</ts1:version>
                        </ts1:addresses>
                        <ts1:addresses>
                            <ts1:careOf>null</ts1:careOf>
                            <ts1:registrationDate>2014-12-30</ts1:registrationDate>
                            <ts1:source>0</ts1:source>
                            <ts1:street>Teijontie 23</ts1:street>
                            <ts1:postCode>25570</ts1:postCode>
                            <ts1:endDate>null</ts1:endDate>
                            <ts1:language>FI</ts1:language>
                            <ts1:type>1</ts1:type>
                            <ts1:country>null</ts1:country>
                            <ts1:city>TEIJO</ts1:city>
                            <ts1:version>1</ts1:version>
                        </ts1:addresses>
                        <ts1:companyForm>OY</ts1:companyForm>
                    </ts1:results>
                    <ts1:resultsFrom>0</ts1:resultsFrom>
                    <ts1:totalResults>-1</ts1:totalResults>
                    <ts1:previousResultsUri>null</ts1:previousResultsUri>
                    <ts1:type>fi.prh.opendata.bis</ts1:type>
                    <ts1:version>1</ts1:version>
                </ts1:responseData>
            </ts1:response>
        </ts1:getCompanyResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

#### Finto : Finnish Thesaurus and Ontology Service - Search

API documentation: http://api.finto.fi/
#### Consumer Gateway

Configuration:
```
5.id=FI_PILOT.GOV.1019125-0.Demo2Service.fintoService.v1
5.path=/api.finto.fi/rest/v1/search/
5.verb=get
```

Service request:

* Method: ```GET```
* Accept: ```text/xml``` or ```application/json```
* X-XRd-MessageId: ```1```
* X-XRd-UserId: ```test```

```N.B.``` Accept header is ignored and response is always returned in ```application/json```, because Provider Gateway is configured to return the response as SOAP attachment, and conversions are not supported for attachments. 

URL:
```
http://localhost:8080/rest-gateway-0.0.4/Consumer/api.finto.fi/rest/v1/search/?query=cat&lang=en
```

Browser-based access:

```
http://localhost:8080/rest-gateway-0.0.4/Consumer/FI_PILOT.GOV.1019125-0.Demo2Service.fintoService.v1/?query=cat&lang=en&X-XRd-UserId=test&X-XRd-MessageId=1&X-XRd-NamespaceSerialize=http://vrk-test.x-road.fi/producer&X-XRd-NamespacePrefixSerialize=ks
```

Consumer  Gateway response:

```
{
    "@context": {
        "skos": "http://www.w3.org/2004/02/skos/core#",
        "onki": "http://schema.onki.fi/onki#",
        "uri": "@id",
        "type": "@type",
        "results": {
            "@id": "onki:results",
            "@container": "@list"
        },
        "prefLabel": "skos:prefLabel",
        "altLabel": "skos:altLabel",
        "hiddenLabel": "skos:hiddenLabel",
        "broader": "skos:broader",
        "@language": null
    },
    "uri": "",
    "results": [
        {
            "uri": "http://iconclass.org/34B12",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "ic"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "afo",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/afo/p1287",
            "type": [
                "skos:Concept",
                "http://www.yso.fi/onto/afo-meta/Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "afo"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "juho",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "jupo",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "kauno",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "kito",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/koko/p37252",
            "type": [
                "skos:Concept",
                "http://www.yso.fi/onto/afo-meta/Concept",
                "http://www.yso.fi/onto/kauno-meta/Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "koko"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "kto",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "kulo",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "liito",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "mero",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/mesh/D002415",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "Cats",
            "lang": "en",
            "hiddenLabel": "Cat",
            "vocab": "mesh"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "muso",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "puho",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "maotao",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/tero/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "tero"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "tsr",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "valo",
            "exvocab": "yso"
        },
        {
            "uri": "http://www.yso.fi/onto/yso/p19378",
            "type": [
                "skos:Concept"
            ],
            "prefLabel": "cat",
            "lang": "en",
            "vocab": "yso"
        }
    ]
}
```

#### Provider Gateway

Configuration:
```
5.id=FI_PILOT.GOV.1019125-0.Demo2Service.fintoService.v1
5.url=http://api.finto.fi/rest/v1/search
5.response.convert=false
5.response.attachment=true
```
Service request:

* Method: ```POST```
* Content-Type: ```text/xml```

```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>fintoService</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <test:fintoService xmlns:test="http://vrk-test.x-road.fi/producer">
            <test:request>
                <test:query>cat</test:query>
                <test:lang>en</test:lang>
            </test:request>
        </test:fintoService>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

API response URL: http://api.finto.fi/rest/v1/search?query=cat&lang=en

REST Gateway response:
```
------=_Part_0_1021174698.1420877957158
Content-Type: text/xml; charset=utf-8
     
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:id="http://x-road.eu/xsd/identifiers" xmlns:sdsb="http://x-road.eu/xsd/sdsb.xsd">
    <SOAP-ENV:Header>
        <sdsb:client id:objectType="SUBSYSTEM">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>0245437-2</id:memberCode>
            <id:subsystemCode>ConsumerService</id:subsystemCode>
        </sdsb:client>
        <sdsb:service id:objectType="SERVICE">
            <id:sdsbInstance>FI_PILOT</id:sdsbInstance>
            <id:memberClass>GOV</id:memberClass>
            <id:memberCode>1019125-0</id:memberCode>
            <id:subsystemCode>Demo2Service</id:subsystemCode>
            <id:serviceCode>fintoService</id:serviceCode>
            <id:serviceVersion>v1</id:serviceVersion>
        </sdsb:service>
        <sdsb:userId>test</sdsb:userId>
        <sdsb:id>0ba036ea-d612-4e74-bf73-59a6f15627c8</sdsb:id>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <ts1:fintoServiceResponse xmlns:ts1="http://vrk-test.x-road.fi/producer">
            <ts1:request>
                <ts1:query>cat</ts1:query>
                <ts1:lang>en</ts1:lang>
            </ts1:request>
            <ts1:response>
                <ts1:data href="response_data"/>
            </ts1:response>
        </ts1:fintoServiceResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
------=_Part_0_1021174698.1420877957158
Content-Type: application/json
Content-ID: response_data

{"@context":{"skos":"http:\/\/www.w3.org\/2004\/02\/skos\/core#","onki":"http:\/\/schema.onki.fi\/onki#","uri":"@id","type":"@type","results":{"@id":"onki:results","@container":"@list"},"prefLabel":"skos:prefLabel","altLabel":"skos:altLabel","hiddenLabel":"skos:hiddenLabel","broader":"skos:broader","@language":null},"uri":"","results":[{"uri":"http:\/\/iconclass.org\/34B12","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"ic"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"afo","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/afo\/p1287","type":["skos:Concept","http:\/\/www.yso.fi\/onto\/afo-meta\/Concept"],"prefLabel":"cat","lang":"en","vocab":"afo"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"juho","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"jupo","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"kauno","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"kito","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/koko\/p37252","type":["skos:Concept","http:\/\/www.yso.fi\/onto\/afo-meta\/Concept","http:\/\/www.yso.fi\/onto\/kauno-meta\/Concept"],"prefLabel":"cat","lang":"en","vocab":"koko"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"kto","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"kulo","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"liito","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"mero","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/mesh\/D002415","type":["skos:Concept"],"prefLabel":"Cats","lang":"en","hiddenLabel":"Cat","vocab":"mesh"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"muso","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"puho","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"maotao","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/tero\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"tero"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"tsr","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"valo","exvocab":"yso"},{"uri":"http:\/\/www.yso.fi\/onto\/yso\/p19378","type":["skos:Concept"],"prefLabel":"cat","lang":"en","vocab":"yso"}]}
------=_Part_0_1021174698.1420877957158--
```