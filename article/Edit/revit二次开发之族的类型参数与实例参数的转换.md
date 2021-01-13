1背景
小伙伴在做revit二次开发的时候，可能需要在族环境中将族的类型参数与实例参数相互转换。

2思路
1.使用族管理器FamilyManager，参见注释1
2.首先获取需要转换的参数（单个与批量），参见注释2，3
3.实例参数转类型参数，或者类型参数转实例转实例参数，参见注释4，5

3代码
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Autodesk.Revit.UI;
using Autodesk.Revit.DB;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;

namespace heiyedeqishi
{
[Transaction(TransactionMode.Manual)]
class Revit_API_Executable1 : IExternalCommand
{
public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
{
UIDocument uidoc = commandData.Application.ActiveUIDocument;
Document doc = uidoc.Document;
try
{
Transaction trans = new Transaction(doc, "参数转换");
trans.Start();
//1得到FamilyManager
FamilyManager familyManager = doc.FamilyManager;
//2通过名称得到族参数
FamilyParameter singleParameter = familyManager.get_Parameter("材料");
//3通过遍历得到族参数
foreach (FamilyParameter familyParameter in familyManager.Parameters)
{
//4类型转实例
familyManager.MakeInstance(familyParameter);
//5实例转类型
familyManager.MakeType(familyParameter);
}
trans.Commit();
return Result.Succeeded;
}
catch (Exception ex)
{
message = ex.Message;
return Result.Failed;
}
}
}
}

4注意事项
1.如果该参数本身就是实例参数，即使再转为实例参数也行，不会报错类型参数，同上
2.因为本身就在族环境中，所以不需要打开族文档，如果需要通过程序打开指定族文档，参见博主的另外一篇文章revit二次开发之批量打开族文档，样板文件，项目文件

