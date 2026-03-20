## Refresh token模式的实现

1. 第一步，我们先确定要搭建的token的雏形。
- Header ：typ = JWT（固定值），alg = HS256（可以自由选择）

- Payload ：uid (用户uid）, typ = "Access/Refresh" ，exp （过期时间）

- Signature : secret（加密算法所需的密钥（有多个轮换的密钥最好））

Payload中”typ“字段的解释： 既然模式下有两种token，我们需要一个方式去区分使用的是哪一种token。所以我们在token 信息载荷Payload里写入两个信息：用户的uid，当前token的类型typ（Access/Refresh)

2. 构建token公式

有了雏形之后，只要获取每部分需要的参数，就可以进行搭建了，jwt-cpp生成token的方式很简单，只需要进行链式调用：

```
std::string token = jwt::create()
            .set_type("JWT")
            .set_algorithm("HS256")
            .set_issued_at(now)
            .set_expires_at(expiry)
            .set_id(jti)
            .set_payload_claim("uid",  jwt::claim(uid))
            .set_payload_claim("typ",  jwt::claim(std::string(typ_str)))
            .sign(jwt::algorithm::hs256{ secret });
```

简单解释一下：

set_payload_claim：在Payload存储你自定义的字段和值。
set_id：设置jti；用户有自己的id，也就是uid，而JWT的id则是jti，设置jti是为了方便对token进行管理

3. 有了公式之后，只需要获取参数值就可以得到token

jti 我们可以通过写一个工具函数GenUuid()获取，这里提供一个windows和linux的版本：

```
#ifdef _WIN32
        // Windows implementation
#include <rpc.h>
        std::string GenerateUid() {
            UUID uuid;
            UuidCreate(&uuid);

            unsigned char* str;
            UuidToStringA(&uuid, &str);

            std::string result(reinterpret_cast<char*>(str));
            RpcStringFreeA(&str);

            return result;
        }
#else
        // Linux implementation
#include <uuid/uuid.h>
        std::string GenerateUid() {
            uuid_t uuid;
            uuid_generate(uuid);

            char str[37];
            uuid_unparse(uuid, str);

            return std::string(str);
        }
#endif
```

now和expiry的获取很简单 ：

```
auto now = std::chrono::system_clock::now();
auto expiry = now + std::chrono::seconds{ ttl_seconds };
//ttl_second是有效时长，一般来说access token = 15 min
//refresh token = 2 weeks
```

最重要的部分来了：secret，也就是密钥

secret最好通过环境变量的方法获取，这里采用本地配置文件的方法去获取secret，安全性相对不那么高，但是复杂度也会更低一些
读取文件首先要有文件，我们先不用手动创建，因为需要一个兜底机制，保证在特殊情况下，文件被意外删除时系统能正常运作，我们创建一个SecretHelper类，帮助我们完成有文件且有密钥时读取密钥，无文件/无密钥时创建文件/创建密钥实现兜底。我们把密钥作为SecretHelper的私有成员，通过getter方法获取密钥，这个类需要几个方法：

加载密钥的LoadFromFile（）

```
std::string LoadFromFile(const std::string& file_path)
{
    std::ifstream file(file_path);
    if(file.is_open()
    {
        Json::Value json_secret;
        file>>root;
        if(!root.isMember("secret")
            return root["secret"].asString();    
    }
    

}
```

获取密钥的GetSecret（）
生成密钥的GenSecret（）
