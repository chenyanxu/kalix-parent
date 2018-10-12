## shiro session配置位置
    D:\java-develop\project\framework-parent\core-parent\framework-core-security\src\main\resources\OSGI-INF\blueprint\security.xml
    <bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
        <property name="globalSessionTimeout" value="1800000"/>
        <property name="sessionDAO" ref="customShiroSessionDAO"/>
        <property name="sessionFactory" ref="SessionFactory"/>
    </bean>
## 程序验证位置
    com.kalix.framework.core.security.impl.ShiroServiceImpl
    getCurrentUserId()
    