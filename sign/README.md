## Signature process

###  Preface

#### 1 Obtain your APPID and SecretKey.

> Example: `AppID`="OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz"  `SecretKey`="zL2gkc8plUqBZObiICG5qdIbmwAr16ps"

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
> Example: `keyTime`="1581782400;1581786000"

#### step.2 build signKey
`signKey=HMAC-SHA1(SecretKey, KeyTime)`

> Example: `signKey`=HMAC-SHA1('zL2gkc8plUqBZObiICG5qdIbmwAr16ps','1581782400;1581786000') => '3368ElqG0cSjECtfLkJt2bl240c='

#### step.3 obtain signContent

> Signature related fields: `appId`,`keyTime`,`sign`

##### 3.1 Signature related fields are on QueryString

> Arrange the request parameters in ascending order of the key field and splice them into: `k1=v1&k2=v2&k3=v3&...`(exculde fields: keyTime,sign)

> Note: Because the QueryString parameter may have special characters, it is recommended to urlEncode，
> As above, splicing QueryString parameters into:`urlEncode(k1)=urlEncode(v1)&urlEncode(k2)=urlEncode(v2)&urlEncode(k3)=urlEncode(v3)&...`

Example: 
PUT https://{DOMAIN}/demo/user/1001?appId=OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz&keyTime=1581782400;1581786000&newPwd=123&newName=Dean&sign=SWWdcOeBQDDMQvSHgnZxfZOJoI0%3D

> `signContent`="appId=OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz&newName=Dean&newPwd=123"

##### 3.2 Signature related fields are on request Body

> Arrange the payload parameters in ascending order of the key field and splice them into: `k1=v1&k2=v2&k3=v3&...`(exculde fields: keyTime,sign)

Example: 
```
PUT https://{DOMAIN}/demo/user/1001
request body:
{
    "appId":"OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz",
    "newPwd":"123",
    "newName":"Dean",
    "keyTime":"1581782400;1581786000",
    "sign":"SWWdcOeBQDDMQvSHgnZxfZOJoI0="
}
```
> `signContent`="appId=OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz&newName=Dean&newPwd=123"


#### step.4 build signature
`signature=HMAC-SHA1(signKey, signContent)`
> Example: `signature`=HMAC-SHA1('srP+FDEc8c1xVqaXoYP7dlRDQBc=', `signContent`) => 'SWWdcOeBQDDMQvSHgnZxfZOJoI0='


### For Example:

#### 1 Signature related fields are on QueryString

> Example: PUT https://{DOMAIN}/demo/user/1001?appId=OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz&keyTime=1581782400;1581786000&newPwd=123&newName=Dean&sign=SWWdcOeBQDDMQvSHgnZxfZOJoI0%3D

```
`appId`="OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz"
`keyTime`="1581782400;1581786000"
==> build `signKey`="3368ElqG0cSjECtfLkJt2bl240c="
==> obtain `signContent`="appId=OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz&newName=Dean&newPwd=123"
==> build  `signature`="SWWdcOeBQDDMQvSHgnZxfZOJoI0="

```

#### 2 Signature related fields are on request Body
Example: 
```
PUT https://{DOMAIN}/demo/user/1001
request body:
{
    "appId":"OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz",
    "newPwd":"123",
    "newName":"Dean",
    "keyTime":"1581782400;1581786000",
    "sign":"SWWdcOeBQDDMQvSHgnZxfZOJoI0="
}

`appId`="OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz"
`keyTime`="1581782400;1581786000"
==> build `signKey`="3368ElqG0cSjECtfLkJt2bl240c="
==> obtain `signContent`="appId=OOOb9l9qWyzBP2OuYHaqgtG8R3o8v1jz&newName=Dean&newPwd=123"
==> build  `signature`="SWWdcOeBQDDMQvSHgnZxfZOJoI0="
```

