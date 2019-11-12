## 空项目目录cmd> npm create umi

选择 ant-design-pro：

Ant Design Pro 脚手架将会自动安装


```

├── config                   # umi 配置，包含路由，构建等配置
├── mock                     # 本地模拟数据
├── public
│   └── favicon.png          # Favicon
├── src
│   ├── assets               # 本地静态资源
│   ├── components           # 业务通用组件
│   ├── e2e                  # 集成测试用例
│   ├── layouts              # 通用布局
│   ├── models               # 全局 dva model
│   ├── pages                # 业务页面入口和常用模板
│   ├── services             # 后台接口服务
│   ├── utils                # 工具库
│   ├── locales              # 国际化资源
│   ├── global.less          # 全局样式
│   └── global.ts            # 全局 JS
├── tests                    # 测试工具
├── README.md
└── package.json

````


## 本地开发

### 安装依赖

npm install
如果网络状况不佳，可以使用 tyarn 进行加速。

npm start

### 构建
npm run build

### 官网地址
https://pro.ant.design/docs/build-cn


## 1、gitlab-runner 配置

### 1.1 进入runner 
kubectl exec -it runner-rc-dvnwb /bin/sh -n kube-system
### 1.2 配置执行器

```
gitlab-ci-multi-runner register -n \
  --url http://192.168.1.4:8888/ \
  --registration-token JLxZRyT9v5U49ZoeVa21 \
  --tag-list=ant-pro-runner \
  --description "share pre runner" \
  --kubernetes-namespace="kube-system" \
  --kubernetes-pull-policy="if-not-present" \
  --kubernetes-image "node:10.17.0-alpine" \
  --executor kubernetes
```

### 1.3 挂载缓存路径

cd /etc/gitlab-runner
```
cat >> config.toml << EOF 
      [[runners.kubernetes.volumes.host_path]]
        name = "npm-cache"
        mount_path = "/root/.npm"
        host_path = "/root/.npm"
      [[runners.kubernetes.volumes.host_path]]
        name = "node-modules-cache"
        mount_path = "/root/modules"
        host_path = "/root/modules"
      [[runners.kubernetes.volumes.pvc]]
        name = "nginx"
        mount_path = "/root/html"

```
说明：

* npm-cache 用来缓存全局的npm 缓存
* node-modules-cache 用来避免每次没有新依赖都install一遍可能更漫长的等待
* nginx 是实现创建好的nfs的pvc名称 mount_path 为在runner 这个ci任务执行容器中挂载的路径

## 2、verynginx 配置

### 2.1 配置matcher

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5ec03de9be254?w=558&h=219&f=png&s=22729)
### 2.2 配置back end

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5ec0e9c50ab8b?w=558&h=196&f=png&s=17311)

### 2.3 nginx后端配置
umi.conf   放到对应的扩展配置目录我这里是verynginx-conf 这个挂载的pvc

    server {                                                       
        listen       80;                                           
                                                                   
        #this line shoud be include in every server block                
        include /opt/verynginx/verynginx/nginx_conf/in_server_block.conf;
                                                                         
        location /umi-ant-pro {                                        
            root   /var/nginx/umi-ant-pro                                                 
            index  index.html;                                 
        }                               
    } 

## 3、umi配置
npm run ui

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5ec816719030f?w=1046&h=705&f=png&s=71020)

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5ec845904bfb5?w=968&h=554&f=png&s=52585)

## 4、拓展设计

ci执行结果

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5ecf3490cee29?w=1454&h=613&f=png&s=92597)

挂载的pvc变化

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5ed27f664aba1)

访问效果
http://192.168.1.4/umi-ant-pro/index.html

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5ed3f781f06a6?w=1386&h=782&f=png&s=53391)

方案一、采用此方式，ci会因pod 的构建而影响效率，即使install后注释掉install环节，

单纯使用缓存 build 发布，但缓存会时效。

方案二、采用docker方式选择dind，但配置编辑镜像的成本同样不低

**方案三、也是经验最佳方式，即不使用gitlab，更不使用jenkins ，而是定制个docker 镜像，放到k8s，达到类似本地开发，
实现，pull自动拉取热更，指向此镜像的访问即可，响应速度s级。**

## 注: 
* verynginx、gitlab-runner、umi针对的执行镜像配置地址

[https://github.com/liangguohun/dynamic-pvc.git](https://github.com/liangguohun/dynamic-pvc.git)

runner 的配置参考前一篇 spring的部署

* 前端demo包含ci配置 https://github.com/liangguohun/base-ant-pro.git
