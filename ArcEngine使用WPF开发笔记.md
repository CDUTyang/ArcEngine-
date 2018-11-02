# ArcGIS控件
> ArcGIS控件：MapControl、TocControl、ToolbarControl、PageLayoutControl。  
Esri Interop程序集为ArcGIS控件提供了能够位于.NET窗体上的控件，这些控件前缀为“Ax”。
# 不使用工具栏Toolbar，调用工具方法
可以使用命令的形式调用工具
> ControlsAddDataCommandClass   增加数据工具  
ControlsMapIdentifyTool  识别工具

对于需要在地图上使用的工具需要设置CurrentTool  
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
