---
title: JUnit + Spring Test
date: 2016-09-05 13:35:49
tags: Spring
---

## Gradle 依赖
```groovy
testCompile 'org.springframework:spring-test:4.3.0.RELEASE'
testCompile 'junit:junit:4.12'
```

## Test 例子

```java
import org.apache.commons.configuration.PropertiesConfiguration;
import org.junit.runner.RunWith;
import org.junit.Test;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;
import java.util.Properties;

@RunWith(SpringRunner.class)
@ContextConfiguration({"classpath:spring-beans-config.xml"})
public class TestYamlPropertiesAndPropertiesConfig {
    @Resource(name = "yamlProperties")
    private Properties yamlProperties;

    @Resource(name = "propertiesConfig")
    private PropertiesConfiguration propertiesConfig;

    @Test
    public void testYamlProperties() {
        System.out.println(yamlProperties.getProperty("mysql.jdbc.url"));
        System.out.println(yamlProperties.getProperty("username"));
    }

    @Test
    public void testPropertiesConfig() {
        System.out.println(propertiesConfig.getString("username"));
        System.out.println(propertiesConfig.getInteger("age", 0));
    }
}
```

> @ContextConfiguration({"classpath:spring-beans-config.xml"}) 用于加载 Spring Bean 的配置文件
> 
> 优点是不需要手动创建 ApplicationContext 了，也能使用 @Autowired 和 @Resource 等注入 Bean
