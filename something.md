## 语法 
### c#8
    Activity.Current is { } activity ? Convert.FromHexString(activity.TraceId.ToHexString()) : null;