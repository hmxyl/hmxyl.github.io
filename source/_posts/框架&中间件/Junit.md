---
title: Junit日常使用记录
date: 2022-10-30 15:39:23
tags: 
  -  Junit
categories: 
  - [框架&中间件, Junit]
description: 'Junit日常使用记录'
---

# 使用Spring配合Junit进行单元测试的总结

##  jar包导入

```xml
<dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-test</artifactId>
     <version>3.2.4.RELEASE</version>
     <scope>test</scope>
</dependency>
```



##  直接对spring中注入的bean进行测试(以DAO为例)

在测试类上添加

 ```java
@RunWith 注解指定使用springJunit的测试运行器
@ContextConfiguration 注解指定测试用的spring配置文件的位置
 ```

 之后我们就可以注入我们需要测试的bean进行测试。Junit在运行测试之前会先解析spring的配置文件,初始化spring中配置的bean

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath*:spring-config-test.xml" })
public class TestProjectDao {

  @Autowired
  ProjectDao projectDao;

  @Test
  public void testCreateProjectCode() {
    long applyTime = System.currentTimeMillis();
    Timestamp ts = new Timestamp(applyTime);
    Map codeMap = projectDao.generateCode("5", "8", ts, "院内");
    String projectCode = (String) codeMap.get("_project_code");
    Timestamp apply_time = (Timestamp) codeMap.get("_apply_time");
    System.out.print(projectCode);
    System.out.print(apply_time.toString());
    Assert.assertTrue(projectCode.length() == 12);
  }
}
```

##  对SpringMVC进行测试

Spring3.2之后出现了org.springframework.test.web.servlet.MockMvc 类,对springMVC单元测试进行支持。样例如下：

```java
package com.jiaoyiping.baseproject;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

import com.jiaoyiping.baseproject.privilege.controller.MeunController;
import com.jiaoyiping.baseproject.training.bean.Person;
import junit.framework.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.servlet.ModelAndView;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
//@ContextConfiguration(classes = {WebMvcConfig.class, MockDataConfig.class})
@ContextConfiguration(
  locations = {
    "classpath:/spring/applicationContext.xml",
    "classpath*:mvc-dispatcher-servlet.xml",
  }
)
public class TestMockMvc {

  @Autowired
  private org.springframework.web.context.WebApplicationContext context;

  MockMvc mockMvc;

  @Before
  public void before() {
    //可以对所有的controller来进行测试
    mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
    //仅仅对单个Controller来进行测试
    // mockMvc = MockMvcBuilders.standaloneSetup(new MeunController()).build();
  }

  @Test
  public void testGetMenu() {
    try {
      System.out.println("----------------------------");
      ResultActions actions = this.mockMvc.perform(get("/menu/manage.action"));
      System.out.println(status());
      // System.out.println(content().toString());
      actions.andExpect(status().isOk());
      // actions.andExpect(content().contentType("text/html"));
      System.out.println("----------------------------");
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  //从controller里直接增加用户(用POST的方式)
  //post("路径").param("属性名","属性值"); 用这种方法来构造POST
  @Test
  public void addPerson() {
    try {
      ResultActions resultActions =
        this.mockMvc.perform(
            post("/person/add")
              .param("name", "用友软件")
              .param("age", "23")
              .param("address", "北京市永丰屯")
          );
      resultActions.andExpect(status().isOk());
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  //得到Controller层返回的ModelAndView的方法：resultActions.andReturn().getModelAndView().getModel().get("person");
  @Test
  public void getPerson() {
    String id = "297e5fb648b0e6d30148b0e6da6d0000";
    try {
      ResultActions resultActions =
        this.mockMvc.perform(post("/person/toEditPerson").param("id", id))
          .andExpect(status().isOk());
      Person person = (Person) (
        resultActions.andReturn().getModelAndView().getModel().get("person")
      );
      Assert.assertEquals(23, person.getAge());
      System.out.println(person.getId());
      System.out.println(person.getName());
      System.out.println(person.getAge());
      System.out.println(person.getAddress());
      Assert.assertEquals(23, person.getAge());
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```

##  测试RestEasy提供的接口(当使用restEasy提供的rest类型接口的时候会用到)

RestEasy提供了 org.jboss.resteasy.core.Dispatcher类来模拟http请求，并返回数据。这样,在测试接口的时候就不必启动容器了
代码如下

```java
package cn.cmri.pds.controller;

import cn.cmri.pds.project.controllor.ProjectTagControllor;
import cn.cmri.pds.project.service.ProjectTagService;
import java.net.URISyntaxException;
import javax.servlet.http.HttpServletResponse;
import org.jboss.resteasy.core.Dispatcher;
import org.jboss.resteasy.mock.MockDispatcherFactory;
import org.jboss.resteasy.mock.MockHttpRequest;
import org.jboss.resteasy.mock.MockHttpResponse;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath*:spring-config-test.xml" })
public class TestProjectTagController {

  @Autowired
  ProjectTagService projectTagService;

  Dispatcher dispatcher;

  @Before
  public void before() {
    ProjectTagControllor projectTagControllor = new ProjectTagControllor();
    projectTagControllor.setProjectTagService(projectTagService);
    dispatcher = MockDispatcherFactory.createDispatcher();
    dispatcher.getRegistry().addSingletonResource(projectTagControllor);
  }

  @Test
  public void testProjectTags() throws URISyntaxException {
    MockHttpRequest request = MockHttpRequest.get("/rest/project/123456/tags");
    MockHttpResponse response = new MockHttpResponse();
    dispatcher.invoke(request, response);
    Assert.assertEquals(HttpServletResponse.SC_NOT_FOUND, response.getStatus());
    Assert.assertEquals("指定的项目不存在", response.getContentAsString());
  }
}
```

