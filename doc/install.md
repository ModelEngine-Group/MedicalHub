**Ruipath部署README**

**概述**

本文档用于指导模型引擎相关组件（datamate、模型、pathbot）的部署操作，包含前置检查、卸载清理及分步部署流程，操作前请确认具备对应权限。

下载地址:
通过网盘分享的文件：ruipath_25.zip
链接: https://pan.baidu.com/s/19DiNizM9TzmkGxWVKUCf7A?pwd=scww 提取码: scww 
--来自百度网盘超级会员v5的分享


里面包括ruipath推理镜像，datamate安装包，appengine安装包，pathobot安装包以及各种脚本
下载后放到master节点空间超过50G的位置，然后执行unzip ruipath_25.zip


**前置检查与卸载清理**

1.  查看model-engine命名空间下所有Pod状态，确认是否存在旧组件Pod：  
    kubectl get pod -n model-engine
2.  若存在datamate、pathbot相关Pod，找到对应安装包，执行卸载脚本：  
    bash uninstall.sh
3.  若存在ruipath-inference相关Pod，登录modelengine前端页面，在「模型服务」模块删除该服务。

**分步部署流程**

1.  **分发镜像**

解压后，找到脚本的位置，在master节点执行分发镜像脚本
bash distribute\_images.sh

流程:
执行过程中需要填写除了本节点以外的节点ip密码
选择镜像那里选3

**2\. 部署datamate**

1.  在master节点，执行datamate部署脚本：  
    bash deploy\_datamate.sh
2.  部署完成后，校验Pod运行状态：kubectl get pod -n model-engine
3.  确认datamate相关所有Pod均处于正常运行状态。正常的话应该是如下所示

![img_v3_0210r_fba73cdc-c2f5-4d8e-9353-f392c1743e6g](./insatll_images/image-001.jpeg)

安装好了之后运行cat /etc/haproxy/haproxy.cfg去找一下datamate前端ip。可以尝试访问

![img_v3_0210s_a66d7d7b-b671-4114-ad36-91b1838c493g](./insatll_images/image-002.jpeg)

例如我这个就是https://192.168.245.53:30000

**4\. 部署模型（ruipath）**

1.  导入密钥，执行

bash deploy-secret.sh -n model-engine

2.  登录modelengine前端页面，进入「模型服务」模块，创建模型服务：  
    

-   服务名称：ruipath
-   镜像地址：ruipath:910b
-   其余参数根据实际需求配置

参考示例:

![img_v3_0210r_8547fd5b-8d86-4d02-9630-d221db52b17g](./insatll_images/image-003.jpeg)

![img_v3_0210r_83448614-bcdf-4a67-9b53-d77b3161183g](./insatll_images/image-004.jpeg)

3.  执行模型部署脚本：  

    bash deploy\_model.sh -n model-engine -d ruipath-infernece -p 主节点ip

4.  查看所有Pod状态，确认模型相关Pod运行正常

运行cat /etc/haproxy/haproxy.cfg可以找到数据飞轮前端的ip。寻找类似这种名字的。

![img_v3_0210s_1dd9b82f-aa8d-4a66-9138-3694019fe33g](./insatll_images/image-005.jpeg)

例如我这个就是110.122.0.37:36666


**4\. 部署appengine**


1.  打开add_role_new.sh，修改文件14行和15行的内容，把命名空间和gaussdb的POD名字为自己环境的名字

2.  在master节点 cd AppEngine_opensource-1.3.4_Aarch64/tools

    bash install.sh --ns appengine --storage-class my-storage-class
    -- ns 命名空间 --storage-class 存储名字




**5\. 部署病理应用（pathbot）**

**前提条件**：appengine已完成部署

1.  在master节点 cd Pathobot_opensource-1.0.0-20260821111055_Aarch64/tools

    bash install.sh --ns appengine --storage-class my-storage-class
    -- ns 命名空间 --storage-class 存储名字

然后按照提示填写需要的内容，会需要镜像仓的密码

**注意事项**

-   所有部署操作需在具备对应权限的服务器上执行。
-   每步部署完成后，务必校验Pod状态，确保组件正常运行后再进行下一步。
-   模型部署时，镜像地址需严格填写ruipath:910b，避免拼写错误。