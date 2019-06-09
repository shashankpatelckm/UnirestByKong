---
layout: default
title: Documentation
rightmenu: true
---

<div id="spy-nav" class="right-menu" markdown="1">
* [Requests](#requests)
* [Route Parameters](#route-parameters)
* [Query Parameters](#query-parameters)
* [Headers](#headers)
* [Basic Authentication](#basic-authentication)
* [Body Data](#body-data)
* [Entity Bodies](#entity-bodies)
* [JSON Patch Bodies](#json-patch-bodies)
* [Basic Forms](#basic-forms)
* [File Uploads](#file-uploads)
* [Responses](#responses)
* [Empty Responses](#empty-responses)
* [String Responses](#string-responses)
* [Object Mapped Responses](#object-mapped-responses)
* [File Responses](#file-responses)

</div>

## Install With [Maven](https://mvnrepository.com/artifact/com.konghq/unirest-java)[:](https://repo.maven.apache.org/maven2/com/konghq/unirest-java/)
```xml
<dependency>
    <groupId>com.konghq</groupId>
    <artifactId>unirest-java</artifactId>
    <version>2.3.02</version>
</dependency>
```

### Upgrading from Previous Versions
See the [Upgrade Guide](UPGRADE_GUIDE.md)


# Requests
So you're probably wondering how using Unirest makes creating requests in Java easier, here is a basic POST request that will explain everything:

```java
HttpResponse<JsonNode> response = Unirest.post("http://httpbin.org/post")
      .header("accept", "application/json")
      .queryString("apiKey", "123")
      .field("parameter", "value")
      .field("foo", "bar")
      .asString();
```

Requests are made when `as[Type]()` is invoked, possible types include `Json`, `String`, `Object` `Empty` and `File`.


## Route Parameters
Sometimes you want to add dynamic parameters in the URL, you can easily do that by adding a placeholder in the URL, and then by setting the route parameters with the `routeParam` function, like:

```java
Unirest.get("http://httpbin.org/{fruit}")
     .routeParam("fruit", "apple")
     .asString();

// Results in `http://httpbin.org/apple`
```
Basically the placeholder `{method}` will be replaced with `apple`.

The placeholder's format is as easy as: `{custom_name}`

All param values will be URL-Encoded for you

## Query Parameters
Query-string params can be built up one by one

```java
Unirest.get("http://httpbin.org")
                .queryString("fruit", "apple")
                .queryString("droid", "R2D2")
                .asString();

// Results in "http://httpbin.org?fruit=apple&droid=R2D2"
```

Again all param values will be URL-Encoded.

You can also pass in query strings as arrays and maps:
```java
Unirest.get("http://httpbin.org")
        .queryString("fruit", Arrays.asList("apple", "orange"))
        .queryString(ImmutableMap.of("droid", "R2D2", "beatle", "Ringo"))
        .asString();

 // Results in "http://httpbin.org?fruit=apple&fruit=orange&droid=R2D2&beatle=Ringo"
```

## Headers
Request headers can be added with the ```header``` method.
```java
Unirest.get("http://httpbin.org")
            .header("Accept", "application/json")
            .header("x-custom-header", "hello")
            .asString();
```

### Basic Authentication
Unirest exposes a shortcut for doing basic auth when you need to. Unirest handles the Base64 encoding part.
Please make sure you are always doing this over HTTPS!

```java
Unirest.get("http://httpbin.org")
            .basicAuth("user", "password1!")
            .asString();

// this adds the header "Authorization: Basic dXNlcjpwYXNzd29yZDEh"
```

## Body Data

### Entity Bodies
You can post entity objects as the full body easily. This is the default behavior of most REST services.

Unless you specify otherwise the default ```Content-Type``` is ```text/plain; charset=UTF-8```

```java
Unirest.post("http://httpbin.org")
                .body("This is the entire body")
                .asEmpty();
```

You can also post as a Object that is serialized using a configured ObjectMapper. (see [Object Mappers](#object-mappers) for implementation details.

```java
// Object Mappers can be configured globally, per config, or on a per-request basis.
// We will set it up in the global config here:

Unirest.config().setObjectMapper(new JacksonObjectMapper());

Unirest.post("http://httpbin.org")
            .header("Content-Type", "application/json")
            .body(new SomeUserObject("Bob"))
            .asEmpty();

// This will use Jackson to serialize the object into JSON.
```

### JSON Patch Bodies
Unirest has full native support for JSON Patch requests (RFC-6902 see http://jsonpatch.com/)
Per the spec, the default ```Content-Type``` for json-patch is ```application/json-patch+json```

```java
     Unirest.jsonPatch("http://httpbin.org")
            .add("/fruits/-", "Apple")
            .remove("/bugs")
            .replace("/lastname", "Flintstone")
            .test("/firstname", "Fred")
            .move("/old/location", "/new/location")
            .copy("/original/location", "/new/location")
            .asJson();
```
will send a request with a body of
```js
  [
     {"op":"add","path":"/fruits/-","value":"Apple"},
     {"op":"remove","path":"/bugs"},
     {"op":"replace","path":"/lastname","value":"Flintstone"},
     {"op":"test","path":"/firstname","value":"Fred"},
     {"op":"move","path":"/new/location","from":"/old/location"},
     {"op":"copy","path":"/new/location","from":"/original/location"}
  ]
```

### Basic Forms
Basic http name value body params can be passed with simple field calls.
The ```Content-Type``` for this type of request is defaulted to  ```application/x-www-form-urlencoded```

```java
Unirest.post("http://httpbin.org")
       .field("fruit", "apple")
       .field("droid", "R2D2")
       .asEmpty();

  // This will post a simple name-value pair body the same as a HTML form. This looks like
  // `fruit=apple&droid=R2D2'
```

### File Uploads
You can also post binary data in a form. Like a file.

The ```Content-Type``` for this type of request is defaulted to  ```multipart/form-data```

```java
Unirest.post("http://httpbin.org")
       .field("upload", new File("/MyFile.zip"))
       .asEmpty();
```

For large files you may want to use a InputStream. Pass it a file name if you want one.
We are using a FileInputStream here but it can actually be any kind of InputStream.

```java
InputStream file = new FileInputStream(new File("/MyFile.zip"));

Unirest.post("http://httpbin.org")
       .field("upload", file, "MyFile.zip")
       .asEmpty();
```

# Responses
Unirest makes the actual request the moment you invoke of it's ```as[type]``` method. These methods also inform Unirest what type to map the response to. Options are ```Empty```, ```String```, ```File```, ```Object```, and ```Json```.

The response returns as a ```HttpResponse<T>``` where the ```HttpResponse``` object has all of the common response data like status and headers. The Body (if present) can be accessed via the desired type with the ```.body()``` method. 

## Empty Responses
If you aren't expecting a body back, ```asEmpty``` is the easiest choice. You will still get back response information like status and headers.

```java
HttpResponse response = Unirest.delete("http://httpbin.org").asEmpty()
```

## String Responses
The next easiest response type is String. You can do whatever you want with it after that.

```java
String body = Unirest.get("http://httpbin.org")
					 .asString()
					 .body();
```

## Object Mapped Responses
Most of the time when consuming RESTful services you probably want to map the response into an object. For this you need to provide the Unirest configuration with a implementation of ```ObjectMapper``` (see [Object Mappers](#object-mappers) for details.). Unirest has several modules that can be included for popular JSON object mappers like Jackson and GSON.

Before an `asObject(Class)` it is necessary to provide a custom implementation of the `ObjectMapper` interface. This should be done only the first time, as the instance of the ObjectMapper will be shared globally.

Unirest offers a few plug-ins implementing popular object mappers like Jackson and Gson. See [mvn central](https://mvnrepository.com/artifact/com.konghq) for details.

For example, serializing Json from\to Object using the popular Jackson ObjectMapper takes only few lines

```java
// Only one time
Unirest.config().setObjectMapper(new JacksonObjectMapper());

// Response to Object
Book book = Unirest.get("http://httpbin.org/books/1")
                   .asObject(Book.class)
                   .getBody();

Author author = Unirest.get("http://httpbin.org/books/{id}/author")
                       .routeParam("id", bookObject.getId())
                       .asObject(Author.class)
                       .getBody();
```

### Errors in Object or JSON parsing
You can't always get what you want. And sometimes results you get from web services will not map into what you expect them to.
When this happens with a ```asObject``` or ```asJson``` request the resulting body will be null, but the response object will contain a ParsingException that allows you to get the error and the original body for inspection.

```java
UnirestParsingException ex = response.getParsingError().get();

ex.getOriginalBody(); // Has the original body as a string.
ex.getMessage(); // Will have the parsing exception.
ex.getCause(); // of course will have the original parsing exception itself.
```

## File Responses
Sometimes you just want to download a file, or maybe capture the response body into a file. Unirest can do both. Just tell Unirest where you want to put the file.

```java
File result = Unirest.get("http://some.file.location/file.zip")
                .asFile("/disk/location/file.zip")
                .getBody();
```



#### Asynchronous Requests
Sometimes, well most of the time, you want your application to be asynchronous and not block, Unirest supports this in Java using anonymous callbacks, or direct method placement:

```java
CompletableFuture<HttpResponse<JsonNode>> future = Unirest.post("http://httpbin.org/post")
  .header("accept", "application/json")
  .field("param1", "value1")
  .field("param2", "value2")
  .asJsonAsync(response -> {
        int code = response.getStatus();
        JsonNode body = response.getBody();
    });
```

#### Custom mappings and handling large responses
Most response methods (```asString```, ```asJson```, and even ```asBinary```) read the entire
response stream into memory. In order to read the original stream and handle large responses you
can use several functional methods like:

```java
   Map r = Unirest.get(MockServer.GET)
                .queryString("foo", "bar")
                .asObject(i -> new Gson().fromJson(i.getContentReader(), HashMap.class))
                .getBody();

```

or consumers:

```java

         Unirest.get(MockServer.GET)
                .thenConsumeAsync(r -> {
                       // something like writing a file to disk
                });
```

# Configuration
Previous versions of unirest had configuration split across several different places. Sometimes it was done on ```Unirest```, sometimes it was done on ```Option```, sometimes it was somewhere else.
All configuration is now done through ```Unirest.config()```

#### Unirest.config()
Unirest config allows easy access to build a configuration just like you would build a request:

```java
    Unirest.config()
           .socketTimeout(500)
           .connectTimeout(1000)
           .concurrency(10, 5)
           .proxy(new Proxy("https://proxy"))
           .setDefaultHeader("Accept", "application/json")
           .followRedirects(false)
           .enableCookieManagement(false)
           .addInterceptor(new MyCustomInterceptor());
```

#### Config Options

| Builder Method  | Impact | Default |
| ------------- | ------------- | ------------- |
| ```socketTimeout(int)``` | Sets the socket timeout for all requests in millis  | 60000 |
| ```connectTimeout(int)``` | Sets the connection timeout for all requests in millis  | 10000 |
| ```concurrency(int, int)``` | Sets concurrency rates; max total, max per route  | 200, 20 |
| ```proxy(proxy)``` | Sets a proxy object for negotiating proxy servers. Can include auth credentials  |  |
| ```setDefaultHeader(String, String)``` | Sets  a default header. Will overwrite if it exists  |  |
| ```setDefaultHeader(String, Supplier<String>)``` | Sets a default header by supplier. Good for setting trace tokens for microservice architectures. Will overwrite if it exists  |  |
| ```addDefaultHeader(String, String)``` | Adds a default header. Multiple for the same name can exist  |  |
| ```addDefaultHeader(String, Supplier<String>)``` | Add a default header by supplier. Good for setting trace tokens for microservice architectures.  |  |
| ```setDefaultBasicAuth(String, String)``` | Add a default Basic Auth Header |  |
| ```followRedirects(boolean)``` | toggle following redirects | true |
| ```enableCookieManagement(boolean)``` | toggle accepting and storing cookies | true |
| ```cookieSpec(String)``` | set a cookie policy. Acceptable values: 'default' (same as Netscape), 'netscape', 'ignoreCookies', 'standard' (RFC 6265 interoprability profile) , 'standard-strict' (RFC 6265 strict profile) | default |
| ```automaticRetries(boolean)``` | toggle disabling automatic retries (up to 4 times) for socket timeouts | true |
| ```verifySsl(boolean)``` |toggle enforcing SSL | true |
| ```addShutdownHook(boolean)``` | toggle to add the clients to the system shutdown hooks automatically | false |
| ```clientCertificateStore(String,String)``` | Add a PKCS12 KeyStore by path for doing client certificates |  |
| ```clientCertificateStore(KeyStore,String)``` | Add a PKCS12 KeyStore for doing client certificates |  |



#### Changing the config
Changing Unirest's config should ideally be done once, or rarely. There are several background threads spawned by both Unirest itself and Apache HttpAsyncClient. Once Unirest has been activated configuration options that are involved in creating the client cannot be changed without an explicit shutdown or reset.

```Java
     Unirest.config()
            .reset()
            .connectTimeout(5000)
```

#### Setting custom Apache Client
You can set your own custom Apache HttpClient and HttpAsyncClient. Note that Unirest settings like timeouts or interceptors are not applied to custom clients.

```java
     Unirest.config()
            .httpClient(myClient)
            .asyncClient(myAsyncClient)
```

#### Multiple Configuration Instances
As usual, Unirest maintains a primary single instance. Sometimes you might want different configurations for different systems. You might also want an instance rather than a static context for testing purposes.

```java

    // this returns the same instance used by Unirest.get("http://somewhere/")
    UnirestInstance unirest = Unirest.primaryInstance();
    // It can be configured and used just like the static context
    unirest.config().connectTimeout(5000);
    String result = unirest.get("http://foo").asString().getBody();

    // You can also get a whole new instance
    UnirestInstance unirest = Unirest.spawnInstance();
```

**WARNING!** If you get a new instance of unirest YOU are responsible for shutting it down when the JVM shuts down. It is not tracked or shut down by ```Unirest.shutDown();```

# Object Mappers
todo


## Exiting an application

Unirest starts a background event loop and your Java application won't be able to exit until you manually shutdown all the threads by invoking:

```java
Unirest.shutdown();
```

Once shutdown, using Unirest again will re-init the system

