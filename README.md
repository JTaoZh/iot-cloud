# iot-cloud

## 设备接入前端解码框架
程序启动时根据参数读取Json文档，该文档可以来自MongoDB、RESTful API或者额外的传入参数。

```
{
    "_id":value, // 解码器ID
    "port":Number, //监听端口
    "option":{
        "enSlip": boolean，  // true:使用SLIP进行解包; false（默认值）:不使用SLIP进行解包
        “decode\_id":{       // 
            "enable":boolean, // true:数据包头部为解码器标识; false:数据包头部不使用解码器标识
            "type":String,  // 解码器标识类型：“int","string",下同
            "value":String/Number,  // 编码器标识内容，取决于type，全匹配
            "description":String // 描述，下同
        }
    },
    ”head":{
        "\_id":{    // 设备ID
            "type":String,
            "size":Number, // 该字段长度，下同
            ”description":String
        }
    },
    "body":[
        // "body"数组内的对象如下所示，同构，可多层嵌套
        {
            "sequence":number, // 该字段为数据包的第几个字段
            "description”：string,
            "type":String,  // 字段类型：“int","string","embedded"
            "size":Number  // 当"type"为”int"或“string"时，此值有效，为该字段长度
            /* 当"type"为"embbeed"时，此值有效，内嵌一个"body"对象，可多层嵌套
            "body":"{   
                "sequence":number, 
                "description”：string,
                "type":String, 
                "size":Number  
            } 
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
```