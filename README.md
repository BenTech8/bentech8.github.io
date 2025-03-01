## 基于Github Pages个人博客

搭建步骤：

1. 本地电脑安装Node.js。安装完毕后，确保环境变量配置好，能正常使用npm命令。

2. 安装hexo

   ```shell
   npm install -g hexo-cli
   ```

3. 初始化项目

   ```shell
   hexo init {project_name}
   ```

4. 将hexo编译生成HTML代码

   ```shell
   cd {project_name}
   hexo generate
   ```

5. 本地启动博客服务

   ```shell
   hexo serve
   ```

6. 查看博客

   博客地址：http://localhost:4000

7. github仓库创建一个名为{username}.github.io的仓库。

8. 本地电脑安装hexo-deployer-git插件。

   ```
   npm install hexo-deployer-git --save
   ```

9. 打开项目根目录下的_config.yml文件，修改Deployment地方，配置github仓库信息。

   ```
   deploy:
     type: git
     repo: https://github.com/BenTech8/bentech8.github.io.git
     branch: master
   ```

10. 配置github仓库用户上传代码认证token。

11. 执行部署。

    ```shell
    hexo deploy
    ```

12. 公网访问博客地址

    地址：http://{username}.github.io
