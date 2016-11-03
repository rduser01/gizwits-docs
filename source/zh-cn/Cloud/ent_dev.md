
title: 企业应用开发
---

h1 概述

 企业应用开发是指企业将设备、APP等客户端接入机智云后，企业根据接入设备的行业特点构建云端业务管理系统以实现自己的管理需要以及商业模式。

 企业应用需要使用机智云云平台的两个核心服务达成目标。分别是实时消息代理（Noti服务）以及企业API。

![Alt text](./1478078696133.png)

h2 消息代理
 机智云平台是开放、中立的平台，消息代理服务可以实时将企业设备数据推送到企业应用系统，企业应用系统基于设备数据实现业务需求。
  
h2 企业API
企业API是从接入机智云平台的企业的运营管理的需求出发，为企业应用系统提供REST API接口服务。为企业提供企业视角全局的设备管理、数据分析等功能，让企业更关注业务管理系统本身，减少不必要的开发成本与时间

企业API基于企业接入到机智云平台所产生的数据，为企业提供了通用的数据计算能力，如设备数据点聚合查询、设备历史数据查询、设备位置服务、设备激活、活跃数据等。企业API的定位是减少企业业务管理系统的开发成本，让企业更关注自己的核心业务


#消息代理



h1  企业API
h2  接入场景与业务流程
h3  设备远程控制
2.1.1场景描述
    该业务场景是指企业通过业务管理系统去控制接入到机智云平台的设备，管理设备的主要需求就是在业务管理系统中对设备发起控制。 
2.1.2接入流程



流程说明：
1、首先企业需要申请eid,esecret参数，企业也可以在开发者中心自行获取,也可像机智云人工申请；
2、设置IP白名单：由企业登录开发者中心自行设置；
3、关联产品：企业的业务系统是管理某个产品下的设备，那么操作之前需要先对Eid与PK进行关联，只有关联某个产品后，才能访问那个产品的相关权限，如控制设备，查询某个产品的相关数据；
4、获取token：相当于登陆鉴权，鉴权通过返回服务端生成的token，作为后续API的输入;
5、发起设备远程控制，如果无法获取设备的did，则调用“设备信息查询”接口获得did；如果通过其他方式获得did，可忽略此步骤。(使用数据实时同步服务实时获取设备数据的可忽略此步骤，业务管理系统一般在收到每条设备上报的数据，会实时将最新的Did进行更新，所以在发起远程控制的时候，无需再次获取Did）
6、如果产品定义了数据点协议，则可以使用“E04数据点方式远程控制”去向设备发起控制；如果是数据透传协议，则通过“E05原始指令方式远程控制”控制。



##协议约定
4.1协议规格描述
关于协议格式的约定：
1、请求方式：本文档所定义接口基于HTTPS(后续服务端将会强制使用HTTPS)协议进行传输，需要注意协议中标注的请求方式，通过GET、PUT、DELETE等进行不同的操作。
2、接口地址：接口地址中当用<>号包含的为变量，需要使用者通过对应的变量替换。例如<product_key>，表示需要将获取的product_key值赋予到接口地址中，<did>表示是需要替换为具体的did
3、请求参数：HTTP请求参数的类型一般分为三种。header表示该参数是在HTTP请求头中；URL表示是通过url传参；BODY表示是Request Body，通常Body中都是Json格式
4.2消息头部
4.2.1HTTP Request Header
本文档协议中设备管理类、设备报表查询类的接口在进行接口访问时，都需要在请求头增加token值，以此校验访问者是否有权访问该接口。token值是通过获取授权接口获得。

请求头格式如下：
Content-Type: application/json 
Authorization: token <token值>,<token值>前面必须加"token "，For Example:
Authorization: token xxxxxxxxxxx
4.2.2HTTP Response Header
本文档协议中的接口在返回报文头部会输出如下信息：
X-RateLimit-Limit: 60     //接口允许访问总量
X-RateLimit-Remaining: 56 //接口剩余访问次数
X-RateLimit-Reset: 1372700873 //调用频率限制重置时间，TS类型
4.2.3Token值的生命周期
Token值有效期为7天， 调用获取token接口返回的expired_at为失效日期时间戳。若现在时间戳 > expired_at时间戳，则需要重新获取token, token调用请见:3.2 E02获取Token

4.2.4企业调用配额说明
企业eid每小时默认允许企业API接口每小时调用5600次，超过阈值则抛错，1小时后恢复接口调用。具体抛错请查看5.2错误信息表系统编码5013的相关说明

##接口协议
3.1E01关联产品
3.1.1业务功能描述
该接口提供创建企业与产品的关联关系能力，一个 product_key 只能关联到一个 enterprise_id，一个enterprise_id可以关联多个product_key。
3.1.2接口地址
    http://enterpriseapi.gizwits.com/v1/products/<product_key>/association
3.1.3请求方式
    POST
3.1.4请求报文
参数	类型	必填	参数类型	描述
{
  "enterprise_id": "ad7e60f0594247dcba017ba76d2f4275",
  "enterprise_secret": "db2ac98200824181b447b03e4d42f99b",
  "product_secret": "8f11ee69eb9d4269ba0777ca5e7280f5"
}
	JSON	是	BODY	Product secret：product key对应的唯一秘钥，在开发者中心产品信息中获取

3.1.5应答报文
响应编码	返回内容	说明
201	{}	 关联成功