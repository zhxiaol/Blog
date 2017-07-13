---
layout: post
title: ocload,poi,pin4j
tags:
- java

categories: java
description: ocload,poi,pin4j
---
#### 区域批量导入
##### 一键上传插件使用 jquery-ocupload.js
> one click upload
> ajax不能做文件上传,既可以不跳页面还不跳页面

```
<html>
<head>
    <script type="text/javascript" src="${pageContext.request.contextPath}/js/jquery.ocupload-1.1.2.js"></script>
    <script type="text/javascript">
        $(function () {
            $("#but1").upload({
                action:'abc',
                name:'myFile'
            })
        })
    </script>
</head>
<body>
    <%--//上传文件且不跳转页面--%>
    <%--<form target="myFrame" action="abc" method="post" enctype="multipart/form-data">--%>
        <%--<input name="myFile" title="file">--%>
        <%--<input type="submit" value="上传">--%>
    <%--</form>--%>
    <%--<iframe name="myFrame" height="0" width="0" frameborder="0"></iframe>--%>
    <input id="but1" type="submit" value="upload">
</body>
</html>
```

###### region.jsp
```javascript
//导入ocupload.js
<script type="text/javascript" src="${pageContext.request.contextPath}/js/jquery.ocupload-1.1.2.js"></script>
// 收派标准数据表格
$('#grid').datagrid({
    toolbar:[ {   //工具栏
    id: 'button-import',
    text: '导入',
    iconCls: 'icon-redo'
}]});

var toolbar =
//绑定一键上传
$(function () {
    $("#button-import").upload({
        action:'${pageContext.request.contextPath}/regionAction_importXls.action',
        name:'myFile'
    })
})
```

##### Apache POI
> Apache POI提供API给Java程式对Microsoft Office格式档案读和写的功能
```java
//导入poi-xxx.jar
public class POITest {
    //使用POI解析Excel
    @Test
    public void test1() throws IOException {
        HSSFWorkbook workbook =new HSSFWorkbook(new FileInputStream(new File("/zhxiaol/aaa.xls")));
        HSSFSheet sheet=workbook.getSheetAt(0);//取出第一个sheet
        for (Row row : sheet) {//取出第一行
            String v1 = row.getCell(0).getStringCellValue();//第一个单元格
            String v2 = row.getCell(1).getStringCellValue();
            String v3 = row.getCell(2).getStringCellValue();
            String v4 = row.getCell(3).getStringCellValue();
            String v5 = row.getCell(4).getStringCellValue();
            System.out.println(v1+"\t"+v2+"\t"+v3+"\t"+v4+"\t"+v5);
        }
    }
}
```

##### pinyin4j
> 汉字转换为拼音
```java
//1.导入pinyin4j.jar
//2.导入PinYin4jUtils

public class Pinyin4JTest {
    @Test
    public void test1(){
        String province="河北省";
        String city="石家庄市";
        String district="长安区";

        //城市编码-->shijiazhuang
        city =city.substring(0,city.length()-1);
        String[] arrcity = PinYin4jUtils.stringToPinyin(city);
        String citycode = StringUtils.join(arrcity, "");
        System.out.println(citycode);

        //简码--HBSJZCA
        province=province.substring(0,province.length()-1);
        district=district.substring(0,district.length()-1);
        String info=province+city+district;
        String[] arrcode = PinYin4jUtils.getHeadByString(info);
        String shortcode = StringUtils.join(arrcode, "");
        System.out.println(shortcode);

    }
}
```

###### 批量导入regionAction
```java
private File myFile;//用来接收文件上传
public String importXls() throws IOException {
    String flag="1";
    try {
        //使用POI解析Excel文件
        HSSFWorkbook workbook=new HSSFWorkbook(new FileInputStream(myFile));
        HSSFSheet sheet = workbook.getSheetAt(0);//获得第一个sheet页
        ArrayList<Region> list = new ArrayList<>();
        for (Row row : sheet) {
            int rowNum = row.getRowNum();
            if(rowNum>0){//第一行标题忽略
                String id = row.getCell(0).getStringCellValue();
                String province = row.getCell(1).getStringCellValue();
                String city = row.getCell(2).getStringCellValue();
                String district = row.getCell(3).getStringCellValue();
                String postcode = row.getCell(4).getStringCellValue();
                //城市编码-->shijiazhuang
                city =city.substring(0,city.length()-1);
                String[] arrcity = PinYin4jUtils.stringToPinyin(city);
                String citycode = StringUtils.join(arrcity, "");
                //简码--HBSJZCA
                province=province.substring(0,province.length()-1);
                district=district.substring(0,district.length()-1);
                String info=province+city+district;
                String[] arrcode = PinYin4jUtils.getHeadByString(info);
                String shortcode = StringUtils.join(arrcode, "");
                Region region=new Region(id,province,city,district,postcode,shortcode,citycode);
                list.add(region);
            }
        }
        service.saveBatch(list);
    } catch (Exception e) {
        e.printStackTrace();
        flag="0";
    }
    HttpServletResponse resp = ServletActionContext.getResponse();
    resp.setContentType("text/html;charset=UTF-8");
    resp.getWriter().print(flag);
    return NONE;
}
```

###### 批量导入 RegionServiceImpl
```java
@Service
@Transactional
public class RegionServiceImpl implements IRegionService {
    @Autowired
    private IRegionDao dao;
    @Override
    public void saveBatch(List<Region> list) {
        for (Region region : list) {
            dao.saveOrUpdate(region);
        }
    }
}
```

###### 区域分页查询
```java
//RegionAction.java
//分页查询
public String pageQuery() throws IOException {
    service.pageQuery(pageBaen);
    this.writePageBean2Json(new String[]{"currentPage","detachedCriteria","pageSize"});
    return NONE;
}

//BaseAction.java
public BaseAction() {
    try {
        ParameterizedType type= (ParameterizedType) this.getClass().getGenericSuperclass();
        Class<T> clazz = (Class<T>) type.getActualTypeArguments()[0];
        detachedCriteria=DetachedCriteria.forClass(clazz);
        pageBaen.setDetachedCriteria(detachedCriteria);
        model=clazz.newInstance();
    } catch (Exception e) {
        e.printStackTrace();
    }
}

protected PageBaen pageBaen = new PageBaen();
DetachedCriteria detachedCriteria =null;
public void setPage(int page) {
    pageBaen.setCurrentPage(page);
}

public void setRows(int rows) {
    pageBaen.setPageSize(rows);
}

public void writePageBean2Json(String[] excludes)throws IOException{
    JsonConfig jsonConfig = new JsonConfig();
    jsonConfig.setExcludes(excludes);
    JSONObject jsonObject=JSONObject.fromObject(pageBaen,jsonConfig);
    String json = jsonObject.toString();
    HttpServletResponse resp = ServletActionContext.getResponse();
    resp.setContentType("text/json;charset=UTF-8");
    resp.getWriter().print(json);
}
```

##### 添加分区
##### EasyUI combobox 下拉框
```javascript
<select class="easyui-combobox">
    <option>北京</option>
    <option>上海</option>
    <option>天津</option>
</select>

<input class="easyui-combobox" data-options="url:'/json/data.json',valueField:'id',textField:'name'">
```

###### 添加分区--选择区域
```javascript
//subarea.jsp
<td>选择区域</td>
<td>
   <input class="easyui-combobox" name="region.id"  
               data-options="valueField:'id',textField:'name',mode:'remote'
               ,url:'${pageContext.request.contextPath}/regionAction_listajax.action'" />
</td>
//RegionAction.java
//查询所有的区域数据 返回json
public String listajax()throws IOException{
    List<Region> list;
    if(StringUtils.isNotBlank(q)){
        list=regionService.findByQ(q);
    }else {
        list=regionService.findAll();
    }

    String[]excludes=new String[]{"subareas"};
    this.WriteList2Json(list,excludes);
    return NONE;
}

//RegionDaoImpl.java
@Repository
public class RegionDaoImpl extends BaseDaoImpl<Region> implements IRegionDao {
    @Override
    public List<Region> findByQ(String q) {
        String hql="FROM Region WHERE province LIKE ? OR city LIKE ? OR district LIKE ?";
        return this.getHibernateTemplate().find(hql,"%"+q+"%","%"+q+"%","%"+q+"%");
    }
}

//BaseAction.java
public void WriteList2Json(List list, String[] excludes) throws IOException {
    JsonConfig jsonConfig = new JsonConfig();
    jsonConfig.setExcludes(excludes);
    JSONArray jsonObject= JSONArray.fromObject(list,jsonConfig);
    String json = jsonObject.toString();
    System.out.println(json);
    HttpServletResponse resp = ServletActionContext.getResponse();
    resp.setContentType("text/json;charset=UTF-8");
    resp.getWriter().print(json);
}

//Region.java
public String getName(){
    return province+city+district;
}
```

###### 添加分区--subarea.jsp
```javascript
//添加分区
$("#save").click(function () {
   var v=$("#addSubareaForm").form("validate");
   if(v){
       $("#addSubareaForm").submit();
   }
      })
<form id="addSubareaForm" action="${pageContext.request.contextPath}/subareaAction_add.action" method="post">
</form>
```
###### 添加分区--struts.xml
```javascript
<!--分区管理-->
<action name="subareaAction_*" class="subareaAction" method="{1}">
    <result name="list">/WEB-INF/pages/base/subarea.jsp</result>
</action>
```

###### 添加分区--SubareaAction.java
```java
public String add(){
    System.out.println(model);
    subareaService.save(model);
    return "list";
}
```
###### 添加分区--SubareaServiceImpl.java
```java
@Resource
private ISubareaDao subareaDao;

@Override
public void save(Subarea model) {
    subareaDao.save(model);
}
```

##### 分区--分页查询
###### 分页查询--subarea.jsp
```javascript
<script type="text/javascript">
   //将指定表单数据序列化成json
                      $.fn.serializeJson=function(){
                          var serializeObj={};
                          var array=this.serializeArray();
                          $(array).each(function(){
                              if(serializeObj[this.name]){
                                  if($.isArray(serializeObj[this.name])){
                                      serializeObj[this.name].push(this.value);
                                  }else{
                                      serializeObj[this.name]=[serializeObj[this.name],this.value];
                                  }
                              }else{
                                  serializeObj[this.name]=this.value;
                              }
                          });
                          return serializeObj;
                      };
                      //绑定事件
                      $(function () {
                          $("#btn").click(function(){
                              var json=$("#searchForm").serializeJson();//{id:xx,name:xxx}
                              $("#grid").datagrid("load",json);//重新发送ajax请求提交参数
                              $("#searchWindow").window("close");//关闭窗口
                          });
                      })
</script>

// 收派标准数据表格
$('#grid').datagrid( {
   url : "${pageContext.request.contextPath}/subareaAction_pageQuery.action"
});
```

###### 分页查询--SubareaAction.java
```java
public String pageQuery()throws IOException{
    //在查询之前封装条件
    DetachedCriteria detachedCriteria = pageBaen.getDetachedCriteria();
    String addresskey = model.getAddresskey();
    Region region = model.getRegion();
    if(StringUtils.isNotBlank(addresskey)){//按照地址模糊查询
        detachedCriteria.add(Restrictions.like("addressKey",addresskey));
    }
    if(region!=null){
        detachedCriteria.createAlias("region","r");//创建别名 用于多表关联查询
        String province = region.getProvince();
        String city =region.getCity();
        String district=region.getDistrict();
        if(StringUtils.isNotBlank(province)){//按照省份模糊查询
            detachedCriteria.add(Restrictions.like("r.province","%"+province+"%"));
        }
        if(StringUtils.isNotBlank(city)){//按照城市模糊查询
            detachedCriteria.add(Restrictions.like("r.city","%"+city+"%"));
        }
        if(StringUtils.isNotBlank(district)){//按照市区模糊查询
            detachedCriteria.add(Restrictions.like("r.district","%"+district+"%"));
        }
    }
    subareaService.pageQuery(pageBaen);
    this.writePageBean2Json(new String[]{"currentPage","pageSize","detachedCriteria","decidedzone","subareas"});
    return NONE;
}
```

###### 分页查询 subarea.hbm.xml
```javascript
<!--关闭懒加载 解决json序列化问题 -->
<many-to-one lazy="false" name="region" class="com.itheima.bos.domain.Region" fetch="select">
    <column name="region_id" length="32" />
</many-to-one>
```

##### 分区数据导出功能
###### 分区导出--subarea.jsp
```javascript
//导出文件 发起一次请求 不能使用ajax 要同步提交
function doExport(){
    window.location.href="${pageContext.request.contextPath}/subareaAction_exportXls.action";
}
```
###### 分区导出--SubareaAction.java
```java
//使用POI写入Excel文件,提供下载
public String exportXls() throws Exception {
    List<Subarea> list = subareaService.findAll();
    //在内存中创建一个Excel文件,通过输出流写入到客户端
    HSSFWorkbook workbook = new HSSFWorkbook();
    //创建一个sheet页
    HSSFSheet sheet = workbook.createSheet("分区数据");
    //创建标题行
    HSSFRow headRow = sheet.createRow(0);
    headRow.createCell(0).setCellValue("分区编号");
    headRow.createCell(1).setCellValue("区域编号");
    headRow.createCell(2).setCellValue("地址关键字");
    headRow.createCell(3).setCellValue("省市区");
    for (Subarea subarea : list) {
        HSSFRow row = sheet.createRow(sheet.getLastRowNum() + 1);
        row.createCell(0).setCellValue(subarea.getId());
        row.createCell(1).setCellValue(subarea.getRegion().getId());
        row.createCell(2).setCellValue(subarea.getAddresskey());
        Region region = subarea.getRegion();
        row.createCell(3).setCellValue(region.getProvince() + region.getCity() + region.getDistrict());
    }
    HttpServletResponse resp = ServletActionContext.getResponse();
    //一个流两个头
    String agent = ServletActionContext.getRequest().getHeader("User-Agent");
    String filename = "分区数据.xls";
    filename = FileUtils.encodeDownloadFilename(filename, agent);
    String type = ServletActionContext.getServletContext().getMimeType(filename);
    resp.setContentType(type);
    resp.setHeader("content-disposition", "attachment;filename=" + filename);
    ServletOutputStream out = resp.getOutputStream();
    workbook.write(out);
    return NONE;
}
```
