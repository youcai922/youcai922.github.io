多文件打包为zip

```java
@RequestMapping(value = "/writeZip", method = RequestMethod.GET)
public void writeZip(HttpServletResponse response){
    String zipname="ceshi";
    List<String> files = new ArrayList();
    files.add("D:\\img\\d3007c46-a7ca-4990-949c-460df412ad4a.png");
    files.add("D:\\img\\project.docx");
    files.add("D:\\img\\response.json");
    String fileName = zipname + ".zip";
    response.setContentType("application/zip");
    response.setHeader("content-disposition", "attachment;filename=" + fileName);

    ZipOutputStream zos = null;
    BufferedInputStream bis = null;
    try{
        zos = new ZipOutputStream(response.getOutputStream());
        byte[] buf = new byte[8192];
        int len;
        for (int i = 0; i < files.size(); i++) {
            File file = new File(files.get(i));
            if (!file.isFile()) continue;
            ZipEntry ze = new ZipEntry(file.getName());
            zos.putNextEntry(ze);
            bis = new BufferedInputStream(new FileInputStream(file));
            while ((len = bis.read(buf)) > 0) {
                zos.write(buf, 0, len);
            }
            zos.closeEntry();
        }
        zos.closeEntry();
    }catch(Exception ex){
        ex.printStackTrace();
    }finally {
        if(bis != null){
            try{
                bis.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
        if(zos != null){
            try{
                zos.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

