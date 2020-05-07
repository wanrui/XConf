## 认证和授权

采用` jwt + cashbin` 实现授权管理分，整体的架构如下

### 认证
在大多数情况下，应该使用外部身份验证服务，因为外部验证服务可以统一的进行用户管理。如果没有外部身份验证服务，您也可以通过本地身份验证来管理系统用户。

第三方认证规则支持

- Gitlab认证
- LDAP认证
- Google认证 

### 授权
fen


casbin 授权规则
```mariadb
CREATE TABLE `casbin_rule` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `p_type` varchar(255) NOT NULL DEFAULT '',
  `v0` varchar(255) NOT NULL DEFAULT '',
  `v1` varchar(255) NOT NULL DEFAULT '',
  `v2` varchar(255) NOT NULL DEFAULT '',
  `v3` varchar(255) NOT NULL DEFAULT '',
  `v4` varchar(255) NOT NULL DEFAULT '',
  `v5` varchar(255) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=106 DEFAULT CHARSET=utf8mb4;
```

```mariadb
CREATE TABLE `data_perm` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `domain_id` int(11) unsigned NOT NULL DEFAULT '1' COMMENT '项目域id',
  `parent_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '菜单ID',
  `name` varchar(50) NOT NULL DEFAULT '' COMMENT '名称',
  `perms` varchar(100) NOT NULL DEFAULT '' COMMENT '数据权限标识',
  `perms_rule` varchar(500) NOT NULL DEFAULT '' COMMENT '数据规则',
  `perms_type` tinyint(3) NOT NULL DEFAULT '1' COMMENT '1=分类 2=数据权限',
  `order_num` int(11) unsigned NOT NULL DEFAULT '1' COMMENT '排序',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `remarks` varchar(500) NOT NULL DEFAULT '' COMMENT '说明',
  PRIMARY KEY (`id`),
  KEY `idx_data_perm_domain_id` (`domain_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COMMENT='数据权限管理';
```
项目域表（数据表）
```mariadb
CREATE TABLE `domain` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '名称',
  `callbackurl` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '回调链接',
  `remark` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '说明',
  `code` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '标识',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `last_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC COMMENT='项目域表';
```

菜单表
```mariadb
CREATE TABLE `menu` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `parent_id` int(11) NOT NULL DEFAULT '0' COMMENT '上级菜单id',
  `domain_id` int(11) NOT NULL DEFAULT '0' COMMENT '项目域id',
  `name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '菜单名称',
  `url` tinytext COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '路由url',
  `perms` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '授权标识',
  `alias` text COLLATE utf8mb4_unicode_ci,
  `menu_type` int(11) NOT NULL DEFAULT '0' COMMENT '类型 1=菜单 2=按钮',
  `icon` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '图标',
  `order_num` int(11) NOT NULL DEFAULT '0' COMMENT '排序',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `last_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_menu_domain_id` (`domain_id`) USING BTREE,
  KEY `perms_index` (`perms`(191))
) ENGINE=InnoDB AUTO_INCREMENT=105 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC COMMENT='菜单表';

/*Data for the table `menu` */
```


角色表
```mariadb
CREATE TABLE `role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '角色名称',
  `domain_id` int(11) NOT NULL DEFAULT '1' COMMENT '项目域id',
  `role_name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '角色名称',
  `remark` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '备注',
  `menu_ids` text COLLATE utf8mb4_unicode_ci,
  `menu_ids_ele` text COLLATE utf8mb4_unicode_ci,
  PRIMARY KEY (`id`),
  UNIQUE KEY `domain_role` (`domain_id`,`role_name`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC COMMENT='角色表';
```

数据权限与角色关系表
```mariadb
CREATE TABLE `role_data_perm` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `role_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '角色id',
  `data_perm_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '数据权限id',
  PRIMARY KEY (`id`),
  KEY `idx_rdp_role_id` (`role_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='数据权限与角色关系表';

```

用户表
```mariadb
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '用户名称',
  `mobile` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '手机号',
  `sex` int(11) NOT NULL DEFAULT '0' COMMENT '性别',
  `realname` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '真实姓名',
  `password` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '密码',
  `salt` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '加密盐',
  `department_id` int(11) NOT NULL DEFAULT '0' COMMENT '部门ID',
  `faceicon` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '头像地址',
  `email` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '邮箱',
  `status` int(11) NOT NULL DEFAULT '1' COMMENT '状态 1=正常 2=禁用',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `last_login_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `title` tinytext COLLATE utf8mb4_unicode_ci COMMENT '职位，头衔',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC COMMENT='用户表';

```

第三方登陆关联表
```mariadb

CREATE TABLE `user_oauth` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增ID',
  `from` tinyint(3) NOT NULL DEFAULT '1' COMMENT '来源 1.dingtalk 2.wechat 3.facebook',
  `user_id` int(11) DEFAULT NULL COMMENT '用户ID',
  `name` varchar(20) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '用户姓名',
  `openid` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '第三方openid',
  `unionid` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '第三方unionid',
  `avatar` varchar(50) COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '头像',
  `extra` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '扩展信息',
  `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC COMMENT='用户第三方登陆关联表';

```

