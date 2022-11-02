#  前端Table数据导出Excel使用HSSFWorkbook

## 一、实现原理：

1. 前端查询列表数据并渲染至table(<table>...</table>)表格

2. 表格html代码传输至后台

3. 后台把html转成Excel输出流返回前端

4. 前端自动调用下载器完成下载

注：因为渲染之后的hmtl代码数据量有可能很大，因此要使用POST方式的form表单方式提交。

## 二、实现步骤：

1. 查询后台数据并且渲染至页面table在此忽略,直接用一下静态html代替:

   ```html
   <div id="table">
       <table id="targetTable">
           <thead>
               <tr>
                   <th style="background: fuchsia;">名次</th>
                   <th>姓名</th>
                   <th>成绩</th>
               </tr>
           </thead>
           <tbody>
               <tr align="center">
                   <td>1</td>
                   <td>小明</td>
                   <td>100</td>
               </tr>
               <tr align="center">
                   <td>2</td>
                   <td>小红</td>
                   <td>95.5</td>
               </tr>
           </tbody>
       </table>
   </div>
   ```

2. 表格html传输至后台进行解析生成excel输出流:

   ```javascript
   <script language="JavaScript">
   function makeFormSubmit(url, excelName, tableHtml) {
       // 创建一个 form
       var form1 = document.createElement("form");
       // 添加到 body 中
       document.body.appendChild(form1);
       // 创建输入
       var input = document.createElement("input");
       input.name = "excelName";
       input.value = excelName;
   
       var input1 = document.createElement("input");
       input1.name = "tableHtml";
       input1.value = tableHtml;
   
       // 将输入框插入到 form 中
       form1.appendChild(input);
       form1.appendChild(input1);
   
       // form 的提交方式
       form1.method = "POST";
       // form 提交路径
       form1.action = url;
       //以附件方式 解决数据量大的问题        
       form1.enctype = "multipart/form-data";
       // 对该 form 执行提交
       form1.submit();
       // 删除该 form
       document.body.removeChild(form1);
   }
   
   function loadExcel() {
       var tableHtml = document.getElementById('table').innerHTML;
       makeFormSubmit("/admin/api/excel/loadExcel", "测试excel", tableHtml);
   }
   </script>
   ```

3. 后台转换代码:

   必要的jar包，maven项目所以直接pom导入:

   ```xml
   <!-- Excel导出相关 -->
   <dependency>
       <groupId>org.apache.poi</groupId>
       <artifactId>poi-examples</artifactId>
       <version>3.8</version>
   </dependency>
   ```

   跨行元素元数据类：

   ```java
   package com.loanofficer.utils.excel;
   
   /**
    * 跨行元素元数据
    *
    */
   public class CrossRangeCellMeta {
   
     public CrossRangeCellMeta(
       int firstRowIndex,
       int firstColIndex,
       int rowSpan,
       int colSpan
     ) {
       super();
       this.firstRowIndex = firstRowIndex;
       this.firstColIndex = firstColIndex;
       this.rowSpan = rowSpan;
       this.colSpan = colSpan;
     }
   
     private int firstRowIndex;
     private int firstColIndex;
     private int rowSpan; // 跨越行数
     private int colSpan; // 跨越列数
   
     int getFirstRow() {
       return firstRowIndex;
     }
   
     int getLastRow() {
       return firstRowIndex + rowSpan - 1;
     }
   
     int getFirstCol() {
       return firstColIndex;
     }
   
     int getLastCol() {
       return firstColIndex + colSpan - 1;
     }
   
     int getColSpan() {
       return colSpan;
     }
   }
   ```

   将html table 转成 excel类

   ```java
   package com.loanofficer.utils.excel;
   
   import java.io.File;
   import java.io.FileOutputStream;
   import java.util.ArrayList;
   import java.util.List;
   import org.apache.commons.lang3.StringUtils;
   import org.apache.commons.lang3.math.NumberUtils;
   import org.apache.poi.hssf.usermodel.HSSFCell;
   import org.apache.poi.hssf.usermodel.HSSFCellStyle;
   import org.apache.poi.hssf.usermodel.HSSFFont;
   import org.apache.poi.hssf.usermodel.HSSFRow;
   import org.apache.poi.hssf.usermodel.HSSFSheet;
   import org.apache.poi.hssf.usermodel.HSSFWorkbook;
   import org.apache.poi.hssf.util.HSSFColor;
   import org.apache.poi.ss.util.CellRangeAddress;
   import org.dom4j.Document;
   import org.dom4j.DocumentException;
   import org.dom4j.DocumentHelper;
   import org.dom4j.Element;
   
   /**
    * 将html table 转成 excel
    *
    * 记录下来所占的行和列，然后填充合并
    */
   public class ConvertHtml2Excel {
   
     /**
      * html表格转excel
      *
      * @param tableHtml 如
      * <table>
      * ..
      * </table>
      * @return
      */
     public static HSSFWorkbook table2Excel(String tableHtml) {
       HSSFWorkbook wb = new HSSFWorkbook();
       HSSFSheet sheet = wb.createSheet();
   
       HSSFCellStyle style = wb.createCellStyle();
       style.setAlignment(HSSFCellStyle.ALIGN_CENTER);
   
       List<CrossRangeCellMeta> crossRowEleMetaLs = new ArrayList<>();
       int rowIndex = 0;
       try {
         Document data = DocumentHelper.parseText(tableHtml);
         // 生成表头
         Element thead = data.getRootElement().element("thead");
         HSSFCellStyle titleStyle = getTitleStyle(wb);
         if (thead != null) {
           List<Element> trLs = thead.elements("tr");
           for (Element trEle : trLs) {
             HSSFRow row = sheet.createRow(rowIndex);
             List<Element> thLs = trEle.elements("th");
             makeRowCell(thLs, rowIndex, row, 0, titleStyle, crossRowEleMetaLs);
             row.setHeightInPoints(17);
             rowIndex++;
           }
         }
         // 生成表体
         Element tbody = data.getRootElement().element("tbody");
         if (tbody != null) {
           HSSFCellStyle contentStyle = getContentStyle(wb);
           List<Element> trLs = tbody.elements("tr");
           for (Element trEle : trLs) {
             HSSFRow row = sheet.createRow(rowIndex);
             List<Element> thLs = trEle.elements("th");
             int cellIndex = makeRowCell(
               thLs,
               rowIndex,
               row,
               0,
               titleStyle,
               crossRowEleMetaLs
             );
             List<Element> tdLs = trEle.elements("td");
             makeRowCell(
               tdLs,
               rowIndex,
               row,
               cellIndex,
               contentStyle,
               crossRowEleMetaLs
             );
             row.setHeightInPoints(18);
             rowIndex++;
           }
         }
         // 合并表头
         for (CrossRangeCellMeta crcm : crossRowEleMetaLs) {
           sheet.addMergedRegion(
             new CellRangeAddress(
               crcm.getFirstRow(),
               crcm.getLastRow(),
               crcm.getFirstCol(),
               crcm.getLastCol()
             )
           );
         }
       } catch (DocumentException e) {
         e.printStackTrace();
       }
   
       //自动调整列宽
       for (int i = 0; i < 15; i++) {
         sheet.autoSizeColumn((short) i);
       }
       return wb;
     }
   
     /**
      * 生产行内容
      *
      * @return 最后一列的cell index
      */
     /**
      * @param tdLs th或者td集合
      * @param rowIndex 行号
      * @param row POI行对象
      * @param startCellIndex
      * @param cellStyle 样式
      * @param crossRowEleMetaLs 跨行元数据集合
      * @return
      */
     private static int makeRowCell(
       List<Element> tdLs,
       int rowIndex,
       HSSFRow row,
       int startCellIndex,
       HSSFCellStyle cellStyle,
       List<CrossRangeCellMeta> crossRowEleMetaLs
     ) {
       int i = startCellIndex;
       for (int eleIndex = 0; eleIndex < tdLs.size(); i++, eleIndex++) {
         int captureCellSize = getCaptureCellSize(rowIndex, i, crossRowEleMetaLs);
         while (captureCellSize > 0) {
           for (int j = 0; j < captureCellSize; j++) {
             // 当前行跨列处理（补单元格）
             row.createCell(i);
             i++;
           }
           captureCellSize = getCaptureCellSize(rowIndex, i, crossRowEleMetaLs);
         }
         Element thEle = tdLs.get(eleIndex);
         String val = thEle.getTextTrim();
         if (StringUtils.isBlank(val)) {
           Element e = thEle.element("a");
           if (e != null) {
             val = e.getTextTrim();
           }
         }
         HSSFCell c = row.createCell(i);
         if (NumberUtils.isNumber(val)) {
           c.setCellValue(Double.parseDouble(val));
           c.setCellType(HSSFCell.CELL_TYPE_NUMERIC);
         } else {
           c.setCellValue(val);
         }
         c.setCellStyle(cellStyle);
         int rowSpan = NumberUtils.toInt(thEle.attributeValue("rowspan"), 1);
         int colSpan = NumberUtils.toInt(thEle.attributeValue("colspan"), 1);
         if (rowSpan > 1 || colSpan > 1) {
           // 存在跨行或跨列
           crossRowEleMetaLs.add(
             new CrossRangeCellMeta(rowIndex, i, rowSpan, colSpan)
           );
         }
         if (colSpan > 1) {
           // 当前行跨列处理（补单元格）
           for (int j = 1; j < colSpan; j++) {
             i++;
             row.createCell(i);
           }
         }
       }
       return i;
     }
   
     /**
      * 获得因rowSpan占据的单元格
      *
      * @param rowIndex 行号
      * @param colIndex 列号
      * @param crossRowEleMetaLs 跨行列元数据
      * @return 当前行在某列需要占据单元格
      */
     private static int getCaptureCellSize(
       int rowIndex,
       int colIndex,
       List<CrossRangeCellMeta> crossRowEleMetaLs
     ) {
       int captureCellSize = 0;
       for (CrossRangeCellMeta crossRangeCellMeta : crossRowEleMetaLs) {
         if (
           crossRangeCellMeta.getFirstRow() < rowIndex &&
           crossRangeCellMeta.getLastRow() >= rowIndex
         ) {
           if (
             crossRangeCellMeta.getFirstCol() <= colIndex &&
             crossRangeCellMeta.getLastCol() >= colIndex
           ) {
             captureCellSize = crossRangeCellMeta.getLastCol() - colIndex + 1;
           }
         }
       }
       return captureCellSize;
     }
   
     /**
      * 获得标题样式
      *
      * @param workbook
      * @return
      */
     private static HSSFCellStyle getTitleStyle(HSSFWorkbook workbook) {
       short titlebackgroundcolor = HSSFColor.GREY_25_PERCENT.index;
       short fontSize = 12;
       String fontName = "宋体";
       HSSFCellStyle style = workbook.createCellStyle();
       style.setVerticalAlignment(HSSFCellStyle.VERTICAL_CENTER);
       style.setAlignment(HSSFCellStyle.ALIGN_CENTER);
       style.setBorderBottom((short) 1);
       style.setBorderTop((short) 1);
       style.setBorderLeft((short) 1);
       style.setBorderRight((short) 1);
       style.setFillPattern(HSSFCellStyle.SOLID_FOREGROUND);
       style.setFillForegroundColor(titlebackgroundcolor); // 背景色
   
       HSSFFont font = workbook.createFont();
       font.setFontName(fontName);
       font.setFontHeightInPoints(fontSize);
       font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);
       style.setFont(font);
       return style;
     }
   
     /**
      * 获得内容样式
      *
      * @param wb
      * @return
      */
     private static HSSFCellStyle getContentStyle(HSSFWorkbook wb) {
       short fontSize = 12;
       String fontName = "宋体";
       HSSFCellStyle style = wb.createCellStyle();
       style.setBorderBottom((short) 1);
       style.setBorderTop((short) 1);
       style.setBorderLeft((short) 1);
       style.setBorderRight((short) 1);
   
       HSSFFont font = wb.createFont();
       font.setFontName(fontName);
       font.setFontHeightInPoints(fontSize);
       style.setFont(font);
       return style;
     }
   
     public static void main(String[] args) {
       String c = new String(
         "<table id=\"targetTable\">\n" +
         " <thead>\n" +
         " <tr align=\"center\">\n" +
         " <th>名次</th>\n" +
         " <th>姓名</th>\n" +
         " <th>成绩</th>\n" +
         " </tr>\n" +
         " </thead>\n" +
         " <tbody>\n" +
         " <tr align=\"center\">\n" +
         " <td>1</td>\n" +
         " <td>小明</td>\n" +
         " <td>100</td>\n" +
         " </tr>\n" +
         " <tr align=\"center\">\n" +
         " <td>2</td>\n" +
         " <td>小红</td>\n" +
         " <td>95.5</td>\n" +
         " </tr>\n" +
         " </tbody>\n" +
         "</table>"
       );
       HSSFWorkbook wb = table2Excel(c);
       try {
         FileOutputStream fos = new FileOutputStream(new File("1.xls"));
         wb.write(fos);
         fos.flush();
         fos.close();
       } catch (Exception e) {
         // TODO Auto-generated catch block
         e.printStackTrace();
       }
     }
   }
   ```

   Controller类

   ```java
   package com.loanofficer.web.api;
   
   import com.loanofficer.utils.excel.ConvertHtml2Excel;
   import java.io.IOException;
   import java.io.UnsupportedEncodingException;
   import java.net.URLEncoder;
   import javax.servlet.ServletOutputStream;
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import org.apache.poi.hssf.usermodel.*;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @Controller
   @RequestMapping(value = "/api/excel")
   public class ExcelController {
   
     @PostMapping(value = "/loadExcel")
     private void loadExcelGet(HttpServletRequest req, HttpServletResponse resp) {
       try {
         req.setCharacterEncoding("UTF-8");
       } catch (UnsupportedEncodingException e) {
         e.printStackTrace();
       }
       // 设置文件的mime类型
       resp.setContentType("application/vnd.ms-excel");
   
       // 存储编码后的文件名
       String excelName = "name";
       // 存储文件名称
       String n = req.getParameter("excelName");
   
       try {
         excelName = URLEncoder.encode(n, "utf-8");
       } catch (UnsupportedEncodingException e1) {
         e1.printStackTrace();
       }
   
       resp.setHeader(
         "content-disposition",
         "attachment;filename=" +
         excelName +
         ".xls;filename*=utf-8''" +
         excelName +
         ".xls"
       );
   
       String tableHtml = req.getParameter("tableHtml");
   
       // 从session中删除saveExcelMsg属性
       req.getSession().removeAttribute("saveExcelMsg");
       // 定义一个输出流
       ServletOutputStream sos = null;
   
       HSSFWorkbook wb = ConvertHtml2Excel.table2Excel(tableHtml);
   
       try {
         // 保存到文件中
         sos = resp.getOutputStream();
         wb.write(sos);
       } catch (Exception e) {
         e.printStackTrace();
       } finally {
         if (sos != null) {
           try {
             sos.close();
           } catch (IOException e) {
             e.printStackTrace();
           }
         }
       }
     }
   }
   ```

   