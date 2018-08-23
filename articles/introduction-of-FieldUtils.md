
## FieldUtils的介绍以及简单用法

### 背景

前几天我需要做一个功能，如何判断一个对象中的各个字段是否有空值，这个对象是数据对象，里面的属性皆为私有(private)属性。
于是便用到了获取Field的值，在网上搜了一下，发现了一种方案，方案如下：

```
   // 获取这个对象的所有属性
   // 也有一种写法是 obj.getClass().getDeclaredFields() 目的一样
   Field[] fields = obj.getClass().getFields();    
   for (Field field : fields) {
     try {
       //设置访问权限(私有的也可以)，不做检查 直接取值
       field.setAccessible(true);
       if(field.get(obj) == null){
          return true;
       }
     } catch (IllegalAccessException e) {
       e.printStackTrace();
     }
   }
```

然鹅，我却并达不到目的，获取不到任何一个属性，我就觉得这种方法不靠谱。于是便想着有没有现成的工具类呢，IDEA打出“FieldUtils”果然有，于是便有了下面的解决方案，完美达成目的：

```java
class FieldCheckUtils {
  /**
   * 检查对象中所有字段是否存在空
   *
   * @param obj 需要检查的对象
   * @return 结果
   */
  static boolean fieldHasNull(Object obj) {
    if (obj == null) {
      return true;
    }
    Field[] fields = FieldUtils.getAllFields(obj.getClass());
    for (Field field : fields) {
      try {
        // 设置属性是可以访问的(私有的也可以)，不做检查 直接取值
        field.setAccessible(true);
        if (isNull(field.get(obj))) {
          return true;
        }
      } catch (IllegalAccessException e) {
        e.printStackTrace();
      }
    }
    return false;
  }

  /**
   * 检查对象中所有字段是否都有值
   *
   * @param obj 需要检查的对象
   * @return 结果
   */
  static boolean fieldHasNoneNull(Object obj) {
    return !fieldHasNull(obj);
  }

  private static boolean isNull(String str) {
    return str == null || "".equals(str.trim()) || "null".equals(str.toLowerCase());
  }

  private static boolean isNull(Object obj) {
    return obj == null || isNull(obj.toString());
  }

}
```

那么为什么`obj.getClass.getFields()`没有效果呢？以及对于FieldUtils还能了解到哪些呢？

### `Class`的`getFields()`怎么失灵了？

仔细看了一下Api文档，发现不是失灵，而是使用方法不对。点击查看接口文档：

```java
/**
     * Returns an array containing {@code Field} objects reflecting all
     * the accessible public fields of the class or interface represented by
     * this {@code Class} object.
     *
     * ....太多了省略
     * @since JDK1.1
     * @jls 8.2 Class Members
     * @jls 8.3 Field Declarations
     */
    @CallerSensitive
    public Field[] getFields() throws SecurityException {
        checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), true);
        return copyFields(privateGetPublicFields(null));
    }
```

这才发现人家只对公有属性起作用，也就是说把数据对象中的private换成public就行了，我也这么试了，能够完成功能。
不过一般不能这么做，最终便采用了`FieldUtils`的方案。

### FieldUtils简介

通过反射结合`Field`的工具类，从休眠【反射】Commons沙盒组件改编和重构而来。

提供的能力是打破程序员编码的范围限制。这可以允许修改不应该被改变的字段，应该慎用这种便捷之处。

