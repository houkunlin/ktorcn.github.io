---
title: Testing
category: clients
permalink: /clients/http-client/testing.html
caption: Testing Http Client (MockEngine) 
---

Ktor exposes a `MockEngine` for the HttpClient. This engine allows to simulate HTTP calls without
actually connecting to the endpoint. It allows to set a code block, that can handle the request,
and generates a response.

{% include artifact.html kind="engine" class="io.ktor.client.engine.mock.MockEngine" artifact="io.ktor:ktor-client-mock:$ktor_version,io.ktor:ktor-client-mock-jvm:$ktor_version,io.ktor:ktor-client-mock-js:$ktor_version,io.ktor:ktor-client-mock-native:$ktor_version" test="true" %}

## Usage

The usage is very simple: the MockEngine class has a [`operator invoke`](https://www.kotlincn.net/docs/reference/operator-overloading.html#invoke) method,
that receives a block/callback that will handle the request. This callback receives an `HttpRequest` as this, a `HttpClientCall` as a parameter,
and must return a `MockHttpResponse`. The MockHttpResponse, includes the HTTP Status Code, a ByteReadChannel with the body, and a set of headers.

A sample illustrating this:

```kotlin
val httpMockEngine = MockEngine { call -> // this: HttpRequest, call: HttpClientCall
    when (url.fullUrl) {
        "https://example.org/" -> {
            MockHttpResponse(
                call,
                HttpStatusCode.OK,
                ByteReadChannel("Hello World!".toByteArray(Charsets.UTF_8)),
                headersOf("Content-Type" to listOf(ContentType.Text.Plain.toString()))
            )
        }
        else -> {
            error("Unhandled ${url.fullUrl}")
        }
    }
}

val client = HttpClient(httpMockEngine)

private val Url.hostWithPortIfRequired: String get() = if (port == protocol.defaultPort) host else hostWithPort
private val Url.fullUrl: String get() = "${protocol.name}://$hostWithPortIfRequired$fullPath"
```
