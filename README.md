#规则引擎的使用
####1.引入规则引擎包

    <dependency>
      <groupId>com.weijinsuo.ruleengine</groupId>
      <artifactId>weijinsuo-ruleengine</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>

####2.配置文件(*.properties)中添加配置
    
    #规则引擎   (!!注意:文件路径不支持  classpath:*)
    # Windows Style
    #resource.rules.config=file:/D:/Files/Rule/RuleConfig/rules.xml
    #groovy.script.packagePath=file:/D:/Files/Rule/groovyRules/
    # Linux Style
    resource.rules.config=file:/Users/guolanrui/zw_workspace/project/weijinsuo-rules-manage/src/main/resources/rules/rules.xml
    groovy.script.packagePath=file:/Users/guolanrui/zw_workspace/project/weijinsuo-rules-manage/src/main/resources/rules/rulefiles
    # HTTP Style
    #resource.rules.config=http://localhost/files/rules.xml
    #groovy.script.packagePath=http://localhost/files/

>`resource.rules.config` 为规则加载文件`rules.xml`路径

>`groovy.script.packagePath` 为规则文件`*.groovy`的存放目录

####3.添加规则加载文件（rules.xml）

>内容如下：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xmlns:lang="http://www.springframework.org/schema/lang" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xsi:schemaLocation="
			http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
			http://www.springframework.org/schema/lang http://www.springframework.org/schema/lang/spring-lang.xsd">
	
	    <context:property-placeholder location="classpath:config-test.properties" />
	    <context:annotation-config />

	    <lang:groovy id="TestRules" refresh-check-delay="2000"
	                 script-source="${groovy.script.packagePath}/TestRules.groovy" />
	</beans>
	
>`<context:property-placeholder location="classpath:config-test.properties" />`中的`location`为2中的配置文件。

>`<lang:groovy id="TestRules" refresh-check-delay="2000"
                 script-source="${groovy.script.packagePath}/TestRules.groovy" />`中的`id`为beanid，规则调用时的`rulename`；`refresh-check-delay`为规则文件`*.groovy`的刷新间隔，单位为毫秒；`script-source`为该规则对应的规则文件的路径。
                 
####4.修改spring配置文件（applicationContext.xml）
>在`beans`标签概要描述中添加如下：

	xmlns:task="http://www.springframework.org/schema/task"
	xsi:schemaLocation="http://www.springframework.org/schema/task
                http://www.springframework.org/schema/task/spring-task.xsd"
                       
>在`<context:component-scan />`标签的 `base-package`属性中添加`com.weijinsuo.ruleengine.engine`将规则引擎载入spring，代码如下：

	<context:component-scan base-package="com.weijinsuo.ruleengine.engine" />


>添加`<task:annotation-driven />`标签，改标签为Spring Task的配置，用于定时检查规则配置文件的变更

####5.编写规则文件
-	**需要实现`com.weijinsuo.ruleengine.engine.BaseRule`接口，规则逻辑处理在`call`方法中实现**

>事例代码如下:

	import com.weijinsuo.ruleengine.engine.BaseRule
	
	class TestRules implements BaseRule {
	    @Override
	    Map call(Map map) {
	        def say = map.get("say");
	        println(say);
	        def returnVal = "Return Value";
	        
	        def returnMap = new HashMap();
	        returnMap.put("val",returnVal);
	
	        return returnMap;
	    }
	}
- 规则开发测试完成后，将规则配置到`rules.xml`，规则自动装载。

>`rules.xml`中添加代码如下：

    <lang:groovy id="TestRules" refresh-check-delay="2000"
                 script-source="${groovy.script.packagePath}/TestRules.groovy" />
                 
####6.规则调用
- 注入规则引擎，代码如下：

		@Autowired
		private RuleEngine ruleEngine;
- 通过`ruleEngine.call(rulename,map)`调用规则,代码如下：

	    @Test
	    public void testRuleEngine(){
	        Map<String,String> map = new HashMap();
	        map.put("say","Hello World");
	        String rulename = "TestRules";
	        Map returnMap = ruleEngine.call(rulename,map);
	        System.out.println("规则返回值:"+returnMap.get("val"));
	    }

