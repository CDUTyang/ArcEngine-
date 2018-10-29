# 批量为影像定义坐标系
### 1 使用GP工具（Define Projection）
``` cs
        /// <summary>
        /// 定义投影
        /// </summary>
        /// <param name="fileName">被定义的数据</param>
        /// <param name="prj">投影文件（prj），或坐标系字符串</param>
        public static void DefineFun(string fileName,string prj)
        {
            Geoprocessor gp = new Geoprocessor();
            //gp.OverwriteOutput = true;
            DefineProjection defPrj = new DefineProjection();
            defPrj.in_dataset = fileName;
            //defPrj.out_dataset = this.fileName;
            defPrj.coor_system = prj;
            gp.Execute(defPrj, null);
        }
```
