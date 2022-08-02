# HttpClient
+ https://www.stevejgordon.co.uk/demystifying-httpclient-internals-sendasync-flow-for-httprequestmessage
+ Part 1 – An introduction to HttpClientFactory
+ Part 2 – Defining Named and Typed Clients
+ Part 3 – Outgoing request middleware with handlers
+ Part 4 – Integrating with Polly for transient fault handling
+ Part 5 – Logging
+ https://www.stevejgordon.co.uk/httpclientfactory-asp-net-core-logging

+ ## C# HttpClient
+ https://zetcode.com/csharp/httpclient/

## Sử dụng HttpClient
+ https://xuanthulab.net/networking-su-dung-httpclient-trong-c-tao-cac-truy-van-http.html
+ https://xuanthulab.net/networking-su-dung-httpmessagehandler-cho-httpclient-trong-c-csharp.html

## HttpClient GetAsync, PostAsync, SendAsync in C# - Carl de Souza
+ https://carldesouza.com/httpclient-getasync-postasync-sendasync-c/

<pre>

HttpRequestMessage request = new HttpRequestMessage()...
HttpClient httpClient = new HttpClient() ...
httpClient.SendAsync(request)...

</pre>

## Mocking HttpClient SendAsync
+ https://carlpaton.github.io/2021/01/mocking-httpclient-sendasync/
+ https://dev.to/gautemeekolsen/how-to-test-httpclient-with-moq-in-c-2ldp
+ https://www.meziantou.net/mocking-an-httpclient-using-an-httpclienthandler.htm

## Fun with HttpClient
+ https://thomaslevesque.com/2016/12/08/fun-with-the-httpclient-pipeline/

## Best Practices
+ https://bytedev.medium.com/net-core-httpclient-best-practices-4c1b20e32c6
+ https://makolyte.com/csharp-how-to-send-a-file-with-httpclient/ (Send File) (Upload File)
+ https://makolyte.com/csharp-disposing-the-request-httpcontent-when-using-httpclient/ (Big File) (Large File) (StreamContent)
+ https://makolyte.com/csharp-how-to-add-request-headers-when-using-httpclient/
+ https://makolyte.com/csharp-how-to-read-response-headers-with-httpclient/
+ https://makolyte.com/csharp-how-to-get-the-status-code-when-using-httpclient/
+ https://makolyte.com/csharp-handling-redirects-with-httpclient/
+ https://makolyte.com/csharp-configuring-how-long-an-httpclient-connection-will-stay-open/
+ https://makolyte.com/csharp-how-to-read-problem-details-json-with-httpclient/ (ProblemDetails)
+ https://makolyte.com/csharp-sending-query-strings-with-httpclient/
+ https://makolyte.com/csharp-the-performance-gains-of-httpclient-reusing-connections/
+ https://makolyte.com/csharp-switch-from-using-httpwebrequest-to-httpclient/ (HttpWebRequest -> HttpClient)
+ https://makolyte.com/csharp-how-to-cancel-an-httpclient-request/
+ https://makolyte.com/csharp-how-to-make-concurrent-requests-with-httpclient/ (Concurrent Requests)
