## 使用球面几何计算距离

> 警告
>
> 对于球形查询，使用' 2dsphere '索引结果。
>
> 对球形查询使用“2d”索引可能会导致不正确的结果，例如对环绕极点的球形查询使用“2d”索引。

2d索引支持在欧几里得平面(平面)上计算距离的查询。索引还支持以下查询操作符和命令，计算距离使用球面几何:

> 注意
>
> 虽然“2d”索引支持使用球面距离的基本查询，但如果您的数据主要是经度和纬度，请考虑移动到“2dsphere”索引。

- [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere)
- [`$centerSphere`](https://docs.mongodb.com/manual/reference/operator/query/centerSphere/#op._S_centerSphere)
- [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near)
- [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear)带有选择的流水线级`spherical: true`

> 重要
>
> 上述操作使用弧度表示距离。其他球形查询操作符则不是这样，比如[' $geoWithin '](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)。
>
> 要使球形查询运算符正常运行，必须将距离转换为弧度，并将弧度转换为应用程序使用的距离单位。
>
> 转换:
>
> * *到弧度*的距离：用与距离测量相同的单位将距离除以球体（例如地球）的半径。
>
> * *弧度到距离*：将弧度乘以要转换距离的单位制中球体（例如地球）的半径。
>
>   地球的赤道半径大约为3,963.2英里或6,378.1公里。

以下查询将从**places**集合中返回半径为**“100”**英里的圆心**“[-74,40.74]”**所描述的圆内的文档:

```powershell
db.places.find( { loc: { $geoWithin: { $centerSphere: [ [ -74, 40.74 ] ,
                                                     100 / 3963.2 ] } } } )
```

> 注意
>
> 如果指定纬度和经度坐标，请先列出**经度**，再列出**纬度**:
>
> * 有效经度值在‘**-180’**和‘**180**’之间，包括两者。
> * 有效纬度值在' **-90** '和' **90** '之间，两者都包括。