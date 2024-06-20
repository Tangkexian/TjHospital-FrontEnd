package com.javaee.onlinehosbackend.extrautil.reflect;

import lombok.Data;
import lombok.Getter;

import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * @Author Sugar
 * @Version 2019/1/23 15:34
 */
@Getter
public class BeanField implements IFieldWrapper {
    private String name;
    private Class<?> type;
    public Field field;
    public Method setter;
    public Method getter;

    public BeanField(Field field, Method setter, Method getter) {
        this.field = field;
        this.setter = setter;
        this.getter = getter;
        this.name = field.getName();
        this.type = field.getType();
    }

    public <T extends Annotation> T getAnnotation(Class<T> annotationClass) {
        return field.getAnnotation(annotationClass);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        BeanField beanField = (BeanField) o;
        return name.equals(beanField.getName()) && type == beanField.getType();
    }

    public Method getGetterMethod() {
        return getter;
    }

    public Method getSetterMethod() {
        return setter;
    }

    @Override
    public int hashCode() {
        return field.hashCode();
    }
}
