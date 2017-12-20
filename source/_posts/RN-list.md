---
title: RN长列表--ListView && SectionList
date: 2017-06-11 00:10:54
tags: React Native
comments: true
categories: "React Native"
---

突然发现自己的博客已经从原来的几天更新一次变成了如今的一个月更新一次[捂脸]，最近这一个月实在是太忙了，上个月月初的时候，接了一个RN的项目，做一个app，那时候老大问谁感兴趣，我直接说我。其实自己对RN一直很有兴趣，但是那时候由于在看Vue，所以并没有去搞，现在刚好有个项目，正好我可以去了解一下RN。说实话，这一答应不要紧，差点儿被RN气死，以前觉得RN应该比较成熟了，经过这段时间做项目，发现RN的坑还是很多的。

<!--more-->

在项目中我用到的比较多的是长列表，所以这篇文章主要是记一下对RN的`ListView`和`SectionList`的一些总结。我项目中使用的RN是0.44，关于为什么不用性能更好的`FlatList`而使用老的`ListView`，是因为自己本身第一次接触RN，所以担心用不好`FlatList`而且担心会有什么bug，毕竟这个是在0.43版本才添加的。

## ListView

对于`ListView`最重要的两个属性，一个是数据源（dataSource），再一个就是列表项渲染（renderRow）。`ListView`可以根据不同的数据结构对应的生成普通长列表和分组长列表。

### 普通长列表

普通长列表使用`cloneWithRows(dataBlob, rowIdentities)`创建datasource，在项目中`dataBlob`的数据结构如下：

```js
[
    {id:1},
    {name:'Mark'},
    ...
]
```

在使用`renderRow(rowData, sectionID, rowID, highlightRow)`渲染列表项的时候，`rowData`就是数组中每一个对象。

### 分组长列表

在项目中，遇到分组长列表，由于RN版本为0.44，所以有`ListView`和`SectionList`两种选择方案，最终我选择了`SectionList`，因为我发现`ListView`的`section`只能是一个简单的字符串，不能是一个对象（这里说实话我也不知道对不对，若不对，请指正并说明如何使用`ListView`实现`section`是一个对象）。

这里我要吐槽一下RN的文档，真的是有些地方写的太简单，比如在`ListView`这里写到使用`cloneWithRowsAndSections`的使用，只是简单的说跟`cloneWithRows`差不多，而接受的数据结构也就简单的说明有三种：

```js
{ sectionID_1: { rowID_1: <rowData1>, ... }, ... }
//or
{ sectionID_1: [ <rowData1>, <rowData2>, ... ], ... }
//or
[ [ <rowData1>, <rowData2>, ... ], ... ]
```

但是具体这里的是些什么，我觉得没有说清楚。所以我在项目中试了下用`ListView`的`cloneWithRowsAndSections`实现带有粘性标题的列表，发现前两种`sectionID`只能是一个简单的字符串或者数字，不能是一个对象，数据结构如下：

```js
const dataSource1 = [
    ['row1','row2'],
    ['row3','row4'],
    ['row5','row6'],
];

const dataSoure2 = {
    'id1':['row1','row2'],
    'id2':['row3','row4'],
    'id3':['row5','row6'],
}

const dataSoure3 = {
    'id1':{'row1','row2'},
    'id2':{'row3','row4'},
    'id3':{'row5','row6'},
}
cloneWithRowsAndSections(dataSource);
```

dataSoure1生成的sectionHeader是数组的下标，而第二个第三个分别是对应的`key`，在项目中我的sectionHeader是一个对象，类似于`{headerName:'',headerContent:''}`，所以无法使用。

## SectionList分组长列表

`SectionList`和`FlatList`一样是新增的高性能长列表，在文档中关于`SectionList`接受的数据结构描述如下：

```js
<SectionList
  renderItem={({item}) => <ListItem title={item.title} />}
  renderSectionHeader={({section}) => <H1 title={section.title} />}
  sections={[ // homogenous rendering between sections
    {data: [...], title: ...},
    {data: [...], title: ...},
    {data: [...], title: ...},
  ]}
/>

<SectionList
  sections={[ // heterogeneous rendering between sections
    {data: [...], title: ..., renderItem: ...},
    {data: [...], title: ..., renderItem: ...},
    {data: [...], title: ..., renderItem: ...},
  ]}
/>
```

说实话我第一次看到这个描述的时候很懵逼，这好像并没说明data里具体的数据结构是什么样子的，`renderSectionHeader`中的数据又该如何取，在网上看了一个简单的讲解才大概知道`sections`里的数据应该怎么写，代码如下：

```js
import React , {Component} from 'react';
import {
    AppRegistry,
    StyleSheet,
    Text,
    View,
    SectionList
} from 'react-native';

const dataSource = [
    {data:[{name:'nader'},{name:'chris'}],key:'A'},
    {data:[{name:'nick'},{name:'amanda'}],key:'B'}
];

export default class SectionListExample extends Component {
    renderItem = (item) => {
        return <Text style={styles.text}>{item.item.name}</Text>
    }

    renderHeader = (headerItem) => {
        return <Text style={styles.header}>{headerItem.section.key}</Text>
    }

    render(){
        return(
            <View style={styles.container}>
                <SectionList
                    renderItem={this.renderItem}
                    renderSectionHeader={this.renderHeader}
                    sections={dataSource}
                    keyExtractor={(item)=>item.name}
                />
            </View>
        )
    }
}
```

这里需要注意一点就是`renderItem`所用的数据`key`必须是`data`，而且`key`必须是唯一的，然后其余的数据用在`headerItem`中，这样就非常容易在`sectionHeader`中渲染出多个后台给定的数据。

#### `SectionList`和`FlatList`需要注意几点：

* 文档中提到<span style='color:red'>为了优化内存占用同时保持滑动的流畅，列表内容会在屏幕外异步绘制。这意味着如果用户滑动的速度超过渲染的速度，则会先看到空白的内容。这是为了优化不得不作出的妥协，而我们也在设法持续改进。</span>所以如果不想在滑动过快导致白屏出现，就只能使用`ListView`。

* 在我的项目中设置`stickySectionHeadersEnabled={true}`的时候，粘性标题在往下滑动一会儿再滑动回去的时候，标题会消失，这不知道是我代码有问题还是本身`SectionList`的粘性标题在安卓下就有问题，不过在官方文档中写到`Only enabled by default on iOS because that is the platform standard there.`所以可能是`SectionList`粘性标题在安卓下就有问题。

* `SectionList`和`FlatList`提供了一个叫`legacyImplementation`的属性，该属性如果设置为`true`则使用旧的`ListView`实现。在上述提到的两个问题中，如果将`SectionList`的`legacyImplementation`设置为`true`，则两个问题均解决。个人觉得这是官方提供的一个降级的办法，所以其实我们可以抛弃`ListView`转而使用`FlatList`和`SectionList`，如果有什么问题，让它们用旧的`ListView`实现。


## 最后

最后列几个长列表（`ListView`，`SectionList`，`FlatList`）的常用属性

`onEndReached`（function）：当列表到达底部时候触发的事件，关于这个事件需要注意一点，<span style='color:red'>当第一次渲染时，如果数据不足一屏（比如初始值是空的），这个事件也会被触发，需要自行过滤</span>

`onEndReachedThreshold`（number）：距离最后一个列表元素多少像素时候触发`onEndReached`事件

`initialListSize`（number）：初始化时候渲染多少条数据，如果不写择时逐条渲染

`showsVerticalScrollIndicator`（bool）：默认情况下，ListView有滚动条，当设置为false的时候不显示该滚动条（继承自ScrollView）

`pageSize`（number，仅`ListView`有）：每次事件循环（每帧）渲染的行数，常用于分页，数据请求回来后渲染多少条，不设置则逐条渲染。