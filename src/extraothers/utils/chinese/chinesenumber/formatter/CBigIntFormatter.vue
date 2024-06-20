package com.javaee.onlinehosbackend.extraother.utils.chinese.chinesenumber.formatter;



import com.javaee.onlinehosbackend.extraother.utils.chinese.chinesenumber.CNumber;
import com.javaee.onlinehosbackend.extraother.utils.chinese.chinesenumber.CNumberList;

import java.math.BigDecimal;
import java.math.BigInteger;

/**
 *
 * @author ForteScarlet <[163邮箱地址]ForteScarlet@163.com>
 * @date Created in 2019/4/16 13:09
 * @since JDK1.8
 **/
public class CBigIntFormatter extends BaseOnlyNumCNumberFormatterBigInteger> implements CNumberList {

    /** 是否为浮点数 */
    private static final boolean FLOAT = false;

    public CBigIntFormatter(String[] chineseStr, boolean negative) {
        super(chineseStr, negative);
    }

    /**
     * 接收汉字字符串，转化为中文数字对象
     * @return
     */
    @Override
    public CNumberBigDecimal> parse() {
        BigDecimal result = this.parseAsBigDecimal();

        return null;
    }


    @Override
    public boolean isFloat() {
        return FLOAT;
    }
}
