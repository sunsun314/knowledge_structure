# ByteBuffer使用理解记录

## 背景
相对于其他的Buffer来说，java自带的nio.ByteBuffer的抽象类总是让我感到有点迷惑，在对于dble项目做网络层重构之际，我趁着这个机会对于这个部分的逻辑做一下梳理，以免每次做nio相关的内容都要重新来看上一遍。

## compact方法

## slice方法

## duplicate方法

## wrap(byte[] array,int offset, int length)

## wrap(byte[] array)


## Buffer.position
position指的是在本Buffer中当前读取游标所在的位置，在进行Buffer中的内容读取的过程中这游标的位置会有自动的更新

## Buffer.capacity
容量，指的是这个buffer最大能容纳多少的内容，初始设定后不能更改。

## Buffer.limit
position可以读取的上界，也就是游标的最大值