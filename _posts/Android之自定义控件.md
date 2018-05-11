---
title: Android之自定义控件
date: 2018-04-03 10:19:58
tags:
---

Author JOY.
<!-- excerpt -->

### 坐标系
#### Android坐标系
左上角为坐标原地(0, 0)。
#### 视图坐标系
getLeft(),getRight(),getTop(),getBottom(),getX(),getY()相对父布局
getRawX(),getRawY()相对Android坐标系原点

### 自定义组合控件
自定义组合控件就是多个控件组合起来成为一个新的控件，主要用来解决多次重复的使用同一类型的布局。比如我们应用的顶部的标题栏，还有弹出的固定样式的dialog，这些都是常用的，所以把他们所需要的控件组合起来重新定义成一个新的控件。

```java
public class XXXClass extends XXXLayout {
  public XXXClass(Context context) {
     super(context);
     initView(context);
  }

  public XXXClass(Context context, AttributeSet attrs) {
     super(context, attrs);
     initView(context);
  }

  public XXXClass(Context context, AttributeSet attrs, int    defStyleAttr) {
     super(context, attrs, defStyleAttr);
     initView(context);
  }

  public void initView(Context context){
    /*
    自定义组合控件布局，初始化，对外提供操作接口。
    */
     LayoutInflater.from(context).inflate(R.layout.xxx, this, true);
  }
}
```
