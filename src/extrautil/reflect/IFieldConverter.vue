package com.javaee.onlinehosbackend.extrautil.reflect;

import java.lang.reflect.Field;

/**
 * @Description
 * @Author Sugar
 * @Date 2020/10/9 11:30
 */
public interface IFieldConverter<T> {
    T convert(Class clazz, Field field);
}
