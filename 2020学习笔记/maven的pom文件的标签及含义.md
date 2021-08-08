1. ```xml
   1. 
      pom作为项目对象模型。通过xml表示maven项目，使用pom.xml来实现。主要描述了项目：包括配置文件；开发者需要遵循的规则，缺陷管理系统，组织和licenses，项目的url，项目的依赖性，以及其他所有的项目相关因素。
   2. [xml] view plain copy print?
   3. <span style="padding:0px; margin:0px"><project xmlns="http://maven.apache.org/POM/4.0.0"     
   4. ​    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"     
   5. xsi:schemaLocation="http://maven.apache.org/POM/4.0.0http://maven.apache.org/maven-v4_0_0.xsd">     
   6. ​    *<!--父项目的坐标。如果项目中没有规定某个元素的值，那么父项目中的对应值即为项目的默认值。 坐标包括group ID，artifact ID和 version。-->*    
   7. ​    <parent>    
   8. ​     *<!--被继承的父项目的构件标识符-->*    
   9. ​     <artifactId/>    
   10. ​     *<!--被继承的父项目的全球唯一标识符-->*    
   11. ​     <groupId/>    
   12. ​     *<!--被继承的父项目的版本-->*    
   13. ​     <version/>    
   14. ​     *<!-- 父项目的pom.xml文件的相对路径。相对路径允许你选择一个不同的路径。默认值是../pom.xml。Maven首先在构建当前项目的地方寻找父项 目的pom，其次在文件系统的这个位置（relativePath位置），然后在本地仓库，最后在远程仓库寻找父项目的pom。-->*    
   15. ​     <relativePath/>    
   16.  </parent>    
   17.  *<!--声明项目描述符遵循哪一个POM模型版本。模型本身的版本很少改变，虽然如此，但它仍然是必不可少的，这是为了当Maven引入了新的特性或者其他模型变更的时候，确保稳定性。-->*       
   18. ​    <modelVersion>4.0.0</modelVersion>     
   19. ​    *<!--项目的全球唯一标识符，通常使用全限定的包名区分该项目和其他项目。并且构建时生成的路径也是由此生成， 如com.mycompany.app生成的相对路径为：/com/mycompany/app-->*     
   20. ​    <groupId>asia.banseon</groupId>     
   21. ​    *<!-- 构件的标识符，它和group ID一起唯一标识一个构件。换句话说，你不能有两个不同的项目拥有同样的artifact ID和groupID；在某个 特定的group ID下，artifact ID也必须是唯一的。构件是项目产生的或使用的一个东西，Maven为项目产生的构件包括：JARs，源 码，二进制发布和WARs等。-->*     
   22. ​    <artifactId>banseon-maven2</artifactId>     
   23. ​    *<!--项目产生的构件类型，例如jar、war、ear、pom。插件可以创建他们自己的构件类型，所以前面列的不是全部构件类型-->*     
   24. ​    <packaging>jar</packaging>     
   25. ​    *<!--项目当前版本，格式为:主版本.次版本.增量版本-限定版本号-->*     
   26. ​    <version>1.0-SNAPSHOT</version>     
   27. ​    *<!--项目的名称, Maven产生的文档用-->*     
   28. ​    <name>banseon-maven</name>     
   29. ​    *<!--项目主页的URL, Maven产生的文档用-->*     
   30. ​    <url>http://www.baidu.com/banseon</url>     
   31. ​    *<!-- 项目的详细描述, Maven 产生的文档用。  当这个元素能够用HTML格式描述时（例如，CDATA中的文本会被解析器忽略，就可以包含HTML标 签）， 不鼓励使用纯文本描述。如果你需要修改产生的web站点的索引页面，你应该修改你自己的索引页文件，而不是调整这里的文档。-->*     
   32. ​    <description>A maven project to study maven.</description>     
   33. ​    *<!--描述了这个项目构建环境中的前提条件。-->*    
   34.  <prerequisites>    
   35.   *<!--构建该项目或使用该插件所需要的Maven的最低版本-->*    
   36. ​    <maven/>    
   37.  </prerequisites>    
   38.  *<!--项目的问题管理系统(Bugzilla, Jira, Scarab,或任何你喜欢的问题管理系统)的名称和URL，本例为 jira-->*     
   39. ​    <issueManagement>    
   40. ​     *<!--问题管理系统（例如jira）的名字，-->*     
   41. ​        <system>jira</system>     
   42. ​        *<!--该项目使用的问题管理系统的URL-->*    
   43. ​        <url>http://jira.baidu.com/banseon</url>     
   44. ​    </issueManagement>     
   45. ​    *<!--项目持续集成信息-->*    
   46.  <ciManagement>    
   47.   *<!--持续集成系统的名字，例如continuum-->*    
   48.   <system/>    
   49.   *<!--该项目使用的持续集成系统的URL（如果持续集成系统有web接口的话）。-->*    
   50.   <url/>    
   51.   *<!--构建完成时，需要通知的开发者/用户的配置项。包括被通知者信息和通知条件（错误，失败，成功，警告）-->*    
   52.   <notifiers>    
   53.    *<!--配置一种方式，当构建中断时，以该方式通知用户/开发者-->*    
   54.    <notifier>    
   55. ​    *<!--传送通知的途径-->*    
   56. ​    <type/>    
   57. ​    *<!--发生错误时是否通知-->*    
   58. ​    <sendOnError/>    
   59. ​    *<!--构建失败时是否通知-->*    
   60. ​    <sendOnFailure/>    
   61. ​    *<!--构建成功时是否通知-->*    
   62. ​    <sendOnSuccess/>    
   63. ​    *<!--发生警告时是否通知-->*    
   64. ​    <sendOnWarning/>    
   65. ​    *<!--不赞成使用。通知发送到哪里-->*    
   66. ​    <address/>    
   67. ​    *<!--扩展配置项-->*    
   68. ​    <configuration/>    
   69.    </notifier>    
   70.   </notifiers>    
   71.  </ciManagement>    
   72.  *<!--项目创建年份，4位数字。当产生版权信息时需要使用这个值。-->*    
   73. ​    <inceptionYear/>    
   74. ​    *<!--项目相关邮件列表信息-->*     
   75. ​    <mailingLists>    
   76. ​     *<!--该元素描述了项目相关的所有邮件列表。自动产生的网站引用这些信息。-->*     
   77. ​        <mailingList>     
   78. ​         *<!--邮件的名称-->*    
   79. ​            <name>Demo</name>     
   80. ​            *<!--发送邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建-->*     
   81. ​            <post>banseon@126.com</post>     
   82. ​            *<!--订阅邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建-->*     
   83. ​            <subscribe>banseon@126.com</subscribe>     
   84. ​            *<!--取消订阅邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建-->*     
   85. ​            <unsubscribe>banseon@126.com</unsubscribe>     
   86. ​            *<!--你可以浏览邮件信息的URL-->*    
   87. ​            <archive>http:/hi.baidu.com/banseon/demo/dev/</archive>     
   88. ​        </mailingList>     
   89. ​    </mailingLists>     
   90. ​    *<!--项目开发者列表-->*     
   91. ​    <developers>     
   92. ​     *<!--某个项目开发者的信息-->*    
   93. ​        <developer>     
   94. ​         *<!--SCM里项目开发者的唯一标识符-->*    
   95. ​            <id>HELLO WORLD</id>     
   96. ​            *<!--项目开发者的全名-->*    
   97. ​            <name>banseon</name>     
   98. ​            *<!--项目开发者的email-->*    
   99. ​            <email>banseon@126.com</email>     
   100. ​            *<!--项目开发者的主页的URL-->*    
   101. ​            <url/>    
   102. ​            *<!--项目开发者在项目中扮演的角色，角色元素描述了各种角色-->*    
   103. ​            <roles>     
   104. ​                <role>Project Manager</role>     
   105. ​                <role>Architect</role>     
   106. ​            </roles>    
   107. ​            *<!--项目开发者所属组织-->*    
   108. ​            <organization>demo</organization>     
   109. ​            *<!--项目开发者所属组织的URL-->*    
   110. ​            <organizationUrl>http://hi.baidu.com/banseon</organizationUrl>     
   111. ​            *<!--项目开发者属性，如即时消息如何处理等-->*    
   112. ​            <properties>     
   113. ​                <dept>No</dept>     
   114. ​            </properties>    
   115. ​            *<!--项目开发者所在时区， -11到12范围内的整数。-->*    
   116. ​            <timezone>-5</timezone>     
   117. ​        </developer>     
   118. ​    </developers>     
   119. ​    *<!--项目的其他贡献者列表-->*     
   120. ​    <contributors>    
   121. ​     *<!--项目的其他贡献者。参见developers/developer元素-->*    
   122. ​     <contributor>    
   123.    <name/><email/><url/><organization/><organizationUrl/><roles/><timezone/><properties/>    
   124. ​     </contributor>         
   125. ​    </contributors>       
   126. ​    *<!--该元素描述了项目所有License列表。 应该只列出该项目的license列表，不要列出依赖项目的 license列表。如果列出多个license，用户可以选择它们中的一个而不是接受所有license。-->*     
   127. ​    <licenses>    
   128. ​     *<!--描述了项目的license，用于生成项目的web站点的license页面，其他一些报表和validation也会用到该元素。-->*     
   129. ​        <license>    
   130. ​         *<!--license用于法律上的名称-->*    
   131. ​            <name>Apache 2</name>     
   132. ​            *<!--官方的license正文页面的URL-->*    
   133. ​            <url>http://www.baidu.com/banseon/LICENSE-2.0.txt</url>     
   134. ​            **<!--项目分发的主要方式：**    
   135. ​              *repo，可以从Maven库下载*    
   136. ​              *manual， 用户必须手动下载和安装依赖-->*    
   137. ​            <distribution>repo</distribution>     
   138. ​            *<!--关于license的补充信息-->*    
   139. ​            <comments>A business-friendly OSS license</comments>     
   140. ​        </license>     
   141. ​    </licenses>     
   142. ​    *<!--SCM(Source Control Management)标签允许你配置你的代码库，供Maven web站点和其它插件使用。-->*     
   143. ​    <scm>     
   144. ​        *<!--SCM的URL,该URL描述了版本库和如何连接到版本库。欲知详情，请看SCMs提供的URL格式和列表。该连接只读。-->*     
   145. ​        <connection>     
   146. ​            scm:svn:http://svn.baidu.com/banseon/maven/banseon/banseon-maven2-trunk(dao-trunk)      
   147. ​        </connection>     
   148. ​        *<!--给开发者使用的，类似connection元素。即该连接不仅仅只读-->*    
   149. ​        <developerConnection>     
   150. ​            scm:svn:http://svn.baidu.com/banseon/maven/banseon/dao-trunk      
   151. ​        </developerConnection>    
   152. ​        *<!--当前代码的标签，在开发阶段默认为HEAD-->*    
   153. ​        <tag/>           
   154. ​        *<!--指向项目的可浏览SCM库（例如ViewVC或者Fisheye）的URL。-->*     
   155. ​        <url>http://svn.baidu.com/banseon</url>     
   156. ​    </scm>     
   157. ​    *<!--描述项目所属组织的各种属性。Maven产生的文档用-->*     
   158. ​    <organization>     
   159. ​     *<!--组织的全名-->*    
   160. ​        <name>demo</name>     
   161. ​        *<!--组织主页的URL-->*    
   162. ​        <url>http://www.baidu.com/banseon</url>     
   163. ​    </organization>    
   164. ​    *<!--构建项目需要的信息-->*    
   165. ​    <build>    
   166. ​     *<!--该元素设置了项目源码目录，当构建项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。-->*    
   167.   <sourceDirectory/>    
   168.   *<!--该元素设置了项目脚本源码目录，该目录和源码目录不同：绝大多数情况下，该目录下的内容 会被拷贝到输出目录(因为脚本是被解释的，而不是被编译的)。-->*    
   169.   <scriptSourceDirectory/>    
   170.   *<!--该元素设置了项目单元测试使用的源码目录，当测试项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。-->*    
   171.   <testSourceDirectory/>    
   172.   *<!--被编译过的应用程序class文件存放的目录。-->*    
   173.   <outputDirectory/>    
   174.   *<!--被编译过的测试class文件存放的目录。-->*    
   175.   <testOutputDirectory/>    
   176.   *<!--使用来自该项目的一系列构建扩展-->*    
   177.   <extensions>    
   178.    *<!--描述使用到的构建扩展。-->*    
   179.    <extension>    
   180. ​    *<!--构建扩展的groupId-->*    
   181. ​    <groupId/>    
   182. ​    *<!--构建扩展的artifactId-->*    
   183. ​    <artifactId/>    
   184. ​    *<!--构建扩展的版本-->*    
   185. ​    <version/>    
   186.    </extension>    
   187.   </extensions>    
   188.   *<!--当项目没有规定目标（Maven2 叫做阶段）时的默认值-->*    
   189.   <defaultGoal/>    
   190.   *<!--这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在最终的打包文件里。-->*    
   191.   <resources>    
   192.    *<!--这个元素描述了项目相关或测试相关的所有资源路径-->*    
   193.    <resource>    
   194. ​    *<!-- 描述了资源的目标路径。该路径相对target/classes目录（例如${project.build.outputDirectory}）。举个例 子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为org/apache/maven /messages。然而，如果你只是想把资源放到源码目录结构里，就不需要该配置。-->*    
   195. ​    <targetPath/>    
   196. ​    *<!--是否使用参数值代替参数名。参数值取自properties元素或者文件里配置的属性，文件在filters元素里列出。-->*    
   197. ​    <filtering/>    
   198. ​    *<!--描述存放资源的目录，该路径相对POM路径-->*    
   199. ​    <directory/>    
   200. ​    *<!--包含的模式列表，例如**/\*.xml.-->*    
   201. ​    <includes/>    
   202. ​    *<!--排除的模式列表，例如**/\*.xml-->*    
   203. ​    <excludes/>    
   204.    </resource>    
   205.   </resources>    
   206.   *<!--这个元素描述了单元测试相关的所有资源路径，例如和单元测试相关的属性文件。-->*    
   207.   <testResources>    
   208.    *<!--这个元素描述了测试相关的所有资源路径，参见build/resources/resource元素的说明-->*    
   209.    <testResource>    
   210. ​    <targetPath/><filtering/><directory/><includes/><excludes/>    
   211.    </testResource>    
   212.   </testResources>    
   213.   *<!--构建产生的所有文件存放的目录-->*    
   214.   <directory/>    
   215.   *<!--产生的构件的文件名，默认值是${artifactId}-${version}。-->*    
   216.   <finalName/>    
   217.   *<!--当filtering开关打开时，使用到的过滤器属性文件列表-->*    
   218.   <filters/>    
   219.   *<!--子项目可以引用的默认插件信息。该插件配置项直到被引用时才会被解析或绑定到生命周期。给定插件的任何本地配置都会覆盖这里的配置-->*    
   220.   <pluginManagement>    
   221.    *<!--使用的插件列表 。-->*    
   222.    <plugins>    
   223. ​    *<!--plugin元素包含描述插件所需要的信息。-->*    
   224. ​    <plugin>    
   225. ​     *<!--插件在仓库里的group ID-->*    
   226. ​     <groupId/>    
   227. ​     *<!--插件在仓库里的artifact ID-->*    
   228. ​     <artifactId/>    
   229. ​     *<!--被使用的插件的版本（或版本范围）-->*    
   230. ​     <version/>    
   231. ​     *<!--是否从该插件下载Maven扩展（例如打包和类型处理器），由于性能原因，只有在真需要下载时，该元素才被设置成enabled。-->*    
   232. ​     <extensions/>    
   233. ​     *<!--在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。-->*    
   234. ​     <executions>    
   235. ​      *<!--execution元素包含了插件执行需要的信息-->*    
   236. ​      <execution>    
   237. ​       *<!--执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标-->*    
   238. ​       <id/>    
   239. ​       *<!--绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段-->*    
   240. ​       <phase/>    
   241. ​       *<!--配置的执行目标-->*    
   242. ​       <goals/>    
   243. ​       *<!--配置是否被传播到子POM-->*    
   244. ​       <inherited/>    
   245. ​       *<!--作为DOM对象的配置-->*    
   246. ​       <configuration/>    
   247. ​      </execution>    
   248. ​     </executions>    
   249. ​     *<!--项目引入插件所需要的额外依赖-->*    
   250. ​     <dependencies>    
   251. ​      *<!--参见dependencies/dependency元素-->*    
   252. ​      <dependency>    
   253. ​       ......    
   254. ​      </dependency>    
   255. ​     </dependencies>         
   256. ​     *<!--任何配置是否被传播到子项目-->*    
   257. ​     <inherited/>    
   258. ​     *<!--作为DOM对象的配置-->*    
   259. ​     <configuration/>    
   260. ​    </plugin>    
   261.    </plugins>    
   262.   </pluginManagement>    
   263.   *<!--使用的插件列表-->*    
   264.   <plugins>    
   265.    *<!--参见build/pluginManagement/plugins/plugin元素-->*    
   266.    <plugin>    
   267. ​    <groupId/><artifactId/><version/><extensions/>    
   268. ​    <executions>    
   269. ​     <execution>    
   270. ​      <id/><phase/><goals/><inherited/><configuration/>    
   271. ​     </execution>    
   272. ​    </executions>    
   273. ​    <dependencies>    
   274. ​     *<!--参见dependencies/dependency元素-->*    
   275. ​     <dependency>    
   276. ​      ......    
   277. ​     </dependency>    
   278. ​    </dependencies>    
   279. ​    <goals/><inherited/><configuration/>    
   280.    </plugin>    
   281.   </plugins>    
   282.  </build>    
   283.  *<!--在列的项目构建profile，如果被激活，会修改构建处理-->*    
   284.  <profiles>    
   285.   *<!--根据环境参数或命令行参数激活某个构建处理-->*    
   286.   <profile>    
   287.    *<!--构建配置的唯一标识符。即用于命令行激活，也用于在继承时合并具有相同标识符的profile。-->*    
   288.    <id/>    
   289.    **<!--自动触发profile的条件逻辑。Activation是profile的开启钥匙。profile的力量来自于它**    
   290.    *能够在某些特定的环境中自动使用某些特定的值；这些环境通过activation元素指定。activation元素并不是激活profile的唯一方式。-->*    
   291.    <activation>    
   292. ​    *<!--profile默认是否激活的标志-->*    
   293. ​    <activeByDefault/>    
   294. ​    *<!--当匹配的jdk被检测到，profile被激活。例如，1.4激活JDK1.4，1.4.0_2，而!1.4激活所有版本不是以1.4开头的JDK。-->*    
   295. ​    <jdk/>    
   296. ​    *<!--当匹配的操作系统属性被检测到，profile被激活。os元素可以定义一些操作系统相关的属性。-->*    
   297. ​    <os>    
   298. ​     *<!--激活profile的操作系统的名字-->*    
   299. ​     <name>Windows XP</name>    
   300. ​     *<!--激活profile的操作系统所属家族(如 'windows')-->*    
   301. ​     <family>Windows</family>    
   302. ​     *<!--激活profile的操作系统体系结构 -->*    
   303. ​     <arch>x86</arch>    
   304. ​     *<!--激活profile的操作系统版本-->*    
   305. ​     <version>5.1.2600</version>    
   306. ​    </os>    
   307. ​    **<!--如果Maven检测到某一个属性（其值可以在POM中通过${名称}引用），其拥有对应的名称和值，Profile就会被激活。如果值**    
   308. ​    *字段是空的，那么存在属性名称字段就会激活profile，否则按区分大小写方式匹配属性值字段-->*    
   309. ​    <property>    
   310. ​     *<!--激活profile的属性的名称-->*    
   311. ​     <name>mavenVersion</name>    
   312. ​     *<!--激活profile的属性的值-->*    
   313. ​     <value>2.0.3</value>    
   314. ​    </property>    
   315. ​    **<!--提供一个文件名，通过检测该文件的存在或不存在来激活profile。missing检查文件是否存在，如果不存在则激活**    
   316. ​    *profile。另一方面，exists则会检查文件是否存在，如果存在则激活profile。-->*    
   317. ​    <file>    
   318. ​     *<!--如果指定的文件存在，则激活profile。-->*    
   319. ​     <exists>/usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/</exists>    
   320. ​     *<!--如果指定的文件不存在，则激活profile。-->*    
   321. ​     <missing>/usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/</missing>    
   322. ​    </file>    
   323.    </activation>    
   324.    *<!--构建项目所需要的信息。参见build元素-->*    
   325.    <build>    
   326. ​    <defaultGoal/>    
   327. ​    <resources>    
   328. ​     <resource>    
   329. ​      <targetPath/><filtering/><directory/><includes/><excludes/>    
   330. ​     </resource>    
   331. ​    </resources>    
   332. ​    <testResources>    
   333. ​     <testResource>    
   334. ​      <targetPath/><filtering/><directory/><includes/><excludes/>    
   335. ​     </testResource>    
   336. ​    </testResources>    
   337. ​    <directory/><finalName/><filters/>    
   338. ​    <pluginManagement>    
   339. ​     <plugins>    
   340. ​      *<!--参见build/pluginManagement/plugins/plugin元素-->*    
   341. ​      <plugin>    
   342. ​       <groupId/><artifactId/><version/><extensions/>    
   343. ​       <executions>    
   344. ​        <execution>    
   345. ​         <id/><phase/><goals/><inherited/><configuration/>    
   346. ​        </execution>    
   347. ​       </executions>    
   348. ​       <dependencies>    
   349. ​        *<!--参见dependencies/dependency元素-->*    
   350. ​        <dependency>    
   351. ​         ......    
   352. ​        </dependency>    
   353. ​       </dependencies>    
   354. ​       <goals/><inherited/><configuration/>    
   355. ​      </plugin>    
   356. ​     </plugins>    
   357. ​    </pluginManagement>    
   358. ​    <plugins>    
   359. ​     *<!--参见build/pluginManagement/plugins/plugin元素-->*    
   360. ​     <plugin>    
   361. ​      <groupId/><artifactId/><version/><extensions/>    
   362. ​      <executions>    
   363. ​       <execution>    
   364. ​        <id/><phase/><goals/><inherited/><configuration/>    
   365. ​       </execution>    
   366. ​      </executions>    
   367. ​      <dependencies>    
   368. ​       *<!--参见dependencies/dependency元素-->*    
   369. ​       <dependency>    
   370. ​        ......    
   371. ​       </dependency>    
   372. ​      </dependencies>    
   373. ​      <goals/><inherited/><configuration/>    
   374. ​     </plugin>    
   375. ​    </plugins>    
   376.    </build>    
   377.    *<!--模块（有时称作子项目） 被构建成项目的一部分。列出的每个模块元素是指向该模块的目录的相对路径-->*    
   378.    <modules/>    
   379.    *<!--发现依赖和扩展的远程仓库列表。-->*    
   380.    <repositories>    
   381. ​    *<!--参见repositories/repository元素-->*    
   382. ​    <repository>    
   383. ​     <releases>    
   384. ​      <enabled/><updatePolicy/><checksumPolicy/>    
   385. ​     </releases>    
   386. ​     <snapshots>    
   387. ​      <enabled/><updatePolicy/><checksumPolicy/>    
   388. ​     </snapshots>    
   389. ​     <id/><name/><url/><layout/>    
   390. ​    </repository>    
   391.    </repositories>    
   392.    *<!--发现插件的远程仓库列表，这些插件用于构建和报表-->*    
   393.    <pluginRepositories>    
   394. ​    *<!--包含需要连接到远程插件仓库的信息.参见repositories/repository元素-->*        
   395. ​    <pluginRepository>    
   396. ​     <releases>    
   397. ​      <enabled/><updatePolicy/><checksumPolicy/>    
   398. ​     </releases>    
   399. ​     <snapshots>    
   400. ​      <enabled/><updatePolicy/><checksumPolicy/>    
   401. ​     </snapshots>    
   402. ​     <id/><name/><url/><layout/>    
   403. ​    </pluginRepository>    
   404.    </pluginRepositories>    
   405.    *<!--该元素描述了项目相关的所有依赖。 这些依赖组成了项目构建过程中的一个个环节。它们自动从项目定义的仓库中下载。要获取更多信息，请看项目依赖机制。-->*    
   406.    <dependencies>    
   407. ​    *<!--参见dependencies/dependency元素-->*    
   408. ​    <dependency>    
   409. ​     ......    
   410. ​    </dependency>    
   411.    </dependencies>    
   412.    *<!--不赞成使用. 现在Maven忽略该元素.-->*    
   413.    <reports/>       
   414.    *<!--该元素包括使用报表插件产生报表的规范。当用户执行“mvn site”，这些报表就会运行。 在页面导航栏能看到所有报表的链接。参见reporting元素-->*    
   415.    <reporting>    
   416. ​    ......    
   417.    </reporting>    
   418.    *<!--参见dependencyManagement元素-->*    
   419.    <dependencyManagement>    
   420. ​    <dependencies>    
   421. ​     *<!--参见dependencies/dependency元素-->*    
   422. ​     <dependency>    
   423. ​      ......    
   424. ​     </dependency>    
   425. ​    </dependencies>    
   426.    </dependencyManagement>    
   427.    *<!--参见distributionManagement元素-->*    
   428.    <distributionManagement>    
   429. ​    ......    
   430.    </distributionManagement>    
   431.    *<!--参见properties元素-->*    
   432.    <properties/>    
   433.   </profile>    
   434.  </profiles>    
   435.  *<!--模块（有时称作子项目） 被构建成项目的一部分。列出的每个模块元素是指向该模块的目录的相对路径-->*    
   436.  <modules/>    
   437. ​    *<!--发现依赖和扩展的远程仓库列表。-->*     
   438. ​    <repositories>     
   439. ​     *<!--包含需要连接到远程仓库的信息-->*    
   440. ​        <repository>    
   441. ​         *<!--如何处理远程仓库里发布版本的下载-->*    
   442. ​         <releases>    
   443. ​          *<!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->*    
   444. ​    <enabled/>    
   445. ​    *<!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。-->*    
   446. ​    <updatePolicy/>    
   447. ​    *<!--当Maven验证构件校验文件失败时该怎么做：ignore（忽略），fail（失败），或者warn（警告）。-->*    
   448. ​    <checksumPolicy/>    
   449.    </releases>    
   450.    *<!-- 如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的 策略。例如，可能有人会决定只为开发目的开启对快照版本下载的支持。参见repositories/repository/releases元素 -->*    
   451.    <snapshots>    
   452. ​    <enabled/><updatePolicy/><checksumPolicy/>    
   453.    </snapshots>    
   454.    *<!--远程仓库唯一标识符。可以用来匹配在settings.xml文件里配置的远程仓库-->*    
   455.    <id>banseon-repository-proxy</id>     
   456.    *<!--远程仓库名称-->*    
   457. ​            <name>banseon-repository-proxy</name>     
   458. ​            *<!--远程仓库URL，按protocol://hostname/path形式-->*    
   459. ​            <url>http://192.168.1.169:9999/repository/</url>     
   460. ​            *<!-- 用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然 而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。-->*    
   461. ​            <layout>default</layout>               
   462. ​        </repository>     
   463. ​    </repositories>    
   464. ​    *<!--发现插件的远程仓库列表，这些插件用于构建和报表-->*    
   465. ​    <pluginRepositories>    
   466. ​     *<!--包含需要连接到远程插件仓库的信息.参见repositories/repository元素-->*    
   467.   <pluginRepository>    
   468.    ......    
   469.   </pluginRepository>    
   470.  </pluginRepositories>    
   471. ​       
   472. ​    *<!--该元素描述了项目相关的所有依赖。 这些依赖组成了项目构建过程中的一个个环节。它们自动从项目定义的仓库中下载。要获取更多信息，请看项目依赖机制。-->*     
   473. ​    <dependencies>     
   474. ​        <dependency>    
   475.    *<!--依赖的group ID-->*    
   476. ​            <groupId>org.apache.maven</groupId>     
   477. ​            *<!--依赖的artifact ID-->*    
   478. ​            <artifactId>maven-artifact</artifactId>     
   479. ​            *<!--依赖的版本号。 在Maven 2里, 也可以配置成版本号的范围。-->*    
   480. ​            <version>3.8.1</version>     
   481. ​            *<!-- 依赖类型，默认类型是jar。它通常表示依赖的文件的扩展名，但也有例外。一个类型可以被映射成另外一个扩展名或分类器。类型经常和使用的打包方式对应， 尽管这也有例外。一些类型的例子：jar，war，ejb-client和test-jar。如果设置extensions为 true，就可以在 plugin里定义新的类型。所以前面的类型的例子不完整。-->*    
   482. ​            <type>jar</type>    
   483. ​            *<!-- 依赖的分类器。分类器可以区分属于同一个POM，但不同构建方式的构件。分类器名被附加到文件名的版本号后面。例如，如果你想要构建两个单独的构件成 JAR，一个使用Java 1.4编译器，另一个使用Java 6编译器，你就可以使用分类器来生成两个单独的JAR构件。-->*    
   484. ​            <classifier></classifier>    
   485. ​            **<!--依赖范围。在项目发布过程中，帮助决定哪些构件被包括进来。欲知详情请参考依赖机制。**    
   486. ​                *- compile ：默认范围，用于编译*      
   487. ​                *- provided：类似于编译，但支持你期待jdk或者容器提供，类似于classpath*      
   488. ​                *- runtime: 在执行时需要使用*      
   489. ​                *- test:    用于test任务时使用*      
   490. ​                *- system: 需要外在提供相应的元素。通过systemPath来取得*      
   491. ​                *- systemPath: 仅用于范围为system。提供相应的路径*      
   492. ​                *- optional:   当项目自身被依赖时，标注依赖是否传递。用于连续依赖时使用-->*     
   493. ​            <scope>test</scope>       
   494. ​            *<!--仅供system范围使用。注意，不鼓励使用这个元素，并且在新的版本中该元素可能被覆盖掉。该元素为依赖规定了文件系统上的路径。需要绝对路径而不是相对路径。推荐使用属性匹配绝对路径，例如${java.home}。-->*    
   495. ​            <systemPath></systemPath>     
   496. ​            *<!--当计算传递依赖时， 从依赖构件列表里，列出被排除的依赖构件集。即告诉maven你只依赖指定的项目，不依赖项目的依赖。此元素主要用于解决版本冲突问题-->*    
   497. ​            <exclusions>    
   498. ​             <exclusion>     
   499. ​                    <artifactId>spring-core</artifactId>     
   500. ​                    <groupId>org.springframework</groupId>     
   501. ​                </exclusion>     
   502. ​            </exclusions>       
   503. ​            *<!--可选依赖，如果你在项目B中把C依赖声明为可选，你就需要在依赖于B的项目（例如项目A）中显式的引用对C的依赖。可选依赖阻断依赖的传递性。-->*     
   504. ​            <optional>true</optional>    
   505. ​        </dependency>    
   506. ​    </dependencies>    
   507. ​    *<!--不赞成使用. 现在Maven忽略该元素.-->*    
   508. ​    <reports></reports>    
   509. ​    *<!--该元素描述使用报表插件产生报表的规范。当用户执行“mvn site”，这些报表就会运行。 在页面导航栏能看到所有报表的链接。-->*    
   510.  <reporting>    
   511.   *<!--true，则，网站不包括默认的报表。这包括“项目信息”菜单中的报表。-->*    
   512.   <excludeDefaults/>    
   513.   *<!--所有产生的报表存放到哪里。默认值是${project.build.directory}/site。-->*    
   514.   <outputDirectory/>    
   515.   *<!--使用的报表插件和他们的配置。-->*    
   516.   <plugins>    
   517.    *<!--plugin元素包含描述报表插件需要的信息-->*    
   518.    <plugin>    
   519. ​    *<!--报表插件在仓库里的group ID-->*    
   520. ​    <groupId/>    
   521. ​    *<!--报表插件在仓库里的artifact ID-->*    
   522. ​    <artifactId/>    
   523. ​    *<!--被使用的报表插件的版本（或版本范围）-->*    
   524. ​    <version/>    
   525. ​    *<!--任何配置是否被传播到子项目-->*    
   526. ​    <inherited/>    
   527. ​    *<!--报表插件的配置-->*    
   528. ​    <configuration/>    
   529. ​    *<!--一组报表的多重规范，每个规范可能有不同的配置。一个规范（报表集）对应一个执行目标 。例如，有1，2，3，4，5，6，7，8，9个报表。1，2，5构成A报表集，对应一个执行目标。2，5，8构成B报表集，对应另一个执行目标-->*    
   530. ​    <reportSets>    
   531. ​     *<!--表示报表的一个集合，以及产生该集合的配置-->*    
   532. ​     <reportSet>    
   533. ​      *<!--报表集合的唯一标识符，POM继承时用到-->*    
   534. ​      <id/>    
   535. ​      *<!--产生报表集合时，被使用的报表的配置-->*    
   536. ​      <configuration/>    
   537. ​      *<!--配置是否被继承到子POMs-->*    
   538. ​      <inherited/>    
   539. ​      *<!--这个集合里使用到哪些报表-->*    
   540. ​      <reports/>    
   541. ​     </reportSet>    
   542. ​    </reportSets>    
   543.    </plugin>    
   544.   </plugins>    
   545.  </reporting>    
   546.  *<!-- 继承自该项目的所有子项目的默认依赖信息。这部分的依赖信息不会被立即解析,而是当子项目声明一个依赖（必须描述group ID和 artifact ID信息），如果group ID和artifact ID以外的一些信息没有描述，则通过group ID和artifact ID 匹配到这里的依赖，并使用这里的依赖信息。-->*    
   547.  <dependencyManagement>    
   548.   <dependencies>    
   549.    *<!--参见dependencies/dependency元素-->*    
   550.    <dependency>    
   551. ​    ......    
   552.    </dependency>    
   553.   </dependencies>    
   554.  </dependencyManagement>       
   555. ​    *<!--项目分发信息，在执行mvn deploy后表示要发布的位置。有了这些信息就可以把网站部署到远程服务器或者把构件部署到远程仓库。-->*     
   556. ​    <distributionManagement>    
   557. ​        *<!--部署项目产生的构件到远程仓库需要的信息-->*    
   558. ​        <repository>    
   559. ​         *<!--是分配给快照一个唯一的版本号（由时间戳和构建流水号）？还是每次都使用相同的版本号？参见repositories/repository元素-->*    
   560.    <uniqueVersion/>    
   561.    <id>banseon-maven2</id>     
   562.    <name>banseon maven2</name>     
   563. ​            <url>file://${basedir}/target/deploy</url>     
   564. ​            <layout/>    
   565.   </repository>    
   566.   *<!--构件的快照部署到哪里？如果没有配置该元素，默认部署到repository元素配置的仓库，参见distributionManagement/repository元素-->*     
   567.   <snapshotRepository>    
   568.    <uniqueVersion/>    
   569.    <id>banseon-maven2</id>    
   570. ​            <name>Banseon-maven2 Snapshot Repository</name>    
   571. ​            <url>scp://svn.baidu.com/banseon:/usr/local/maven-snapshot</url>     
   572.    <layout/>    
   573.   </snapshotRepository>    
   574.   *<!--部署项目的网站需要的信息-->*     
   575. ​        <site>    
   576. ​         *<!--部署位置的唯一标识符，用来匹配站点和settings.xml文件里的配置-->*     
   577. ​            <id>banseon-site</id>     
   578. ​            *<!--部署位置的名称-->*    
   579. ​            <name>business api website</name>     
   580. ​            *<!--部署位置的URL，按protocol://hostname/path形式-->*    
   581. ​            <url>     
   582. ​                scp://svn.baidu.com/banseon:/var/www/localhost/banseon-web      
   583. ​            </url>     
   584. ​        </site>    
   585.   *<!--项目下载页面的URL。如果没有该元素，用户应该参考主页。使用该元素的原因是：帮助定位那些不在仓库里的构件（由于license限制）。-->*    
   586.   <downloadUrl/>    
   587.   *<!--如果构件有了新的group ID和artifact ID（构件移到了新的位置），这里列出构件的重定位信息。-->*    
   588.   <relocation>    
   589.    *<!--构件新的group ID-->*    
   590.    <groupId/>    
   591.    *<!--构件新的artifact ID-->*    
   592.    <artifactId/>    
   593.    *<!--构件新的版本号-->*    
   594.    <version/>    
   595.    *<!--显示给用户的，关于移动的额外信息，例如原因。-->*    
   596.    <message/>    
   597.   </relocation>    
   598.   *<!-- 给出该构件在远程仓库的状态。不得在本地项目中设置该元素，因为这是工具自动更新的。有效的值有：none（默认），converted（仓库管理员从 Maven 1 POM转换过来），partner（直接从伙伴Maven 2仓库同步过来），deployed（从Maven 2实例部 署），verified（被核实时正确的和最终的）。-->*    
   599.   <status/>           
   600. ​    </distributionManagement>    
   601. ​    *<!--以值替代名称，Properties可以在整个POM中使用，也可以作为触发条件（见settings.xml配置文件里activation元素的说明）。格式是<name>value</name>。-->*    
   602. ​    <properties/>    
   603. </project>  </span>
   ```

   