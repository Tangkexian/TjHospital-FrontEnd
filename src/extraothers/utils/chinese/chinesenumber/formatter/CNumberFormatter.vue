package com.javaee.onlinehosbackend.extraother.utils.chinese.chinesenumber.formatter;


import com.javaee.onlinehosbackend.extraother.utils.chinese.chinesenumber.CNumber;

import java.math.BigDecimal;

/**
 * 数字格式化实现类，指定三个实现类：
 *
 * Integer
 * Long
 * BigInteger
 *
 * @author ForteScarlet <[163邮箱地址]ForteScarlet@163.com>
 * @date Created in 2019/4/16 11:40
 * @since JDK1.8
 **/
public interface CNumberFormatterN extends Number> {

    /**
     * 接收汉字字符串，转化为中文数字对象
     * @return 转化结果
     */
    CNumberBigDecimal> parse();


}

