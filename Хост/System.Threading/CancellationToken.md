```cs
[DebuggerDisplay("IsCancellationRequested = {IsCancellationRequested}")]
public readonly struct CancellationToken : IEquatable<CancellationToken>
{
    public static CancellationToken None => default;

    public bool IsCancellationRequested => _source != null && _source.IsCancellationRequested;

    public bool CanBeCanceled => _source != null;

    public WaitHandle WaitHandle => (_source ?? CancellationTokenSource.s_neverCanceledSource).WaitHandle;

    internal CancellationToken(CancellationTokenSource? source) => _source = source;

    public CancellationToken(bool canceled) : this(canceled ? CancellationTokenSource.s_canceledSource : null)
    {
    }

    public CancellationTokenRegistration Register(Action callback) => Register(callback, useSynchronizationContext: false);

    public CancellationTokenRegistration Register(Action callback, bool useSynchronizationContext)
    {
        ArgumentNullException.ThrowIfNull(callback);
        return Register(
            (Action<object?>)(static obj => ((Action)obj!)()),
            callback,
            useSynchronizationContext,
            useExecutionContext: true);
    }

    public CancellationTokenRegistration Register(Action<object?> callback, object? state) =>
        Register(callback, state, useSynchronizationContext: false, useExecutionContext: true);

    public CancellationTokenRegistration Register(Action<object?, CancellationToken> callback, object? state) =>
        Register(callback, state, useSynchronizationContext: false, useExecutionContext: true);

    public CancellationTokenRegistration Register(Action<object?> callback, object? state, bool useSynchronizationContext) =>
        Register(callback, state, useSynchronizationContext, useExecutionContext: true);

    public CancellationTokenRegistration UnsafeRegister(Action<object?> callback, object? state) =>
        Register(callback, state, useSynchronizationContext: false, useExecutionContext: false);

    public CancellationTokenRegistration UnsafeRegister(Action<object?, CancellationToken> callback, object? state) =>
        Register(callback, state, useSynchronizationContext: false, useExecutionContext: false);

    private CancellationTokenRegistration Register(Delegate callback, object? state, bool useSynchronizationContext, bool useExecutionContext)
    {
        ArgumentNullException.ThrowIfNull(callback);

        CancellationTokenSource? source = _source;
        return source != null ?
            source.Register(callback, state, useSynchronizationContext ? SynchronizationContext.Current : null, useExecutionContext ? ExecutionContext.Capture() : null) :
            default; // Nothing to do for tokens than can never reach the canceled state. Give back a dummy registration.
    }

    public bool Equals(CancellationToken other) => _source == other._source;

    public override bool Equals([NotNullWhen(true)] object? other) => other is CancellationToken && Equals((CancellationToken)other);

    public override int GetHashCode() => (_source ?? CancellationTokenSource.s_neverCanceledSource).GetHashCode();

    public static bool operator ==(CancellationToken left, CancellationToken right) => left.Equals(right);

    public static bool operator !=(CancellationToken left, CancellationToken right) => !left.Equals(right);

    public void ThrowIfCancellationRequested()
    {
        if (IsCancellationRequested)
            ThrowOperationCanceledException();
    }

    [DoesNotReturn]
    private void ThrowOperationCanceledException() =>
        throw new OperationCanceledException(SR.OperationCanceled, this);
}
```
