## 您想为Unity3D Ui创建自定义网格，但是发现缺少文档？ 
在这篇博客文章中，我将描述
- 如何实现基本的自定义Unity UI网格
- 指出所有使您看到不可见或不存在的网格的陷阱
## TL; DR
- 要实现自己的UI网格，请从MaskableGraphic派生并实现OnPopulateMesh（）。

- 不要忘记在更改纹理或影响用户界面元素并应触发重新渲染的其他可设置编辑器的属性时调用SetVerticesDirty / SetMaterialDirty。

- 不要忘记设置UIVertex的颜色，否则由于alpha = 0，即完全透明，您将看不到任何东西。

- 您可以在[此处](https://gist.github.com/halgrimmur/bbe50a7bb2621f0a577524edb663669f)查看完整的，最少的代码示例。

## 在Unity UI中渲染迷你地图的过程
我的用例很简单：我想为当前的益智游戏项目[Puzzle Pelago](https://www.hallgrimgames.com/puzzle-pelago)创建关卡预览，并且想尝试基于自定义UI网格创建一个简单的切片系统。我一直在关注的要求是，它的行为应与所有其他UI元素统一，即应该放在RectTransform中，应该在蒙版ScrollView中工作，并且应该响应禁用状态着色，因为它将驻留在内部一个按钮。  

我最终得到的结果是这样的：

**等等，迷你地图现在看起来和行为都像一个适当的UI元素😄另外，我还清理了级别按钮。现在看起来好多了，对富有成效的周末而言！🎉
感谢@OskSta与使用MaskableGraphic😊提示[#madewithunity](https://twitter.com/hashtag/madewithunity?src=hash&ref_src=twsrc%5Etfw) [#indiedev](https://twitter.com/hashtag/indiedev?src=hash&ref_src=twsrc%5Etfw) [#uidesign](https://twitter.com/hashtag/uidesign?src=hash&ref_src=twsrc%5Etfw) [pic.twitter.com/Jit4XtmZpq](https://t.co/Jit4XtmZpq)

-Hallgrim Games(@HallgrimGames)2018年11月24日**

![](https://images.squarespace-cdn.com/content/v1/5b46535df8370aa62c65c0de/1543178646796-AH3ZVI4D04FZZE3SIBHP/ke17ZwdGBToddI8pDm48kKBVsAsyuyS5AHJH3rB6GnsUqsxRUqqbr1mOJYKfIPR7LoDQ9mXPOjoJoqy81S2I8N_N4V1vUb5AoIIIbLZhVYy7Mythp_T-mtop-vrsUOmeInPi9iDjx9w8K4ZfjXt2dpQtpWQCyUpXNY4P4y4CvWBwte8QBhLpqLltq8GhvOzKCjLISwBs8eEdxAxTptZAUg/Screenshot+2018-11-25+at+21.43.00.png?format=750w)

那里的路还不错，但有时还是令人沮丧，因为我在网上发现的全都是论坛帖子和unity自己的源代码。因此，在这里我想构建一个简化的示例，在该示例中，我们将使用一个脚本在UI元素内渲染带纹理的四边形网格。这将为构建您可能要构建的任何类型（平面，2d）UI几何图形带来所有障碍。

## Unity场景设置
好吧，让我们如下设置场景： 

1. 打开您要使用的Unity项目和场景。如果场景中还没有Canvas，请创建一个！对于本教程，我将所有属性保留为默认值。

2. 在Canvas中，创建一个ScrollView-我们将要检查新的UI组件是否在其中起作用！

3. 在ScrollView> Viewport> Content内，创建一个新的空游戏对象-我们称其为MyUiElement

4. 将CanvasRenderer组件添加到新的游戏对象，然后添加新的脚本：MyUiElement

5. 在您最喜欢的c＃编辑器中打开新脚本(我爱Rider，顺便说一句)，然后返回Unity的场景。

6. 为了使我们的生活更轻松，我们将希望将“场景视图”的渲染模式设置为“阴影线框”，以便我们可以详细查看UI网格几何。同样，切换到2D视图透视图，选择我们的“ MyUiElement”对象并按F键也很有用，因此单位缩放恰好正确。

![](https://images.squarespace-cdn.com/content/v1/5b46535df8370aa62c65c0de/1543178255700-9280231VCEPXUZLJY2C3/ke17ZwdGBToddI8pDm48kPPBhnSyEAcbnSCAibtJV8NZw-zPPgdn4jUwVcJE1ZvWQUxwkmyExglNqGp0IvTJZUJFbgE-7XRK3dMEBRBhUpyqza7ZOnF9fAwTIpNbXmTX3xkBpu8Gn-Mlmy51ya0sXU2O8BtXH6vy6lUXq1Q_MRI/Screenshot+2018-11-25+at+21.37.27.png?format=500w)

![](https://images.squarespace-cdn.com/content/v1/5b46535df8370aa62c65c0de/1543178284511-67INF9LL5CK1RLBFXP9G/ke17ZwdGBToddI8pDm48kICpf7RKTy3YIu7-F_Fxr9l7gQa3H78H3Y0txjaiv_0fDoOvxcdMmMKkDsyUqMSsMWxHk725yiiHCCLfrh8O1z5QPOohDIaIeljMHgDF5CVlOqpeNLcJ80NK65_fV7S1UVyDgVdZ4vzqujkwKkC8O9HRmUW58_HRMpKUmCtAXBgrOpYghpI-Ha_TwZsqqmJXng/Screenshot+2018-11-25+at+19.01.26.png?format=750w)

## 在C＃中实现自定义Unity UI网格脚本
现在，我们可以继续实施新的[c＃脚本](https://gist.github.com/halgrimmur/bbe50a7bb2621f0a577524edb663669f)！

首先，我们的新脚本至少需要从[Graphic](https://docs.unity3d.com/ScriptReference/UI.Graphic.html)派生。但是，例如，如果需要在ScrollViews内部进行遮罩，则最好从[MaskableGraphic](https://docs.unity3d.com/ScriptReference/UI.MaskableGraphic.html)派生。否则，[我们的图形也将在蒙版之外渲染](https://twitter.com/HallgrimGames/status/1066296348016865280)。哈哈。

另外，我们希望能够在编辑器中设置网格单元的大小，因此我们应该为此添加一个公共字段。
```c#
public class MyUiElement : MaskableGraphic
{
    public float GridCellSize = 40f;
}

```


接下来，我们希望能够为我们的UI元素使用纹理。查看unity自己的实现，例如Graphic（[源代码](https://github.com/Pinkuburu/Unity-Technologies-ui/blob/master/UnityEngine.UI/UI/Core/Graphic.cs)）基类的实现或默认[Image](https://docs.unity3d.com/ScriptReference/UI.Image.html)（[源代码](https://github.com/Pinkuburu/Unity-Technologies-ui/blob/master/UnityEngine.UI/UI/Core/Image.cs)）UI元素的实现，我们可以看到一个常见的模式是……

- …将“纹理/材质”插槽定义为属性，以便在检查器中更改纹理时，即使在编辑模式下，我们也可以触发Unity UI重新渲染。这是通过调用SetMaterialDirty()和SetVerticesDirty()完成的。

- …将mainTexture实现为默认的重写属性，这样，如果未提供纹理，则返回默认的白色纹理。

```c#
[SerializeField]
    Texture m_Texture;
    
    // make it such that unity will trigger our ui element to redraw whenever we change the texture in the inspector
    public Texture texture
    {
        get
        {
            return m_Texture;
        }
        set
        {
            if (m_Texture == value)
                return;
 
            m_Texture = value;
            SetVerticesDirty();
            SetMaterialDirty();
        }
    }
    public override Texture mainTexture
    {
        get
        {
            return m_Texture == null ? s_WhiteTexture : m_Texture;
        }
    }

```

接下来，我们必须重写OnPopulateMesh()进行渲染。它使用一个有用的小辅助对象来构建网格，将[VertexHelper](https://docs.unity3d.com/ScriptReference/UI.VertexHelper.html)作为其参数。它为您跟踪顶点索引，并允许您添加顶点，uv和tris，而无需执行大量数组算术和索引跟踪。建立新的网格物体之前，必须先进行Clear()处理。

我发现使用一点四边形辅助函数AddQuad()很有用（您也可能）：

```c#

// helper to easily create quads for our ui mesh. You could make any triangle-based geometry other than quads, too!
    void AddQuad(VertexHelper vh, Vector2 corner1, Vector2 corner2, Vector2 uvCorner1, Vector2 uvCorner2)
    {
        var i = vh.currentVertCount;
            
        UIVertex vert = new UIVertex();
        vert.color = this.color;  // Do not forget to set this, otherwise 

        vert.position = corner1;
        vert.uv0 = uvCorner1;
        vh.AddVert(vert);

        vert.position = new Vector2(corner2.x, corner1.y);
        vert.uv0 = new Vector2(uvCorner2.x, uvCorner1.y);
        vh.AddVert(vert);

        vert.position = corner2;
        vert.uv0 = uvCorner2;
        vh.AddVert(vert);

        vert.position = new Vector2(corner1.x, corner2.y);
        vert.uv0 = new Vector2(uvCorner1.x, uvCorner2.y);
        vh.AddVert(vert);
            
        vh.AddTriangle(i+0,i+2,i+1);
        vh.AddTriangle(i+3,i+2,i+0);
    }

    // actually update our mesh
    protected override void OnPopulateMesh(VertexHelper vh)
    {
        // Let's make sure we don't enter infinite loops
        if (GridCellSize <= 0)
        {
            GridCellSize = 1f;
            Debug.LogWarning("GridCellSize must be positive number. Setting to 1 to avoid problems.");            
        }
        
        // Clear vertex helper to reset vertices, indices etc.
        vh.Clear();
        
        // Bottom left corner of the full RectTransform of our UI element
        var bottomLeftCorner = new Vector2(0,0) - rectTransform.pivot;
        bottomLeftCorner.x *= rectTransform.rect.width;
        bottomLeftCorner.y *= rectTransform.rect.height;

        // Place as many square grid tiles as fit inside our UI RectTransform, at any given GridCellSize
        for (float x = 0; x < rectTransform.rect.width-GridCellSize; x += GridCellSize)
        {
            for (float y = 0; y < rectTransform.rect.height-GridCellSize; y += GridCellSize)
            {
                AddQuad(vh, 
                    bottomLeftCorner + x*Vector2.right + y*Vector2.up,
                    bottomLeftCorner + (x+GridCellSize)*Vector2.right + (y+GridCellSize)*Vector2.up,
                    Vector2.zero, Vector2.one); // UVs
            }
        }
        
        Debug.Log("Mesh was redrawn!");
    }
```

注意，在AddQuad()函数中，我们设置位置，uv和**颜色**！由于在UI材质中，默认情况下将纹理与颜色相乘。保留默认值，即(r = 0，g = 0，b = 0，a = 0)，这将产生100％透明的材料。因此，您所看到的只是一无所有，如果您想知道为什么，可能就是这样。在这里，我们使用组件的继承色槽。

因为我们希望每当调整RectTransform的大小时都更新网格，所以我们还应该重写OnRectTransformDimensionsChange()：

```c#
 protected override void OnRectTransformDimensionsChange()
    {
        base.OnRectTransformDimensionsChange();
        SetVerticesDirty();
        SetMaterialDirty();
    }
```
这应该做。现在，回到我们的Unity场景，我们应该在RectTransform内部看到一个白色正方形的网格。要更改此设置，我们可以在纹理插槽中选择unity的默认纹理之一。
![](https://images.squarespace-cdn.com/content/v1/5b46535df8370aa62c65c0de/1543177806593-P5UND1FOOUAZOQ8NKBDG/ke17ZwdGBToddI8pDm48kK8zpuZf8P0iexEfgDLtdPgUqsxRUqqbr1mOJYKfIPR7LoDQ9mXPOjoJoqy81S2I8N_N4V1vUb5AoIIIbLZhVYxCRW4BPu10St3TBAUQYVKcmXTIgnxoZCjFMOEt5ufuEPhUmegl9wOo8i4SdoMYYSFLJdBAtF7-9u3RoxO7RJ7O/Screenshot+2018-11-25+at+21.28.48.png?format=750w)

调整RectTransform的大小或网格单元大小的值，我们可以看到网格会自动更新。进入播放模式，我们还应该能够在滚动视图的内容周围拖动，并正确屏蔽网格。

![](https://images.squarespace-cdn.com/content/v1/5b46535df8370aa62c65c0de/1543177928617-AU7TNSPV6HTKQ5RIECSR/ke17ZwdGBToddI8pDm48kE2AO3Zmv2MFaQJDlkSG6Z9Zw-zPPgdn4jUwVcJE1ZvWQUxwkmyExglNqGp0IvTJZUJFbgE-7XRK3dMEBRBhUpyInvjIeR_YbL_TZzGbwcjtTYWggz_EWOrGTZJbmxiIYr2J2iSoA0-mOulKZbsqNNk/Screenshot+2018-11-25+at+21.31.34.png?format=500w)

![](https://images.squarespace-cdn.com/content/v1/5b46535df8370aa62c65c0de/1543177984432-3OMOLL8WF24NDS3AVH7O/ke17ZwdGBToddI8pDm48kDJQpvlbJTLZ-B12fIQrZZRZw-zPPgdn4jUwVcJE1ZvWQUxwkmyExglNqGp0IvTJZUJFbgE-7XRK3dMEBRBhUpwLpNiwG8Bg5UAMR_vmuKIQg6hhnrRNSN5o3_XKoM3YFx93Vab2sHU8IlGhhnhIhVU/Screenshot+2018-11-25+at+21.31.49.png?format=500w)

![](https://images.squarespace-cdn.com/content/v1/5b46535df8370aa62c65c0de/1543178129724-98GWEALW48HEYTD3QODA/ke17ZwdGBToddI8pDm48kC3xbAYiIn2Ki9gxsIqHWpBZw-zPPgdn4jUwVcJE1ZvWQUxwkmyExglNqGp0IvTJZamWLI2zvYWH8K3-s_4yszcp2ryTI0HqTOaaUohrI8PIZtrpsr1XjNz6GKKME5mdux_PONL_z6ZFIpaIW55UPqw/image-asset.png?format=750w)

## 结论
您可以在此处查看完整的代码示例。

当然，我们也不限于渲染四边形，因为我们在此处创建的基本几何图形由三角形组成。因此，任何2D网格都应该可以绘制，并且原则上也可以设置动画！
