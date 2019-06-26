现在JAVA开发人员基本都在使用IDEA，那么使用idea一定要学会如何把项目打成war包  
1.使用快捷键Ctrl+Alt+Shift+s 或者 File菜单下面的 Project Structure按钮,打开Project Structure界面
<img src="https://github.com/Don-Lee/Notes/blob/master/Images/war1.png"/>  
2. 选择Artifacts，点击右边+,依次选择Web Application:Archive 和 For 'service:war exploded', 请注意此处的输出路径,war包会输出在此路径下
<img src="https://github.com/Don-Lee/Notes/blob/master/Images/war2.png"/>  
3. 点击下图红色箭头所指+图标，选择Directory Content,添加你的WebRoot目录，然后点击OK  
<img src="https://github.com/Don-Lee/Notes/blob/master/Images/war3.png"/>  
4. 点击“Build”,选择“Build Artifacts”  
<img src="https://github.com/Don-Lee/Notes/blob/master/Images/war4.png"/>  
5. 选择需要打包的项目并点击build  
<img src="https://github.com/Don-Lee/Notes/blob/master/Images/war5.png"/>  
 至此,打包工作已完成

