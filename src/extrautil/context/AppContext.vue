package com.javaee.onlinehosbackend.extrautil.context;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;

@Component
@Slf4j
public class AppContext {
    /**
     * 上下文对象实例
     */
    private static ApplicationContext applicationContext;

    public AppContext() {

    }

    public AppContext(ApplicationContext context) {
        applicationContext = context;
    }

    /**
     * 获取applicationContext
     *
     * @return
     */
    public static ApplicationContext getApplicationContext() {
        if (applicationContext != null) {
            return applicationContext;
        }
        synchronized (AppContext.class) {
            if (applicationContext != null) {
                return applicationContext;
            }
            log.info("ApplicationContext未初始化，自动加载：classpath*:/**/spring-context*.xml");
            try {
                applicationContext = new ClassPathXmlApplicationContext(new String[]{
                        "classpath*:/**/spring-context*.xml"});
            } catch (BeansException e) {
                e.printStackTrace();
            }
        }

        Assert.state(applicationContext != null, "ApplicationContext have not been initialized yet, please call AppContext.setApplicationContext(context) to initialize it");
        return applicationContext;
    }

    /**
     * 在WEB项目中可以使用WebApplicationContextUtils.getWebApplicationContext(servletContext)获取Context进行初始化，以便与Web容器使用同一个Context;
     *
     * @param context
     */
    public static void setApplicationContext(ApplicationContext context) {
        applicationContext = context;
    }

    /**
     * 通过name获取 Bean.
     *
     * @param name 参数传入要获取的实例的类名 首字母小写，这是默认的
     * @return
     */
    public static <T> T getBean(String name) {
        try {
            return (T) getApplicationContext().getBean(name);
        } catch (NoSuchBeanDefinitionException ex) {
            log.debug(ex.getMessage());
            return null;
        }
    }

    /**
     * 通过class获取Bean.
     *
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T getBean(Class<T> clazz) {
        try {
            return getApplicationContext().getBean(clazz);
        } catch (NoSuchBeanDefinitionException ex) {
            log.debug(ex.getMessage());
            return null;
        }
    }

    /**
     * 通过name,以及Clazz返回指定的Bean
     *
     * @param name
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T getBean(String name, Class<T> clazz) {
        try {
            return getApplicationContext().getBean(name, clazz);
        } catch (NoSuchBeanDefinitionException ex) {
            log.debug(ex.getMessage());
            return null;
        }
    }
}