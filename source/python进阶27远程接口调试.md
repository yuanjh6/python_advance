# python进阶27远程接口调试
需要将项目部署到armbox上，但是发现arm上调试非常繁琐。由于没有图形界面，所以无法使用pycharm工具，只能在终端中执行python命令，然后通过pdb进行调试。对于单线程尚无问题，但是对于web等多线程项目，pdb就无法进行调试了。  
之前做java时曾用过远程调试，查了下，pycharm其实也有这个功能。当然必须是专业版，免费版是不行的。  

## 原始代码：打印所属平台信息  
![](_v_images/20200907212820542_2073497232.png)  

## 添加Python Interpreter  
![](_v_images/20200907212909177_935270107.png)  

## 添加ssh interpreter连接信息   
![](_v_images/20200907213015067_226535770.png)  

## ssh interpreter密码信息  
这里大多数都会显示输入密码信息，本人这里是由于已经建立了免密的ssh连接，所以没有弹出密码框。  
![](_v_images/20200907213346683_1710582529.png)  

弹出密码样式  
![](_v_images/20200907214539917_1222010010.png)  

## interpreter配置信息  
![](_v_images/20200907215106544_6011886.png)  

## 修改执行代码的interpreter   
![](_v_images/20200907213917852_589214063.png)  

## 验证结果正确  
![](_v_images/20200907214126694_222326212.png)  


## 参考
PyCharm远程开发调试:https://blog.csdn.net/yejingtao703/article/details/80292486  
SSH 三步解决免密登录：https://blog.csdn.net/jeikerxiao/article/details/84105529  
ssh隧道解决pycharm跨过跳板机连接服务器问题：https://blog.csdn.net/huangbx_tx/article/details/93339715  