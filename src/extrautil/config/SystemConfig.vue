package com.javaee.onlinehosbackend.extrautil.config;


import com.javaee.onlinehosbackend.extrautil.context.AppContext;
import lombok.NonNull;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.MapPropertySource;
import org.springframework.core.env.StandardEnvironment;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Properties;

/**
 * 系统配置工具类
 *
 * @Author Sugar
 * @Version 2018/12/7 14:09
 */
public class SystemConfig extends org.springframework.beans.factory.config.PropertyPlaceholderConfigurer {
    private static ConfigurableEnvironment env;
    private static MapPropertySource mapPropertySource;

    public static void setEnvironment(@NonNull ConfigurableEnvironment environment) {
        env = environment;
        if (mapPropertySource != null) {
            env.getPropertySources().addFirst(mapPropertySource);
        }
    }

    private static ConfigurableEnvironment getEnv() {
        if (env == null) {
            env = AppContext.getBean(ConfigurableEnvironment.class);
        }
        return env;
    }

    /**
     * 读取配置，可能为null
     *
     * @param key
     * @return
     */
    public static String getConfig(String key) {
        return getEnv().getProperty(key);
    }

    /**
     * 如果为null，则返回空字符串
     *
     * @param key
     * @return
     */
    public static String getString(String key) {
        return getString(key, "");
    }

    public static String getString(String key, String def) {
        String val = getEnv().getProperty(key);
        if (val == null) {
            return def;
        }
        return val.trim();
    }

    public static int getInt(String key) {
        return getInteger(key, 0);
    }

    public static int getInt(String key, int def) {
        return getInteger(key, def);
    }

    public static Integer getInteger(String key, Integer def) {
        String val = getString(key, "");
        if (val.matches("-?\\d+(\\.\\d+)")) {
            return Integer.valueOf(val.substring(0, val.indexOf(".")));
        }
        if (!val.matches("-?\\d+")) {
            return def;
        }
        return Integer.valueOf(val);
    }

    public static long getLong(String key) {
        return getLong(key, 0L);
    }

    public static long getLong(String key, long def) {
        return getLong(key, Long.valueOf(def));
    }

    public static Long getLong(String key, Long def) {
        String val = getString(key, "");
        if (val.matches("-?\\d+(\\.\\d+)")) {
            return Long.valueOf(val.substring(0, val.indexOf(".")));
        }
        if (!val.matches("-?\\d+")) {
            return def;
        }
        return Long.valueOf(val);
    }

    public static float getFloat(String key) {
        return getFloat(key, 0F);
    }

    public static float getFloat(String key, float def) {
        return getFloat(key, Float.valueOf(def));
    }

    public static Float getFloat(String key, Float def) {
        String val = getString(key, "");
        if (!val.matches("-?\\d+(\\.\\d+)?")) {
            return def;
        }
        return Float.valueOf(val);
    }

    public static double getDouble(String key, double def) {
        return getDouble(key, Double.valueOf(def));
    }

    public static Double getDouble(String key, Double def) {
        String val = getString(key, "");
        if (!val.matches("-?\\d+(\\.\\d+)?")) {
            return def;
        }
        return Double.valueOf(val);
    }

    public static boolean getBoolean(String key) {
        return getBoolean(key, Boolean.valueOf(false));
    }

    public static boolean getBoolean(String key, boolean def) {
        return getBoolean(key, Boolean.valueOf(def));
    }

    public static Boolean getBoolean(String key, Boolean def) {
        String val = getString(key, "");
        if (val.isEmpty()) {
            return def;
        }
        return val.equalsIgnoreCase("true");
    }

    @Override
    protected void convertProperties(Properties properties) {
        HashMap<String, Object> map = new HashMap<>();
        Iterator<Object> keys = properties.keySet().iterator();
        while (keys.hasNext()) {
            String key = keys.next().toString();
            String value = properties.getProperty(key);
            map.put(key, value);
        }
        super.convertProperties(properties);
        mapPropertySource = new MapPropertySource("configMap", map);
        if (env == null) {
            env = new StandardEnvironment();
        }
        getEnv().getPropertySources().addFirst(mapPropertySource);
    }
}
