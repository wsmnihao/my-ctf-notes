# [攻防世界-[NO.GFSJ0234] [PHP2]

## 漏洞信息
- **漏洞类型**：源代码泄露
- **风险等级**：中危->高危（源代码泄露会衍生其他漏洞）
- **利用难度**：极易
- **关键函数**：无
		只有配置不当
[[PHP反序列化&串行化]]
## 解题过程
### 漏洞发现
```代码
	1、信息挖掘
	#首页只有一个提示语：
		Can you anthenticate to this website?
		#注释：**“你能通过这个网站的身份验证吗？”**
		
	2、检查源代码
	#没有任何代码构造语句
	
	3、尝试访问首页和一些关键目录及敏感文件
		手工收入较慢的话，或者按需使用，可以尝试用御剑目录扫描工具尝试
	/index.phps
	/login.phps
	/admin.phps
	/config.phps
	/flag.phps
	/backup.phps
	/index.php.bak
	/login.php.bak
	/index.php~
	/.index.php.swp
	#带s和不带s的都尝试一下
	
	
	4、漏洞发现
	/index.phps
	#是一个源代码泄露漏洞
	#返回一些配置代码，示例如下：
	
	<?php
	if("admin"===$_GET[id]) {
	  echo("<p>not allowed!</p>");
	  exit();
	}

	$_GET[id] = urldecode($_GET[id]);
	if($_GET[id] == "admin")
	{
	  echo "<p>Access granted!</p>";
	  echo "<p>Key: xxxxxxx </p>";
	}
	?>
	

// 漏洞代码片段
```
### 代码分析
```代码
	===和==的验证通过问题
	
	#===是强验证，==是弱验证
	
	1、第一层$_GET[id])是否完全等于admin，如果相等，就返回not allowed！
	
	2、中间是将$_GET[id]的值进行URL解码
	
	3、第二层$_GET[id]的验证比较松散，只要符合其中的一个就返回flag/key
	

```
### 绕过方式
```代码
	1、尝试构建绕过代码
		关键数据：admin
	构建命令
		http://example.com/?id=%61%64%6D%69%6E	
		#参考URL编解码和ASCII编码表比对，或者快一点就找ai帮忙编译
		
	2、绕过无效，尝试双重绕过
	#在原基础上，将%号URL编码为%25，这样既绕过了第一层限制，%后面跟65，实意为ASCII表中的a，对应admin的第一个字符，以此类推，构建一下绕过代码
	
	3、代码
	#尝试绕过
	http://目标网址/?id=%2561%2564%256D%2569%256E	
	
	4、flag值
	flag： cyberpeace{0a7a4bffec9d817fad109e6671f58df4}
	
	完美！！！