# 文件 API - FileApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `createFile`](#1-createfile)
  - [2. `presignGetUrl`](#2-presigngeturl)
  - [3. `deleteFile`](#3-deletefile)
- [接口一览](#接口一览)

## 由来

`FileApi` 定义在 **`com.lm.starter.module.infra.api.file`** 包下，由基础设施模块（`lingman-module-infra`）提供，作为文件存储的统一入口。业务工程引入 Starter 后，直接注入该接口即可完成文件的上传、预签名读取和删除操作，**无需**经过 Feign 与 `CommonResult` 包装。

该接口封装了底层文件存储（本地、S3、FTP 等）的差异，调用方只需关心文件内容和路径，不需要了解具体存储实现。

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.starter</groupId>
   	<artifactId>lingman-module-infra</artifactId>
   </dependency>
   ```

2. **在业务代码中注入并调用**：

   ```java
   import com.lm.starter.module.infra.api.file.FileApi;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;

   @Service
   public class YourBizServiceImpl implements YourBizService {

       @Resource
       private FileApi fileApi;

       /** 保存文件并返回访问路径 */
       public String saveFile(byte[] content, String name) {
           return fileApi.createFile(content, name);
       }

       /** 生成预签名读取 URL */
       public String getPresignedUrl(String fileUrl, int expireSeconds) {
           return fileApi.presignGetUrl(fileUrl, expireSeconds);
       }

       /** 按 ID 删除文件 */
       public void removeFile(Long fileId) {
           fileApi.deleteFile(fileId);
       }
   }
   ```

## 方法

### 1. `createFile`

**保存文件，并返回文件的访问路径**。提供三个重载：

```java
// 最简形式：仅传内容
default String createFile(byte[] content) {
    return createFile(content, null, null, null);
}

// 指定文件名
default String createFile(byte[] content, String name) {
    return createFile(content, name, null, null);
}

// 完整形式：内容、文件名、目录、MIME 类型
String createFile(@NotEmpty(message = "文件内容不能为空") byte[] content,
                  String name, String directory, String type);
```

- **content**：文件内容（`byte[]`，必填）
- **name**：文件名称（`String`，允许空）
- **directory**：目录（`String`，允许空）
- **type**：文件的 MIME 类型（`String`，允许空）
- **返回**：文件的访问路径（`String`）

### 2. `presignGetUrl`

**生成文件预签名地址，用于读取**。适用于需要对文件访问进行临时授权的场景。

```java
String presignGetUrl(@NotEmpty(message = "URL 不能为空") String url,
                     Integer expirationSeconds);
```

- **url**：完整的文件访问地址（`String`，必填）
- **expirationSeconds**：访问有效期，单位秒（`Integer`）
- **返回**：文件预签名地址（`String`）

### 3. `deleteFile`

**删除文件**。提供两个重载，按 ID 或按 URL 删除：

```java
// 通过文件 ID 删除
void deleteFile(Long id);

// 通过文件 URL 删除
void deleteFile(String url);
```

## 接口一览

```java
public interface FileApi {
    default String createFile(byte[] content) {
        return createFile(content, null, null, null);
    }

    default String createFile(byte[] content, String name) {
        return createFile(content, name, null, null);
    }

    String createFile(@NotEmpty(message = "文件内容不能为空") byte[] content,
                      String name, String directory, String type);

    String presignGetUrl(@NotEmpty(message = "URL 不能为空") String url,
                         Integer expirationSeconds);

    void deleteFile(Long id);

    void deleteFile(String url);
}
```
