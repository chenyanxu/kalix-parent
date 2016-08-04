# kalix develop manual
## config admin
## event admin
### create event post
``public class ShiroRealm extends AuthorizingRealm implements IAuthorizingRealm {
      private IUserLoginService userLoginService;
      private EventAdmin eventAdmin;``

``<bean id="auditUserLoginEventImpl" class="com.kalix.admin.audit.biz.biz.AuditUserLoginEventImpl">
            <property name="dao" ref="auditBeanDao"/>
        </bean>
        <service interface="org.osgi.service.event.EventHandler" ref="auditUserLoginEventImpl">
            <service-properties>
                <entry key="event.topics">
                    <array value-type="java.lang.String">
                        <value>com/kalix/userlogin</value>
                        <value>com/kalix/userlogout</value>
                    </array>
                </entry>
            </service-properties>
        </service>``

