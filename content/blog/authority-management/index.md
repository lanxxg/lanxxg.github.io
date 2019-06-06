---
title: 前端与服务端权限交互设计
date: "2019-06-06"
description: About authority management.
---

在后台管理系统中，经常涉及到权限管理。  

> 本文基于React举例

## 关于权限

一般来讲，权限控制可以分为两部分，API权限控制和UI权限控制。API权限顾名思义就是接口访问的权限，UI权限又可以细分为页面权限和功能权限。  

API权限控制是权限控制的底线，假设用户没有某个按钮执行操作权限，如果只是简单的把按钮隐藏起来是不可行的。因为理论上用户可以模拟接口请求服务端来执行操作。
真正的权限校验要在服务端完成，这样的话即使用户绕过前端的限制请求了后端的接口也无法完成操作。

---

## 服务端

### 数据库设计
* `user` 用户表
   * 系统用户。
* `role` 角色表
   * 权限集合。
* `permission` 权限表
   * 对资源进行粒度细分，为每个粒度提供一个唯一的标识`code`，我们根据资源唯一标识做权限控制。
* `user_role_map` 用户-角色关联表
   * 当用户与角色是一对一关系时，可以不需要此关联表，在用户表关联用户和角色的映射关系。
* `role_permission_map` 角色-权限关联表
   * 角色和权限的映射关系。

**用户与角色**：一个用户可以只对应一个角色，也可以对应多个角色。  
**角色与权限**： 一个角色可以绑定很多权限，一个权限也可以被不同的角色绑定，多对多的关系。

### 接口权限控制
  根据当前操作的权限标识（如果操作涉及多个权限标识，或运算）与用户权限进行校验，如果有权限则处理请求，否则提示没有权限。

---

## 前端

前面提到，UI权限控制分为页面（路由）权限和功能权限。要做UI权限控制，需要知道当前用户所拥有的权限，因此我们要服务端返回用户的权限集合，比如：
```
{
  permission: ['dashboard', 'example', 'example/one', 'user/create']
}
```
在`react-router`v4版本中，路由也是组件，所以无论是页面权限还是功能权限都可以通过[条件渲染](https://reactjs.org/docs/conditional-rendering.html)来控制。

### 页面权限控制实现有两种方式

1. 一种是全部菜单栏显示，当用户去访问没有权限的菜单页面时，显示403页面提示用户没有权限。  
2. 另一种是只显示用户有权限的菜单，如果用户在浏览器地址栏手动输入没有权限的路由强行访问，则显示404页面。  

   * 定义菜单数组时用`id`属性作为菜单项唯一权限标识，与后端`permission`表`code`字段对应：

    ```javascript
    // menu.js
    // id: 菜单项唯一权限标识
    const menuData = [
      {
        id: 'dashboard',
        name: 'dashboard',
        icon: 'icon1',
        path: 'dashboard',
      },
      {
        id: 'example',
        name: 'example',
        icon: 'icon2',
        path: 'example',
        children: [
          {
            id: 'example/one',
            name: 'one',
            path: 'one',
          },
        ],
      },
    ];
    ```

   * 在递归生成菜单的时候根据权限标识过滤掉没有权限的菜单

    ```javascript
    const renderMenuItems = () => {
      menuData.map(item => {
        const menuItem = getMenuItem(item);
        if (Array.isArray(permission)) {
          for (let i = 0; i < permission.length; i += 1) {
            const element = permission[i];
            if (element.indexOf(item.id) >= 0) {
              // 菜单项
              return menuItem;
            }
          }
        }
        return null;
      })
    }

    const getMenuItem = item => {
      if (item.children) {
        const childrenItems = renderMenuItems(item.children);
        return (
          // ...
        )
      }
      return (
        // ...
      )
    }
    ```

   * 前面只是对菜单进行权限校验，决定了菜单项是否显示。我们还需要在匹配路由时对页面的显示做权限校验。

    ```javascript
    // 根据路由匹配菜单项
    const menuItem = () => {
      // location.pathname
      return item;
    }
    // 菜单项权限标识与用户权限校验，有权限显示路由页面没有权限显示404页面
    // 实现思路和菜单一样
    ```

### 功能权限的实现

简单代码示例：
```jsx
if (permission.includes('user/create')) {
  return <Button>添加用户</Button>;
}
return null;
```

胡说八道~