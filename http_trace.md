### http_trace
```
CREATE TABLE http_trace (
    tracingid varchar(64) PRIMARY KEY,
    method varchar(10),
    path text,
    requestbody text,
    responsebody text,
    statuscode int,
    started_at timestamptz,
    ended_at timestamptz
);
```
