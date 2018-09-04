# Jmeter接口自动化框架(三) Jenkins的相关配置

## 配置相关插件

如综述中所说,Jenkins主要完成的工作主要是拉取代码后交由maven，由maven完成具体的测试步骤,整个测试过程要用过哪些Jenkins的插件和配套环境呢,首先是需要安装Jenkins自身git和maven以及配套的Jenkins插件，其次是权限管理插件和邮件发送插件，以及测试要用的性能分析插件和页面保存插件，那么一步一步来吧。

## Jenkins的安装

Jenkins的安装主要方法有两种直接运行JAR和放到Web容器中,其实都是安装在了家目录的.jenkins目录下,运行JAR包或war包后直接按提示下一步操作并设置好管理员用户密码(下载和如何运行请直接百度)。

## 基础配置

进入主界面后首页安装Jenkins的全局工具配置，进入`系统设置-->全局工具配置`页面,分别把`JDK`,`Git`,`Maven`三个配置项内的路径配置对应的安装路径。

## 插件的安装

进入`系统设置-->管理插件页面`,分别搜索和安装以下插件

| 插件名                            | 作用描述                 |
| --------------------------------- | ------------------------ |
| Git                               | 管理Git                  |
| Role-based Authorization Strategy | 用户及项目权限管理       |
| Performance                       | 解析和展示Jmeter性能结果 |
| HTML Publisher plugin             | 转移保存和展示HTML页面   |
| Email Extension                   | 发送构建构建邮件         |
把以上插件都安装好之后,再配置相关选项。如果在安装插件的时候因为长城的问题看不到插件列表，进入插件页面的`高级`选项,配置访问代理或把升级站点换成`http://mirrors.shu.edu.cn/jenkins//updates/update-center.json`就可以正常访问了。

## 权限控制

1. 安装了权限控制插件就可以对角色和项目进行人员的权限控制和访问控制，进入`系统设置-->Manage and Assign Roles-->Manage Roles`页面,先进行角色管理，全局角色主要用于全局控制。
  ![03-持续集成测试的Jenkins配置-2018827102339](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827102339.png)
项目角色主要用于对项目的分别控制。
  ![03-持续集成测试的Jenkins配置-2018827102436](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827102436.png)

2. 配置好角色后返回上一层进入`Assign Roles`页面,进行用户配置,用户可以在`用户管理`里进行添加。给角色分配不同的权限以达到控制项目和权限的作用。
  ![03-持续集成测试的Jenkins配置-2018827102744](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827102744.png)

## 新建测试工程及配置

进入到Jekins的首页,点击`新建任务`,输入一个任务名后选项自由风格的软件项目,然后点确定。关于项目的命名问题必须规范化，以便控制和管理，如我在这里配置和Jtest开头和Japp开头的,以便不同的人员只能看到对应的项目。

1. `源码管理配置`,配置对应的git仓库,如果是帐号密码访问在高级里配置帐号和密码,如果是密钥访问提前在系统里配置好密钥。
  ![03-持续集成测试的Jenkins配置-20188271162](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-20188271162.png)

2. `构建触发器`,触发器主要决定了Jekins什么情况或什么时候来执行相关测试,这里先简单的配置为定时执行,触发命令参考Jenkins帮助,这里的命令为周一到周五每天早上8点到晚上8点初的第15分,每4小时执行一次。
  ![03-持续集成测试的Jenkins配置-2018827111057](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827111057.png)

3. `构建`,在构建项主要执行的maven命令和脚本的选择范围,这里执行的以名字Hll开头的脚本，clean和install会清理和安装最新的JAVA代码,方便Jmeter在调用中使用。
  ![03-持续集成测试的Jenkins配置-2018827111332](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827111332.png)

4. `构建后操作`,构建后操作主要作用是测试完成对测试结果进行解析和保存并发送邮件通知，这里一个一个说

5. `性能解析插件`在构建后操作选项里点击添加`Publish Performance test result report`步骤,并配置以下相关内容,这个插件主要是把Jmeter的结果解析为性能报告。
  ![03-持续集成测试的Jenkins配置-2018827111813](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827111813.png)

6. `页面转移和保存插件`在构建后操作选项里点击添加`Publish HTML reports`步骤,并配置以下相关内容,这个插件主要是把配置指定路径的页面转移到Job里,方便查看。这里可以配置显示上面的性能页面和Jmeter自己生成的测试报告（3.0以后版本支持自己Jmeter报告），我这里是显示Jmeter自己生成的报告，把路径配置到指定文件夹。这里有个特殊的地方，在上一章配置工程的时候有提过`jmeter-maven-plugin`这个插件生成的结果文件夹是有时间格式化的,每次生成路径都会变化,这个功能是在代码里写死的不能配置,所以要稍把原来的功能修改一下,把插件的源代码下载下来修改相应地方的源代码，完成后重新打包用自己的新包就可以用这个功能配合使用来显示Jmeter自己生成的报告了。
  ![03-持续集成测试的Jenkins配置-2018827112650](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827112650.png)
配置完成后执行后进入对应的页面
  ![03-持续集成测试的Jenkins配置-201882711297](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-201882711297.png)
进入后这个页面就是显示的是Jmeter自己生成的那个性能报告
  ![03-持续集成测试的Jenkins配置-2018827112919](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827112919.png)  
7. `邮件通知功能`如果要使用邮件插件,必须在系统设置里先设置系统自带的邮件发送管理员设置和插件发送设置
  系统管理发送邮件要设置好
  ![03-持续集成测试的Jenkins配置-2018827114033](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827114033.png)
  插件的邮件发送设置,注意现在很多邮箱的密码是授权码而不是你使用的密码,授权码的获取要对应邮件安全管理里去获取
  ![03-持续集成测试的Jenkins配置-2018827114040](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827114040.png)
8. `邮件设置及模板`构建完成后要对构建结果发送对应的项目成员,查看构建结果和测试报告,在构建后操作选项里点击添加`Editable Email Notification`步骤,按哪下设置
  ![03-持续集成测试的Jenkins配置-2018827113637](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827113637.png)
  邮件可以使用模板,现在都是看脸的世界,发出去的邮件也不能太简陋,可以把邮件的模版稍微设计一下,复制到如上图的地方。可以用一些Jenkins的环境变量代替邮件的内容，这个收到的邮件的人就可以直接在在邮件里打开报告地址。
  这是之前用的一个简单模板，发出去的是这个样子。
  ![03-持续集成测试的Jenkins配置-2018827123736](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827123736.png)

  有同学要模版,这个是Jenkins可以用的构建模版,另一个需要后台服务支持,先不发了,到时候和后台应用一起发

  ```html

  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">

<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	<title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建结果</title>
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />
</head>

<body style="margin: 0; padding: 0;">
	<table border="0" cellpadding="0" cellspacing="0" width="100%">
		<tr>
			<td style="padding: 10px 0 30px 0;">
				<table align="center" border="0" cellpadding="0" cellspacing="0" width="600" style="border: 1px solid #cccccc; border-collapse: collapse;">
					<tr>
						<td align="center" bgcolor="#e5ffff" style="padding: 40px 0 30px 0; color: #153643; font-size: 28px; font-weight: bold; font-family:Microsoft YaHei,Arial, sans-serif;">
							<!-- <img src="images/h1.gif" alt="Creating Email Magic" width="300" height="50" style="display: block;" /> -->
							<b>自动化测试构建结果</b>
						</td>
					</tr>
					<tr>
						<td bgcolor="#ffffff" style="padding: 40px 30px 40px 30px;">
							<table border="0" cellpadding="0" cellspacing="0" width="100%">
								<tr>
									<td style="padding: 20px 0 30px 0; color: #153643; font-family:Microsoft YaHei, Arial, sans-serif; font-size: 24px;">
										<b>项目:${PROJECT_NAME}</b>
									</td>
								</tr>
								<tr>
									<td style="padding: 20px 0 5px 0; color: #153643; font-family:Microsoft YaHei, Arial, sans-serif; font-size: 16px; line-height: 20px;">
										<b>构建结果</b> - ${BUILD_STATUS}
									</td>
								</tr>
								<tr>
									<td style="padding: 20px 0 5px 0; color: #153643; font-family:Microsoft YaHei, Arial, sans-serif; font-size: 16px; line-height: 20px;">
										<b>构建编号</b> - 第${BUILD_NUMBER}次构建
									</td>
								</tr>
								<tr>
									<td style="padding: 20px 0 5px 0; color: #153643; font-family:Microsoft YaHei, Arial, sans-serif; font-size: 16px; line-height: 20px;">
										<b>触发原因</b> - ${CAUSE}
									</td>
								</tr>
								<tr>
									<td style="padding: 20px 0 5px 0; color: #153643; font-family:Microsoft YaHei, Arial, sans-serif; font-size: 16px; line-height: 20px;">
										<b>构建URL</b> - <a href="${PROJECT_URL}">项目构建URL</a>
									</td>
								</tr>
								<tr>
									<td style="padding: 20px 0 5px 0; color: #153643; font-family:Microsoft YaHei, Arial, sans-serif; font-size: 16px; line-height: 20px;">
										<b>报告明细</b> - <a href="${PROJECT_URL}/PerformanceReport/"><b>明细报告URL</b></a>
									</td>
								</tr>
								<tr>
									<td style="padding: 20px 0 5px 0; color: #153643; font-family:Microsoft YaHei, Arial, sans-serif; font-size: 16px; line-height: 20px;">
										本邮件为自动化测试构建时的通知，请勿回复。
									</td>
								</tr>
							</table>
						</td>
					</tr>
					<tr>
						邮件标签
					</tr>
				</table>
			</td>
		</tr>
	</table>
</body>

</html>

  ```

## 框架总结

到这里一个基本的接口自动化框架就可以使用了,剩下的内容我们基于这个框架做一些优化,包括怎么把Jmeter的测试结果持久化起来方便汇报查看和回归,怎么用自己的平台独立运行Jmeter，基于属于一些二次开发的内容。

## 下一章内容

有同学就发现这邮件发送没有直接的测试结果，还要别人点进去看，而且Jmeter的自己的报告也很搓，不直观也很难看，那么下一章我们主要说怎么把Jmeter的结果存到数据库，并利用数据库的数据自己收集和整理测试报告，并发送邮件，整理后的报告就可以直接显示结果和错误内容了。
  ![03-持续集成测试的Jenkins配置-2018827124818](http://owo8mviga.bkt.clouddn.com/03-持续集成测试的Jenkins配置-2018827124818.png)