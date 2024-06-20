package com.javaee.onlinehosbackend.extrautil.utils;

/**
 * @Description
 * @Author sugar
 * @Date 2020/10/20 12:18
 */
public class DigitUtil {
    private final static char[] DIGITS = {
            '0' , '1' , '2' , '3' , '4' , '5' ,
            '6' , '7' , '8' , '9' , 'A' , 'B' ,
            'C' , 'D' , 'E' , 'F' , 'G' , 'H' ,
            'I' , 'J' , 'K' , 'L' , 'M' , 'N' ,
            'O' , 'P' , 'Q' , 'R' , 'S' , 'T' ,
            'U' , 'V' , 'W' , 'X' , 'Y' , 'Z'
    };

    public static final class SHIFT {
        public static final int BINARY = 1;
        public static final int OCTAL = 3;
        public static final int HEX = 4;
    }

    /**
     *
     * @param val
     * @param shift hex:4, octal:3, binary:1
     * @param minLen
     * @return
     */
    public static final byte[] intToDigit(int val, int shift, int minLen) {
        int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
        int size = Math.max(((mag + (shift - 1)) / shift), minLen);
        int radix = 1 << shift;
        int mask = radix - 1;
        byte[] bits = new byte[size];
        int index = size;
        do {
            bits[--index] = (byte)(val & mask);
            val >>>= shift;
        } while (val != 0 && index > 0);
        return bits;
    }

    public static final byte[] longToDigit(long val, int shift, int minLen) {
        int mag = Long.SIZE - Long.numberOfLeadingZeros(val);
        int size = Math.max(((mag + (shift - 1)) / shift), minLen);
        int radix = 1 << shift;
        int mask = radix - 1;
        byte[] bits = new byte[size];
        int index = size;
        do {
            bits[--index] = (byte)(val & mask);
            val >>>= shift;
        } while (val != 0 && index > 0);
        return bits;
    }

    public static final String intToDigitString(int val, int shift, int minLen) {
        byte[] digits = intToDigit(val, shift, minLen);
        StringBuffer str = new StringBuffer(digits.length);
        for (byte digit : digits) {
            str.append(DIGITS[digit]);
        }
        return str.toString();
    }

//    public static int log(int x, int y) {
//        return (int)(Math.log(x) / Math.log(y));
//    }
}
