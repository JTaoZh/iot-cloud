# iot-cloud

## 设备接入前端解码框架

编码包括head、body、checkcode、endcode四部分，其中包含多个字段，所有字段类型均可以为十六进制（Number）或字符串（string），由字段中的type指定。

### head 
在head部分有可选的decoder\_id及必选的device\_id。

decoder\_id是指定编码头部的第一个字段，指定使用哪个解码器。若一个程序中使用多个解码器，则必需开启decoder\_id字段。

device\_id是编码头部的第二个字段（若不使用decoder\_id，则为第一个字段）。

### body
body部分包含多个字段，字段的排序由"sequence"确定。

### checkcode
checkcode，校验码，是可选字段，由"type"指定校验方式。

### endcode
endcode，结束符，可选字段（当启用SLIP时则忽略）。

### config
config字段是编码器的配置，配置是否向消息总线发送消息以及在MongoDB记录消息。

为减小消息通信的包大小，使用Msgpack进行压缩。

在MongoDB进行存储时，记录head和body字段（不改变body字段的嵌套结构），每个字段只记录key和实际接收到的value值。

### 配置文档总览
程序启动时根据参数读取Json文档，该文档可以来自MongoDB、RESTful API或者额外的传入参数。

```
{
    "decoder":[
        // decoder对象示例如下
        {
            "_id":value, // 解码器ID
            "config":{
                "port":Number, //监听端口
                "msg_en":boolean, //是否向消息总线发送消息
                "msg_ip":string, // 消息总线IP 
                "msg_port":Number, // 消息总线端口
                "db_en":boolean, //是否记录至MongoDB
                “db_ip": string,    // 数据库IP
                "db_port": Number,  // 数据库端口
                "enSlip": boolean  // true:使用SLIP进行解包; false（默认值）:不使用SLIP进行解包
            },
            ”head":{
                “decoder_id":{       // 
                    "enable":boolean, // true:数据包头部为解码器标识; false:数据包头部不使用解码器标识
                    "type":String,  // 解码器标识类型：“int","string",下同
                    "value":String/Number,  // 编码器标识内容，取决于type，全匹配
                    "description":String // 描述，下同
                }
                "device_id":{    // 设备ID
                    "type":String,
                    "size":Number, // 该字段长度，下同
                    ”description":String
                }
            },
            "body":[
                // "body"数组内的对象如下所示，同构，可多层嵌套
                {
                    "key":string,   // 字段名
                    "sequence":number, // 该字段为数据包的第几个字段
                    "description”：string,
                    "type":String,  // 字段类型：“int","string","embedded"
                    "size":Number  // 当"type"为”int"或“string"时，此值有效，为该字段长度
                    /* 当"type"为"embedded"时，此值有效，内嵌一个"body"对象，可多层嵌套
                    "body":[{   
                        “key":string, 
                        "sequence":number, 
                        "description”：string,
                        "type":String, 
                        "size":Number  
                    }]
                    */           
                }
            ],
            "checkcode":{
                "enable":boolean,   // 是否使用校验码
                "type":"string"     // 校验方式
            }
            "endcode":{
                "enable":boolean,   // 是否使用结束码，若使用SLIP编码，则忽略endcode
                "type":String, 
                "size":Number, 
                ”description":String
            }
        }
    ]
}
```

---

### 示例

* decoder 示例
```
{
    "decoder":[
        {
            "_id":1, // 解码器ID
            "config":{
                "enSlip": false
            },
            ”head":{
                “decoder_id":{       
                    "enable":true, 
                    "type":"string",  
                    "value":"example",  
                    "description":"示例" 
                }
                "device_id":{   
                    "type":“string”,
                    "size":2, 
                    ”description":"示例设备ID"
                }
            },
            "body":[
                {
                    "key":"key2",   
                    "sequence":2, 
                    "description”："字段2",
                    "type":“string”,
                    "size":2            
                },
                {
                    "key":"key1",   
                    "sequence":1, 
                    "description”："嵌套字段1",
                    "type":“embedded”,
                    "body":[{
                        "key":"key1_2",   
                        "sequence":2, 
                        "description”："字段1.2",
                        "type":“string”,
                        "size":2  
                    },
                    {
                        "key":"key1_1",   
                        "sequence":1, 
                        "description”："字段1.1",
                        "type":“string”,
                        "size":1  
                    }
                    ]           
                },
            ],
            "checkcode":{
                "enable":false                
            }
            "endcode":{
                "enable":false
            }
        }
    ]
}
```

* 输入数据（字符串形式）
```
example01a23ce
```

* 输出记录（Json形式）
```
{
    "device_id":"01",
    "key1":{
        "key1_1":"a",
        "key1_2":"23",
    },
    "key2":"ce"
}
```
