origin是远程仓（remote repository）克隆（clone）到本地的默认名字。  
git push --set-upstream origin F1338317_penghy	设置源(目前公司account)  

git pull origin master	更新master代码到分支  

git branch       查看分支  
git branch name  创建分支  
git checkout     切换分支  
git merge 		 合并分支  
git log			 查看历史记录  
git reset		 恢复版本  
git reflog		 查看命令记录  

git status		 查看当前状态  
git add .		 把所有内容添加到本地git换成区  
git commit -m 'desc'	提交代码，推送修改到git库  
git push origin myBranch(origin myBranch可省略)  

git branch F1338317_penghy		要在本地创建分支  然后对应源  
git push --set-upstream origin F1338317_penghy		设置上游  
git checkout master		git pull	git checkout F1338317_penghy	git merge master  
# 
执行以下命令查看是否使用了淘宝的镜像地址，如果是，就改为原来的地址：  
npm config get registry  
npm config set registry=http://registry.npmjs.org  
改回淘宝的地址：npm config set registry=https://registry.npm.taobao.org   registry=http://registry.npm.taobao.org/  
把npm的镜像完全设为了淘宝的镜像，要注意有些依赖包只有npm原始镜像里面才有，而淘宝里面没有  

如果之前安装过vue的2.0版本，你需要把2.0相关的删除：npm uni -g vue-cli或者cnpm uni -g vue-cli  
如果没有安装或vue的2.0版本的话，直接在全局的命令窗口：cnpm i -g @vue/cli  
检查版本号	vue -V，vue create 项目名称  
package.json文件的scripts下的vue-cli-service serve后面加--open默认打开浏览器


tree /f > list.txt 获取项目代码目录结构
