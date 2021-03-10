## 一、PropertyPlaceholderConfigurer
加载方式:
``` xml
<context:property-placeholder location="classpath:xxx.properties"/>
<!-- 需要添加context的声明 -->
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation=
“http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd”
```
或
``` xml
<bean id="configProperties" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="ignoreResourceNotFound" value="true"/>
    <property name="locations">
    <list>
        <value>classpath:xxx.properties</value>
    </list>
    </property>
</bean>
```
这种方式下，可以用 **${key}** 读取，或者在字段上使用 **@Value** 读取




## 二、PropertiesFactoryBean
加载方式：
``` xml
<util:properties id="prop" location="classpath:xxx.properties"/>
<!-- 需要添加util的声明 -->
xmlns:util="http://www.springframework.org/schema/util"
xsi:schemaLocation="http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
```
或
``` xml
<bean id="prop" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
    <property name="ignoreResourceNotFound" value="true"/>
    <property name="locations">
    <list>
        <value>classpath:/xxx.properties</value>
    </list>
    </property>
</bean>
```
这种方式下，可以用 **#{prop['key']}** 读取



## 三、@PropertySource

加载方式：

```java
@Configuration
@PropertySource("classpath:my.properties")
public class MyConfig {

}
```

使用方式：在字段上使用 **@Value** 读取，通过`Environment#getProperty`获取

