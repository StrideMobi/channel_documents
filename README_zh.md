# 项目名称

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
                "target_cities":["{city}", "${city}"], 
                "target_device_types":[{device_type}, {device_type}], 
                "target_platform": [{target_platform}, {target_platform}], 
                "min_osv": {min_osv}, 
                "max_osv": {max_osv}, 
                "siteid_blacklist": ["${siteid}", "${siteid}"], 


                // 结算信息
                "billing_event": "${billing_event}",
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
| offer.status | 参考<a href="#广告状态-offer.status" style="color:blue">广告状态</a> | integer | 1 |
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
| offer.target_countrys | 目标国家列表 | array | ["US","CN"] |
| offer.target_cities | 目标城市列表 | array | ["SIN", "LON", "DFW"] |
| offer.target_device_types | 参考<a href="#设备类型-offertarget_device_types" style="color:blue">设备类型</a> | array | [1,2] |
| offer.target_platform | 参考<a href="#设备平台-offertarget_platform" style="color:blue">设备平台</a> | array | [1,2] |
| offer.min_osv | 参考<a href="#操作系统版本-offerosv" style="color:blue">操作系统版本</a> | integer | 80101 |
| offer.max_osv | 参考<a href="#操作系统版本-offerosv" style="color:blue">操作系统版本</a> | integer | 150101 |
| offer.siteid_blacklist | 媒体黑名单 | array | ["siteid1","siteid2"] |
| offer.billing_event | 计费事件类型 | string | install |
| offer.billing_type | 参考<a href="#计费类型-offerbilling_type" style="color:blue">计费类型</a> | integer | 1 |
| offer.billing_price | 计费价格 | number | 3.1 |
| offer.billing_event_cap | 日计费事件数预算 | integer | 10000 |
| offer.impression_cap | 日展示预算 | integer | 20000000 |
| offer.click_cap | 日点击预算 | integer | 20000000 |


# impression/click
渠道通过该接口上报用户点击或展示广告的事件。
每个offer上报地址不同, get_offer接口会返回具体地址。例：
```
http://www.sttt.com/click?ad_type={}&channel_id=3&city={}&click_id={}&country={}&creative_id={}&device_brand={}&device_model={}&device_os_version={}&device_platform={}&device_type={}&gaid={}&gaid_md5={}&gaid_sha1={}&ip={}&offer_id=4&site_id={}&user_agent={}
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
| creative_id | string | 尽量填写 | 创意ID |

**注意**：
- iOS时 idfa、idfv 以及对应Sha1、MD5值必填其一
- Android时 gaid 以及对应Sha1、MD5值必填其一
- click_id、site_id 均限制24个字符,超过会被截断
- City为IATA城市代码，如"SIN"、"LON"、"DFW"等,参考<a href="https://www.ccra.com/airport-codes" style="color:blue">https://www.ccra.com/airport-codes</a>

## 返回结果
impression/click接口会返回json格式的结果, 示例如下：
```json
{
    "code": 0,
    "msg": "success"
}
```

code、msg参考<a href="#impressionclick返回结果说明" style="color:blue">impression/click返回结果说明</a>

## Postback

StrideMobi检测相关事件后会给渠道回传。
可回传的事件有:
- install: 安装事件
- reject: 拒绝的安装, 通常是mmp反作弊导致
- event: 安装后事件,例如注册、付费等. 此事件回传时会带有event_name参数来区分不同的事件
- convertion: 转换事件. StrideMobi和渠道协定的计费事件,即get_offer获取到offer的billing_event. 此事件回传时会带有payout参数来表达结算金额.
回传地址由渠道对接时提供, install、reject、event、convertion可使用不同的地址。

回传参数如下：

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
| payout | number | 结算金额, 仅postback_type为convertion时返回 |


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

## 国家列表
| 国家代码 | 国家名称 |
|:--------|:--------|
| AL | 阿尔巴尼亚共和国 |
| DZ | 阿尔及利亚民主人民共和国 |
| AF | 阿富汗伊斯兰国 |
| AR | 阿根廷共和国 |
| AE | 阿拉伯联合酋长国 |
| AW | 阿鲁巴 |
| OM | 阿曼苏丹国 |
| AZ | 阿塞拜疆共和国 |
| EG | 阿拉伯埃及共和国 |
| ET | 埃塞俄比亚联邦民主共和国 |
| IE | 爱尔兰 |
| EE | 爱沙尼亚共和国 |
| AD | 安道尔公国 |
| AO | 安哥拉共和国 |
| AI | 安圭拉 |
| AG | 安提瓜和巴布达 |
| AT | 奥地利共和国 |
| AU | 澳大利亚联邦 |
| CN | 中华人民共和国 |
| BB | 巴巴多斯 |
| PG | 巴布亚新几内亚独立国 |
| BS | 巴哈马联邦 |
| PK | 巴基斯坦伊斯兰共和国 |
| PY | 巴拉圭共和国 |
| PS | 巴勒斯坦国 |
| BH | 巴林国 |
| PA | 巴拿马共和国 |
| BR | 巴西联邦共和国 |
| BY | 白俄罗斯共和国 |
| BM | 百慕大群岛 |
| BG | 保加利亚共和国 |
| MP | 北马里亚纳自由联邦 |
| BJ | 贝宁共和国 |
| BE | 比利时王国 |
| IS | 冰岛共和国 |
| PR | 波多黎各 |
| BA | 波斯尼亚和黑塞哥维那共和国 |
| PL | 波兰共和国 |
| BO | 玻利维亚共和国 |
| BZ | 伯利兹 |
| BW | 博茨瓦纳共和国 |
| BT | 不丹王国 |
| BF | 布基纳法索 |
| BI | 布隆迪共和国 |
| BV | 布维岛 |
| KP | 朝鲜民主主义人民共和国 |
| GQ | 赤道几内亚共和国 |
| DK | 丹麦王国 |
| DE | 德意志联邦共和国 |
| TP | 东帝汶 |
| TG | 多哥共和国 |
| DO | 多米尼加共和国 |
| DM | 多米尼克联邦 |
| RU | 俄罗斯联邦 |
| EC | 厄瓜多尔共和国 |
| ER | 厄立特里亚国 |
| FR | 法兰西共和国 |
| FO | 法罗群岛 |
| PF | 法属波利尼西亚 |
| GF | 法属圭亚那 |
| TF | 法属南部领土 |
| VA | 梵蒂冈城国 |
| PH | 菲律宾共和国 |
| FJ | 斐济群岛共和国 |
| FI | 芬兰共和国 |
| CV | 佛得角共和国 |
| FK | 福克兰群岛 |
| GM | 冈比亚共和国 |
| CG | 刚果共和国 |
| CD | 刚果民主共和国 |
| CO | 哥伦比亚共和国 |
| CR | 哥斯达黎加共和国 |
| GD | 格林纳达 |
| GL | 格陵兰 |
| GE | 格鲁吉亚 |
| CU | 古巴共和国 |
| GP | 瓜德罗普 |
| GU | 关岛 |
| GY | 圭亚那合作共和国 |
| KZ | 哈萨克斯坦共和国 |
| HT | 海地共和国 |
| KR | 大韩民国 |
| NL | 荷兰王国 |
| AN | 荷属安的列斯 |
| HM | 赫德岛和麦克唐纳岛 |
| HN | 洪都拉斯共和国 |
| KI | 基里巴斯共和国 |
| DJ | 吉布提共和国 |
| KG | 吉尔吉斯共和国 |
| GN | 几内亚共和国 |
| GW | 几内亚比绍共和国 |
| CA | 加拿大 |
| GH | 加纳共和国 |
| GA | 加蓬共和国 |
| KH | 柬埔寨王国 |
| CZ | 捷克共和国 |
| ZW | 津巴布韦共和国 |
| CM | 喀麦隆共和国 |
| QA | 卡塔尔国 |
| KY | 开曼群岛 |
| CC | 科科斯 |
| KM | 科摩罗伊斯兰联邦共和国 |
| CI | 科特迪瓦共和国 |
| KW | 科威特国 |
| HR | 克罗地亚共和国 |
| KE | 肯尼亚共和国 |
| CK | 库克群岛 |
| LV | 拉脱维亚共和国 |
| LS | 莱索托王国 |
| LA | 老挝人民民主共和国 |
| LB | 黎巴嫩共和国 |
| LT | 立陶宛共和国 |
| LR | 利比里亚共和国 |
| LY | 大阿拉伯利比亚人民社会主义民众国 |
| LI | 列支敦士登公国 |
| RE | 留尼汪 |
| LU | 卢森堡大公国 |
| RW | 卢旺达共和国 |
| RO | 罗马尼亚 |
| MG | 马达加斯加共和国 |
| MV | 马尔代夫共和国 |
| MT | 马耳他共和国 |
| MW | 马拉维共和国 |
| MY | 马来西亚 |
| ML | 马里共和国 |
| MH | 马绍尔群岛共和国 |
| MQ | 马提尼克 |
| YT | 马约特 |
| MU | 毛里求斯共和国 |
| MR | 毛里塔尼亚伊斯兰共和国 |
| US | 美利坚合众国 |
| UM | 美国本土外小岛屿 |
| AS | 美属萨摩亚 |
| VI | 美属维尔京群岛 |
| MN | 蒙古国 |
| MS | 蒙特塞拉特 |
| BD | 孟加拉人民共和国 |
| PE | 秘鲁共和国 |
| FM | 密克罗尼西亚联邦 |
| MM | 缅甸联邦 |
| MD | 摩尔多瓦共和国 |
| MA | 摩洛哥王国 |
| MC | 摩纳哥公国 |
| MZ | 莫桑比克共和国 |
| MX | 墨西哥合众国 |
| NA | 纳米尼亚 |
| ZA | 南非共和国 |
| AQ | 南极洲 |
| GS | 南乔治亚岛和南桑德韦奇岛 |
| YU | 南斯拉夫联盟共和国 |
| NR | 瑙鲁共和国 |
| NP | 尼泊尔王国 |
| NI | 尼加拉瓜共和国 |
| NE | 尼日尔共和国 |
| NG | 尼日利亚联邦共和国 |
| NU | 纽埃 |
| NO | 挪威王国 |
| NF | 诺福克岛 |
| PW | 帕劳共和国 |
| PN | 皮特凯恩 |
| PT | 葡萄牙共和国 |
| MK | 前南斯拉夫马其顿共和国 |
| JP | 日本国 |
| SE | 瑞典王国 |
| CH | 瑞士联邦 |
| SV | 萨尔瓦多共和国 |
| WS | 萨摩亚独立国 |
| SL | 塞拉利昂共和国 |
| SN | 塞内加尔共和国 |
| CY | 塞浦路斯共和国 |
| SC | 塞舌尔共和国 |
| SA | 沙特阿拉伯王国 |
| CX | 圣诞岛 |
| ST | 圣多美和普林西比民主共和国 |
| SH | 圣赫勒拿 |
| KN | 圣基茨和尼维斯联邦 |
| LC | 圣卢西亚 |
| SM | 圣马力诺共和国 |
| PM | 圣皮埃尔和密克隆 |
| VC | 圣文森特和格林纳丁斯 |
| LK | 斯里兰卡民主社会主义共和国 |
| SK | 斯洛伐克共和国 |
| SI | 斯洛文尼亚共和国 |
| SJ | 斯瓦尔巴岛和扬马延岛 |
| SZ | 斯威士兰王国 |
| SD | 苏丹共和国 |
| SR | 苏里南共和国 |
| SB | 所罗门群岛 |
| SO | 索马里共和国 |
| TJ | 塔吉克斯坦共和国 |
| TW | 中国台湾 |
| TH | 泰王国 |
| TZ | 坦桑尼亚联合共和国 |
| TO | 汤加王国 |
| TC | 特克斯和凯科斯群岛 |
| TT | 特立尼达和多巴哥共和国 |
| TN | 突尼斯共和国 |
| TV | 图瓦卢 |
| TR | 土耳其共和国 |
| TM | 土库曼斯坦 |
| TK | 托克劳 |
| WF | 瓦利斯和富图纳 |
| VU | 瓦努阿图共和国 |
| GT | 危地马拉共和国 |
| VE | 委内瑞拉共和国 |
| BN | 文莱达鲁萨兰国 |
| UG | 乌干达共和国 |
| UA | 乌克兰 |
| UY | 乌拉圭东岸共和国 |
| UZ | 乌兹别克斯坦 |
| ES | 西班牙王国 |
| EH | 西撒哈拉 |
| GR | 希腊共和国 |
| HK | 中国香港特别行政区 |
| SG | 新加坡共和国 |
| NC | 新喀里多尼亚 |
| NZ | 新西兰 |
| HU | 匈牙利共和国 |
| SY | 阿拉伯叙利亚共和国 |
| JM | 牙买加 |
| AM | 亚美尼亚共和国 |
| YE | 也门共和国 |
| IQ | 伊拉克共和国 |
| IR | 伊朗伊斯兰共和国 |
| IL | 以色列国 |
| IT | 意大利共和国 |
| IN | 印度共和国 |
| ID | 印度尼西亚共和国 |
| GB | 大不列颠及北爱尔兰联合王国 |
| VG | 英属维尔京群岛 |
| IO | 英属印度洋领地 |
| JO | 约旦哈希姆王国 |
| VN | 越南社会主义共和国 |
| ZM | 赞比亚共和国 |
| TD | 乍得共和国 |
| GI | 直布罗陀 |
| CL | 智利共和国 |
| CF | 中非共和国 |