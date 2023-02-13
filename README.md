# Pyrite

**Request**

```go
type Request struct {
	Owner       *Server
	Remote      *net.UDPAddr
	Session     string
	SessionData interface{}
	Body        string
}

func (r *Request) SetSession(data interface{})
func (r *Request) DelSession()
```

**Response**

```go
type Response struct {
	Body string
}
```

**Server**

```go
type Server struct {
	ip      net.IP
	router  map[string]func(Request) Response
	session map[string]interface{}
    rtt     map[string]int64
}

func NewServer(serverAddr string) Server
func (s *Server) AddRouter(identifier string, controller func(Request) Response)
func (s *Server) SetSession(session string, data interface{})
func (s *Server) DelSession(session string)
func (s *Server) Tell(remote *net.UDPAddr, identifier, body string) string
func (s *Server) Start()
```

**Client**

```go
type Client struct {
	server  net.UDPAddr
	router  map[string]funcRequest) Response
	Session string
}

func NewClient(serverAddr net.UDPAddr) Client
func (c *Client) AddRouter(identifier string, controller func(Request) Response)
func (c *Client) DelSession()
func (c *Client) Tell(identifier, body string)
```




