---
title: code-snippet
tags:
  - null
categories:
  - null
date: 2019-04-11 08:45:56
---

## spring mvc 文件上传下载
加入依赖jar

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

增加 spring 配置
```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="10248576"/><!--1M: 10248576-->
    <property name="maxInMemorySize" value="10248576"/>
</bean>
```
### 上传
```java
@PostMapping("/upload")
public String fileUpload(HttpServletRequest request) throws IOException {
    MultipartHttpServletRequest mRequest = (MultipartHttpServletRequest) request;

    String basePath = request.getServletContext().getRealPath("/upload/");
    // 判断文件目录是否存在
    File uploadFile = new File(basePath);
    if (!uploadFile.exists()) {
        uploadFile.mkdirs();
    }

    // 获得上传的文件
    Map<String, MultipartFile> fileMap = mRequest.getFileMap();
    for (Map.Entry<String, MultipartFile> fileEntry : fileMap.entrySet()) {
        String paramName = fileEntry.getKey();
        MultipartFile multipartFile = fileEntry.getValue();
        if (multipartFile == null || StringUtils.isBlank(multipartFile.getOriginalFilename())) {
            continue;
        }

        String filename = multipartFile.getOriginalFilename();

        // 上传成为压缩文件
        String zipName = basePath + File.separator + StringUtils.substringBeforeLast(filename, ".") + ".zip";
        ZipOutputStream outputStream = new ZipOutputStream(new FileOutputStream(zipName));
        outputStream.putNextEntry(new ZipEntry(filename));

        // 流拷贝, 内部会关闭输入和输出流
        org.springframework.util.FileCopyUtils.copy(multipartFile.getInputStream(), outputStream);
    }

    return "fileupload";
}
```

### 下载
```java
@GetMapping("/download")
@ResponseBody
public ResponseEntity<FileSystemResource> download() throws Exception {
    String filename = "报告单 - 副本.doc";
    File file = new File("D:\\workspace\\2019\\document\\management" + File.separator + filename);

    HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
    String header = request.getHeader("User-Agent").toUpperCase();
    HttpStatus status = HttpStatus.CREATED;

    //下载显示的文件名，解决中文名称乱码问题
    if (header.contains("MSIE") || header.contains("TRIDENT") || header.contains("EDGE")) {
        filename = URLEncoder.encode(filename, "UTF-8");
        filename = filename.replace("+", "%20");    // IE下载文件名空格变+号问题
        status = HttpStatus.OK;
    }
    else {
        filename = new String(filename.getBytes("UTF-8"), "ISO8859-1");
    }

    HttpHeaders headers = new HttpHeaders();
    headers.setContentLength(file.length());
    //通知浏览器以attachment（下载方式）打开
    headers.setContentDispositionFormData("attachment", filename);
    //application/octet-stream ： 二进制流数据（最常见的文件下载）。
    headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
    return new ResponseEntity<>(new FileSystemResource(file), headers, status);
}
```



### 获取当前类文件的绝对路径

```java
MyClass.class.getProtectionDomain().getCodeSource().getLocation().getPath()
```

### 获取当前jar包位置

```java
return new File(MyClass.class.getProtectionDomain().getCodeSource().getLocation()
    .toURI()).getPath();
```

### 数据库连接池大小多少合适

```
connections = ((core_count * 2) + effective_spindle_count)
```

这个公式[来自HikariCP](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing#the-formula)

其中，`core_count`是**CPU核心数**，注意：**核心数不包括超线程**， `effective_spindle_count` 的含义是**有效磁盘数**，如果你的服务器使用的是带有16个磁盘的RAID，那么`valid_spindle_count=16`。它实质上是服务器可以管理多少个并行I / O请求的度量。旋转硬盘一次（通常）一次只能处理一个I / O请求，如果你有16个，则系统可以同时处理16个I / O请求。

