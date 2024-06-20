package com.javaee.onlinehosbackend.extrautil.utils;

import lombok.NonNull;

/**
 * @Description BCD操作工具类
 * @Author Sugar
 * @Date 2020/9/8 16:05
 */
public class BcdUtil {

    public static String bytes2BcdString(@NonNull final byte[] bytes, final int start, final int length) {
        return bcd2String(bytes, start, start + length);
    }

    public static String bcd2String(final byte[] bytes, final int start, final int end) {
        assert start < end : "start < end";

        final int length = end - start;
        final StringBuilder builder = new StringBuilder(length << 2);
        for (int i = start; i < end; i++) {
            builder.append((bytes[i] & 0xf0) >>> 4);// 高四位
            builder.append(bytes[i] & 0x0f);// 低四位
        }
        return "0".equalsIgnoreCase(builder.toString().substring(0, 1))
                ? builder.toString().substring(1)
                : builder.toString();
    }


    /**
     * BCD字节数组===>String
     *
     * @param bytes
     * @return 十进制字符串
     */
    public static String bcd2String(byte[] bytes) {
        return bcd2String(bytes, 0, bytes.length);
    }

    /**
     * 字符串==>BCD字节数组
     *
     * @param str
     * @return BCD字节数组
     */
    public static byte[] string2Bcd(String str) {
        // 奇数,前补零
        if ((str.length() & 0x1) == 1) {
            str = "0" + str;
        }

        byte ret[] = new byte[str.length() / 2];
        byte bs[] = str.getBytes();
        for (int i = 0; i < ret.length; i++) {

            byte high = ascii2Bcd(bs[2 * i]);
            byte low = ascii2Bcd(bs[2 * i + 1]);

            // TODO 只遮罩BCD低四位?
            ret[i] = (byte) ((high << 4) | low);
        }
        return ret;
    }

    private static byte ascii2Bcd(byte asc) {
        if ((asc >= '0') && (asc <= '9'))
            return (byte) (asc - '0');
        else if ((asc >= 'A') && (asc <= 'F'))
            return (byte) (asc - 'A' + 10);
        else if ((asc >= 'a') && (asc <= 'f'))
            return (byte) (asc - 'a' + 10);
        else
            return (byte) (asc - 48);
    }
}
