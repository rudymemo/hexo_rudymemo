---
title: Oracle,Java 加解密互通实现（转载）
date: 2021-08-05 11:45:10
mp3: /music/Postiljonen - On the Run.mp3
cover: 'https://i.loli.net/2021/11/09/X8jUcWsLPM4ivrb.jpg'
typora-root-url: ..\..\themes\diaspora\source


---



## 包定义

```sql
CREATE OR REPLACE PACKAGE WX_CRYPTO IS
  -- 定义 DES encrypt 函数
  FUNCTION DES_ENCRYPT(INPUT_STRING IN VARCHAR2) RETURN VARCHAR2;
  FUNCTION DES_ENCRYPT(INPUT_STRING IN VARCHAR2, KEY_STRING IN VARCHAR2)
    RETURN VARCHAR2;

  -- 定义 DES decrypt 函数
  FUNCTION DES_DECRYPT(INPUT_STRING IN VARCHAR2) RETURN VARCHAR2 DETERMINISTIC;
  FUNCTION DES_DECRYPT(INPUT_STRING IN VARCHAR2, KEY_STRING IN VARCHAR2)
    RETURN VARCHAR2 DETERMINISTIC;
END WX_CRYPTO;
```



## 包体定义

```sql
CREATE OR REPLACE PACKAGE BODY WX_CRYPTO IS
  -- 实现 DES encrypt 函数（一个参数）
  FUNCTION DES_ENCRYPT(INPUT_STRING IN VARCHAR2 -- 明文
                       ) RETURN VARCHAR2 AS
  BEGIN
    RETURN WX_CRYPTO.DES_ENCRYPT(INPUT_STRING => INPUT_STRING,
                                 KEY_STRING   => '12345678');
  END;
  -- 实现 DES encrypt 函数（两个参数）
  FUNCTION DES_ENCRYPT(INPUT_STRING IN VARCHAR2, -- 明文
                       KEY_STRING   IN VARCHAR2 -- 密钥
                       ) RETURN VARCHAR2 AS
    V_TEXT        VARCHAR2(4000); -- 明文。长度要是 8 的倍数，若不足 8 的倍数，则用隐藏字符串 chr(0) 补足
    V_TEXT_RAW    RAW(2048); -- 明文
    V_KEY_RAW     RAW(128); -- 密钥
    V_ENCRYPT_RAW RAW(2048); -- 加密后的字符串
  BEGIN
    IF INPUT_STRING IS NULL THEN
      RETURN NULL;
    END IF;
    IF INSTR(INPUT_STRING, '{DES}') = 1 THEN
      RETURN INPUT_STRING;
    END IF;
    -- 向右补足。CHR(0) 隐藏字符串
    V_TEXT := RPAD(INPUT_STRING,
                   (TRUNC(LENGTHB(INPUT_STRING) / 8) + 1) * 8,
                   CHR(0));
    -- 转换成 16 进制
    V_TEXT_RAW    := SYS.UTL_I18N.STRING_TO_RAW(V_TEXT, 'ZHS16GBK');
    V_KEY_RAW     := SYS.UTL_I18N.STRING_TO_RAW(KEY_STRING, 'ZHS16GBK');
    V_ENCRYPT_RAW := SYS.DBMS_OBFUSCATION_TOOLKIT.DESENCRYPT(INPUT => V_TEXT_RAW,
                                                         KEY   => V_KEY_RAW);
    RETURN '{DES}' || RAWTOHEX(V_ENCRYPT_RAW);
  END;

  -- 实现 DES decrypt 函数（一个参数）
  FUNCTION DES_DECRYPT(INPUT_STRING IN VARCHAR2 -- 明文
                       ) RETURN VARCHAR2 DETERMINISTIC AS
  BEGIN
    RETURN WX_CRYPTO.DES_DECRYPT(INPUT_STRING => INPUT_STRING,
                                 KEY_STRING   => '12345678');
  END;
  -- 实现 DES decrypt 函数（两个参数）
  FUNCTION DES_DECRYPT(INPUT_STRING IN VARCHAR2, -- 密文
                       KEY_STRING   IN VARCHAR2 -- 密钥
                       ) RETURN VARCHAR2 DETERMINISTIC AS
    V_TEXT_RAW       RAW(2048); -- 密文
    V_KEY_RAW        RAW(128); -- 密钥
    V_DECRYPT_RAW    RAW(2048); -- 解密后的明文
    V_DECRYPT_STRING VARCHAR2(4000); -- 解密后的明文
  BEGIN
    IF INPUT_STRING IS NULL THEN
      RETURN NULL;
    END IF;
    IF INSTR(INPUT_STRING, '{DES}') = 1 THEN
      V_TEXT_RAW := HEXTORAW(SUBSTR(INPUT_STRING, 6));
      -- 转换成 16 进制
      V_KEY_RAW := SYS.UTL_I18N.STRING_TO_RAW(KEY_STRING, 'ZHS16GBK');
      -- 解密
      V_DECRYPT_RAW := SYS.DBMS_OBFUSCATION_TOOLKIT.DESDECRYPT(INPUT => V_TEXT_RAW,
                                                           KEY   => V_KEY_RAW);
      -- RAW_TO_CHAR 转换成字符串
      V_DECRYPT_STRING := SYS.UTL_I18N.RAW_TO_CHAR(V_DECRYPT_RAW, 'ZHS16GBK');
      -- RTRIM 去除字符串右侧的隐藏字符串 CHR(0)
      RETURN RTRIM(V_DECRYPT_STRING, CHR(0));
    END IF;
    RETURN INPUT_STRING;
  END;
END WX_CRYPTO;
```

> 注意：
>
> 1、sql中默认密钥为`12345678`
>
> 2、加密后会加上：`{DES}`前缀；解密的时候会用`SUBSTR`掉截取掉`{DES}`前缀

# JAVA 加解密实现

## DES 工具类实现

```java
import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;
import javax.crypto.spec.IvParameterSpec;
import java.io.UnsupportedEncodingException;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * Oracle des 加解密 java 实现
 *
 * @author YL
 */
public class Des {
    private static String ALGORITHM_1 = "DES";
    private static String ALGORITHM_2 = "DES/CBC/NoPadding";
    private static String CHARSET = "gbk";

    /**
     * 加密
     *
     * @param str 原始字符
     * @param key 加密 key
     */
    public static String encrypt(String str, String key) {
        try {
            DESKeySpec desKey = new DESKeySpec(key.getBytes(CHARSET));
            SecretKey secretKey = SecretKeyFactory.getInstance(ALGORITHM_1).generateSecret(desKey);

            Cipher cipher = Cipher.getInstance(ALGORITHM_2);
            cipher.init(Cipher.ENCRYPT_MODE, secretKey, new IvParameterSpec(new byte[8]));

            byte[] bytes = str.getBytes(CHARSET);
            System.out.println("src ---> " + Arrays.toString(bytes));
            byte[] inBytes = new byte[((bytes.length / 8) + 1) * 8];
            for (int i = 0; i < bytes.length; i++) {
                inBytes[i] = bytes[i];
            }

            System.out.println("inBytes ---> " + Arrays.toString(inBytes));
            byte[] eBytes = cipher.doFinal(inBytes);
            String hexStr = HexUtils.encodeHexString(eBytes, false);

            System.out.println(str + " encrypted (hex)   :" + hexStr);
            return hexStr;
        } catch (InvalidKeyException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (InvalidKeySpecException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (NoSuchAlgorithmException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (BadPaddingException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (InvalidAlgorithmParameterException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (NoSuchPaddingException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (IllegalBlockSizeException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (UnsupportedEncodingException e) {
            throw new RRException(e.getMessage(), -99, e);
        }
    }

    /**
     * 解密
     *
     * @param str 加密字符
     * @param key 解密 key
     */
    public static String decrypt(String str, String key) {
        try {
            DESKeySpec desKey = new DESKeySpec(key.getBytes(CHARSET));
            SecretKey secretKey = SecretKeyFactory.getInstance(ALGORITHM_1).generateSecret(desKey);

            Cipher cipher = Cipher.getInstance(ALGORITHM_2);
            cipher.init(Cipher.DECRYPT_MODE, secretKey, new IvParameterSpec(new byte[8]));

            byte[] bytes = HexUtils.decodeHex(str);
            byte[] dBytes = cipher.doFinal(bytes);
            System.out.println("decryptBytes ---> " + Arrays.toString(dBytes));

            List<Byte> list = new ArrayList<>();
            for (byte dByte : dBytes) {
                if (dByte != 0) {
                    list.add(dByte);
                }
            }
            byte[] copy = new byte[list.size()];
            for (int i = 0; i < list.size(); i++) {
                copy[i] = list.get(i);
            }

            return new String(copy, CHARSET);
        } catch (InvalidKeyException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (InvalidKeySpecException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (NoSuchAlgorithmException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (BadPaddingException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (InvalidAlgorithmParameterException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (NoSuchPaddingException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (IllegalBlockSizeException e) {
            throw new RRException(e.getMessage(), -99, e);
        } catch (UnsupportedEncodingException e) {
            throw new RRException(e.getMessage(), -99, e);
        }
    }
}
```

## HexUtils 工具类

```java
/**
 * hex 编码
 *
 * @author YL
 */
public abstract class HexUtils {
    private HexUtils() {}

    /**
     * Used to build output as Hex
     */
    private static final char[] DIGITS_LOWER =
            {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};

    /**
     * Used to build output as Hex
     */
    private static final char[] DIGITS_UPPER =
            {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};

    /**
     * 将表示十六进制值的String转换为这些相同值的字节数组
     *
     * @param data 十六进制数字的字符串
     *
     * @return 包含从提供的char数组解码的二进制数据的字节数组
     */
    public static byte[] decodeHex(final String data) {
        return decodeHex(data.toCharArray());
    }

    /**
     * 将表示十六进制值的字符数组转换为这些相同值的字节数组
     *
     * @param data 十六进制数字的字符数组
     *
     * @return 包含从提供的char数组解码的二进制数据的字节数组
     */
    public static byte[] decodeHex(final char[] data) {
        final int len = data.length;

        if ((len & 0x01) != 0) {
            throw new IllegalArgumentException("Odd number of characters.");
        }

        final byte[] out = new byte[len >> 1];

        // two characters form the hex value.
        for (int i = 0, j = 0; j < len; i++) {
            int f = toDigit(data[j], j) << 4;
            j++;
            f = f | toDigit(data[j], j);
            j++;
            out[i] = (byte) (f & 0xFF);
        }
        return out;
    }

    /**
     * 将字节数组转换为字符数组，按顺序表示每个字节的十六进制值
     *
     * @param data 需要转换为十六进制字符数组的 byte 数组
     *
     * @return 十六进制字符数组
     */
    public static char[] encodeHex(final byte[] data) {
        return encodeHex(data, true);
    }

    /**
     * Converts an array of bytes into an array of characters representing the hexadecimal values of each byte in order.
     * The returned array will be double the length of the passed array, as it takes two characters to represent any
     * given byte.
     *
     * @param data        a byte[] to convert to Hex characters
     * @param toLowerCase <code>true</code> converts to lowercase, <code>false</code> to uppercase
     *
     * @return A char[] containing hexadecimal characters in the selected case
     */
    public static char[] encodeHex(final byte[] data, final boolean toLowerCase) {
        return encodeHex(data, toLowerCase ? DIGITS_LOWER : DIGITS_UPPER);
    }

    /**
     * Converts an array of bytes into an array of characters representing the hexadecimal values of each byte in order.
     * The returned array will be double the length of the passed array, as it takes two characters to represent any
     * given byte.
     *
     * @param data     a byte[] to convert to Hex characters
     * @param toDigits the output alphabet (must contain at least 16 chars)
     *
     * @return A char[] containing the appropriate characters from the alphabet
     * For best results, this should be either upper- or lower-case hex.
     */
    private static char[] encodeHex(final byte[] data, final char[] toDigits) {
        final int l = data.length;
        final char[] out = new char[l << 1];
        // two characters form the hex value.
        for (int i = 0, j = 0; i < l; i++) {
            out[j++] = toDigits[(0xF0 & data[i]) >>> 4];
            out[j++] = toDigits[0x0F & data[i]];
        }
        return out;
    }

    /**
     * Converts an array of bytes into a String representing the hexadecimal values of each byte in order. The returned
     * String will be double the length of the passed array, as it takes two characters to represent any given byte.
     *
     * @param data a byte[] to convert to Hex characters
     *
     * @return A String containing lower-case hexadecimal characters
     */
    public static String encodeHexString(final byte[] data) {
        return encodeHexString(data, true);
    }

    /**
     * Converts an array of bytes into a String representing the hexadecimal values of each byte in order. The returned
     * String will be double the length of the passed array, as it takes two characters to represent any given byte.
     *
     * @param data        a byte[] to convert to Hex characters
     * @param toLowerCase <code>true</code> converts to lowercase, <code>false</code> to uppercase
     *
     * @return A String containing lower-case hexadecimal characters
     */
    public static String encodeHexString(final byte[] data, final boolean toLowerCase) {
        return new String(encodeHex(data, toLowerCase));
    }

    /**
     * Converts a hexadecimal character to an integer.
     *
     * @param ch    A character to convert to an integer digit
     * @param index The index of the character in the source
     *
     * @return An integer
     */
    private static int toDigit(final char ch, final int index) {
        final int digit = Character.digit(ch, 16);
        if (digit == -1) {
            throw new IllegalArgumentException("Illegal hexadecimal character " + ch + " at index " + index);
        }
        return digit;
    }
}
```

## 测试

> Oracle

```sql
-- Oracle 加密
SELECT WX_CRYPTO.DES_ENCRYPT('测试123！@#。【】[] ', '7agyrpho') FROM DUAL;
-- 输出
{DES}EBD9F4F8AC29DDFCD74FD951A2B84F064D5CD6976143B08644AC7B62569BB4E6

-- Oracle 解密
SELECT WX_CRYPTO.DES_DECRYPT('{DES}EBD9F4F8AC29DDFCD74FD951A2B84F064D5CD6976143B08644AC7B62569BB4E6',
                             '7agyrpho')
  FROM DUAL;
-- 输出
测试123！@#。【】[] 


-- Oracle 解密 Java 的密文
SELECT WX_CRYPTO.DES_DECRYPT('{DES}EBD9F4F8AC29DDFCD74FD951A2B84F064D5CD6976143B086',
                             '7agyrpho')
  FROM DUAL;
-- 输出
测试123！@#。【】[]
```

> Java

```java
import org.junit.Test;

public class DesTest {

    @Test
    public void test() {
        String str = "测试123！@#。【】[] ";
        String key = "7agyrpho";
        // Java 加密
        String ee = Des.encrypt(str, key);
        System.out.println("ee ---> " + ee);
        // Java 解密
        String dd = Des.decrypt(ee, key);
        System.out.println("dd ---> " + dd);
    }
}
// 日志输出
ee ---> EBD9F4F8AC29DDFCD74FD951A2B84F064D5CD6976143B086
dd ---> 测试123！@#。【】[] 

// Java 解密 Oracle 的密文
String dd = Des.decrypt("EBD9F4F8AC29DDFCD74FD951A2B84F064D5CD6976143B08644AC7B62569BB4E6", "7agyrpho");
System.out.println("dd ---> " + dd);
// 日志输出
dd ---> 测试123！@#。【】[]
```

- **本文作者：** forever杨
- **转载链接：** https://blog.yl-online.top/posts/5145849a.html
- **版权声明：** 本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。如果文章内容对你有用，请记录到你的笔记中。本博客站点随时会停止服务，请不要收藏、转载！

