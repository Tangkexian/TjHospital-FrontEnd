package com.javaee.onlinehosbackend.extrautil.paging;

import java.io.Serializable;
import java.util.List;

/**
 * @Author Sugar
 * @Version 2018/5/16 11:50
 */
public interface IPage extends Serializable {
    void setTotalCount(long totalCount);
    <T> void setList(List<T> dataList);
    <T> Class<T> getClazz();
}
