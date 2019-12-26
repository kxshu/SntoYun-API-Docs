#  开放接口说明文档



##  Access Token获取

- 请求方式: `POST`

- 请求路径:`/auth/v1/token?appId=xxxxxx&appKey=yyyyyy`

- 请求参数说明:


  |  参数  |  类型  |       含义        |  
  | :----: | :----: | :---------------: |   
  | appId  | string |      应用Id       |  
  | appKey | string | 应用对应的API KEY |  


- 响应内容使用JSON格式,内容如下:

  ```json
  {
      "access_token":"1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656",
  }
  ```



## 人脸对比接口 

###  能力介绍 

#### 接口能力

- **两张人脸图片相似度对比**：比对两张图片中人脸的相似度，并返回相似度分值；

#### 业务应用

 用于比对多张图片中的人脸相似度并返回两两比对的得分，可用于判断两张脸是否是同一人的可能性大小。

典型应用场景：如**人证合一验证**，**用户认证**等，可与您现有的人脸库进行比对验证。



### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"



例如此接口，使用HTTPS POST发送：

```
https://aiapi.snto.com/face/v1/match?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



### 请求说明

**注意事项**

- **请求体格式化**：Content-Type为`application/json`，通过`json`格式化请求体。
- **Base64编码**：请求的图片需经过`Base64编码`，图片的base64编码指将图片数据编码成一串字符串，使用该字符串代替图像地址。您可以首先得到图片的二进制，然后用Base64格式编码即可。需要注意的是，图片的base64编码是不包含图片头的，如`data:image/jpg;base64,`
- **图片格式**：现支持PNG、JPG、JPEG、BMP，**不支持GIF图片**



**请求示例**

`http方法`:`POST`

`请求url`:  `https://aiapi.snto.com/face/v1/match`

`url参数`:

|     参数     |                              值                              |
| :----------: | :----------------------------------------------------------: |
| access_token | 可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**" |



`Header`:

|     参数     |        值        |
| :----------: | :--------------: |
| Content-Type | application/json |



Body中放置请求参数，参数详情如下：

**请求参数**

|      参数       | 必选 |  类型  |                             说明                             |
| :-------------: | :--: | :----: | :----------------------------------------------------------: |
|    img_data     |  是  | string | 图片信息(**总数据大小应小于10M**)，图片上传方式根据img_type来判断。 **两张图片通过json格式上传，格式参考表格下方示例** |
|    img_type     |  是  | string | 图片类型<br/>**BASE64**:图片的base64值，base64编码后的图片数据，编码后的图片大小不超过2M |
**请求代码示例**

```json
curl -i -k 'https://cloud.snto.com/face/v1/match?access_token=[Access Token获取到的token]' --data '[{"img_data":"sfasq35sadvsvqwr5q...","img_type":"BASE64"},{"img_data":"sfasq35sadvsvqwr5q...","img_type":"BASE64"}]'
```



### 返回说明

**返回参数**

| 参数名     | 必选 |  类型  |               说明                |
| :--------- | :--: | :----: | :-------------------------------: |
| sim        |  是  | float  | 人脸相似度值,推荐相似度值超过0.76 |
| face_list  |  是  | array  |            人脸信息表             |
| +face_info |  是  | object |           人脸rect信息            |

**返回示例**

```json
{
    "sim": 0.864,
    "face_list": [
      {
        "face": {
          "y": 16,             //人脸左上角顶点的y轴坐标
          "x": 1,              //人脸左上角顶点的x轴坐标
          "height": 158,       //人脸区域的高度
          "width": 177         //人脸区域的宽度
        }
      },
      {
        "face": {
          "y": 21,
          "x": 45,
          "height": 173,
          "width": 116
        }
      }
    ]
  }
```



##  人脸底库管理

**人脸库结构**

人脸库、用户组、用户、用户下的人脸**层级关系**如下所示：

```
|- 人脸库(appid)
   |- 用户组一（groupId）
      |- 用户01（uid）
         |- 人脸（faceid）
      |- 用户02（uid）
         |- 人脸（faceid）
         |- 人脸（faceid）
         ....
       ....
   |- 用户组二（groupId）
   |- 用户组三（groupId）
   ....
```



**关于人脸库的设置限制**

- 每个开发者账号可以创建100个appid；
- **每个appid对应一个人脸库，且不同appid之间，人脸库互不相通**；
- 每个人脸库下，可以创建多个用户组，用户组（group）数量**没有限制**；
- 每个用户组（group）下，可添加**无限**个user_id，**无限**张人脸（注：为了保证查询速度，单个group中的人脸容量上限建议为**80万**）；
- 每个用户（user_id）所能注册的最大人脸数量**5**；

`提醒：每个人脸库对应一个appid，一定确保不要轻易删除后台应用列表中的appid，删除后则此人脸库将失效，无法进行任何查找！`

---



### 用户组创建

#### 接口描述

用于创建一个空的用户组，如果用户组已存在 则返回错误。

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/group_add?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`http方法`:`POST`

`请求url`:  `https://aiapi.snto.com/face/v1/faceset/group_add`

`url参数`:

|     参数     |                              值                              |
| :----------: | :----------------------------------------------------------: |
| access_token | 可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**" |

`Header`:

|     参数     |        值        |
| :----------: | :--------------: |
| Content-Type | application/json |



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数 | 类型   | 必选 | 说明                                                         |
| ---- | ------ | ---- | ------------------------------------------------------------ |
| gid  | string | 是   | 用户组id，标识一组用户（由数字、字母、下划线组成），长度限制128B |

**示例代码**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/add?access_token=[Access Token获取到的token]' --data '{"gid":"12312424"}'
```



**返回说明**

通过返回的code判断操作情况,失败则可以通过errmsg查看具体的错误消息

```json
{
    "data":{},
    "code":200 //删除成功
}
```

```json
{
    "data":{},
    "code":400,
    "errmsg":"用户组已经存在"
}
```

---



### 用户组删除

#### 接口描述

删除用户组下所有的用户及人脸，如果组不存在 则返回错误。

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/group_del?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`http方法`:`POST`

`请求url`:  `https://aiapi.snto.com/face/v1/faceset/group_del`

`url参数`:

|     参数     |                              值                              |
| :----------: | :----------------------------------------------------------: |
| access_token | 可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**" |

`Header`:

|     参数     |        值        |
| :----------: | :--------------: |
| Content-Type | application/json |



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数 | 类型   | 必选 | 说明                                                         |
| ---- | ------ | ---- | ------------------------------------------------------------ |
| gid  | string | 是   | 用户组id，标识一组用户（由数字、字母、下划线组成），长度限制128B |

**示例代码**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/group_del?access_token=[Access Token获取到的token]' --data '{"gid":"12312424"}'
```



**返回说明**

通过返回的code判断操作情况,失败则可以通过errmsg查看具体的错误消息

```json
{
    "data":{},
    "code":200 //删除成功
}
```

```json
{
    "data":{},
    "code":400,
    "errmsg":"用户组不存在"
}
```

---

### 用户组信息查询

#### 接口描述

用于查询应用下的组列表信息

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/group_list?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`http方法`:`POST`

`请求url`:  `https://aiapi.snto.com/face/v1/faceset/group_list`

`url参数`:

|     参数     |                              值                              |
| :----------: | :----------------------------------------------------------: |
| access_token | 可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**" |

`Header`:

|     参数     |        值        |
| :----------: | :--------------: |
| Content-Type | application/json |

Body中放置请求参数，参数详情如下：

**请求参数**

| 参数  | 类型  | 必选 | 说明                           |
| ----- | ----- | ---- | ------------------------------ |
| start | int32 | 否   | 默认值0，起始序号              |
| count | int32 | 否   | 返回值数量,默认为100,最大值500 |

**示例代码**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/group_list?access_token=[Access Token获取到的token]' --data '{"start":100}'
```



**返回说明**

| 参数       | 类型  | 必选 | 说明       |
| ---------- | ----- | ---- | ---------- |
| group_list | array | 是   | 组列表信息 |

- 返回示例

  ```json
  {
      "group_list": [
          "test1",
          "test2"
      ]
  }
  ```

---



###  创建用户

#### 接口描述

用于向人脸库中新增用户，及组内用户的人脸图片，

典型应用场景：构建您的人脸库，如**会员人脸注册**等。

####  调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/create_user?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`http方法`:`POST`

`请求url`:  `https://aiapi.snto.com/face/v1/faceset/create_user`

`url参数`:

|     参数     |                              值                              |
| :----------: | :----------------------------------------------------------: |
| access_token | 可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**" |

`Header`:

|     参数     |        值        |
| :----------: | :--------------: |
| Content-Type | application/json |



Body中放置请求参数，参数详情如下：

**请求参数**

|   参数   | 必选 |  类型  |                             说明                             |
| :------: | :--: | :----: | :----------------------------------------------------------: |
| img_data |  是  | string | 图片信息(**总数据大小应小于10M**)，图片上传方式根据image_type来判断。<br/>注：组内每个uid下的人脸图片数目上限为5张 |
| img_type |  是  | string | 图片类型<br/>**BASE64**:图片的base64值，base64编码后的图片数据； |
|   gid    |  是  | string | 用户组id，标识一组用户（由数字、字母、下划线组成），长度限制128B。**产品建议**：根据您的业务需求，可以将需要注册的用户，按照业务划分，分配到不同的group下，例如按照会员手机尾号作为groupid，用于刷脸支付、会员计费消费等，这样可以尽可能控制每个group下的用户数与人脸数，提升检索的准确率 |
|   uid    |  是  | string |       用户id（由数字、字母、下划线组成），长度限制128B       |

```
说明：人脸注册完毕后，生效时间一般为5s以内，之后便可以进行人脸搜索或认证操作
```

**请求代码示例**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/create_user?access_token=[Access Token获取到的token]' --data '{"img_data":"sfasq35sadvsvqwr5q...","img_type":"BASE64","gid":"test_group","uid":"test_user"}'
```



**返回说明**

| 字段     | 必选 |  类型  |          说明           |
| :------- | :--: | :----: | :---------------------: |
| face_id  |  是  | string |    人脸图片唯一标识     |
| location |  是  | array  |   人脸在图片中的位置    |
| + x      |  是  |  int   | 人脸左上角顶点的x轴坐标 |
| + y      |  是  |  int   | 人脸左上角顶点的y轴坐标 |
| + height |  是  |  int   |     人脸区域的高度      |
| + width  |  是  |  int   |     人脸区域的宽度      |

**返回示例**

```json
{
  "face_id": "2fa64a88a9d5118916f9a303782a97d3",
  "location": {
      "x": 117,
      "y": 131,
      "width": 172,
      "height": 170
  }
}
```

---



### 人脸底库添加

#### 接口描述

用于**补全用户人脸库信息**,添加用户底库

#### 请求说明

**注意事项**：

- **请求体格式化**：Content-Type为`application/json`，通过`json`格式化请求体。
- **Base64编码**：请求的图片需经过`Base64编码`，图片的base64编码指将图片数据编码成一串字符串，使用该字符串代替图像地址。您可以首先得到图片的二进制，然后用Base64格式编码即可。需要注意的是，图片的base64编码是不包含图片头的，如`data:image/jpg;base64,`
- **图片格式**：现支持PNG、JPG、JPEG、BMP，**不支持GIF图片**

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/face_add?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/faceset/face_add`



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数     |  类型  | 是否必须 | 说明                                                         |
| -------- | :----: | :------: | ------------------------------------------------------------ |
| img_data | string |    是    | 图片信息(**总数据大小应小于10M**)，图片上传方式根据image_type来判断<br>注：组内每个uid下的人脸图片数目上限为5张 |
| img_type | string |    是    | 图片类型<br/>**BASE64**:图片的base64值，base64编码后的图片数据 |
| gid      | string |    是    | 用户组id，标识一组用户（由数字、字母、下划线组成），长度限制128B |
| uid      | string |    是    | 用户id（由数字、字母、下划线组成），长度限制128B             |



**请求代码示例**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/face_add?access_token=[Access Token获取到的token]' --data '{"img_data":"sfasq35sadvsvqwr5q...","img_type":"BASE64","gid":"test_group","uid":"test_user"}'
```



**返回说明**

| 字段     | 必选 |  类型  |          说明           |
| :------- | :--: | :----: | :---------------------: |
| face_id  |  是  | string |   人脸图片唯一标识id    |
| location |  是  | array  |   人脸在图片中的位置    |
| + x      |  是  |  int   | 人脸左上角顶点的x轴坐标 |
| + y      |  是  |  int   | 人脸左上角顶点的y轴坐标 |
| + height |  是  |  int   |     人脸区域的高度      |
| + width  |  是  |  int   |     人脸区域的宽度      |

**返回示例**

```json
{
  "face_id": "2fa64a88a9d5118916f9a303782a97d3",
  "location": {
      "x": 117,
      "y": 131,
      "width": 172,
      "height": 170
  }
}
```

---



### 人脸底库删除

#### 接口描述

删除用户的某一张人脸，如果该用户只有一张人脸图片，则同时删除用户。

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/face_del?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/faceset/face_del`



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数    |  类型  | 是否必须 | 说明                                                         |
| ------- | :----: | :------: | ------------------------------------------------------------ |
| gid     | string |    是    | 用户组id，标识一组用户（由数字、字母、下划线组成），长度限制128B |
| uid     | string |    是    | 用户id（由数字、字母、下划线组成），长度限制128B             |
| face_id | string |    是    | 需要删除的人脸图片id                                         |

**请求代码示例**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/face_del?access_token=[Access Token获取到的token]' --data '{"gid":"test_group","uid":"test_user"，"face_id":"ebe86f9194f7e8038715f8d37fe18641"}'
```



**返回说明**

通过返回的code判断操作情况,失败则可以通过errmsg查看具体的错误消息

```json
{
    "data":{},
    "code":200 //删除成功
}
```

```json
{
    "data":{},
    "code":400,
    "errmsg":"用户不存在"
}
```

---



###  用户信息查询

#### 接口能力

获取人脸库中某个用户的信息(信息和用户所属的组)。

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/user_info?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656

```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/faceset/user_info`



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数 |  类型  | 是否必须 | 说明                                                         |
| ---- | :----: | :------: | ------------------------------------------------------------ |
| gid  | string |    是    | 用户组id，标识一组用户（由数字、字母、下划线组成），长度限制128B |
| uid  | string |    是    | 用户id，标识一组用户（由数字、字母、下划线组成），长度限制128B |

**请求代码示例**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/user_info?access_token=[Access Token获取到的token]' --data '{"gid":"test_group","uid":"12345"}'

```



**返回说明**

| 参数       | 类型  | 必选 | 说明             |
| ---------- | ----- | ---- | ---------------- |
| group_list | array | 是   | 用户所属的组列表 |

- 返回示例

  ```json
  {
      "user_list": {
          "gid1",
          "gid2"
      }
  }
  ```

___



### 人脸信息查询

#### 接口能力

用于获取一个用户的全部人脸列表。

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/face_info?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/faceset/face_info`



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数 |  类型  | 是否必须 | 说明                                                         |
| ---- | :----: | :------: | ------------------------------------------------------------ |
| gid  | string |    是    | 用户组id，标识一组用户（由数字、字母、下划线组成），长度限制128B |
| uid  | string |    是    | 用户id（由数字、字母、下划线组成），长度限制128B             |

**请求代码示例**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/face_info?access_token=[Access Token获取到的token]' --data '{"gid":"test_group","uid":"test_user"}'
```



**返回说明**

| 参数      | 类型   | 必选 | 说明     |
| --------- | ------ | ---- | -------- |
| face_list | array  | 是   | 人脸列表 |
| + face_id | string | 是   | 人脸ID   |

- 返回示例

  ```json
  {
    "face_list": [
        {
            "face_id": "fid1"
        },
        {
            "face_id": "fid2"
        }
    ]
  }
  ```

---



### 获取用户列表

#### 接口能力

用于查询指定用户组中的用户列表。

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/user_list?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/faceset/user_list`



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数  |  类型  | 是否必须 | 说明                                                         |
| ----- | :----: | :------: | ------------------------------------------------------------ |
| gid   | string |    是    | 用户组id，标识一组用户（由数字、字母、下划线组成），长度限制128B |
| start | uint32 |    否    | 默认值0，起始序号                                            |
| count | uint32 |    否    | 返回值数量,默认为100,最大值500                               |

**请求代码示例**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/user_list?access_token=[Access Token获取到的token]' --data '{"gid":"test_group"}'
```



**返回说明**

| 参数      | 类型  | 必选 | 说明     |
| --------- | ----- | ---- | -------- |
| user_list | array | 是   | 用户列表 |

- 返回示例

  ```json
  {
      "user_list": [
          "uid1",
          "uid2"
      ]
  }
  ```



---



###  用户移动

#### 接口能力

用于将已经存在于人脸库中的用户**复制或者移动到一个新的组**。新的用户信息将覆盖目标组同一用户的信息

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/move_user?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656

```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/faceset/move_user`



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数    | 类型   | 必选 | 说明                     |
| ------- | ------ | ---- | ------------------------ |
| action  | 0 或 1 | 是   | 0:表示移动， 1:表示复制  |
| uid     | string | 是   | 移动的用户id             |
| src_gid | string | 是   | 从指定的用户组里移动用户 |
| dst_gid | string | 是   | 移动到指定的用户组中     |

**请求代码示例**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/move_user?access_token=[Access Token获取到的token]' --data '{"gid":"test_group"}'

```



**返回说明**

通过返回的code判断操作情况,失败则可以通过errmsg查看具体的错误消息

```json
{
    "code":"200",
    "data":{},
    "errmsg":""
}
```

---



### 用户删除

#### 接口能力

用于将用户从某个组中删除。

#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/faceset/delete_user?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656

```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/faceset/delete_user`



Body中放置请求参数，参数详情如下：

**请求参数**

| 参数 | 类型   | 必选 | 说明                                                 |
| ---- | ------ | ---- | ---------------------------------------------------- |
| gid  | string | 是   | 用户组ID,当值为**@ALL**时,表示删除所用组线面的该用户 |
| uid  | string | 是   | 用户ID                                               |

**请求代码示例**

```
curl -i -k 'https://cloud.snto.com/face/v1/faceset/delete_user?access_token=[Access Token获取到的token]' --data '{"gid":"test_group"，"uid":"test_user"}'

```



---

## 人脸搜索

#### 接口能力

也称为1：N识别，在指定人脸集合中，找到最相似的人脸

**提示:**进行人脸查找相关操作前，建议先阅读**人脸库管理**相关内容



#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/search?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/search`



Body中放置请求参数，参数详情如下：

| 参数         | 类型   | 必选 | 说明                                                         |
| ------------ | ------ | ---- | ------------------------------------------------------------ |
| img_data     | string | 是   | 图片信息(**总数据大小应小于10M**)，图片上传方式根据image_type来判断 |
| img_type     | string | 是   | 图片类型<br/>**BASE64**:图片的base64值，base64编码后的图片数据，编码后的图片大小不超过2M |
| gid_list     | array  | 是   | 从指定的group中进行查找 ,上限**10个**                        |
| uid          | string | 否   | 当需要对特定用户进行比对时，指定uid进行比对。即人脸认证功能。 |
| sim_thresold | fload  | 否   | 相似度阈值,默认设置为0.77,如果设置过低,容易造成误匹配        |

`说明:该接口返回第一个相似度超过阈值的用户信息`



**请求示例代码**

```
curl -i -k 'https://cloud.snto.com/face/v1/search?access_token=[Access Token获取到的token]' --data '{"img_data":"sfasq35sadvsvqwr5q...","img_type":"BASE64","gid":["test_group"],"uid":"test_user"}'
```



**返回说明**

|   字段    |  类型  | 必选 |           说明           |
| :-------: | :----: | :--: | :----------------------: |
| user_list | array  |  是  |     匹配到的用户列表     |
|    gid    | string |  是  |       用户所属组Id       |
|    uid    | string |  是  |          用户id          |
|    iid    | string |  是  | 用户底库下面具体的图片id |
|    sim    | float  |  是  |         相似度值         |

**返回示例**

```json
{
    "user_list":[
        {
            "gid":"test",
            "uid":"test",
            "iid":"2fa64a88a9d5118916f9a303782a97d3",
            "sim":"0.882"
        }
    ]
}
```



## 人脸M:N搜索

#### 接口能力

M：N识别的原理，相当于在多个人脸的图片中，先分别找出所有人脸，然后分别在待查找的人脸集合中，分别做1：N识别，最后将识别结果汇总在一起进行返回。



#### 调用方式

**请求URL数据格式**

向API服务地址使用POST发送请求，必须在URL中带上参数`access_token`，可通过后台的App Key和App Id生成，具体方式请参考"**Access Token获取**"

例如此接口，使用HTTPS POST发送：

```
https://cloud.snto.com/face/v1/multi_search?access_token=1729.b6cbbbd30c0f08904507135f149e8cd2.1572329694.7997656
```



**请求示例**

`HTTP请求方法`:`POST`

`请求URL`: `https://aiapi.snto.com/face/v1/multi_search`



Body中放置请求参数，参数详情如下：

| 参数         | 类型   | 必选 | 说明                                                         |
| ------------ | ------ | ---- | ------------------------------------------------------------ |
| img_data     | string | 是   | 图片信息(**总数据大小应小于10M**)，图片上传方式根据image_type来判断 |
| img_type     | string | 是   | 图片类型<br/>**BASE64**:图片的base64值，base64编码后的图片数据，编码后的图片大小不超过2M |
| gid_list     | array  | 是   | 从指定的group中进行查找 ,上限**10个**                        |
| sim_thresold | fload  | 否   | 相似度阈值,默认设置为0.77,如果设置过低,容易造成误匹配        |

`说明:该接口返回第一个相似度超过阈值的用户信息`



**请求示例代码**

```
curl -i -k 'https://cloud.snto.com/face/v1/search?access_token=[Access Token获取到的token]' --data '{"img_data":"sfasq35sadvsvqwr5q...","img_type":"BASE64","gid":["test_group"]}'
```



**返回说明**

|   字段    |  类型  | 必选 |           说明           |
| :-------: | :----: | :--: | :----------------------: |
| face_num  |  int   |  是  |         face_num         |
| face_list | arrat  |  是  |                          |
| user_list | array  |  是  |     匹配到的用户列表     |
|    gid    | string |  是  |       用户所属组Id       |
|    iid    | string |  是  | 用户底库下面具体的图片id |
|    sim    | float  |  是  |         相似度值         |

**返回示例**

```json
{
    "face_list":[
        {
            "face_id":"xadfadfd",
            "location":{
                "x": 31.95568085,
                "y": 120.3764267,
                "width": 87,
                "height": 85,
                "rotation": -5
            },
            "user_list":[
                {
                    "gid":"test",
                    "uid":"test",
                    "iid":"2fa64a88a9d5118916f9a303782a97d3",
                    "sim":"0.882"
                }
            ]
        }        
    ]
}
```







