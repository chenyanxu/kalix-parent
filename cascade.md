# 级联删除使用说明

## 概述
    在业务中经常有多表依赖关系的存在，如C依赖于B，B依赖于A，在对A进行删除操作时，希望可以删除掉B表中关联A表的信息，同时删除C表中关联B表的信息。
    
    我们可以使用OneToOne、ManyToOne关系注解，让OpenJPA去做这样的级联删除操作，但由于我们当前使用的是OSGI的框架，需要减少bundle的相互依赖，即及在A中不知道存在B，B中不知道存在C。这样我们需要定义自己的级联删除关系，并通过代码来实现。
    
## 在实体类中定义依赖关系

```java
    @KalixCascade(beans = "com.kalix.admin.core.entities.OrganizationBean", deletable = true, foreignKey = "orgid")
    private long orgid;   //所在机构
```

    @KalixCascade 自定义级联关系注解
    beans 当前属性（字段）所依赖的实体类（全名）
    deletable 是否级联删除
    foreignKey 当前属性（字段）在表中的字段名（外键）

## OSGI 实体类Bundle启动，注册级联关系

```java
// 继承抽象类BundleActivatorImpl
public class InitActivator extends BundleActivatorImpl implements BundleActivator {
    @Override
    public void start(BundleContext bundleContext) throws Exception {
        super.start(bundleContext);
        // 反注册当前实体类
        unRegisterCascade(DutyBean.class);  
        unRegisterCascade(DutyUserBean.class);
        // 重新注册当前实体类
        registerCascade(DutyBean.class);
        registerCascade(DutyUserBean.class);
    }
}
```