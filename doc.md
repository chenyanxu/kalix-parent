# kalix develop manual
## config admin
### class code
    public class Applicationconfig implements ManagedService {}
### blueprint code
    <service id="managedService" interface="org.osgi.service.cm.ManagedService" ref="applicationconfig">
            <service-properties>
                <entry key="service.pid" value="ConfigApplication" />
            </service-properties>
    </service>
## event admin
### create event post
    public class ShiroRealm extends AuthorizingRealm implements IAuthorizingRealm {
      private IUserLoginService userLoginService;
      private EventAdmin eventAdmin;

### blueprint file:

    <bean id="auditUserLoginEventImpl" class="com.kalix.admin.audit.biz.biz.AuditUserLoginEventImpl">
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
            </service>

## blueprint 中使用BundleContext
### declare
    public class Applicationconfig implements ManagedService {
        private EventAdmin eventAdmin;
        private BundleContext bundleContext;

### blueprint use
    <bean id="applicationconfig" class="com.kalix.framework.core.impl.web.Applicationconfig">
            <property name="bundleContext" ref="blueprintBundleContext"/>
    </bean>

