# APACHE

## Excel
[官网](http://poi.apache.org/)


xml 代码
```xml
<dependency>   
  <groupId>org.apache.poi</groupId>  
  <artifactId>poi</artifactId>  
  <version>3.10.1</version>  
</dependency>   
   
 <dependency>  
     <groupId>org.apache.poi</groupId>  
     <artifactId>poi-ooxml</artifactId>  
     <version>3.9</version>  
 </dependency>  
```

Apache POI API的一些重点：

- Apache POI包含 Excel 97(-2007)文件格式(.xls)的Java实现 – HSSF。（彩蛋：H指Horrible）
- Apache POI包含 Excel 2007 OOXML文件格式(.xlsx)的Java实现 – XSSF。
- Apache POI的HSSF和XSSF API提供了读写和修改Excel电子表格的功能。
- Apache POI也提供了SXSSF API（流式XSSF），它是XSSF的扩展，用于写入非常大的excel文件。SXSSF API需求较小的内存，适用于在堆内存受限时处理较大excel文件的情况。
- 可以选择两种模型：Event Model和User Model。Event Model需求较小的内存，流式读取并处理每个单元；User Model更具备面向对象的特征，方便操作。
- Apache POI提供了对excel附加功能的完美支持，如公式、单元格样式、颜色、字体、数据验证、图像和超链接等。

以下是2个简单的demo:

1. 读取
	```java
    package com.cwind.poi;

    import java.io.File;
    import java.io.FileInputStream;
    import java.util.Iterator;

    import org.apache.poi.ss.usermodel.Cell;
    import org.apache.poi.ss.usermodel.DateUtil;
    import org.apache.poi.ss.usermodel.Row;
    import org.apache.poi.xssf.usermodel.XSSFSheet;
    import org.apache.poi.xssf.usermodel.XSSFWorkbook;

    public class SimpleDatasheetReader {

        public static void main(String[] args){
            try {
                File excel = new File("RunningMan.xlsx");
                FileInputStream fis = new FileInputStream(excel);
                XSSFWorkbook book = new XSSFWorkbook(fis);
                XSSFSheet sheet = book.getSheetAt(0);

                Iterator<Row> itr = sheet.iterator();

                System.out.println(itr.hasNext());

                // Iterating over Excel file in Java
                while (itr.hasNext()) {
                    Row row = itr.next();
                    System.out.println(row.getLastCellNum());
                    // Iterating over each column of Excel file
                    Iterator<Cell> cellIterator = row.cellIterator();
                    while (cellIterator.hasNext()) {

                        Cell cell = cellIterator.next();

                        switch (cell.getCellType()) {
                        case Cell.CELL_TYPE_STRING:
                            System.out.print(cell.getStringCellValue() + "\t");
                            break;
                        case Cell.CELL_TYPE_NUMERIC:
                            if(DateUtil.isCellDateFormatted(cell)){
                                System.out.print(cell.getDateCellValue() + "\t");
                            }else{
                                System.out.print(cell.getNumericCellValue() + "\t");
                            }

                            break;
                        case Cell.CELL_TYPE_BOOLEAN:
                            System.out.print(cell.getBooleanCellValue() + "\t");
                            break;
                        default:

                        }
                    }
                    System.out.println("");
                }

            }catch(Exception ex){
                ex.printStackTrace();
            }

        }
    }
    ```
2. 读取
	```java
    	package com.cwind.poi;

    import java.io.FileOutputStream;

    import org.apache.poi.hssf.usermodel.HSSFWorkbook;
    import org.apache.poi.ss.usermodel.Cell;
    import org.apache.poi.ss.usermodel.Row;
    import org.apache.poi.ss.usermodel.Sheet;
    import org.apache.poi.ss.usermodel.Workbook;
    import org.apache.poi.xssf.usermodel.XSSFWorkbook;

    /**
     * @author Billy Chen
     */
    public class SimpleDatasheetWriter {

        private static final String[] titles = {
                "姓名", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday"};

        //sample data to fill the sheet.
        private static final String[][] data = {
                {"AngelaBaby", "跑了", "跑了", "跑了", "跑了", "跑了"},
                {"邓超", "跑了", "跑了", "没跑", "跑了", "跑了" },
                {"王祖蓝", "没跑", "没跑", "没跑", "跑了", "跑了" },
                {"王宝强", "跑了", "跑了", "跑了", "跑了", "跑了" },
                {"郑恺", "跑了", "跑了", "跑了", "跑了", "跑了" }
        };

        public static void main(String[] args) throws Exception {
            Workbook wb;

            if(args.length > 0 && args[0].equals("-xls")) wb = new HSSFWorkbook();
            else wb = new XSSFWorkbook();

            Sheet sheet = wb.createSheet("Running Man");

            Row headerRow = sheet.createRow(0);
            headerRow.setHeightInPoints(12.75f);
            for (int i = 0; i < titles.length; i++) {
                Cell cell = headerRow.createCell(i);
                cell.setCellValue(titles[i]);
            }

            Row row;
            Cell cell;
            int rownum = 1;
            for (int i = 0; i < data.length; i++, rownum++) {
                row = sheet.createRow(rownum);
                if(data[i] == null) continue;

                for (int j = 0; j < data[i].length; j++) {
                    cell = row.createCell(j);
                    cell.setCellValue(data[i][j]);
                }
            }

            System.out.println("Default column width: " + sheet.getRow(0).getLastCellNum());
            System.out.println("Default column width: " + sheet.getRow(0).getPhysicalNumberOfCells());

            // Write the output to a file
            String file = "RunningMan.xls";
            if(wb instanceof XSSFWorkbook) file += "x";
            FileOutputStream out = new FileOutputStream(file);
            wb.write(out);
            out.close();
        }
    }
    ```
    
    
## 关于java反射