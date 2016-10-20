# 级联删除使用说明

- [概述](##概述)
- [实现思路](#实现思路)
    - [注册](#注册)
    - [解析](#解析)
- [在实体类中定义依赖关系](#在实体类中定义依赖关系)
- [OSGI实体类Bundle启动注册级联关系](#OSGI实体类Bundle启动注册级联关系)
    
## 概述
> 在业务中经常有多表依赖关系的存在，如C依赖于B，B依赖于A，在对A进行删除操作时，
> 希望可以删除掉B表中关联A表的信息，同时删除C表中关联B表的信息。

```java

```
> 我们可以使用OneToOne、ManyToOne关系注解，让OpenJPA去做这样的级联删除操作，但
> 由于我们当前使用的是OSGI的框架，需要减少Bundle的相互依赖，即在A中不引用B，
> B中不引用C。这样我们需要定义自己的级联删除关系，并通过代码来实现。

## 实现思路

### 注册

> 在实体类Bundle启动时，注册自己与其他实体类的依赖关系。继承OSGI的BundleActivator
> 接口，实现抽象类BundleActivatorImpl，在BundleActivatorImpl中实现依赖关系的注册
> 方法。需要注册依赖关系的实体类Bundle，只要继承BundleActivatorImpl并调用注册方法
> 即可实现实体类依赖关系的注册。

```java
    /**
     * 注册级联信息
     *
     * @param me    注册的实体类
     * @throws Exception
     */
    public void registerCascade(Class<? extends PersistentEntity> me) throws Exception {
        // 获取当前需要注册实体类的table注解信息
        Table table = me.getAnnotation(Table.class);
        // 没有table注解信息，不需要注册
        if (table == null) {
            return;
        }

        // 从redis中取出级联信息
        String jedisString = cacheManager.get(KalixCascade.alias);
        JSONObject jsonCascade = new JSONObject();
        if (jedisString != null) {
            jsonCascade = new JSONObject(jedisString);
        }

        Field[] fields = me.getDeclaredFields();
        for (Field f : fields) {
            // 查找是否有依赖注解关系
            KalixCascade cascade = f.getAnnotation(KalixCascade.class);
            if (cascade != null) {
                // 是否需要级联删除
                if (cascade.deletable()) {
                    // 保存级联删除信息到json
                    JSONObject object = new JSONObject();
                    object.put("operation", "delete");
                    object.put("table", table.name());
                    object.put("primaryKey", "id");
                    object.put("foreignKey", cascade.foreignKey());

                    jsonCascade = writeJson(jsonCascade, cascade.beans(), me.getName(), object);
                }
            }
        }
        // 保存级联信息到redis
        cacheManager.save(KalixCascade.alias, jsonCascade.toString());
    }
```

> 考虑到在实际业务操作中，可能会有取消依赖的需求，同时增加对实体类反注册的实现。

```java
    /**
     * 反注册级联信息
     *
     * @param me 反注册的实体类
     * @throws Exception
     */
    public void unRegisterCascade(Class<? extends PersistentEntity> me) throws Exception {
        if (cacheManager.exists(KalixCascade.alias)) {
            String jedisString = cacheManager.get(KalixCascade.alias);
            // 存在级联信息
            if (jedisString != null) {
                boolean isChange = false;
                JSONObject jsonCascade = new JSONObject(jedisString);
                // 遍历json级联信息
                for (Iterator mainCascadeIterator = jsonCascade.keys(); mainCascadeIterator.hasNext(); ) {
                    String mainCascadeKey = (String) mainCascadeIterator.next();
                    JSONObject mainJsonCascade = jsonCascade.getJSONObject(mainCascadeKey);
                    // 有需要反注册的信息
                    if (mainJsonCascade.has(me.getName())) {
                        isChange = true;
                        // 删除
                        mainJsonCascade.remove(me.getName());
                        // 保存至json
                        jsonCascade.put(mainCascadeKey, mainJsonCascade);
                    }
                }

                // 有反注册信息，修改redis缓存
                if (isChange) {
                    cacheManager.save(KalixCascade.alias, jsonCascade.toString());
                }
            }
        }
    }
```

> redis中存储的依赖关系json串

```json
{
  "com.kalix.admin.core.entities.OrganizationBean": {
    "com.kalix.schedule.plan.personalplan.entities.PersonalPlanBean": {
      "operation": "delete",
      "foreignKey": "orgId",
      "table": "schedule_personalplan",
      "primaryKey": "id"
    },
    "com.kalix.admin.duty.entities.DutyBean": {
      "operation": "delete",
      "foreignKey": "orgid",
      "table": "sys_duty",
      "primaryKey": "id"
    },
    "com.kalix.schedule.plan.workreport.entities.WorkReportBean": {
      "operation": "delete",
      "foreignKey": "orgId",
      "table": "schedule_workreport",
      "primaryKey": "id"
    }
  },
  "com.kalix.admin.duty.entities.DutyBean": {
    "com.kalix.admin.duty.entities.DutyUserBean": {
      "operation": "delete",
      "foreignKey": "dutyId",
      "table": "sys_duty_user",
      "primaryKey": "id"
    }
  },
  "com.kalix.admin.core.entities.UserBean": {
    "com.kalix.schedule.plan.personalplan.entities.PersonalPlanBean": {
      "operation": "delete",
      "foreignKey": "userId",
      "table": "schedule_personalplan",
      "primaryKey": "id"
    },
    "com.kalix.schedule.plan.workreport.entities.WorkReportBean": {
      "operation": "delete",
      "foreignKey": "userId",
      "table": "schedule_workreport",
      "primaryKey": "id"
    }
  }
}
```

### 解析

> 在删除某个实体类时，从redis中取出依赖关系信息，并使用递归方式搜素是否有关于自己的依赖信息，
> 同时组合好需要执行的sql语句。

```java
    @Override
    public void beforeDeleteEntity(Long id, JsonStatus status) {
        String jedisString = cacheManager.get(KalixCascade.alias);
        if (jedisString != null && !jedisString.isEmpty()) {
            Map<String, String> map = new HashMap<>();
            map.put(super.persistentClass.getName(), "");
            // 递归方式搜素是否有关于自己的依赖信息，并存储在map中
            map = getCascade(map, new JSONObject(jedisString), super.persistentClass.getName(), id, "");

            for (String key : map.keySet()) {
                if (map.get(key) != null && !map.get(key).isEmpty()) {
                    dao.updateNativeQuery(map.get(key));
                }
            }
        }
    }
    
    /**
     * 解析redis中存储的级联信息
     *
     * @param map   查询结果map<实体类名, 执行语句> 
     * @param jsonCascade   redis中存储的级联信息
     * @param cascadeKey    要解析的key（实体类）
     * @param id    删除的实体类主键值
     * @param searchSql 拼接的操作查询串
     * @return
     */
    private Map<String, String> getCascade(Map<String, String> map, JSONObject jsonCascade, String cascadeKey, Long id, String searchSql) {
        if (jsonCascade.has(cascadeKey)) {
            JSONObject mainJsonCascade = jsonCascade.getJSONObject(cascadeKey);

            for (Iterator iter = mainJsonCascade.keys(); iter.hasNext(); ) {
                String key = (String) iter.next();
                // 判断需要查找的key是否在map中存在，防止循环递归
                if (map.get(key) == null || map.get(key).isEmpty()) {
                    JSONObject object = mainJsonCascade.getJSONObject(key);
                    if (object != null) {
                        String operation = object.get("operation").toString();
                        String table = object.get("table").toString();
                        String primaryKey = object.get("primaryKey").toString();
                        String foreignKey = object.get("foreignKey").toString();

                        String operationSql = "";
                        String mySearchSql = "";
                        
                        if (searchSql == "") {
                            // 不存在查询语句
                            operationSql = operation + " from " + table + " where " + foreignKey + " = " + id;
                            mySearchSql = "select " + primaryKey + " from " + table + " where " + foreignKey + " = " + id;
                        } 
                        else { 
                            // 存在查询语句，拼接in条件
                            operationSql = operation + " from " + table + " where " + foreignKey + " in (" + searchSql + ") ";
                            mySearchSql = "select id from " + table + " where " + foreignKey + " in (" + searchSql + ") ";
                        }

                        // 递归查询
                        map = getCascade(map, jsonCascade, key, id, mySearchSql);
                        
                        // 查询结果存储于map
                        map.put(key, operationSql);
                    }
                }
            }
        }

        return map;
    }
```

## 在实体类中定义依赖关系

```java
    @KalixCascade(beans = "com.kalix.admin.core.entities.OrganizationBean", deletable = true, foreignKey = "orgid")
    private long orgid;   //所在机构
```

    @KalixCascade 自定义级联关系注解
    beans 当前属性（字段）所依赖的实体类（全名）
    deletable 是否级联删除
    foreignKey 当前属性（字段）在表中的字段名（外键）

## OSGI实体类Bundle启动注册级联关系

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