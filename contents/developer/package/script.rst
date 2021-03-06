---
title: 脚本代码
description: 编写脚本代码
---

=================
脚本代码
=================

目前支持python，之后会支持javascript

在软件包中创建一个脚本
==========================
TODO

通过浏览器访问脚本
========================
使用 ``@`` 定位一个脚本，典型的::

   http://<server.com>/<path/to/context>/@<package_name>:<script_name>
   http://localhost/files/@zopen.archive:file_stats

通过浏览器调用的脚本，自动绑定如下变量:

- root: 站点根对象
- context: 访问路径的上下文对象
- request: 请求对象
- view: 最终的渲染视图。可操作这个对象，动态改变界面. view对象的api，参看《UI交互组件》一章

脚本可以返回一个html文本，也可以返回一个页面布局信息::

    {'left': '',  # 左侧区域，通常是导航栏
     'right': '',  # 右侧区域，通常是附加辅助信息
     'main': '',  # 主正文区域
     'title': '',  # 页面标题
     'description': '' # 页面描述信息}

在脚本中调用脚本
====================
可以这样调用脚本::

   root.call_script('zopen.api:calc', aa='blabla', bb='blabla')

在脚本中调用脚本，可直接使用传入的参数，另外还可以访问 ``root`` 变量.

如果希望异步执行，可以::

   root.call_script_async('zopen.api:calc',  aa='blabla', bb='blabla')

注意，异步执行传入的参数，必须是简单参数，不能是复杂的对象。

流程查看视图
================
只需使用特殊的python脚本命名前缀，就可实现流程单的多种查看方式。

对于表单的名字 foobar，命名方式为::

 view_foobar_xxx

其中xxx为真正的脚本名称。

如果需要改变默认的视图，只需要::

 flow_container.views.set_default('xxx_account.xxx_package:view_foobar_xxx')

使用代码创建python脚本
==============================
python脚本可以直接通过浏览器调用

脚本采用python语言书写，存放在scripts中. 其中:

- setup: 用于部署
- upgrade(last_version='') : 用于升级

在软件包里面注册一个代码::

  root.packages.register_script('zopen.sales', 
        {'name':'setup',
         'permission':'Access',
         'template':'standard',
         'args':'',
         'script':'print "hello, world"',
        })

其中template是输出套用的模板：

- standard: 系统标准模版，支持应用权限、编辑和设置
- blank: 不使用现有模版页首和页脚，但保持现有界面风格, 保留css/js的模板
- json: json api
- kss: 页面调用处理脚本(ajax)
- none: 不使用任何模版

也可以得到一个代码::

  root.packages.get_script(name)

查看软件包的所有代码::

  root.packages.list_scripts(package_name)

导入导出
===============
为了方便书写，可导出一个标准的python函数::

  root.packages.export_script('zopen.sales:setup')

导出结果如下::

    @view_config(permission='zopen.Access', use_template='standard', icon=u'')
    def setup(redirect = True):
        """安装脚本

        初始化规则"""

        app = context.add_datacontainer('zopen.remind:remind', title='提醒')
        app.index()
        #创建规则

