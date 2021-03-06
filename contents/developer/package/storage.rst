---
title: 数据存储
description: 负责系统数据的存取，可以基于多种方式来存储。数据父子树状组织存放，有唯一的ID标识，支持版本，支持回收站，支持关系，可搜索
---

==================
数据存储
==================

.. Contents::
.. sectnum::

这里介绍系统的最底层，数据存储层。 数据分容器、条目2种，可设置属性，有唯一UID，相互建立关系关系，删除数据自动进入回收站，变更内容支持版本，数据可索引后进行搜索。

容器和条目
================
所有内容在系统中按照树状的层次结构存储，典型的站点结构如下::

    +- 站点根/
    |
    |----+- 栏目/
    |    |---+- 文件夹1/
    |    |   |- 文件1
    |    |   |- 快捷方式
    |    |   |- 子文件2/
    |    |
    |    |---+- 文件夹2/
    | 	 |   |- ….
    |
    |----+- 表单容器/
    |    |- 表单1
    |    |- 表单2

系统中的对象，可简单的抽象为2种对象：

- 容器类对象Container：如网站根、应用容器AppContainer、文件夹Folder、数据容器DataContainer
- 条目类对象Item：如文件File、快捷方式ShortCut、数据项DataItem

访问对象
-----------
容器提供类似dict的访问方法::

   root.keys()   # root是全局可访问的对象，返回： ['container1', 'container2']
   root.values()
   root.items()
   container1 = root['container1']
   
容器中的名字
-------------
任何对象在容器中有唯一的名字 ``name`` ::

  container1.name   # 'container1'
  container2.name   # 'container2'
  container1['item1'].name        # 'item1'

找到容器
----------
任何对象可得到其所在的容器 ``parent`` ::

  container1.parent  # root
  item1.parent       # container1
  sub_container1.parent # container1

删除
---------
删除某个包含的内容::

  del root['container2']  # 整个容器删除
  del container1['item1']

排序的容器
--------------
容器类对象都支持对包含内容进行排序(注意：如果容器包含的内容数量大，为提高性能，可对部分内容进行排序)::

  root.set_order(('container2', 'container1'))
  container.ordered_keys()  # ('container2', 'container1')

也可以直接访问排序的容器::

  root.set_container_order(('container2', 'container1'))
  container.ordered_container_keys()  # ('container2', 'container1')
  root.ordered_containers()

移动复制
----------
可以对内容进行移动、改名或者复制::

    item1.move_to(cotainer1, 'item_1')  # 改名
    item1.move_to(cotainer2)   # 移动
    sub_container.copy_to(container2, 'new_container') # 复制

标识和定位对象
======================================

路径定位
-----------------
可叠加内容的名字、以及包含该内容的所有容器的名字，形成对象路径，用于定位一个内容::

   path = root.object_path(obj) # 返回: '/container2/item_1'
   obj = root.object_by_path('/container2/item_1')  # 返回item1

数据库里面的对象，一旦发生移动或者改名，对象的路径就发生变化。这样用路径就不能来永久标识对象。

可以直接根据当前请求地址，得到对象url地址::

   obj.absolute_url(request)

有时候无法得到request(如系统自动任务)，这时候可以根据系统的配置来计算url::

   root.object_url(obj)

唯一标识定位
----------------
系统的所有对象，创建后均会注册一个永久的ID，无论以后对象是否移动或者改名，都不会改变::

   uid = root.object_uid(obj)
   obj = root.object_by_uid(uid)  # 通过uid找到对象

web访问地址为::

   obj.uid_url(request)

对象类型: object_types
=============================
约定属性 ``object_types`` 表示对象类型，让不同类型的对象有不同的行为。

容器和条目的object_types分别是 ``(Container, )`` 和 ``(Item, )`` , 系统还可以是如下对象：

应用容器 AppContainer
------------------------------
只有在应用容器里面，才能部署其他的应用，网站根就是一个应用容器。
应用容器里可以存放 表单容器、文件夹和子栏目. 

添加一个子文件夹::

  folder = app_container.add_folder(name, title="计划中心")

添加一个流程容器::

  collection = app_container.add_datacontainer(name='plan', 
                item_metadata="zopen.plan:plan",  # 表单的定义
                )

添加一个子应用容器::

  sub_container = app_container.add_appcontainer(name='plans', 
                                                metadata="zopen.plan:appcontainer",)

注意部署的子应用名字不能重复，可以通过下面的方法选择一个名字(自带加上)::

  app_contaner.choose_name('plans') # 如果重复，则返回 plans-1 / plans-2

应用容器的object_types是 ``('AppContainer', 'Container')``

应用容器可以管理子栏目导航，子栏目可以是一个子应用(应用容器、文件夹、数据容器)::

  app_container.navs.append(sub_container)  # 添加一个应用
  app_container.navs.insert(0, sub_container) # 插入到最前面
  app_container.navs.clear() # 清除所有 
  tabs = app_container.navs.get()  # 返回 应用或者脚本名的列表
  app_container.navs.remove(sub_container) # 去除一个列表
  app_container.navs.json() # json格式的导航设置
  app_container.settings.get('navs_dropdown_limit') # 栏目下拉展示的门限

文件夹 Folder
-----------------------
文件夹用来存放文件和文件的快捷方式，文件夹还能存放子文件夹::

  sub_folder = folder.add_folder(name)
  new_file = folder.add_file(name, data='', content_type='')
  shortcut = folder.add_shortcut(obj, version_id='')

文件夹的object_types是： ``('Folder', 'Container')``


文件 File
-------------
文件的object_types为 ``('File', 'Item')``

文件是最基础的内容形态，用于存放非结构化的数据，不能包含其他内容::

  my_file.set_data('this is long long text')
  my_file.content_type = 'text/plain'  

文件的下载是通过云查看服务器实现的。可以设定权限生成一个下载地址::

  download_url = my_file.download_url(request, mime='text/html', disposition='attachment')

使用上面的地址下载，会自动检查权限，并记录日志，最后跳转到云查看服务器下载。

也可以跳过权限，直接得到下载地址::

  my_file.transformed_url(mime='text/html', subfile=None, disposition='attachment', filename='')

快捷方式 ShortCut
---------------------
分为：

- 文件快捷方式, 其object_types为: ``('FileShortCut', 'Item')`` 
- 文件夹快捷方式，object_types: ``('FolderShortCut', 'Item')``

快捷方式可以指向其他的文件或者文件夹::

  shortcut.shortcut_orign

数据容器 DataContainer
-------------------------
数据容器的object_types为： ``('DataContainer', 'Container')`` , 用于存放表单数据项::

  item = collection.add_item({'title':'the title', 'description':'the desc'}, name='', **options)

数据项 DataItem
-------------------
数据项用来存放结构化的表单数据，是系统的基础内容，不能包含其他内容.

其object_types为： ``('DataItem', 'Item')``

站点对象 root
------------------
根站点root, 是一个特殊AppContainer, 这个对象在所有的脚本中可以直接使用。

可以查看自身的运行信息::

  root.sys_info

返回如下信息:

- version: 当前运行版本
- application: 应用名
- account: 比如zopen
- instance: 实例名
- operator: 本站点operator名字
- api_url: 本站点的api访问地址
- oc_api_url: oc的api地址

查看站点的运营选项参数::

    root.operation_options

可以是如下参数：

- sms: 短信数量
- apps_packages: 软件包数量
- flow_records: 数据库记录
- docsdue: 文档使用期限
- docs_quota: 文件存储限额(M)
- docs_users: 文档许可用户数
- docs_publish: 文档发布
- flow_customize: 流程定制
- apps_scripting: 允许开发软件包

工作台个人区 HomeContainer/Home
-------------------------------------
每个人都有一个工作台个人区，这里是个人自行部署的应用。

可以设置每个工作台的默认导航::

  homecontainer.default_navs.insert(0, 'zopen.sales:newest')
  homecontainer.default_navs.append('zopen.sales:newest')

Metadata元数据
=======================
所有内容对象都可以自定义字段，可以通过 ``metadata`` 进一步了解对象的详细字段，说明对象编辑、显示和存储信息。

应用容器天气查看，可通过 ``metadata`` 来进行应用设置天气区域等字段(软件包zopen.weather的appcontainer表单)::

  appcontainer.metadata = ('zopen.weather:appcontainer', )

数据容器可能是故障跟踪，有故障跟踪的一些设置项需要定义(软件包zopen.issuetracker的issue_container表单)::

  datacontainer.metadata = ('zopen.issuetracker:issue_container', )

具体的一个故障单数据项，则可能是(软件包zopen.isssuetracker的issue表单)::

  dataitemitem.metadata = ('zopen.issuetracker:issue', )

如果这里有多个，表示继承。metadata 的具体定义和使用，参照 《表单处理》 一节

对象属性
==============================================

基础属性
--------------------------------------
系统的所有对象，都包括一组标准的属性，有系统自动维护，或者有特殊的含义。属性也称作元数据，metadata.

metadata保存在 ``item.md`` 属性中::

   title = item.md['title']
   title = item.md.get('title', 'no title')

   item.md['title'] = 'new title'
   item.md.set('title', 'new title')
   item.md.update(title='new title')

对象一旦加入到仓库，可以查看其创建人、修改人，创建时间、修改时间::

   item.md['creators']
   item.md['contributors']
   item.md['created']
   item.md['modified']

其他的基础属性，还包括::

  obj.md['identifier'] 这个也就是文件的编号
  obj.md['expires'] 对象的失效时间
  obj.md['effective'] 对象的生效时间

自定义属性
---------------
可自由设置属性，对于需要在日历上显示的对象，通常有如下属性::

  obj.md['responsibles'] = ('users.panjy', 'users.lei') # 负责人
  obj.md['start'] = datetime.now() # 开始时间 
  obj.md['end'] = datetime.now(), 结束时间

对于联系人类型的对象，通常可以有如下表单属性::

  obj.md['mail'] = 'panjy@foobar.com' #邮件
  obj.md['mobile'] = '232121' # 手机

经费相关的属性::

  obj.md['amount'] = 211

地理相关的属性::

  obj.md['longitude'] = 123123.12312 #经度
  obj.md['latitude'] = 12312.12312 # 纬度

属性集
---------------
为了避免命名冲突，更好的分类组织属性，系统使用属性集(mdset: metadata set)，来扩展一组属性.

创建一个属性集::

  obj.mdset.new('archive')

设置一个新的属性集内容::

  obj.mdset.set('archive', {'number':'DE33212', 'copy':33})
  
活动属性集的内的属性值的存取::

  number = obj.mdset['archive']['number']
  obj.mdset['archive']['number'] = 'DD222'

也可以批量更改属性值::

  obj.mdset['archive'].update(copy=34, number='ES33')

删除属性集::

  obj.mdset.remove('archive')

查看对象所有属性集::

  obj.mdset.keys()  # 返回： [archive, ]

对象设置信息
----------------
通常对于容器会有一系列的设置信息，如显示方式、添加子项的设置、关联流程等等.

由于使用频繁，提供专门的操作接口::

   container.settings[setting_name]
   container.settings.get(setting_name, default_value)

   container.settings[setting_name] = 'blabla'
   container.settings.set(setting_name, 'blabla')

如果需要继承上级容器的设置，可以::

   container.settings.get(setting_name, inherit=True, default=default_value)

具体包括：

1) 和表单相关的设置::

    datacontainer.settings['item_metadata'] = ('zopen.sales:query',)   # 包含条目的表单定义

2) 流程相关的::

    datacontainer.settings['item_workflow'] = ('zopen.sales:query',): 容器的工作流定义(list)

3) 默认视图::

    container.settings['default_view'] = 'index' # 默认视图是什么

4) 和属性集相关的设置::

    container.settings['item_mdsets'] = ('archive_archive', 'zopen.contract:contract') : 表单属性集(list)

5) 和阶段相关的设置::

    container.settings['item_stage'] = ('zopen.sales:query',)

6) 容器表格显示列::

    container.settings['grid_columns'] = ('title', 'size', 'created', 'zopen.sales:query',)

7) 相关的流程，包括容器相关流程和条目相关的流程::

    container.settings['item_related_datacontainers'] =
                (root.object_uid(datacontainer1), root.object_uid(datacontainer2))
    container.settings['container_related_datacontainers'] = (root.object_uid(datacontainer3),)

容器视图
=================
所有容器都可以进行不同的视图切换，比如:

- 文件夹有列表、缩略图等视图
- 数据容器有 日历、列表等视图

可以查看所有的可选视图::

  >>> folder.views.items()
  [('tabular', '内容列表'), ('listing', '摘要清单'),
   ('thumbnail', '缩略图'), ('taggroup', '分类清单'),
   ('update', '最近更新'), ('category','纵览列表') ]

  >>> appcontainer.views.items()
  [('listing', '应用列表'), ('workitems', '流程视图')]

  >>> datacontainer.views.items()
  [('tabular', '内容列表'), ('calendar', '日历'), ('update', '最近更新')]

设置从其的默认视图::

  >>> container.views.set_default('tabular')

也可以使用软件包中的视图::

  >>> container.views.set_default('zopen.project:overview')

得到默认视图(如果没有设置默认视图，则使用第一个可选视图)::

  >>> container.views.get_default()

得到某个视图的url地址::

  >>> container.views.get_url(view, request)

关系
================

每一个对象都可以和其他的对象建立各种关系.  常用关系类型包括：

- children:比如任务的分解，计划的分解
- attachment：这个主要用于文件的附件
- related :一般关联，比如工作日志和任务之间的关联，文件关联等
- comment_attachment：评注中的附件，和被评注对象之间的关联
- favorit:内容与收藏之间的关联
- "shortcut" 快捷方式

可以查出所有的关系类型::

  doc1.relations.keys()  

将doc2设置为doc1的附件（doc1指向doc2的附件关系） ::
  
  doc1.relations.add('attachment', doc2, metadata={}) 

删除上面设置的那条关系::

  doc1.relations.remove('attachment', doc2) 

设置关系的元数据（关系不存在不会建立该关系）::

  doc1.relations.set_metadata('attachment', doc2, {'number':01, 'size':23}) 

得到关系的元数据（关系不存在返回None）::

  doc1.relations.get_metadata('attachment', doc2) 

查看所有的附件::

  doc1.relations.list_targets('attachment')

清除某种或所有的关系::

  doc1.relations.clean(type='attachment')

附件查看主文件::

  doc2.relations.list_sources('attachment')

版本管理
==================

版本信息查看
----------------------
文件File、数据项Item支持版本管理，可以保存多个版本，每个版本有唯一从1开始自增长的ID来标识::

   >>> context.revisions.keys(include_temp=True) 
   [1, 2, 4, 5]

也可以得到全部历史版本::

   >>> context.revisions.values(include_temp=False) 

可得到某个版本信息::

   >>> context.revisions.history_info(2)
   {'revision_id' : 2, # 版本ID
    'major_version' : '1',   # 版本号
    'minor_version' : '0',  # 版次号
    'user' : 'users.panjy',  # 版本修改人
    'timestamp' : 12312312.123,  # 版本修改时间
    'comment' : 'some comments',   # 版本说明
   }

得到历史版本对象::

   >>> obj = context.revisions.get(revision_id=2)
   >>> obj.revisions.version_info() # 该对象的版本信息

head()得到最新的工作版本对象::

   >>> obj.revisions.head() is context

也可以找到最新的定版版本::

   >>> obj.revisions.head(tagged=True) is context

版本更新
------------------
任何对象都可以保存历史版本，一旦保存当前对象的版本号发生变化::

   rev = context.revisions.save()

文档每次变更，默认保存为临时版本，临时版本定期会自动清理，不会永久存储。

可以对文档定版打上版本号，一旦定版，版本就是正式版本，可永久存储::

  context.revisions.tag(revision_id=None, major_version=None, minor_version=None, as_principal=None, comment=''):

- 如果不传revision_id，表示对当前的工作版本进行定版
- 如果不传 major_version，继续沿用上一个version_number
- 如果不传 minor_version，自动增长上一个revision_number

删除一个版本::

   context.revisions.remove(revision_id)

分支、合并
----------------
如果对原始文件没有直接修改权限，则需要通过分支、合并的方法来做。

fork一个文档进行修改, 实际上就是拷贝文档到新的位置::

  >>> obj = context.revisions.fork(container, new_name='')

分支采用关系记录，可以查看所有分支版本::

  >>> context.relations.list_sources('fork') 
  >>> obj.relations.list_target('fork')

可以查看当时分支的版本号::

  >>> obj.relations.get_metadata('fork', context)
  {'revision':2}

编辑完成，可以合并版本::

  >>> context.revisions.merge(obj)

如果版本冲突，会有异常抛出::

  >>> context.revisions.merge(obj)
  VersionConflicted: ...

则需要开始进行手工合并::

  >>> obj.revisions.start_resolve()

此时可以检查冲突信息(resolve是需要合并的版本)::

  >>> obj.relations.get_metadata('fork', context)
  {'revision':2, 'resolve':3}

手工合并完成，并作出标识，解决冲突::

  >>> obj.revisions.resolve()
  >>> obj.relations.get_metadata('fork', context)
  {'revision':3, 'resolve':3}
  >>> context.revisions.merge(obj)

如果冲突解决期间，context再次发生变化，仍然可能导致merge失败, 需要再次合并::

  >>> context.revisions.merge(obj)
  VersionConflicted: ...
  >>> obj.revisions.start_resolve()
  (手工合并obj和context的内容差异)
  >>> obj.revisions.resolve()
  >>> context.revisions.merge(obj)

注意：即便obj和context之间没有fork关系，也可以直接保存为新版本。
merge之后obj不会被删除，如果不需要，可以再次手工删除。

权限控制
================

系统中可以直接修改权限来进行权限管理，也可以通过修改角色来进行权限管理。

角色
--------
系统支持如下角色，角色ID为字符串类型, 可以枚举系统对象所有的角色::

  obj.allowed_roles

不同对象使用的角色不同，系统全部角色包括：

- 'Manager' 管理员
- 'Editor' 编辑人
- 'Owner' 拥有者
- 'Collaborator' 添加人
- 'Creator': 文件夹创建人
- 'ContainerCreator': 子栏目/容器创建人
- 'Responsible' 负责人
- Delegator 委托人
- 'Subscriber' 订阅人
- 'Accessor' 访问者
- 'Reader' : 5级查看人
- 'Reader3'
- 'Reader2'
- 'Reader1'
- 'PrivateReader' 超级查看人
- 'PrivateReader3' 仅仅文件授权的时候用，不随保密变化
- 'PrivateReader2' 仅仅文件授权的时候用，不随保密变化
- 'PrivateReader1' 仅仅文件授权的时候用，不随保密变化

授权
--------------
在obj对象上，授予用户某个角色::

  obj.acl.grant_role(role_id, pid)

同上，禁止角色::

  obj.acl.deny_role(role_id, pid)

同上，取消角色::

  obj.acl.unset_role(role_id, pid)

检查权限
-------------
检查当前用户对某对象是否有某种权限，可使用 ``permit`` 方法::

  obj.acl.check_permission(permission_id)

如果有该权限即返回True，反之返回False

系统中常用权限，权限ID为字符串类型，下文中权限ID将用permisson_id来代替。

- 'Public'：公开，任何人都可以访问
- 'Manage'：管理，文件夹管理员，文件负责人，可以管理
- 'Review': 审核，文件夹管理员、审核人，才可以审核
- 'View'：查看的权限
- 'Access'：文件夹、容器/栏目访问的权限
- 'Edit'：编辑，文件负责人、文件的编辑人，都可以编辑
- 'Operate': 调整对象的一些设置信息，比如订阅人、属性设置等
- 'Add'：添加文件、流程单
- 'AddFolder': 添加文件夹
- 'AddContainer': 添加容器(子栏目)
- 'Logined': 是否登录
- 'Delegate': 委托

'Access'和'View'的区别，需要进入文件夹(Access)，但是不希望查看文件夹包含的文档(View)。

读取权限设置
---------------
根据角色来获取obj对象上拥有该角色的用户ID::

  obj.acl.role_principals(role_id)

得到某个用户在obj上的所有角色::

  obj.acl.principal_roles(user_id)

得到上层以及全局的授权信息::

  obj.acl.inherited_role_principals(role_id)

得到某个用户在上层继承的角色::

  obj.acl.inherited_principal_roles(user_id)

对象的状态
===========================
每一个对象存在一组状态，存放在对象的 ``stati`` 属性中::

   'visible.default' in context.stati

文档的状态
----------------
modify: 发布

- modify.default	草稿
- modify.pending	待审
- modify.archived	发布/存档 (只读)
- modify.history_default 普通历史版本
- modify.history_archived 发布的历史版本

visible: 保密

- visible.default	普通
- visible.private	保密

attach: 附件状态

- master: 带附件的主文件
- attachment: 附件
- none：普通文件

数据项的状态
-----------------
- flowsheet.active, '活动', '流程单正在处理中'
- flowsheet.pending, '暂停', '暂停处理该流程单'
- flowsheet.abandoned, '废弃', '流程单已被废弃，不可做任何其他处理'
- flowsheet.finished, '完结', '流程单已经处理完成'


数据容器的状态
-----------------------
- datamanager.started', '活动', '流程启动, 正式使用'),
- datamanager.finished', '关闭', '流程已经冻结, 禁止添加新流程'),
- datamanager.planning', '规划中', '流程规划中, 新建流程单会自动暂停'),
- datamanager.template', '模板', '将流程作为模板, 新建流程单会自动暂停')),

状态变化
----------------
使用 ``state`` ，来控制对象状态的变化::

    # 不进行权限检查，直接发布某个文档
    context.state.set('modify.archived', do_check=False)
    # 设置文件夹为受控，需要检查权限
    context.state.set('folder.control', do_check=True)

也可以得到某个状态::

    context.state.get('visible') # 得到可见状态	

标签组
============

标签组实现了多维度、多层次、可管理的分类管理. 

设置标签组
-------------
标签组在容器(文件夹、数据容器、应用容器)上设置，可得到标签组设置::

  container.tag_groups.list_items() # TODO

输出为::

  [{'group': '按产品',
    'required':true,
    'single':true,
    'tags': [{'name':'wps'},
             {'name':'游戏'},
             {'name':'天下'},
             {'name':'传奇'},
             {'name':'毒霸'}
   ]},

   {'group': '按部门'
    'required':true,
    'single':true,
    'tags': [{'name':'研发', 
              'children':[{'name':'产品'}, 
                          {'name':'测试'},
                          {'name':'软件'},
                          {'name':'硬件',
                           'children':[{'name':'电子'}, 
                                       {'name':'机械'}]
                          },
                         ]
             },
             {'name':'市场'},
            ]
   }]

也可以导出为文本形式的标签组，用于编辑::

  container.tag_groups.export_text()

或者导入::

  container.tag_groups.import_text()

标签组存在必选和单选控制，可以校验::

  container.tag_groups.check(tags) # 返回: {'required':[], 'single':[]}

标签组设置可以继承上层设置, 可以通过这个变量来控制::

  container.tag_groups.inherit = True

标签维护
-------------
如果要添加一个标签::

  context.add_tag('完成') # TODO

如果这个标签所在的标签组是单选的，会自动去除其他的标签。

注意，标签存放在名字叫做 ``subjects`` 的属性中，可以直接维护::

  context.md['subjects']
  context.md['subjects'] = ['完成', '部门']

回收站
============

系统所有内容，删除之后，都将进入回收站。

一旦进入回收站，系统会定期对回收站的内容进行清理。

可以通过操作历史，查找到最近删除的内容，你可以恢复删除项::

  root.recycle_bin.restore(uid, new_parent)

也可以彻底删除::

  root.recycle_bin.purge(uid)

或者清空整个回收站::

  root.recycle_bin.clear()

个人信息profile
=========================
每个人会有一些个性化的设置, 比如个人偏好设置等。

可以设置profile::

   root.profiles.set(pid, name, value)

获取::

   root.profiles.get(pid, name, default)

得到某个人的HOME容器::

   home_container = root.profiles.get_home(pid)
