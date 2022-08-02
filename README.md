# HttpClient

## In C# we can consume RestAPI using the following ways:
+ HttpWebRequest or HttpWebResponse
+ WebClient
+ HttpClient
+ RestSharp
+ Refit (Xamarin)
+ Flurl.Http
+ FluentRest

## Overview HttpClient
+ https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests
+ https://www.infoq.com/articles/creating-http-sdks-dotnet-6/
+ https://www.stevejgordon.co.uk/demystifying-httpclient-internals-sendasync-flow-for-httprequestmessage
+ Part 1 – An introduction to HttpClientFactory
+ Part 2 – Defining Named and Typed Clients
+ Part 3 – Outgoing request middleware with handlers
+ Part 4 – Integrating with Polly for transient fault handling
+ Part 5 – Logging
+ https://www.stevejgordon.co.uk/httpclientfactory-asp-net-core-logging
+ https://www.thecodebuzz.com/using-httpclient-best-practices-and-anti-patterns/

## C# HttpClient
+ https://zetcode.com/csharp/httpclient/
+ https://www.aspnetmonsters.com/2016/08/2016-08-27-httpclientwrong/index.html
+ https://www.albahari.com/nutshell/cs5ch16.aspx

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
+ http://justcodesnippets.durlut.ro/index.php/2018/11/01/httpclient-with-error-logging-handler-and-retry-handler/

Usage:
<pre>
            var cookieContainer = new System.Net.CookieContainer();

            using (var httpClientHandler = new HttpClientHandler() { UseCookies = true, CookieContainer = cookieContainer })
            using (var errorLoggingHandler = new Handlers.LoggingHandler<AirWatchLogDto>(_httpGetLogger, httpClientHandler))
            //using (var retryHandler = new RetryHandler(errorLoggingHandler) { RetryCounterCount = retryCounterCount })
            using (var httpClient = new HttpClients.AirWatchCustomHeadersHttpClient(httpClientHandler, _appSettingsService))
            {
                HttpRequestMessage httpRequestMessage = new HttpRequestMessage(httpMethod, requestUri);

                if (httpMethod == HttpMethod.Post && postObject != null)
                {
                    string formattedJson = Newtonsoft.Json.JsonConvert.SerializeObject(postObject);
                    httpRequestMessage.Content = new StringContent(formattedJson, Encoding.UTF8, "application/json");
                }

                var httpReponseMessage = await httpClient.SendAsync(httpRequestMessage);
                string content = await httpReponseMessage.Content.ReadAsStringAsync();
                return httpReponseMessage;
            }
</pre>

Retry Handler:
<pre>
    public class RetryHandler : DelegatingHandler
    {
        private int _retryCounterCount = 3;
        public int RetryCounterCount { get { return _retryCounterCount; } set { _retryCounterCount = value; } }

        public RetryHandler(HttpMessageHandler innerHandler) : base(innerHandler)
        {
        }

        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            int counter = 0;
            while (true)
            {
                try
                {
                    counter++;                    
                    // base.SendAsync calls the inner handler
                    var response = await base.SendAsync(request, cancellationToken);
                    
                    if (counter >= RetryCounterCount)
                    {
                        return response;
                    }
                    if (response.StatusCode == HttpStatusCode.ServiceUnavailable)
                    {
                        // 503 Service Unavailable
                        // Wait a bit and try again later
                        await Task.Delay(5000, cancellationToken);
                        continue;
                    }

                    if (response.StatusCode == (HttpStatusCode)429)
                    {
                        // 429 Too many requests
                        // Wait a bit and try again later
                        await Task.Delay(1000, cancellationToken);
                        continue;
                    }

                    // Not something we can retry, return the response as is
                    return response;
                }
                catch (Exception ex) when (IsNetworkError(ex))
                {
                    if (counter >= RetryCounterCount)
                    {
                        throw;
                    }
                    // Network error
                    // Wait a bit and try again later
                    await Task.Delay(2000, cancellationToken);
                    continue;
                }
            }
        }

        private static bool IsNetworkError(Exception ex)
        {
            // Check if it's a network error
            if (ex is SocketException)
                return true;
            if (ex.InnerException != null)
                return IsNetworkError(ex.InnerException);
            return false;
        }
    }
</pre>

Logging Handler:
<pre>
    //http://www.thomaslevesque.com/2016/12/08/fun-with-the-httpclient-pipeline/
    public class LoggingHandler<T> : DelegatingHandler
    {
        private readonly IHttpClientEventLogger<T> _logger;

        public LoggingHandler(IHttpClientEventLogger<T> logger, HttpMessageHandler innerHandler) : base(innerHandler)
        {
            _logger = logger;
        }

        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            try
            {
                _logger.MapRequest(request);
                
                var response = await base.SendAsync(request, cancellationToken);
                _logger.MapResponse(response);
                return response;
            }
            catch (Exception ex)
            {
                _logger.MapResponseException(ex);
                throw;
            }
            finally
            {
                _logger.LogEvent();
            }
        }
    }

    public class ErrorLoggingHandler<T> : DelegatingHandler
    {
        private readonly IHttpClientEventLogger<T> _logger;

        public ErrorLoggingHandler(IHttpClientEventLogger<T> logger, HttpMessageHandler innerHandler) : base(innerHandler)
        {
            _logger = logger;
        }

        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            bool hasError = false;
            try
            {
                _logger.MapRequest(request);

                var response = await base.SendAsync(request, cancellationToken);
                
                return response;
            }
            catch (Exception ex)
            {
                hasError = true;
                _logger.MapResponseException(ex);
                throw;
            }
            finally
            {
                if(hasError)
                    _logger.LogEvent();
            }
        }
    }
</pre>

## Best Practices
+ https://bytedev.medium.com/net-core-httpclient-best-practices-4c1b20e32c6 (Best Practices)
+ https://vladiliescu.net/wiki/netcore/ (Best Practices) (Properly)
+ https://www.thinktecture.com/en/asp-net-core/aspnet-core-webapi-performance/ (Performance)
+ https://code-maze.com/using-streams-with-httpclient-to-improve-performance-and-memory-usage/ (Streams) (Performance) (Best Practices)
<pre>
private async Task CreateCompanyWithStream()
{
    var companyForCreation = new CompanyForCreationDto
    {
        Name = "Eagle IT Ltd.",
        Country = "USA",
        Address = "Eagle IT Street 289"
    };

    var ms = new MemoryStream();
    await JsonSerializer.SerializeAsync(ms, companyForCreation);
    ms.Seek(0, SeekOrigin.Begin);

    var request = new HttpRequestMessage(HttpMethod.Post, "companies");
    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    using (var requestContent = new StreamContent(ms))
    {
        request.Content = requestContent;
        requestContent.Headers.ContentType = new MediaTypeHeaderValue("application/json");

        using (var response = await _httpClient.SendAsync(request, HttpCompletionOption.ResponseHeadersRead))
        {
            response.EnsureSuccessStatusCode();

            var content = await response.Content.ReadAsStreamAsync();
            var createdCompany = await JsonSerializer.DeserializeAsync<CompanyDto>(content, _options);
        }    
    }    
}
</pre>
+ https://josef.codes/efficient-file-uploads-with-dotnet/ (Efficient) (Upload File) (Streams)
  + Read file content with a stream
  + Using a stream allows us to operate on small portions of the file in chunks instead of allocating the whole file.
  + When using a stream, the flow works (kind of) like this:
    + Get a small chunk from the file (buffer).
    + Send this chunk to the receiver (in our case our FileUpload API).
    + Repeat 1-2 until the whole file has been sent.
+ https://johnthiriet.com/efficient-post-calls/ (StreamContent)
+ https://makolyte.com/csharp-disposing-the-request-httpcontent-when-using-httpclient/ (Big File) (Large File) (StreamContent)
+ https://makolyte.com/csharp-how-to-send-a-file-with-httpclient/ (Send File) (Upload File)
+ https://makolyte.com/tag/httpclient/
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
+ https://makolyte.com/csharp-get-and-send-json-with-httpclient/
+ https://makolyte.com/csharp-how-to-unit-test-code-that-uses-httpclient/
+ https://makolyte.com/csharp-newtonsoft-extension-methods-for-httpclient/
+ https://makolyte.com/event-driven-dotnet-how-to-consume-an-sse-endpoint-with-httpclient/
+ https://makolyte.com/csharp-how-to-use-polly-to-do-retries/
+ https://makolyte.com/csharp-circuit-breaker-with-polly/
+ https://makolyte.com/csharp-how-to-change-the-httpclient-timeout-per-request/
+ https://makolyte.com/csharp-how-to-get-the-status-code-when-using-httpclient/
+ https://makolyte.com/how-to-use-toxiproxy-to-verify-your-code-can-handle-timeouts-and-unavailable-endpoints/ (Timeouts)
+ https://makolyte.com/csharp-handling-redirects-with-httpclient/
+ https://makolyte.com/tag/issuccessstatuscode/
+ https://makolyte.com/tag/ensuresuccessstatuscode/
+ https://stackoverflow.com/questions/45739753/correct-use-of-ensuresuccessstatuscode-and-issuccessstatuscode

## ASP.NET Core Web API Performance – Throughput For Upload And Download
+ https://www.thinktecture.com/en/asp-net-core/aspnet-core-webapi-performance/

## HttpClient -> SendAsync -> StreamContent -> Custom Header (Bearer Authentication)
+ https://stackoverflow.com/questions/48344819/send-large-file-via-httpclient
+ https://stackoverflow.com/questions/41378457/c-httpclient-file-upload-progress-when-uploading-multiple-file-as-multipartfo
+ https://stackoverflow.com/questions/35907642/custom-header-to-httpclient-request (HAY HAY HAY)
+ https://stackoverflow.com/questions/12022965/adding-http-headers-to-httpclient
+ https://makolyte.com/csharp-how-to-add-request-headers-when-using-httpclient/
+ https://code-maze.com/using-streams-with-httpclient-to-improve-performance-and-memory-usage/
+ https://xuanthulab.net/networking-su-dung-httpclient-trong-c-tao-cac-truy-van-http.html
+ https://www.stevejgordon.co.uk/using-httpcompletionoption-responseheadersread-to-improve-httpclient-performance-dotnet
+ https://www.tugberkugurlu.com/archive/efficiently-streaming-large-http-responses-with-httpclient
+ https://brokul.dev/sending-files-and-additional-data-using-httpclient-in-net-core
