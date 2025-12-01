# 渠道对接文档

<a href="README_en.md" style="color:blue">English</a> | <a href="README_zh.md" style="color:blue">中文</a> 


# 对接简介
渠道伙伴和StrideMobi对接的完整流程如下：
* 从渠道经理处获得渠道id、token、以及相关接口的地址, 并提供回传接收地址给到渠道经理
* 获取offer列表, 参考<a href="#get_offer" style="color:blue">get_offer</a>
* 发送展示点击, 参考<a href="#impressionclick" style="color:blue">impression/click</a>
* 获取回传, 参考<a href="#postback" style="color:blue">postback</a>


# get_offer
渠道通过该接口拉取可用的offer列表
## Request
渠道发送如下GET请求获取offer列表
```
https://{domain}/{api_version}/channel/get_offers?channel_id=${channel_id}&timestamp=${timestamp}&sign=${sign}
```

### 参数说明
|参数|含义|样例|
|:--|:--|--:|
|{domain}|StrideMobi API 域名,请从渠道经理处获取||
|{api_version}|API版本，当前版本为V1.0|V1.0|
|{channel_id}|对接时获取的渠道id|113|
|{timestamp}|当前UTC时间戳|1760170094|
|{sign}|使用对接时获取的token所做的签名 <a href="#sign生成逻辑" style="color:blue">点击查看 sign 生成逻辑</a>|6e0fc913cdd5da0913aef0dc9755edbddf7fd9733328322908f9be3bdcc325d6|

### Response
访问API后，将得到以下结果：
```json
{
    "code": 0,
    "message": "success",
    "offer_num": {offer_num}, // 广告数量
        "offers": [
            {
                // offer基础信息
                "offer_id": {offer_id},
                "offer_name": "{offer_name}", // 广告名称
                "status": {status}, // 广告状态
                "promotion_type": {promotion_type}, 
                "app_name": "{app_name}", // 广告app名称
                "package_name": "{package_name}", // 应用包名
                "preview_link": "{preview_link}", 
                "click_url": "{click_url}",
                "is_support_vta": {is_support_vta}, 
                "impression_url": "{impression_url}", 
                "kpi_note":"{kpi_note}", 

                // 素材信息
                "icon_url" : "{icon_url}", 
                "image_url": "{image_url}",
                "video_url": "{video_url}", 

                // 定向信息
                "target_countrys":["{country}", "{country}"],
                "target_cities":["{city}", "{city}"], 
                "target_device_types":[{device_type}, {device_type}], 
                "target_platform": [{target_platform}, {target_platform}], 
                "min_osv": {min_osv}, 
                "max_osv": {max_osv}, 
                "siteid_blacklist": ["{siteid}", "{siteid}"], 


                // 结算信息
                "billing_event": "{billing_event}",
                "billing_type": {billing_type},
                "billing_price": 100,

                // 预算信息
                "billing_event_cap": {billing_event_cap}, 
                "impression_cap": {impression_cap},
                "click_cap": {click_cap}
            },
            {
                // 多个offer重复此结构
            }
        ]
}
```
字段说明
*其中有些字段的含义较为复杂，表格后会详细说明*

| 字段名 | 含义 | 取值类型 | 取值样例 |
|:------|:------|:------|------:|
| code | 响应状态码，0表示成功 | integer | 0 |
| msg | 响应消息描述 | string | success |
| offer_num | 广告数量 | integer | 10 |
| offers | 广告列表数组,内容个数和offer_num一致 | array |  |
| offer.offer_id | 广告的唯一ID | integer | 28531223 |
| offer.offer_name | 广告名称 | string | shein |
| offer.status | 参考<a href="#广告状态-offerstatus" style="color:blue">广告状态</a> | integer | 1 |
| offer.promotion_type | 参考<a href="#广告实体-offerpromotion_type" style="color:blue">广告实体</a> | integer | 1 |
| offer.app_name | 广告app名称 | string | shein |
| offer.package_name | 应用包名 | string | com.zzkko  |
| offer.preview_link | 应用预览链接 | string | https://play.google.com/store/apps/details?id=com.zzkko |
| offer.click_url | 点击跳转链接 | string | https://t.xxx.com/click?params=${params} |
| offer.is_support_vta | 是否支持VTA追踪 | boolean | 0/1 |
| offer.impression_url | 曝光追踪链接(只有支持VTA时才有值) | string | https://t.stridetracking.com/imp?params=${params} |
| offer.kpi_note | KPI说明 | string | "cvr大于0.5%，付费率超过1%" |
| offer.icon_url | 图标URL | array | ["https://cdn.stridemobi.com/${package_name}/icon.png"] |
| offer.image_url | 图片URL | array | ["https://cdn.stridemobi.com/${package_name}/image.png"] |
| offer.video_url | 视频URL | array | ["https://cdn.stridemobi.com/${package_name}/video.mp4"] |
| offer.target_countrys | 目标国家列表,参考<a href="#国家城市代码" style="color:blue">国家城市代码</a> | array | ["US","CN"] |
| offer.target_cities | 目标城市列表,参考<a href="#国家城市代码" style="color:blue">国家城市代码</a> | array | ["SIN", "LON", "DFW"] |
| offer.target_device_types | 参考<a href="#设备类型-offertarget_device_types" style="color:blue">设备类型</a> | array | [1,2] |
| offer.target_platform | 参考<a href="#设备平台-offertarget_platform" style="color:blue">设备平台</a> | array | [1,2] |
| offer.min_osv | 参考<a href="#操作系统版本-offerosv" style="color:blue">操作系统版本</a> | integer | 80101 |
| offer.max_osv | 参考<a href="#操作系统版本-offerosv" style="color:blue">操作系统版本</a> | integer | 150101 |
| offer.siteid_blacklist | 媒体黑名单 | array | ["siteid1","siteid2"] |
| offer.billing_event | StrideMobi和渠道之间的结算事件 | string | install |
| offer.billing_type | billing_event对应的分类,参考<a href="#计费类型-offerbilling_type" style="color:blue">计费类型</a> | integer | 1 |
| offer.billing_price | 计费价格 | number | 3.1 |
| offer.billing_event_cap | 日计费事件数cap | integer | 10000 |
| offer.impression_cap | 日展示cap | integer | 20000000 |
| offer.click_cap | 日点击cap | integer | 20000000 |


# impression/click
渠道通过该接口上报用户点击或展示广告的事件。
每个offer上报地址不同, get_offer接口会返回具体地址。例：
```
http://www.sttt.com/click?ad_type={}&channel_id=3&city={}&click_id={}&country={}&bundle={}&creative_id={}&device_brand={}&device_model={}&device_os_version={}&device_platform={}&device_type={}&gaid={}&gaid_md5={}&gaid_sha1={}&ip={}&offer_id=4&site_id={}&user_agent={}&passthrough={}
```
展示和点击参数一致，只是path不同。

## 请求参数说明
get_offer返回链接部分参数已经确定，部分参数为{}，渠道需要在上报时替换为具体值。
以下是参数说明

| 参数名 | 类型 | 填写说明 | 含义 |
|:------|:------|:------|:------|
| offer_id | int | 会有确定值, 无需渠道填写 | 广告的唯一ID |
| channel_id | int | 会有确定值, 无需渠道填写 | 渠道的唯一ID |
| click_id | string | 自选 | 点击id,可供渠道自己进行追踪和归因 |
| site_id | string | 自选 | 媒体站点ID，渠道内部区分不同的媒体来源 |
| device_platform | int | 必填 | 参考<a href="#设备平台-offertarget_platform" style="color:blue">设备平台</a> |
| device_type | int | 必填 | 参考<a href="#设备类型-offertarget_device_types" style="color:blue">设备类型</a> |
| device_os_version | string | 必填 | 可传原始字符串如16.3.1，也可转换为<a href="#操作系统版本-offerosv" style="color:blue">操作系统版本</a>格式 |
| device_brand | string | 尽量填写 | 设备品牌 |
| device_model | string | 尽量填写 | 设备型号 |
| idfv | string | ios时尽量填写 | iOS设备的IDFV标识符 |
| idfv_sha1 | string | ios时尽量填写 | iOS设备的IDFV的SHA1值 |
| idfv_md5 | string | ios时尽量填写 | iOS设备的IDFV的MD5值 |
| idfa | string | ios时尽量填写 | iOS设备的IDFA广告标识符 |
| idfa_sha1 | string | ios时尽量填写 | iOS设备的IDFA的SHA1值 |
| idfa_md5 | string | ios时尽量填写 | iOS设备的IDFA的MD5值 |
| gaid | string | android时尽量填写 | Android设备的GAID广告标识符 |
| gaid_sha1 | string | android时尽量填写 | Android设备的GAID的SHA1值 |
| gaid_md5 | string | android时尽量填写 | Android设备的GAID的MD5值 |
| ad_type | string | 必填 | 广告类型 |
| country | string | 有国家定向时必填 | 用户所在国家代码，如"US"、"CN"等 |
| city | string | 有城市定向时必填 | 用户所在城市 |
| ip | string | 必填 | 用户IP地址 |
| user_agent | string | 必填 | 用户浏览器的User-Agent信息 |
| bundle | string | 尽量填写 | 流量来源包名, 例如com.zzkko, id128883|
| creative_id | string | 尽量填写 | 创意ID |
| passthrough | string | 自选 | 渠道自定义参数, 会原封不动地回传 |

**注意**：
- iOS时 idfa、idfv 以及对应Sha1、MD5值必填其一
- Android时 gaid 以及对应Sha1、MD5值必填其一
- click_id、site_id 均限制24个字符,超过会被截断
- passthrough限制200个字符, 超过会被截断

## 返回结果
impression/click接口会返回json格式的结果, 示例如下：
```json
{
    "code": 0,
    "msg": "success"
}
```

code、msg参考<a href="#impressionclick返回结果说明" style="color:blue">impression/click返回结果说明</a>

## postback

StrideMobi检测相关事件后会给渠道回传. 渠道可在对接时提供回传地址,相关宏参数会在回传时替换为具体值.
例:
```
http://www.sttt.com/postback?type={postback_type}&price={payout}
```

可回传的事件有:
- install: 安装事件
- reject: 拒绝的安装, 通常是mmp反作弊导致
- event: 安装后事件,例如注册、付费等. 此事件回传时会带有event_name参数来区分不同的事件
- convertion: 结算事件. StrideMobi和渠道协定的计费事件,即get_offer获取到offer的billing_event. 此事件回传时会带有payout参数来表达结算金额.

**注意：** 
* convertion必填, 渠道可通过该回传统计和StrideMobi之间的交易金额.
* 为降低交互次数install、event回传中不包含被convertion覆盖的事件. 例如CPI offer install回传不会被触发, CPE offer event回传中不会包含event_name=billing_event的事件.

宏参数如下：
| 参数名 | 类型 | 说明 |
|:------|:------|:------|
| postback_type | string | 取值为install、reject、event |
| channel_id | int | 渠道的唯一ID |
| site_id | string | 对应点击上报时的值 |
| offer_id | int | 对应点击上报时的值 |
| click_id | string | 点击id,可供渠道自己进行追踪和归因 |
| reject_reason | string | 拒绝原因, 仅postback_type为reject时返回 |
| event_name | string | 事件名称, 仅postback_type为event时返回 |
| event_value | string | 事件值, 仅postback_type为event时返回 |
| payout | number | 结算金额 |
| passthrough | string | 对应点击上报时的值 |

# 附录
## sign生成逻辑
sign的生成逻辑为
1. 将channel_id、timestamp、token拼接成一个字符串，例如：1131760170094token
2. 对拼接后的字符串进行sha256加密，得到sign

以下是个语言的签名样例
* PHP
```php
<?php
$channel_id = "channel_123";
$timestamp = "1620000000";
$token = "abc123";

// 拼接字符串
$data = $channel_id . $timestamp . $token;

// 计算SHA-256哈希
$hash = hash('sha256', $data);

echo "SHA-256 Hash: " . $hash;
?>
```

* Python
```python
import hashlib

channel_id = "channel_123"
timestamp = "1620000000"
token = "abc123"

# 拼接字符串
data = channel_id + timestamp + token

# 计算SHA-256哈希
hash_object = hashlib.sha256(data.encode('utf-8'))
hash_hex = hash_object.hexdigest()

print("SHA-256 Hash:", hash_hex)
```

* JS
```js
const crypto = require('crypto');

const channel_id = "channel_123";
const timestamp = "1620000000";
const token = "abc123";

// 拼接字符串
const data = channel_id + timestamp + token;

// 计算SHA-256哈希
const hash = crypto.createHash('sha256');
hash.update(data);
const hash_hex = hash.digest('hex');

console.log("SHA-256 Hash:", hash_hex);
```

* Golang
```golang
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"
)

func main() {
    channel_id := "channel_123"
    timestamp := "1620000000"
    token := "abc123"

    // 拼接字符串
    data := channel_id + timestamp + token

    // 计算SHA-256哈希
    hash := sha256.Sum256([]byte(data))
    hashString := hex.EncodeToString(hash[:])

    fmt.Println("SHA-256 Hash:", hashString)
}
```

## 枚举取值
### 广告状态 offer.status
| 状态码 | 状态描述 |
|:------|:------|
| 1 | 正常生效 |
| 20 | 人工暂停 |
| 27 | 到达各类Cap暂停生效，待第二天重启 |
| 其他 | 其他各类暂停 |

### 广告实体 offer.promotion_type
| 类型码 | 类型描述 |
|:------|:------|
| 1 | app |
| 2 | website |

### 设备类型 offer.target_device_types
| 类型码 | 类型描述 |
|:------|:------|
| 1 | 手机 |
| 2 | 平板 |

### 设备平台 offer.target_platform
| 类型码 | 类型描述 |
|:------|:------|
| 1 | 安卓 |
| 2 | 苹果 |

### 操作系统版本 offer.osv
对点分割的三位版本号转换为数字
转换方法为：将形如“主版本号.次版本号.修订号”的三段式字符串，按位补齐两位后依次拼接成 6 位数字。最终合并为整数。
例:
* "8.1.1"转换为80101
* "15.1.1"转换为150101
* "16.12.0"转换为161200

### 计费类型 offer.billing_type
| 类型码 | 类型描述 |
|:------|:------|
| 1 | CPI |
| 2 | CPE |

## impression/click返回结果说明
| code | msg | 含义 |
|------|-----|------|
| 1 | ok | 成功 |
| 1001 | param_invalid | 传参错误 |
| 1004 | internal_error | StrideMobi内部错误 |
| 2006 | offer_not_running | Offer未运行 |
| 3001 | country_target_mismatch | 国家定向不满足 |
| 3002 | city_target_mismatch | 城市定向不满足 |
| 3003 | device_type_target_mismatch | 设备类型定向不满足 |
| 3004 | device_platform_target_mismatch | 设备平台定向不满足 |
| 3005 | os_version_target_mismatch | 操作系统版本定向不满足 |

## 国家城市代码
国家城市代码和OpenRTB一样采用UNECE相关标准, 可以在<a href="https://unece.org/trade/cefact/unlocode-code-list-country-and-territory" style="color:blue">UNECE网站</a>查询. 
国家代码取2位大写字母,如US、CN等.
城市代码取NameWoDiacritics字段并转换为小写.如果您的流量来自于OpenRTB,将device.geo.city转换为小写即可.

以下是参考:
| 国家代码 | 国家名称 | 城市列表 |
|:--------|:--------|:--------|
| AL | 阿尔巴尼亚共和国 | 参考 [https://service.unece.org/trade/locode/al.htm](https://service.unece.org/trade/locode/al.htm) |
| DZ | 阿尔及利亚民主人民共和国 | 参考 [https://service.unece.org/trade/locode/dz.htm](https://service.unece.org/trade/locode/dz.htm) |
| AF | 阿富汗伊斯兰国 | 参考 [https://service.unece.org/trade/locode/af.htm](https://service.unece.org/trade/locode/af.htm) |
| AR | 阿根廷共和国 | 参考 [https://service.unece.org/trade/locode/ar.htm](https://service.unece.org/trade/locode/ar.htm) |
| AE | 阿拉伯联合酋长国 | 参考 [https://service.unece.org/trade/locode/ae.htm](https://service.unece.org/trade/locode/ae.htm) |
| AW | 阿鲁巴 | 参考 [https://service.unece.org/trade/locode/aw.htm](https://service.unece.org/trade/locode/aw.htm) |
| OM | 阿曼苏丹国 | 参考 [https://service.unece.org/trade/locode/om.htm](https://service.unece.org/trade/locode/om.htm) |
| AZ | 阿塞拜疆共和国 | 参考 [https://service.unece.org/trade/locode/az.htm](https://service.unece.org/trade/locode/az.htm) |
| EG | 阿拉伯埃及共和国 | 参考 [https://service.unece.org/trade/locode/eg.htm](https://service.unece.org/trade/locode/eg.htm) |
| ET | 埃塞俄比亚联邦民主共和国 | 参考 [https://service.unece.org/trade/locode/et.htm](https://service.unece.org/trade/locode/et.htm) |
| IE | 爱尔兰 | 参考 [https://service.unece.org/trade/locode/ie.htm](https://service.unece.org/trade/locode/ie.htm) |
| EE | 爱沙尼亚共和国 | 参考 [https://service.unece.org/trade/locode/ee.htm](https://service.unece.org/trade/locode/ee.htm) |
| AD | 安道尔公国 | 参考 [https://service.unece.org/trade/locode/ad.htm](https://service.unece.org/trade/locode/ad.htm) |
| AO | 安哥拉共和国 | 参考 [https://service.unece.org/trade/locode/ao.htm](https://service.unece.org/trade/locode/ao.htm) |
| AI | 安圭拉 | 参考 [https://service.unece.org/trade/locode/ai.htm](https://service.unece.org/trade/locode/ai.htm) |
| AG | 安提瓜和巴布达 | 参考 [https://service.unece.org/trade/locode/ag.htm](https://service.unece.org/trade/locode/ag.htm) |
| AT | 奥地利共和国 | 参考 [https://service.unece.org/trade/locode/at.htm](https://service.unece.org/trade/locode/at.htm) |
| AU | 澳大利亚联邦 | 参考 [https://service.unece.org/trade/locode/au.htm](https://service.unece.org/trade/locode/au.htm) |
| CN | 中华人民共和国 | 参考 [https://service.unece.org/trade/locode/cn.htm](https://service.unece.org/trade/locode/cn.htm) |
| BB | 巴巴多斯 | 参考 [https://service.unece.org/trade/locode/bb.htm](https://service.unece.org/trade/locode/bb.htm) |
| PG | 巴布亚新几内亚独立国 | 参考 [https://service.unece.org/trade/locode/pg.htm](https://service.unece.org/trade/locode/pg.htm) |
| BS | 巴哈马联邦 | 参考 [https://service.unece.org/trade/locode/bs.htm](https://service.unece.org/trade/locode/bs.htm) |
| PK | 巴基斯坦伊斯兰共和国 | 参考 [https://service.unece.org/trade/locode/pk.htm](https://service.unece.org/trade/locode/pk.htm) |
| PY | 巴拉圭共和国 | 参考 [https://service.unece.org/trade/locode/py.htm](https://service.unece.org/trade/locode/py.htm) |
| PS | 巴勒斯坦国 | 参考 [https://service.unece.org/trade/locode/ps.htm](https://service.unece.org/trade/locode/ps.htm) |
| BH | 巴林国 | 参考 [https://service.unece.org/trade/locode/bh.htm](https://service.unece.org/trade/locode/bh.htm) |
| PA | 巴拿马共和国 | 参考 [https://service.unece.org/trade/locode/pa.htm](https://service.unece.org/trade/locode/pa.htm) |
| BR | 巴西联邦共和国 | 参考 [https://service.unece.org/trade/locode/br.htm](https://service.unece.org/trade/locode/br.htm) |
| BY | 白俄罗斯共和国 | 参考 [https://service.unece.org/trade/locode/by.htm](https://service.unece.org/trade/locode/by.htm) |
| BM | 百慕大群岛 | 参考 [https://service.unece.org/trade/locode/bm.htm](https://service.unece.org/trade/locode/bm.htm) |
| BG | 保加利亚共和国 | 参考 [https://service.unece.org/trade/locode/bg.htm](https://service.unece.org/trade/locode/bg.htm) |
| MP | 北马里亚纳自由联邦 | 参考 [https://service.unece.org/trade/locode/mp.htm](https://service.unece.org/trade/locode/mp.htm) |
| BJ | 贝宁共和国 | 参考 [https://service.unece.org/trade/locode/bj.htm](https://service.unece.org/trade/locode/bj.htm) |
| BE | 比利时王国 | 参考 [https://service.unece.org/trade/locode/be.htm](https://service.unece.org/trade/locode/be.htm) |
| IS | 冰岛共和国 | 参考 [https://service.unece.org/trade/locode/is.htm](https://service.unece.org/trade/locode/is.htm) |
| PR | 波多黎各 | 参考 [https://service.unece.org/trade/locode/pr.htm](https://service.unece.org/trade/locode/pr.htm) |
| BA | 波斯尼亚和黑塞哥维那共和国 | 参考 [https://service.unece.org/trade/locode/ba.htm](https://service.unece.org/trade/locode/ba.htm) |
| PL | 波兰共和国 | 参考 [https://service.unece.org/trade/locode/pl.htm](https://service.unece.org/trade/locode/pl.htm) |
| BO | 玻利维亚共和国 | 参考 [https://service.unece.org/trade/locode/bo.htm](https://service.unece.org/trade/locode/bo.htm) |
| BZ | 伯利兹 | 参考 [https://service.unece.org/trade/locode/bz.htm](https://service.unece.org/trade/locode/bz.htm) |
| BW | 博茨瓦纳共和国 | 参考 [https://service.unece.org/trade/locode/bw.htm](https://service.unece.org/trade/locode/bw.htm) |
| BT | 不丹王国 | 参考 [https://service.unece.org/trade/locode/bt.htm](https://service.unece.org/trade/locode/bt.htm) |
| BF | 布基纳法索 | 参考 [https://service.unece.org/trade/locode/bf.htm](https://service.unece.org/trade/locode/bf.htm) |
| BI | 布隆迪共和国 | 参考 [https://service.unece.org/trade/locode/bi.htm](https://service.unece.org/trade/locode/bi.htm) |
| BV | 布维岛 | 参考 [https://service.unece.org/trade/locode/bv.htm](https://service.unece.org/trade/locode/bv.htm) |
| KP | 朝鲜民主主义人民共和国 | 参考 [https://service.unece.org/trade/locode/kp.htm](https://service.unece.org/trade/locode/kp.htm) |
| GQ | 赤道几内亚共和国 | 参考 [https://service.unece.org/trade/locode/gq.htm](https://service.unece.org/trade/locode/gq.htm) |
| DK | 丹麦王国 | 参考 [https://service.unece.org/trade/locode/dk.htm](https://service.unece.org/trade/locode/dk.htm) |
| DE | 德意志联邦共和国 | 参考 [https://service.unece.org/trade/locode/de.htm](https://service.unece.org/trade/locode/de.htm) |
| TP | 东帝汶 | 参考 [https://service.unece.org/trade/locode/tp.htm](https://service.unece.org/trade/locode/tp.htm) |
| TG | 多哥共和国 | 参考 [https://service.unece.org/trade/locode/tg.htm](https://service.unece.org/trade/locode/tg.htm) |
| DO | 多米尼加共和国 | 参考 [https://service.unece.org/trade/locode/do.htm](https://service.unece.org/trade/locode/do.htm) |
| DM | 多米尼克联邦 | 参考 [https://service.unece.org/trade/locode/dm.htm](https://service.unece.org/trade/locode/dm.htm) |
| RU | 俄罗斯联邦 | 参考 [https://service.unece.org/trade/locode/ru.htm](https://service.unece.org/trade/locode/ru.htm) |
| EC | 厄瓜多尔共和国 | 参考 [https://service.unece.org/trade/locode/ec.htm](https://service.unece.org/trade/locode/ec.htm) |
| ER | 厄立特里亚国 | 参考 [https://service.unece.org/trade/locode/er.htm](https://service.unece.org/trade/locode/er.htm) |
| FR | 法兰西共和国 | 参考 [https://service.unece.org/trade/locode/fr.htm](https://service.unece.org/trade/locode/fr.htm) |
| FO | 法罗群岛 | 参考 [https://service.unece.org/trade/locode/fo.htm](https://service.unece.org/trade/locode/fo.htm) |
| PF | 法属波利尼西亚 | 参考 [https://service.unece.org/trade/locode/pf.htm](https://service.unece.org/trade/locode/pf.htm) |
| GF | 法属圭亚那 | 参考 [https://service.unece.org/trade/locode/gf.htm](https://service.unece.org/trade/locode/gf.htm) |
| TF | 法属南部领土 | 参考 [https://service.unece.org/trade/locode/tf.htm](https://service.unece.org/trade/locode/tf.htm) |
| VA | 梵蒂冈城国 | 参考 [https://service.unece.org/trade/locode/va.htm](https://service.unece.org/trade/locode/va.htm) |
| PH | 菲律宾共和国 | 参考 [https://service.unece.org/trade/locode/ph.htm](https://service.unece.org/trade/locode/ph.htm) |
| FJ | 斐济群岛共和国 | 参考 [https://service.unece.org/trade/locode/fj.htm](https://service.unece.org/trade/locode/fj.htm) |
| FI | 芬兰共和国 | 参考 [https://service.unece.org/trade/locode/fi.htm](https://service.unece.org/trade/locode/fi.htm) |
| CV | 佛得角共和国 | 参考 [https://service.unece.org/trade/locode/cv.htm](https://service.unece.org/trade/locode/cv.htm) |
| FK | 福克兰群岛 | 参考 [https://service.unece.org/trade/locode/fk.htm](https://service.unece.org/trade/locode/fk.htm) |
| GM | 冈比亚共和国 | 参考 [https://service.unece.org/trade/locode/gm.htm](https://service.unece.org/trade/locode/gm.htm) |
| CG | 刚果共和国 | 参考 [https://service.unece.org/trade/locode/cg.htm](https://service.unece.org/trade/locode/cg.htm) |
| CD | 刚果民主共和国 | 参考 [https://service.unece.org/trade/locode/cd.htm](https://service.unece.org/trade/locode/cd.htm) |
| CO | 哥伦比亚共和国 | 参考 [https://service.unece.org/trade/locode/co.htm](https://service.unece.org/trade/locode/co.htm) |
| CR | 哥斯达黎加共和国 | 参考 [https://service.unece.org/trade/locode/cr.htm](https://service.unece.org/trade/locode/cr.htm) |
| GD | 格林纳达 | 参考 [https://service.unece.org/trade/locode/gd.htm](https://service.unece.org/trade/locode/gd.htm) |
| GL | 格陵兰 | 参考 [https://service.unece.org/trade/locode/gl.htm](https://service.unece.org/trade/locode/gl.htm) |
| GE | 格鲁吉亚 | 参考 [https://service.unece.org/trade/locode/ge.htm](https://service.unece.org/trade/locode/ge.htm) |
| CU | 古巴共和国 | 参考 [https://service.unece.org/trade/locode/cu.htm](https://service.unece.org/trade/locode/cu.htm) |
| GP | 瓜德罗普 | 参考 [https://service.unece.org/trade/locode/gp.htm](https://service.unece.org/trade/locode/gp.htm) |
| GU | 关岛 | 参考 [https://service.unece.org/trade/locode/gu.htm](https://service.unece.org/trade/locode/gu.htm) |
| GY | 圭亚那合作共和国 | 参考 [https://service.unece.org/trade/locode/gy.htm](https://service.unece.org/trade/locode/gy.htm) |
| KZ | 哈萨克斯坦共和国 | 参考 [https://service.unece.org/trade/locode/kz.htm](https://service.unece.org/trade/locode/kz.htm) |
| HT | 海地共和国 | 参考 [https://service.unece.org/trade/locode/ht.htm](https://service.unece.org/trade/locode/ht.htm) |
| KR | 大韩民国 | 参考 [https://service.unece.org/trade/locode/kr.htm](https://service.unece.org/trade/locode/kr.htm) |
| NL | 荷兰王国 | 参考 [https://service.unece.org/trade/locode/nl.htm](https://service.unece.org/trade/locode/nl.htm) |
| AN | 荷属安的列斯 | 参考 [https://service.unece.org/trade/locode/an.htm](https://service.unece.org/trade/locode/an.htm) |
| HM | 赫德岛和麦克唐纳岛 | 参考 [https://service.unece.org/trade/locode/hm.htm](https://service.unece.org/trade/locode/hm.htm) |
| HN | 洪都拉斯共和国 | 参考 [https://service.unece.org/trade/locode/hn.htm](https://service.unece.org/trade/locode/hn.htm) |
| KI | 基里巴斯共和国 | 参考 [https://service.unece.org/trade/locode/ki.htm](https://service.unece.org/trade/locode/ki.htm) |
| DJ | 吉布提共和国 | 参考 [https://service.unece.org/trade/locode/dj.htm](https://service.unece.org/trade/locode/dj.htm) |
| KG | 吉尔吉斯共和国 | 参考 [https://service.unece.org/trade/locode/kg.htm](https://service.unece.org/trade/locode/kg.htm) |
| GN | 几内亚共和国 | 参考 [https://service.unece.org/trade/locode/gn.htm](https://service.unece.org/trade/locode/gn.htm) |
| GW | 几内亚比绍共和国 | 参考 [https://service.unece.org/trade/locode/gw.htm](https://service.unece.org/trade/locode/gw.htm) |
| CA | 加拿大 | 参考 [https://service.unece.org/trade/locode/ca.htm](https://service.unece.org/trade/locode/ca.htm) |
| GH | 加纳共和国 | 参考 [https://service.unece.org/trade/locode/gh.htm](https://service.unece.org/trade/locode/gh.htm) |
| GA | 加蓬共和国 | 参考 [https://service.unece.org/trade/locode/ga.htm](https://service.unece.org/trade/locode/ga.htm) |
| KH | 柬埔寨王国 | 参考 [https://service.unece.org/trade/locode/kh.htm](https://service.unece.org/trade/locode/kh.htm) |
| CZ | 捷克共和国 | 参考 [https://service.unece.org/trade/locode/cz.htm](https://service.unece.org/trade/locode/cz.htm) |
| ZW | 津巴布韦共和国 | 参考 [https://service.unece.org/trade/locode/zw.htm](https://service.unece.org/trade/locode/zw.htm) |
| CM | 喀麦隆共和国 | 参考 [https://service.unece.org/trade/locode/cm.htm](https://service.unece.org/trade/locode/cm.htm) |
| QA | 卡塔尔国 | 参考 [https://service.unece.org/trade/locode/qa.htm](https://service.unece.org/trade/locode/qa.htm) |
| KY | 开曼群岛 | 参考 [https://service.unece.org/trade/locode/ky.htm](https://service.unece.org/trade/locode/ky.htm) |
| CC | 科科斯 | 参考 [https://service.unece.org/trade/locode/cc.htm](https://service.unece.org/trade/locode/cc.htm) |
| KM | 科摩罗伊斯兰联邦共和国 | 参考 [https://service.unece.org/trade/locode/km.htm](https://service.unece.org/trade/locode/km.htm) |
| CI | 科特迪瓦共和国 | 参考 [https://service.unece.org/trade/locode/ci.htm](https://service.unece.org/trade/locode/ci.htm) |
| KW | 科威特国 | 参考 [https://service.unece.org/trade/locode/kw.htm](https://service.unece.org/trade/locode/kw.htm) |
| HR | 克罗地亚共和国 | 参考 [https://service.unece.org/trade/locode/hr.htm](https://service.unece.org/trade/locode/hr.htm) |
| KE | 肯尼亚共和国 | 参考 [https://service.unece.org/trade/locode/ke.htm](https://service.unece.org/trade/locode/ke.htm) |
| CK | 库克群岛 | 参考 [https://service.unece.org/trade/locode/ck.htm](https://service.unece.org/trade/locode/ck.htm) |
| LV | 拉脱维亚共和国 | 参考 [https://service.unece.org/trade/locode/lv.htm](https://service.unece.org/trade/locode/lv.htm) |
| LS | 莱索托王国 | 参考 [https://service.unece.org/trade/locode/ls.htm](https://service.unece.org/trade/locode/ls.htm) |
| LA | 老挝人民民主共和国 | 参考 [https://service.unece.org/trade/locode/la.htm](https://service.unece.org/trade/locode/la.htm) |
| LB | 黎巴嫩共和国 | 参考 [https://service.unece.org/trade/locode/lb.htm](https://service.unece.org/trade/locode/lb.htm) |
| LT | 立陶宛共和国 | 参考 [https://service.unece.org/trade/locode/lt.htm](https://service.unece.org/trade/locode/lt.htm) |
| LR | 利比里亚共和国 | 参考 [https://service.unece.org/trade/locode/lr.htm](https://service.unece.org/trade/locode/lr.htm) |
| LY | 大阿拉伯利比亚人民社会主义民众国 | 参考 [https://service.unece.org/trade/locode/ly.htm](https://service.unece.org/trade/locode/ly.htm) |
| LI | 列支敦士登公国 | 参考 [https://service.unece.org/trade/locode/li.htm](https://service.unece.org/trade/locode/li.htm) |
| RE | 留尼汪 | 参考 [https://service.unece.org/trade/locode/re.htm](https://service.unece.org/trade/locode/re.htm) |
| LU | 卢森堡大公国 | 参考 [https://service.unece.org/trade/locode/lu.htm](https://service.unece.org/trade/locode/lu.htm) |
| RW | 卢旺达共和国 | 参考 [https://service.unece.org/trade/locode/rw.htm](https://service.unece.org/trade/locode/rw.htm) |
| RO | 罗马尼亚 | 参考 [https://service.unece.org/trade/locode/ro.htm](https://service.unece.org/trade/locode/ro.htm) |
| MG | 马达加斯加共和国 | 参考 [https://service.unece.org/trade/locode/mg.htm](https://service.unece.org/trade/locode/mg.htm) |
| MV | 马尔代夫共和国 | 参考 [https://service.unece.org/trade/locode/mv.htm](https://service.unece.org/trade/locode/mv.htm) |
| MT | 马耳他共和国 | 参考 [https://service.unece.org/trade/locode/mt.htm](https://service.unece.org/trade/locode/mt.htm) |
| MW | 马拉维共和国 | 参考 [https://service.unece.org/trade/locode/mw.htm](https://service.unece.org/trade/locode/mw.htm) |
| MY | 马来西亚 | 参考 [https://service.unece.org/trade/locode/my.htm](https://service.unece.org/trade/locode/my.htm) |
| ML | 马里共和国 | 参考 [https://service.unece.org/trade/locode/ml.htm](https://service.unece.org/trade/locode/ml.htm) |
| MH | 马绍尔群岛共和国 | 参考 [https://service.unece.org/trade/locode/mh.htm](https://service.unece.org/trade/locode/mh.htm) |
| MQ | 马提尼克 | 参考 [https://service.unece.org/trade/locode/mq.htm](https://service.unece.org/trade/locode/mq.htm) |
| YT | 马约特 | 参考 [https://service.unece.org/trade/locode/yt.htm](https://service.unece.org/trade/locode/yt.htm) |
| MU | 毛里求斯共和国 | 参考 [https://service.unece.org/trade/locode/mu.htm](https://service.unece.org/trade/locode/mu.htm) |
| MR | 毛里塔尼亚伊斯兰共和国 | 参考 [https://service.unece.org/trade/locode/mr.htm](https://service.unece.org/trade/locode/mr.htm) |
| US | 美利坚合众国 | 参考 [https://service.unece.org/trade/locode/us.htm](https://service.unece.org/trade/locode/us.htm) |
| UM | 美国本土外小岛屿 | 参考 [https://service.unece.org/trade/locode/um.htm](https://service.unece.org/trade/locode/um.htm) |
| AS | 美属萨摩亚 | 参考 [https://service.unece.org/trade/locode/as.htm](https://service.unece.org/trade/locode/as.htm) |
| VI | 美属维尔京群岛 | 参考 [https://service.unece.org/trade/locode/vi.htm](https://service.unece.org/trade/locode/vi.htm) |
| MN | 蒙古国 | 参考 [https://service.unece.org/trade/locode/mn.htm](https://service.unece.org/trade/locode/mn.htm) |
| MS | 蒙特塞拉特 | 参考 [https://service.unece.org/trade/locode/ms.htm](https://service.unece.org/trade/locode/ms.htm) |
| BD | 孟加拉人民共和国 | 参考 [https://service.unece.org/trade/locode/bd.htm](https://service.unece.org/trade/locode/bd.htm) |
| PE | 秘鲁共和国 | 参考 [https://service.unece.org/trade/locode/pe.htm](https://service.unece.org/trade/locode/pe.htm) |
| FM | 密克罗尼西亚联邦 | 参考 [https://service.unece.org/trade/locode/fm.htm](https://service.unece.org/trade/locode/fm.htm) |
| MM | 缅甸联邦 | 参考 [https://service.unece.org/trade/locode/mm.htm](https://service.unece.org/trade/locode/mm.htm) |
| MD | 摩尔多瓦共和国 | 参考 [https://service.unece.org/trade/locode/md.htm](https://service.unece.org/trade/locode/md.htm) |
| MA | 摩洛哥王国 | 参考 [https://service.unece.org/trade/locode/ma.htm](https://service.unece.org/trade/locode/ma.htm) |
| MC | 摩纳哥公国 | 参考 [https://service.unece.org/trade/locode/mc.htm](https://service.unece.org/trade/locode/mc.htm) |
| MZ | 莫桑比克共和国 | 参考 [https://service.unece.org/trade/locode/mz.htm](https://service.unece.org/trade/locode/mz.htm) |
| MX | 墨西哥合众国 | 参考 [https://service.unece.org/trade/locode/mx.htm](https://service.unece.org/trade/locode/mx.htm) |
| NA | 纳米尼亚 | 参考 [https://service.unece.org/trade/locode/na.htm](https://service.unece.org/trade/locode/na.htm) |
| ZA | 南非共和国 | 参考 [https://service.unece.org/trade/locode/za.htm](https://service.unece.org/trade/locode/za.htm) |
| AQ | 南极洲 | 参考 [https://service.unece.org/trade/locode/aq.htm](https://service.unece.org/trade/locode/aq.htm) |
| GS | 南乔治亚岛和南桑德韦奇岛 | 参考 [https://service.unece.org/trade/locode/gs.htm](https://service.unece.org/trade/locode/gs.htm) |
| YU | 南斯拉夫联盟共和国 | 参考 [https://service.unece.org/trade/locode/yu.htm](https://service.unece.org/trade/locode/yu.htm) |
| NR | 瑙鲁共和国 | 参考 [https://service.unece.org/trade/locode/nr.htm](https://service.unece.org/trade/locode/nr.htm) |
| NP | 尼泊尔王国 | 参考 [https://service.unece.org/trade/locode/np.htm](https://service.unece.org/trade/locode/np.htm) |
| NI | 尼加拉瓜共和国 | 参考 [https://service.unece.org/trade/locode/ni.htm](https://service.unece.org/trade/locode/ni.htm) |
| NE | 尼日尔共和国 | 参考 [https://service.unece.org/trade/locode/ne.htm](https://service.unece.org/trade/locode/ne.htm) |
| NG | 尼日利亚联邦共和国 | 参考 [https://service.unece.org/trade/locode/ng.htm](https://service.unece.org/trade/locode/ng.htm) |
| NU | 纽埃 | 参考 [https://service.unece.org/trade/locode/nu.htm](https://service.unece.org/trade/locode/nu.htm) |
| NO | 挪威王国 | 参考 [https://service.unece.org/trade/locode/no.htm](https://service.unece.org/trade/locode/no.htm) |
| NF | 诺福克岛 | 参考 [https://service.unece.org/trade/locode/nf.htm](https://service.unece.org/trade/locode/nf.htm) |
| PW | 帕劳共和国 | 参考 [https://service.unece.org/trade/locode/pw.htm](https://service.unece.org/trade/locode/pw.htm) |
| PN | 皮特凯恩 | 参考 [https://service.unece.org/trade/locode/pn.htm](https://service.unece.org/trade/locode/pn.htm) |
| PT | 葡萄牙共和国 | 参考 [https://service.unece.org/trade/locode/pt.htm](https://service.unece.org/trade/locode/pt.htm) |
| MK | 前南斯拉夫马其顿共和国 | 参考 [https://service.unece.org/trade/locode/mk.htm](https://service.unece.org/trade/locode/mk.htm) |
| JP | 日本国 | 参考 [https://service.unece.org/trade/locode/jp.htm](https://service.unece.org/trade/locode/jp.htm) |
| SE | 瑞典王国 | 参考 [https://service.unece.org/trade/locode/se.htm](https://service.unece.org/trade/locode/se.htm) |
| CH | 瑞士联邦 | 参考 [https://service.unece.org/trade/locode/ch.htm](https://service.unece.org/trade/locode/ch.htm) |
| SV | 萨尔瓦多共和国 | 参考 [https://service.unece.org/trade/locode/sv.htm](https://service.unece.org/trade/locode/sv.htm) |
| WS | 萨摩亚独立国 | 参考 [https://service.unece.org/trade/locode/ws.htm](https://service.unece.org/trade/locode/ws.htm) |
| SL | 塞拉利昂共和国 | 参考 [https://service.unece.org/trade/locode/sl.htm](https://service.unece.org/trade/locode/sl.htm) |
| SN | 塞内加尔共和国 | 参考 [https://service.unece.org/trade/locode/sn.htm](https://service.unece.org/trade/locode/sn.htm) |
| CY | 塞浦路斯共和国 | 参考 [https://service.unece.org/trade/locode/cy.htm](https://service.unece.org/trade/locode/cy.htm) |
| SC | 塞舌尔共和国 | 参考 [https://service.unece.org/trade/locode/sc.htm](https://service.unece.org/trade/locode/sc.htm) |
| SA | 沙特阿拉伯王国 | 参考 [https://service.unece.org/trade/locode/sa.htm](https://service.unece.org/trade/locode/sa.htm) |
| CX | 圣诞岛 | 参考 [https://service.unece.org/trade/locode/cx.htm](https://service.unece.org/trade/locode/cx.htm) |
| ST | 圣多美和普林西比民主共和国 | 参考 [https://service.unece.org/trade/locode/st.htm](https://service.unece.org/trade/locode/st.htm) |
| SH | 圣赫勒拿 | 参考 [https://service.unece.org/trade/locode/sh.htm](https://service.unece.org/trade/locode/sh.htm) |
| KN | 圣基茨和尼维斯联邦 | 参考 [https://service.unece.org/trade/locode/kn.htm](https://service.unece.org/trade/locode/kn.htm) |
| LC | 圣卢西亚 | 参考 [https://service.unece.org/trade/locode/lc.htm](https://service.unece.org/trade/locode/lc.htm) |
| SM | 圣马力诺共和国 | 参考 [https://service.unece.org/trade/locode/sm.htm](https://service.unece.org/trade/locode/sm.htm) |
| PM | 圣皮埃尔和密克隆 | 参考 [https://service.unece.org/trade/locode/pm.htm](https://service.unece.org/trade/locode/pm.htm) |
| VC | 圣文森特和格林纳丁斯 | 参考 [https://service.unece.org/trade/locode/vc.htm](https://service.unece.org/trade/locode/vc.htm) |
| LK | 斯里兰卡民主社会主义共和国 | 参考 [https://service.unece.org/trade/locode/lk.htm](https://service.unece.org/trade/locode/lk.htm) |
| SK | 斯洛伐克共和国 | 参考 [https://service.unece.org/trade/locode/sk.htm](https://service.unece.org/trade/locode/sk.htm) |
| SI | 斯洛文尼亚共和国 | 参考 [https://service.unece.org/trade/locode/si.htm](https://service.unece.org/trade/locode/si.htm) |
| SJ | 斯瓦尔巴岛和扬马延岛 | 参考 [https://service.unece.org/trade/locode/sj.htm](https://service.unece.org/trade/locode/sj.htm) |
| SZ | 斯威士兰王国 | 参考 [https://service.unece.org/trade/locode/sz.htm](https://service.unece.org/trade/locode/sz.htm) |
| SD | 苏丹共和国 | 参考 [https://service.unece.org/trade/locode/sd.htm](https://service.unece.org/trade/locode/sd.htm) |
| SR | 苏里南共和国 | 参考 [https://service.unece.org/trade/locode/sr.htm](https://service.unece.org/trade/locode/sr.htm) |
| SB | 所罗门群岛 | 参考 [https://service.unece.org/trade/locode/sb.htm](https://service.unece.org/trade/locode/sb.htm) |
| SO | 索马里共和国 | 参考 [https://service.unece.org/trade/locode/so.htm](https://service.unece.org/trade/locode/so.htm) |
| TJ | 塔吉克斯坦共和国 | 参考 [https://service.unece.org/trade/locode/tj.htm](https://service.unece.org/trade/locode/tj.htm) |
| TW | 中国台湾 | 参考 [https://service.unece.org/trade/locode/tw.htm](https://service.unece.org/trade/locode/tw.htm) |
| TH | 泰王国 | 参考 [https://service.unece.org/trade/locode/th.htm](https://service.unece.org/trade/locode/th.htm) |
| TZ | 坦桑尼亚联合共和国 | 参考 [https://service.unece.org/trade/locode/tz.htm](https://service.unece.org/trade/locode/tz.htm) |
| TO | 汤加王国 | 参考 [https://service.unece.org/trade/locode/to.htm](https://service.unece.org/trade/locode/to.htm) |
| TC | 特克斯和凯科斯群岛 | 参考 [https://service.unece.org/trade/locode/tc.htm](https://service.unece.org/trade/locode/tc.htm) |
| TT | 特立尼达和多巴哥共和国 | 参考 [https://service.unece.org/trade/locode/tt.htm](https://service.unece.org/trade/locode/tt.htm) |
| TN | 突尼斯共和国 | 参考 [https://service.unece.org/trade/locode/tn.htm](https://service.unece.org/trade/locode/tn.htm) |
| TV | 图瓦卢 | 参考 [https://service.unece.org/trade/locode/tv.htm](https://service.unece.org/trade/locode/tv.htm) |
| TR | 土耳其共和国 | 参考 [https://service.unece.org/trade/locode/tr.htm](https://service.unece.org/trade/locode/tr.htm) |
| TM | 土库曼斯坦 | 参考 [https://service.unece.org/trade/locode/tm.htm](https://service.unece.org/trade/locode/tm.htm) |
| TK | 托克劳 | 参考 [https://service.unece.org/trade/locode/tk.htm](https://service.unece.org/trade/locode/tk.htm) |
| WF | 瓦利斯和富图纳 | 参考 [https://service.unece.org/trade/locode/wf.htm](https://service.unece.org/trade/locode/wf.htm) |
| VU | 瓦努阿图共和国 | 参考 [https://service.unece.org/trade/locode/vu.htm](https://service.unece.org/trade/locode/vu.htm) |
| GT | 危地马拉共和国 | 参考 [https://service.unece.org/trade/locode/gt.htm](https://service.unece.org/trade/locode/gt.htm) |
| VE | 委内瑞拉共和国 | 参考 [https://service.unece.org/trade/locode/ve.htm](https://service.unece.org/trade/locode/ve.htm) |
| BN | 文莱达鲁萨兰国 | 参考 [https://service.unece.org/trade/locode/bn.htm](https://service.unece.org/trade/locode/bn.htm) |
| UG | 乌干达共和国 | 参考 [https://service.unece.org/trade/locode/ug.htm](https://service.unece.org/trade/locode/ug.htm) |
| UA | 乌克兰 | 参考 [https://service.unece.org/trade/locode/ua.htm](https://service.unece.org/trade/locode/ua.htm) |
| UY | 乌拉圭东岸共和国 | 参考 [https://service.unece.org/trade/locode/uy.htm](https://service.unece.org/trade/locode/uy.htm) |
| UZ | 乌兹别克斯坦 | 参考 [https://service.unece.org/trade/locode/uz.htm](https://service.unece.org/trade/locode/uz.htm) |
| ES | 西班牙王国 | 参考 [https://service.unece.org/trade/locode/es.htm](https://service.unece.org/trade/locode/es.htm) |
| EH | 西撒哈拉 | 参考 [https://service.unece.org/trade/locode/eh.htm](https://service.unece.org/trade/locode/eh.htm) |
| GR | 希腊共和国 | 参考 [https://service.unece.org/trade/locode/gr.htm](https://service.unece.org/trade/locode/gr.htm) |
| HK | 中国香港特别行政区 | 参考 [https://service.unece.org/trade/locode/hk.htm](https://service.unece.org/trade/locode/hk.htm) |
| SG | 新加坡共和国 | 参考 [https://service.unece.org/trade/locode/sg.htm](https://service.unece.org/trade/locode/sg.htm) |
| NC | 新喀里多尼亚 | 参考 [https://service.unece.org/trade/locode/nc.htm](https://service.unece.org/trade/locode/nc.htm) |
| NZ | 新西兰 | 参考 [https://service.unece.org/trade/locode/nz.htm](https://service.unece.org/trade/locode/nz.htm) |
| HU | 匈牙利共和国 | 参考 [https://service.unece.org/trade/locode/hu.htm](https://service.unece.org/trade/locode/hu.htm) |
| SY | 阿拉伯叙利亚共和国 | 参考 [https://service.unece.org/trade/locode/sy.htm](https://service.unece.org/trade/locode/sy.htm) |
| JM | 牙买加 | 参考 [https://service.unece.org/trade/locode/jm.htm](https://service.unece.org/trade/locode/jm.htm) |
| AM | 亚美尼亚共和国 | 参考 [https://service.unece.org/trade/locode/am.htm](https://service.unece.org/trade/locode/am.htm) |
| YE | 也门共和国 | 参考 [https://service.unece.org/trade/locode/ye.htm](https://service.unece.org/trade/locode/ye.htm) |
| IQ | 伊拉克共和国 | 参考 [https://service.unece.org/trade/locode/iq.htm](https://service.unece.org/trade/locode/iq.htm) |
| IR | 伊朗伊斯兰共和国 | 参考 [https://service.unece.org/trade/locode/ir.htm](https://service.unece.org/trade/locode/ir.htm) |
| IL | 以色列国 | 参考 [https://service.unece.org/trade/locode/il.htm](https://service.unece.org/trade/locode/il.htm) |
| IT | 意大利共和国 | 参考 [https://service.unece.org/trade/locode/it.htm](https://service.unece.org/trade/locode/it.htm) |
| IN | 印度共和国 | 参考 [https://service.unece.org/trade/locode/in.htm](https://service.unece.org/trade/locode/in.htm) |
| ID | 印度尼西亚共和国 | 参考 [https://service.unece.org/trade/locode/id.htm](https://service.unece.org/trade/locode/id.htm) |
| GB | 大不列颠及北爱尔兰联合王国 | 参考 [https://service.unece.org/trade/locode/gb.htm](https://service.unece.org/trade/locode/gb.htm) |
| VG | 英属维尔京群岛 | 参考 [https://service.unece.org/trade/locode/vg.htm](https://service.unece.org/trade/locode/vg.htm) |
| IO | 英属印度洋领地 | 参考 [https://service.unece.org/trade/locode/io.htm](https://service.unece.org/trade/locode/io.htm) |
| JO | 约旦哈希姆王国 | 参考 [https://service.unece.org/trade/locode/jo.htm](https://service.unece.org/trade/locode/jo.htm) |
| VN | 越南社会主义共和国 | 参考 [https://service.unece.org/trade/locode/vn.htm](https://service.unece.org/trade/locode/vn.htm) |
| ZM | 赞比亚共和国 | 参考 [https://service.unece.org/trade/locode/zm.htm](https://service.unece.org/trade/locode/zm.htm) |
| TD | 乍得共和国 | 参考 [https://service.unece.org/trade/locode/td.htm](https://service.unece.org/trade/locode/td.htm) |
| GI | 直布罗陀 | 参考 [https://service.unece.org/trade/locode/gi.htm](https://service.unece.org/trade/locode/gi.htm) |
| CL | 智利共和国 | 参考 [https://service.unece.org/trade/locode/cl.htm](https://service.unece.org/trade/locode/cl.htm) |
| CF | 中非共和国 | 参考 [https://service.unece.org/trade/locode/cf.htm](https://service.unece.org/trade/locode/cf.htm) |