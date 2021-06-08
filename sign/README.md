## Signature process

###  Preface

#### 1 Obtain your APPID and SecretKey.
Example:
> `AppID`="9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn"  `SecretKey`="Dmg40YVklLzHLc7K1D3TZQKuHp5mzhYW"

#### 2 Prepare an encryption method for `HMAC-SHA1`
Example for java:

```java
public static String hmacSHA1ToBase64(String key, String content) {
		try {
			byte[] data = key.getBytes("UTF-8");
			SecretKey secretKey = new SecretKeySpec(data, "HmacSHA1");
			Mac mac = Mac.getInstance("HmacSHA1");
			mac.init(secretKey);

			byte[] contentBytes = content.getBytes("UTF-8");
			byte[] encryptBytes = mac.doFinal(contentBytes);
			return new Base64Encoder().encode(encryptBytes);
		}catch (Exception e) {
			throw new CheckedException(e.getMessage());
		}
		return "";
	}
```

### Get Started

#### step.1 build keyTime(Validity period of sign):
`keyTime=startTimestampSecond+";"+endTimestampSecond`

**please note that: startTimestampSecond should be greater than current time(ex: `startTimestampSecond=currentTimeSecond+10s`)
,because our service will verify the validate of this**

Example: 
> `keyTime`="1581782400;1581786000"

#### step.2 build signKey
`signKey=HMAC-SHA1(SecretKey, KeyTime)`

Example: 
> `signKey`=HMAC-SHA1('Dmg40YVklLzHLc7K1D3TZQKuHp5mzhYW','1581782400;1581786000') => 'AKVN4wrJCelZ2JG2R6XD7lYKFdI='

#### step.3 obtain signContent

Signature related fields: `appId`,`keyTime`,`sign`

##### 3.1 Signature related fields are on QueryString

Arrange the request parameters in ascending order of the key field and splice them into: `k1=v1&k2=v2&k3=v3&...`(exculde fields: `keyTime`,`sign`)

> Note: <br>Because the QueryString parameter may have special characters, it is recommended to urlEncode,
> As above, splicing QueryString parameters into:`urlEncode(k1)=urlEncode(v1)&urlEncode(k2)=urlEncode(v2)&urlEncode(k3)=urlEncode(v3)&...`

Example: 

```
PUT https://{DOMAIN}/demo/user/1001?appId=9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn&keyTime=1581782400;1581786000&newPwd=123&newName=Dean&sign=SWWdcOeBQDDMQvSHgnZxfZOJoI0%3D
```

> `signContent`="appId=9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn&newName=Dean&newPwd=123"

##### 3.2 Signature related fields are on request Body

Arrange the payload parameters in ascending order of the key field and splice them into: `k1=v1&k2=v2&k3=v3&...`(exculde fields: `keyTime`,`sign`)

Example: 
```
PUT https://{DOMAIN}/demo/user/1001
request body:
{
    "appId":"9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn",
    "newPwd":"123",
    "newName":"Dean",
    "keyTime":"1581782400;1581786000",
    "sign":"dIMjxgE7gHjPWlAKY4eIgI0i98Y="
}
```
> `signContent`="appId=9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn&newName=Dean&newPwd=123"


#### step.4 build signature
`signature=HMAC-SHA1(signKey, signContent)`

Example:
>  `signature`=HMAC-SHA1('AKVN4wrJCelZ2JG2R6XD7lYKFdI=', `signContent`) => 'dIMjxgE7gHjPWlAKY4eIgI0i98Y='


### For Example:

#### 1 Signature related fields are on QueryString

Example: 

```json
PUT https://{DOMAIN}/demo/user/1001?appId=9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn&keyTime=1581782400;1581786000&newPwd=123&newName=Dean&sign=dIMjxgE7gHjPWlAKY4eIgI0i98Y%3D

`appId`="9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn"
`keyTime`="1581782400;1581786000"
==> build `signKey`="AKVN4wrJCelZ2JG2R6XD7lYKFdI=="
==> obtain `signContent`="appId=9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn&newName=Dean&newPwd=123"
==> build  `signature`="dIMjxgE7gHjPWlAKY4eIgI0i98Y="

```

#### 2 Signature related fields are on request Body
Example: 

```json
PUT https://{DOMAIN}/demo/user/1001
request body:
{
    "appId":"9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn",
    "newPwd":"123",
    "newName":"Dean",
    "keyTime":"1581782400;1581786000",
    "sign":"dIMjxgE7gHjPWlAKY4eIgI0i98Y="
}

`appId`="9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn"
`keyTime`="1581782400;1581786000"
==> build `signKey`="AKVN4wrJCelZ2JG2R6XD7lYKFdI=="
==> obtain `signContent`="appId=9ft8PvZ1ZQK6vpBJ8JnEFvqIQbWe0yKn&newName=Dean&newPwd=123"
==> build  `signature`="dIMjxgE7gHjPWlAKY4eIgI0i98Y="
```

