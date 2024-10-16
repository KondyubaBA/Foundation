<details>
  <summary>HttpContext</summary>

```cs
[DebuggerDisplay("{DebuggerToString(),nq}")]
[DebuggerTypeProxy(typeof(HttpContextDebugView))]
public abstract class HttpContext
{
    public abstract IFeatureCollection Features { get; }

    public abstract HttpRequest Request { get; }

    public abstract HttpResponse Response { get; }

    public abstract ConnectionInfo Connection { get; }

    public abstract WebSocketManager WebSockets { get; }

    public abstract ClaimsPrincipal User { get; set; }

    public abstract IDictionary<object, object?> Items { get; set; }

    public abstract IServiceProvider RequestServices { get; set; }

    public abstract CancellationToken RequestAborted { get; set; }

    public abstract string TraceIdentifier { get; set; }

    public abstract ISession Session { get; set; }

    public abstract void Abort();

    private string DebuggerToString()
    {
        return HttpContextDebugFormatter.ContextToString(this, reasonPhrase: null);
    }

    private sealed class HttpContextDebugView(HttpContext context)
    {
        private readonly HttpContext _context = context;

        // Hide server specific implementations, they combine IFeatureCollection and many feature interfaces.
        public HttpContextFeatureDebugView Features => new HttpContextFeatureDebugView(_context.Features);
        public HttpRequest Request => _context.Request;
        public HttpResponse Response => _context.Response;
        public Endpoint? Endpoint => _context.GetEndpoint();
        public ConnectionInfo Connection => _context.Connection;
        public WebSocketManager WebSockets => _context.WebSockets;
        public ClaimsPrincipal User => _context.User;
        public IDictionary<object, object?> Items => _context.Items;
        public CancellationToken RequestAborted => _context.RequestAborted;
        public IServiceProvider RequestServices => _context.RequestServices;
        public string TraceIdentifier => _context.TraceIdentifier;
        // The normal session property throws if accessed before/without the session middleware.
        public ISession? Session => _context.Features.Get<ISessionFeature>()?.Session;
    }

    [DebuggerDisplay("Count = {Items.Length}")]
    private sealed class HttpContextFeatureDebugView(IFeatureCollection features)
    {
        private readonly IFeatureCollection _features = features;

        [DebuggerBrowsable(DebuggerBrowsableState.RootHidden)]
        public KeyValuePair<string, object>[] Items => _features.Select(pair => new KeyValuePair<string, object>(pair.Key.FullName ?? string.Empty, pair.Value)).ToArray();
    }
}
```  
</details>
