package com.javaee.onlinehosbackend.extrautil.reflect;

import java.lang.reflect.Field;

/**
 * @Author Sugar
 * @Version 2019/1/23 15:03
 */
public interface IFieldFilter {
    String name();
    boolean filter(Field field);
}
