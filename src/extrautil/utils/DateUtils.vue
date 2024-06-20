package com.javaee.onlinehosbackend.extrautil.utils;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.Instant;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.util.Calendar;
import java.util.Date;

public class DateUtils {
	public static boolean inTime(String time, String startTime, String endTime) {
		startTime = startTime.replace(":", "");
		if (startTime.length() == 4) {
			startTime = startTime + "00";
		}
		endTime = endTime.replace(":", "");
		if (endTime.length() == 4) {
			endTime = endTime + "00";
		}
		int now = Integer.parseInt(time);
		if (now >= Integer.parseInt(startTime) && now <= Integer.parseInt(endTime)) {
			return true;
		}
		return false;
	}

	/**
* 是否在两个日期段内
*
* @param date
*            yyyy-MM-dd
* @return
*/
	public static boolean inDate(String date, String startDate, String endDate) {
		return !afterToday(date, startDate) && !beforeToday(date, endDate);
	}

	/**
* 是否早于该天
*
* @param date
*            yyyy-MM-dd
*
* @return
*/
	public static boolean beforeToday(String currDate, String date) {
		SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		try {
			Date d = sformat.parse(date + " 00:00:00");
			Date now = sformat.parse(currDate + " 00:00:00");
			return d.before(now);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return false;
	}

	/**
* 是否早于该日期时间
*
* @param date
*            yyyy-MM-dd HH:mm:ss
*
* @return
*/
	public static boolean beforeTodaytime(String currDate, String date) {
		SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		try {
			Date d = sformat.parse(date);
			Date now = sformat.parse(currDate);
			return d.before(now);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return false;
	}

	/**
* 是否晚于该天
*
* @param date
*            yyyy-MM-dd
* @return
*/
	public static boolean afterToday(String currDate, String date) {
		SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		try {
			Date d = sformat.parse(date + " 00:00:00");
			Date now = sformat.parse(currDate + " 00:00:00");
			return d.after(now);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return false;
	}

	/**
* 是否晚于该日期时间
*
* @param date
*            yyyy-MM-dd HH:mm:ss
* @return
*/
	public static boolean afterTodaytime(String currDate, String date) {
		SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		try {
			Date d = sformat.parse(date);
			Date now = sformat.parse(currDate);
			return d.after(now);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return false;
	}

	/**
* 将字符串时间转换成日期格式
*
* @param datetime
*            String 默认返回yyyy-MM-dd HH:mm:ss格式的时间
*
* @return
*/
	public static Date toDate(String datetime) {
		if (datetime == null || datetime.length() < 0) {
			return null;
		}
		if (datetime.matches("\\d{14,}")) {
			datetime = datetime.substring(0, 4) + "-" + datetime.substring(4, 6) + "-" + datetime.substring(6, 8) + " " + datetime.substring(8, 10) + ":" + datetime.substring(10, 12) + ":" + datetime.substring(12, 14);
		} else if (datetime.matches("\\d{4}-\\d{2}-\\d{2}")) {
			datetime = datetime + " 00:00:00";
		} else if (datetime.matches("\\d{2}:\\d{2}:\\d{2}(.\\d+)?")) {
			datetime = Utils.getServerTime("yyyy-MM-dd") + " " + datetime;
		} else if (!datetime.matches("\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}:\\d{2}(.\\d+)?")) {
			try {
				return new Date(datetime);
			} catch (Exception e) {
				System.err.println("Unparseable date: \"" + datetime + "\"");
				return null;
			}
		}
		try {
			SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
			return sformat.parse(datetime);
		} catch (Exception e) {
			System.err.println("Unparseable date: \"" + datetime + "\"");
			return null;
		}
	}

	public static final int MILLISECOND = 0;
	public static final int SECOND = 1;
	public static final int MINUTE = 2;
	public static final int HOUR = 3;
	public static final int DAY = 4;

	/**
* 计算时间差 (时间单位,开始时间,结束时间) 调用方法
*
* @unit ：0-毫秒；1-秒；2-分；3-小时；4-天
* */
	public static long timeDiff(String time1, String time2, int unit) {
		return timeDiff(time1, time2, unit, true);
	}

	/**
* 计算时间差 (时间单位,开始时间,结束时间) 调用方法
*
* @unit ：0-毫秒；1-秒；2-分；3-小时；4-天
* @flag : true-返回绝对值,false-区分正分
* */
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
* 格式化指定时间和格式
*
* @param date
* @param parrent
* @return
*/
	public static String parrentDate(String date, String parrent) {
		SimpleDateFormat sformat = new SimpleDateFormat(parrent);
		try {
			Date d = sformat.parse(date);
			return sformat.format(d.getTime());
		} catch (ParseException e) {
			e.printStackTrace();
			return date;
		}
	}

	/**
* 获取当前日期前一天日期
*
* @return
*/
	public static String getServerNextDay() {
		SimpleDateFormat sformat = new SimpleDateFormat("yyyy-MM-dd");
		Calendar calendar = Calendar.getInstance();
		calendar.setTime(new Date());
		calendar.add(Calendar.DAY_OF_MONTH, -1);
		return sformat.format(calendar.getTime());
	}

	/**
* 将Long类型的时间戳转换成String 类型的时间格式，时间格式为：yyyy-MM-dd HH:mm:ss
*/
	public static String formatTimestamp(long seconds){
		DateTimeFormatter ftf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
		return ftf.format(LocalDateTime.ofInstant(Instant.ofEpochSecond(seconds), ZoneId.systemDefault()));
	}
}
