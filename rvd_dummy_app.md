### rvd start
```
rvd -listen tcp:7500 -service 7500 -network ";239.1.1.10"
```

### client app
```
using TIBCO.Rendezvous;

class DummyServer
{
    static Transport transport;

    static void Main(string[] args)
    {
        // RV 초기화
        TIBCO.Rendezvous.Environment.Initialize();

        // RVD가 같은 PC면 daemon=127.0.0.1:7500
        transport = new NetTransport(
            service: "7500",
            network: "239.1.1.10;192.168.10.151",
            daemon:  "tcp:127.0.0.1:7500"
        );

        // 클라이언트가 보낼 Subject "RRR"을 Listen
        Listener listener = new Listener(
            Queue.Default,
            transport,
            subject: "RRR",
            new MessageHandler(OnRequest)
        );

        Console.WriteLine("Dummy Server Started. Listening on subject RRR...");
        DispatchLoop();
    }

    static void OnRequest(object o, Message msg)
    {
        Console.WriteLine("Received request: " + msg.ToString());

        // Response 메시지 생성
        Message reply = new Message();
        reply.AddField("result", "OK");
        reply.AddField("timestamp", DateTime.Now.ToString("HH:mm:ss"));

        // Request로부터 응답채널 정보 가져오기
        SendReply(reply, msg);
    }

    static void SendReply(Message reply, Message req)
    {
        try
        {
            transport.SendReply(reply, req);
        }
        catch (Exception ex)
        {
            Console.WriteLine("Reply send error: " + ex);
        }
    }

    static void DispatchLoop()
    {
        while (true)
        {
            Queue.Default.Dispatch();
        }
    }
}

```


### client app
```
using TIBCO.Rendezvous;

class DummyClient
{
    static Transport transport;

    static void Main()
    {
        TIBCO.Rendezvous.Environment.Initialize();

        transport = new NetTransport(
            service: "7500",
            network: "239.1.1.10;192.168.10.151",
            daemon:  "tcp:192.168.10.151:7500"
        );

        // Reply 받을 Listener
        Listener listener = new Listener(
            Queue.Default,
            transport,
            inbox: out string inbox,
            new MessageHandler(OnReply)
        );

        while (true)
        {
            // Request 생성
            Message msg = new Message();
            msg.SendSubject = "RRR";
            msg.ReplySubject = inbox;

            msg.AddField("query", "Hello Server");

            // Send
            transport.Send(msg);

            Console.WriteLine("Sent request to RRR.");
            
            // 응답 대기
            Queue.Default.Dispatch();

            Thread.Sleep(1000);
        }
    }

    static void OnReply(object o, Message msg)
    {
        Console.WriteLine("Reply received: " + msg.ToString());
    }
}

```
