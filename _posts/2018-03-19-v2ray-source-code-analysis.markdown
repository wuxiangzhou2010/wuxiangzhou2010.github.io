---
layout: post
title: "V2ray source code analysis"
date: 2018-03-19 12:12 +0800
categories: tools
published: true
---

All the analysis are based on v2ray-core [4.18.0](https://github.com/v2ray/v2ray-core/tree/v4.18.0) release

## main

- distro

  - all/

    - configure v2ray what modules to be used
    - 这个文件中都是模块儿引入的语句， 通过引入模块儿， 相应的 `init()` function 被调用，如 feature, configLoader 之类的功能都能够被注册。

- json/

      通过init() function register `JSON` config loader

- jsonem/

      通过init() function register  a `JSON` config loader

- confloader/

  - configloader.go

    获取 config 文件的一个 io.ReadCloser 对象

  - external

    注册生效的 configfileloader

## main.go

the v2ray start entry point

- function:flag.parse: 分析傳入的參數
  - config, 类型 string, 指定的 config 文件
  - version, 类型 bool, 是否仅打印當前的版本號
  - test, 类型 bool, 是否仅測試 config 文件的有效性
  - format，类型 string, 指定 config 文件的類型
- function: startV2Ray()
  - `getConfigFilePath()`
    - 检查启动参数是否已经指定 config 文件的路径, 如果已经指定则直接返回 启动参数 中的路径
    - 检查当前目录中的配置文件是否有效
    - 检查环境变量中的配置文件是否有效 --> v2ray.location.config
  - `confloader.LoadConfig()`
    - 如果尚无有效的 configloader(EffectiveConfigFileLoader), 则返回 stdin,
    - 如果已经有有效的 configloader, 则返回 load 完之后的结果(io.ReadCloser, error)(external.loadConfigFile)
      - `"stdin"`:
      - `"http://"` 或 `"https://"` (通过 commander 先下载)
      - fixed file in ENV
    - `core.LoadConfig--> (*Config, error)`
      - getExtension--> configLoaderByExt--> loader 通过后缀来加载 loader
      - configLoaderByName--> Loader 通过文件名来加载 loader
    - core.New(config) : 通过 config 来创建一个 v2ray instance
    - `server.Start()`
      - start all features
    - `signal.Notify()`: 监听信号
      - os.Signal

## v2ray.go

```go
// 只要Runnable()的都是 server
type server interface{}
// Instance combines all functionalties in v2ray
type  Instance struct{
    features [] features.Feature
}
```

通过指定的 config 文件 生成一个 v2ray Instance 对象

```go
func New(config *Config)(*Instance, error){}
```

`New()` will do the following

- 根据配置文件创建 app 对象并添加相应的 feature
  - config.App--> `createObject()` --> `addFeature()`
- 添加必须的 features--> essentialFeatures
- add inbound and outbound manger--> addInboundHandlers()/addOutboundHandlers()

  通过相应配置生成 inbound/outbound handlers, 并添加进 manager

  - InboundHandlerConfig
  - OutboundHandlerConfig

Instance should contain following feature at least

- InboundHandlerManager
- OutboundHandlerManager
- Dispatcher

feature 注册以后依次调用 `start()` 方法启动

```go
func (s *Instance) Start() error{}
```

## features

`type Feature interface`

- `dns`
  - `type Client interface`
    - LookupIP()
  - feature implemented by `localdns/Client` in feature and `server` in app/dns
- `inbound`
  - `type Handler interface`
    - Tag()
    - GetRandomInboundProxy()
  - `type Manager interface`
    - GetHandler()
    - AddHandler()
    - RemoveHandler()
- `outbound`
  - `type Handler interface`
    - Tag()
    - Dispatch()
  - `type HandlerSelector interface`
    - Select()
  - `type Manager interface`
    - GetHandler()
    - GetDefaultHander()
    - AddHandler()
    - RemoveHandler()
- `policy`
- `routing`
  - `type Dispatcher interface`
    - Dispatch()
  - `type Router interface`
    - PickRoute()
- `stats`

## app

- commander
- dispacther

  - default.go

    - `type DefaultDispatcher struct`
      - Dispatch()
      - routedDispatch()
      - getLink()
    - sniffer()
    - shouldOverride()

  - sniffer.go

    - `type SniffResult interface`
      - SniffHttp
      - SniffTLS
      - SniffBittorrent
    - `type Sniffer struct`
      - sniff()

- dns/
  - server.go
  - udpns.go
  - nameserver.go
- proxyman/

  - inbound/

    - inbound.go

      - `type Manager struct`

        ```go
        type Manager struct {
            access          sync.RWMutex
            untaggedHandler []inbound.Handler
            taggedHandlers  map[string]inbound.Handler
            running         bool
        }
        ```

        - AddHandler()
        - GetHandler()
        - RemoveHandler()
        - Start()
          - taggedHandler.Start()
          - untaggedHandler.Start()

      - NewHandler() --> create a inbound handler
        - NewAlwaysOnInboundHandler()
        - NewDynamicInboundHandler()
      - New() --> create a inbound handler manager
        - a manager contain untaggedHanders and taggedHandlers

    - always.go
      - `type AlwaysOnInboundHandler struct`
        - Tag()
        - GetInbound()
        - GetRandomInboundProxy()
        - Start()
          - worker.Start()
      - NewAlwaysOnInboundHandler()
        - proxyman.ReceiverConfig
        - common.CreateObject
        - tcpworker
        - udpworker
    - dynamic.go
      - `type DynamicInboundHandler struct`
        - Tag()
        - GetRandomInboundProxy()
        - allocatePort()
        - closeWorkers()
        - refresh()
        - Start()
          - h.task.Start()
    - worker.go
      - `type worker interface`
        - `type tcpWorker struct`
          - Proxy()
          - Start()
            - internet.ListenTCP
        - `type udpWorker struct`
          - Proxy()
          - Start()
            - udp.ListenUDP

  - outbound/

    - handler.go

      ```go
      type Handler struct {
          tag             string
          senderSettings  *proxyman.SenderConfig
          streamSettings  *internet.MemoryStreamConfig
          proxy           proxy.Outbound
          outboundManager outbound.Manager
          mux             *mux.ClientManager
      }
      ```

      - `type Handler struct`
        - Tag()
        - Dispatch()
        - Address()
        - Dial()
        - GetOutbount()
        - Start()
      - NewHandler() -- create a new handler based on the given configuration
        - the config is core.OutboundHandlerConfig

    - outbound.go

      ```go
       // Manager is to manage all outbound handlers.
      type Manager struct {
          access           sync.RWMutex
          defaultHandler   outbound.Handler
          taggedHandler    map[string]outbound.Handler
          untaggedHandlers []outbound.Handler
          running          bool
      }
      ```


      - `type Manager struct`
        - GetDefaultHandler()
        - GetHandler()
        - AddHandler()
        - RemoveHandler()
        - Select()
        - Start()
      - New() --> crreate ougbound handler manager manages all outbound handlers
        - a manager contains defaultHandler/taggedHandler/untaggedHandlers

- command/

  - `type handlerServer struct`
    - AddInbound()
    - RemoveOutbound()
    - AlterInbound()
    - AddOutbound()
    - RemoveOutbound()
    - AlterOutbound()

- router

  - router.go
    - `type Router struct`
      - PickRoute()
      - pickRouteInternal()
      - domainStrategy
  - balancing.go
    - `type BalancingStrategy interface`
      - PickOutbound()
  - condition.go
    - `type Condition interface` --> `Apply()`
      - type DomainMatcher struct
      - type MultiGeoIPMatcher struct
      - type PortMatcher struct
      - type NetworkMatcher struct
      - type UserMatcher struct
      - type InboundTagMatcher struct
  - config.go
    - `type Rule struct`
      - GetTag()
      - Apply()

- reserve
- stats
- log
- policy

## config.go

- `type ConfigFormat struct`
  - name
  - extension
  - Loader
- RegisterConfigLoader(format)

  - Protobuf --> loadProtobufConfig()
  - json

  注册相应 format 的 config 文件的加载器

- LoadConfig()

  通过文件名，文件后缀读取相应的 config， 生成 config 对象以供使用。

## infra

- bazel: for release
- conf

  - command/

    register the `config` command

  - json/

    json reader, filer the comments in the json file

  - serial/

    load JSON config

  - v2ray config
  - vmess config
  - http config
  - socks config
  - ss config
  - router config
  - transport config
  - dns config
  - freedom config
  - reverse config

- control

  - `type Command interface` --> ConfigCommand/LoveCommand/FetchCommand/UUIDCommand/VerifyCommand

    - Name()
    - Description()
    - Execute()

  - `control.RegisterCommand()`

    注册相应的 command 以供调用

- vprotogen

  generate protobuf

## common

- bitmask

  bits manipulation

- buf

  byte pool

- dice

  random generator

  ```go
  // Roll returns a non-negative number between 0 (inclusive) and n (exclusive).
  func Roll(n int) int {
  if n == 1 {
      return 0
  }
  return rand.Intn(n)
  }
  ```

  Here `1` is treated as a special case. which is increase the performance for Roll(1) significantly.

  ```sh
  xihajuan2010@instance-1:~/go/src/v2ray.com/core/common/dice$ go test -bench=.
  goos: linux
  goarch: amd64
  pkg: v2ray.com/core/common/dice
  BenchmarkRoll1    1000000000         2.77 ns/op
  BenchmarkRoll20   30000000        40.4 ns/op
  BenchmarkIntn1    50000000        33.8 ns/op
  BenchmarkIntn20   50000000        38.5 ns/op
  ```

- errors
- log
- net
- platform
- uuid
- task
- stack
- signal

get environment value
windows related

## context.go

- FromContext
- MustFromContext

returns an `Instance` from the given context.

## core.go

version and statement

## proxy

package proxy contains all proxies used by v2ray. to implement an inbound or outbound proxy, one needs to do the following:

1. Implement the interfaces(s) below
2. Register a config creator through `common.RegisterConfig`

```go
type Inbound interface{}
type Outbound interface{}
type UserManager interface{}
type GetInbound interface{}
type GetOutbound interface{}
```

- blackhole
- dokodemo
- freedom
- http
- ss
- socks
- vmess

## transport

- internet/

  - http/
  - tcp/
    - config.go
      - protocolName --> `tcp`
    - dialer.go
      - Dial()
    - hub.go
      - `type Listener struct`
        - ListenTCP()
  - udp/
  - kcp/
  - tls/
  - webSocket/
  - domainsocket/
  - headers/
  - tcp_hub.go
    - RegisterTransportListener()
    - `type Listener interface`
      - ListenTCP()
      - ListenSystem() --> tcp
      - ListenSystemPacket()--> udp

- pipe/
- dialer.go
  - RegisterTransportDialer()
  - Dial()
  - DialSystem() --> calls system dialer
  - transportDialerCache

## release

- config/

  config in the release pkg

- doc/
- verify/
- release scripts

## call stack

    init proxyman
    app inbound handler --> NewHandler --> AlwaysOnInboundHandler/DynamicInboundHandler --> init the tcp/udp worker --> worker.Start()-->ListenTCP/ListenUDP --> callback --> transport -- > Dispatch (route dns ) --> outbound handler -->process -->dialer.Dial
    pickroute --> Resolve --> LookupIP --> HandleResponse--> updateIP
    app--> proxyman-->inbound worker tcpworker udpworker callback

    socks5 server --> process --> processTCP/handleUDPPayload

    app--> proxyman-->outbound manager GetDefaultHandler

strings.ToLower()
exec.Command
