# react-context-menu

[![NPM version](https://img.shields.io/npm/v/react-use-contextmenu.svg?style=flat)](https://npmjs.org/package/react-use-contextmenu)
[![NPM downloads](http://img.shields.io/npm/dm/react-use-contextmenu.svg?style=flat)](https://npmjs.org/package/react-use-contextmenu)

## Install

```bash
$ pnpm install react-use-contextmenu
```

## run | build

```bash
$ npm run dev
$ npm run build
```



## 为什么会有这个组件而不是用社区的

看了几款react生态下的相关插件，无法适配当前业务的复杂性。

例如:


- 当菜单超出屏幕时需要做UI适配，会导致顶部和底部列表的选项无法全部展示
- 当菜单处于局部滚动条内时，当用户滚动时，菜单也应该对应的滚动，否则无法继续操作 - 未处理
- 需要适配多级菜单和动态展开,展开式也需要对UI做判断，需要适配屏幕(目前只适配了两级，多级情况太复杂了，也不知道交互要怎么搞，几乎不会出现这种情况)
- 菜单禁用状态适配
- 动态生成菜单


其它社区优秀插件： [React ContextMenu](https://github.com/vkbansal/react-contextmenu/issues/339) 2020年已不维护


# ContextMenu 组件描述

用于需要对该项进行多个选项操作时。

## 基础

右键菜单分为多个部分，下面是一些相关概念。你也可以基于此实现其它相关菜单的功能


### MenuItem 数据结构

<API src="./index.tsx" hideTitle  exports='["ContextMenuItem"]'></API>

### DEMO

```jsx
import useContextMenu from 'react-use-contextmenu';
import React from 'react'


export default function MyContextMenu () {
  const { Trigger, ContextMenu } = useContextMenu()
  return <div>
    <Trigger data={{id: '1j24iej1h2r23'}}>我是触发器，单击我会把我的data传给右键菜单的处理函数</Trigger>
    <ContextMenu menus={[{ label: '操作1', value: '1' }]} onClick={(e, data, menu) => {
      alert(`data.id: ${data.id} menu.value: ${menu.value}`)
    }}/>
  </div>
}

```

### useContextMenu

逻辑解耦， 只负责处理处理右键和定位逻辑， 返回的Trigger和ContextMenu是包含菜单功能逻辑的菜单和触发器，用的时候只要加一些配置即可，可配置唤醒方式，单击，右键等事件，默认右键唤醒

<API src="./useContextMenu.tsx" hideTitle exports='["HooksProps"]'></API>


### Trigger 触发器

用来根据hooks初始化时配置的操作事件唤醒菜单和传递参数。

`注：为了无缝接入其它组件库，所有在Trigger上的参数都会被渲染到子元素的props上`[接入Antd Table](/components/context-menu#table列表右键菜单)

<API src="./useContextMenu.tsx" hideTitle exports='["TriggerProps"]'></API>

### ContextMenu 

展示菜单UI的组件和对应的菜单处理函数。

<API src="./useContextMenu.tsx" hideTitle exports='["ContextMenuProps"]'></API>

## 触发方式

可通过初始化hooks时配置触发方式来适配其它激活操作。

```tsx
import useContextMenu from 'react-use-contextmenu';
import React from 'react'


export default function MyContextMenu () {
  const { Trigger, ContextMenu } = useContextMenu({ event: 'click' })
  const { Trigger: Trigger2, ContextMenu: ContextMenu2 } = useContextMenu({ event: 'mousemove' })

  return <div>
    <Trigger data={{id: 'click'}}>单击触发</Trigger>
    <ContextMenu menus={[{ label: '操作1', value: '1' }]} onClick={(e, data, menu) => {
      alert(`data.id: ${data.id} menu.value: ${menu.value}`)
    }}/>
    <Trigger2 data={{id: 'mousemove'}} tag="span">鼠标移动触发</Trigger2>
    <ContextMenu2 menus={[{ label: '操作1', value: '1' }]} onClick={(e, data, menu) => {
      alert(`data.id: ${data.id} menu.value: ${menu.value}`)
    }}/>
  </div>
}
```

## 多级菜单 & 异步动态加载

```tsx
import useContextMenu from 'react-use-contextmenu';
import React from 'react'
import { ReloadOutlined } from '@ant-design/icons';


export default function MyContextMenu () {
  const { Trigger, ContextMenu } = useContextMenu()


  return <div>
    <Trigger data={{id: '123'}}>右键我</Trigger>
    <ContextMenu menus={[{ label: '操作1', value: '1' },
    {
  label: '动态异步菜单',
  value: 'key2',
  icon: <ReloadOutlined />,
  children: () => {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve([
          { label: '操作2-1',
            value: 'key2-1'
          },
          { label: '操作2-2',
            value: 'key2-2'
          },
          { label: '操作2-3',
            value: 'key2-3'
          },
          { label: '操作2-4',
            value: 'key2-4'
          }
        ])
      }, 1000)
    })
  }
},
    ]} onClick={(e, data, menu) => {
      alert(`data.id: ${data.id} menu.value: ${menu.value}`)
    }}/>
  </div>
}
```

## Table列表右键菜单

无缝接入ant design [Table](https://4x.ant.design/components/table-cn/#components-table-demo-edit-row)组件， 为每项加入菜单, 重写render row方法，把所有属性同步到`Trigger`即可



```tsx
import React, {  useState } from 'react';
import useContextMenu from 'react-use-contextmenu';
import { Button, Table  } from 'antd'
import { ReloadOutlined } from '@ant-design/icons';


const menus = [{
  label: '操作1',
  value: 'key1',
  children: [
    { label: '操作1-1',
      value: 'key1-1',
    },
    { label: '操作1-2',
      value: 'key1-2'
    }
  ],
  icon: <ReloadOutlined />,
},
{
  label: '操作3',
  value: 'key3',
},
{
  label: '操作被禁用',
  value: 'key4',
  disabled: true,
}]

export default () => {
  const { Trigger, ContextMenu } = useContextMenu()
  return <div >
    <ContextMenu menus={menus} onClick={(e, item, menu)=> {
      console.log(item, menu, '----')
    }}/>
    <Table 
      dataSource={[
      {
        key: '1',
        name: '胡彦斌',
        age: 32,
        address: '西湖区湖底公园1号',
      },
      {
        key: '2',
        name: '胡彦祖',
        age: 42,
        address: '西湖区湖底公园1号',
      },
    ]}
    columns={ [
      {
        title: '姓名',
        dataIndex: 'name',
        key: 'name',
      },
      {
        title: '年龄',
        dataIndex: 'age',
        key: 'age',
      },
      {
        title: '住址',
        dataIndex: 'address',
        key: 'address',
      },
    ]}
    components={{
      body: {
        row: (item) => {
          const { index,
          moveRow,
          className,
          style,
          ...restProps} = item;
          return <Trigger data={item} tag="tr" className={`${className}`}
            style={style}
            {...restProps}
            >
          </Trigger>
        }
      }
    }}
    />
  </div>
}
```


## 自定义封装

<Alert type="error">
  <span style="color: red">
  注：以下API适用于个人封装，与hooks用法无关，可不了解
  </span>
</Alert>
<API src="./index.tsx"></API>

