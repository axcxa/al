# al
点滴记忆
忘记密码？
如果您在使用emlog系统过程中忘记了自己的后台密码，可以使用emlog密码重置工具来重置密码：

工具下载地址：https://github.com/axcxa/al/blob/master/passwd.zip

emlog管理员密码重置工具，用于在忘记管理员帐号和密码的情况下找回管理员帐号和密码。

使用方法：

1. 将下载的zip包解压。
2. 将解压后的 passwd.php文件上传到emlog的根目录
3. 在浏览器里访问：http:你网站的域名/passwd.php 按照提示操作重置密码。重置后如果该文件没自动删除请务必手动删除。

passwd.php

<?php
/**
 * axcxa密码重置工具
 * @copyright (c) Emlog All Rights Reserved
 */

define('EMLOG_ROOT', dirname(__FILE__));
define('DEL_INSTALLER', 1);

require_once EMLOG_ROOT.'/config.php';
require_once EMLOG_ROOT.'/include/lib/function.base.php';

header('Content-Type: text/html; charset=UTF-8');
doStripslashes();

$act = isset($_GET['action'])? $_GET['action'] : '';

$DB = MySql::getInstance();
$CACHE = Cache::getInstance();
$sql = "SELECT username FROM ".DB_PREFIX."user  WHERE uid=1";
$row = $DB->once_fetch_array($sql);
$user_name = $row['username'] ? $row['username'] : '';

if(!$act){
?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" lang="zh-CN">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<style type="text/css">
<!--
body {background-color:#F7F7F7;font-family: Arial;font-size: 14px;line-height:150%;}
.main {background-color:#FFFFFF;font-size: 14px;color: #666666;width:750px;margin:30px auto;padding:10px;list-style:none;border:#DFDFDF 1px solid; border-radius: 4px;}
.title{text-align:center; font-size: 14px;}
.input {border: 1px solid #CCCCCC;font-family: Arial;font-size: 14px;height:20px; width: 150px;background-color:#F7F7F7;color: #666666;}
.submit{cursor: pointer;font-size: 12px;padding: 4px 10px;}
.care{color:#0066CC;}
.title2{font-size:18px;color:#666666;border-bottom: #CCCCCC 1px solid; margin:40px 0px 20px 0px;padding:10px 0px;}
.center{text-align:center;}
.main li{ margin:20px 0px;}
.notice{font-size: 16px;color:#F60;}
-->
</style>
</head>
<body>
<form name="form1" method="post" action="passwd.php?action=chpwd">
<div class="main">
<p class="title">管理员密码重置</p>
<div class="b">
<p class="title2"></p>
<p class="center">你确定要将管理员 <span class="notice"><?php echo $user_name;?></span> 的密码重置为<span class="notice"><input name="passwd" type="text" class="input" value="123456"></span> 吗？</p>
</div>
<div>
<p class="center">
<input type="submit" class="submit" value="确定重置">
</p>
</div>
<p class="title2"></p>
<div class="center">© emlog</div>
</div>
</form>
</body>
</html>
<?php
}
if($act == 'chpwd'){
	$adminpw = isset($_POST['passwd']) ? addslashes(trim($_POST['passwd'])) : '';

	if($adminpw == ''){
		emMsg('密码不能为空!');
	}elseif(strlen($adminpw) < 6){
		emMsg('登录密码不得小于6位');
	}

	$PHPASS = new PasswordHash(8, true);
	$adminpw_hash = $PHPASS->HashPassword($adminpw);

	$sql = "update ".DB_PREFIX."user set password='$adminpw_hash'  WHERE uid=1";
	$DB->query($sql);

	$result = "
		<p style=\"font-size:24px; border-bottom:1px solid #E6E6E6; padding:10px 0px;\">恭喜！你的管理员密码已重置!</p>
		<p><b>用户名</b>：{$user_name}</p>
		<p><b>新的密码</b>：$adminpw</p>";

	if (DEL_INSTALLER === 1 && !@unlink('./passwd.php') || DEL_INSTALLER === 0) {
	    $result .= '<p style="color:red;margin:10px 20px;">警告：请手动删除根目录下安装文件：passwd.php</p> ';
	}
	$result .= "<p style=\"text-align:right;\"><a href=\"./\">访问首页</a> | <a href=\"./admin/\">登录后台</a></p>";
	emMsg($result, 'none');
}
?>
