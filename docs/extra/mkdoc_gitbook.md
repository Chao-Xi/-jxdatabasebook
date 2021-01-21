# **GitHub+Travis+Mkdocs自动化构建文档库(DataBase)**

```
mkdocs new jxdatabasebook

cd jxdatabasebook

$ ls
docs            mkdocs.yml

$  git init
Initialized empty Git repository in /Users/../Devops_sap/Tech_Books/jxdatabasebook/.git/

￥ git remote add origin https://github.com/Chao-Xi/jxdatabasebook.git

$  mkdocs gh-deploy --clean
INFO    -  Cleaning site directory 
INFO    -  Building documentation to directory: /Users/.../Devops_sap/Tech_Books/jxdatabasebook/site 
INFO    -  Documentation built in 0.10 seconds 
WARNING -  Version check skipped: No version specified in previous deployment. 
INFO    -  Copying '/Users/.../Devops_sap/Tech_Books/jxdatabasebook/site' to 'gh-pages' branch and pushing to GitHub. 
INFO    -  Your documentation should shortly be available at: https://Chao-Xi.github.io/jxdatabasebook/


$ pip3 install mkdocs-awesome-pages-plugin
$  mkdocs gh-deploy --clean
```


* [https://travis-ci.com](https://travis-ci.com)
* [https://travis-ci.com/github/Chao-Xi/jxdatabasebook](https://travis-ci.com/github/Chao-Xi/jxdatabasebook)


	* `GITHUB_NAME`: `github password`
	* `GITHUB_EMAIL`: `xichao2015@outlook.com`
	* `GITHUB_API_KEY`: `73c...7d98 `




