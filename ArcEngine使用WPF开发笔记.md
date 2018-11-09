# 1 ArcGIS控件
> ArcGIS控件：MapControl、TocControl、ToolbarControl、PageLayoutControl。  
Esri Interop程序集为ArcGIS控件提供了能够位于.NET窗体上的控件，这些控件前缀为“Ax”。
# 2 不使用工具栏Toolbar，调用工具方法
可以使用命令的形式调用工具
> ControlsAddDataCommandClass   增加数据工具  
ControlsMapIdentifyTool  识别工具

对于需要在地图上使用的工具，例如拖拽，需要设置CurrentTool  
> Tool和Class的区别：

调用ArcGIS自带的工具
``` cs
ControlsMapIdentifyTool mapIdentifyTool = new ControlsMapIdentifyToolClass();
mapIdentifyTool.OnCreate(SvApplication.AxControls.MapControl.Object);
SvApplication.AxControls.MapControl.CurrentTool = mapIdentifyTool as ESRI.ArcGIS.SystemUI.ITool;
```
调用识别工具
``` cs
ControlsMapIdentifyTool mapIdentifyTool = new ControlsMapIdentifyToolClass();
mapIdentifyTool.OnCreate(SvApplication.AxControls.MapControl.Object);
SvApplication.AxControls.MapControl.CurrentTool = mapIdentifyTool as ESRI.ArcGIS.SystemUI.ITool;
```
调用地图全图工具
``` cs
ControlsMapFullExtentCommand controlsMapFullExtentCommand = new ControlsMapFullExtentCommand();
controlsMapFullExtentCommand.OnCreate(SvApplication.DrawMapAxControls.MapControl.Object);
controlsMapFullExtentCommand.OnClick();
```
# 布局视图的使用开发
将数据视图中的地图复制到布局视图中
> 数据视图与布局视图的同步，首先要保证数据的一致性，其次就是数据显示范围的一致性。  
涉及IMapControl接口的OnMapReplaced事件和OnAfterScreenDraw事件

``` cs
        public static void CopyToLayoutMapControl()
        {
            IObjectCopy pObjectCopy = new ObjectCopyClass();
            object copyFormMap = SvApplication.DrawMapAxControls.MapControl.Map;
            object copiedMap = pObjectCopy.Copy(copyFormMap);//复制地图
            object copyToMap = SvApplication.DrawMapAxControls.PageLayoutControl.ActiveView.FocusMap;
            pObjectCopy.Overwrite(copiedMap, ref copyToMap);//复制地图
            SvApplication.DrawMapAxControls.PageLayoutControl.ActiveView.Refresh();
            SvApplication.DrawMapAxControls.TocControl.SetBuddyControl(SvApplication.DrawMapAxControls.PageLayoutControl);
        }
```
## PageControl
> PageLayout类提供了在布局视图中控制元素的属性和方法
## 将一个mxd的布局复制到另一个mxd中
``` cs
/// <summary>
/// 替换模板，将模板的layout复制到现有地图中
/// </summary>
/// <param name="temFileName">mxt模板位置</param>
/// <param name="pageControl">PageLayoutControl控件</param>
public static void CopyLayout(string temFileName, AxPageLayoutControl pageControl)
{
    var map = pageControl.ActiveView.FocusMap;
    var pageLayout = pageControl.PageLayout;
    //读取新模板
    IMapDocument pMapDocument = new MapDocumentClass();
    pMapDocument.Open(temFileName, "");
    var tempPageLayout = pMapDocument.PageLayout;
    var tempPage = tempPageLayout.Page;
    var curPage = pageLayout.Page;
    //替换单位
    curPage.Units = tempPage.Units;
    curPage.Orientation = tempPage.Orientation;
    //替换页面尺寸
    double width;
    double height;
    tempPage.QuerySize(out width, out height);
    curPage.PutCustomSize(width, height);
    //删除当前Layout中除了mapframe外的所有elements
    var pGraphicsCont = pageLayout as IGraphicsContainer;
    pGraphicsCont.Reset();
    var pElement = pGraphicsCont.Next();
    IElement pMapFrameElement = null;
    IMapFrame pMapFrame = null;
    while (pElement != null)
    {
        if(pElement is IMapFrame)
        {
            pMapFrameElement = pElement;
            pMapFrame = pElement as IMapFrame;
        }
        else
        {
            pGraphicsCont.DeleteElement(pElement);
            pGraphicsCont.Reset();
        }
        pElement = pGraphicsCont.Next();
    }

    //遍历模板的PageLayout中的所有元素，并且替换当前PageLayout中的所有元素
    var pTempGraphicsCont = tempPageLayout as IGraphicsContainer;
    pTempGraphicsCont.Reset();

    pElement = pTempGraphicsCont.Next();
    var pArray = new ESRI.ArcGIS.esriSystem.Array() as IArray;

    IMapSurroundFrame pTempMapSurroundFrame;
    IMapSurround pTempMapSurround;
    while (pElement != null)
    {
        if (pElement is IMapFrame)
        {
            pMapFrameElement.Geometry = pElement.Geometry;
        }
        else
        {
            if (pElement is IMapSurroundFrame)
            {
                pTempMapSurroundFrame = pElement as IMapSurroundFrame;
                pTempMapSurroundFrame.MapFrame = pMapFrame;
                pTempMapSurround = pTempMapSurroundFrame.MapSurround;
                map.AddMapSurround(pTempMapSurround);
            }
            pArray.Add(pElement);
        }
        pElement = pTempGraphicsCont.Next();
    }

    var pElementCount = pArray.Count;
    //将模板PageLayout中的其它元素（除了MapFrameElement和MapSurroundFrame外的元素）添加到当前PageLayout中去
    for (int i = 0; i < pElementCount; i++)
    {
        pGraphicsCont.AddElement(pArray.Element[pElementCount - 1 - i] as IElement,0);
    }
    pageControl.ActiveView.Refresh();
}

```


# 3 操作数据文档
## 3.1 操作地图文档Mxd
### 3.1.1 打开地图文档
调用ArcGIS自带工具
``` cs
ControlsOpenDocCommand controlsOpenDocCommand = new ControlsOpenDocCommand();
            controlsOpenDocCommand.OnCreate(SvApplication.DrawMapAxControls.MapControl.Object);
            controlsOpenDocCommand.OnClick();
```

### 3.1.2 保存地图文档
1、直接使用IMapDocument接口的Open方法来打开MXD文件，编辑过后进行保存。
> 这种情况比较简单，直接使用IMapDocument的save或者saveas的方法来进行保存就可以，可以在帮助中查到例子。

2、使用Engine中带的OpenDocument方法来打开MXD文件，然后编辑过之后要进行保存。
``` cs
IMxdContents pMxdC;    
pMxdC = axMapControl1.Map as IMxdContents ;    
IMapDocument pMapDocument = new MapDocumentClass();    
pMapDocument.Open (axMapControl1.DocumentFilename,"");    
pMapDocument.ReplaceContents (pMxdC);    
pMapDocument.SaveAs ("d:\aa2.mxd",true,true);   
```
3、使用自己写的添加数据的工具直接添加数据，也就是说一开始没有MXD文件，在编辑完之后需要把当前的地图保存为一个MXD文件。
``` cs
IMxdContents pMxdC;    
pMxdC = axMapControl1.Map as IMxdContents ;    
IMapDocument pMapDocument = new MapDocumentClass ();  
//这里可以直接调用  另存为  工具
pMapDocument.New ("d:\aa3.mxd");    
pMapDocument.ReplaceContents (pMxdC);    
pMapDocument.Save (true,true);
```
### 3.1.2 另存地图文档
调用ArcGIS自带工具
``` cs
            ControlsSaveAsDocCommand controlsSaveAsDocCommand = new ControlsSaveAsDocCommand();
            controlsSaveAsDocCommand.OnCreate(SvApplication.DrawMapAxControls.MapControl.Object);
            controlsSaveAsDocCommand.OnClick();
```

# 4 调用ArcMap的功能
使用Domodal方法打开ArcMap窗口  
添加图例
``` cs
        /// <summary>
        /// 添加图例到地图中
        /// </summary>
        /// <param name="pageLayout">IPageLayout接口</param>
        /// <param name="map">IMap接口</param>
        /// <param name="posX">以图例为单位的X坐标值，用于图例的开头,2</param>
        /// <param name="posY">以图例为单位的Y坐标值，用于图例的开头,2</param>
        /// <param name="legW">在X和Y方向上以Legend的页面为单位的长度,5</param>
        public static void AddLegend(IPageLayout pageLayout, IMap map, Double posX, Double posY, Double legW)
        {
            if (pageLayout == null || map == null)
            {
                return;
            }
            var graphicsContainer = pageLayout as IGraphicsContainer; // Dynamic Cast
            var mapFrame = graphicsContainer.FindFrame(map) as IMapFrame; // Dynamic Cast
            var uid = new UIDClass() as IUID;
            uid.Value = "esriCarto.Legend";
            var mapSurroundFrame = mapFrame.CreateSurroundFrame((UID)uid, null); // Explicit Cast
            //Get aspect ratio
            var querySize = mapSurroundFrame.MapSurround as IQuerySize; // Dynamic Cast
            double w = 0;
            double h = 0;
            querySize.QuerySize(ref w, ref h);
            double aspectRatio = w / h;
            //打开向导窗口
            var legendWizardClass = new LegendWizard();
            legendWizardClass.PageLayout = pageLayout;
            legendWizardClass.InitialLegendFrame = mapSurroundFrame;
            var flag = legendWizardClass.DoModal(0);
            var legendFrame = legendWizardClass.LegendFrame;
            //将图例设置赋值给mapSurroundFrame
            mapSurroundFrame = legendFrame;
            //插入图例
            var envelope = new EnvelopeClass();
            envelope.PutCoords(posX, posY, (posX * legW), (posY * legW / aspectRatio));
            var element = mapSurroundFrame as IElement; // Dynamic Cast
            element.Geometry = envelope;
            graphicsContainer.AddElement(element, 0);
            RefreshPageLayout(pageLayout as IActiveView);
        }
```
添加指北针
``` cs
/// <summary>
/// 添加指北针
/// </summary>
/// <param name="pageLayout">IPageLayout接口</param>
/// <param name="map">IMap接口</param>
public static void AddNorthArrow(IPageLayout pageLayout, IMap map)
{
    if(pageLayout == null || map == null)
    {
        return;
    }
    //打开指北针选择窗口
    var northArrowWin = new NorthArrowSelector();
    var objNA = northArrowWin.DoModal(0);
    //返回指北针对象
    var style = northArrowWin.GetStyle(0) as IMapSurround;
    var envelope = new EnvelopeClass() as IEnvelope;
    envelope.PutCoords(2, 2, 8, 8); //  Specify the location and size of the north arrow
    var uid = new UIDClass() as IUID;
    uid.Value = "esriCarto.MarkerNorthArrow";
    // Create a Surround. Set the geometry of the MapSurroundFrame to give it a location
    // Activate it and add it to the PageLayout's graphics container
    var graphicsContainer = pageLayout as IGraphicsContainer;
    var activeView = pageLayout as IActiveView;
    var frameElement = graphicsContainer.FindFrame(map);
    var mapFrame = frameElement as IMapFrame;
    var mapSurroundFrame = mapFrame.CreateSurroundFrame(uid as UID, null);
    //将返回的对象赋值给MapSurround
    mapSurroundFrame.MapSurround = style;
    //插入指北针
    var element = mapSurroundFrame as IElement;
    element.Geometry = envelope;
    element.Activate(activeView.ScreenDisplay);
    graphicsContainer.AddElement(element, 0);
    RefreshPageLayout(activeView);
}

```
这里只写了调用ArcMap中指北针和图例的添加方法，其他元素同理

添加比例尺文本
``` cs
IGraphicsContainer pGraphicsContainer = pActiveView.GraphicsContainer;
IMapFrame pMapFrame = pGraphicsContainer.FindFrame(pActiveView.FocusMap) as IMapFrame;
IMapSurroundFrame pMapSurroundFrame = new MapSurroundFrameClass();
pMapSurroundFrame.MapFrame = pMapFrame;

IStyleGalleryItem pStyleGalleryItem = SymbolUtilty.GetItemFromServerStyle("Scale Texts", "Absolute Scale");
pMapSurroundFrame.MapSurround = (IMapSurround)pStyleGalleryItem.Item;

IElement pElement = axPageControl.FindElementByName("ScaleText");
if (pElement != null)
{
    pGraphicsContainer.DeleteElement(pElement);  //删除已经存在的比例尺
}
IElementProperties pElePro = null;
pElement = (IElement)pMapSurroundFrame;
pElement.Geometry = (IGeometry)pEnv;
pElePro = pElement as IElementProperties;
pElePro.Name = "ScaleText";
pGraphicsContainer.AddElement(pElement, 0);
pActiveView.PartialRefresh(esriViewDrawPhase.esriViewGraphics, null, null);
```

调用ArcMap的地图导出功能
``` cs
public static bool OutputAllMap(IActiveView view)
{
    //获取地图范围
    var mapExtent = view.Extent;
    //获取地图像素
    var ExportResolution = view.ScreenDisplay.DisplayTransformation.Resolution;
    //int OutputResolution = int.Parse(ExportResolution);
    //exportrect参数
    tagRECT exportrect = new tagRECT();
    exportrect.left = view.ExportFrame.left;
    exportrect.top = view.ExportFrame.top;
    exportrect.right = view.ExportFrame.right;
    exportrect.bottom = view.ExportFrame.bottom;
    IEnvelope envelop = new EnvelopeClass();
    envelop.PutCoords(exportrect.left, exportrect.top, exportrect.right, exportrect.bottom);

    var exportWin = new ExportFileDialog() as IExportFileDialog;
    //pMapPixelBounds指定一个包络，表示输出图像的高度和宽度（以像素为单位）
    //pVisiblePixelBounds用于确定对话框的“Clip Output to Graphics Extent”复选框控件是否可见;从PageLayout对象导出时，通常会以页为单位指定一个表示页面高度和宽度的envelop;
    //从Map对象导出时，为pVisiblePixelBounds参数指定null
    //pMapExtent参数将用于确定是否启用对话框的“Write World File”和“Write GeoTIFF Tags”复选框控件。从PageLayout对象导出时，将null指定给pMapExtent参数。从Map对象导出时，指定一个以世界单位表示地图范围的包络。
    //dPixelBoundsResolution参数应该是一个double，表示计算pMapPixelBounds的分辨率
    var flag = exportWin.DoModal(envelop, null, mapExtent, ExportResolution);
    if (flag == true)
    {
        var export = exportWin.Export;
        var resolution = export.Resolution;
        var pixelBounds = export.PixelBounds;
        exportrect.left = (int)pixelBounds.XMin;
        exportrect.top = (int)pixelBounds.YMin;
        exportrect.right = (int)pixelBounds.XMax;
        exportrect.bottom = (int)pixelBounds.YMax;
        //开始输出文件
        view.Output(export.StartExporting(), (int)resolution, ref exportrect, null, null);
        export.FinishExporting();
        export.Cleanup();
        return true;
    }
    else
        return false;
}
```

# 5符号库
包括指北针、比例尺等符号库。
