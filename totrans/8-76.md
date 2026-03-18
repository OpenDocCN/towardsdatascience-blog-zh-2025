# 利用 Polars 和 Geopandas 在几秒钟内生成数百万个横断面

> 原文：[`towardsdatascience.com/harnessing-polars-and-geopandas-to-generate-millions-of-transects-in-seconds-d37b176a0b57/`](https://towardsdatascience.com/harnessing-polars-and-geopandas-to-generate-millions-of-transects-in-seconds-d37b176a0b57/)

![曼哈顿每隔 100 英尺有横断面。如果你能在这里进行横断面，你就能在任何地方进行横断面！](img/758c4905543e33dc68605cde73d11f05.png)

曼哈顿每隔 100 英尺有横断面。如果你能在这里进行横断面，你就能在任何地方进行横断面！

### **问题：**

我需要在纽约市每条街道上每 10 英尺生成一个横断面，而且我必须在纽约一分钟内完成。ArcGIS Pro 内置的“沿线生成横断面”工具将需要多天时间才能运行，所以我必须想出至少稍微快一点的方法。

为了解决这个问题，我转向了我最喜欢的两只熊，Geopandas 和 Polars，使用空间算法和一点线性代数，在约 60 秒内制作了 420 万个横断面。

**为什么选择横断面？** 在这种情况下，我会继续使用这些横断面将道路线分割成相等的间隔，以进行风暴损害分析。用另一条线分割一条线比用点分割一条线更容易，因为你可以保证它们会相交，你不必担心浮点数的不精确。但是，横断面对于许多应用都非常有用，例如采样河流深度剖面和从曲线线创建规则网格。

### **过程：**

首先，我使用 Geopandas 内置的 interpolate 函数在每个线路上以 10 英尺的间隔生成点。一旦我有了这些点，我就将它们切换到 Polars，在那里我能够利用 Polars 的内置并行性和直观的语法，快速将这些点转换为（大致）垂直的横断面。最后，我将横断面放入 GeoDataFrame 中，以进行进一步的空间分析。

在 Geopandas 和 Polars 之间来回切换可能听起来很烦人，但每个都有其自身的优势——GeoPandas 具有空间操作，而 Polars 非常快，并且我强烈偏好其语法。Polars 的.to_pandas()和.from_pandas()方法工作得非常好，以至于我几乎注意不到切换。虽然将熊猫和北极熊放入同一个动物园围栏可能会有灾难性的后果，但在这种情况下，它们能够相当愉快地一起玩耍。

**要求：** 此代码是在 mamba 环境中开发的，使用了以下包，并且尚未在其他版本上进行测试

```py
geopandas==1.0.1
pandas==2.2.3
polars==1.18.0
Shapely==2.0.6
```

本文使用的数据来自纽约市开放数据，可在[此处](https://data.cityofnewyork.us/City-Government/NYC-Street-Centerline-CSCL-/inkn-q76z/about_data)下载。在这个例子中，我使用了 2024 年 12 月 24 日发布的纽约市街道中心线。

### **步骤 1：使用 Geopandas 在每条线上每 10 英尺生成点**

为了在每条线上创建横断面，我们首先需要找到每个横断面的中心点。由于我们希望每 10 英尺有一个横断面，我们将从数据集中的每条线上的每 10 英尺抓取一个点。我们还会从每个横断面点略微偏离的位置抓取一个额外的点；这将允许我们找到使横断面垂直于原始线的方向。在我们到达所有这些之前，我们需要加载数据。

```py
import geopandas as gpd

ROADS_LOC = r"NYC Street CenterlineNYC_Roads.shp"
ROADS_GDF = (
   gpd.read_file(ROADS_LOC)
      .reset_index(names="unique_id")
      .to_crs("epsg:6539")
)
```

除了加载数据外，我们还根据其索引为每条道路分配一个唯一 _id，并确保 GeoDataFrame 在一个以英尺为基准单位的投影坐标系中——由于这些道路线位于纽约市，我们使用[纽约-长岛州平面](https://spatialreference.org/ref/epsg/6539/)作为坐标参考系统。唯一 _id 列将允许我们将生成的横断面链接回原始道路段。

数据加载完成后，我们需要以 10 英尺的间隔沿每条线进行采样。我们可以使用 GeoPandas 的 interpolate 函数来完成此操作，如下所示：

```py
import pandas as pd
from typing import Tuple, Union

def interpolate_points(
    gdf: gpd.GeoDataFrame, distance: Union[float, int], is_transect: bool
) -> Tuple[gpd.GeoDataFrame, gpd.GeoDataFrame]:
    gdf = gdf.loc[gdf.length > distance]

    new_points = gdf.interpolate(distance)

    new_gdf = gpd.GeoDataFrame(
        geometry=new_points,
        data={
            "unique_id": gdf["unique_id"],
            "distance": distance,
            "is_transect": is_transect,
        },
        crs=gdf.crs,
    )

    return gdf, new_gdf

def get_road_points(roads: gpd.GeoDataFrame, segment_length: int) 
-> gpd.GeoDataFrame:
    working_roads = roads.copy()
    road_points = []
    for segment_distance in range(
        segment_length, int(roads.length.max()) + 1, segment_length
    ):
        working_roads, new_gdf = interpolate_points(
            working_roads, segment_distance - 1, False
        )
        road_points.append(new_gdf)

        working_roads, new_gdf = interpolate_points(
            working_roads, segment_distance, True
        )
        road_points.append(new_gdf)

    return pd.concat(road_points, ignore_index=True)
```

get_road_points()函数遍历 roads GeoDataFrame 中的所有段距离，给定段长度*s*。每次迭代*i*，我们只选择长度大于*s * i*的道路。我们在每条线上的*s * i*距离处采样一个点。这些点将成为我们新横断面的中心点。由于我们希望新的横断面垂直于它们产生的线，我们在距离*(s * i) -1.*处也采样一个点。这给出了线的近似方向，我们将在构建横断面时使用它。由于此方法输出一个 GeoSeries，我们构建一个新的 GeoDataFrame，保留点沿线的距离，点是否将被用作横断面，以及我们之前创建的唯一 _id 列。最后，我们将所有这些 GeoDataframes 连接起来，得到如下所示的内容：

![沿纽约道路线生成的样点](img/44c18619be5e7887a5fc533e485fa5fa.png)

沿纽约道路线生成的样点

让我们用我们的道路数据集试一试，看看它是否工作：

```py
import matplotlib.pyplot as plt

points_10ft = get_road_points(ROADS_GDF, 10)
fig, ax = plt.subplots(figsize=(10, 12))

ROADS_GDF.loc[ROADS_GDF['unique_id'] == 100].plot(ax=ax, color='blue')
points_10ft.loc[(points_10ft['unique_id'] == 100) &amp; 
                (points_10ft['is_transect'])].plot(ax=ax, color='red')
```

![沿比弗街每 10 英尺采样一次样点](img/7257bd16e79b241504a653c4622086c6.png)

沿比弗街每 10 英尺采样一次样点

### **步骤 2：使用 Polars 和旋转矩阵将那些点转换为横断面**

现在我们已经得到了所需的点，我们可以使用 Polar 的 from_pandas()函数将我们的 GeoDataFrame 转换为 Polars。在我们这样做之前，我们想要保存几何列中的信息——我们的 x 和 y 位置——然后删除几何列。

```py
import polars as pl 

def roads_to_polars(road_points: gpd.GeoDataFrame) -> pl.DataFrame:
    road_points.loc[:, "x"] = road_points.geometry.x
    road_points.loc[:, "y"] = road_points.geometry.y
    pl_points = pl.from_pandas(road_points.drop("geometry", axis=1))
    return pl_points
```

最后，我们拥有了构建横断面的所有必要条件。本质上，我们只需要找到每个横断面的角度，构建所需长度的横断面，并旋转和转换横断面，使其在空间中正确定位。这听起来可能很复杂，但如果我们将其分解为步骤，我们并没有做任何过于复杂的事情。

1.  将每个横断面点与它前面的点对齐。这将使我们能够通过围绕前面的点旋转，使每个横断面垂直于原始线。实际上，我们可以将前面的点称为“支点”以使这一点更清晰。

1.  将横断面点/支点对平移，使支点位于 0, 0 的位置。通过这样做，我们可以轻松地旋转横断面点，使其位于 y = 0 的位置。

1.  将横断面点旋转，使其位于 y = 0 的位置。通过这样做，构建新点以形成横断面的数学变得简单，即加法。

1.  在旋转的横断面点上方和下方构建新的点，以便我们得到一条长度等于线长的新的线。

1.  将新点旋转回相对于支点的原始方向。

1.  将新点平移，使支点返回其原始位置。

![横断面构建算法的图形表示](img/50b3406103355b5cdee1404c83ea48e1.png)

横断面构建算法的图形表示

让我们看看执行此操作的代码。在我的旧笔记本电脑上，这段代码大约需要 3 秒钟生成 420 万个横断面。

```py
 def make_lines(
  pl_points: pl.DataFrame, line_length: Union[float, int]) 
  -> pl.DataFrame:
    return (
        pl_points.sort("unique_id", "distance")
        # Pair each transect point with its preceding pivot point
        # over() groups by unique id, and shift(1) moves 
        # each column down by 1 row
        .with_columns(
            origin_x=pl.col("x").shift(1).over(pl.col("unique_id")),
            origin_y=pl.col("y").shift(1).over(pl.col("unique_id")),
        )
        # Remove pivot point rows
        .filter(pl.col("is_transect"))
        # Translate each point so the pivot is now (0, 0)
        .with_columns(
            translated_x=pl.col("x") - pl.col("origin_x"),
            translated_y=pl.col("y") - pl.col("origin_y"),
        )
        # Calculate angle theta between translated transect point and x-axis
        .with_columns(theta=pl.arctan2(
            pl.col("translated_y"), pl.col("translated_x")
            )
        )
        # Calculate terms in rotation matrix
        .with_columns(
          theta_cos=pl.col("theta").cos(), 
          theta_sin=pl.col("theta").sin()
        )
        # Add y-coordinates above and below x axis so we have a line of desired
        # length. Since we know that x = 1 (the distance between the transect
        # and pivot points), we don't have to explicitly set that term.
        .with_columns(
            new_y1=-line_length / 2,
            new_y2=line_length / 2,
        )
        # Apply rotation matrix to points (1, new_y1) and (1, new_y2) 
        # and translate back to the original location
        .with_columns(
            new_x1=pl.col("theta_cos")
            - (pl.col("new_y1") * pl.col("theta_sin"))
            + pl.col("origin_x"),
            new_y1=pl.col("theta_sin")
            + (pl.col("new_y1") * pl.col("theta_cos"))
            + pl.col("origin_y"),
            new_x2=pl.col("theta_cos")
            - (pl.col("new_y2") * pl.col("theta_sin"))
            + pl.col("origin_x"),
            new_y2=pl.col("theta_sin")
            + (pl.col("new_y2") * pl.col("theta_cos"))
            + pl.col("origin_y"),
        )
        # Construct a Well-Known Text representation of the transects
        # for easy conversion to GeoPandas
        .with_columns(
            wkt=pl.concat_str(
                pl.lit("LINESTRING ("),
                pl.col("new_x1"),
                pl.col("new_y1"),
                pl.lit(","),
                pl.col("new_x2"),
                pl.col("new_y2"),
                pl.lit(")"),
                separator=" ",
            )
        )
    )
```

**我们实际上是如何完成旋转的？**

我们使用旋转矩阵绕原点旋转横断面点。这是一个 2×2 的矩阵，当应用于二维平面上的一个点时，将点绕原点（0, 0）旋转θ度。 

![逆时针旋转矩阵，由 https://en.wikipedia.org/wiki/Rotation_matrix 提供](img/fa4bd4b92e5ab98c7d1b7aa452ac65f8.png)

逆时针旋转矩阵，由[`en.wikipedia.org/wiki/Rotation_matrix`](https://en.wikipedia.org/wiki/Rotation_matrix)提供

要使用它，我们只需要计算θ，然后[乘法](https://en.wikipedia.org/wiki/Matrix_multiplication)点（x, y）和旋转矩阵。这给出了公式 _new_x = x*cos(θ)-y*sin(θ), new*y = x*sin(θ) + y*cos(θ)*。我们使用以下代码计算结果：

```py
.with_columns(
      new_x1=pl.col("theta_cos")
      - (pl.col("new_y1") * pl.col("theta_sin"))
      + pl.col("origin_x"),
      new_y1=pl.col("theta_sin")
      + (pl.col("new_y1") * pl.col("theta_cos"))
      + pl.col("origin_y"),
      new_x2=pl.col("theta_cos")
      - (pl.col("new_y2") * pl.col("theta_sin"))
      + pl.col("origin_x"),
      new_y2=pl.col("theta_sin")
      + (pl.col("new_y2") * pl.col("theta_cos"))
      + pl.col("origin_y"),
  )
```

我们能够简化计算，因为我们知道对于我们的点(x, y)来说，x 始终为 1。我们不需要计算*x*cos(θ)-y*sin(θ)*来找到旋转点的 x 值，我们只需做*cos(θ)-y*sin(θ)*，这样就可以节省一点计算量。当然，如果您的旋转点和横断面点不是 1 个单位距离，这就不适用了，但对我们来说这已经足够好了。一旦完成旋转，我们只需要将新旋转的点通过旋转点和(0,0)之间的距离进行平移，然后，我们就完成了。如果您对线性代数和仿射变换感兴趣，我强烈推荐 3Blue1Brown 系列的[线性代数本质](https://www.youtube.com/watch?v=fNk_zzaMoSs&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab)。

使用上述函数，我们可以将我们的 GeoDataFrame 转换为 Polars，并按如下方式计算新的横断面：

```py
points_10ft_pl = roads_to_polars(points_10ft)
transects_10ft = make_lines(points_10ft_pl, 10)
transects_10ft.select(['unique_id', 'wkt'])
```

![包含我们新生成横断面的 WKT Linestrings 的 Polars DataFrame](img/070013ab517ebcd8b457d77552dba552.png)

包含我们新生成横断面的 WKT Linestrings 的 Polars DataFrame

### **步骤 3：将横断面转换回 Geopandas**

终于！我们得到了所需横断面的几何坐标，但我们希望它们以 GeoDataFrame 的形式呈现，这样我们就可以回到 GIS 操作的舒适区。由于我们在 make_lines()函数的末尾将新横断面构建为[WKT Linestrings](https://libgeos.org/specifications/wkt/)，因此执行此操作的代码很简单：

```py
transects_gdf = gpd.GeoDataFrame(
    crs=ROADS_GDF.crs,
    data={"unique_id": transects_10ft.select("unique_id").to_series()},
    geometry=gpd.GeoSeries.from_wkt(
        transects_10ft.select("wkt").to_series().to_list()
    ),
)
```

就这样！我们现在有一个包含 420 万个横断面的 GeoDataFrame，每个横断面都通过唯一标识符列与其原始道路相关联。

让我们检查一下我们的新几何形状：

```py
ROAD_ID = 98237
fig, ax = plt.subplots(figsize=(10, 12))
ROADS_GDF.loc[ROADS_GDF['unique_id'] == ROAD_ID].plot(ax=ax, color='blue')
transects_gdf.loc[transects_gdf['unique_id'] == ROAD_ID].plot(ax=ax, color='green')
ax.set_aspect("equal")
```

![沿人行道每隔 10 英尺设置的横断面](img/ca06cca30a3ebadf483f38d92860ad99.png)

沿着人行道每隔 10 英尺设置的横断面

![沿 Rockaway Beach Blvd 的横断面（唯一标识符 = 99199）](img/901018d66903d32115efb6daa8bd2abe.png)

沿着 Rockaway Beach Blvd 的横断面（唯一标识符 = 99199）

沿着人行道每隔 10 英尺设置的横断面看起来不错！我们在 WKT 转换中丢失了一些精度，所以可能还有更好的方法来做这件事，但对我来说已经足够好了。

### **整合所有内容**

为了方便起见，我已经将生成横断面的代码组合在下面。这仅适用于投影到平面坐标系统的数据，如果您想要使用距离横断面点超过 1 个单位的旋转点，您需要修改代码。

```py
from typing import Tuple, Union

import geopandas as gpd
import pandas as pd
import polars as pl

def interpolate_points(
    gdf: gpd.GeoDataFrame, distance: Union[float, int], is_transect: bool
) -> Tuple[gpd.GeoDataFrame, gpd.GeoDataFrame]:
    gdf = gdf.loc[gdf.length > distance]

    new_points = gdf.interpolate(distance)

    new_gdf = gpd.GeoDataFrame(
        geometry=new_points,
        data={
            "unique_id": gdf["unique_id"],
            "distance": distance,
            "is_transect": is_transect,
        },
        crs=gdf.crs,
    )

    return gdf, new_gdf

def get_road_points(roads: gpd.GeoDataFrame, segment_length: int) -> gpd.GeoDataFrame:
    working_roads = roads.copy()
    road_points = []
    for segment_distance in range(
        segment_length, int(roads.length.max()) + 1, segment_length
    ):
        working_roads, new_gdf = interpolate_points(
            working_roads, segment_distance - 1, False
        )
        road_points.append(new_gdf)

        working_roads, new_gdf = interpolate_points(
            working_roads, segment_distance, True
        )
        road_points.append(new_gdf)

    return pd.concat(road_points, ignore_index=True)

def roads_to_polars(road_points: gpd.GeoDataFrame) -> pl.DataFrame:
    road_points.loc[:, "x"] = road_points.geometry.x
    road_points.loc[:, "y"] = road_points.geometry.y
    pl_points = pl.from_pandas(road_points.drop("geometry", axis=1))
    return pl_points

def make_lines(pl_points: pl.DataFrame, line_length: Union[float, int]) -> pl.DataFrame:
    return (
        pl_points.sort("unique_id", "distance")
        .with_columns(
            origin_x=pl.col("x").shift(1).over(pl.col("unique_id")),
            origin_y=pl.col("y").shift(1).over(pl.col("unique_id")),
        )
        .filter(pl.col("is_transect"))
        .with_columns(
            translated_x=pl.col("x") - pl.col("origin_x"),
            translated_y=pl.col("y") - pl.col("origin_y"),
        )
        .with_columns(theta=pl.arctan2(pl.col("translated_y"), pl.col("translated_x")))
        .with_columns(theta_cos=pl.col("theta").cos(), theta_sin=pl.col("theta").sin())
        .with_columns(
            new_y1=-line_length / 2,
            new_y2=line_length / 2,
        )
        .with_columns(
            new_x1=pl.col("theta_cos")
            - (pl.col("new_y1") * pl.col("theta_sin"))
            + pl.col("origin_x"),
            new_y1=pl.col("theta_sin")
            + (pl.col("new_y1") * pl.col("theta_cos"))
            + pl.col("origin_y"),
            new_x2=pl.col("theta_cos")
            - (pl.col("new_y2") * pl.col("theta_sin"))
            + pl.col("origin_x"),
            new_y2=pl.col("theta_sin")
            + (pl.col("new_y2") * pl.col("theta_cos"))
            + pl.col("origin_y"),
        )
        .with_columns(
            wkt=pl.concat_str(
                pl.lit("LINESTRING ("),
                pl.col("new_x1"),
                pl.col("new_y1"),
                pl.lit(","),
                pl.col("new_x2"),
                pl.col("new_y2"),
                pl.lit(")"),
                separator=" ",
            )
        )
    )
```

```py
ROADS_LOC = r"NYC Street CenterlineNYC_Roads.shp"
ROADS_GDF = gpd.read_file(ROADS_LOC).reset_index(names="unique_id").to_crs("epsg:6539")

points_10ft = get_road_points(roads=ROADS_GDF, segment_length=10)
points_10ft_pl = roads_to_polars(road_points=points_10ft)
transects_10ft = make_lines(pl_points=points_10ft_pl, line_length=10)
transects_gdf = gpd.GeoDataFrame(
    crs=ROADS_GDF.crs,
    data={"unique_id": transects_10ft.select("unique_id").to_series()},
    geometry=gpd.GeoSeries.from_wkt(transects_10ft.select("wkt").to_series().to_list()),
)
```

感谢阅读！可能还有更多方法可以使这段代码优化或更灵活，但它为我解决了一个大问题，将计算时间从几天缩短到几秒。只需一点理论知识，并知道何时使用什么工具，以前看似无法克服的问题变得惊人地简单。我仅用了大约 30 分钟和一点维基百科的线性代数复习资料就写出了这篇博客的代码，节省了我大量时间，更重要的是，保护了我免于向客户解释为什么项目无端延迟。如果这对您有帮助，请告诉我！

![Bartel-Pritchard Square/Prospect Park West with transects spaced every 10ft. I later used the unique_id field to ensure roads were only split using transects belonging to that road.](img/dfede077eea86fe352766c4ddce0ce04.png)

Bartel-Pritchard Square/Prospect Park West with transects spaced every 10ft. I later used the unique_id field to ensure roads were only split using transects belonging to that road.

除非另有说明，所有图片均由作者提供。
