# 社交应用 API - SocialClientApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `getAuthorizeUrl`](#1-getauthorizeurl)
  - [2. `createWxMpJsapiSignature`](#2-createwxmpjsapisignature)
  - [3. `getWxMaPhoneNumberInfo`](#3-getwxmaphonenumberinfo)
  - [4. `getWxaQrcode`](#4-getwxaqrcode)
  - [5. `getWxaSubscribeTemplateList`](#5-getwxasubscribetemplatelist)
  - [6. `sendWxaSubscribeMessage`](#6-sendwxasubscribemessage)
  - [7. `uploadWxaOrderShippingInfo`](#7-uploadwxaordershippinginfo)
  - [8. `notifyWxaOrderConfirmReceive`](#8-notifywxaorderconfirmreceive)
- [参数 / 返回说明](#参数--返回说明)
  - [SocialWxJsapiSignatureRespDTO](#socialwxjsapisignaturerespdto)
  - [SocialWxPhoneNumberInfoRespDTO](#socialwxphonenumberinforespdto)
  - [SocialWxQrcodeReqDTO](#socialwxqrcodereqdto)
  - [SocialWxaSubscribeMessageSendReqDTO](#socialwxasubscribemessagesendreqdto)
  - [SocialWxaSubscribeTemplateRespDTO](#socialwxasubscribetemplaterespdto)
  - [SocialWxaOrderUploadShippingInfoReqDTO](#socialwxaorderuploadshippinginforeqdto)
  - [SocialWxaOrderNotifyConfirmReceiveReqDTO](#socialwxaordernotifyconfirmreceivereqdto)
- [接口一览](#接口一览)

## 由来

`SocialClientApi` 定义在 **`com.lm.starter.module.system.api.social`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可完成社交平台（以微信生态为主）的授权、JS-SDK 签名、小程序能力调用等，**无需**经过 Feign 与 `CommonResult` 包装。

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，本接口方法**直接返回**所需的数据结构（例如 `String`、DTO、`byte[]`、`List<DTO>`），或在成功时返回 `void`；失败时以异常表达（异常类型/错误码以 Starter 实现为准）。入参上标注的 `@Valid` 会触发 `jakarta.validation` 校验。

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.starter</groupId>
   	<artifactId>lingman-module-system</artifactId>
   </dependency>
   ```

2. **在业务代码中注入并调用**（示例：获取授权 URL + 获取 JSAPI 签名）：

   ```java
   import com.lm.starter.module.system.api.social.SocialClientApi;
   import com.lm.starter.module.system.api.social.dto.SocialWxJsapiSignatureRespDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private SocialClientApi socialClientApi;
   
       public String buildAuthorizeUrl(Integer socialType, Integer userType, String redirectUri) {
           return socialClientApi.getAuthorizeUrl(socialType, userType, redirectUri);
       }
   
       public SocialWxJsapiSignatureRespDTO jsapiSignature(Integer userType, String url) {
           return socialClientApi.createWxMpJsapiSignature(userType, url);
       }
   }
   ```

## 方法

### 1. `getAuthorizeUrl`

**获得社交平台的授权 URL**。

```java
String getAuthorizeUrl(Integer socialType, Integer userType, String redirectUri);
```

### 2. `createWxMpJsapiSignature`

**创建微信公众号 JS SDK 初始化所需的签名**。

```java
SocialWxJsapiSignatureRespDTO createWxMpJsapiSignature(Integer userType, String url);
```

### 3. `getWxMaPhoneNumberInfo`

**获得微信小程序的手机信息**（基于手机授权码 `phoneCode`）。

```java
SocialWxPhoneNumberInfoRespDTO getWxMaPhoneNumberInfo(Integer userType, String phoneCode);
```

### 4. `getWxaQrcode`

**获得小程序二维码**，返回二维码图片的二进制内容（`byte[]`）。

```java
byte[] getWxaQrcode(@Valid SocialWxQrcodeReqDTO reqVO);
```

### 5. `getWxaSubscribeTemplateList`

**获得微信小程序订阅模板列表**。

```java
List<SocialWxaSubscribeTemplateRespDTO> getWxaSubscribeTemplateList(Integer userType);
```

### 6. `sendWxaSubscribeMessage`

**发送微信小程序订阅消息**。

```java
void sendWxaSubscribeMessage(SocialWxaSubscribeMessageSendReqDTO reqDTO);
```

### 7. `uploadWxaOrderShippingInfo`

**上传订单发货到微信小程序**。

```java
void uploadWxaOrderShippingInfo(Integer userType, SocialWxaOrderUploadShippingInfoReqDTO reqDTO);
```

### 8. `notifyWxaOrderConfirmReceive`

**通知订单收货到微信小程序**。

```java
void notifyWxaOrderConfirmReceive(Integer userType, SocialWxaOrderNotifyConfirmReceiveReqDTO reqDTO);
```

## 参数 / 返回说明

### SocialWxJsapiSignatureRespDTO

- **appId**：公众号 appId
- **nonceStr**：随机串
- **timestamp**：时间戳
- **url**：参与签名的 URL
- **signature**：签名

### SocialWxPhoneNumberInfoRespDTO

- **phoneNumber**：带区号手机号
- **purePhoneNumber**：不带区号手机号
- **countryCode**：区号

### SocialWxQrcodeReqDTO

- **scene**：场景值（必填）
- **path**：页面路径（必填；不带参数，参数请放在 `scene`）
- **width**：二维码宽度（可选）
- **autoColor**：自动配置线条颜色（可选）
- **checkPath**：是否检查 page 是否存在（可选）
- **hyaline**：是否需要透明底色（可选）

### SocialWxaSubscribeMessageSendReqDTO

- **userId**：用户编号（必填）
- **userType**：用户类型（必填）
- **templateTitle**：订阅消息模板标题（必填）
- **page**：点击卡片跳转页（可选；本小程序内页面）
- **messages**：模板内容参数（可选，`Map<String, String>`）

### SocialWxaSubscribeTemplateRespDTO

- **id**：模板编号
- **title**：模板标题
- **content**：模板内容
- **example**：模板内容示例
- **type**：模板类型（2：一次性订阅；3：长期订阅）

### SocialWxaOrderUploadShippingInfoReqDTO

- **openid**：支付者 openid（必填）
- **transactionId**：原支付交易对应的微信订单号（必填）
- **logisticsType**：物流模式（必填）
- **logisticsNo**：物流发货单号（可选）
- **expressCompany**：物流公司编号（可选）
- **itemDesc**：商品信息描述（必填）
- **receiverContact**：收件人手机号（必填）

### SocialWxaOrderNotifyConfirmReceiveReqDTO

- **transactionId**：原支付交易对应的微信订单号（必填）
- **receivedTime**：快递签收时间（必填）

## 接口一览

```java
public interface SocialClientApi {
    String getAuthorizeUrl(Integer socialType, Integer userType, String redirectUri);

    SocialWxJsapiSignatureRespDTO createWxMpJsapiSignature(Integer userType, String url);

    SocialWxPhoneNumberInfoRespDTO getWxMaPhoneNumberInfo(Integer userType, String phoneCode);

    byte[] getWxaQrcode(@Valid SocialWxQrcodeReqDTO reqVO);

    List<SocialWxaSubscribeTemplateRespDTO> getWxaSubscribeTemplateList(Integer userType);

    void sendWxaSubscribeMessage(SocialWxaSubscribeMessageSendReqDTO reqDTO);

    void uploadWxaOrderShippingInfo(Integer userType, SocialWxaOrderUploadShippingInfoReqDTO reqDTO);

    void notifyWxaOrderConfirmReceive(Integer userType, SocialWxaOrderNotifyConfirmReceiveReqDTO reqDTO);
}
```