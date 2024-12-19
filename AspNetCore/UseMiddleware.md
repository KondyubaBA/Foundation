```cs
if (typeof(IMiddleware).IsAssignableFrom(middleware))
{
  if (args.Length != 0)
  {
    throw new NotSupportedException(Resources.FormatException_UseMiddlewareExplicitArgumentsNotSupported(typeof(IMiddleware)));
  }
  InterfaceMiddlewareBinder @object = new InterfaceMiddlewareBinder(middleware);
  return app.Use(@object.CreateMiddleware);
}
MethodInfo[] methods = middleware.GetMethods(BindingFlags.Instance | BindingFlags.Public);
MethodInfo methodInfo = null;
MethodInfo[] array = methods;
foreach (MethodInfo methodInfo2 in array)
{
  if (string.Equals(methodInfo2.Name, "Invoke", StringComparison.Ordinal) || string.Equals(methodInfo2.Name, "InvokeAsync", StringComparison.Ordinal))
  {
    if ((object)methodInfo != null)
    {
      throw new InvalidOperationException(Resources.FormatException_UseMiddleMutlipleInvokes("Invoke", "InvokeAsync"));
    }
    methodInfo = methodInfo2;
  }
}
if ((object)methodInfo == null)
{
  throw new InvalidOperationException(Resources.FormatException_UseMiddlewareNoInvokeMethod("Invoke", "InvokeAsync", middleware));
}
if (!typeof(Task).IsAssignableFrom(methodInfo.ReturnType))
{
  throw new InvalidOperationException(Resources.FormatException_UseMiddlewareNonTaskReturnType("Invoke", "InvokeAsync", "Task"));
}
ParameterInfo[] parameters = methodInfo.GetParameters();
if (parameters.Length == 0 || parameters[0].ParameterType != typeof(HttpContext))
{
  throw new InvalidOperationException(Resources.FormatException_UseMiddlewareNoParameters("Invoke", "InvokeAsync", "HttpContext"));
}
ReflectionMiddlewareBinder object2 = new ReflectionMiddlewareBinder(app, middleware, args, methodInfo, parameters);
return app.Use(object2.CreateMiddleware);
```
