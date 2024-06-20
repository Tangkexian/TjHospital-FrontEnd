package com.javaee.onlinehosbackend.extrautil.utils;

/**
 * @Description 位操作工具类
 * @Author Sugar
 * @Date 2020/9/8 16:30
 */
public class BitUtil {
    private static final int MASK = 0xFF;
    private static final int INT16_MAXIMUM = Short.MAX_VALUE - Short.MIN_VALUE + 1;

    /**
     * 将byte转换为一个长度为8的byte数组，数组每个值代表bit
     */
    public static byte[] byteToBit(byte b) {
        byte[] array = new byte[8];
        for (int i = 7; i >= 0; i--) {
            array[i] = (byte) (b & 1);//获取最低位
            b = (byte) (b >> 1);//每次右移一位
        }
        return array;
    }

    public static byte intToByte(final int i) {
        return (byte) (i & MASK);
    }

    public static byte[] intTo1Byte(final int i) {
        return new byte[]{intToByte(i)};
    }

    public static int intFrom1Byte(final byte[] bytes, final int startIndex) {
        return bytes[startIndex] & MASK;
    }

    public static int intFrom1Byte(final byte b) {
        return b & MASK;
    }

    public static byte[] intTo2Bytes(final int value) {
        return new byte[]{
                (byte) ((value >>> 8) & MASK),
                (byte) (value & MASK)
        };
    }

    public static short shortFrom2Bytes(final byte[] bytes, final int startIndex) {
        int val = ((bytes[startIndex] & MASK) << 8) | (bytes[startIndex + 1] & MASK);
        if (val > Short.MAX_VALUE) {
            return (short) (val - INT16_MAXIMUM);
        }
        return (short) val;
    }

    public static short shortFrom2Bytes(final byte[] bytes) {
        return shortFrom2Bytes(bytes, 0);
    }

    public static int intFrom2Bytes(final byte[] bytes, final int startIndex) {
        return ((bytes[startIndex] & MASK) << 8) | (bytes[startIndex + 1] & MASK);
    }

    public static int intFrom2Bytes(final byte[] bytes) {
        return intFrom2Bytes(bytes, 0);
    }

    public static byte[] intTo3Bytes(int value) {
        return new byte[]{
                (byte) ((value >>> 16) & MASK),
                (byte) ((value >>> 8) & MASK),
                (byte) (value & MASK)
        };
    }

    public static int intFrom3Bytes(final byte[] bytes, final int startIndex) {
        return ((bytes[startIndex] & MASK) << 16)
                |
                ((bytes[startIndex + 1] & MASK) << 8)
                |
                ((bytes[startIndex + 2] & MASK));
    }

    public static int intFrom3Bytes(final byte[] bytes) {
        return intFrom3Bytes(bytes, 0);
    }

    public static byte[] intTo4Bytes(final int value) {
        return new byte[]{
                (byte) ((value >>> 24) & MASK),
                (byte) ((value >>> 16) & MASK),
                (byte) ((value >>> 8) & MASK),
                (byte) (value & MASK)
        };
    }

    public static int intFrom4Bytes(final byte[] bytes) {
        return intFrom4Bytes(bytes, 0);
    }

    public static int intFrom4Bytes(final byte[] bytes, final int startIndex) {
        return ((bytes[startIndex] & MASK) << 24)
                |
                ((bytes[startIndex + 1] & MASK) << 16)
                |
                ((bytes[startIndex + 2] & MASK) << 8)
                |
                ((bytes[startIndex + 3] & MASK));
    }

    public static int intFromBytes(final byte[] bytes, int start, int len) {
        switch (len) {
            case 1:
                return intFrom1Byte(bytes, start);
            case 2:
                return intFrom2Bytes(bytes, start);
            case 3:
                return intFrom3Bytes(bytes, start);
            case 4:
                return intFrom4Bytes(bytes, start);
            default: {
                throw new IllegalArgumentException("len ∈ [1,4]");
            }
        }
    }

    public static int intFromBytes(final byte[] bytes) {
        return intFromBytes(bytes, 0, bytes.length);
    }

    public static final byte[] intTo32Bit(int value) {
        return intToBits(value, 32);
    }

    public static final byte[] longTo64Bit(long value) {
        return DigitUtil.longToDigit(value, DigitUtil.SHIFT.BINARY, 64);
    }

    public static final String intTo32BitString(int value) {
        return bitsToString(intTo32Bit(value));
    }

    public static final String longTo64BitString(long value) {
        return bitsToString(longTo64Bit(value));
    }

    public static final byte[] intTo16Bit(int value) {
        return intToBits(value, 16);
    }

    public static final String intTo16BitString(int value) {
        return bitsToString(intTo16Bit(value));
    }

    public static final byte[] intTo8Bit(int value) {
        return intToBits(value, 8);
    }

    public static final String intTo8BitString(int value) {
        return bitsToString(intTo8Bit(value));
    }

    /**
     * 将int转成二进制
     *
     * @param val
     * @return
     */
    public static final byte[] intToBits(int val) {
        return intToBits(val, 1);
    }

    /**
     * 将int转成二进制
     *
     * @param val
     * @param minLen
     * @return
     */
    public static final byte[] intToBits(int val, int minLen) {
        return DigitUtil.intToDigit(val, DigitUtil.SHIFT.BINARY, minLen);
    }

    public static final String bitsToString(byte[] bits) {
        StringBuilder str = new StringBuilder(bits.length);
        for (byte b : bits) {
            str.append(b);
        }
        return str.toString();
    }

    public static final String bitsToStringWithFormat(byte[] bits) {
        StringBuilder str = new StringBuilder(bits.length);
        int n = bits.length % 4;
        for (int i = 0; i < bits.length; i++) {
            if (i > 0 && n + i % 4 == 0) {
                str.append(",");
            }
            str.append(bits[i]);
        }
        return str.toString();
    }

    /**
     * 把byte转为字符串的bit
     *
     * @param b
     * @return
     */
    public static String byteToBitString(byte b) {
        return ""
                + (byte) ((b >> 7) & 0x1) + (byte) ((b >> 6) & 0x1)
                + (byte) ((b >> 5) & 0x1) + (byte) ((b >> 4) & 0x1)
                + (byte) ((b >> 3) & 0x1) + (byte) ((b >> 2) & 0x1)
                + (byte) ((b >> 1) & 0x1) + (byte) ((b >> 0) & 0x1);
    }

    /**
     * 获取一个字节的第m到第n位
     *
     * @param value
     * @param start >0
     * @param end   >0
     * @return
     */
    public static byte[] getBitRange(byte value, int start, int end) {
        byte[] rangeArray = new byte[end - start + 1];
        if (start > 7 || start < 0) {
            throw new RuntimeException("illegal start param");
        }
        if (end > 7 || end < 0) {
            throw new RuntimeException("illegal end param");
        }
        if (start > end) {
            throw new RuntimeException("start can not bigger than end");
        }
        if (start == end) {
            rangeArray[0] = (byte) getBitAsInt(value, start);
            return rangeArray;
        }
        for (int i = end; i < start; i--) {
            rangeArray[i] = (byte) getBitAsInt(value, start);
        }
        return rangeArray;
    }

    /**
     * bit转Byte
     */
    public static byte bitToByte(String bits) {
        if (bits == null) {
            return 0;
        }
        int len = bits.length();
        if (len != 4 && len != 8) {
            return 0;
        }
        if (len == 8) {// 8 bit处理
            if (bits.charAt(0) == '0') {// 正数
                return (byte) Integer.parseInt(bits, 2);
            } else {// 负数
                return (byte) (Integer.parseInt(bits, 2) - 256);
            }
        } else {//4 bit处理
            return (byte) Integer.parseInt(bits, 2);
        }
    }

    public static final int bitsToInt(byte[] bits) {
        return Integer.valueOf(bitsToString(bits), 2);
    }

    public static final byte[] reverse(byte[] bits) {
        byte[] newBits = new byte[bits.length];
        for (int i = 0; i < bits.length; i++) {
            newBits[i] = bits[bits.length - 1 - i];
        }
        return newBits;
    }

    /**
     * 从int类型数据中取指定位的值
     *
     * @param value
     * @param bitIndex 位索引
     * @param length   位长度
     * @return
     */
    public static int getBitRangeAsInt(int value, int bitIndex, int length) {
        return (value << (Integer.SIZE - (bitIndex + length))) >> (Integer.SIZE - length);
    }

    public static int getBitRangeAsUnsignedInt(int value, int bitIndex, int length) {
        return (value & getMark(bitIndex, length)) >> bitIndex;
    }

    public static int getBitRangeAsInt(long value, int bitIndex, int length) {
        return (int) ((value << (Long.SIZE - (bitIndex + length))) >> (Long.SIZE - length));
    }

    public static int getBitRangeAsUnsignedInt(long value, int bitIndex, int length) {
        return (int) ((value & getMarkForLong(bitIndex, length)) >> bitIndex);
    }

    public static int getBitRangeAsInt(int value, int length) {
        return (value << (Integer.SIZE - length)) >> (Integer.SIZE - length);
    }

    public static int getBitRangeAsUnsignedInt(int value, int length) {
        return value & getMark(0, length);
    }

    public static int getBitRangeAsInt(long value, int length) {
        return (int) ((value << Long.SIZE - length) >> (Long.SIZE - length));
    }

    public static int getBitRangeAsUnsignedInt(long value, int length) {
        return (int) (value & getMarkForLong(0, length));
    }

    public static int getBitAsInt(int value, int bitIndex) {
        return (value >> bitIndex) & 0x1;
    }

    public static int getBitAsInt(long value, int bitIndex) {
        return (int) ((value >> bitIndex) & 0x1);
    }

    public static long getBitRangeAsLong(long value, int bitIndex, int length) {
        return (value << (Long.SIZE - (bitIndex + length))) >> (Long.SIZE - length);
    }

    public static long getBitRangeAsUnsignedLong(long value, int bitIndex, int length) {
        return (value & getMarkForLong(bitIndex, length)) >> bitIndex;
    }

    public static long getBitRangeAsLong(long value, int length) {
        return (value << (Long.SIZE - length)) >>> (Long.SIZE - length);
    }

    public static long getBitRangeAsUnsignedLong(long value, int length) {
        return value & getMarkForLong(0, length);
    }

    /**
     * 将指定位的值设置到int中
     *
     * @param bitValue
     * @param bitIndex 位索引
     * @param length   位长度
     * @return
     */
    public static int setBitAt(int bitValue, int bitIndex, int length) {
        return bitValue << bitIndex & getMark(bitIndex, length);
    }

    public static int setBitAt(int bitValue, int bitIndex) {
        return bitValue << bitIndex;
    }

    public static long setBitToLong(long bitValue, int bitIndex, int length) {
        return bitValue << bitIndex & getMarkForLong(bitIndex, length);
    }

    public static long setBitToLong(long bitValue, int bitIndex) {
        return bitValue << bitIndex;
    }

    public static int getMark(int bitIndex, int length) {
        if (bitIndex + length > 32) {
            throw new IndexOutOfBoundsException("位的长度超过int值范围");
        }
        int i = 0;
        int sum = 0;
        if (bitIndex == 0) {
            sum = 1;
            i = 1;
        }
        for (; i < length; i++) {
            sum |= (2 << (bitIndex + i - 1));
        }
        return sum;
    }

    public static long getMarkForLong(int bitIndex, int length) {
        int i = 0;
        long sum = 0;
        if (bitIndex == 0) {
            sum = 1L;
            i = 1;
        }
        for (; i < length; i++) {
            sum |= (2L << (bitIndex + i - 1));
        }
        return sum;
    }
}
