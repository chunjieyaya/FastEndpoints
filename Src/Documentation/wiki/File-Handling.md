# handling file uploads

the following example relays back the image data uploaded to the endpoint in order to demonstrate both receiving and sending of file data:

```csharp
public class MyEndpoint : Endpoint<MyRequest>
{
    public override void Configure()
    {
        Verbs(Http.POST);
        Routes("/api/uploads/image");
        AllowFileUploads();
    }

    public override async Task HandleAsync(MyRequest req, CancellationToken ct)
    {
        if (Files.Count > 0)
        {
            var file = Files[0];

            await SendStreamAsync(
                stream: file.OpenReadStream(),
                fileName: "test.png",
                fileLengthBytes: file.Length,
                contentType: "image/png");

            return;
        }
        await SendNoContentAsync();
    }
}
```

endpoints by default won't allow `multipart/form-data` content uploads. you'd have to enable file uploads by using the `AllowFileUploads()` method in the handler configuration like shown above. the received files are exposed to the endpoint handler via the `Files` property which is of `IFormFileCollection` type.

## binding files to dto
file data can also be automatically bound to the request dto by simply adding an `IFormFile` property with a matching name.
```csharp
public class MyRequest
{
    public int Width { get; set; }
    public int Height { get; set; }
    public IFormFile File1 { get; set; }
    public IFormFile File2 { get; set; }
    public IFormFile File3 { get; set; }
}
```

# sending file responses

there are 3 methods you can use to send file data down to the client.

**SendStreamAsync()** - you can supply a `System.IO.Stream` to this method for reading binary data from.

**SendFileAsync()** - you can supply a `System.IO.FileInfo` instance as the source of the binary data.

**SendBytesAsync()** - you can supply a byte array as the source of data to be sent to the client.

all three methods allow you to optionally specify the `content-type` and `file name`. if file name is specified, the `Content-Disposition: attachment` response header will be set with the given file name so that a file download will be initiated by the client/browser. range requests/ partial responses are also supported by setting the `enableRangeProcessing` parameter to `true`.

# write to response stream

instead of using the above methods, you also have the choice of writing directly to the http response stream.
> **[see here](https://github.com/dj-nitehawk/FastEndpoints-FileHandling-Demo)** for an example project that stores and retrieves images in mongodb.