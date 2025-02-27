Change Log
==========

## Version 3.12.15

_2022-09-26_

* Fix: Flatten pom file to make okhttp module independent.

## Version 3.12.14

_2022-09-20_

* Fix: Don't expose HTTP headers when setting invalid value. Fix [information exposure](https://security.snyk.io/vuln/SNYK-JAVA-COMSQUAREUPOKHTTP3-2958044).

## Version 3.12.0

_2018-11-16_

 *  **OkHttp now supports TLS 1.3.** This requires either Conscrypt or Java 11+.

 *  **Proxy authenticators are now asked for preemptive authentication.** OkHttp will now request
    authentication credentials before creating TLS tunnels through HTTP proxies (HTTP `CONNECT`).
    Authenticators should identify preemptive authentications by the presence of a challenge whose
    scheme is "OkHttp-Preemptive".

 *  **OkHttp now offers full-operation timeouts.** This sets a limit on how long the entire call may
    take and covers resolving DNS, connecting, writing the request body, server processing, and
    reading the full response body. If a call requires redirects or retries all must complete within
    one timeout period.

    Use `OkHttpClient.Builder.callTimeout()` to specify the default duration and `Call.timeout()` to
    specify the timeout of an individual call.

 *  New: Return values and fields are now non-null unless otherwise annotated.
 *  New: `LoggingEventListener` makes it easy to get basic visibility into a call's performance.
    This class is in the `logging-interceptor` artifact.
 *  New: `Headers.Builder.addUnsafeNonAscii()` allows non-ASCII values to be added without an
    immediate exception.
 *  New: Headers can be redacted in `HttpLoggingInterceptor`.
 *  New: `Headers.Builder` now accepts dates.
 *  New: OkHttp now accepts `java.time.Duration` for timeouts on Java 8+ and Android 26+.
 *  New: `Challenge` includes all authentication parameters.
 *  New: Upgrade to BouncyCastle 1.60, Conscrypt 1.4.0, and Okio 1.15.0. We don't yet require
    Kotlin-friendly Okio 2.x but OkHttp works fine with that series.

    ```kotlin
    implementation("org.bouncycastle:bcprov-jdk15on:1.60")
    implementation("org.conscrypt:conscrypt-openjdk-uber:1.4.0")
    implementation("com.squareup.okio:okio:1.15.0")
    ```

 *  Fix: Handle dispatcher executor shutdowns gracefully. When there aren't any threads to carry a
    call its callback now gets a `RejectedExecutionException`.
 *  Fix: Don't permanently cache responses with `Cache-Control: immutable`. We misunderstood the
    original `immutable` proposal!
 *  Fix: Change `Authenticator`'s `Route` parameter to be nullable. This was marked as non-null but
    could be called with null in some cases.
 *  Fix: Don't create malformed URLs when `MockWebServer` is reached via an IPv6 address.
 *  Fix: Don't crash if the system default authenticator is null.
 *  Fix: Don't crash generating elliptic curve certificates on Android.
 *  Fix: Don't crash doing platform detection on RoboVM.
 *  Fix: Don't leak socket connections when web socket upgrades fail.


## Version 3.11.0

_2018-07-12_

 *  **OkHttp's new okhttp-tls submodule tames HTTPS and TLS.**

    `HeldCertificate` is a TLS certificate and its private key. Generate a certificate with its
    builder then use it to sign another certificate or perform a TLS handshake. The
    `certificatePem()` method encodes the certificate in the familiar PEM format
    (`--- BEGIN CERTIFICATE ---`); the `privateKeyPkcs8Pem()` does likewise for the private key.

    `HandshakeCertificates` holds the TLS certificates required for a TLS handshake. On the server
    it keeps your `HeldCertificate` and its chain. On the client it keeps the root certificates
    that are trusted to sign a server's certificate chain. `HandshakeCertificates` also works with
    mutual TLS where these roles are reversed.

    These classes make it possible to enable HTTPS in MockWebServer in [just a few lines of
    code][https_server_sample].

 *  **OkHttp now supports prior knowledge cleartext HTTP/2.** Enable this by setting
    `Protocol.H2_PRIOR_KNOWLEDGE` as the lone protocol on an `OkHttpClient.Builder`. This mode
    only supports `http:` URLs and is best suited in closed environments where HTTPS is
    inappropriate.

 *  New: `HttpUrl.get(String)` is an alternative to `HttpUrl.parse(String)` that throws an exception
    when the URL is malformed instead of returning null. Use this to avoid checking for null in
    situations where the input is known to be well-formed. We've also added `MediaType.get(String)`
    which is an exception-throwing alternative to `MediaType.parse(String)`.
 *  New: The `EventListener` API previewed in OkHttp 3.9 has graduated to a stable API. Use this
    interface to track metrics and monitor HTTP requests' size and duration.
 *  New: `okhttp-dnsoverhttps` is an experimental API for doing DNS queries over HTTPS. Using HTTPS
    for DNS offers better security and potentially better performance. This feature is a preview:
    the API is subject to change.
 *  New: `okhttp-sse` is an early preview of Server-Sent Events (SSE). This feature is incomplete
    and is only suitable for experimental use.
 *  New: MockWebServer now supports client authentication (mutual TLS). Call `requestClientAuth()`
    to permit an optional client certificate or `requireClientAuth()` to require one.
 *  New: `RecordedRequest.getHandshake()` returns the HTTPS handshake of a request sent to
    `MockWebServer`.
 *  Fix: Honor the `MockResponse` header delay in MockWebServer.
 *  Fix: Don't release HTTP/2 connections that have multiple canceled calls. We had a bug where
    canceling calls would cause the shared HTTP/2 connection to be unnecessarily released. This
    harmed connection reuse.
 *  Fix: Ensure canceled and discarded HTTP/2 data is not permanently counted against the limited
    flow control window. We had a few bugs where window size accounting was broken when streams
    were canceled or reset.
 *  Fix: Recover gracefully if the TLS session returns an unexpected version (`NONE`) or cipher
    suite (`SSL_NULL_WITH_NULL_NULL`).
 *  Fix: Don't change Conscrypt configuration globally. We migrated from a process-wide setting to
    configuring only OkHttp's TLS sockets.
 *  Fix: Prefer TLSv1.2 where it is available. On certain older platforms it is necessary to opt-in
    to TLSv1.2.
 *  New: `Request.tag()` permits multiple tags. Use a `Class<?>` as a key to identify tags. Note
    that `tag()` now returns null if the request has no tag. Previously this would return the
    request itself.
 *  New: `Headers.Builder.addAll(Headers)`.
 *  New: `ResponseBody.create(MediaType, ByteString)`.
 *  New: Embed R8/ProGuard rules in the jar. These will be applied automatically by R8.
 *  Fix: Release the connection if `Authenticator` throws an exception.
 *  Fix: Change the declaration of `OkHttpClient.cache()` to return a `@Nullable Cache`. The return
    value has always been nullable but it wasn't declared properly.
 *  Fix: Reverse suppression of connect exceptions. When both a call and its retry fail, we now
    throw the initial exception which is most likely to be actionable.
 *  Fix: Retain interrupted state when throwing `InterruptedIOException`. A single interrupt should
    now be sufficient to break out an in-flight OkHttp call.
 *  Fix: Don't drop a call to `EventListener.callEnd()` when the response body is consumed inside an
    interceptor.


## Version 3.10.0

_2018-02-24_

 *  **The pingInterval() feature now aggressively checks connectivity for web
    sockets and HTTP/2 connections.**

    Previously if you configured a ping interval that would cause OkHttp to send
    pings, but it did not track whether the reply pongs were received. With this
    update OkHttp requires that every ping receive a response: if it does not
    the connection will be closed and the listener's `onFailure()` method will
    be called.

    Web sockets have always been had pings, but pings on HTTP/2 connections is
    new in this release. Pings are used for connections that are busy carrying
    calls and for idle connections in the connection pool. (Pings do not impact
    when pooled connections are evicted).

    If you have a configured ping interval, you should confirm that it is long
    enough for a roundtrip from client to server. If your ping interval is too
    short, slow connections may be misinterpreted as failed connections. A ping
    interval of 30 seconds is reasonable for most use cases.

 *  **OkHttp now supports [Conscrypt][conscrypt].** Conscrypt is a Java Security
    Provider that integrates BoringSSL into the Java platform. Conscrypt
    supports more cipher suites than the JVM’s default provider and may also
    execute more efficiently.

    To use it, first register a [Conscrypt dependency][conscrypt_dependency] in
    your build system.

    OkHttp will use Conscrypt if you set the `okhttp.platform` system property
    to `conscrypt`.

    Alternatively, OkHttp will also use Conscrypt if you install it as your
    preferred security provider. To do so, add the following code to execute
    before you create your `OkHttpClient`.

    ```
    Security.insertProviderAt(
        new org.conscrypt.OpenSSLProvider(), 1);
    ```

    Conscrypt is the bundled security provider on Android so it is not necessary
    to configure it on that platform.

 *  New: `HttpUrl.addQueryParameter()` percent-escapes more characters.
    Previously several ASCII punctuation characters were not percent-escaped
    when used with this method. This does not impact already-encoded query
    parameters in APIs like `HttpUrl.parse()` and
    `HttpUrl.Builder.addEncodedQueryParameter()`.
 *  New: CBC-mode ECDSA cipher suites have been removed from OkHttp's default
    configuration: `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA` and
    `TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA`. This tracks a [Chromium
    change][remove_cbc_ecdsa] to remove these cipher suites because they are
    fragile and rarely-used.
 *  New: Don't fall back to common name (CN) verification for hostnames. This
    behavior was deprecated with RFC 2818 in May 2000 and was recently dropped
    from major web browsers.
 *  New: Honor the `Retry-After` response header. HTTP 503 (Unavailable)
    responses are retried automatically if this header is present and its delay
    is 0 seconds. HTTP 408 (Client Timeout) responses are retried automatically
    if the header is absent or its delay is 0 seconds.
 *  New: Allow request bodies for all HTTP methods except GET and HEAD.
 *  New: Automatic module name of `okhttp3` for use with the Java Platform
    Module System.
 *  New: Log gzipped bodies when `HttpLoggingInterceptor` is used as a network
    interceptor.
 *  New: `Protocol.QUIC` constant. This protocol is not supported but this
    constant is included for completeness.
 *  New: Upgrade to Okio 1.14.0.

     ```xml
     <dependency>
       <groupId>com.squareup.okio</groupId>
       <artifactId>okio</artifactId>
       <version>1.14.0</version>
     </dependency>

     com.squareup.okio:okio:1.14.0
     ```

 *  Fix: Handle `HTTP/1.1 100 Continue` status lines, even on requests that did
    not send the `Expect: continue` request header.
 *  Fix: Do not count web sockets toward the dispatcher's per-host connection
    limit.
 *  Fix: Avoid using invalid HTTPS sessions. This prevents OkHttp from crashing
    with the error, `Unexpected TLS version: NONE`.
 *  Fix: Don't corrupt the response cache when a 304 (Not Modified) response
    overrides the stored "Content-Encoding" header.
 *  Fix: Gracefully shut down the HTTP/2 connection before it exhausts the
    namespace of stream IDs (~536 million streams).
 *  Fix: Never pass a null `Route` to `Authenticator`. There was a bug where
    routes were omitted for eagerly-closed connections.

## Version 3.9.1

_2017-11-18_

 *  New: Recover gracefully when Android's DNS crashes with an unexpected
    `NullPointerException`.
 *  New: Recover gracefully when Android's socket connections crash with an
    unexpected `ClassCastException`.
 *  Fix: Don't include the URL's fragment in `encodedQuery()` when the query
    itself is empty.

## Version 3.9.0

_2017-09-03_

 *  **Interceptors are more capable.** The `Chain` interface now offers access
    to the call and can adjust all call timeouts. Note that this change is
    source-incompatible for code that implements the `Chain` interface.
    We don't expect this to be a problem in practice!

 *  **OkHttp has an experimental new API for tracking metrics.** The new
    `EventListener` API is designed to help developers monitor HTTP requests'
    size and duration. This feature is an unstable preview: the API is subject
    to change, and the implementation is incomplete. This is a big new API we
    are eager for feedback.

 *  New: Support ALPN via Google Play Services' Dynamic Security Provider. This
    expands HTTP/2 support to older Android devices that have Google Play
    Services.
 *  New: Consider all routes when looking for candidate coalesced connections.
    This increases the likelihood that HTTP/2 connections will be shared.
 *  New: Authentication challenges and credentials now use a charset. Use this in
    your authenticator to support user names and passwords with non-ASCII
    characters.
 *  New: Accept a charset in `FormBody.Builder`. Previously form bodies were
    always UTF-8.
 *  New: Support the `immutable` cache-control directive.
 *  Fix: Don't crash when an HTTP/2 call is redirected while the connection is
    being shut down.
 *  Fix: Don't drop headers of healthy streams that raced with `GOAWAY` frames.
    This bug would cause HTTP/2 streams to occasional hang when the connection
    was shutting down.
 *  Fix: Honor `OkHttpClient.retryOnConnectionFailure()` when the response is a
    HTTP 408 Request Timeout. If retries are enabled, OkHttp will retry exactly
    once in response to a 408.
 *  Fix: Don't crash when reading the empty `HEAD` response body if it specifies
    a `Content-Length`.
 *  Fix: Don't crash if the thread is interrupted while reading the public
    suffix database.
 *  Fix: Use relative resource path when loading the public suffix database.
    Loading the resource using a path relative to the class prevents conflicts
    when the OkHttp classes are relocated (shaded) by allowing multiple private
    copies of the database.
 *  Fix: Accept cookies for URLs that have an IPv6 address for a host.
 *  Fix: Don't log the protocol (HTTP/1.1, h2) in HttpLoggingInterceptor if the
    protocol isn't negotiated yet! Previously we'd log HTTP/1.1 by default, and
    this was confusing.
 *  Fix: Omit the message from MockWebServer's HTTP/2 `:status` header.
 *  Fix: Handle 'Expect: 100 Continue' properly in MockWebServer.


## Version 3.8.1

_2017-06-18_

 *  Fix: Recover gracefully from stale coalesced connections. We had a bug where
    connection coalescing (introduced in OkHttp 3.7.0) and stale connection
    recovery could interact to cause a `NoSuchElementException` crash in the
    `RouteSelector`.


## Version 3.8.0

_2017-05-13_


 *  **OkHttp now uses `@Nullable` to annotate all possibly-null values.** We've
    added a compile-time dependency on the JSR 305 annotations. This is a
    [provided][maven_provided] dependency and does not need to be included in
    your build configuration, `.jar` file, or `.apk`. We use
    `@ParametersAreNonnullByDefault` and all parameters and return types are
    never null unless explicitly annotated `@Nullable`.

 *  **Warning: this release is source-incompatible for Kotlin users.**
    Nullability was previously ambiguous and lenient but now the compiler will
    enforce strict null checks.

 *  New: The response message is now non-null. This is the "Not Found" in the
    status line "HTTP 404 Not Found". If you are building responses
    programmatically (with `new Response.Builder()`) you must now always supply
    a message. An empty string `""` is permitted. This value was never null on
    responses returned by OkHttp itself, and it was an old mistake to permit
    application code to omit a message.

 *  The challenge's scheme and realm are now non-null. If you are calling
    `new Challenge(scheme, realm)` you must provide non-null values. These were
    never null in challenges created by OkHttp, but could have been null in
    application code that creates challenges.

 *  New: The `TlsVersion` of a `Handshake` is now non-null. If you are calling
    `Handshake.get()` with a null TLS version, you must instead now provide a
    non-null `TlsVersion`. Cache responses persisted prior to OkHttp 3.0 did not
    store a TLS version; for these unknown values the handshake is defaulted to
    `TlsVersion.SSL_3_0`.

 *  New: Upgrade to Okio 1.13.0.

     ```xml
     <dependency>
       <groupId>com.squareup.okio</groupId>
       <artifactId>okio</artifactId>
       <version>1.13.0</version>
     </dependency>

     com.squareup.okio:okio:1.13.0
     ```

 *  Fix: gracefully recover when Android 7.0's sockets throw an unexpected
    `NullPointerException`.

## Version 3.7.0

_2017-04-15_

 *  **OkHttp no longer recovers from TLS handshake failures by attempting a TLSv1 connection.**
    The fallback was necessary for servers that implemented version negotiation incorrectly. Now
    that 99.99% of servers do it right this fallback is obsolete.
 *  Fix: Do not honor cookies set on a public domain. Previously a malicious site could inject
    cookies on top-level domains like `co.uk` because our cookie parser didn't honor the [public
    suffix][public_suffix] list. Alongside this fix is a new API, `HttpUrl.topPrivateDomain()`,
    which returns the privately domain name if the URL has one.
 *  Fix: Change `MediaType.charset()` to return null for unexpected charsets.
 *  Fix: Don't skip cache invalidation if the invalidating response has no body.
 *  Fix: Don't use a cryptographic random number generator for web sockets. Some Android devices
    implement `SecureRandom` incorrectly!
 *  Fix: Correctly canonicalize IPv6 addresses in `HttpUrl`. This prevented OkHttp from trusting
    HTTPS certificates issued to certain IPv6 addresses.
 *  Fix: Don't reuse connections after an unsuccessful `Expect: 100-continue`.
 *  Fix: Handle either `TLS_` or `SSL_` prefixes for cipher suite names. This is necessary for
    IBM JVMs that use the `SSL_` prefix exclusively.
 *  Fix: Reject HTTP/2 data frames if the stream ID is 0.
 *  New: Upgrade to Okio 1.12.0.

     ```xml
     <dependency>
       <groupId>com.squareup.okio</groupId>
       <artifactId>okio</artifactId>
       <version>1.12.0</version>
     </dependency>

     com.squareup.okio:okio:1.12.0
     ```

 *  New: Connection coalescing. OkHttp may reuse HTTP/2 connections across calls that share an IP
    address and HTTPS certificate, even if their domain names are different.
 *  New: MockWebServer's `RecordedRequest` exposes the requested `HttpUrl` with `getRequestUrl()`.


## Version 3.6.0

_2017-01-29_

 *  Fix: Don't crash with a "cache is closed" error when there is an error initializing the cache.
 *  Fix: Calling `disconnect()` on a connecting `HttpUrlConnection` could cause it to retry in an
    infinite loop! This regression was introduced in OkHttp 2.7.0.
 *  Fix: Drop cookies that contain ASCII NULL and other bad characters. Previously such cookies
    would cause OkHttp to crash when they were included in a request.
 *  Fix: Release duplicated multiplexed connections. If we concurrently establish connections to an
    HTTP/2 server, close all but the first connection.
 *  Fix: Fail the HTTP/2 connection if first frame isn't `SETTINGS`.
 *  Fix: Forbid spaces in header names.
 *  Fix: Don't offer to do gzip if the request is partial.
 *  Fix: MockWebServer is now usable with JUnit 5. That update [broke the rules][junit_5_rules].
 *  New: Support `Expect: 100-continue` as a request header. Callers can use this header to
    pessimistically hold off on transmitting a request body until a server gives the go-ahead.
 *  New: Permit network interceptors to rewrite the host header for HTTP/2. This makes it possible
    to do domain fronting.
 *  New: charset support for `Credentials.basic()`.


## Version 3.5.0

_2016-11-30_

 *  **Web Sockets are now a stable feature of OkHttp.** Since being introduced as a beta feature in
    OkHttp 2.3 our web socket client has matured. Connect to a server's web socket with
    `OkHttpClient.newWebSocket()`, send messages with `send()`, and receive messages with the
    `WebSocketListener`.

    The `okhttp-ws` submodule is no longer available and `okhttp-ws` artifacts from previous
    releases of OkHttp are not compatible with OkHttp 3.5. When upgrading to the new package
    please note that the `WebSocket` and `WebSocketCall` classes have been merged. Sending messages
    is now asynchronous and they may be enqueued before the web socket is connected.

 *  **OkHttp no longer attempts a direct connection if the system's HTTP proxy fails.** This
    behavior was surprising because OkHttp was disregarding the user's specified configuration. If
    you need to customize proxy fallback behavior, implement your own `java.net.ProxySelector`.

 *  Fix: Support TLSv1.3 on devices that support it.

 *  Fix: Share pooled connections across equivalent `OkHttpClient` instances. Previous releases had
    a bug where a shared connection pool did not guarantee shared connections in some cases.
 *  Fix: Prefer the server's response body on all conditional cache misses. Previously we would
    return the cached response's body if it had a newer `Last-Modified` date.
 *  Fix: Update the stored timestamp on conditional cache hits.
 *  New: Optimized HTTP/2 request header encoding. More headers are HPACK-encoded and string
    literals are now Huffman-encoded.
 *  New: Expose `Part` headers and body in `Multipart`.
 *  New: Make `ResponseBody.string()` and `ResponseBody.charStream()` BOM-aware. If your HTTP
    response body begins with a [byte order mark][bom] it will be consumed and used to select a
    charset for the remaining bytes. Most applications should not need a byte order mark.

 *  New: Upgrade to Okio 1.11.0.

     ```xml
     <dependency>
       <groupId>com.squareup.okio</groupId>
       <artifactId>okio</artifactId>
       <version>1.11.0</version>
     </dependency>

     com.squareup.okio:okio:1.11.0
     ```

 *  Fix: Avoid sending empty HTTP/2 data frames when there is no request body.
 *  Fix: Add a leading `.` for better domain matching in `JavaNetCookieJar`.
 *  Fix: Gracefully recover from HTTP/2 connection shutdowns at start of request.
 *  Fix: Be lenient if a `MediaType`'s character set is `'single-quoted'`.
 *  Fix: Allow horizontal tab characters in header values.
 *  Fix: When parsing HTTP authentication headers permit challenge parameters in any order.


## Version 3.4.2

_2016-11-03_

 *  Fix: Recover gracefully when an HTTP/2 connection is shutdown. We had a
    bug where shutdown HTTP/2 connections were considered usable. This caused
    infinite loops when calls attempted to recover.


## Version 3.4.1

_2016-07-10_

 *  **Fix a major bug in encoding HTTP headers.** In 3.4.0 and 3.4.0-RC1 OkHttp
    had an off-by-one bug in our HPACK encoder. This bug could have caused the
    wrong headers to be emitted after a sequence of HTTP/2 requests! Everyone
    who is using OkHttp 3.4.0 or 3.4.0-RC1 should upgrade for this bug fix.


## Version 3.4.0

_2016-07-08_

 *  New: Support dynamic table size changes to HPACK Encoder.
 *  Fix: Use `TreeMap` in `Headers.toMultimap()`. This makes string lookups on
    the returned map case-insensitive.
 *  Fix: Don't share the OkHttpClient's `Dispatcher` in `HttpURLConnection`.


## Version 3.4.0-RC1

_2016-07-02_

 *  **We’ve rewritten HttpURLConnection and HttpsURLConnection.** Previously we
    shared a single HTTP engine between two frontend APIs: `HttpURLConnection`
    and `Call`. With this release we’ve rearranged things so that the
    `HttpURLConnection` frontend now delegates to the `Call` APIs internally.
    This has enabled substantial simplifications and optimizations in the OkHttp
    core for both frontends.

    For most HTTP requests the consequences of this change will be negligible.
    If your application uses `HttpURLConnection.connect()`,
    `setFixedLengthStreamingMode()`, or `setChunkedStreamingMode()`, OkHttp will
    now use a async dispatcher thread to establish the HTTP connection.

    We don’t expect this change to have any behavior or performance
    consequences. Regardless, please exercise your `OkUrlFactory` and
    `HttpURLConnection` code when applying this update.

 *  **Cipher suites may now have arbitrary names.** Previously `CipherSuite` was
    a Java enum and it was impossible to define new cipher suites without first
    upgrading OkHttp. With this change it is now a regular Java class with
    enum-like constants. Application code that uses enum methods on cipher
    suites (`ordinal()`, `name()`, etc.) will break with this change.

 *  Fix: `CertificatePinner` now matches canonicalized hostnames. Previously
    this was case sensitive. This change should also make it easier to configure
    certificate pinning for internationalized domain names.
 *  Fix: Don’t crash on non-ASCII `ETag` headers. Previously OkHttp would reject
    these headers when validating a cached response.
 *  Fix: Don’t allow remote peer to arbitrarily size the HPACK decoder dynamic
    table.
 *  Fix: Honor per-host configuration in Android’s network security config.
    Previously disabling cleartext for any host would disable cleartext for all
    hosts. Note that this setting is only available on Android 24+.
 *  New: HPACK compression is now dynamic. This should improve performance when
    transmitting request headers over HTTP/2.
 *  New: `Dispatcher.setIdleCallback()` can be used to signal when there are no
    calls in flight. This is useful for [testing with
    Espresso][okhttp_idling_resource].
 *  New: Upgrade to Okio 1.9.0.

     ```xml
     <dependency>
       <groupId>com.squareup.okio</groupId>
       <artifactId>okio</artifactId>
       <version>1.9.0</version>
     </dependency>
     ```


## Version 3.3.1

_2016-05-28_

 *  Fix: The plaintext check in HttpLoggingInterceptor incorrectly classified
    newline characters as control characters. This is fixed.
 *  Fix: Don't crash reading non-ASCII characters in HTTP/2 headers or in cached
    HTTP headers.
 *  Fix: Retain the response body when an attempt to open a web socket returns a
    non-101 response code.


## Version 3.3.0

_2016-05-24_

 *  New: `Response.sentRequestAtMillis()` and `receivedResponseAtMillis()`
    methods track the system's local time when network calls are made. These
    replace the `OkHttp-Sent-Millis` and `OkHttp-Received-Millis` headers that were
    present in earlier versions of OkHttp.
 *  New: Accept user-provided trust managers in `OkHttpClient.Builder`. This
    allows OkHttp to satisfy its TLS requirements directly. Otherwise OkHttp
    will use reflection to extract the `TrustManager` from the
    `SSLSocketFactory`.
 *  New: Support prerelease Java 9. This gets ALPN from the platform rather than
    relying on the alpn-boot bootclasspath override.
 *  New: `HttpLoggingInterceptor` now logs connection failures.
 *  New: Upgrade to Okio 1.8.0.

     ```xml
     <dependency>
       <groupId>com.squareup.okio</groupId>
       <artifactId>okio</artifactId>
       <version>1.8.0</version>
     </dependency>
     ```

 *  Fix: Gracefully recover from a failure to rebuild the cache journal.
 *  Fix: Don't corrupt cache entries when a cache entry is evicted while it is
    being updated.
 *  Fix: Make logging more consistent throughout OkHttp.
 *  Fix: Log plaintext bodies only. This uses simple heuristics to differentiate
    text from other data.
 *  Fix: Recover from `REFUSED_STREAM` errors in HTTP/2. This should improve
    interoperability with Nginx 1.10.0, which [refuses][nginx_959] streams
    created before HTTP/2 settings have been acknowledged.
 *  Fix: Improve recovery from failed routes.
 *  Fix: Accommodate tunneling proxies that close the connection after an auth
    challenge.
 *  Fix: Use the proxy authenticator when authenticating HTTP proxies. This
    regression was introduced in OkHttp 3.0.
 *  Fix: Fail fast if network interceptors transform the response body such that
    closing it doesn't also close the underlying stream. We had a bug where
    OkHttp would attempt to reuse a connection but couldn't because it was still
    held by a prior request.
 *  Fix: Ensure network interceptors always have access to the underlying
    connection.
 *  Fix: Use `X509TrustManagerExtensions` on Android 17+.
 *  Fix: Unblock waiting dispatchers on MockWebServer shutdown.


## Version 3.2.0

_2016-02-25_

 *  Fix: Change the certificate pinner to always build full chains. This
    prevents a potential crash when using certificate pinning with the Google
    Play Services security provider.
 *  Fix: Make IPv6 request lines consistent with Firefox and Chrome.
 *  Fix: Recover gracefully when trimming the response cache fails.
 *  New: Add multiple path segments using a single string in `HttpUrl.Builder`.
 *  New: Support SHA-256 pins in certificate pinner.


## Version 3.1.2

_2016-02-10_

 *  Fix: Don’t crash when finding the trust manager on Robolectric. We attempted
    to detect the host platform and got confused because Robolectric looks like
    Android but isn’t!
 *  Fix: Change `CertificatePinner` to skip sanitizing the certificate chain
    when no certificates were pinned. This avoids an SSL failure in insecure
    “trust everyone” configurations, such as when talking to a development
    HTTPS server that has a self-signed certificate.


## Version 3.1.1

_2016-02-07_

 *  Fix: Don't crash when finding the trust manager if the Play Services (GMS)
    security provider is installed.
 *  Fix: The previous release introduced a performance regression on Android,
    caused by looking up CA certificates. This is now fixed.


## Version 3.1.0

_2016-02-06_

 *  New: WebSockets now defer some writes. This should improve performance for
    some applications.
 *  New: Override `equals()` and `hashCode()` in our new cookie class. This
    class now defines equality by value rather than by reference.
 *  New: Handle 408 responses by retrying the request. This allows servers to
    direct clients to retry rather than failing permanently.
 *  New: Expose the framed protocol in `Connection`. Previously this would
    return the application-layer protocol (HTTP/1.1 or HTTP/1.0); now it always
    returns the wire-layer protocol (HTTP/2, SPDY/3.1, or HTTP/1.1).
 *  Fix: Permit the trusted CA root to be pinned by `CertificatePinner`.
 *  Fix: Silently ignore unknown HTTP/2 settings. Previously this would cause
    the entire connection to fail.
 *  Fix: Don’t crash on unexpected charsets in the logging interceptor.
 *  Fix: `OkHttpClient` is now non-final for the benefit of mocking frameworks.
    Mocking sophisticated classes like `OkHttpClient` is fragile and you
    shouldn’t do it. But if that’s how you want to live your life we won’t stand
    in your way!


## Version 3.0.1

_2016-01-14_

 *  Rollback OSGi support. This was causing library jars to include more classes
    than expected, which interfered with Gradle builds.


## Version 3.0.0

_2016-01-13_

This release commits to a stable 3.0 API. Read the 3.0.0-RC1 changes for advice
on upgrading from 2.x to 3.x.

 *  **The `Callback` interface now takes a `Call`**. This makes it easier to
    check if the call was canceled from within the callback. When migrating
    async calls to this new API, `Call` is now the first parameter for both
    `onResponse()` and `onFailure()`.
 *  Fix: handle multiple cookies in `JavaNetCookieJar` on Android.
 *  Fix: improve the default HTTP message in MockWebServer responses.
 *  Fix: don't leak file handles when a conditional GET throws.
 *  Fix: Use charset specified by the request body content type in OkHttp's
    logging interceptor.
 *  Fix: Don't eagerly release pools on cache hits.
 *  New: Make OkHttp OSGi ready.
 *  New: Add already-implemented interfaces Closeable and Flushable to the cache.

## Version 3.0.0-RC1

_2016-01-02_

OkHttp 3 is a major release focused on API simplicity and consistency. The API
changes are numerous but most are cosmetic. Applications should be able to
upgrade from the 2.x API to the 3.x API mechanically and without risk.

Because the release includes breaking API changes, we're changing the project's
package name from `com.squareup.okhttp` to `okhttp3`. This should make it
possible for large applications to migrate incrementally. The Maven group ID
is now `com.squareup.okhttp3`. For an explanation of this strategy, see Jake
Wharton's post, [Java Interoperability Policy for Major Version
Updates][major_versions].

This release obsoletes OkHttp 2.x, and all code that uses OkHttp's
`com.squareup.okhttp` package should upgrade to the `okhttp3` package. Libraries
that depend on OkHttp should upgrade quickly to prevent applications from being
stuck on the old version.

 *  **There is no longer a global singleton connection pool.** In OkHttp 2.x,
    all `OkHttpClient` instances shared a common connection pool by default.
    In OkHttp 3.x, each new `OkHttpClient` gets its own private connection pool.
    Applications should avoid creating many connection pools as doing so
    prevents connection reuse. Each connection pool holds its own set of
    connections alive so applications that have many pools also risk exhausting
    memory!

    The best practice in OkHttp 3 is to create a single OkHttpClient instance
    and share it throughout the application. Requests that needs a customized
    client should call `OkHttpClient.newBuilder()` on that shared instance.
    This allows customization without the drawbacks of separate connection
    pools.

 *  **OkHttpClient is now stateless.** In the 2.x API `OkHttpClient` had getters
    and setters. Internally each request was forced to make its own complete
    snapshot of the `OkHttpClient` instance to defend against racy configuration
    changes. In 3.x, `OkHttpClient` is now stateless and has a builder. Note
    that this class is not strictly immutable as it has stateful members like
    the connection pool and cache.

 *  **Get and Set prefixes are now avoided.** With ubiquitous builders
    throughout OkHttp these accessor prefixes aren't necessary. Previously
    OkHttp used _get_ and _set_ prefixes sporadically which make the API
    inconsistent and awkward to explore.

 *  **OkHttpClient now implements the new `Call.Factory` interface.** This
    interface will make your code easier to test. When you test code that makes
    HTTP requests, you can use this interface to replace the real `OkHttpClient`
    with your own mocks or fakes.

    The interface will also let you use OkHttp's API with another HTTP client's
    implementation. This is useful in sandboxed environments like Google App
    Engine.

 *  **OkHttp now does cookies.** We've replaced `java.net.CookieHandler` with
    a new interface, `CookieJar` and added our own `Cookie` model class. This
    new cookie follows the latest RFC and supports the same cookie attributes
    as modern web browsers.

 *  **Form and Multipart bodies are now modeled.** We've replaced the opaque
    `FormEncodingBuilder` with the more powerful `FormBody` and
    `FormBody.Builder` combo. Similarly we've upgraded `MultipartBuilder` into
    `MultipartBody`, `MultipartBody.Part`, and `MultipartBody.Builder`.

 *  **The Apache HTTP client and HttpURLConnection APIs are deprecated.** They
    continue to work as they always have, but we're moving everything to the new
    OkHttp 3 API. The `okhttp-apache` and `okhttp-urlconnection` modules should
    be only be used to accelerate a transition to OkHttp's request/response API.
    These deprecated modules will be dropped in an upcoming OkHttp 3.x release.

 *  **Canceling batches of calls is now the application's responsibility.**
    The API to cancel calls by tag has been removed and replaced with a more
    general mechanism. The dispatcher now exposes all in-flight calls via its
    `runningCalls()` and `queuedCalls()` methods. You can write code that
    selects calls by tag, host, or whatever, and invokes `Call.cancel()` on the
    ones that are no longer necessary.

 *  **OkHttp no longer uses the global `java.net.Authenticator` by default.**
    We've changed our `Authenticator` interface to authenticate web and proxy
    authentication failures through a single method. An adapter for the old
    authenticator is available in the `okhttp-urlconnection` module.

 *  Fix: Don't throw `IOException` on `ResponseBody.contentLength()` or `close()`.
 *  Fix: Never throw converting an `HttpUrl` to a `java.net.URI`. This changes
    the `uri()` method to handle malformed percent-escapes and characters
    forbidden by `URI`.
 *  Fix: When a connect times out, attempt an alternate route. Previously route
    selection was less efficient when differentiating failures.
 *  New: `Response.peekBody()` lets you access the response body without
    consuming it. This may be handy for interceptors!
 *  New: `HttpUrl.newBuilder()` resolves a link to a builder.
 *  New: Add the TLS version to the `Handshake`.
 *  New: Drop `Request.uri()` and `Request#urlString()`. Just use
    `Request.url().uri()` and `Request.url().toString()`.
 *  New: Add URL to HTTP response logging.
 *  New: Make `HttpUrl` the blessed URL method of `Request`.


## Version 2.7.5

_2016-02-25_

 *  Fix: Change the certificate pinner to always build full chains. This
    prevents a potential crash when using certificate pinning with the Google
    Play Services security provider.


## Version 2.7.4

_2016-02-07_

 *  Fix: Don't crash when finding the trust manager if the Play Services (GMS)
    security provider is installed.
 *  Fix: The previous release introduced a performance regression on Android,
    caused by looking up CA certificates. This is now fixed.


## Version 2.7.3

_2016-02-06_

 *  Fix: Permit the trusted CA root to be pinned by `CertificatePinner`.


## Version 2.7.2

_2016-01-07_

 *  Fix: Don't eagerly release stream allocations on cache hits. We might still
    need them to handle redirects.


## Version 2.7.1

_2016-01-01_

 *  Fix: Don't do a health check on newly-created connections. This is
    unnecessary work that could put the client in an inconsistent state if the
    health check fails.


## Version 2.7.0

_2015-12-13_

 *  **Rewritten connection management.** Previously OkHttp's connection pool
    managed both idle and active connections for HTTP/2, but only idle
    connections for HTTP/1.x. With this update the connection pool manages both
    idle and active connections for everything. OkHttp now detects and warns on
    connections that were allocated but never released, and will enforce HTTP/2
    stream limits. This update also fixes `Call.cancel()` to not do I/O on the
    calling thread.
 *  Fix: Don't log gzipped data in the logging interceptor.
 *  Fix: Don't resolve DNS addresses when connecting through a SOCKS proxy.
 *  Fix: Drop the synthetic `OkHttp-Selected-Protocol` response header.
 *  Fix: Support 204 and 205 'No Content' replies in the logging interceptor.
 *  New: Add `Call.isExecuted()`.


## Version 2.6.0

_2015-11-22_

 *  **New Logging Interceptor.** The `logging-interceptor` subproject offers
    simple request and response logging. It may be configured to log headers and
    bodies for debugging. It requires this Maven dependency:

     ```xml
     <dependency>
       <groupId>com.squareup.okhttp</groupId>
       <artifactId>logging-interceptor</artifactId>
       <version>2.6.0</version>
     </dependency>
     ```

    Configure basic logging like this:

    ```java
    HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
    loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BASIC);
    client.networkInterceptors().add(loggingInterceptor);
    ```

    **Warning:** Avoid `Level.HEADERS` and `Level.BODY` in production because
    they could leak passwords and other authentication credentials to insecure
    logs.

 *  **WebSocket API now uses `RequestBody` and `ResponseBody` for messages.**
    This is a backwards-incompatible API change.

 *  **The DNS service is now pluggable.** In some situations this may be useful
    to manually prioritize specific IP addresses.

 *  Fix: Don't throw when converting an `HttpUrl` to a `java.net.URI`.
    Previously URLs with special characters like `|` and `[` would break when
    subjected to URI’s overly-strict validation.
 *  Fix: Don't re-encode `+` as `%20` in encoded URL query strings. OkHttp
    prefers `%20` when doing its own encoding, but will retain `+` when that is
    provided.
 *  Fix: Enforce that callers call `WebSocket.close()` on IO errors. Error
    handling in WebSockets is significantly improved.
 *  Fix: Don't use SPDY/3 style header concatenation for HTTP/2 request headers.
    This could have corrupted requests where multiple headers had the same name,
    as in cookies.
 *  Fix: Reject bad characters in the URL hostname. Previously characters like
    `\0` would cause a late crash when building the request.
 *  Fix: Allow interceptors to change the request method.
 *  Fix: Don’t use the request's `User-Agent` or `Proxy-Authorization` when
    connecting to an HTTPS server via an HTTP tunnel. The `Proxy-Authorization`
    header was being leaked to the origin server.
 *  Fix: Digits may be used in a URL scheme.
 *  Fix: Improve connection timeout recovery.
 *  Fix: Recover from `getsockname` crashes impacting Android releases prior to
    4.2.2.
 *  Fix: Drop partial support for HTTP/1.0. Previously OkHttp would send
    `HTTP/1.0` on connections after seeing a response with `HTTP/1.0`. The fixed
    behavior is consistent with Firefox and Chrome.
 *  Fix: Allow a body in `OPTIONS` requests.
 *  Fix: Don't percent-encode non-ASCII characters in URL fragments.
 *  Fix: Handle null fragments.
 *  Fix: Don’t crash on interceptors that throw `IOException` before a
    connection is attempted.
 *  New: Support [WebDAV][webdav] HTTP methods.
 *  New: Buffer WebSocket frames for better performance.
 *  New: Drop support for `TLS_DHE_DSS_WITH_AES_128_CBC_SHA`, our only remaining
    DSS cipher suite. This is consistent with Firefox and Chrome which have also
    dropped these cipher suite.

## Version 2.5.0

_2015-08-25_

 *  **Timeouts now default to 10 seconds.** Previously we defaulted to never
    timing out, and that was a lousy policy. If establishing a connection,
    reading the next byte from a connection, or writing the next byte to a
    connection takes more than 10 seconds to complete, you’ll need to adjust
    the timeouts manually.

 *  **OkHttp now rejects request headers that contain invalid characters.** This
    includes potential security problems (newline characters) as well as simple
    non-ASCII characters (including international characters and emoji).

 *  **Call canceling is more reliable.**  We had a bug where a socket being
     connected wasn't being closed when the application used `Call.cancel()`.

 *  **Changing a HttpUrl’s scheme now tracks the default port.** We had a bug
    where changing a URL from `http` to `https` would leave it on port 80.

 *  **Okio has been updated to 1.6.0.**
     ```xml
     <dependency>
       <groupId>com.squareup.okio</groupId>
       <artifactId>okio</artifactId>
       <version>1.6.0</version>
     </dependency>
     ```

 *  New: `Cache.initialize()`. Call this on a background thread to eagerly
    initialize the response cache.
 *  New: Fold `MockWebServerRule` into `MockWebServer`. This makes it easier to
    write JUnit tests with `MockWebServer`. The `MockWebServer` library now
    depends on JUnit, though it continues to work with all testing frameworks.
 *  Fix: `FormEncodingBuilder` is now consistent with browsers in which
    characters it escapes. Previously we weren’t percent-encoding commas,
    parens, and other characters.
 *  Fix: Relax `FormEncodingBuilder` to support building empty forms.
 *  Fix: Timeouts throw `SocketTimeoutException`, not `InterruptedIOException`.
 *  Fix: Change `MockWebServer` to use the same logic as OkHttp when determining
    whether an HTTP request permits a body.
 *  Fix: `HttpUrl` now uses the canonical form for IPv6 addresses.
 *  Fix: Use `HttpUrl` internally.
 *  Fix: Recover from Android 4.2.2 EBADF crashes.
 *  Fix: Don't crash with an `IllegalStateException` if an HTTP/2 or SPDY
    write fails, leaving the connection in an inconsistent state.
 *  Fix: Make sure the default user agent is ASCII.


## Version 2.4.0

_2015-05-22_

 *  **Forbid response bodies on HTTP 204 and 205 responses.** Webservers that
    return such malformed responses will now trigger a `ProtocolException` in
    the client.

 *  **WebSocketListener has incompatible changes.** The `onOpen()` method is now
    called on the reader thread, so implementations must return before further
    websocket messages will be delivered. The `onFailure()` method now includes
    an HTTP response if one was returned.

## Version 2.4.0-RC1

_2015-05-16_

 *  **New HttpUrl API.** It's like `java.net.URL` but good. Note that
    `Request.Builder.url()` now throws `IllegalArgumentException` on malformed
    URLs. (Previous releases would throw a `MalformedURLException` when calling
    a malformed URL.)

 *  **We've improved connect failure recovery.** We now differentiate between
    setup, connecting, and connected and implement appropriate recovery rules
    for each. This changes `Address` to no longer use `ConnectionSpec`. (This is
    an incompatible API change).

 *  **`FormEncodingBuilder` now uses `%20` instead of `+` for encoded spaces.**
    Both are permitted-by-spec, but `%20` requires fewer special cases.

 *  **Okio has been updated to 1.4.0.**
     ```xml
     <dependency>
       <groupId>com.squareup.okio</groupId>
       <artifactId>okio</artifactId>
       <version>1.4.0</version>
     </dependency>
     ```

 *  **`Request.Builder` no longer accepts null if a request body is required.**
    Passing null will now fail for request methods that require a body. Instead
    use an empty body such as this one:

    ```java
        RequestBody.create(null, new byte[0]);
    ```

 * **`CertificatePinner` now supports wildcard hostnames.** As always with
   certificate pinning, you must be very careful to avoid [bricking][brick]
   your app. You'll need to pin both the top-level domain and the `*.` domain
   for full coverage.

    ```java
     client.setCertificatePinner(new CertificatePinner.Builder()
         .add("publicobject.com",   "sha1/DmxUShsZuNiqPQsX2Oi9uv2sCnw=")
         .add("*.publicobject.com", "sha1/DmxUShsZuNiqPQsX2Oi9uv2sCnw=")
         .add("publicobject.com",   "sha1/SXxoaOSEzPC6BgGmxAt/EAcsajw=")
         .add("*.publicobject.com", "sha1/SXxoaOSEzPC6BgGmxAt/EAcsajw=")
         .add("publicobject.com",   "sha1/blhOM3W9V/bVQhsWAcLYwPU6n24=")
         .add("*.publicobject.com", "sha1/blhOM3W9V/bVQhsWAcLYwPU6n24=")
         .add("publicobject.com",   "sha1/T5x9IXmcrQ7YuQxXnxoCmeeQ84c=")
         .add("*.publicobject.com", "sha1/T5x9IXmcrQ7YuQxXnxoCmeeQ84c=")
         .build());
    ```

 *  **Interceptors lists are now deep-copied by `OkHttpClient.clone()`.**
    Previously clones shared interceptors, which made it difficult to customize
    the interceptors on a request-by-request basis.

 *  New: `Headers.toMultimap()`.
 *  New: `RequestBody.create(MediaType, ByteString)`.
 *  New: `ConnectionSpec.isCompatible(SSLSocket)`.
 *  New: `Dispatcher.getQueuedCallCount()` and
    `Dispatcher.getRunningCallCount()`. These can be useful in diagnostics.
 *  Fix: OkHttp no longer shares timeouts between pooled connections. This was
    causing some applications to crash when connections were reused.
 *  Fix: `OkApacheClient` now allows an empty `PUT` and `POST`.
 *  Fix: Websockets no longer rebuffer socket streams.
 *  Fix: Websockets are now better at handling close frames.
 *  Fix: Content type matching is now case insensitive.
 *  Fix: `Vary` headers are not lost with `android.net.http.HttpResponseCache`.
 *  Fix: HTTP/2 wasn't enforcing stream timeouts when writing the underlying
    connection. Now it is.
 *  Fix: Never return null on `call.proceed()`. This was a bug in call
    cancelation.
 *  Fix: When a network interceptor mutates a request, that change is now
    reflected in `Response.networkResponse()`.
 *  Fix: Badly-behaving caches now throw a checked exception instead of a
    `NullPointerException`.
 *  Fix: Better handling of uncaught exceptions in MockWebServer with HTTP/2.

## Version 2.3.0

_2015-03-16_

 *  **HTTP/2 support.** We've done interop testing and haven't seen any
    problems. HTTP/2 support has been a big effort and we're particularly
    thankful to Adrian Cole who has helped us to reach this milestone.

 *  **RC4 cipher suites are no longer supported by default.** To connect to
    old, obsolete servers relying on these cipher suites, you must create a
    custom `ConnectionSpec`.

 *  **Beta WebSockets support.**. The `okhttp-ws` subproject offers a new
    websockets client. Please try it out! When it's ready we intend to include
    it with the core OkHttp library.

 *  **Okio updated to 1.3.0.**

    ```xml
    <dependency>
      <groupId>com.squareup.okio</groupId>
      <artifactId>okio</artifactId>
      <version>1.3.0</version>
    </dependency>
    ```

 *  **Fix: improve parallelism of async requests.** OkHttp's Dispatcher had a
    misconfigured `ExecutorService` that limited the number of worker threads.
    If you're using `Call.enqueue()` this update should significantly improve
    request concurrency.

 *  **Fix: Lazily initialize the response cache.** This avoids strict mode
    warnings when initializing OkHttp on Android‘s main thread.

 *  **Fix: Disable ALPN on Android 4.4.** That release of the feature was
    unstable and prone to native crashes in the underlying OpenSSL code.
 *  Fix: Don't send both `If-None-Match` and `If-Modified-Since` cache headers
    when both are applicable.
 *  Fix: Fail early when a port is out of range.
 *  Fix: Offer `Content-Length` headers for multipart request bodies.
 *  Fix: Throw `UnknownServiceException` if a cleartext connection is attempted
    when explicitly forbidden.
 *  Fix: Throw a `SSLPeerUnverifiedException` when host verification fails.
 *  Fix: MockWebServer explicitly closes sockets. (On some Android releases,
    closing the input stream and output stream of a socket is not sufficient.
 *  Fix: Buffer outgoing HTTP/2 frames to limit how many outgoing frames are
    created.
 *  Fix: Avoid crashing when cache writing fails due to a full disk.
 *  Fix: Improve caching of private responses.
 *  Fix: Update cache-by-default response codes.
 *  Fix: Reused `Request.Builder` instances no longer hold stale URL fields.
 *  New: ConnectionSpec can now be configured to use the SSL socket's default
    cipher suites. To use, set the cipher suites to `null`.
 *  New: Support `DELETE` with a request body.
 *  New: `Headers.of(Map)` creates headers from a Map.


## Version 2.2.0

_2014-12-30_

 *  **`RequestBody.contentLength()` now throws `IOException`.**
    This is a source-incompatible change. If you have code that calls
    `RequestBody.contentLength()`, your compile will break with this
    update. The change is binary-compatible, however: code compiled
    for OkHttp 2.0 and 2.1 will continue to work with this update.

 *  **`COMPATIBLE_TLS` no longer supports SSLv3.** In response to the
    [POODLE](https://googleonlinesecurity.blogspot.ca/2014/10/this-poodle-bites-exploiting-ssl-30.html)
    vulnerability, OkHttp no longer offers SSLv3 when negotiation an
    HTTPS connection. If you continue to need to connect to webservers
    running SSLv3, you must manually configure your own `ConnectionSpec`.

 *  **OkHttp now offers interceptors.** Interceptors are a powerful mechanism
    that can monitor, rewrite, and retry calls. The [project
    wiki](https://github.com/square/okhttp/wiki/Interceptors) has a full
    introduction to this new API.

 *  New: APIs to iterate and selectively clear the response cache.
 *  New: Support for SOCKS proxies.
 *  New: Support for `TLS_FALLBACK_SCSV`.
 *  New: Update HTTP/2 support to `h2-16` and `hpack-10`.
 *  New: APIs to prevent retrying non-idempotent requests.
 *  Fix: Drop NPN support. Going forward we support ALPN only.
 *  Fix: The hostname verifier is now strict. This is consistent with the hostname
    verifier in modern browsers.
 *  Fix: Improve `CONNECT` handling for misbehaving HTTP proxies.
 *  Fix: Don't retry requests that failed due to timeouts.
 *  Fix: Cache 302s and 308s that include appropriate response headers.
 *  Fix: Improve pooling of connections that use proxy selectors.
 *  Fix: Don't leak connections when using ALPN on the desktop.
 *  Fix: Update Jetty ALPN to `7.1.2.v20141202` (Java 7) and `8.1.2.v20141202` (Java 8).
    This fixes a bug in resumed TLS sessions where the wrong protocol could be
    selected.
 *  Fix: Don't crash in SPDY and HTTP/2 when disconnecting before connecting.
 *  Fix: Avoid a reverse DNS-lookup for a numeric proxy address
 *  Fix: Resurrect http/2 frame logging.
 *  Fix: Limit to 20 authorization attempts.

## Version 2.1.0

_2014-11-11_

 *  New: Typesafe APIs for interacting with cipher suites and TLS versions.
 *  Fix: Don't crash when mixing authorization challenges with upload retries.


## Version 2.1.0-RC1

_2014-11-04_

 *  **OkHttp now caches private responses**. We've changed from a shared cache
    to a private cache, and will now store responses that use an `Authorization`
    header. This means OkHttp's cache shouldn't be used on middleboxes that sit
    between user agents and the origin server.

 *  **TLS configuration updated.** OkHttp now explicitly enables TLSv1.2,
    TLSv1.1 and TLSv1.0 where they are supported. It will continue to perform
    only one fallback, to SSLv3. Applications can now configure this with the
    `ConnectionSpec` class.

    To disable TLS fallback:

    ```java
    client.setConnectionSpecs(Arrays.asList(
        ConnectionSpec.MODERN_TLS, ConnectionSpec.CLEARTEXT));
    ```

    To disable cleartext connections, permitting `https` URLs only:

    ```java
    client.setConnectionSpecs(Arrays.asList(
        ConnectionSpec.MODERN_TLS, ConnectionSpec.COMPATIBLE_TLS));
    ```

 *  **New cipher suites.** Please confirm that your webservers are reachable
    with this limited set of cipher suites.

    ```
                                             Android
    Name                                     Version

    TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256  5.0
    TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256    5.0
    TLS_DHE_RSA_WITH_AES_128_GCM_SHA256      5.0
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA     4.0
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA     4.0
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA       4.0
    TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA       4.0
    TLS_ECDHE_ECDSA_WITH_RC4_128_SHA         4.0
    TLS_ECDHE_RSA_WITH_RC4_128_SHA           4.0
    TLS_DHE_RSA_WITH_AES_128_CBC_SHA         2.3
    TLS_DHE_DSS_WITH_AES_128_CBC_SHA         2.3
    TLS_DHE_RSA_WITH_AES_256_CBC_SHA         2.3
    TLS_RSA_WITH_AES_128_GCM_SHA256          5.0
    TLS_RSA_WITH_AES_128_CBC_SHA             2.3
    TLS_RSA_WITH_AES_256_CBC_SHA             2.3
    SSL_RSA_WITH_3DES_EDE_CBC_SHA            2.3  (Deprecated in 5.0)
    SSL_RSA_WITH_RC4_128_SHA                 2.3
    SSL_RSA_WITH_RC4_128_MD5                 2.3  (Deprecated in 5.0)
    ```

 *  **Okio updated to 1.0.1.**

    ```xml
    <dependency>
      <groupId>com.squareup.okio</groupId>
      <artifactId>okio</artifactId>
      <version>1.0.1</version>
    </dependency>
    ```

 *  **New APIs to permit easy certificate pinning.** Be warned, certificate
    pinning is dangerous and could prevent your application from trusting your
    server!

 *  **Cache improvements.** This release fixes some severe cache problems
    including a bug where the cache could be corrupted upon certain access
    patterns. We also fixed a bug where the cache was being cleared due to a
    corrupted journal. We've added APIs to configure a request's `Cache-Control`
    headers, and to manually clear the cache.

 *  **Request cancellation fixes.** This update fixes a bug where synchronous
    requests couldn't be canceled by tag. This update avoids crashing when
    `onResponse()` throws an `IOException`. That failure will now be logged
    instead of notifying the thread's uncaught exception handler. We've added a
    new API, `Call.isCanceled()` to check if a call has been canceled.

 *  New: Update `MultipartBuilder` to support content length.
 *  New: Make it possible to mock `OkHttpClient` and `Call`.
 *  New: Update to h2-14 and hpack-9.
 *  New: OkHttp includes a user-agent by default, like `okhttp/2.1.0-RC1`.
 *  Fix: Handle response code `308 Permanent Redirect`.
 *  Fix: Don't skip the callback if a call is canceled.
 *  Fix: Permit hostnames with underscores.
 *  Fix: Permit overriding the content-type in `OkApacheClient`.
 *  Fix: Use the socket factory for direct connections.
 *  Fix: Honor `OkUrlFactory` APIs that disable redirects.
 *  Fix: Don't crash on concurrent modification of `SPDY` SPDY settings.

## Version 2.0.0

This release commits to a stable 2.0 API. Read the 2.0.0-RC1 changes for advice
on upgrading from 1.x to 2.x.

_2014-06-21_

 *  **API Change**: Use `IOException` in `Callback.onFailure()`. This is
    a source-incompatible change, and is different from OkHttp 2.0.0-RC2 which
    used `Throwable`.
 *  Fix: Fixed a caching bug where we weren't storing rewritten request headers
    like `Accept-Encoding`.
 *  Fix: Fixed bugs in handling the SPDY window size. This was stalling certain
    large downloads
 *  Update the language level to Java 7. (OkHttp requires Android 2.3+ or Java 7+.)

## Version 2.0.0-RC2

_2014-06-11_

This update fixes problems in 2.0.0-RC1. Read the 2.0.0-RC1 changes for
advice on upgrading from 1.x to 2.x.

 *  Fix: Don't leak connections! There was a regression in 2.0.0-RC1 where
    connections were neither closed nor pooled.
 *  Fix: Revert builder-style return types from OkHttpClient's timeout methods
    for binary compatibility with OkHttp 1.x.
 *  Fix: Don't skip client stream 1 on SPDY/3.1. This fixes SPDY connectivity to
    `https://google.com`, which doesn't follow the SPDY/3.1 spec!
 *  Fix: Always configure NPN headers. This fixes connectivity to
    `https://facebook.com` when SPDY and HTTP/2 are both disabled. Otherwise an
    unexpected NPN response is received and OkHttp crashes.
 *  Fix: Write continuation frames when HPACK data is larger than 16383 bytes.
 *  Fix: Don't drop uncaught exceptions thrown in async calls.
 *  Fix: Throw an exception eagerly when a request body is not legal. Previously
    we ignored the problem at request-building time, only to crash later with a
    `NullPointerException`.
 *  Fix: Include a backwards-compatible `OkHttp-Response-Source` header with
    `OkUrlFactory `responses.
 *  Fix: Don't include a default User-Agent header in requests made with the Call
    API. Requests made with OkUrlFactory will continue to have a default user
    agent.
 *  New: Guava-like API to create headers:

    ```java
    Headers headers = Headers.of(name1, value1, name2, value2, ...).
    ```

 *  New: Make the content-type header optional for request bodies.
 *  New: `Response.isSuccessful()` is a convenient API to check response codes.
 *  New: The response body can now be read outside of the callback. Response
    bodies must always be closed, otherwise they will leak connections!
 *  New: APIs to create multipart request bodies (`MultipartBuilder`) and form
    encoding bodies (`FormEncodingBuilder`).

## Version 2.0.0-RC1

_2014-05-23_

OkHttp 2 is designed around a new API that is true to HTTP, with classes for
requests, responses, headers, and calls. It uses modern Java patterns like
immutability and chained builders. The API now offers asynchronous callbacks
in addition to synchronous blocking calls.

#### API Changes

 *  **New Request and Response types,** each with their own builder. There's also
    a `RequestBody` class to write the request body to the network and a
    `ResponseBody` to read the response body from the network. The standalone
    `Headers` class offers full access to the HTTP headers.

 *  **Okio dependency added.** OkHttp now depends on
    [Okio](https://github.com/square/okio), an I/O library that makes it easier
    to access, store and process data. Using this library internally makes OkHttp
    faster while consuming less memory. You can write a `RequestBody` as an Okio
    `BufferedSink` and a `ResponseBody` as an Okio `BufferedSource`. Standard
    `InputStream` and `OutputStream` access is also available.

 *  **New Call and Callback types** execute requests and receive their
    responses. Both types of calls can be canceled via the `Call` or the
    `OkHttpClient`.

 *  **URLConnection support has moved to the okhttp-urlconnection module.**
    If you're upgrading from 1.x, this change will impact you. You will need to
    add the `okhttp-urlconnection` module to your project and use the
    `OkUrlFactory` to create new instances of `HttpURLConnection`:

    ```java
    // OkHttp 1.x:
    HttpURLConnection connection = client.open(url);

    // OkHttp 2.x:
    HttpURLConnection connection = new OkUrlFactory(client).open(url);
    ```

 *  **Custom caches are no longer supported.** In OkHttp 1.x it was possible to
    define your own response cache with the `java.net.ResponseCache` and OkHttp's
    `OkResponseCache` interfaces. Both of these APIs have been dropped. In
    OkHttp 2 the built-in disk cache is the only supported response cache.

 *  **HttpResponseCache has been renamed to Cache.** Install it with
    `OkHttpClient.setCache(...)` instead of `OkHttpClient.setResponseCache(...)`.

 *  **OkAuthenticator has been replaced with Authenticator.** This new
    authenticator has access to the full incoming response and can respond with
    whichever followup request is appropriate. The `Challenge` class is now a
    top-level class and `Credential` is replaced with a utility class called
    `Credentials`.

 *  **OkHttpClient.getFollowProtocolRedirects() renamed to
    getFollowSslRedirects()**. We reserve the word _protocol_ for the HTTP
    version being used (HTTP/1.1, HTTP/2). The old name of this method was
    misleading; it was always used to configure redirects between `https://` and
    `http://` schemes.

 *  **RouteDatabase is no longer public API.** OkHttp continues to track which
    routes have failed but this is no exposed in the API.

 *  **ResponseSource is gone.** This enum exposed whether a response came from
    the cache, network, or both. OkHttp 2 offers more detail with raw access to
    the cache and network responses in the new `Response` class.

 *  **TunnelRequest is gone.** It specified how to connect to an HTTP proxy.
    OkHttp 2 uses the new `Request` class for this.

 *  **Dispatcher** is a new class that manages the queue of asynchronous calls. It
    implements limits on total in-flight calls and in-flight calls per host.

#### Implementation changes

 * Support Android `TrafficStats` socket tagging.
 * Drop authentication headers on redirect.
 * Added support for compressed data frames.
 * Process push promise callbacks in order.
 * Update to http/2 draft 12.
 * Update to HPACK draft 07.
 * Add ALPN support. Maven will use ALPN on OpenJDK 8.
 * Update NPN dependency to target `jdk7u60-b13` and `Oracle jdk7u55-b13`.
 * Ensure SPDY variants support zero-length DELETE and POST.
 * Prevent leaking a cache item's InputStreams when metadata read fails.
 * Use a string to identify TLS versions in routes.
 * Add frame logger for HTTP/2.
 * Replacing `httpMinorVersion` with `Protocol`. Expose HTTP/1.0 as a potential protocol.
 * Use `Protocol` to describe framing.
 * Implement write timeouts for HTTP/1.1 streams.
 * Avoid use of SPDY stream ID 1, as that's typically used for UPGRADE.
 * Support OAuth in `Authenticator`.
 * Permit a dangling semicolon in media type parsing.

## Version 1.6.0

_2014-05-23_

 * Offer bridges to make it easier to migrate from OkHttp 1.x to OkHttp 2.0.
   This adds `OkUrlFactory`, `Cache`, and `@Deprecated` annotations for APIs
   dropped in 2.0.

## Version 1.5.4

_2014-04-14_

 * Drop ALPN support in Android. There's a concurrency bug in all
   currently-shipping versions.
 * Support asynchronous disconnects by breaking the socket only. This should
   prevent flakiness from multiple threads concurrently accessing a stream.

## Version 1.5.3

_2014-03-29_

 * Fix bug where the Content-Length header was not always dropped when
   following a redirect from a POST to a GET.
 * Implement basic support for `Thread.interrupt()`. OkHttp now checks
   for an interruption before doing a blocking call. If it is interrupted,
   it throws an `InterruptedIOException`.

## Version 1.5.2

_2014-03-17_

 * Fix bug where deleting a file that was absent from the `HttpResponseCache`
   caused an IOException.
 * Fix bug in HTTP/2 where our HPACK decoder wasn't emitting entries in
   certain eviction scenarios, leading to dropped response headers.

## Version 1.5.1

_2014-03-11_

 * Fix 1.5.0 regression where connections should not have been recycled.
 * Fix 1.5.0 regression where transparent Gzip was broken by attempting to
   recover from another I/O failure.
 * Fix problems where spdy/3.1 headers may not have been compressed properly.
 * Fix problems with spdy/3.1 and http/2 where the wrong window size was being
   used.
 * Fix 1.5.0 regression where conditional cache responses could corrupt the
   connection pool.


## Version 1.5.0

_2014-03-07_


##### OkHttp no longer uses the default SSL context.

Applications that want to use the global SSL context with OkHttp should configure their
OkHttpClient instances with the following:

```java
okHttpClient.setSslSocketFactory(HttpsURLConnection.getDefaultSSLSocketFactory());
```

A simpler solution is to avoid the shared default SSL socket factory. Instead, if you
need to customize SSL, do so for your specific OkHttpClient instance only.

##### Synthetic headers have changed

Previously OkHttp added a synthetic response header, `OkHttp-Selected-Transport`. It
has been replaced with a new synthetic header, `OkHttp-Selected-Protocol`.

##### Changes

 * New: Support for `HTTP-draft-09/2.0`.
 * New: Support for `spdy/3.1`. Dropped support for `spdy/3`.
 * New: Use ALPN on Android platforms that support it (4.4+)
 * New: CacheControl model and parser.
 * New: Protocol selection in MockWebServer.
 * Fix: Route selection shouldn't use TLS modes that we know will fail.
 * Fix: Cache SPDY responses even if the response body is closed prematurely.
 * Fix: Use strict timeouts when aborting a download.
 * Fix: Support Shoutcast HTTP responses like `ICY 200 OK`.
 * Fix: Don't unzip if there isn't a response body.
 * Fix: Don't leak gzip streams on redirects.
 * Fix: Don't do DNS lookups on invalid hosts.
 * Fix: Exhaust the underlying stream when reading gzip streams.
 * Fix: Support the `PATCH` method.
 * Fix: Support request bodies on `DELETE` method.
 * Fix: Drop the `okhttp-protocols` module.
 * Internal: Replaced internal byte array buffers with pooled buffers ("OkBuffer").


## Version 1.3.0

_2014-01-11_

 * New: Support for "PATCH" HTTP method in client and MockWebServer.
 * Fix: Drop `Content-Length` header when redirected from POST to GET.
 * Fix: Correctly read cached header entries with malformed header names.
 * Fix: Do not directly support any authentication schemes other than "Basic".
 * Fix: Respect read timeouts on recycled connections.
 * Fix: Transmit multiple cookie values as a single header with delimiter.
 * Fix: Ensure `null` is never returned from a connection's `getHeaderFields()`.
 * Fix: Persist proper `Content-Encoding` header to cache for GZip responses.
 * Fix: Eliminate rare race condition in SPDY streams that would prevent connection reuse.
 * Fix: Change HTTP date formats to UTC to conform to RFC2616 section 3.3.
 * Fix: Support SPDY header blocks with trailing bytes.
 * Fix: Allow `;` as separator for `Cache-Control` header.
 * Fix: Correct bug where HTTPS POST requests were always automatically buffered.
 * Fix: Honor read timeout when parsing SPDY headers.


## Version 1.2.1

_2013-08-23_

 * Resolve issue with 'jar-with-dependencies' artifact creation.
 * Fix: Support empty SPDY header values.


## Version 1.2.0

_2013-08-11_

 *  New APIs on OkHttpClient to set default timeouts for connect and read.
 *  Fix bug when caching SPDY responses.
 *  Fix a bug with SPDY plus half-closed streams. (thanks kwuollett)
 *  Fix a bug in `Content-Length` reporting for gzipped streams in the Apache
    HTTP client adapter. (thanks kwuollett)
 *  Work around the Alcatel `getByInetAddress` bug (thanks k.kocel)
 *  Be more aggressive about testing pooled sockets before reuse. (thanks
    warpspin)
 *  Include `Content-Type` and `Content-Encoding` in the Apache HTTP client
    adapter. (thanks kwuollett)
 *  Add a media type class to OkHttp.
 *  Change custom header prefix:

    ```
    X-Android-Sent-Millis is now OkHttp-Sent-Millis
    X-Android-Received-Millis is now OkHttp-Received-Millis
    X-Android-Response-Source is now OkHttp-Response-Source
    X-Android-Selected-Transport is now OkHttp-Selected-Transport
    ```
 *  Improve cache invalidation for POST-like requests.
 *  Bring MockWebServer into OkHttp and teach it SPDY.


## Version 1.1.1

_2013-06-23_

 * Fix: ClassCastException when caching responses that were redirected from
   HTTP to HTTPS.


## Version 1.1.0

_2013-06-15_

 * Fix: Connection reuse was broken for most HTTPS connections due to a bug in
   the way the hostname verifier was selected.
 * Fix: Locking bug in SpdyConnection.
 * Fix: Ignore null header values (for compatibility with HttpURLConnection).
 * Add URLStreamHandlerFactory support so that `URL.openConnection()` uses
   OkHttp.
 * Expose the transport ("http/1.1", "spdy/3", etc.) via magic request headers.
   Use `X-Android-Transports` to write the preferred transports and
   `X-Android-Selected-Transport` to read the negotiated transport.


## Version 1.0.2

_2013-05-11_

 * Fix: Remove use of Java 6-only APIs.
 * Fix: Properly handle exceptions from `NetworkInterface` when querying MTU.
 * Fix: Ensure MTU has a reasonable default and upper-bound.


## Version 1.0.1

_2013-05-06_

 * Correct casing of SSL in method names (`getSslSocketFactory`/`setSslSocketFactory`).


## Version 1.0.0

_2013-05-06_

Initial release.

 [brick]: https://noncombatant.org/2015/05/01/about-http-public-key-pinning/
 [webdav]: https://tools.ietf.org/html/rfc4918
 [major_versions]: http://jakewharton.com/java-interoperability-policy-for-major-version-updates/
 [nginx_959]: https://trac.nginx.org/nginx/ticket/959
 [okhttp_idling_resource]: https://github.com/JakeWharton/okhttp-idling-resource
 [bom]: https://en.wikipedia.org/wiki/Byte_order_mark
 [junit_5_rules]: https://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4-rulesupport
 [public_suffix]: https://publicsuffix.org/
 [maven_provided]: https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html
 [remove_cbc_ecdsa]: https://developers.google.com/web/updates/2016/12/chrome-56-deprecations#remove_cbc-mode_ecdsa_ciphers_in_tls
 [conscrypt]: https://github.com/google/conscrypt/
 [conscrypt_dependency]: https://github.com/google/conscrypt/#download
 [https_server_sample]: https://github.com/square/okhttp/blob/master/samples/guide/src/main/java/okhttp3/recipes/HttpsServer.java