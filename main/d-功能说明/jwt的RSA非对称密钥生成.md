# jwt的RSA非对称密钥生成

## 生成密钥文件

使用JDK自带的keytool工具，执行后会在当前目录生成djj_secret.jks（.jks文件是Java中的密钥管理库）。

```bash
keytool -genkey -alias djj -keyalg RSA -storetype PKCS12 -keystore djj.jks


keytool -genkeypair -alias djj -validity 36500 -keyalg RSA  -keypass 123456 -keystore djj-jwt.jks -storepass 123456
```

- 参数解析

  -genkey：创建证书

  -alias：证书的别名。在一个证书库文件中，别名是唯一用来区分多个证书的标识符

  -keyalg：密钥的算法，非对称加密的话就是RSA

  -keystore：证书库文件保存的位置和文件名。如果路径写错的话，会出现报错信息。如果在路径下，证书库文件不存在，那么就会创建一个

  -keysize：密钥长度，一般都是1024

  -validity：证书的有效期，单位是天。比如36500的话，就是100年

- PKCS12：一种二进制格式

密钥库口令：edgarding

## 提取公钥

```bash
keytool -list -rfc -keystore djj.jks -storepass edgarding | openssl x509 -inform pem -pubkey


# 查看公钥
keytool -list -rfc --keystore djj-jwt.jks | openssl x509 -inform pem -pubkey
```

- 参数解析

  -keystore：密钥文件

  -storepass：密钥密码

djj.jks文件查看内容如下：

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsxQCAKJWFpOYFLvUapTW
49iTFeawaHv3kEfmkbqQGo50fXdIkldfA9ySYKWr8szG+2zqp8aW2f0YhFDddzpU
3BNV6PYXp3fK7nj2NZJgEQHZzz0RTVP82keijn8nt7KYcufYS5fb76scssGbyJ0p
S/X/L0VoDCAVLqGwBIUpKnDg8Ri54+eUAySr2BqnIHTcTKyQK5Y75mLn6dsDMsFD
MropCgp/41j4hCMUhUZXYEYtUmYs7OaZhqD9Um6EH+nkoB9cnybzpQ0cVbtaqoJ4
sju894xMG8KPe2uwB1vnZwKE/wpJnAh4wymBI6xXbOOcHz9t/cmsaUch0HCYsL0X
rwIDAQAB
-----END PUBLIC KEY-----
-----BEGIN CERTIFICATE-----
MIIDdzCCAl+gAwIBAgIEDmLq+DANBgkqhkiG9w0BAQsFADBsMRAwDgYDVQQGEwdV
bmtub3duMRAwDgYDVQQIEwdVbmtub3duMRAwDgYDVQQHEwdVbmtub3duMRAwDgYD
VQQKEwdVbmtub3duMRAwDgYDVQQLEwdVbmtub3duMRAwDgYDVQQDEwdVbmtub3du
MB4XDTIxMTEyNzA1NDkyOVoXDTIyMDIyNTA1NDkyOVowbDEQMA4GA1UEBhMHVW5r
bm93bjEQMA4GA1UECBMHVW5rbm93bjEQMA4GA1UEBxMHVW5rbm93bjEQMA4GA1UE
ChMHVW5rbm93bjEQMA4GA1UECxMHVW5rbm93bjEQMA4GA1UEAxMHVW5rbm93bjCC
ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALMUAgCiVhaTmBS71GqU1uPY
kxXmsGh795BH5pG6kBqOdH13SJJXXwPckmClq/LMxvts6qfGltn9GIRQ3Xc6VNwT
Vej2F6d3yu549jWSYBEB2c89EU1T/NpHoo5/J7eymHLn2EuX2++rHLLBm8idKUv1
/y9FaAwgFS6hsASFKSpw4PEYuePnlAMkq9gapyB03EyskCuWO+Zi5+nbAzLBQzK6
KQoKf+NY+IQjFIVGV2BGLVJmLOzmmYag/VJuhB/p5KAfXJ8m86UNHFW7WqqCeLI7
vPeMTBvCj3trsAdb52cChP8KSZwIeMMpgSOsV2zjnB8/bf3JrGlHIdBwmLC9F68C
AwEAAaMhMB8wHQYDVR0OBBYEFG2NwfkdeEEO6n4o3YqwO+4I9zCdMA0GCSqGSIb3
DQEBCwUAA4IBAQCNFfxcAiZDsdajRDAmibIdv3pH7ceGmLusEsYs6XQECHnj6suS
l8I1iXIkaiT3WfyZ87RZSyAMs+vK15rDOR8m9xKDqJT5u69mAn/3nKCiuK8imTH7
2cbzllWuZS05SIGEs1qHS1CLmSH3GrfGnUHcjfjPBzQsWPLEzmTmSW3tIGuJaGLQ
uQJVVYXtOo2ubo8+YV9MS0Yqpka7AtIOFSBnLnmaW4RDlsXZKJhUAi2LJZ8E8RCJ
T/cP697tG4DDdmkwAdmFsgMCDAnAfDbeW2BGKBzFMwPrB2cIt9G1zd3l6hqu5kSd
FVcW/cr2tdO5v+/cPPZywrvK+3GmC05lK6T5
-----END CERTIFICATE-----

```

public key就是公钥部分了，直接创建一个pubkey.txt文件复制进去即可。