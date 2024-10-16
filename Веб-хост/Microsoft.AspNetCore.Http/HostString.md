<details>
  <summary>HostString</summary>

```cs
[DebuggerDisplay("{Value}")]
public readonly struct HostString : IEquatable<HostString>
{
    private static readonly SearchValues<char> s_safeHostStringChars =
        SearchValues.Create("%-.0123456789:ABCDEFGHIJKLMNOPQRSTUVWXYZ[]abcdefghijklmnopqrstuvwxyz");

    private static readonly IdnMapping s_idnMapping = new();

    private readonly string _value;

    public HostString(string value)
    {
        _value = value;
    }

    public HostString(string host, int port)
    {
        ArgumentNullException.ThrowIfNull(host);

        if (port <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(port), Resources.Exception_PortMustBeGreaterThanZero);
        }

        int index;
        if (!host.Contains('[')
            && (index = host.IndexOf(':')) >= 0
            && index < host.Length - 1
            && host.IndexOf(':', index + 1) >= 0)
        {
            // IPv6 without brackets ::1 is the only type of host with 2 or more colons
            host = $"[{host}]";
        }

        _value = host + ":" + port.ToString(CultureInfo.InvariantCulture);
    }

    public string Value
    {
        get { return _value; }
    }

    public bool HasValue
    {
        get { return !string.IsNullOrEmpty(_value); }
    }

    public string Host
    {
        get
        {
            GetParts(_value, out var host, out _);

            return host.ToString();
        }
    }

    public int? Port
    {
        get
        {
            GetParts(_value, out _, out var port);

            if (!StringSegment.IsNullOrEmpty(port)
                && int.TryParse(port.AsSpan(), NumberStyles.None, CultureInfo.InvariantCulture, out var p))
            {
                return p;
            }

            return null;
        }
    }

    public override string ToString()
    {
        return ToUriComponent();
    }

    public string ToUriComponent()
    {
        if (!HasValue)
        {
            return string.Empty;
        }

        if (!_value.AsSpan().ContainsAnyExcept(s_safeHostStringChars))
        {
            return _value;
        }

        GetParts(_value, out var host, out var port);

        var encoded = s_idnMapping.GetAscii(host.Buffer!, host.Offset, host.Length);

        return StringSegment.IsNullOrEmpty(port)
            ? encoded
            : string.Concat(encoded, ":", port.AsSpan());
    }

    public static HostString FromUriComponent(string uriComponent)
    {
        if (!string.IsNullOrEmpty(uriComponent))
        {
            int index;
            if (uriComponent.Contains('['))
            {
                // IPv6 in brackets [::1], maybe with port
            }
            else if ((index = uriComponent.IndexOf(':')) >= 0
                && index < uriComponent.Length - 1
                && uriComponent.IndexOf(':', index + 1) >= 0)
            {
                // IPv6 without brackets ::1 is the only type of host with 2 or more colons
            }
            else if (uriComponent.Contains("xn--", StringComparison.Ordinal))
            {
                // Contains punycode
                if (index >= 0)
                {
                    // Has a port
                    var port = uriComponent.AsSpan(index);
                    uriComponent = string.Concat(s_idnMapping.GetUnicode(uriComponent, 0, index), port);
                }
                else
                {
                    uriComponent = s_idnMapping.GetUnicode(uriComponent);
                }
            }
        }
        return new HostString(uriComponent);
    }

    public static HostString FromUriComponent(Uri uri)
    {
        ArgumentNullException.ThrowIfNull(uri);

        return new HostString(uri.GetComponents(
            UriComponents.NormalizedHost | // Always convert punycode to Unicode.
            UriComponents.HostAndPort, UriFormat.Unescaped));
    }

    public static bool MatchesAny(StringSegment value, IList<StringSegment> patterns)
    {
        if (value == null)
        {
            throw new ArgumentNullException(nameof(value));
        }
        ArgumentNullException.ThrowIfNull(patterns);

        // Drop the port
        GetParts(value, out var host, out var port);

        for (int i = 0; i < port.Length; i++)
        {
            if (port[i] < '0' || '9' < port[i])
            {
                throw new FormatException($"The given host value '{value}' has a malformed port.");
            }
        }

        var count = patterns.Count;
        for (int i = 0; i < count; i++)
        {
            var pattern = patterns[i];

            if (pattern == "*")
            {
                return true;
            }

            if (StringSegment.Equals(pattern, host, StringComparison.OrdinalIgnoreCase))
            {
                return true;
            }

            // Sub-domain wildcards: *.example.com
            if (pattern.StartsWith("*.", StringComparison.Ordinal) && host.Length >= pattern.Length)
            {
                // .example.com
                var allowedRoot = pattern.Subsegment(1);

                var hostRoot = host.Subsegment(host.Length - allowedRoot.Length);
                if (hostRoot.Equals(allowedRoot, StringComparison.OrdinalIgnoreCase))
                {
                    return true;
                }
            }
        }

        return false;
    }

    public bool Equals(HostString other)
    {
        if (!HasValue && !other.HasValue)
        {
            return true;
        }
        return string.Equals(_value, other._value, StringComparison.OrdinalIgnoreCase);
    }

    public override bool Equals(object? obj)
    {
        if (ReferenceEquals(null, obj))
        {
            return !HasValue;
        }
        return obj is HostString && Equals((HostString)obj);
    }

    public override int GetHashCode()
    {
        return (HasValue ? StringComparer.OrdinalIgnoreCase.GetHashCode(_value) : 0);
    }

    public static bool operator ==(HostString left, HostString right)
    {
        return left.Equals(right);
    }

    public static bool operator !=(HostString left, HostString right)
    {
        return !left.Equals(right);
    }

    private static void GetParts(StringSegment value, out StringSegment host, out StringSegment port)
    {
        int index;
        port = null;
        host = null;

        if (StringSegment.IsNullOrEmpty(value))
        {
            return;
        }
        else if ((index = value.IndexOf(']')) >= 0)
        {
            // IPv6 in brackets [::1], maybe with port
            host = value.Subsegment(0, index + 1);
            // Is there a colon and at least one character?
            if (index + 2 < value.Length && value[index + 1] == ':')
            {
                port = value.Subsegment(index + 2);
            }
        }
        else if ((index = value.IndexOf(':')) >= 0
            && index < value.Length - 1
            && value.IndexOf(':', index + 1) >= 0)
        {
            // IPv6 without brackets ::1 is the only type of host with 2 or more colons
            host = $"[{value}]";
            port = null;
        }
        else if (index >= 0)
        {
            // Has a port
            host = value.Subsegment(0, index);
            port = value.Subsegment(index + 1);
        }
        else
        {
            host = value;
            port = null;
        }
    }
}
```
</details>
