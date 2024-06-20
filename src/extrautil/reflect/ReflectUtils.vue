package com.javaee.onlinehosbackend.extrautil.reflect;


import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.lang.reflect.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * java bean反射工具类
 *
 * @Author Sugar
 * @Version 2018/6/1 10:48
 */
public class ReflectUtils {
    protected static final Logger logger = LoggerFactory.getLogger(ReflectUtils.class);
    protected static final Map<String, List<? extends IFieldWrapper>> BEAN_FIELD_CACHE = new HashMap<>();//属性缓存
    protected static final Map<String, List<Field>> FIELD_CACHE = new HashMap<>();//属性缓存
    protected static final Map<String, List<Method>> SETTER_CACHE = new HashMap<>();//setter方法缓存
    protected static final Map<String, List<Method>> GETTER_CACHE = new HashMap<>();//getter方法缓存
    private static final IFieldConverter<BeanField> beanFieldConverter =  null;

    /**
     * 获取类中非final和static的属性(包括父类中的属性)，带setter和getter方法
     *
     * @param clazz
     * @return
     */
    public static final List<BeanField> getBeanFields(Class clazz) {
        return getBeanFields(clazz, true, null, beanFieldConverter);
    }

    /**
     * 获取类中非final和static的属性(包括父类中的属性)，带setter和getter方法
     *
     * @param clazz
     * @param filter
     * @return
     */
    public static final List<BeanField> getBeanFields(Class clazz, IFieldFilter filter) {
        return getBeanFields(clazz, true, filter, beanFieldConverter);
    }

    /**
     * 获取类中非final和static的属性，带setter和getter方法
     *
     * @param clazz
     * @param superClass 是否包含父类中的属性
     * @return
     */
    public static final List<BeanField> getBeanFields(Class clazz, boolean superClass) {
        return getBeanFields(clazz, superClass, null, beanFieldConverter);
    }

    /**
     * 获取类中非final和static的属性，带setter和getter方法
     *
     * @param clazz
     * @param superClass 是否包含父类中的属性
     * @param filter
     * @return
     */
    public static final <T extends IFieldWrapper> List<T> getBeanFields(Class clazz, boolean superClass, IFieldFilter filter, IFieldConverter<T> converter) {
        String className = clazz.getName();
        String cacheKey = className + ":" + superClass + (filter == null ? "" : ":" + filter.name());
        List fields = BEAN_FIELD_CACHE.get(cacheKey);
        if (fields != null) {
            return fields;
        }
        synchronized (clazz) {
            fields = BEAN_FIELD_CACHE.get(cacheKey);
            if (fields != null) {
                return fields;
            }

            fields = new ArrayList<>();
            Field[] declared = clazz.getDeclaredFields();
            for (Field field : declared) {
                int mod = field.getModifiers();
                if (Modifier.isFinal(mod) || Modifier.isStatic(mod)) {
                    continue;
                }
                if (filter != null && !filter.filter(field)) {
                    continue;
                }
                fields.add(converter.convert(clazz, field));
            }
            if (superClass) {//包含父类中的属性
                Class<?> superClazz = clazz;
                while ((superClazz = superClazz.getSuperclass()) != null) {
                    Field[] lst = superClazz.getDeclaredFields();
                    for (Field field : lst) {
                        int mod = field.getModifiers();
                        if (Modifier.isFinal(mod) || Modifier.isStatic(mod)) {
                            continue;
                        }
                        if (filter != null && !filter.filter(field)) {
                            continue;
                        }
                        boolean exists = false;
                        for (Object o : fields) {
                            IFieldWrapper f = (IFieldWrapper) o;
                            if (f.getField().getName().equals(field.getName()) && f.getField().getType() == field.getType()) {
                                exists = true;
                                break;
                            }
                        }
                        if (!exists) {
                            fields.add(converter.convert(clazz, field));
                        }
                    }
                }
            }
            BEAN_FIELD_CACHE.put(cacheKey, fields);
        }
        return fields;
    }

    /**
     * 获取类或所有父类中非final和static的属性
     *
     * @param clazz
     * @return
     */
    public static final List<Field> getFields(Class clazz) {
        return getFields(clazz, true, null);
    }

    /**
     * 获取类或所有父类中非final和static的属性
     *
     * @param clazz
     * @param filter
     * @return
     */
    public static final List<Field> getFields(Class clazz, IFieldFilter filter) {
        return getFields(clazz, true, filter);
    }

    /**
     * 获取类或所有父类中非final和static的属性
     *
     * @param clazz
     * @param superClass
     * @return
     */
    public static final List<Field> getFields(Class clazz, boolean superClass) {
        return getFields(clazz, superClass, null);
    }

    /**
     * 获取类中非final和static的属性
     *
     * @param clazz
     * @param superClass 是否包含父类中的属性
     * @param filter
     * @return
     */
    public static final List<Field> getFields(Class clazz, boolean superClass, IFieldFilter filter) {
        String className = clazz.getName();
        String cacheKey = className + ":" + superClass + (filter == null ? "" : ":" + filter.name());
        List<Field> fields = FIELD_CACHE.get(cacheKey);
        if (fields != null) {
            return fields;
        }
        synchronized (clazz) {
            fields = FIELD_CACHE.get(cacheKey);
            if (fields != null) {
                return fields;
            }
            fields = new ArrayList<>();
            Field[] declared = clazz.getDeclaredFields();
            for (Field field : declared) {
                int mod = field.getModifiers();
                if (Modifier.isFinal(mod) || Modifier.isStatic(mod)) {
                    continue;
                }
                if (filter != null && !filter.filter(field)) {
                    continue;
                }
                fields.add(field);
            }
            if (superClass) {//包含父类中的属性
                Class<?> superClazz = clazz;
                while ((superClazz = superClazz.getSuperclass()) != null) {
                    Field[] lst = superClazz.getDeclaredFields();
                    for (Field field : lst) {
                        int mod = field.getModifiers();
                        if (Modifier.isFinal(mod) || Modifier.isStatic(mod)) {
                            continue;
                        }
                        if (filter != null && !filter.filter(field)) {
                            continue;
                        }
                        if (!contains(fields, field)) {
                            fields.add(field);
                        }
                    }
                }
            }
            FIELD_CACHE.put(cacheKey, fields);
        }
        return fields;
    }

    /**
     * 获取类或所有父类中setter方法
     *
     * @param clazz
     * @return
     */
    public static final List<Method> getSetterMethods(Class clazz) {
        return getSetterMethods(clazz, true);
    }

    /**
     * 获取类中setter方法
     *
     * @param clazz
     * @param superClass 是否包含父类中的方法
     * @return
     */
    public static final List<Method> getSetterMethods(Class clazz, boolean superClass) {
        String className = clazz.getName();
        List<Method> methods = SETTER_CACHE.get(className + ":" + superClass);
        if (methods != null) {
            return methods;
        }
        methods = new ArrayList<>();
        Method[] declared = clazz.getDeclaredMethods();
        for (Method method : declared) {
            if (!method.getName().matches("set[A-Z]\\w+") || method.getParameterCount() != 1) {
                continue;
            }
            int mod = method.getModifiers();
            if (Modifier.isStatic(mod)) {
                continue;
            }
            methods.add(method);
        }
        if (superClass) {//包含父类中的方法
            Class<?> superClazz = clazz;
            while ((superClazz = superClazz.getSuperclass()) != null) {
                Method[] lst = superClazz.getDeclaredMethods();
                for (Method method : lst) {
                    if (!method.getName().matches("set[A-Z]\\w+") || method.getParameterCount() != 1) {
                        continue;
                    }
                    int mod = method.getModifiers();
                    if (Modifier.isStatic(mod)) {
                        continue;
                    }
                    if (!contains(methods, method)) {
                        methods.add(method);
                    }
                }
            }
        }
        SETTER_CACHE.put(className + ":" + superClass, methods);
        return methods;
    }

    /**
     * 获取类或所有父类中getter方法
     *
     * @param clazz
     * @return
     */
    public static final List<Method> getGetterMethods(Class clazz) {
        return getGetterMethods(clazz, true);
    }

    /**
     * 获取类中getter方法
     *
     * @param clazz
     * @param superClass 是否包含父类中的方法
     * @return
     */
    public static final List<Method> getGetterMethods(Class clazz, boolean superClass) {
        String className = clazz.getName();
        List<Method> methods = GETTER_CACHE.get(className + ":" + superClass);
        if (methods != null) {
            return methods;
        }
        methods = new ArrayList<>();
        Method[] declared = clazz.getDeclaredMethods();
        for (Method method : declared) {
            if (!method.getName().matches("(get|is)[A-Z]\\w+") || method.getParameterCount() != 0) {
                continue;
            }
            int mod = method.getModifiers();
            if (Modifier.isStatic(mod)) {
                continue;
            }
            methods.add(method);
        }
        if (superClass) {//包含父类中的方法
            Class<?> superClazz = clazz;
            while ((superClazz = superClazz.getSuperclass()) != null) {
                Method[] lst = superClazz.getDeclaredMethods();
                for (Method method : lst) {
                    if (!method.getName().matches("(get|is)[A-Z]\\w+") || method.getParameterCount() != 0) {
                        continue;
                    }
                    int mod = method.getModifiers();
                    if (Modifier.isStatic(mod)) {
                        continue;
                    }
                    if (!contains(methods, method)) {
                        methods.add(method);
                    }
                }
            }
        }
        GETTER_CACHE.put(className + ":" + superClass, methods);
        return methods;
    }

    /**
     * 根据字段获取getter方法名
     *
     * @param fieldName
     * @return
     */
    public static final String getGetterMethodName(String fieldName) {
        return "get" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);
    }

    /**
     * 根据字段获取getter方法名
     *
     * @param field
     * @return
     */
    public static final String getGetterMethodName(Field field) {
        String fieldName = field.getName();
        if (field.getType() == boolean.class || field.getType().isAssignableFrom(Boolean.class)) {
            return "is" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);
        }
        return getGetterMethodName(field.getName());
    }

    /**
     * 根据字段获取setter方法名
     *
     * @param fieldName
     * @return
     */
    public static final String getSetterMethodName(String fieldName) {
        return "set" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);
    }

    /**
     * 根据字段获取setter方法名
     *
     * @param field
     * @return
     */
    public static final String getSetterMethodName(Field field) {
        return getSetterMethodName(field.getName());
    }

    /**
     * 根据setter或getter方法获取字段名
     *
     * @param method
     * @return
     */
    public static final String getFieldName(Method method) {
        String name = method.getName().replaceFirst("^(get|set|is)", "");
        if (name.isEmpty()) {
            return null;
        }
        return Character.toLowerCase(name.charAt(0)) + name.substring(1);
    }

    public static final Method getMethod(Class clazz, String method) {
        try {
            return clazz.getMethod(method);
        } catch (NoSuchMethodException e) {
            if (logger.isTraceEnabled()) {
                logger.trace("!No such method: {}", method);
            }
            return null;
        }
    }

    /**
     * 获取setter方法
     *
     * @param clazz
     * @param fieldName
     * @return
     */
    public static final Method getSetterMethod(Class clazz, String fieldName) {
        List<Method> methods = getSetterMethods(clazz);
        if (methods == null || methods.isEmpty()) {
            return null;
        }
        String name = getSetterMethodName(fieldName);
        for (Method method : methods) {
            if (method.getName().equals(name)) {
                return method;
            }
        }
        return null;
    }

    /**
     * 获取setter方法
     *
     * @param clazz
     * @param field
     * @return
     */
    public static final Method getSetterMethod(Class clazz, Field field) {
        return getSetterMethod(clazz, field.getName());
    }

    /**
     * 获取getter方法
     *
     * @param clazz
     * @param fieldName
     * @return
     */
    public static final Method getGetterMethod(Class clazz, String fieldName) {
        List<Method> methods = getGetterMethods(clazz);
        if (methods == null || methods.isEmpty()) {
            return null;
        }
        String name = getGetterMethodName(fieldName);
        for (Method method : methods) {
            if (method.getName().equals(name)) {
                return method;
            }
        }
        return null;
    }

    /**
     * 获取getter方法
     *
     * @param clazz
     * @param field
     * @return
     */
    public static final Method getGetterMethod(Class clazz, Field field) {
        return getGetterMethod(clazz, field.getName());
    }

    /**
     * 通过setter或getter方法获取属性
     *
     * @param clazz
     * @param method
     * @return
     */
    public static final Field getField(Class clazz, Method method) {
        List<Field> fields = getFields(clazz);
        if (fields == null || fields.isEmpty()) {
            return null;
        }
        String name = getFieldName(method);
        if (name == null) {
            return null;
        }
        for (Field field : fields) {
            if (field.getName().equals(name)) {
                return field;
            }
        }
        return null;
    }

    /**
     * 判断是否包含相同的field
     *
     * @param list
     * @param field
     * @return
     */
    public static final boolean contains(List<Field> list, Field field) {
        if (list == null || list.isEmpty()) {
            return false;
        }
        for (Field f : list) {
            if (f.getName().equals(field.getName()) && f.getType() == field.getType()) {
                return true;
            }
        }
        return false;
    }

    /**
     * 判断是否包含相同的method
     *
     * @param list
     * @param method
     * @return
     */
    protected static final boolean contains(List<Method> list, Method method) {
        if (list == null || list.isEmpty()) {
            return false;
        }
        for (Method m : list) {
            if (!m.getName().equals(method.getName())) {
                continue;
            }
            TypeVariable[] var = m.getTypeParameters();
            TypeVariable[] var2 = method.getTypeParameters();
            if (var == null && var2 == null) {
                return true;
            }
            //判断参数个数是否相同
            if (var.length != var2.length) {
                continue;
            }
            //判断参数类型是否相同
            boolean flag = true;
            for (int i = 0; i < var.length; i++) {
                if (!var[i].getTypeName().equals(var2[i].getTypeName())) {
                    flag = false;
                    break;
                }
            }
            if (flag) {
                return true;
            }
        }
        return false;
    }

    /**
     * 获取泛型类型
     *
     * @param field
     * @return
     */
    public static Class getGenericType(Field field) {
        Type type = field.getGenericType();
        Class<?> genericClazz = Map.class;
        if (type instanceof ParameterizedType) { // 如果是泛型参数的类型
            ParameterizedType paramType = (ParameterizedType) type;
            type = paramType.getActualTypeArguments()[0];
            if (type instanceof Class<?>) {
                genericClazz = (Class<?>) type; // 得到泛型里的class类型对象
            }
        }
        return genericClazz;
    }

    /**
     * 直接读取对象属性值, 无视private/protected修饰符, 不经过getter方法.
     *
     * @param obj
     * @param fieldName
     * @return
     */
    public static Object getFieldValue(final Object obj, final String fieldName) {
        List<Field> fields = getFields(obj.getClass());
        if (fields != null && !fields.isEmpty()) {
            for (Field field : fields) {
                if (field.getName().equals(fieldName)) {
                    if (!field.isAccessible()) {
                        field.setAccessible(true);
                    }
                    Object value = null;
                    try {
                        value = field.get(obj);
                    } catch (IllegalAccessException e) {
                        logger.error(e.getMessage(), e);
                    }
                    return value;
                }
            }
        }
        throw new IllegalArgumentException("Could not find field [" + fieldName + "] on target [" + obj + "]");
    }

    /**
     * 直接设置对象属性值, 无视private/protected修饰符, 不经过setter方法.
     *
     * @param instance
     * @param fieldName
     * @return
     */
    public static void setFieldValue(Object instance, String fieldName, Object value) {
        List<Field> fields = getFields(instance.getClass());
        if (fields != null && !fields.isEmpty()) {
            for (Field field : fields) {
                if (field.getName().equals(fieldName)) {
                    if (!field.isAccessible()) {
                        field.setAccessible(true);
                    }
                    try {
                        field.set(instance, value);
                    } catch (IllegalAccessException e) {
                        logger.error(e.getMessage(), e);
                    }
                    return;
                }
            }
        }
        throw new IllegalArgumentException("Could not find field [" + fieldName + "] on target [" + instance + "]");
    }

    /**
     * 直接设置对象属性值, 无视private/protected修饰符, 不经过setter方法.
     *
     * @param instance
     * @param field
     * @return
     */
    public static void setFieldValue(Object instance, Field field, Object value) {
        if (!field.isAccessible()) {
            field.setAccessible(true);
        }
        try {
            field.set(instance, value);
        } catch (IllegalAccessException e) {
            logger.error(e.getMessage(), e);
        }
    }

    public static boolean isVoidReturnType(Method method) {
        return method.getReturnType() == Void.TYPE;
    }
}
