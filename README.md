# Channel Integration Documentation

<a href="README_zh.md" style="color:blue">中文</a> | <a href="README.md" style="color:blue">English</a>


# Integration Overview
The complete process for channel partners to integrate with StrideMobi is as follows:
* Obtain channel ID, token, and relevant API addresses from your account manager, and provide postback receiving address to your account manager
* Get offer list, refer to <a href="#get_offer" style="color:blue">get_offer</a>
* Send impression/click, refer to <a href="#impressionclick" style="color:blue">impression/click</a>
* Receive postback, refer to <a href="#postback" style="color:blue">postback</a>


# get_offer
Channel partners can use this interface to pull available offer lists

## Request URL
```
https://api.stridemobi.com/api/v1/get_offer
```

## Request Method
GET

## Request Parameters
| Parameter | Type | Required | Description |
|:---------|:------|:---------|:------------|
| domain | string | Yes | Access domain name (fixed value, please contact the operations manager for the specific value) |
| api_version | string | Yes | API version, fixed value "v1" |
| channel_id | int | Yes | Channel unique identifier, can be obtained from the platform |
| timestamp | int | Yes | Unix timestamp (in seconds) |
| device_type | int | Yes | Device type, see [Device Type](#device-type-offertarget_device_types) |
| platform | int | Yes | Device platform, see [Device Platform](#device-platform-offertarget_platform) |
| country | string | Yes | Country code, uppercase 2 letters (e.g., US, CN), see [Country/City Codes](#country-city-codes) |
| city | string | No | City code, lowercase, see [Country/City Codes](#country-city-codes) |
| osv | int | No | Operating system version, see [Operating System Version](#operating-system-version-offerosv) |
| ip | string | No | User IP address |
| ua | string | No | User-Agent |
| limit | int | No | Number of offers to return per page, default 20, maximum 100 |
| page | int | No | Page number, default 1 |
| offer_ids | string | No | Comma-separated list of offer IDs to filter |
| sign | string | Yes | Signature, see [Sign Generation Logic](#sign-generation-logic) for generation method |

## Response Format
```json
{
    "code": 1,
    "msg": "success",
    "data": {
        "total": 100,
        "offers": [
            {
                "offer_id": "123",
                "offer_name": "Example App Install",
                "description": "Install and open the app",
                "preview_link": "https://example.com/app",
                "tracking_link": "https://tr.stridemobi.com/click?offer_id=123&aff_id=456&source={source}&sub1={sub1}&sub2={sub2}&sub3={sub3}&sub4={sub4}&sub5={sub5}",
                "payout": 1.5,
                "currency": "USD",
                "target": {
                    "countries": ["US", "CA", "UK"],
                    "device_types": [1],
                    "platforms": [1, 2]
                },
                "creative": "https://example.com/creative.jpg",
                "status": 1,
                "promotion_type": 1,
                "billing_type": 1,
                "daily_cap": 1000,
                "hourly_cap": 100,
                "remaining_daily_cap": 500,
                "remaining_hourly_cap": 50
            }
        ]
    }
}
```

## Response Parameters Description
| Parameter | Type | Description |
|:---------|:------|:------------|
| code | int | Status code, 1 for success |
| msg | string | Status message |
| data.total | int | Total number of offers |
| data.offers | array | Offer list |
| data.offers[].offer_id | string | Offer ID |
| data.offers[].offer_name | string | Offer name |
| data.offers[].description | string | Offer description |
| data.offers[].preview_link | string | Preview link |
| data.offers[].tracking_link | string | Tracking link, you need to replace the macros with actual values |
| data.offers[].payout | float | Payout amount |
| data.offers[].currency | string | Currency code |
| data.offers[].target.countries | array | Target countries |
| data.offers[].target.device_types | array | Target device types |
| data.offers[].target.platforms | array | Target platforms |
| data.offers[].creative | string | Creative URL |
| data.offers[].status | int | Offer status |
| data.offers[].promotion_type | int | Promotion type |
| data.offers[].billing_type | int | Billing type |
| data.offers[].daily_cap | int | Daily cap |
| data.offers[].hourly_cap | int | Hourly cap |
| data.offers[].remaining_daily_cap | int | Remaining daily cap |
| data.offers[].remaining_hourly_cap | int | Remaining hourly cap |

## Error Codes
| Error Code | Description |
|:-----------|:------------|
| 1001 | Invalid parameters |
| 1002 | API key or app ID not found |
| 1003 | No permission |
| 1004 | Internal server error |

# impression/click
Channels use this interface to report user ad click or impression events.
Each offer has a different reporting address, which will be returned by the get_offer interface. Example:
```
http://www.sttt.com/click?ad_type={}&channel_id=3&city={}&click_id={}&country={}&bundle={}&creative_id={}&device_brand={}&device_model={}&device_os_version={}&device_platform={}&device_type={}&lang={}&gaid={}&gaid_md5={}&gaid_sha1={}&ip={}&offer_id=4&site_id={}&user_agent={}&passthrough={}
```
Impression and click parameters are the same, only the path is different.

## Request Parameters Description
get_offer returns a link with some parameters already determined, while others are {}. Channels need to replace these with specific values when reporting.

| Parameter | Type | Filling Instructions | Description |
|:---------|:------|:------------------|:------------|
| offer_id | int | Has a determined value, no need for channel to fill | Unique identifier of the ad |
| channel_id | int | Has a determined value, no need for channel to fill | Unique identifier of the channel |
| click_id | string | Optional | Click ID, for channel's own tracking and attribution |
| site_id | string | Optional | Media site ID, for channel to distinguish different media sources |
| device_platform | int | Required | See [Device Platform](#device-platform-offertarget_platform) |
| device_type | int | Required | See [Device Type](#device-type-offertarget_device_types) |
| device_os_version | string | Required | Can pass original string like 16.3.1, or convert to [Operating System Version](#operating-system-version-offerosv) format |
| device_brand | string | Fill if possible | Device brand |
| device_model | string | Fill if possible | Device model |
| lang | string | Fill if possible | Device language |
| idfv | string | Fill if possible for iOS | iOS device IDFV identifier |
| idfv_sha1 | string | Fill if possible for iOS | SHA1 value of iOS device IDFV |
| idfv_md5 | string | Fill if possible for iOS | MD5 value of iOS device IDFV |
| idfa | string | Fill if possible for iOS | iOS device IDFA advertising identifier |
| idfa_sha1 | string | Fill if possible for iOS | SHA1 value of iOS device IDFA |
| idfa_md5 | string | Fill if possible for iOS | MD5 value of iOS device IDFA |
| gaid | string | Fill if possible for Android | Android device GAID advertising identifier |
| gaid_sha1 | string | Fill if possible for Android | SHA1 value of Android device GAID |
| gaid_md5 | string | Fill if possible for Android | MD5 value of Android device GAID |
| ad_type | string | Fill if possible | Ad type |
| country | string | Required when country targeting is set | User's country code, such as "US", "CN", etc. Refer to [Country/City Codes](#countrycity-codes) |
| city | string | Required when city targeting is set | User's city. Refer to [Country/City Codes](#countrycity-codes) |
| ip | string | Required | User IP address |
| user_agent | string | Required | User browser's User-Agent information |
| bundle | string | Fill if possible | Traffic source package name, e.g., com.zzkko, id128883 |
| creative_id | string | Fill if possible | Creative ID |
| passthrough | string | Optional | Channel custom parameters, will be passed back unchanged |

**Notes**:
- For iOS, at least one of idfa, idfv and their corresponding Sha1, MD5 values is required
- For Android, at least one of gaid and its corresponding Sha1, MD5 values is required
- click_id and site_id are limited to 24 characters, will be truncated if exceeded
- passthrough is limited to 200 characters, will be truncated if exceeded

## Return Result
The impression/click interface returns results in JSON format, example as follows:
```json
{
    "code": 0,
    "msg": "success"
}
```

Code and msg reference [impression/click Return Result Description](#impressionclick-return-result-description)

# postback

## Postback Mechanism
StrideMobi uses postback URLs to track conversions. When a user completes an action (e.g., installs an app), StrideMobi will send a request to your postback URL to notify you of the conversion.

## Setting Up Postback URL
You can set up your postback URL in the StrideMobi platform. The postback URL should contain macros that will be replaced with actual values when the postback is sent.

**Example Postback URL:**
```
https://yourserver.com/postback?offer_id={offer_id}&payout={payout}&source={source}&sub1={sub1}&sub2={sub2}&sub3={sub3}&sub4={sub4}&sub5={sub5}&click_id={click_id}
```

## Postback Parameters
| Parameter | Description |
|:---------|:------------|
| {postback_type} | Value can be install, reject, event |
| {channel_id} | Unique channel ID |
| {site_id} | Corresponding value from click reporting |
| {offer_id} | Corresponding value from click reporting |
| {click_id} | Click ID, available for channel's own tracking and attribution |
| {rejected_reason} | Rejection reason, returned only when postback_type is reject |
| {rejected_sub_reason} | Rejection sub-reason, returned only when postback_type is reject |
| {event_name} | Event name, returned only when postback_type is event |
| {event_value} | Event value, returned only when postback_type is event |
| {payout} | Settlement amount |
| {passthrough} | Corresponding value from click reporting |
| {source} | Traffic source identifier |
| {sub1} | Custom parameter 1 |
| {sub2} | Custom parameter 2 |
| {sub3} | Custom parameter 3 |
| {sub4} | Custom parameter 4 |
| {sub5} | Custom parameter 5 |
| {country} | Country code |
| {ip} | User IP address |

# Appendix
## sign Generation Logic

### Sign Generation Logic
To ensure API security, all requests need to include a sign parameter. The sign is generated using the following steps:

1. Sort all request parameters (excluding sign) in alphabetical order by parameter name
2. Concatenate the parameter name and value in the format of "name=value"
3. Concatenate all these strings with "&"
4. Append your API secret key to the end
5. Calculate the MD5 hash of the resulting string

**Example:**
If the request parameters are:
- api_key: abc123
- app_id: app456
- country: US
- device_type: 1
- platform: 1

And the API secret key is: secret789

The sign generation process would be:
1. Sort parameters: api_key, app_id, country, device_type, platform
2. Concatenate: "api_key=abc123&app_id=app456&country=US&device_type=1&platform=1"
3. Append secret key: "api_key=abc123&app_id=app456&country=US&device_type=1&platform=1secret789"
4. Calculate MD5: "e10adc3949ba59abbe56e057f20f883e"

### Code Example
**PHP:**
```php
function generate_sign($params, $api_secret) {
    ksort($params);
    $str = '';
    foreach ($params as $key => $value) {
        if ($key != 'sign') {
            $str .= $key . '=' . $value . '&';
        }
    }
    $str = rtrim($str, '&') . $api_secret;
    return md5($str);
}
```

**Python:**
```python
import hashlib
def generate_sign(params, api_secret):
    sorted_params = sorted(params.items())
    str_params = '&'.join([f"{k}={v}" for k, v in sorted_params if k != 'sign'])
    str_to_sign = str_params + api_secret
    return hashlib.md5(str_to_sign.encode()).hexdigest()
```

**Java:**
```java
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.*;

public class SignGenerator {
    public static String generateSign(Map<String, String> params, String apiSecret) {
        Map<String, String> sortedParams = new TreeMap<>(params);
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<String, String> entry : sortedParams.entrySet()) {
            if (!"sign".equals(entry.getKey())) {
                sb.append(entry.getKey()).append("=").append(entry.getValue()).append("&");
            }
        }
        if (sb.length() > 0) {
            sb.setLength(sb.length() - 1);
        }
        sb.append(apiSecret);
        return md5(sb.toString());
    }
    
    private static String md5(String input) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] messageDigest = md.digest(input.getBytes());
            BigInteger no = new BigInteger(1, messageDigest);
            String hashtext = no.toString(16);
            while (hashtext.length() < 32) {
                hashtext = "0" + hashtext;
            }
            return hashtext;
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## Enumeration Values

### 5.1 Offer Status Codes
| Status Code | Description |
|:------------|:------------|
| 1 | Active |
| 20 | Manually paused |
| 27 | Paused due to reaching cap, will restart the next day |
| Others | Various other paused statuses |

### 5.2 Promotion Type (offer.promotion_type)
| Type Code | Description |
|:----------|:------------|
| 1 | app |
| 2 | website |

### 5.3 Device Type (offer.target_device_types)
| Type Code | Description |
|:----------|:------------|
| 1 | Mobile |
| 2 | Tablet |

### 5.4 Platform (offer.target_platform)
| Type Code | Description |
|:----------|:------------|
| 1 | Android |
| 2 | iOS |

### 5.5 OS Version Conversion (offer.osv)
Convert a three-part version number separated by dots to a numeric value.
Conversion method: For a string in the format "major.minor.patch", pad each part to two digits and concatenate them to form a 6-digit number, then convert to an integer.

Examples:
* "8.1.1" converts to 80101
* "15.1.1" converts to 150101
* "16.12.0" converts to 161200

### 5.6 Billing Type (offer.billing_type)
| Type Code | Description |
|:----------|:------------|
| 1 | CPI |
| 2 | CPE |

## impression/click Response Codes
| code | msg | Meaning |
|------|-----|---------|
| 1 | ok | Success |
| 1001 | param_invalid | Invalid parameters |
| 1004 | internal_error | StrideMobi internal error |
| 2006 | offer_not_running | Offer not running |
| 3001 | country_target_mismatch | Country targeting not met |
| 3002 | city_target_mismatch | City targeting not met |
| 3003 | device_type_target_mismatch | Device type targeting not met |
| 3004 | device_platform_target_mismatch | Device platform targeting not met |
| 3005 | os_version_target_mismatch | OS version targeting not met |

## Country and City Codes
Country and city codes follow the same standards as OpenRTB, adopting UNECE standards. You can查询 them on the [UNECE website](https://unece.org/trade/cefact/unlocode-code-list-country-and-territory).
Country and city codes are based on UNECE standard.

* Country codes are 2 uppercase letters, such as US, CN, etc.
* City codes are taken from the NameWoDiacritics field and must be **converted to lowercase**. If your traffic comes from OpenRTB, simply convert device.geo.city to lowercase.

Below is a reference:
| Country Code | Country Name | City List |
|:-------------|:-------------|:----------|
| AL | Republic of Albania | Reference [https://service.unece.org/trade/locode/al.htm](https://service.unece.org/trade/locode/al.htm) |
| DZ | People's Democratic Republic of Algeria | Reference [https://service.unece.org/trade/locode/dz.htm](https://service.unece.org/trade/locode/dz.htm) |
| AF | Islamic State of Afghanistan | Reference [https://service.unece.org/trade/locode/af.htm](https://service.unece.org/trade/locode/af.htm) |
| AR | Argentine Republic | Reference [https://service.unece.org/trade/locode/ar.htm](https://service.unece.org/trade/locode/ar.htm) |
| AE | United Arab Emirates | Reference [https://service.unece.org/trade/locode/ae.htm](https://service.unece.org/trade/locode/ae.htm) |
| AW | Aruba | Reference [https://service.unece.org/trade/locode/aw.htm](https://service.unece.org/trade/locode/aw.htm) |
| OM | Sultanate of Oman | Reference [https://service.unece.org/trade/locode/om.htm](https://service.unece.org/trade/locode/om.htm) |
| AZ | Republic of Azerbaijan | Reference [https://service.unece.org/trade/locode/az.htm](https://service.unece.org/trade/locode/az.htm) |
| EG | Arab Republic of Egypt | Reference [https://service.unece.org/trade/locode/eg.htm](https://service.unece.org/trade/locode/eg.htm) |
| ET | Federal Democratic Republic of Ethiopia | Reference [https://service.unece.org/trade/locode/et.htm](https://service.unece.org/trade/locode/et.htm) |
| IE | Ireland | Reference [https://service.unece.org/trade/locode/ie.htm](https://service.unece.org/trade/locode/ie.htm) |
| EE | Republic of Estonia | Reference [https://service.unece.org/trade/locode/ee.htm](https://service.unece.org/trade/locode/ee.htm) |
| AD | Principality of Andorra | Reference [https://service.unece.org/trade/locode/ad.htm](https://service.unece.org/trade/locode/ad.htm) |
| AO | Republic of Angola | Reference [https://service.unece.org/trade/locode/ao.htm](https://service.unece.org/trade/locode/ao.htm) |
| AI | Anguilla | Reference [https://service.unece.org/trade/locode/ai.htm](https://service.unece.org/trade/locode/ai.htm) |
| AG | Antigua and Barbuda | Reference [https://service.unece.org/trade/locode/ag.htm](https://service.unece.org/trade/locode/ag.htm) |
| AT | Republic of Austria | Reference [https://service.unece.org/trade/locode/at.htm](https://service.unece.org/trade/locode/at.htm) |
| AU | Commonwealth of Australia | Reference [https://service.unece.org/trade/locode/au.htm](https://service.unece.org/trade/locode/au.htm) |
| CN | People's Republic of China | Reference [https://service.unece.org/trade/locode/cn.htm](https://service.unece.org/trade/locode/cn.htm) |
| BB | Barbados | Reference [https://service.unece.org/trade/locode/bb.htm](https://service.unece.org/trade/locode/bb.htm) |
| PG | Independent State of Papua New Guinea | Reference [https://service.unece.org/trade/locode/pg.htm](https://service.unece.org/trade/locode/pg.htm) |
| BS | Commonwealth of the Bahamas | Reference [https://service.unece.org/trade/locode/bs.htm](https://service.unece.org/trade/locode/bs.htm) |
| PK | Islamic Republic of Pakistan | Reference [https://service.unece.org/trade/locode/pk.htm](https://service.unece.org/trade/locode/pk.htm) |
| PY | Republic of Paraguay | Reference [https://service.unece.org/trade/locode/py.htm](https://service.unece.org/trade/locode/py.htm) |
| PS | State of Palestine | Reference [https://service.unece.org/trade/locode/ps.htm](https://service.unece.org/trade/locode/ps.htm) |
| BH | Kingdom of Bahrain | Reference [https://service.unece.org/trade/locode/bh.htm](https://service.unece.org/trade/locode/bh.htm) |
| PA | Republic of Panama | Reference [https://service.unece.org/trade/locode/pa.htm](https://service.unece.org/trade/locode/pa.htm) |
| BR | Federative Republic of Brazil | Reference [https://service.unece.org/trade/locode/br.htm](https://service.unece.org/trade/locode/br.htm) |
| BY | Republic of Belarus | Reference [https://service.unece.org/trade/locode/by.htm](https://service.unece.org/trade/locode/by.htm) |
| BM | Bermuda Islands | Reference [https://service.unece.org/trade/locode/bm.htm](https://service.unece.org/trade/locode/bm.htm) |
| BG | Republic of Bulgaria | Reference [https://service.unece.org/trade/locode/bg.htm](https://service.unece.org/trade/locode/bg.htm) |
| MP | Commonwealth of the Northern Mariana Islands | Reference [https://service.unece.org/trade/locode/mp.htm](https://service.unece.org/trade/locode/mp.htm) |
| BJ | Republic of Benin | Reference [https://service.unece.org/trade/locode/bj.htm](https://service.unece.org/trade/locode/bj.htm) |
| BE | Kingdom of Belgium | Reference [https://service.unece.org/trade/locode/be.htm](https://service.unece.org/trade/locode/be.htm) |
| IS | Republic of Iceland | Reference [https://service.unece.org/trade/locode/is.htm](https://service.unece.org/trade/locode/is.htm) |
| PR | Puerto Rico | Reference [https://service.unece.org/trade/locode/pr.htm](https://service.unece.org/trade/locode/pr.htm) |
| BA | Republic of Bosnia and Herzegovina | Reference [https://service.unece.org/trade/locode/ba.htm](https://service.unece.org/trade/locode/ba.htm) |
| PL | Republic of Poland | Reference [https://service.unece.org/trade/locode/pl.htm](https://service.unece.org/trade/locode/pl.htm) |
| BO | Republic of Bolivia | Reference [https://service.unece.org/trade/locode/bo.htm](https://service.unece.org/trade/locode/bo.htm) |
| BZ | Belize | Reference [https://service.unece.org/trade/locode/bz.htm](https://service.unece.org/trade/locode/bz.htm) |
| BW | Republic of Botswana | Reference [https://service.unece.org/trade/locode/bw.htm](https://service.unece.org/trade/locode/bw.htm) |
| BT | Kingdom of Bhutan | Reference [https://service.unece.org/trade/locode/bt.htm](https://service.unece.org/trade/locode/bt.htm) |
| BF | Burkina Faso | Reference [https://service.unece.org/trade/locode/bf.htm](https://service.unece.org/trade/locode/bf.htm) |
| BI | Republic of Burundi | Reference [https://service.unece.org/trade/locode/bi.htm](https://service.unece.org/trade/locode/bi.htm) |
| BV | Bouvet Island | Reference [https://service.unece.org/trade/locode/bv.htm](https://service.unece.org/trade/locode/bv.htm) |
| KP | Democratic People's Republic of Korea | Reference [https://service.unece.org/trade/locode/kp.htm](https://service.unece.org/trade/locode/kp.htm) |
| GQ | Republic of Equatorial Guinea | Reference [https://service.unece.org/trade/locode/gq.htm](https://service.unece.org/trade/locode/gq.htm) |
| DK | Kingdom of Denmark | Reference [https://service.unece.org/trade/locode/dk.htm](https://service.unece.org/trade/locode/dk.htm) |
| DE | Federal Republic of Germany | Reference [https://service.unece.org/trade/locode/de.htm](https://service.unece.org/trade/locode/de.htm) |
| TP | East Timor | Reference [https://service.unece.org/trade/locode/tp.htm](https://service.unece.org/trade/locode/tp.htm) |
| TG | Togolese Republic | Reference [https://service.unece.org/trade/locode/tg.htm](https://service.unece.org/trade/locode/tg.htm) |
| DO | Dominican Republic | Reference [https://service.unece.org/trade/locode/do.htm](https://service.unece.org/trade/locode/do.htm) |
| DM | Commonwealth of Dominica | Reference [https://service.unece.org/trade/locode/dm.htm](https://service.unece.org/trade/locode/dm.htm) |
| RU | Russian Federation | Reference [https://service.unece.org/trade/locode/ru.htm](https://service.unece.org/trade/locode/ru.htm) |
| EC | Republic of Ecuador | Reference [https://service.unece.org/trade/locode/ec.htm](https://service.unece.org/trade/locode/ec.htm) |
| ER | State of Eritrea | Reference [https://service.unece.org/trade/locode/er.htm](https://service.unece.org/trade/locode/er.htm) |
| FR | French Republic | Reference [https://service.unece.org/trade/locode/fr.htm](https://service.unece.org/trade/locode/fr.htm) |
| FO | Faroe Islands | Reference [https://service.unece.org/trade/locode/fo.htm](https://service.unece.org/trade/locode/fo.htm) |
| PF | French Polynesia | Reference [https://service.unece.org/trade/locode/pf.htm](https://service.unece.org/trade/locode/pf.htm) |
| GF | French Guiana | Reference [https://service.unece.org/trade/locode/gf.htm](https://service.unece.org/trade/locode/gf.htm) |
| TF | French Southern Territories | Reference [https://service.unece.org/trade/locode/tf.htm](https://service.unece.org/trade/locode/tf.htm) |
| VA | Vatican City State | Reference [https://service.unece.org/trade/locode/va.htm](https://service.unece.org/trade/locode/va.htm) |
| PH | Republic of the Philippines | Reference [https://service.unece.org/trade/locode/ph.htm](https://service.unece.org/trade/locode/ph.htm) |
| FJ | Republic of Fiji | Reference [https://service.unece.org/trade/locode/fj.htm](https://service.unece.org/trade/locode/fj.htm) |
| FI | Republic of Finland | Reference [https://service.unece.org/trade/locode/fi.htm](https://service.unece.org/trade/locode/fi.htm) |
| CV | Republic of Cape Verde | Reference [https://service.unece.org/trade/locode/cv.htm](https://service.unece.org/trade/locode/cv.htm) |
| FK | Falkland Islands | Reference [https://service.unece.org/trade/locode/fk.htm](https://service.unece.org/trade/locode/fk.htm) |
| GM | Republic of The Gambia | Reference [https://service.unece.org/trade/locode/gm.htm](https://service.unece.org/trade/locode/gm.htm) |
| CG | Republic of the Congo | Reference [https://service.unece.org/trade/locode/cg.htm](https://service.unece.org/trade/locode/cg.htm) |
| CD | Democratic Republic of the Congo | Reference [https://service.unece.org/trade/locode/cd.htm](https://service.unece.org/trade/locode/cd.htm) |
| CO | Republic of Colombia | Reference [https://service.unece.org/trade/locode/co.htm](https://service.unece.org/trade/locode/co.htm) |
| CR | Republic of Costa Rica | Reference [https://service.unece.org/trade/locode/cr.htm](https://service.unece.org/trade/locode/cr.htm) |
| GD | Grenada | Reference [https://service.unece.org/trade/locode/gd.htm](https://service.unece.org/trade/locode/gd.htm) |
| GL | Greenland | Reference [https://service.unece.org/trade/locode/gl.htm](https://service.unece.org/trade/locode/gl.htm) |
| GE | Georgia | Reference [https://service.unece.org/trade/locode/ge.htm](https://service.unece.org/trade/locode/ge.htm) |
| CU | Republic of Cuba | Reference [https://service.unece.org/trade/locode/cu.htm](https://service.unece.org/trade/locode/cu.htm) |
| GP | Guadeloupe | Reference [https://service.unece.org/trade/locode/gp.htm](https://service.unece.org/trade/locode/gp.htm) |
| GU | Guam | Reference [https://service.unece.org/trade/locode/gu.htm](https://service.unece.org/trade/locode/gu.htm) |
| GY | Co-operative Republic of Guyana | Reference [https://service.unece.org/trade/locode/gy.htm](https://service.unece.org/trade/locode/gy.htm) |
| KZ | Republic of Kazakhstan | Reference [https://service.unece.org/trade/locode/kz.htm](https://service.unece.org/trade/locode/kz.htm) |
| HT | Republic of Haiti | Reference [https://service.unece.org/trade/locode/ht.htm](https://service.unece.org/trade/locode/ht.htm) |
| KR | Republic of Korea | Reference [https://service.unece.org/trade/locode/kr.htm](https://service.unece.org/trade/locode/kr.htm) |
| NL | Kingdom of the Netherlands | Reference [https://service.unece.org/trade/locode/nl.htm](https://service.unece.org/trade/locode/nl.htm) |
| AN | Netherlands Antilles | Reference [https://service.unece.org/trade/locode/an.htm](https://service.unece.org/trade/locode/an.htm) |
| HM | Heard Island and McDonald Islands | Reference [https://service.unece.org/trade/locode/hm.htm](https://service.unece.org/trade/locode/hm.htm) |
| HN | Republic of Honduras | Reference [https://service.unece.org/trade/locode/hn.htm](https://service.unece.org/trade/locode/hn.htm) |
| KI | Republic of Kiribati | Reference [https://service.unece.org/trade/locode/ki.htm](https://service.unece.org/trade/locode/ki.htm) |
| DJ | Republic of Djibouti | Reference [https://service.unece.org/trade/locode/dj.htm](https://service.unece.org/trade/locode/dj.htm) |
| KG | Kyrgyz Republic | Reference [https://service.unece.org/trade/locode/kg.htm](https://service.unece.org/trade/locode/kg.htm) |
| GN | Republic of Guinea | Reference [https://service.unece.org/trade/locode/gn.htm](https://service.unece.org/trade/locode/gn.htm) |
| GW | Republic of Guinea-Bissau | Reference [https://service.unece.org/trade/locode/gw.htm](https://service.unece.org/trade/locode/gw.htm) |
| CA | Canada | Reference [https://service.unece.org/trade/locode/ca.htm](https://service.unece.org/trade/locode/ca.htm) |
| GH | Republic of Ghana | Reference [https://service.unece.org/trade/locode/gh.htm](https://service.unece.org/trade/locode/gh.htm) |
| GA | Gabonese Republic | Reference [https://service.unece.org/trade/locode/ga.htm](https://service.unece.org/trade/locode/ga.htm) |
| KH | Kingdom of Cambodia | Reference [https://service.unece.org/trade/locode/kh.htm](https://service.unece.org/trade/locode/kh.htm) |
| CZ | Czech Republic | Reference [https://service.unece.org/trade/locode/cz.htm](https://service.unece.org/trade/locode/cz.htm) |
| ZW | Republic of Zimbabwe | Reference [https://service.unece.org/trade/locode/zw.htm](https://service.unece.org/trade/locode/zw.htm) |
| CM | Republic of Cameroon | Reference [https://service.unece.org/trade/locode/cm.htm](https://service.unece.org/trade/locode/cm.htm) |
| QA | State of Qatar | Reference [https://service.unece.org/trade/locode/qa.htm](https://service.unece.org/trade/locode/qa.htm) |
| KY | Cayman Islands | Reference [https://service.unece.org/trade/locode/ky.htm](https://service.unece.org/trade/locode/ky.htm) |
| CC | Cocos (Keeling) Islands | Reference [https://service.unece.org/trade/locode/cc.htm](https://service.unece.org/trade/locode/cc.htm) |
| KM | Union of the Comoros | Reference [https://service.unece.org/trade/locode/km.htm](https://service.unece.org/trade/locode/km.htm) |
| CI | Republic of Côte d'Ivoire | Reference [https://service.unece.org/trade/locode/ci.htm](https://service.unece.org/trade/locode/ci.htm) |
| KW | State of Kuwait | Reference [https://service.unece.org/trade/locode/kw.htm](https://service.unece.org/trade/locode/kw.htm) |
| HR | Republic of Croatia | Reference [https://service.unece.org/trade/locode/hr.htm](https://service.unece.org/trade/locode/hr.htm) |
| KE | Republic of Kenya | Reference [https://service.unece.org/trade/locode/ke.htm](https://service.unece.org/trade/locode/ke.htm) |
| CK | Cook Islands | Reference [https://service.unece.org/trade/locode/ck.htm](https://service.unece.org/trade/locode/ck.htm) |
| LV | Republic of Latvia | Reference [https://service.unece.org/trade/locode/lv.htm](https://service.unece.org/trade/locode/lv.htm) |
| LS | Kingdom of Lesotho | Reference [https://service.unece.org/trade/locode/ls.htm](https://service.unece.org/trade/locode/ls.htm) |
| LA | Lao People's Democratic Republic | Reference [https://service.unece.org/trade/locode/la.htm](https://service.unece.org/trade/locode/la.htm) |
| LB | Lebanese Republic | Reference [https://service.unece.org/trade/locode/lb.htm](https://service.unece.org/trade/locode/lb.htm) |
| LT | Republic of Lithuania | Reference [https://service.unece.org/trade/locode/lt.htm](https://service.unece.org/trade/locode/lt.htm) |
| LR | Republic of Liberia | Reference [https://service.unece.org/trade/locode/lr.htm](https://service.unece.org/trade/locode/lr.htm) |
| LY | Great Socialist People's Libyan Arab Jamahiriya | Reference [https://service.unece.org/trade/locode/ly.htm](https://service.unece.org/trade/locode/ly.htm) |
| LI | Principality of Liechtenstein | Reference [https://service.unece.org/trade/locode/li.htm](https://service.unece.org/trade/locode/li.htm) |
| RE | Réunion | Reference [https://service.unece.org/trade/locode/re.htm](https://service.unece.org/trade/locode/re.htm) |
| LU | Grand Duchy of Luxembourg | Reference [https://service.unece.org/trade/locode/lu.htm](https://service.unece.org/trade/locode/lu.htm) |
| RW | Republic of Rwanda | Reference [https://service.unece.org/trade/locode/rw.htm](https://service.unece.org/trade/locode/rw.htm) |
| RO | Romania | Reference [https://service.unece.org/trade/locode/ro.htm](https://service.unece.org/trade/locode/ro.htm) |
| MG | Republic of Madagascar | Reference [https://service.unece.org/trade/locode/mg.htm](https://service.unece.org/trade/locode/mg.htm) |
| MV | Republic of Maldives | Reference [https://service.unece.org/trade/locode/mv.htm](https://service.unece.org/trade/locode/mv.htm) |
| MT | Republic of Malta | Reference [https://service.unece.org/trade/locode/mt.htm](https://service.unece.org/trade/locode/mt.htm) |
| MW | Republic of Malawi | Reference [https://service.unece.org/trade/locode/mw.htm](https://service.unece.org/trade/locode/mw.htm) |
| MY | Malaysia | Reference [https://service.unece.org/trade/locode/my.htm](https://service.unece.org/trade/locode/my.htm) |
| ML | Republic of Mali | Reference [https://service.unece.org/trade/locode/ml.htm](https://service.unece.org/trade/locode/ml.htm) |
| MH | Republic of the Marshall Islands | Reference [https://service.unece.org/trade/locode/mh.htm](https://service.unece.org/trade/locode/mh.htm) |
| MQ | Martinique | Reference [https://service.unece.org/trade/locode/mq.htm](https://service.unece.org/trade/locode/mq.htm) |
| YT | Mayotte | Reference [https://service.unece.org/trade/locode/yt.htm](https://service.unece.org/trade/locode/yt.htm) |
| MU | Republic of Mauritius | Reference [https://service.unece.org/trade/locode/mu.htm](https://service.unece.org/trade/locode/mu.htm) |
| MR | Islamic Republic of Mauritania | Reference [https://service.unece.org/trade/locode/mr.htm](https://service.unece.org/trade/locode/mr.htm) |
| US | United States of America | Reference [https://service.unece.org/trade/locode/us.htm](https://service.unece.org/trade/locode/us.htm) |
| UM | United States Minor Outlying Islands | Reference [https://service.unece.org/trade/locode/um.htm](https://service.unece.org/trade/locode/um.htm) |
| AS | American Samoa | Reference [https://service.unece.org/trade/locode/as.htm](https://service.unece.org/trade/locode/as.htm) |
| VI | United States Virgin Islands | Reference [https://service.unece.org/trade/locode/vi.htm](https://service.unece.org/trade/locode/vi.htm) |
| MN | Mongolia | Reference [https://service.unece.org/trade/locode/mn.htm](https://service.unece.org/trade/locode/mn.htm) |
| MS | Montserrat | Reference [https://service.unece.org/trade/locode/ms.htm](https://service.unece.org/trade/locode/ms.htm) |
| BD | People's Republic of Bangladesh | Reference [https://service.unece.org/trade/locode/bd.htm](https://service.unece.org/trade/locode/bd.htm) |
| PE | Republic of Peru | Reference [https://service.unece.org/trade/locode/pe.htm](https://service.unece.org/trade/locode/pe.htm) |
| FM | Federated States of Micronesia | Reference [https://service.unece.org/trade/locode/fm.htm](https://service.unece.org/trade/locode/fm.htm) |
| MM | Union of Myanmar | Reference [https://service.unece.org/trade/locode/mm.htm](https://service.unece.org/trade/locode/mm.htm) |
| MD | Republic of Moldova | Reference [https://service.unece.org/trade/locode/md.htm](https://service.unece.org/trade/locode/md.htm) |
| MA | Kingdom of Morocco | Reference [https://service.unece.org/trade/locode/ma.htm](https://service.unece.org/trade/locode/ma.htm) |
| MC | Principality of Monaco | Reference [https://service.unece.org/trade/locode/mc.htm](https://service.unece.org/trade/locode/mc.htm) |
| MZ | Republic of Mozambique | Reference [https://service.unece.org/trade/locode/mz.htm](https://service.unece.org/trade/locode/mz.htm) |
| MX | United Mexican States | Reference [https://service.unece.org/trade/locode/mx.htm](https://service.unece.org/trade/locode/mx.htm) |
| NA | Namibia | Reference [https://service.unece.org/trade/locode/na.htm](https://service.unece.org/trade/locode/na.htm) |
| ZA | Republic of South Africa | Reference [https://service.unece.org/trade/locode/za.htm](https://service.unece.org/trade/locode/za.htm) |
| AQ | Antarctica | Reference [https://service.unece.org/trade/locode/aq.htm](https://service.unece.org/trade/locode/aq.htm) |
| GS | South Georgia and the South Sandwich Islands | Reference [https://service.unece.org/trade/locode/gs.htm](https://service.unece.org/trade/locode/gs.htm) |
| YU | Federal Republic of Yugoslavia | Reference [https://service.unece.org/trade/locode/yu.htm](https://service.unece.org/trade/locode/yu.htm) |
| NR | Republic of Nauru | Reference [https://service.unece.org/trade/locode/nr.htm](https://service.unece.org/trade/locode/nr.htm) |
| NP | Kingdom of Nepal | Reference [https://service.unece.org/trade/locode/np.htm](https://service.unece.org/trade/locode/np.htm) |
| NI | Republic of Nicaragua | Reference [https://service.unece.org/trade/locode/ni.htm](https://service.unece.org/trade/locode/ni.htm) |
| NE | Republic of Niger | Reference [https://service.unece.org/trade/locode/ne.htm](https://service.unece.org/trade/locode/ne.htm) |
| NG | Federal Republic of Nigeria | Reference [https://service.unece.org/trade/locode/ng.htm](https://service.unece.org/trade/locode/ng.htm) |
| NU | Niue | Reference [https://service.unece.org/trade/locode/nu.htm](https://service.unece.org/trade/locode/nu.htm) |
| NO | Kingdom of Norway | Reference [https://service.unece.org/trade/locode/no.htm](https://service.unece.org/trade/locode/no.htm) |
| NF | Norfolk Island | Reference [https://service.unece.org/trade/locode/nf.htm](https://service.unece.org/trade/locode/nf.htm) |
| PW | Republic of Palau | Reference [https://service.unece.org/trade/locode/pw.htm](https://service.unece.org/trade/locode/pw.htm) |
| PN | Pitcairn | Reference [https://service.unece.org/trade/locode/pn.htm](https://service.unece.org/trade/locode/pn.htm) |
| PT | Portuguese Republic | Reference [https://service.unece.org/trade/locode/pt.htm](https://service.unece.org/trade/locode/pt.htm) |
| MK | Former Yugoslav Republic of Macedonia | Reference [https://service.unece.org/trade/locode/mk.htm](https://service.unece.org/trade/locode/mk.htm) |
| JP | Japan | Reference [https://service.unece.org/trade/locode/jp.htm](https://service.unece.org/trade/locode/jp.htm) |
| SE | Kingdom of Sweden | Reference [https://service.unece.org/trade/locode/se.htm](https://service.unece.org/trade/locode/se.htm) |
| CH | Swiss Confederation | Reference [https://service.unece.org/trade/locode/ch.htm](https://service.unece.org/trade/locode/ch.htm) |
| SV | Republic of El Salvador | Reference [https://service.unece.org/trade/locode/sv.htm](https://service.unece.org/trade/locode/sv.htm) |
| WS | Independent State of Samoa | Reference [https://service.unece.org/trade/locode/ws.htm](https://service.unece.org/trade/locode/ws.htm) |
| SL | Republic of Sierra Leone | Reference [https://service.unece.org/trade/locode/sl.htm](https://service.unece.org/trade/locode/sl.htm) |
| SN | Republic of Senegal | Reference [https://service.unece.org/trade/locode/sn.htm](https://service.unece.org/trade/locode/sn.htm) |
| CY | Republic of Cyprus | Reference [https://service.unece.org/trade/locode/cy.htm](https://service.unece.org/trade/locode/cy.htm) |
| SC | Republic of Seychelles | Reference [https://service.unece.org/trade/locode/sc.htm](https://service.unece.org/trade/locode/sc.htm) |
| SA | Kingdom of Saudi Arabia | Reference [https://service.unece.org/trade/locode/sa.htm](https://service.unece.org/trade/locode/sa.htm) |
| CX | Christmas Island | Reference [https://service.unece.org/trade/locode/cx.htm](https://service.unece.org/trade/locode/cx.htm) |
| ST | Democratic Republic of São Tomé and Príncipe | Reference [https://service.unece.org/trade/locode/st.htm](https://service.unece.org/trade/locode/st.htm) |
| SH | Saint Helena | Reference [https://service.unece.org/trade/locode/sh.htm](https://service.unece.org/trade/locode/sh.htm) |
| KN | Federation of Saint Kitts and Nevis | Reference [https://service.unece.org/trade/locode/kn.htm](https://service.unece.org/trade/locode/kn.htm) |
| LC | Saint Lucia | Reference [https://service.unece.org/trade/locode/lc.htm](https://service.unece.org/trade/locode/lc.htm) |
| SM | Republic of San Marino | Reference [https://service.unece.org/trade/locode/sm.htm](https://service.unece.org/trade/locode/sm.htm) |
| PM | Saint Pierre and Miquelon | Reference [https://service.unece.org/trade/locode/pm.htm](https://service.unece.org/trade/locode/pm.htm) |
| VC | Saint Vincent and the Grenadines | Reference [https://service.unece.org/trade/locode/vc.htm](https://service.unece.org/trade/locode/vc.htm) |
| LK | Democratic Socialist Republic of Sri Lanka | Reference [https://service.unece.org/trade/locode/lk.htm](https://service.unece.org/trade/locode/lk.htm) |
| SK | Slovak Republic | Reference [https://service.unece.org/trade/locode/sk.htm](https://service.unece.org/trade/locode/sk.htm) |
| SI | Republic of Slovenia | Reference [https://service.unece.org/trade/locode/si.htm](https://service.unece.org/trade/locode/si.htm) |
| SJ | Svalbard and Jan Mayen Islands | Reference [https://service.unece.org/trade/locode/sj.htm](https://service.unece.org/trade/locode/sj.htm) |
| SZ | Kingdom of Swaziland | Reference [https://service.unece.org/trade/locode/sz.htm](https://service.unece.org/trade/locode/sz.htm) |
| SD | Republic of the Sudan | Reference [https://service.unece.org/trade/locode/sd.htm](https://service.unece.org/trade/locode/sd.htm) |
| SR | Republic of Suriname | Reference [https://service.unece.org/trade/locode/sr.htm](https://service.unece.org/trade/locode/sr.htm) |
| SB | Solomon Islands | Reference [https://service.unece.org/trade/locode/sb.htm](https://service.unece.org/trade/locode/sb.htm) |
| SO | Somali Republic | Reference [https://service.unece.org/trade/locode/so.htm](https://service.unece.org/trade/locode/so.htm) |
| TJ | Republic of Tajikistan | Reference [https://service.unece.org/trade/locode/tj.htm](https://service.unece.org/trade/locode/tj.htm) |
| TW | Taiwan, China | Reference [https://service.unece.org/trade/locode/tw.htm](https://service.unece.org/trade/locode/tw.htm) |
| TH | Kingdom of Thailand | Reference [https://service.unece.org/trade/locode/th.htm](https://service.unece.org/trade/locode/th.htm) |
| TZ | United Republic of Tanzania | Reference [https://service.unece.org/trade/locode/tz.htm](https://service.unece.org/trade/locode/tz.htm) |
| TO | Kingdom of Tonga | Reference [https://service.unece.org/trade/locode/to.htm](https://service.unece.org/trade/locode/to.htm) |
| TC | Turks and Caicos Islands | Reference [https://service.unece.org/trade/locode/tc.htm](https://service.unece.org/trade/locode/tc.htm) |
| TT | Republic of Trinidad and Tobago | Reference [https://service.unece.org/trade/locode/tt.htm](https://service.unece.org/trade/locode/tt.htm) |
| TN | Tunisian Republic | Reference [https://service.unece.org/trade/locode/tn.htm](https://service.unece.org/trade/locode/tn.htm) |
| TV | Tuvalu | Reference [https://service.unece.org/trade/locode/tv.htm](https://service.unece.org/trade/locode/tv.htm) |
| TR | Republic of Turkey | Reference [https://service.unece.org/trade/locode/tr.htm](https://service.unece.org/trade/locode/tr.htm) |
| TM | Turkmenistan | Reference [https://service.unece.org/trade/locode/tm.htm](https://service.unece.org/trade/locode/tm.htm) |
| TK | Tokelau | Reference [https://service.unece.org/trade/locode/tk.htm](https://service.unece.org/trade/locode/tk.htm) |
| WF | Wallis and Futuna | Reference [https://service.unece.org/trade/locode/wf.htm](https://service.unece.org/trade/locode/wf.htm) |
| VU | Republic of Vanuatu | Reference [https://service.unece.org/trade/locode/vu.htm](https://service.unece.org/trade/locode/vu.htm) |
| GT | Republic of Guatemala | Reference [https://service.unece.org/trade/locode/gt.htm](https://service.unece.org/trade/locode/gt.htm) |
| VE | Republic of Venezuela | Reference [https://service.unece.org/trade/locode/ve.htm](https://service.unece.org/trade/locode/ve.htm) |
| BN | Negara Brunei Darussalam | Reference [https://service.unece.org/trade/locode/bn.htm](https://service.unece.org/trade/locode/bn.htm) |
| UG | Republic of Uganda | Reference [https://service.unece.org/trade/locode/ug.htm](https://service.unece.org/trade/locode/ug.htm) |
| UA | Ukraine | Reference [https://service.unece.org/trade/locode/ua.htm](https://service.unece.org/trade/locode/ua.htm) |
| UY | Eastern Republic of Uruguay | Reference [https://service.unece.org/trade/locode/uy.htm](https://service.unece.org/trade/locode/uy.htm) |
| UZ | Republic of Uzbekistan | Reference [https://service.unece.org/trade/locode/uz.htm](https://service.unece.org/trade/locode/uz.htm) |
| ES | Kingdom of Spain | Reference [https://service.unece.org/trade/locode/es.htm](https://service.unece.org/trade/locode/es.htm) |
| EH | Western Sahara | Reference [https://service.unece.org/trade/locode/eh.htm](https://service.unece.org/trade/locode/eh.htm) |
| GR | Hellenic Republic | Reference [https://service.unece.org/trade/locode/gr.htm](https://service.unece.org/trade/locode/gr.htm) |
| HK | Hong Kong Special Administrative Region of China | Reference [https://service.unece.org/trade/locode/hk.htm](https://service.unece.org/trade/locode/hk.htm) |
| SG | Republic of Singapore | Reference [https://service.unece.org/trade/locode/sg.htm](https://service.unece.org/trade/locode/sg.htm) |
| NC | New Caledonia | Reference [https://service.unece.org/trade/locode/nc.htm](https://service.unece.org/trade/locode/nc.htm) |
| NZ | New Zealand | Reference [https://service.unece.org/trade/locode/nz.htm](https://service.unece.org/trade/locode/nz.htm) |
| HU | Republic of Hungary | Reference [https://service.unece.org/trade/locode/hu.htm](https://service.unece.org/trade/locode/hu.htm) |
| SY | Syrian Arab Republic | Reference [https://service.unece.org/trade/locode/sy.htm](https://service.unece.org/trade/locode/sy.htm) |
| JM | Jamaica | Reference [https://service.unece.org/trade/locode/jm.htm](https://service.unece.org/trade/locode/jm.htm) |
| AM | Republic of Armenia | Reference [https://service.unece.org/trade/locode/am.htm](https://service.unece.org/trade/locode/am.htm) |
| YE | Republic of Yemen | Reference [https://service.unece.org/trade/locode/ye.htm](https://service.unece.org/trade/locode/ye.htm) |
| IQ | Republic of Iraq | Reference [https://service.unece.org/trade/locode/iq.htm](https://service.unece.org/trade/locode/iq.htm) |
| IR | Islamic Republic of Iran | Reference [https://service.unece.org/trade/locode/ir.htm](https://service.unece.org/trade/locode/ir.htm) |
| IL | State of Israel | Reference [https://service.unece.org/trade/locode/il.htm](https://service.unece.org/trade/locode/il.htm) |
| IT | Italian Republic | Reference [https://service.unece.org/trade/locode/it.htm](https://service.unece.org/trade/locode/it.htm) |
| IN | Republic of India | Reference [https://service.unece.org/trade/locode/in.htm](https://service.unece.org/trade/locode/in.htm) |
| ID | Republic of Indonesia | Reference [https://service.unece.org/trade/locode/id.htm](https://service.unece.org/trade/locode/id.htm) |
| GB | United Kingdom of Great Britain and Northern Ireland | Reference [https://service.unece.org/trade/locode/gb.htm](https://service.unece.org/trade/locode/gb.htm) |
| VG | British Virgin Islands | Reference [https://service.unece.org/trade/locode/vg.htm](https://service.unece.org/trade/locode/vg.htm) |
| IO | British Indian Ocean Territory | Reference [https://service.unece.org/trade/locode/io.htm](https://service.unece.org/trade/locode/io.htm) |
| JO | Hashemite Kingdom of Jordan | Reference [https://service.unece.org/trade/locode/jo.htm](https://service.unece.org/trade/locode/jo.htm) |
| VN | Socialist Republic of Vietnam | Reference [https://service.unece.org/trade/locode/vn.htm](https://service.unece.org/trade/locode/vn.htm) |
| ZM | Republic of Zambia | Reference [https://service.unece.org/trade/locode/zm.htm](https://service.unece.org/trade/locode/zm.htm) |
| TD | Republic of Chad | Reference [https://service.unece.org/trade/locode/td.htm](https://service.unece.org/trade/locode/td.htm) |
| GI | Gibraltar | Reference [https://service.unece.org/trade/locode/gi.htm](https://service.unece.org/trade/locode/gi.htm) |
| CL | Republic of Chile | Reference [https://service.unece.org/trade/locode/cl.htm](https://service.unece.org/trade/locode/cl.htm) |
| CF | Central African Republic | Reference [https://service.unece.org/trade/locode/cf.htm](https://service.unece.org/trade/locode/cf.htm) |