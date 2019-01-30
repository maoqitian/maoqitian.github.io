---
title: FastDFS服务器集群部署和集成客户端到SpringBoot
date: 2019-01-12 16:33:11
toc: true #是否显示文章目录
categories:  
- 后端 #分类
tags: 
- FastDFS
- spring boot
- Java
- CentOS
- Maven
---
> FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题，同时也能做到在集群环境下一台机子上传文件，同时该组下的其他节点下也备份了上传的文件。做分布式系统开发时，其中要解决的一个问题就是图片、音视频、文件共享的问题和数据备份，分布式文件系统正好可以解决这个需求。FastDFS的服务主要有两个角色Tracker和Storage，Tracker服务用于负责调度storage节点与client通信，在访问上起负载均衡的作用，和记录storage节点的运行状态，是连接client和storage节点的枢纽，Storage用于保存文件
### 1.FastDFS集群部署
 
#### 整体部署模块图
  
![FastDFS部署示意图](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/FastDFS%E9%83%A8%E7%BD%B2%E7%A4%BA%E6%84%8F%E5%9B%BE.png)
#### 环境准备
  | 名称 | 描述 |
  |-|-|
  | centos系统版本|6.9|
  |libfatscommon|FastDFS分离出的一些公用函数包|
  |FastDFS|FastDFS主程序|
  |fastdfs-nginx-module|FastDFS和nginx的关联模块|
  |nginx|nginx1.15.5|

#### 安装编译环境
    
```
    yum install git gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y
```
#### 磁盘安装路径说明
 |说明|位置|
 |-|-|
 |FastDFS所以安装包安装位置|/usr/local/src|
 |tracker数据 |/data/fdfs/tracker|
 |Storage数据 |/data/fdfs/Storage|
 |配置文件路径|/etc/fdfs|
    
#### 安装libfatscommon
    
- [下载libfatscommon](https://github.com/happyfish100/libfastcommon)

- 解压、安装
      
```
  unzip libfastcommon-master.zip
  cd libfastcommon-master
  ./make.sh && ./make.sh install #编译安装      
```
#### 安装FastDFS
    
- [下载FastDFS](https://github.com/happyfish100/fastdfs)
- 解压、安装
      
```
        unzip fastdfs-master.zip
        cd fastdfs-master
        ./make.sh && ./make.sh install #编译安装
        cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
        cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
        cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf #客户端文件，测试用
        cp /usr/local/src/fastdfs/conf/http.conf /etc/fdfs/ #供nginx访问使用
        cp /usr/local/src/fastdfs/conf/mime.types /etc/fdfs/ #供nginx访问使用
```
![etc目录下fdfs目录](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/etc%E7%9B%AE%E5%BD%95%E4%B8%8Bfdfs%E7%9B%AE%E5%BD%95.png)    
      
#### 安装fastdfs-nginx-module
     
- [下载fastdfs-nginx-module](https://github.com/happyfish100/fastdfs-nginx-module)
- 解压、安装
         
```
         unzip fastdfs-nginx-module-master.zip
         cp /usr/local/src/fastdfs-nginx-module-master/src/mod_fastdfs.conf /etc/fdfs #复制配置文件到fdfs目录
```
#### 安装nginx
    
- [下载nginx](http://nginx.org/download/nginx-1.15.5.tar.gz) 
- 解压、安装
        
```
        tar -zxvf nginx-1.15.5.tar.gz
        cd nginx-1.15.5
        #添加fastdfs-nginx-module模块
        ./configure --add-module=/usr/local/src/fastdfs-nginx-module-master/src/ 
        make && make install #编译安装
```
#### FastDFS集群部署配置
   
- tracker配置
    
 ```
     #服务器ip为 xxx.xxx.78.12, xxx.xxx.78.13
     vim /etc/fdfs/tracker.conf
     #需要修改的内容如下
     port=22122  # tracker服务器端口（默认22122,一般不修改）
     base_path=/data/fdfs/tracker #存储日志和数据的根目录
```
![tracker配置](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/tracker%E9%85%8D%E7%BD%AE.png)

- Storage配置
    
```
     vim /etc/fdfs/storage.conf
     #需要修改的内容如下
     port=23000  # storage服务端口（默认23000,一般不修改）
     base_path=/data/fdfs/storage  # 数据和日志文件存储根目录
     store_path0=/data/fdfs/storage  # 第一个存储目录
     tracker_server=xxx.xxx.78.12:22122  # 服务器1
     tracker_server=xxx.xxx.78.13:22122  # 服务器2
     http.server_port=8888  # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致)
```
- client配置
    
```
     vim /etc/fdfs/client.conf
     #需要修改的内容如下
     base_path=/home/moe/dfs
     tracker_server=xxx.xxx.78.12:22122  # 服务器1
     tracker_server=xxx.xxx.78.13:22122  # 服务器2
```
- 配置nginx访问
```
     vim /etc/fdfs/mod_fastdfs.conf
     #需要修改的内容如下
     tracker_server=xxx.xxx.78.12:22122  # 服务器1
     tracker_server=xxx.xxx.78.13:22122  # 服务器2
     url_have_group_name=true
     store_path0=/data/fdfs/storage
     
     #配置nginx.config
     vim /usr/local/nginx/conf/nginx.conf
     #添加如下配置
     server {
     listen       8888;    ## 该端口为storage.conf中的http.server_port相同
     server_name  localhost;
     location ~/group[0-9]/ {
        ngx_fastdfs_module;
     }
     ......
     ......
     error_page   500 502 503 504  /50x.html;
     location = /50x.html {
     root   html;
     }
     }
```
#### 启动服务、测试
    
```
    启动之前我们还需要在防火墙开通端口
    vim  /etc/sysconfig/iptables
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 23000 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 8888 -j ACCEPT

    service iptables restart #重启防火墙
```
![防火墙端口](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/%E9%98%B2%E7%81%AB%E5%A2%99%E7%AB%AF%E5%8F%A3.png)

- 每个服务的启动、关闭和重启操作
```
    #tracker
    /etc/init.d/fdfs_trackerd start #启动tracker服务
    /etc/init.d/fdfs_trackerd restart #重启动tracker服务
    /etc/init.d/fdfs_trackerd stop #停止tracker服务
    chkconfig fdfs_trackerd on #自启动tracker服务
    
    #storage
    /etc/init.d/fdfs_storaged start #启动storage服务
    /etc/init.d/fdfs_storaged restart #重动storage服务
    /etc/init.d/fdfs_storaged stop #停止动storage服务
    chkconfig fdfs_storaged on #自启动storage服务
    
    #nginx
    /usr/local/nginx/sbin/nginx #启动nginx
    /usr/local/nginx/sbin/nginx -s reload #重启nginx
    /usr/local/nginx/sbin/nginx -s stop #停止nginx
```
![查看运行的服务](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/%E6%9F%A5%E7%9C%8B%E8%BF%90%E8%A1%8C%E7%9A%84%E6%9C%8D%E5%8A%A1.png)

#### 检测集群  
    
```
 # 会显示会有几台storage服务器,有2台就会显示 Storage 1-Storage 2的详细信息
 /usr/bin/fdfs_monitor /etc/fdfs/storage.conf   
```
![检测集群1](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/%E6%A3%80%E6%B5%8B%E9%9B%86%E7%BE%A41.png)
    
![检测集群2](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/%E6%A3%80%E6%B5%8B%E9%9B%86%E7%BE%A42.png)
#### 图片上传测试
  
```
    #上传成功返回 文件访问 ID
    # fdfs_upload_file 客户端配置文件      上传文件路径
    fdfs_upload_file /etc/fdfs/client.conf /data/test.png
    
```
![上传文件成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6%E6%88%90%E5%8A%9F.png)

#### 测试文件访问
```
    http://xxx.xxx.78.12/group1/M00/00/00/rB9ODFvXuSiAWBYBAALSAkm_6RQ360.png
    http://xxx.xxx.78.13/group1/M00/00/00/rB9ODFvXuSiAWBYBAALSAkm_6RQ360.png
```
- 测试nginx默认端口80 访问刚刚上传的文件，两个地址都能访问通一个文件，达到数据备份目的。

>至此，FastDFS服务器部署完成    
    
### FastDFS客户端集成到SpringBoot
#### 编译获取FastDFS jar包
- 首先根据官方源码提示，我们先下载源码使用maven编译成jar包放到公司maven私服（Nexus），或者你本地的maven私服（也有其他ant等方式，具体请查看github）[FastDFS-java-client-SDK源码下载地址](https://github.com/happyfish100/fastdfs-client-java)
```
#编译jar包（解压下载的FastDFS-java-client-SDK源码，使用mvn命令需要先有maven环境）
mvn clean install
    
```
![编译打包FastDFS-java-client](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85FastDFS-java-client.png)
    
![fastdfs-client-java打包成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/fastdfs-client-java%E6%89%93%E5%8C%85%E6%88%90%E5%8A%9F.png)
    
#### maven项目pom.xml中添加依赖
```
     <dependency>
       <groupId>org.csource</groupId>
       <artifactId>fastdfs-client-java</artifactId>
       <version>1.27-SNAPSHOT</version>
     </dependency>
```
- 接下来我们在项目resources目录下添加fdfs_client.conf文件
  
```
     connect_timeout = 30
     network_timeout = 30
     charset = UTF-8
     http.tracker_http_port = 80
     http.anti_steal_token = no
     http.secret_key = 123456
     #前面配置的集群tracker服务器地址
     tracker_server = xxx.xxx.78.12:22122
     tracker_server = xxx.xxx.78.13:22122
```
#### 代码编写
- 写一个上传文件对象类
     
```
     /**
      * @Author: maoqitian
      * @Date: 2018/10/26 0026 17:57
      * @Description: FastDFS 文件类
      */
      public class FastDFSFileEntity {
      //文件名称
      private String name;
      //内容
      private byte[] content;
      //文件类型
      private String ext;
      //md5值
      private String md5;
      //作者
      private String author;
      public FastDFSFileEntity(String name, byte[] content, String ext, String height,
                       String width, String author) {
        super();
        this.name = name;
        this.content = content;
        this.ext = ext;
        this.author = author;
       }

       public FastDFSFileEntity(String name, byte[] content, String ext) {
        super();
        this.name = name;
        this.content = content;
        this.ext = ext;
        }

        public String getName() {
        return name;
        }

        public void setName(String name) {
        this.name = name;
        }

        public byte[] getContent() {
        return content;
        }

        public void setContent(byte[] content) {
        this.content = content;
        }

        public String getExt() {
        return ext;
        }

        public void setExt(String ext) {
        this.ext = ext;
        }

        public String getMd5() {
        return md5;
        }

        public void setMd5(String md5) {
        this.md5 = md5;
        }

        public String getAuthor() {
        return author;
        }

        public void setAuthor(String author) {
        this.author = author;
        }
      }
```
- 编写FastDFS操作类，主要是加载初始化配置Tracker服务器，文件上传，下载，删除等操作工具类
   
```
     /**
      * @Author: maoqitian
      * @Date: 2018/10/29 0029 9:30
      * @Description: FastDFS 操作类
      */
      public class FastDFSClient {

       private static org.slf4j.Logger logger = LoggerFactory.getLogger(FastDFSClient.class);
       //双重守护单例
       private static volatile FastDFSClient mInstance;

       /**
        * 加载配置信息
        **/
       static {
        try {
            String filePath=new ClassPathResource("fdfs_client.conf").getFile().getAbsolutePath();
            ClientGlobal.init(filePath);
        }catch (Exception e){
            logger.error("FastDFS Client Init Fail!",e);
        }
       }

       private FastDFSClient(){

       }

        public static FastDFSClient getInstance(){
        if(mInstance == null){
           synchronized (FastDFSClient.class){
               if(mInstance == null){
                   mInstance=new FastDFSClient();
               }
           }
        }
        return mInstance;
       }

       /**
        * @Author maoqitian
        * @Description 上传文件
        * @Date 2018/10/29 0029 9:42
        * @Param [fastDFSFileEntity]
        * @return java.lang.String[]
        **/
        public  String[] upload(FastDFSFileEntity file){
        logger.info("File Name: " + file.getName() + "File Length:" + file.getContent().length);

        NameValuePair[] metalist=new NameValuePair[1];

        metalist[0]=new NameValuePair("author",file.getAuthor());

        long startTime = System.currentTimeMillis();
        String[] uploadResults= null;
        StorageClient storageClient=null;
        try {

            storageClient=getTrackerClient();
            uploadResults = storageClient.upload_file(file.getContent(),file.getExt(),metalist);
        }catch (IOException e){
            logger.error("IO Exception when uploadind the file:"+file.getName(),e);
        }
        catch (Exception e){
            logger.error("Non IO Exception when uploadind the file:"+file.getName(),e);
        }
        logger.info("upload_file time used:" + (System.currentTimeMillis() - startTime) + " ms");
        if(uploadResults==null && storageClient!=null){
            logger.error("upload file fail, error code:" + storageClient.getErrorCode());
        }
        String groupName = uploadResults[0];
        String remoteFileName = uploadResults[1];

        logger.info("upload file successfully!!!" + "group_name:" + groupName + ", remoteFileName:" + " " + remoteFileName);
        return uploadResults;
        }


        public  FileInfo getFile(String groupName, String remoteFileName) {
        try {
            StorageClient storageClient = getTrackerClient();
            return storageClient.get_file_info(groupName, remoteFileName);
        } catch (IOException e) {
            logger.error("IO Exception: Get File from Fast DFS failed", e);
        } catch (Exception e) {
            logger.error("Non IO Exception: Get File from Fast DFS failed", e);
        }
        return null;
        }

        public  InputStream downFile(String groupName, String remoteFileName) {
        try {
            StorageClient storageClient = getTrackerClient();
            byte[] fileByte = storageClient.download_file(groupName, remoteFileName);
            InputStream ins = new ByteArrayInputStream(fileByte);
            return ins;
        } catch (IOException e) {
            logger.error("IO Exception: Get File from Fast DFS failed", e);
        } catch (Exception e) {
            logger.error("Non IO Exception: Get File from Fast DFS failed", e);
        }
        return null;
        }
        /**
         * @Author maoqitian
         * @Description
         * @Date 2018/10/31 0031 11:19
         * @Param [remoteFileName]
         * @return int -1 失败 0成功
         **/
        public int deleteFile(String remoteFileName)
            throws Exception {
        StorageClient storageClient = getTrackerClient();
        int i = storageClient.delete_file("group1", remoteFileName);
        logger.info("delete file successfully!!!" + i);
        return i;
        }

        public StorageServer[] getStoreStorages(String groupName)
            throws IOException {
        TrackerClient trackerClient = new TrackerClient();
        TrackerServer trackerServer = trackerClient.getConnection();
        return trackerClient.getStoreStorages(trackerServer, groupName);
        }

         public ServerInfo[] getFetchStorages(String groupName,
                                                String remoteFileName) throws IOException {
        TrackerClient trackerClient = new TrackerClient();
        TrackerServer trackerServer = trackerClient.getConnection();
        return trackerClient.getFetchStorages(trackerServer, groupName, remoteFileName);
        }

        public  String getTrackerUrl() throws IOException {
        return "http://"+getTrackerServer().getInetSocketAddress().getHostString()+":"+ClientGlobal.getG_tracker_http_port()+"/";
        }

        /**
         * @Author maoqitian
         * @Description 获取 StorageClient
         * @Date 2018/10/29 0029 10:33
         * @Param []
         * @return org.csource.fastdfs.StorageClient
         **/
        private StorageClient getTrackerClient() throws IOException{
        TrackerServer trackerServer=getTrackerServer();
        StorageClient storageClient=new StorageClient(trackerServer,null);
        return storageClient;
        }
        /**
         * @Author maoqitian
         * @Description 获取 TrackerServer
         * @Date 2018/10/29 0029 10:34
         * @Param []
         * @return org.csource.fastdfs.TrackerServer
         **/
        private  TrackerServer getTrackerServer() throws IOException {
        TrackerClient trackerClient=new TrackerClient();
        TrackerServer trackerServer = trackerClient.getConnection();
        return trackerServer;
      }

```
- Controller编写，接收请求并上传文件返回文件访问路径（这里写一个文件上传的例子，其他文件下载，删除等功能可根据自己需求进行编写）
```
    /**
	 * @Author maoqitian
	 * @Description  上传文件
	 * @Date 2018/10/30 0030 15:07
	 * @Param [file]
	 * @return com.gxxmt.common.utils.ResultApi
	 **/
	  @RequestMapping("/upload")
	  public ResultApi upload(@RequestParam("file") MultipartFile file) throws Exception {
		if (file.isEmpty()) {
			throw new RRException("上传文件不能为空");
		}
		String url;
		//此处域名获取可以根据自需求编写
		String domainUrl = OSSFactory.build().getDomainPath();
		logger.info("配置的域名为"+domainUrl);
		if (StringUtils.isNotBlank(domainUrl)){
			url = uploadFile(file,domainUrl);
			return ResultApi.success.put("url",url);
		}else {
			return ResultApi.error("域名配置为空,请先配置对象存储域名");
		}
	  }
	  
	 /**
     * @Author maoqitian
     * @Description 上传文件到 FastDFS
     * @Date 2018/10/29 0029 11:11
     * @Param [file]
	 * @Param [domainName] 域名
	 * @return path 文件访问路径
     **/
	 public String uploadFile(MultipartFile file,String domainName) throws IOException {

		String[] fileAbsolutePath={};
		String fileName=file.getOriginalFilename();
		String ext=fileName.substring(fileName.lastIndexOf(".")+1);
		byte[] file_buff=null;
		InputStream inputStream = file.getInputStream();
		if(inputStream!=null){
			int available = inputStream.available();
			file_buff=new byte[available];
			inputStream.read(file_buff);
		}
		inputStream.close();
		FastDFSFileEntity fastDFSFileEntity=new FastDFSFileEntity(fileName,file_buff,ext);
		try {
			fileAbsolutePath=FastDFSClient.getInstance().upload(fastDFSFileEntity);
			logger.info(fileAbsolutePath.toString());
		}catch (Exception e){
			logger.error("upload file Exception!",e);
			throw new RRException("文件上传出错"+e);
		}
		if(fileAbsolutePath == null){
			logger.error("upload file failed,please upload again!");
			throw new RRException("文件上传失败，请重新上传");
		}
		String path=domainName+fileAbsolutePath[0]+ "/"+fileAbsolutePath[1];
		return path;
	  }
```
#### 测试部署成果
    
- 上传一个图片，由日志打印我们可以看出图片已经上传成功 
    
![FastDFS-java-client 上传图片成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/FastDFS-java-client%20%E4%B8%8A%E4%BC%A0%E5%9B%BE%E7%89%87%E6%88%90%E5%8A%9F.png)

- 测试访问上传的图片
      
![测试上传的图片是否可以进行访问](https://github.com/maoqitian/MaoMdPhoto/raw/master/FastDFS/%E6%B5%8B%E8%AF%95%E4%B8%8A%E4%BC%A0%E7%9A%84%E5%9B%BE%E7%89%87%E6%98%AF%E5%90%A6%E5%8F%AF%E4%BB%A5%E8%BF%9B%E8%A1%8C%E8%AE%BF%E9%97%AE.png)

### 最后说点    
- 到此，FastDFS服务器集群部署和集成客户端到SpringBoot中已经完成，以后我们就可以愉快的使用FastDFS服务保存我们的图片等并备份。如果文章中有写得不对的地方，请给我留言指出，大家一起学习进步。如果觉得我的文章给予你帮助，也请给我一个喜欢和关注。