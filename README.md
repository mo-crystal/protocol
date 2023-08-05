# Pyrite

### Struct

**Server**

```go
type Server struct {
    ip      net.IP
    router  map[string]func(Request) Response
    session map[string]interface{}
    rtt     map[string]int64
    lastAccept map[string]int64
    maxLifeTime int64
}

func NewServer(serverAddr string, enableSession bool, maxTime int64) (Server, error)
func (s *Server) AddRouter(identifier string, controller func(Request) Response)
func (s *Server) SetSession(session string, data interface{})
func (s *Server) DelSession(session string)
func (s *Server) Tell(remote *net.UDPAddr, identifier, body string) (Response,error)
func (s *Server) Start()
func (s *Server) GC()
```

**Client**

```go
type Client struct {
    server  net.UDPAddr
    router  map[string]funcRequest) Response
    Session string
}

func NewClient(serverAddr net.UDPAddr) (Client, error)
func SayHello() error
func (c *Client) AddRouter(identifier string, controller func(Request) Response)
func (c *Client) DelSession()
func (c *Client) Tell(identifier, body string) (Response, error)
```

### Process Control

服务机：S

客户机：C

**报文格式**

```go
Sequence (int32)
Identifier (string, end with \0)
Body (bytes)
```

**握手阶段**

```go
C -> S
    Session: ""
    Identifier: "hello"
    Sequence: 0
    //空行
    Body:""

S -> C
    Session: session(New Client Session)
    Identifier: "hello"
    Sequence: 0
    //空行
    Body: maxLifeTime (服务器的最大连接保持时间)

C -> S
    Session: session
    Identifier: "established"
    Sequence: 0
    //空行
    Body:""
```

握手阶段双方各自记录相应的 rtt，用于后期处理超时问题。

**请求流程**

```go
Sender -> Receiver
    Session: client session (用户 session)
    Identifier: (用户设定的 router identifier)
    Sequence: n (由发送当前上下文而定)
    //空行
    Body: (自定义数据)

Receiver -> Sender
    Session: client session (请求用户 session)
    Identifier: "ack"
    Sequence: n
    //空行
    Body: data (服务器答复数据)
```

若 3 个 rtt 后 Sender 仍未收到 ack，此时判定为超时，进行重传操作。

重传三次后仍未收到 ack 答复，则判定为不可达，并向上层反应错误，等待上层处理。

Receiver 收到重复数据包时，直接返回 ack 并不做任何处理。

**保活流程**

客户端若仍有连接需求，需在 `maxLifeTime` 时间以内发送保活请求，防止服务器将连接释放。

```go
C -> S
    Session: session (客户端session)
    Identifier: "keep-alive"
    Sequence: 0
    //空行
    Body:""

S -> C
    Session: session(New Client Session)
    Identifier: "keep-alive"
    Sequence: 0
    //空行
    Body: ""
```

若该请求因不可抗力超时，则客户端认为本次连接已断开，若有需求需重新进行握手操作。

服务端的连接将由 `GC` 进行回收。

**资源回收**

调用 `GC` 对所有连接的 `lastAccept` 检测，对于过长时间未进行操作的连接进行资源回收，释放连接。

# Implements

[Pyrite (Go)](https://github.com/mo-crystal/Pyrite)

[Pyrite (Cpp)](https://github.com/mo-crystal/Pyrite-cpp)
