1.导入easyExcel的Maven依赖
```
<dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>easyexcel</artifactId>
       <version>1.1.2-beta5</version>
</dependency>
```
2.List数据导入Excel
```
OutputStream out = new FileOutputStream("C:\\Users\\liu76\\Desktop\\a.xlsx");
ExcelWriter writer = EasyExcelFactory.getWriter(out);
Sheet sheet1 = new Sheet(1, 3);
sheet1.setSheetName("第一个sheet");
sheet1.setHead(createTestListStringHead());
sheet1.setAutoWidth(Boolean.TRUE);
writer.write1(createTestListObject(), sheet1);

writer.finish();
out.close();


 public static List<List<String>> createTestListStringHead(){
        List<List<String>> head=new ArrayList<>();
        List<String> headCoulumn1 = new ArrayList<String>();
        List<String> headCoulumn2 = new ArrayList<String>();
        List<String> headCoulumn3 = new ArrayList<String>();
        List<String> headCoulumn4 = new ArrayList<String>();
        List<String> headCoulumn5 = new ArrayList<String>();

        headCoulumn1.add("第一列");headCoulumn1.add("第一列");headCoulumn1.add("第一列");
        headCoulumn2.add("第一列");headCoulumn2.add("第一列");headCoulumn2.add("第一列");

        headCoulumn3.add("第二列");headCoulumn3.add("第二列");headCoulumn3.add("第二列");
        headCoulumn4.add("第三列");headCoulumn4.add("第三列2");headCoulumn4.add("第三列2");
        headCoulumn5.add("第一列");headCoulumn5.add("第3列");headCoulumn5.add("第4列");

        head.add(headCoulumn1);
        head.add(headCoulumn2);
        head.add(headCoulumn3);
        head.add(headCoulumn4);
        head.add(headCoulumn5);
        return head;
    }
    
    
     public static List<List<Object>> createTestListObject(){
           List<List<Object>> object=  new ArrayList<>();
           for (int i=0;i<1000;i++){
               List<Object> da =new ArrayList<>();
               da.add("字符串"+i);
               da.add(Long.valueOf(187837834L+i));
               da.add(Integer.valueOf(2233+i));
               da.add(Double.valueOf(2233.00+i));
               da.add(Float.valueOf(2233.0f+i));
               da.add(new Date());
               da.add(new BigDecimal("3434343433554545"+i));
               da.add(Short.valueOf((short)i));
               object.add(da);
           }
           return object;
        }

```
