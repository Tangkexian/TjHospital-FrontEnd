package com.javaee.onlinehosbackend.extrautil.utils;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.UnsupportedEncodingException;
import java.math.BigDecimal;
import java.net.URLDecoder;
import java.net.URLEncoder;
import java.sql.Timestamp;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.Map.Entry;

/**
 * 常用方法工具类
 * @Author Sugar
 * @Version 2016年12月19日 下午5:45:25
 */
public class Utils {
    private static Logger logger = LoggerFactory.getLogger(Utils.class);

    public static final String REG_MOBILE = "^(86)?1[3456789]\\d{9}$";// 手机号正则表达试
    public static final String REG_TEL = "^(0\\d{2,3}-?\\d{7,8}(-\\d{3,4})?)|(400\\d{7})$";// 验证座机
    public static final String REG_MAIL = "^\\w+([\\-+\\.]\\w+)*@\\w+([\\-\\.]\\w+)*\\.\\w+([\\-\\.]\\w+)*$";// 验证邮箱

    public static final int MILLISECOND = 0;
    public static final int SECOND = 1;
    public static final int MINUTE = 2;
    public static final int HOUR = 3;
    public static final int DAY = 4;

    public static byte[] hexStringToBytes(String hexString) {
        if (hexString == null || hexString.equals("")) {
            return null;
        }
        hexString = hexString.toUpperCase();
        int length = hexString.length() / 2;
        char[] hexChars = hexString.toCharArray();
        byte[] d = new byte[length];
        for (int i = 0; i < length; i++) {
            int pos = i * 2;
            d[i] = (byte) (charToByte(hexChars[pos]) << 4 | charToByte(hexChars[pos + 1]));
        }
        return d;
    }

    public static String bytesToHexString(byte[] src) {
        StringBuilder stringBuilder = new StringBuilder("");
        if (src == null || src.length <= 0) {
            return null;
        }
        for (int i = 0; i < src.length; i++) {
            int v = src[i] & 0xFF;
            String hv = Integer.toHexString(v);
            if (hv.length() < 2) {
                stringBuilder.append(0);
            }
            stringBuilder.append(hv);
        }
        return stringBuilder.toString();
    }

    private static byte charToByte(char c) {
        return (byte) "0123456789ABCDEF" .indexOf(c);
    }

    public static boolean isEmpty(String str) {
        if (str == null || str.equals("")) {
            return true;
        }
        return false;
    }

    /**
     * 返回不为空的字符串
     *
     * @param obj
     * @return
     */
    public static String getString(Object obj) {
        if (obj == null || obj.toString().equals("null")) {
            return "";
        }
        return obj.toString();
    }

    /**
     * 返回不为空的字符串
     *
     * @param obj
     * @param def 默认值
     * @return
     */
    public static String getString(Object obj, String def) {
        if (obj == null || obj.toString().equals("null")) {
            return def;
        }
        return obj.toString().trim();
    }

    /**
     * 返回给定格式的服务器时间
     *
     * @param parrent String 默认返回yyyy-MM-dd HH:mm:ss格式的时间
     * @return
     */
    public static String getServerTime(String parrent) {
        if (parrent == null || parrent.equals("")) {
            parrent = "yyyy-MM-dd HH:mm:ss";
        }
        try {
            Calendar c = Calendar.getInstance(Locale.CHINESE);
            SimpleDateFormat sformat = new SimpleDateFormat(parrent, Locale.CHINA);
            return sformat.format(c.getTime());
        } catch (Exception e) {
            e.printStackTrace();
            return (new Timestamp(System.currentTimeMillis()) + "").substring(0, 19);
        }
    }

    /**
     * 返回给定格式的服务器时间
     *
     * @param parrent String 默认返回yyyy-MM-dd HH:mm:ss格式的时间
     * @param field   the calendar field,eg:Calendar.WEEK_OF_YEAR
     * @param offset  the amount of date or time to be added to the field, eg:-2
     * @return
     */
    public static String getServerTime(String parrent, int field, int offset) {
        if (parrent == null || parrent.equals("")) {
            parrent = "yyyy-MM-dd HH:mm:ss";
        }
        try {
            Calendar c = Calendar.getInstance(Locale.CHINESE);
            SimpleDateFormat sformat = new SimpleDateFormat(parrent);
            c.add(field, offset);
            return sformat.format(c.getTime());
        } catch (Exception e) {
            e.printStackTrace();
            return (new Timestamp(System.currentTimeMillis()) + "").substring(0, 19);
        }
    }

    /**
     * 计算时间，返回yyyy-MM-dd HH:mm:ss
     *
     * @param datetime String 原始时间
     * @param field    the calendar field,eg:Calendar.WEEK_OF_YEAR
     * @param offset   the amount of date or time to be added to the field, eg:-2
     * @return
     */
    public static String getDatetime(String datetime, int field, int offset) {
        String parrent = "yyyy-MM-dd HH:mm:ss";
        try {
            SimpleDateFormat sformat = new SimpleDateFormat(parrent);
            Calendar c = Calendar.getInstance(Locale.CHINESE);
            c.setTime(sformat.parse(datetime));
            c.add(field, offset);
            return sformat.format(c.getTime());
        } catch (Exception e) {
            e.printStackTrace();
            return datetime;
        }
    }

    /**
     * 计算日期，返回yyyy-MM-dd
     *
     * @param date   String 原始时间
     * @param field  the calendar field,eg:Calendar.WEEK_OF_YEAR
     * @param offset the amount of date or time to be added to the field, eg:-2
     * @return
     */
    public static String getDate(String date, int field, int offset) {
        String parrent = "yyyy-MM-dd";
        try {
            date = date.substring(0, 10);
            SimpleDateFormat sformat = new SimpleDateFormat(parrent);
            Calendar c = Calendar.getInstance(Locale.CHINESE);
            c.setTime(sformat.parse(date));
            c.add(field, offset);
            return sformat.format(c.getTime());
        } catch (Exception e) {
            e.printStackTrace();
            return date;
        }
    }

    public static double parseDouble(Object str) {
        if (str == null) {
            return 0;
        }
        String s = str.toString().trim();
        if (!s.matches("-?\\d+(\\.\\d+)?")) {
            return 0;
        }
        return Double.parseDouble(s);
    }

    public static float parseFloat(Object str) {
        if (str == null) {
            return 0;
        }
        String s = str.toString().trim();
        if (!s.matches("-?\\d+(\\.\\d+)?")) {
            return 0;
        }
        return Float.parseFloat(s);
    }

    public static int parseInt(Object str) {
        return parseInt(str, 0);
    }

    public static int parseInt(Object str, int defaultValue) {
        if (str == null || str.equals("")) {
            return defaultValue;
        }
        String s = str.toString().trim();
        if (s.matches("-?\\d+(\\.\\d+)")) {
            return Integer.parseInt(s.substring(0, s.indexOf(".")));
        }
        if (!s.matches("-?\\d+")) {
            return defaultValue;
        }
        return Integer.parseInt(s);
    }

    public static long parseLong(Object str) {
        if (str == null || str.equals("")) {
            return 0;
        }
        String s = str.toString().trim();
        if (s.matches("-?\\d+(\\.\\d+)")) {
            return Long.parseLong(s.substring(0, s.indexOf(".")));
        }
        if (!s.matches("-?\\d+")) {
            return 0;
        }
        return Long.parseLong(s);
    }

    /**
     * 左补字符0
     *
     * @param str
     * @param length
     * @return
     */
    public static String fill0Left(String str, int length) {
        if (str == null) {
            str = "";
        }
        int len = length - str.length();
        if (len > 0) {
            for (int i = 0; i < len; i++) {
                str = "0" + str;
            }
        }
        return str;
    }

    /**
     * Map 转成 String
     *
     * @param map
     * @return
     */
    public static String toString(Map<?, ?> map) {
        if (map == null) {
            return "";
        }
        if (map.isEmpty()) {
            return "{}";
        }
        StringBuffer buf = new StringBuffer();
        buf.append("{");

        Iterator<?> i = map.entrySet().iterator();
        boolean hasNext = i.hasNext();
        while (hasNext) {
            Entry<?, ?> e = (Entry<?, ?>) (i.next());
            Object key = e.getKey();
            Object value = e.getValue();
            buf.append((key == map ? "(this Map)" : key));

            buf.append("=");
            if (value == map) {
                buf.append("(this Map)");
            } else {
                if (value instanceof Object[]) {
                    Object[] objs = (Object[]) value;
                    buf.append("[");
                    for (int j = 0; j < objs.length; j++) {
                        if (j > 0) {
                            buf.append(", ");
                        }
                        buf.append(objs[j]);
                    }
                    buf.append("]");
                } else if (value instanceof Map) {
                    buf.append(toString((Map<?, ?>) value));
                } else {
                    buf.append(value);
                }
            }
            hasNext = i.hasNext();
            if (hasNext) {
                buf.append(", ");
            }
        }
        buf.append("}");
        return buf.toString();
    }

    /**
     * Object[] 转成 String
     *
     * @param objs
     * @return
     */
    public static String toString(Object[] objs) {
        if (objs == null) {
            return "";
        }
        StringBuffer buf = new StringBuffer();
        buf.append("[");
        for (int j = 0; j < objs.length; j++) {
            if (j > 0) {
                buf.append(", ");
            }
            buf.append(String.valueOf(objs[j]));
        }
        buf.append("]");
        return buf.toString();
    }

    /**
     * List, Object>> 转成 String
     *
     * @param objs
     * @return
     */
    public static String toString(ListString, Object>> objs) {
        if (objs == null) {
            return "";
        }
        StringBuffer buf = new StringBuffer();
        buf.append("[");
        int j = 0;
        for (MapString, Object> map : objs) {
            if (j > 0) {
                buf.append(", ");
            }
            buf.append(toString(map));
            j++;
        }
        buf.append("]");
        return buf.toString();
    }

    /**
     * 是否早于当天
     *
     * @param date yyyy-MM-dd
     * @return
     */
    public static boolean beforeToday(String date) {
        SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            Date d = sformat.parse(date + " 00:00:00");
            Date now = sformat.parse(getServerTime("yyyy-MM-dd") + " 00:00:00");
            return d.before(now);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return false;
    }

    /**
     * 是否晚于当天
     *
     * @param date yyyy-MM-dd
     * @return
     */
    public static boolean afterToday(String date) {
        SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            Date d = sformat.parse(date + " 00:00:00");
            Date now = sformat.parse(getServerTime("yyyy-MM-dd") + " 00:00:00");
            return d.after(now);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return false;
    }

    /**
     * 是否是当天
     *
     * @param date yyyy-MM-dd
     * @return
     */
    public static boolean isToday(String date) {
        return date.equals(getServerTime("yyyy-MM-dd"));
    }

    /**
     * 是否是当天(工作日)
     *
     * @param weekday yyyy-MM-dd
     * @return
     */
    @SuppressWarnings("deprecation")
    public static boolean isWeekday(int weekday) {
        return (new Date().getDay() == weekday - 1) ? true : false;
    }

    /**
     * 是否在两个日期段内
     *
     * @param startDate yyyy-MM-dd
     * @param endDate   yyyy-MM-dd
     * @return
     */
    public static boolean inDate(String startDate, String endDate) {
        return !afterToday(startDate) && !beforeToday(endDate);
    }


    /**
     * 是否在两个时间段内
     *
     * @param startTime HHmm
     * @param endTime   HHmm
     * @return
     */
    public static boolean inTime(String startTime, String endTime) {
        startTime = startTime.replace(":", "");
        if (startTime.length() == 4) {
            startTime = startTime + "00";
        }
        endTime = endTime.replace(":", "");
        if (endTime.length() == 4) {
            endTime = endTime + "00";
        }
        int now = Integer.parseInt(getServerTime("HHmmss"));
        if (now >= Integer.parseInt(startTime) && now <= Integer.parseInt(endTime)) {
            return true;
        }
        return false;
    }

    /**
     * 是否早于当前时间
     *
     * @param time HHmm
     * @return
     */
    public static boolean beforeNow(String time) {
        time = time.replace(":", "");
        if (time.length() == 4) {
            time = time + "00";
        }
        int now = Integer.parseInt(getServerTime("HHmmss"));
        if (Integer.parseInt(time) < now) {
            return true;
        }
        return false;
    }

    /**
     * 是否晚于当前时间
     *
     * @param time HHmm
     * @return
     */
    public static boolean afterNow(String time) {
        time = time.replace(":", "");
        if (time.length() == 4) {
            time = time + "00";
        }
        int now = Integer.parseInt(getServerTime("HHmmss"));
        if (Integer.parseInt(time) > now) {
            return true;
        }
        return false;
    }

    public static String toString(double d) {
        BigDecimal bd = new BigDecimal(d);
        String s = bd.toPlainString();
        String[] ss = s.split("\\.");
        if (ss.length == 2) {
            if (ss[1].length() > 10) {
                return ss[0] + "." + ss[1].substring(0, 10);
            }
        }
        return s;
    }

    public static double toDouble(double d) {
        BigDecimal bd = new BigDecimal(d);
        String s = bd.toPlainString();
        String[] ss = s.split("\\.");
        if (ss.length == 2) {
            if (ss[1].length() > 10) {
                return Double.parseDouble(ss[0] + "." + ss[1].substring(0, 10));
            }
        }
        return Double.parseDouble(s);
    }

    public static String getWeekStr(int week) {
        switch (week) {
            case 1:
                return "周日";
            case 2:
                return "周一";
            case 3:
                return "周二";
            case 4:
                return "周三";
            case 5:
                return "周四";
            case 6:
                return "周五";
            case 7:
                return "周六";
        }
        return "未知[" + week + "]";
    }

    public static String encode(String str) {
        try {
            return URLEncoder.encode(str, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return str;
    }

    public static String decode(String str) {
        try {
            return URLDecoder.decode(str, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return str;
    }

    public static String simpleDatetime(String datetime) {
        if (datetime == null) {
            return "";
        }
        if (datetime.matches("\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}:\\d{2}")) {
            return datetime.substring(5, 16);
        } else if (datetime.matches("\\d{4}-\\d{2}-\\d{2}")) {
            return datetime.substring(5, 10);
        } else if (datetime.matches("\\d{2}:\\d{2}:\\d{2}")) {
            return datetime.substring(0, 5);
        }
        return datetime;
    }

    public static String formatDate(String datetime, String format) {
        if (datetime == null) {
            return "";
        }
        SimpleDateFormat sdf = new SimpleDateFormat(format, Locale.CHINA);
        try {
            return sdf.format(sdf.parse(datetime));
        } catch (Exception e) {
            e.printStackTrace();
            return datetime;
        }
    }

    /**
     * 当date2 >= date1 时返回true
     *
     * @param date1
     * @param date2
     * @return
     */
    public static boolean matchDate(String date1, String date2) {
        if (date1 == null || date1.equals("")) {
            return true;
        }
        if (date2 == null || date2.equals("")) {
            return false;
        }
        SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            Date d1 = sformat.parse(date1 + " 00:00:00");
            Date d2 = sformat.parse(date2 + " 00:00:00");
            return d1.equals(d2) || d1.before(d2);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return false;
    }

    /**
     * 是否是有效的手机号
     *
     * @param str
     * @return
     */
    public static boolean isMobile(String str) {
        if (str == null || str.equals("")) {
            return false;
        }
        return str.matches(REG_MOBILE);
    }

    /**
     * 是否是有效的座机
     *
     * @param str
     * @return
     */
    public static boolean isTelephone(String str) {
        if (str == null || str.equals("")) {
            return false;
        }
        return str.matches(REG_TEL);
    }

    /**
     * 是否是有效的邮箱号
     *
     * @param str
     * @return
     */
    public static boolean isMail(String str) {
        if (str == null || str.equals("")) {
            return false;
        }
        return str.matches(REG_MAIL);
    }

    /**
     * 是否是有效身份证号
     *
     * @param no
     * @return
     */
    public static boolean isIDNo(String no) {
        if (no == null || !no.matches("^\\d{18}|(\\d{17}[xX])$")) {
            return false;
        }
        // 系数
        int[] c = new int[]{7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2};
        int sum = 0;
        for (int i = 0; i < 17; i++) {
            sum += Integer.parseInt(String.valueOf(no.charAt(i))) * c[i];
        }
        int m = sum % 11;
        char[] flag = new char[]{'1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2'};
        char a = no.charAt(17);
        if (m == 2) {
            if (a == 'x' || a == 'X') {
                return true;
            }
            return false;
        }
        if (flag[m] == a) {
            return true;
        }
        return false;
    }

    /**
     * 验证IMSI号
     *
     * @param imsi
     * @return
     */
    public static boolean isImsi(String imsi) {
        return (imsi != null && imsi.matches("^4600[0123567]\\d{10}|(?!4600)[02345679]\\d{14}$"));
    }

    public static String date(int y, int m, int d) {
        String date = y + "-";
        if (m < 10) {
            date += "0" + m + "-";
        } else {
            date += m + "-";
        }
        if (d < 10) {
            date += "0" + d;
        } else {
            date += d;
        }
        return date;
    }

    /**
     * 将日期字符串转成日期对象
     * @param datetime
     * @return
     */
    public static Date toDate(String datetime) {
        if (datetime == null || datetime.length() < 0) {
            return null;
        }
        if (datetime.matches("\\d{14,}")) {
            datetime = datetime.substring(0, 4) + "-" + datetime.substring(4, 6) + "-" + datetime.substring(6, 8) + " " + datetime.substring(8, 10) + ":"
                    + datetime.substring(10, 12) + ":" + datetime.substring(12, 14);
        } else if (datetime.matches("\\d{4}-\\d{2}-\\d{2}")) {
            datetime = datetime + " 00:00:00";
        } else if (datetime.matches("\\d{1,10}")) {
            try {
                return new Date(Long.parseLong(datetime) * 1000);
            } catch (Exception e) {
                logger.error("Unparseable date: \"" + datetime + "\"", e);
                return null;
            }
        } else if (datetime.matches("\\d{1,13}")) {
            try {
                return new Date(Long.parseLong(datetime));
            } catch (Exception e) {
                logger.error("Unparseable date: \"" + datetime + "\"", e);
                return null;
            }
        } else if (datetime.matches("[a-zA-Z]{3} [a-zA-Z]{3} \\d{1,2} \\d{1,2}:\\d{1,2}:\\d{1,2} [a-zA-Z]{3} \\d{4}")) {
            try {
                return new SimpleDateFormat("EEE MMM dd HH:mm:ss zzz yyyy", Locale.US).parse(datetime);
            } catch (Exception e) {
                logger.error("Unparseable date: \"" + datetime + "\"", e);
                return null;
            }
        }

        try {
            SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            return sformat.parse(datetime);
        } catch (Exception e) {
            logger.error("Unparseable date: \"" + datetime + "\"", e);
            return null;
        }
    }

    /**
     * 计算时间差 (时间单位,开始时间,结束时间) 调用方法
     *
     * @unit ：0-毫秒；1-秒；2-分；3-小时；4-天
     */
    public static long timeDiff(String time1, String time2, int unit) {
        return timeDiff(time1, time2, unit, true);
    }

    /**
     * 计算时间差 (时间单位,开始时间,结束时间) 调用方法
     *
     * @unit ：0-毫秒；1-秒；2-分；3-小时；4-天
     * @flag : true-返回绝对值,false-区分正分
     */
    public static long timeDiff(String time1, String time2, int unit, boolean flag) {
        // 单位(如：不足1天(24小时) 则返回0)，开始时间，结束时间
        Date date1 = toDate(time1);
        Date date2 = toDate(time2);
        long ltime = date2.getTime() - date1.getTime();
        if (flag) {
            ltime = Math.abs(ltime);
        }
        if (unit == SECOND) {
            return ltime / 1000;// 返回秒
        } else if (unit == MINUTE) {
            return ltime / 60000;// 返回分钟
        } else if (unit == HOUR) {
            return ltime / 3600000;// 返回小时
        } else if (unit == DAY) {
            return ltime / 86400000;// 返回天数
        } else {
            return ltime;// 毫秒
        }
    }

    /**
     * 获得一个UUID
     *
     * @return String UUID
     */
    public static String getUUID() {
        String s = UUID.randomUUID().toString();
        // 去掉“-”符号
        return s.substring(0, 8) + s.substring(9, 13) + s.substring(14, 18) + s.substring(19, 23) + s.substring(24);
    }

    /**
     * @功能: 10进制串转为BCD码
     * @参数: 10进制串
     * @结果: BCD码
     */
    public static byte[] string2Bcd(String asc) {
        int len = asc.length();
        int mod = len % 2;
        if (mod != 0) {
            asc = "0" + asc;
            len = asc.length();
        }
        byte abt[] = new byte[len];
        if (len >= 2) {
            len = len / 2;
        }
        byte bbt[] = new byte[len];
        abt = asc.getBytes();
        int j, k;
        for (int p = 0; p < asc.length() / 2; p++) {
            if ((abt[2 * p] >= '0') && (abt[2 * p] <= '9')) {
                j = abt[2 * p] - '0';
            } else if ((abt[2 * p] >= 'a') && (abt[2 * p] <= 'z')) {
                j = abt[2 * p] - 'a' + 0x0a;
            } else {
                j = abt[2 * p] - 'A' + 0x0a;
            }
            if ((abt[2 * p + 1] >= '0') && (abt[2 * p + 1] <= '9')) {
                k = abt[2 * p + 1] - '0';
            } else if ((abt[2 * p + 1] >= 'a') && (abt[2 * p + 1] <= 'z')) {
                k = abt[2 * p + 1] - 'a' + 0x0a;
            } else {
                k = abt[2 * p + 1] - 'A' + 0x0a;
            }
            int a = (j << 4) + k;
            byte b = (byte) a;
            bbt[p] = b;
        }
        return bbt;
    }

    /**
     * 将bcd码转成十进制，过滤掉填充字符F
     *
     * @param bytes
     * @return
     */
    public static String bcd2String(byte[] bytes) {
        StringBuffer temp = new StringBuffer(bytes.length * 2);
        for (int i = 0; i < bytes.length; i++) {
            byte b = (byte) ((bytes[i] & 0xf0) >>> 4);
            if (b != 15) {
                temp.append(b);
            }
            b = (byte) (bytes[i] & 0x0f);
            if (b != 15) {
                temp.append(b);
            }
        }
        return temp.toString();
    }

    public static boolean inArrary(String[] array, String obj) {
        if (array == null || obj == null) {
            return false;
        }
        for (String arr : array) {
            if (obj.equals(arr)) {
                return true;
            }
        }
        return false;
    }

    public static boolean inArrary(int[] array, int obj) {
        if (array == null) {
            return false;
        }
        for (int arr : array) {
            if (obj == arr) {
                return true;
            }
        }
        return false;
    }

    /**
     * 获取最近12个月，经常用于统计图表的X轴
     */
    public static String[] getLast12Months() {
        String[] last12Months = new String[12];
        Calendar cal = Calendar.getInstance();
        cal.set(Calendar.MONTH, cal.get(Calendar.MONTH) + 1); // 要先+1,才能把本月的算进去/span>
        for (int i = 0; i < 12; i++) {
            cal.set(Calendar.MONTH, cal.get(Calendar.MONTH) - 1); // 逐次往前推1个月
            last12Months[11 - i] = cal.get(Calendar.YEAR) + "-" + Utils.fill0Left((cal.get(Calendar.MONTH) + 1) + "", 2);
        }
        return last12Months;
    }

    //生成随机数字和字母,
    public static String getStringRandom(int length) {
        StringBuilder val = new StringBuilder();
        Random random = new Random();

        //参数length，表示生成几位随机数
        for (int i = 0; i < length; i++) {

            String charOrNum = random.nextInt(2) % 2 == 0 ? "char" : "num";
            //输出字母还是数字
            if ("char" .equalsIgnoreCase(charOrNum)) {
                //输出是大写字母还是小写字母
                int temp = random.nextInt(2) % 2 == 0 ? 65 : 97;
                val.append((char) (random.nextInt(26) + temp));
            } else if ("num" .equalsIgnoreCase(charOrNum)) {
                val.append(String.valueOf(random.nextInt(10)));
            }
        }
        return val.toString();
    }

    public static String prettyTime(long millis) {
        long d = 0, h = 0, m = 0;
        long s = millis / 1000;
        long mi = millis % 1000;
        if (s >= 60) {
            m = s / 60;
            s = s % 60;
        }
        if (m >= 60) {
            h = m / 60;
            m = m % 60;
        }
        if (h > 24) {
            d = h / 24;
            h = h % 24;
        }
        if (d > 0) {
            return d + "天" + h + "时" + m + "分" + s + "秒" + mi + "毫秒";
        } else if (h > 0) {
            return h + "时" + m + "分" + s + "秒" + mi + "毫秒";
        } else if (m > 0) {
            return m + "分" + s + "秒" + mi + "毫秒";
        } else if (s > 0) {
            return s + "秒" + mi + "毫秒";
        }
        return mi + "毫秒";
    }

    public static void main(String[] args) {
        System.out.println(isImsi("100081212120000"));
    }
}
