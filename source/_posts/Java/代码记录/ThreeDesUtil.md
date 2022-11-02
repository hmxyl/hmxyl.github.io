```java
package com.dayainfo.ssp.user.cas.service;

import org.apache.commons.codec.binary.Base64;
import org.springframework.stereotype.Component;

import javax.crypto.Cipher;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESedeKeySpec;
import javax.crypto.spec.IvParameterSpec;
import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.NoSuchAlgorithmException;
import java.security.Security;
import java.security.spec.InvalidKeySpecException;

/**
 * 3des工具类
 *
 * @author: DH
 * @date: 2022/4/9
 * @desc:
 */
@Component
public class ThreeDesUtil {
    private static final String UTF_8 = "UTF-8";
    // 翻转IV
    private static final String IV = "12345678";
    private static final String ALGORITHM = "DESede";
    //算法/工作模式/填充模式
    private static final String DESEDE_CBC_PADDING = "DESede/CBC/PKCS7Padding";
    // 双方协商约定密钥
    private static final String DES_KEY = "TesePassKey";

    static {
        Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider());
    }

    /**
     * 加密：
     * <p>第一步：3Des加密(CBC工作模式)</p>
     * <p>第二步：Base64加密</p>
     * <p>第三步：UrlSafe处理</p>
     *
     * @param plainText
     * @return
     * @throws Exception
     */
    public String encode(String plainText) throws Exception {
        // 第一步：3Des加密(CBC工作模式)
        byte[] byteCBC = encodeByCBC(plainText);
        // 第二步：Base64加密
        byte[] byteBase64 = Base64.encodeBase64(byteCBC);
        // 第三步：UrlSafe处理
        return UrlSafe(new String(byteBase64, UTF_8));
    }


    /**
     * 解密
     * <p>第一步：convertUrlSafe处理</p>
     * <p>第二步：Base64解密</p>
     * <p>第三步：3Des解密(CBC工作模式)</p>
     *
     * @param encodeStr
     * @return
     * @throws Exception
     */
    public String decode(String encodeStr) throws Exception {
        // 第一步：convertUrlSafe处理
        String result = convertUrlSafe(encodeStr);
        // 第二步：Base64解密
        byte[] byteBase64 = Base64.decodeBase64(result.getBytes(UTF_8));
        // 第三步：3Des解密(CBC工作模式)
        return decodeByCBC(byteBase64);
    }

    /**
     * 3Des加密(CBC工作模式)
     */
    private byte[] encodeByCBC(String dataStr) throws Exception {
        byte[] data = dataStr.getBytes(UTF_8);
        //获取SecretKey对象
        Key secretKey = getSecretKey();
        //获取指定转换的密码对象Cipher（参数：算法/工作模式/填充模式）
        Cipher cipher = Cipher.getInstance(DESEDE_CBC_PADDING);
        //创建向量参数规范也就是初始化向量:keyIv长度必须等于8
        IvParameterSpec ips = new IvParameterSpec(IV.getBytes());
        //用密钥和一组算法参数规范初始化此Cipher对象（加密模式）
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, ips);
        //执行加密操作
        return cipher.doFinal(data);
    }

    /**
     * 3Des解密(CCB工作模式)
     */
    private String decodeByCBC(byte[] data) throws Exception {
        //获取SecretKey对象
        Key secretKey = getSecretKey();
        //获取指定转换的密码对象Cipher（参数：算法/工作模式/填充模式）
        Cipher cipher = Cipher.getInstance(DESEDE_CBC_PADDING);
        //创建向量参数规范也就是初始化向量:keyIv长度必须等于8
        IvParameterSpec ips = new IvParameterSpec(IV.getBytes());
        //用密钥和一组算法参数规范初始化此Cipher对象（加密模式）
        cipher.init(Cipher.DECRYPT_MODE, secretKey, ips);
        //执行加密操作
        return new String(cipher.doFinal(data), UTF_8);
    }

    private Key getSecretKey() throws InvalidKeyException, NoSuchAlgorithmException, InvalidKeySpecException, UnsupportedEncodingException {
        // 加密字节固定为24位，不足留空，超过弃用
        final byte[] key = new byte[24];
        final byte[] tempKey = DES_KEY.getBytes(UTF_8);
        for (int i = 0; i < tempKey.length; i++) {
            key[i] = tempKey[i];
        }
        //创建一个DESedeKeySpec密钥规范对象
        DESedeKeySpec spec = new DESedeKeySpec(key);
        //转换指定算法的密钥的SecretKeyFactory对象
        SecretKeyFactory keyfactory = SecretKeyFactory.getInstance(ALGORITHM);
        //根据提供的密码密钥规范生成SecretKey对象。
        return keyfactory.generateSecret(spec);
    }

    /**
     * 接口获取token参数还原处理
     *
     * @param returnValue
     * @return
     */
    private static String convertUrlSafe(String returnValue) {
        String incoming = returnValue.replace('_', '/').replace('-', '+');
        switch (returnValue.length() % 4) {
            case 2:
                incoming += "==";
                break;
            case 3:
                incoming += "=";
                break;
        }
        return incoming;
    }

    /**
     * 文档定义方法
     *
     * @param str
     * @return
     */
    private String UrlSafe(String str) {
        return trimEnd(str, "=").replace('+', '-').replace('/', '_');
    }

    /*
     * 删除末尾字符串
     */
    public String trimEnd(String inStr, String suffix) {
        while (inStr.endsWith(suffix)) {
            inStr = inStr.substring(0, inStr.length() - suffix.length());
        }
        return inStr;
    }

}
```

