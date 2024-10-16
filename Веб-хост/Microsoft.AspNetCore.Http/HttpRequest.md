<details>
  <summary>HttpRequest</summary>

```cs
[DebuggerDisplay("{DebuggerToString(),nq}")]
[DebuggerTypeProxy(typeof(HttpRequestDebugView))]
public abstract class HttpRequest
{
    public abstract HttpContext HttpContext { get; }

    public abstract string Method { get; set; }

    public abstract string Scheme { get; set; }

    public abstract bool IsHttps { get; set; }

    public abstract HostString Host { get; set; }

    public abstract PathString PathBase { get; set; }

    public abstract PathString Path { get; set; }

    public abstract QueryString QueryString { get; set; }

    public abstract IQueryCollection Query { get; set; }

    public abstract string Protocol { get; set; }

    public abstract IHeaderDictionary Headers { get; }

    public abstract IRequestCookieCollection Cookies { get; set; }

    public abstract long? ContentLength { get; set; }

    public abstract string? ContentType { get; set; }

    public abstract Stream Body { get; set; }

    public virtual PipeReader BodyReader { get => throw new NotImplementedException(); }

    public abstract bool HasFormContentType { get; }

    public abstract IFormCollection Form { get; set; }

    public abstract Task<IFormCollection> ReadFormAsync(CancellationToken cancellationToken = new CancellationToken());

    public virtual RouteValueDictionary RouteValues { get; set; } = null!;

    private string DebuggerToString()
    {
        return HttpContextDebugFormatter.RequestToString(this);
    }

    private sealed class HttpRequestDebugView(HttpRequest request)
    {
        private readonly HttpRequest _request = request;

        public string Method => _request.Method;
        public string Scheme => _request.Scheme;
        public bool IsHttps => _request.IsHttps;
        public HostString Host => _request.Host;
        public PathString PathBase => _request.PathBase;
        public PathString Path => _request.Path;
        public QueryString QueryString => _request.QueryString;
        public IQueryCollection Query => _request.Query;
        public string Protocol => _request.Protocol;
        public IHeaderDictionary Headers => _request.Headers;
        public IRequestCookieCollection Cookies => _request.Cookies;
        public long? ContentLength => _request.ContentLength;
        public string? ContentType => _request.ContentType;
        public bool HasFormContentType => _request.HasFormContentType;
        public IFormCollection? Form => _request.HttpContext.Features.Get<IFormFeature>()?.Form;
        public RouteValueDictionary RouteValues => _request.RouteValues;
    }
}
```  
</details>
