---
title: JavaGuns 学习
published: 2026-01-13 09:50:48
description: 记录学习过程中的细节
tags: [java]
category: 学习
draft: false
---

使用的 JavaGuns 框架的版本是 7.3.2

## 关于 BaseEntity

```diff lang="java" title="BaseEntity定义"
@Data
public class BaseEntity implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 创建时间
     */
    @TableField(value = "create_time", fill = FieldFill.INSERT)
    private Date createTime;

    /**
     * 创建人
     */
    @TableField(value = "create_user", fill = FieldFill.INSERT)
    private Long createUser;

    /**
     * 更新时间
     */
    @TableField(value = "update_time", fill = FieldFill.UPDATE)
    private Date updateTime;

    /**
     * 更新人
     */
    @TableField(value = "update_user", fill = FieldFill.UPDATE)
    private Long updateUser;

}
```

JavaGuns 使用 BaseEntity 来管理创建时间、创建人、修改时间、修改人这四个字段，但是实际使用下来还是发现有一些不合理的地方。

1. 通过 FieldFill 可以知道，修改时间和修改人这两个字段在第一次创建的时候是没有进行任何处理的，这不符合业务逻辑，第一次创建的时候，修改时间和修改人应该和创建时间和创建人保持一致。
2. JavaGuns 中 CustomMetaObjectHandler 实现了 MetaObjectHandler 类用于自动填充字段，但是对于 update 方法，每次都会自动填充更新时间和更新人，即使数据没有任何修改，只要触发了 update 就会更新时间，这也不符合业务逻辑。

```diff lang="java" title="CustomMetaObjectHandler中的部分方法"
    @Override
    public void insertFill(MetaObject metaObject) {

        try {
            // 设置createUser（BaseEntity)
            setValue(metaObject, CREATE_USER, this.getUserUniqueId());

            // 设置createTime（BaseEntity)
            setValue(metaObject, CREATE_TIME, new Date());

+           // 设置updateUser（BaseEntity) 
+           setValue(metaObject, UPDATE_USER, this.getUserUniqueId());

            // 设置删除标记 默认N-删除
            setDelFlagDefaultValue(metaObject);

            // 设置状态字段 默认1-启用
            setStatusDefaultValue(metaObject);

            // 设置乐观锁字段，从0开始
            setValue(metaObject, VERSION_FLAG, 0L);

            // 设置组织id
            setValue(metaObject, ORG_ID, this.getUserOrgId());

        } catch (ReflectionException e) {
            log.warn("CustomMetaObjectHandler处理过程中无相关字段，不做处理");
        }

    }

    @Override
    public void updateFill(MetaObject metaObject) {

        try {
            // 设置updateUser（BaseEntity)
            setValue(metaObject, UPDATE_USER, this.getUserUniqueId());

            // // 设置updateTime（BaseEntity)
            setValue(metaObject, UPDATE_TIME, new Date());
        } catch (ReflectionException e) {
            log.warn("CustomMetaObjectHandler处理过程中无相关字段，不做处理");
        }

    }
```

所以最后的修改是在自己的表中不要继承 BaseEntity，让 MySQL 管理 update_time 字段，让 MyBatis 管理 update_user 字段，同时修改 insertFill 方法让 insert 和 update 都会自动填充 update_user 字段。

```java
    /**
     * 创建时间
     */
    @TableField(value = "create_time", fill = FieldFill.INSERT)
    private Date createTime;

    /**
     * 创建人
     */
    @TableField(value = "create_user", fill = FieldFill.INSERT)
    private Long createUser;

    /**
     * 更新时间
     */
    @TableField(value = "update_time")
    private Date updateTime;

    /**
     * 更新人
     */
    @TableField(value = "update_user", fill = FieldFill.INSERT_UPDATE)
    private Long updateUser;
```

执行这段 SQL 语句将 update_time 修改成为依赖其他字段的更新自动更新时间。

```sql
MODIFY COLUMN update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间';
```

:::important
MySQL 中的 ON UPDATE 只有在其他字段发生数据变更的时候才会触发，即使当 MyBatis 向某个字段，比如 update_user 写入了相同的数据，也不会触发 update_time 的更新，符合业务逻辑。
:::