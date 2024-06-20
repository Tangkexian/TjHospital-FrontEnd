package com.javaee.onlinehosbackend.extrautil.context;


import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.event.ApplicationEnvironmentPreparedEvent;
import org.springframework.boot.context.event.ApplicationPreparedEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;

/**
 * @Description
 * @Author sugar
 * @Date 2020/10/19 11:58
 */
@Slf4j
public class AppListeners {
    public static class EnvironmentPreparedEventListener implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {
        @Override
        public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
            log.info("--------[Application environment prepared]--------");

        }
    }

    public static class ApplicationPreparedEventListener implements ApplicationListener<ApplicationPreparedEvent> {
        @Override
        public void onApplicationEvent(ApplicationPreparedEvent event) {
            log.info("--------[Application context prepared]--------");
            AppContext.setApplicationContext(event.getApplicationContext());
        }
    }

    public static class ContextRefreshedEventListener implements ApplicationListener<ContextRefreshedEvent> {
        @Override
        public void onApplicationEvent(ContextRefreshedEvent event) {
            if (event.getApplicationContext().getParent() != null) {
                log.info("=========[Application context refreshed]=========");
                AppContext.setApplicationContext(event.getApplicationContext());
            }
        }
    }
}
