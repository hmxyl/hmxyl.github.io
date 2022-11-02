# echarts 图保存到word

1. 项目工程截图如下：

![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002.jpg)   

需要echarts包和生成word的freemarker-2.3.8.jar包

2. index.jsp页面中的代码：

```jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>  
<%  
String path = request.getContextPath();  
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  
%>  
  
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 01 Transitional//EN">  
<html>  
  <head>  
	<base href="<%=basePath%>">  
	  
	<title>ECharts 生成报表 并保存图片</title>  
	<meta http-equiv="pragma" content="no-cache">  
	<meta http-equiv="cache-control" content="no-cache">  
	<meta http-equiv="expires" content="0">      
	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">  
	<meta http-equiv="description" content="This is my page">  
	<!-- 
	<link rel="stylesheet" type="text/css" href="styles.css"> 
	-->  
  </head>  
	
  <body>  
	<!-- 为ECharts准备一个具备大小（宽高）的Dom -->  
	<div id="main" style="height:400px"></div>  
	<!-- ECharts单文件引入 -->  
	<script src="<%=basePath %>js/echarts/build/dist/echarts.js"></script>  
	<script type="text/javascript">  
		// 路径配置  
		require.config({  
			paths: {  
				echarts: '<%=basePath %>js/echarts/build/dist'  
			}  
		});  
		  
		// 使用  
		require(  
			[  
				'echarts',  
				'echarts/chart/bar' // 使用柱状图就加载bar模块，按需加载  
			],  
			function (ec) {  
				// 基于准备好的dom，初始化echarts图表  
				var myChart = ec.init(document.getElementById('main'));   
				  
				var option = {  
					tooltip: {  
						show: true  
					},  
					legend: {  
						data:['销量']  
					},  
					xAxis : [  
						{  
							type : 'category',  
							data : ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]  
						}  
					],  
					yAxis : [  
						{  
							type : 'value'  
						}  
					],  
					series : [  
						{  
							"name":"销量",  
							"type":"bar",  
							"data":[5, 20, 40, 10, 10, 20]  
						}  
					]  
				};  
		  
				// 为echarts对象加载数据   
				myChart.setOption(option);   
				setTimeout(exportImage, 2000);  
				function exportImage(){  
					var data = "a="+encodeURIComponent(myChart.getDataURL("png"));  
					var xmlhttp;  
					if (window.XMLHttpRequest) { // code for IE7+, Firefox, Chrome, Opera, Safari  
						xmlhttp = new XMLHttpRequest();  
					} else { // code for IE6, IE5  
						xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");  
					}  
					xmlhttp.open("POST","<%=path%>/servlet/saveImage",true);  
					xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");  
					xmlhttp.onreadystatechange = function() {  
						if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {  
							alert("保存成功");  
						}  
					}  
					xmlhttp.send(data);  
				}  
				  
			}  
		);  
	</script>  
</body>  
</html> 
```

3. web.xml

```xml
<?xml version="0" encoding="UTF-8"?>  
<web-app version="5"   
	xmlns="http://java.sun.com/xml/ns/javaee"   
	xmlns:xsi="http://www.worg/2001/XMLSchema-instance"   
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee   
	http://java.sun.com/xml/ns/javaee/web-app_2_xsd">  
  <display-name></display-name>   
  <welcome-file-list>  
	<welcome-file>index.jsp</welcome-file>  
  </welcome-file-list>  
	
  <servlet>  
	<servlet-name>saveImage</servlet-name>  
	<servlet-class>com.servlet.SaveImage</servlet-class>  
  </servlet>  
  
  <servlet-mapping>  
	<servlet-name>saveImage</servlet-name>  
	<url-pattern>/servlet/saveImage</url-pattern>  
  </servlet-mapping>  
</web-app>  

```

4. SaveImage.java

   ```java
   package com.servlet;  
     
   import java.io.File;  
   import java.io.FileInputStream;  
   import java.io.FileOutputStream;  
   import java.io.IOException;  
   import java.io.InputStream;  
   import java.io.OutputStream;  
   import java.util.ArrayList;  
   import java.util.HashMap;  
   import java.util.List;  
   import java.util.Map;  
     
   import javax.servlet.ServletException;  
   import javax.servlet.http.HttpServlet;  
   import javax.servlet.http.HttpServletRequest;  
   import javax.servlet.http.HttpServletResponse;  
     
   import com.word.WordUtil;  
     
   import sun.misc.BASE64Decoder;  
   import sun.misc.BASE64Encoder;  
     
   public class SaveImage extends HttpServlet {  
   	private static final long serialVersionUID = -1915463532411657451L;  
   	   
   	public void init() throws ServletException {    
   		 
   	}   
   	protected void doGet(HttpServletRequest request,HttpServletResponse response) throws ServletException, IOException {  
   		  
   	}  
      
   	protected void doPost(HttpServletRequest request,HttpServletResponse response) throws ServletException, IOException {  
   		String a = request.getParameter("a");  
   		try {  
   			String[] url = a.split(",");  
   			String u = url[1];  
   			// Base64解码  
   			byte[] b = new BASE64Decoder().decodeBuffer(u);  
   			WordUtil wordUtil = new WordUtil();  
   			String fileStr = wordUtil.saveFile();   
   			// 生成图片  
   			OutputStream out = new FileOutputStream(new File(fileStr+"\\test.png"));  
   			out.write(b);  
   			out.flush();  
   			out.close();  
   			  
   			//数据模拟，如果是真实编写，可以建service层，dao层进行数据的获取  
   			Map<String, Object> dataMap = new HashMap<String, Object>();  
   			dataMap = getData();  
   			//生产word  
   			wordUtil.createWord("ftl", fileStr+"\\test.doc", dataMap);  
   		} catch (Exception e) {  
   			e.printStackTrace();  
   		}  
   	}  
   	  
   	public Map<String, Object> getData() {  
   		Map<String, Object> dataMap = new HashMap<String, Object>();  
   		WordUtil wordUtil = new WordUtil();  
   		String fileStr = wordUtil.saveFile();  
   		  
   		dataMap.put("image", getImageStr(fileStr+"\\test.png"));  
   		List<Map<String,Object>> list = new ArrayList<Map<String,Object>>();  
   		for (int i = 0; i < 2; i++) {  
   			Map<String,Object> map = new HashMap<String,Object>();  
   			map.put("xuhao", i);  
   			map.put("neirong", "内容"+i);  
   			list.add(map);  
   			  
   		}  
   		dataMap.put("list", list);  
   		dataMap.put("info", "测试");  
   		return dataMap;  
   	}  
   	public String getImageStr(String imgFile) {  
   		InputStream in = null;  
   		byte[] data = null;  
   		try {  
   			in = new FileInputStream(imgFile);  
   			data = new byte[in.available()];  
   			in.read(data);  
   			in.close();  
   		} catch (IOException e) {  
   			e.printStackTrace();  
   		}  
   		BASE64Encoder encoder = new BASE64Encoder();  
   		return encoder.encode(data);  
   	}  
   }  
   ```

   5. WordUtil.java

      ```java
      package com.word;  
        
      import java.io.BufferedWriter;  
      import java.io.File;  
      import java.io.FileInputStream;  
      import java.io.FileNotFoundException;  
      import java.io.FileOutputStream;  
      import java.io.IOException;  
      import java.io.InputStream;  
      import java.io.OutputStreamWriter;  
      import java.io.Writer;  
      import java.util.Map;  
        
      import sun.misc.BASE64Encoder;  
      import freemarker.template.Configuration;  
      import freemarker.template.Template;  
      import freemarker.template.TemplateException;  
        
      public class WordUtil {  
      	private Configuration configuration = null;  
        
      	public WordUtil() {  
      		configuration = new Configuration();  
      		configuration.setDefaultEncoding("utf-8");  
        
      	}  
        
      	public void createWord(String templetName, String filePathName, Map<String, Object> dataMap) {  
      		configuration.setClassForTemplateLoading(this.getClass(), "/com/word"); // FTL文件所存在的位置  
      		Template t = null;  
      		try {  
      			// 获取模版文件  
      			t = configuration.getTemplate(templetName);  
      		} catch (IOException e) {  
      			e.printStackTrace();  
      		}  
      		// 生成文件的路径和名称  
      		File outFile = new File(filePathName);  
      		Writer out = null;  
      		try {  
      			out = new BufferedWriter(new OutputStreamWriter(  
      					new FileOutputStream(outFile)));  
      		} catch (FileNotFoundException e1) {  
      			eprintStackTrace();  
      		}  
        
      		try {  
      			t.process(dataMap, out);  
      		} catch (TemplateException e) {  
      			e.printStackTrace();  
      		} catch (IOException e) {  
      			e.printStackTrace();  
      		}  
      	}  
        
      	public String getImageStr(String imgFile) {  
      		InputStream in = null;  
      		byte[] data = null;  
      		try {  
      			in = new FileInputStream(imgFile);  
      			data = new byte[in.available()];  
      			in.read(data);  
      			in.close();  
      		} catch (IOException e) {  
      			e.printStackTrace();  
      		}  
      		BASE64Encoder encoder = new BASE64Encoder();  
      		return encoder.encode(data);  
      	}  
      	public String saveFile() {  
      		String nowpath = System.getProperty("user.dir");  
      		String path = nowpath.replace("bin", "webapps");  
      		path += "\\"+"TestWeb"+"\\"+"word";  
      		File tmp = new File(path);  
      		if (!tmp.exists()) {  
      			tmp.mkdirs();  
      		}  
      		return path;  
      	}  
      }  
      ```

      6. 模版的制作

         - 新建word文档，内容如下图

           ![image-20211213174321240](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211213174321240.png)   

         - 把word文档另存为Word 2003 XML 文档，打开内容，如下图 

           ![image-20211213174404109](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211213174404109.png)   

           在`<w:tr>`之前`<#list list as list>`,在`</w:tr>`之后加`</#list>`，如果不循环，不需要加。 

           接在把图片生产的一大串字符改成`${image}`,试情况改此文件的编码。

         - 把修改好的xml后缀改为ftl。模版做好，放在指定位置即可。跑完程序后，生产如下word

           ![image-20211213174517008](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211213174517008.png)   