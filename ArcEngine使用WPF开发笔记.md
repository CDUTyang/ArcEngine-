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

地图导出

# 5符号库
包括指北针、比例尺等符号库。
