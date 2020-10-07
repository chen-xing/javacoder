## java中的int与byte的转化

### 1、基础准备

##### 1.1、原码

就是二进制码，最高位为符号位，0表示正数，1表示负数，剩余部分表示真值



##### 1.2、反码

在原码的基础上，正数反码就是他本身，负数除符号位之外全部按位取反



##### 1.3、补码

正数的补码就是自己本身， 负数的补码是在自身反码的基础上加1



### 2、对应到java操作

```
&（与）, | （或）, ^ （异或），~ （按位取反）
& :当2个都为1的时候为1， 其他都是0 。 1&1 = 1， 1&0 = 0， 0&0 = 0； 他的作用是清0
| : 当2个只要有一个为1，就是1. 1|0 = 1; 0|0 = 0,  1|1 = 1;
^: 相同为0， 不相同为1， 1^0 = 1, 1^1 = 0,  0^0 = 0; 他的作用是定位翻转。
~: 按位取反，0变为1， 1变为0；
```



**举例说明**

+ 之所以要明确原码，反码，补码，**是因为java中变量都是以补码的形式保存的。**
+ 比如 整行30 他的原码是：0001 1110. 正数，所以反码，补码都是0001 1110. 
+ 对于负数：-7 ，他的原码是 1000 0111， 第一位1表示是此数是负数。他的反码是：1111 1000, 补码在反码的基础上加1， 所以它的补码是1111 1001， 所以他的二进制数就是1111 1001



**java为什么采用补码**

+ 如果用源码，那么0000 0000 和1000 0000 貌似都0， +0 ， 和- 0. 所以这造成了问题
+ cpu计算器只有加法没有减法， 减法需要用正数和负数相加得到**

### 3、oxff截取操作

```
oxff
16进制的255，2进制的11111111，&oxff后的作用我认为是，得到低8位
比如：src[0] =  (byte) ((value>>8) & 0xFF)
```



### 4、int转byte

```
 /**
     * int到byte[] 由高位到低位
     * @param i 需要转换为byte数组的整行值。
     * @return byte数组
     */
    public static byte[] intToByteArray(int i) {
        byte[] result = new byte[4];
        result[0] = (byte)((i >> 24) & 0xFF);
        result[1] = (byte)((i >> 16) & 0xFF);
        result[2] = (byte)((i >> 8) & 0xFF);
        result[3] = (byte)(i & 0xFF);
        return result;
    }

    /**
     * byte[]转int
     * @param bytes 需要转换成int的数组
     * @return int值
     */
    public static int byteArrayToInt(byte[] bytes) {
        int value=0;
        for(int i = 0; i < 4; i++) {
            int shift= (3-i) * 8;
            value +=(bytes[i] & 0xFF) << shift;
        }
        return value;
    }
```



![人工博客](http://oss.94rg.com/oneblog/20200314112023114.jpg-94rg002)

### 5、后续

更多精彩，敬请关注， [ 程序员导航网](https://chenzhuofan.top)  [https://chenzhuofan.top](https://chenzhuofan.top)