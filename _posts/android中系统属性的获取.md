---
title: android中系统属性的获取
date: 2018-01-17 17:16:09
tags:
---

Author JOY.
<!-- excerpt -->

### 获取java属性：
System.getProperty("user.name")；

### 三方应用获取系统属性：

```java
private String getAndroidOsSystemProperties(String key) {
    String ret;
    Method systemProperties_get = null;
    try {
        systemProperties_get = Class.forName("android.os.SystemProperties").getMethod("get", String.class);
        if ((ret = (String) systemProperties_get.invoke(null, key)) != null)
            return ret;
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
    return "";
}
```
### 内置应用获取系统属性：
SystemProperties.get("property_name"))

[相关参考](http://joyflyaway.com/2016/10/24/Android系统属性/)
