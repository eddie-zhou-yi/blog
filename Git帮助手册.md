# Git 帮助手册

1. 初始化仓库：

   ```shell
   git init
   ```

2. 设置单个仓库的提交用户名与邮箱

   ```shell
   git config user.name "eddie-zhou-yi"
   git config user.email "401700108@qq.com"
   ```

3. 设置全局的git提交用户名与邮箱

   ```shell
   git config --global user.name "eddie-zhou-yi"
   git config --global user.email "401700108@qq.com"
   ```

4. 查看仓库的配置信息

   ```shell
   git config --list
   git config --local --list
   git config --global --list
   ```

5. 