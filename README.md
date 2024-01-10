# AndroidLibXrayLite

## Build requirements
* JDK
* Android SDK
* Go
* gomobile

## Build instructions
1. `git clone [repo] && cd AndroidLibXrayLite`
2. `gomobile init`
3. `go mod tidy -v`
4. `gomobile bind -v -androidapi 19 -ldflags='-s -w' ./`

## Use in Golang
```go
package main

import (
	"log"

	libv2ray "github.com/2dust/AndroidLibXrayLite"
)

// 实现 V2RayVPNServiceSupportsSet 接口
type MySupportSet struct{}

func (s *MySupportSet) Setup(Conf string) int        { return 0 }
func (s *MySupportSet) Prepare() int                 { return 0 }
func (s *MySupportSet) Shutdown() int                { return 0 }
func (s *MySupportSet) Protect(int) bool             { return true }
func (s *MySupportSet) OnEmitStatus(int, string) int { return 0 }

func main() {
	// 创建 V2RayPoint 实例
	v := libv2ray.NewV2RayPoint(&MySupportSet{}, false)

	// 设置服务器域名和配置文件内容
    // v.DomainName (服务器域名) 在 libv2ray 库中主要用于 DNS 预解析，它会预先解析你设置的域名到 IP 地址，然后在实际建立连接时使用这个 IP 地址，以避免 DNS 污染或者解析延迟 (真实用法是直接填要连接的服务器的域名或地址就行了)
	v.DomainName = "www.cyberlight.xyz"
	v.ConfigureFileContent = `
		{
			"inbounds": [
			  {
				"listen": "0.0.0.0",
				"port": 12345,
				"protocol": "shadowsocks",
				"settings":  {
					"password": "asdasd",
					"method": "aes-256-gcm",
					"level": 0,
					"email": "love@xray.com",
					"network": "tcp,udp"
				},
				"tag": "ss-inbound"
			  }
			],
			"routing": {
			  "rules": [
				{
				  "type": "field",
				  "inboundTag": ["ss-inbound"],
				  "source": ["192.168.11.144"],
				  "outboundTag": "direct"
				},
				{
				  "type": "field",
				  "inboundTag": ["ss-inbound"],
				  "network": "tcp,udp",
				  "outboundTag": "block"
				}
			  ]
			},
			"outbounds": [
			  {
				"protocol": "freedom",
				"settings": {},
				"tag": "direct"
			  },
			  {
				"protocol": "blackhole",
				"settings": {},
				"tag": "block"
			  }
			]
		  }
    `

	// 启动 xray
	err := v.RunLoop(false)
	if err != nil {
		log.Fatal(err)
	}

	// 停止 xray
	err = v.StopLoop()
	if err != nil {
		log.Fatal(err)
	}
}
```