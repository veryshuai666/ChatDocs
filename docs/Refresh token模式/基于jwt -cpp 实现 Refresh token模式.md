# Refresh token模式

## Refresh token 模式的作用

        使用token的本质是api使用安全，也就是要确保当前操作是用户本人在执行。一般来说，我们会认为真正确保我们账号安全的是账号和密码。可在底层并非如此，api执行操作时，确认当前操作用户身份的依据是token（token里的用户唯一标识，如uid等等）。

        单token模式下，token需要承担确认用户身份合法性，以及访问路由且传递用户身份信息（uid或者其他身份标识）这两层责任。所以，单token模式下尴尬的点在于，有效期太短需要频繁刷新，而单token下要怎么刷新token呢？显然拿着一个已经过期的有效证明是苍白无力的。只有登录才能确保是用户本人操作，所以避不开频繁登录这个缺陷。有效期太长会导致意外泄漏的风险很大，一旦被token盗取，用户需要马上重新登录

        Refresh token 模式则是拆分了这个问题，由access token去访问路由，传递用户信息。在access token上还加码了Refresh token。Refresh token在用户登录时获取，继承了用户由正确的账号密码得来的合法性。Refresh token持有者可以选择刷新token pair，取缔当前token pair并产生新的合法token pair，从而确保合法用户对访问路由，也就是（除了账号密码之外）账户使用的最高操纵权限。

        为了解决单token模式的局限性，Access token和Refresh token的有效时长是不同的。一般来说，Access Token的有效时长是15分钟，Refresh token的有效时长是两周及以上。这么做有几点考量：1. 短效的Access token能防止被意外泄漏后，窃取者长期使用该token进行操作。2. 长效的Refresh token能避免用户短时间内多次登录，优化用户体验。3. 轮换机制降低了token意外泄漏的风险，因为token pair一直在进行刷新。且高活跃度用户可以避开重复登录的麻烦。

## JWT 简单介绍

        JWT 由三个部分组成：Header , Payload , Signature，这里不解释整个设计的原理和由来。另外，这里的实现只是众多实现中的一种，对于细节的处理根据实际可以有不同处理。

![](C:\Users\23726\AppData\Roaming\marktext\images\2026-03-11-18-08-12-82880b8ef2b8f78e89d7cc1a337c02cc.png)

### Header

alg ：签名算法
必要字段。服务端在校验时根据这个字段去判断和解析token。可以自由选择安全性高的算法

typ：token类型
可选字段。这里的类型和后面要说的type不一样，RFC 7519 协议要求JWT的header的typ如果存在必须是固定值JWT

### Payload

Payload 是用户信息的载体，可以根据不同需求存储需要的信息。但是这里不能存储敏感信息，一般来说只存储最主要的信息，比如 用户唯一标识 uid，token的唯一标识 jti ，token的创建时间和过期时间

### Signature

Signature是token安全性的最重要保障，存在目的是确保token传递的信息不被修改，可以简单了解一下加密公式：

`Base64url { alg [ Base64url（Header）. Base64url（Payload），secret ] }`

这里的alg就是Header的alg字段





















