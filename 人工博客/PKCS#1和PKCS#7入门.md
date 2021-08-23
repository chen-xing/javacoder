### PKCS#1和PKCS#7入门

### 摘要

p1是裸签名，p7是包含签名证书信息,签名原文信息,时间戳信息的签名结构。attached是包含签名原文，detached 是不包含签名原文的。



### 关键字

p1和p7的区别，attached 和detached 

### 1、PKCS#1和PKCS#7的区别

+ P1签名:即裸签名,签名值中只有签名信息.

+  p7签名:即,签名中可以带有其他的附加信息,例如签名证书信息,签名原文信息,时间戳信息等.
   所以要注意,不要p7的签名,用p1的方式来验签,这样是不对的.是错误的.
   
   

PKCS#1：定义RSA公开密钥算法加密和签名机制，主要用于组织PKCS#7中所描述的数字签名和数字信封[22]。
PKCS#7：定义一种通用的消息语法，包括数字签名和加密等用于增强的加密机制，PKCS#7与PEM兼容，所以不需其他密码操作，就可以将加密的消息转换成PEM消息[26]。



### 2、目前常见的几种方式:(P1签名验签)

```objectivec
contentType==CT_MESSAGE时，为待签名的消息；
contentType==CT_BASE64_DATA时，为待签名的base64编码数据；
contentType==CT_HASH时为待签名的HASH；
contentType==CT_FILE_URL时为待签名文件地址URL
contentType== CT_STORAGE时为待签名内容存储编号
```

+ CT_MESSAGE:即,告诉签名方,我发过去的内容,就是签名的原文.一般来说,因为签名值都是字节数组,所以签名方会根据约定的编码方式,对这个签名原文按规定的编码方式,取它的字节数组之后,进行签名
+ CT_BASE64_DATA:因为有时候,我们要签名的原文,它就是一个byte[],但是为了方便传输,一般签名方都要求接受的是一个字符串.所以就有了这种签名方式.即:我们需要对签名原文的byte[]做base64编码,签名方收到之后,再做解码,并且把解码后的byte[]进行签名.
所以我们本身验签的时候,只要传入byte[]就行.
第三方返回的签名值,一般也是签名值byte[]做base64编码后的字符串,所以我们要做base64解码,才能获取到签名值byte[].
+ CT_HASH:告诉签名方,我这个是对签名原文hash过了的hash串,你们直接对这个值进行加密即可,不需要再hash了.
   所以这种方式,我们本地验签的时候,要将原文放入进行验签,而不是hash过后的hash值(因为验签的时候,它又会hash一次)
+ 

验签示例代码

```
  /**
     * 测试hash方式的签名验签
     * @throws Exception 
     */
    @Test
    public void testSignWithHash() {
        try {
            CertificateFactory cf = CertificateFactory.getInstance("X.509");  
            Certificate usercert = cf.generateCertificate(new FileInputStream(USER_CERT)); 
            PublicKey publicKey = usercert.getPublicKey();
            //读取pfx证书上的秘钥对信息
            BouncyCastleProvider provider = new BouncyCastleProvider();
            Signature signature = Signature.getInstance("SHA1withRSA"); 
            MessageDigest digest = MessageDigest.getInstance("SHA-1", provider);

            List<UserInfo> userInfoList = userInfoService.findAll();
            UserInfo userInfo = userInfoList.get(0);
            Map<String, Object> puserCert = new HashMap<>();
            puserCert.put("userInfo", userInfo);
            List<UserCert> userCertList = userCertService.getListByParam(puserCert);
            UserCert userCert = userCertList.get(0);
            YuanZi_P1SignRequest yuanZiP1SignRequest = new YuanZi_P1SignRequest();
            yuanZiP1SignRequest.setAlias(userInfo.getAlias());
            yuanZiP1SignRequest.setHashAlg("SHA1");
            yuanZiP1SignRequest.setContentType("CT_HASH");
            String content = "我是签名原文";

            byte[] dataToSign = digest.digest(content.getBytes(Charset.forName("UTF-8")));
            System.out.println("dataToSign:"+Base64.encodeBytes(dataToSign));
            yuanZiP1SignRequest.setContent(Base64.encodeBytes(dataToSign));
            YuanZi_P1SignResponse yuanZiP1SignResponse = yuanZiService.signP1(yuanZiP1SignRequest);
            System.out.println("原子服务返回的签名值:"+yuanZiP1SignResponse.getSignedData());
            byte[] dataHadSignature = Base64.decode(yuanZiP1SignResponse.getSignedData());

            YuanZi_VerifyP1SignRequest yuanZiVerifyP1SignRequest = new YuanZi_VerifyP1SignRequest();
            yuanZiVerifyP1SignRequest.setSignedData(yuanZiP1SignResponse.getSignedData());
            yuanZiVerifyP1SignRequest.setContentType("CT_HASH");
            yuanZiVerifyP1SignRequest.setContent(Base64.encodeBytes(dataToSign));
            yuanZiVerifyP1SignRequest.setHashAlg("SHA1");
            yuanZiVerifyP1SignRequest.setCert(userCert.getCertBuf().getCertsignBuf());
            YuanZi_VerifyP1SignResponse yuanZiVerifyP1SignResponse = yuanZiService.verifyP1Sign(yuanZiVerifyP1SignRequest);
            System.out.println(yuanZiVerifyP1SignResponse.getCode());

            //本地验签
            boolean verify=false;  
            signature.initVerify(usercert);  //使用公钥初始化签名对象,用于验证签名  
            signature.update(content.getBytes(Charset.forName("UTF-8")));
            verify= signature.verify(dataHadSignature); //得到验证结果  
            System.out.println("本地的验签结果:"+verify);
        } catch (Exception e) {
            e.printStackTrace();
            // TODO: handle exception
        }
    }
```



### 3、PKCS7 的 attached 和 detached 方式的数字签名

+  attached 方式是将签名内容和原文放在一起，按 PKCS7 的格式打包。PKCS7的结构中有一段可以放明文，但明文必需进行ASN.1编码。在进行数字签名验证的同时，提取明文。这里的明文实际上是真正内容的摘要。
+  detached 方式打包的 PKCS7格式包中不包含明文信息。因此在验证的时候，还需要传递明文才能验证成功。同理，这里的明文实际上是真正内容的摘要

从搜索结果来看，detached 方式的应用要频繁得多。