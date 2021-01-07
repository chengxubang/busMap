springboot整合jsp,完成公交车站路线图
=


如需源码请联系[程序帮](http://ll032.cn/HZ6vHa)：QQ1022287044


开发环境：
-----
1. jdk 8
2. intellij idea
3. tomcat 8
4. mysql 5.7
5. maven 3.6

所用技术：
-----
1. springboot
2. jsp
3. 数据静态初始化

项目介绍
----
使用springboot整合jsp，在后端写入公交路线名称和详细站点，前端页面可条件查询具体的内容，如公交路线，公交名称，车俩信息等。

运行效果
----
前台用户端：

- 路线选择

![路线选择](/image/路线选择.png)

- 路线详情

![路线详情](/image/路线详情.png)

数据准备:
----
- BusData.txt

![数据准备](/image/数据准备.png)

准备工作:
-----
1. pom.xml加入jsp模板引擎支持：
```diff
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
```
2. springboot配置jsp
```diff
spring.mvc.view.prefix=/
spring.mvc.view.suffix=.jsp
```

重要代码:
----
1. bus数据初始化
```diff
@PostConstruct
private void initBusData(){
    try{
        File file = new File(BusMap.getClass().getResource("/").getPath());
        FileReader fileReader = new FileReader(file.getPath()+"/static/BusData.txt","GBK"); //初始化BusData.txt 数据
        List<String> readLines = fileReader.readLines();
        for(String str:readLines){
            if(!"".equals(str)){
                String[] data=str.split("#");
                String way=data[0];         //几路线
                String location=data[1];/   /地名
                String[] locations=location.split(",");
                List<Bus> list=new ArrayList<>();
                for(int i=0;i<locations.length;i++){
                    int busnum=0;
                    if(i%4==0){	            //随机busnum
                        busnum=1;
                    }if(i%5==0){
                        busnum=2;
                    }
                    Bus bus=new Bus(locations[i],busnum);
                    list.add(bus);
                }
                WayList.add(way);	        //添加路线
                BusMap.put(way,list);       //添加车站
            }
        }
    }catch (Exception e){
        e.printStackTrace();
    }
}
```
2. 路线查询
```diff
@RequestMapping("/way")
public String search(HttpServletRequest request,String way) {
    try {
         if(null==way||"".equalsIgnoreCase(way)){
             request.setAttribute("list", BusMap.WayList); //没有搜索默认显示所有路线
             return "way";
         }else{
             List<String> wayList=new ArrayList<>();
             //模糊查询路线
             for(String str:BusMap.WayList){
                 if(str.indexOf(way)>-1){
                     wayList.add(str);
                 }
             }
             if(wayList.size()>0){
                 request.setAttribute("list", wayList); //模糊搜索出来的路线列表
                 return "way";
             }else{
                 return "noView"; //没有所选路线
             }
         }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return "way";
}

```
3. 公交车路线站展示
```diff
@RequestMapping("/view")
public String view(HttpServletRequest request,String way) {
    try {
        List<Bus> list= BusMap.getBusMap(way);
        if(list.size()>0){
            request.setAttribute("list",list ); //获取总路线
            request.setAttribute("firstBus", list.get(0).getLocation()); 		    //第一站
            request.setAttribute("lastBus", list.get(list.size()-1).getLocation()); //最后一站
            int size = list.size();
            size =(size-1)*99;
            request.setAttribute("size",size);
            return "view";
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return "noView";//没有对应公交车站
}
//前端页面数据渲染
<div class="pageContent" style="background: #eeeeee;">
    <div class="pageFormContent" layoutH="55">
        <div class="timeText">${firstBus}<----->${lastBus}
            <span>（ 首/末班车时间：<span style="color: red">6:00 / 23:00</span>）</span>
        </div>
        <div class="timezone" style="margin-top: 20px">
            <c:forEach var="list" items="${list}" varStatus="s">
                <div class="time" <c:if test="${s.index!=0}"> style="top: ${s.index*100+25}px;" a="1" </c:if> ><a onclick="javascript:alert(1);">${s.index+1}</a>
                    <h2>${list.location}</h2>
                    <c:if test="${list.busNum>0}">
                        <span class="timezone3"></span>
                        <div>
                            <p><span style="padding-left: 30px;">${list.busNum}辆公交</span></p>
                        </div>
                    </c:if>
                </div>
            </c:forEach>
        </div>
    </div>
    <div class="formBar"></div>
</div>
```

项目总结
----
1. 项目存放路径最好不要带中文路径，否则可能存在静态busData资源初始化失败
2. 页面时间车站路线所采用时间轴方式展示，长度动态计算，部分浏览器显示可能有点错位
3. 其他后续迭代功能后续开发，敬请关注

