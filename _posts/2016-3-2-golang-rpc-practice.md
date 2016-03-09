---
layout: post
title: Golang RPC 实践及填坑
---
一直用Golang标准库里的的RPC package来进行远程调用,简单好用.
但是随着任务数量的增大, 发现简单的像包里面的示例那样的代码出现了各种各样的问题,下面就把我踩过的一些坑记录一下吧.
首先是最初使用的文档里的版本,使用HTTP来发送请求.

server.go

```Go
func ListenRPC() {
	rpc.Register(NewWorker())
	rpc.HandleHTTP()
	l, e := net.Listen("tcp", ":4200")
	if e != nil {
		log.Fatal("Error: listen 4200 error:", e)
	}
 	go http.Serve(l, nil)
}
```

client.go

```Go
func call(srv string, rpcname string, args interface{}, reply interface{}) error {
	c, errx := rpc.DialHTTP("tcp", srv+":4200")
	if errx != nil {
		return fmt.Errorf("ConnectError: %s", errx.Error())
	}
	defer c.Close()
	return c.Call(rpcname, args, reply)
}
```

这样四五台机器的情况是够用了, 但是后来集群的机器增加到了十二台,
当请求大了之后发现总有很多任务卡住,通过call函数发送任务之后总会有没有返回的情况.
于是转而直接用tcp,效率有很大提升.

server.go

```Go
func ListenRPC() {
	rpc.Register(NewWorker())
	l, e := net.Listen("tcp", ":4200")
	if e != nil {
		log.Fatal("Error: listen 4200 error:", e)
	}
	go func() {
		for {
			conn, err := l.Accept()
			if err != nil {
				log.Print("Error: accept rpc connection", err.Error())
				continue
			}
			go rpc.ServeConn(conn)
		}
	}()
}
```

client.go

```Go
func call(srv string, rpcname string, args interface{}, reply interface{}) error {
    c, errx := rpc.Dial("tcp", srv+":4200")
	if errx != nil {
		return fmt.Errorf("ConnectError: %s", errx.Error())
	}
	defer c.Close()
	return c.Call(rpcname, args, reply)
}
```

这样局面有所改观,但是还是有任务卡住,概率大概是0.01%, 也就是一万个call里会有一个没有响应. 
仔细研究后发现这个rpc package有两大坑:

1. rpc包里的rpc.Dial函数没有timeout, 系统默认是没有timeout的,所以在这里可能卡住.所以我们可以采用net包里的
net.DialTimeout函数.

2. rpc包里默认使用gobCodec来编码解码, 这里io可能会卡住而不返回错误,所以我们要自己编写加入timeout的codec.
注意server这边读写都有timeout,但是client这边只有写有timeout,因为读的话并不能预知任务完成的时间.
于是就有了接下来这个版本的rpc,几十万个任务下来没有任何问题.

server.go

```Go
func TimeoutCoder(f func(interface{}) error, e interface{}, msg string) error {
	echan := make(chan error, 1)
	go func() { echan <- f(e) }()
	select {
	case e := <-echan:
		return e
	case <-time.After(time.Minute):
		return fmt.Errorf("Timeout %s", msg)
	}
}
func (c *gobServerCodec) ReadRequestHeader(r *rpc.Request) error {
	return TimeoutCoder(c.dec.Decode, r, "server read request header")
}

func (c *gobServerCodec) ReadRequestBody(body interface{}) error {
	return TimeoutCoder(c.dec.Decode, body, "server read request body")
}

func (c *gobServerCodec) WriteResponse(r *rpc.Response, body interface{}) (err error) {
	if err = TimeoutCoder(c.enc.Encode, r, "server write response"); err != nil {
		if c.encBuf.Flush() == nil {
			log.Println("rpc: gob error encoding response:", err)
			c.Close()
		}
		return
	}
	if err = TimeoutCoder(c.enc.Encode, body, "server write response body"); err != nil {
		if c.encBuf.Flush() == nil {
			log.Println("rpc: gob error encoding body:", err)
			c.Close()
		}
		return
	}
	return c.encBuf.Flush()
}

func (c *gobServerCodec) Close() error {
	if c.closed {
		// Only call c.rwc.Close once; otherwise the semantics are undefined.
		return nil
	}
	c.closed = true
	return c.rwc.Close()
}

func ListenRPC() {
	rpc.Register(NewWorker())
	l, e := net.Listen("tcp", ":4200")
	if e != nil {
		log.Fatal("Error: listen 4200 error:", e)
	}
	go func() {
		for {
			conn, err := l.Accept()
			if err != nil {
				log.Print("Error: accept rpc connection", err.Error())
				continue
			}
			go func(conn net.Conn) {
				buf := bufio.NewWriter(conn)
				srv := &gobServerCodec{
					rwc:    conn,
					dec:    gob.NewDecoder(conn),
					enc:    gob.NewEncoder(buf),
					encBuf: buf,
				}
				err = rpc.ServeRequest(srv)
				if err != nil {
					log.Print("Error: server rpc request", err.Error())
				}
				srv.Close()
			}(conn)
		}
	}()
}
```

client.go

```Go
type gobClientCodec struct {
	rwc    io.ReadWriteCloser
	dec    *gob.Decoder
	enc    *gob.Encoder
	encBuf *bufio.Writer
}

func (c *gobClientCodec) WriteRequest(r *rpc.Request, body interface{}) (err error) {
	if err = TimeoutCoder(c.enc.Encode, r, "client write request"); err != nil {
		return
	}
	if err = TimeoutCoder(c.enc.Encode, body, "client write request body"); err != nil {
		return
	}
	return c.encBuf.Flush()
}

func (c *gobClientCodec) ReadResponseHeader(r *rpc.Response) error {
	return c.dec.Decode(r)
}

func (c *gobClientCodec) ReadResponseBody(body interface{}) error {
	return c.dec.Decode(body)
}

func (c *gobClientCodec) Close() error {
	return c.rwc.Close()
}

func call(srv string, rpcname string, args interface{}, reply interface{}) error {
	conn, err := net.DialTimeout("tcp", srv+":4200", time.Second*10)
	if err != nil {
		return fmt.Errorf("ConnectError: %s", err.Error())
	}
	encBuf := bufio.NewWriter(conn)
	codec := &gobClientCodec{conn, gob.NewDecoder(conn), gob.NewEncoder(encBuf), encBuf}
	c := rpc.NewClientWithCodec(codec)
	err = c.Call(rpcname, args, reply)
	errc := c.Close()
	if err != nil && errc != nil {
		return fmt.Errorf("%s %s", err, errc)
	}
	if err != nil {
		return err
	} else {
		return errc
	}
}

```