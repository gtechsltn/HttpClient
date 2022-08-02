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
+ https://bytedev.medium.com/net-core-httpclient-best-practices-4c1b20e32c6 (Best Practices)
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

