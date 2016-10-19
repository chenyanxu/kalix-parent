# faq

## 部署配置

### kalix项目的发布路径

1. 修改配置文件（/etc/ConfigWebContext.cfg）
```xml
#config for web context
path=/
#false load uncompress js/css version. true load compress version
deploy = true
```

2. 修改webcontext（framework-parent\webapp-parent\framework-webapp-main\osgi.bnd）

```xml
#-----------------------------------------------------------------
# Use this file to add customized Bnd instructions for the bundle
#-----------------------------------------------------------------

Bundle-Activator: ${bundle.namespace}.internal.InitActivator
Import-Package: com.kalix.framework.core.security.authc.filter,\
                com.kalix.framework.core.impl.web,\
                com.kalix.framework.core.security.web.evn,\
                com.kalix.framework.core.security.filter,\
                org.apache.camel.component.servlet, \
                org.apache.shiro,org.apache.shiro.web.env, \
                org.apache.shiro.web.servlet, \
                org.apache.shiro.subject, \
                org.apache.shiro.web.tags, \
                javax.servlet.jsp,\
                org.apache.shiro.session,*
Include-Resource: src/main/webapp,src/main/resources
Web-ContextPath: /
Bundle-Category: Kalix Framework WebApp
```