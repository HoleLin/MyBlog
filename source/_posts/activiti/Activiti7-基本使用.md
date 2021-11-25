---
title: Activiti7-基本使用
date: 2021-11-20 22:02:08
index_img: /img/cover/Activiti.png
cover: /img/cover/Activiti.png
tags:
categories:
- Activiti7
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [Activiti官网文档](https://www.activiti.org/userguide/#_getting_started)

### 基本概念

#### XML

* ##### Process

  * **id**：此属性是**必需的，**并且映射到Activiti对象的**key**属性`ProcessDefinition`。
    * 这里需要注意的重要一点是，这与调用 `startProcessInstanceById`方法不同。该方法需要 Activiti 引擎在部署时生成的 String id，可以通过调用该`processDefinition.getId()`方法来检索。生成的id格式为**key:version**，长度限制为**64个字符**。如果您收到`ActivitiException`说明生成的 id 太长的信息
  * **name**：此属性是**可选的**，映射到 a 的*name*属性`ProcessDefinition`。

* ##### Event

  * 事件用于对生命周期过程中发生的事情进行建模。事件总是可视化为一个圆圈。在 BPMN 2.0 中，有两个主要的事件类别：*捕获*或*抛出*事件。

    - **捕获：**当流程执行到达事件时，它将等待触发发生。触发器的类型由内部图标或 XML 中的类型声明定义。通过未填充的内部图标（即白色），捕捉事件与投掷事件在视觉上有所区别。

      

    - **抛出：**当流程执行到达事件时，触发触发器。触发器的类型由内部图标或 XML 中的类型声明定义。投掷事件通过填充黑色的内部图标在视觉上与捕获事件区分开来。

  * ###### Timer Event

    * 定时器事件是由定义的定时器触发的事件。它们可以用作start event开始事件、intermediate event中间事件或boundary event边界事件。时间事件的行为取决于所使用的业务日历。每个定时器事件都有一个默认的业务日历，但也可以在定时器事件定义上定义业务日历

    ```xml
    <timerEventDefinition activiti:businessCalendarName="custom">
        <timeDate>2011-03-11T12:13:14</timeDate>
        <timeDuration>P10D</timeDuration>
        <timeCycle activiti:endDate="2015-02-25T16:42:11+00:00">R3/PT10H</timeCycle>
    </timerEventDefinition>
    ```

  * ###### Error Event

    ```xml
    <endEvent id="myErrorEndEvent">
      <errorEventDefinition errorRef="myError" />
    </endEvent>
    ```

  * ######  Signal Event 

    * 信号事件是引用命名信号的事件。信号是一个全局范围的事件（广播语义），并被传递给所有活动的处理程序（等待流程实例/捕获信号事件）。

    ```xml
    <definitions... >
    	<!-- declaration of the signal -->
    	<signal id="alertSignal" name="alert" />
    
    	<process id="catchSignal">
    		<intermediateThrowEvent id="throwSignalEvent" name="Alert">
    			<!-- signal event definition -->
    			<signalEventDefinition signalRef="alertSignal" />
    		</intermediateThrowEvent>
    		...
    		<intermediateCatchEvent id="catchSignalEvent" name="On Alert">
    			<!-- signal event definition -->
    			<signalEventDefinition signalRef="alertSignal" />
    		</intermediateCatchEvent>
    		...
    	</process>
    </definitions>
    ```

  * ###### Message Event

    * 消息事件是引用命名消息的事件。消息具有名称和有效载荷。与信号不同，消息事件总是针对单个接收者。

    ```xml
    <definitions id="definitions"
      xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
      xmlns:activiti="http://activiti.org/bpmn"
      targetNamespace="Examples"
      xmlns:tns="Examples">
    
      <message id="newInvoice" name="newInvoiceMessage" />
      <message id="payment" name="paymentMessage" />
    
      <process id="invoiceProcess">
    
        <startEvent id="messageStart" >
        	<messageEventDefinition messageRef="newInvoice" />
        </startEvent>
        ...
        <intermediateCatchEvent id="paymentEvt" >
        	<messageEventDefinition messageRef="payment" />
        </intermediateCatchEvent>
        ...
      </process>
    
    </definitions>
    ```

    

#### 流程引擎:**ProcessEngines**

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

ProcessEngineConfiguration.createProcessEngineConfigurationFromResourceDefault();

ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource);

ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource, String beanName);

ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream);

ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName);

ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();

ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
```

```
ProcessEngine processEngine = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration()
  .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE)
  .setJdbcUrl("jdbc:h2:mem:my-own-db;DB_CLOSE_DELAY=1000")
  .setAsyncExecutorActivate(false)
  .buildProcessEngine();
```

- **org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration**：流程引擎以独立方式使用。Activiti 将处理交易。默认情况下，只有在引擎启动时才会检查数据库（如果没有 Activiti schema 或 schema 版本不正确会抛出异常）。
- **org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration**：这是一个用于单元测试的便利类。Activiti 将处理交易。默认使用 H2 内存数据库。数据库将在引擎启动和关闭时创建和删除。使用它时，可能不需要额外的配置（除非使用例如作业执行程序或邮件功能）。
- **org.activiti.spring.SpringProcessEngineConfiguration**：在 Spring 环境中使用流程引擎时使用。
- **org.activiti.engine.impl.cfg.JtaProcessEngineConfiguration**：当引擎以独立模式运行时使用，带有 JTA 事务。

#### 配置数据库信息

- **jdbcUrl**：数据库的 JDBC URL。
- **jdbcDriver**：特定数据库类型的驱动程序的实现。
- **jdbcUsername** : 连接到数据库的用户名。
- **jdbcPassword**：连接数据库的密码。
- **jdbcMaxActiveConnections**：连接池在任何时候最多可以包含的活动连接数。默认值为 10。
- **jdbcMaxIdleConnections**：连接池在任何时候最多可以包含的空闲连接数。
- **jdbcMaxCheckoutTime**：在强制返回连接之前可以从连接池中*检出*连接的时间量（以毫秒为单位）。默认值为 20000（20 秒）。
- **jdbcMaxWaitTime**：这是一个低级设置，它使池有机会打印日志状态并在花费异常长的情况下重新尝试获取连接（以避免在池配置错误时永远静默失败）默认是 20000（20 秒）。
- **databaseType**：通常不需要指定此属性，因为它是从数据库连接元数据中自动分析的。仅应在自动检测失败的情况下指定。可能的值：{h2、mysql、oracle、postgres、mssql、db2}。此设置将确定将使用哪些创建/删除脚本和查询。
- **databaseSchemaUpdate**：允许设置在流程引擎启动和关闭时处理数据库架构的策略。
  - `false` （默认）：在创建流程引擎时根据库检查数据库模式的版本，如果版本不匹配则抛出异常。
  - `true`：在构建流程引擎时，会执行检查并在必要时执行架构更新。如果架构不存在，则会创建它。
  - `create-drop`：在创建流程引擎时创建模式，并在关闭流程引擎时删除模式。

#### 数据库表名说明

Activiti 的数据库名称都以**ACT_ 开头**。第二部分是表格用例的双字符标识。这个用例也将大致匹配服务 API。

- **ACT_RE_** *：*RE*代表`repository`. 带有此前缀的表包含*静态*信息，例如流程定义和流程资源（图像、规则等）。
- **ACT_RU_** *：*RU*代表`runtime`. 这些是运行时表，包含流程实例、用户任务、变量、作业等的运行时数据。Activiti 只在流程实例执行期间存储运行时数据，并在流程实例结束时删除记录。这使运行时表既小又快。
- **ACT_ID_** *：*ID*代表`identity`。这些表包含身份信息，例如用户、组等。
- **ACT_HI_** *：*HI*代表`history`. 这些是包含历史数据的表，例如过去的流程实例、变量、任务等。
- **ACT_GE_** *：`general`数据，用于各种用例。

#### 监听器

* 调度的所有事件都是 的子类型`org.activiti.engine.delegate.event.ActivitiEvent`。该事件自曝（如果有的话）`type`，`executionId`，`processInstanceId`和`processDefinitionId`。某些事件包含与发生的事件相关的附加上下文.

* 事件侦听器的唯一要求是实现`org.activiti.engine.delegate.event.ActivitiEventListener`. 

* 该`isFailOnException()`方法确定在`onEvent(..)`调度事件时该方法抛出异常时的行为。如果`false`返回，则忽略异常。当`true`返回，异常不会被忽略和冒泡，未能有效地与当前进行命令。如果事件是 API 调用（或任何其他事务性操作，例如作业执行）的一部分，则事务将被回滚。如果事件监听器中的行为不是关键业务，建议返回`false`。

* **org.activiti.engine.delegate.event.BaseEntityEventListener**：一个事件侦听器基类，可用于侦听特定类型实体或所有实体的实体相关事件。它隐藏了类型检查并提供了 4 种应该被覆盖的方法：`onCreate(..)`，`onUpdate(..)`以及`onDelete(..)`何时创建、更新或删除实体。对于所有其他实体相关事件，`onEntityEvent(..) `调用 。

* 可以使用 API ( `RuntimeService`)向引擎添加和删除其他事件侦听器：

  ```java
  void addEventListener(ActivitiEventListener listenerToAdd);
  
  void addEventListener(ActivitiEventListener listenerToAdd, ActivitiEventType... types);
  
  void removeEventListener(ActivitiEventListener listenerToRemove);
  ```

##### 将侦听器添加到流程定义

* 下面的代码段向流程定义添加了 2 个侦听器。第一个侦听器将接收任何类型的事件，具有基于完全限定类名的侦听器实现。第二个侦听器仅在作业成功执行或失败时收到通知，使用已在`beans`流程引擎配置的属性中定义的侦听器。

  ```xml
  <process id="testEventListeners">
    <extensionElements>
      <activiti:eventListener class="org.activiti.engine.test.MyEventListener" />
      <activiti:eventListener delegateExpression="${testEventListener}" events="JOB_EXECUTION_SUCCESS,JOB_EXECUTION_FAILURE" />
    </extensionElements>
  
    ...
  
  </process>
  ```

* 对于与实体相关的事件，还可以将侦听器添加到流程定义中，这些侦听器仅在特定实体类型的实体事件发生时收到通知。下面的代码片段展示了如何实现这一点。它可以用于所有实体事件（第一个示例）或仅用于特定事件类型（第二个示例）。

  ```xml
  <process id="testEventListeners">
    <extensionElements>
      <activiti:eventListener class="org.activiti.engine.test.MyEventListener" entityType="task" />
      <activiti:eventListener delegateExpression="${testEventListener}" events="ENTITY_CREATED" entityType="task" />
    </extensionElements>
  
    ...
  
  </process>
  ```

* 支持的值为`entityType`：`attachment`、`comment`、`execution`、`identity-link`、`job`、`process-instance`、`process-definition`、`task`。

#### 启动流程

```java
public static void main(String[] args) {

  // Create Activiti process engine
  ProcessEngine processEngine = ProcessEngineConfiguration
    .createStandaloneProcessEngineConfiguration()
    .buildProcessEngine();

  // Get Activiti services
  RepositoryService repositoryService = processEngine.getRepositoryService();
  RuntimeService runtimeService = processEngine.getRuntimeService();

  // Deploy the process definition
  repositoryService.createDeployment()
    .addClasspathResource("FinancialReportProcess.bpmn20.xml")
    .deploy();

  // Start a process instance
  runtimeService.startProcessInstanceByKey("financialReport");
}
```

```java
public class TenMinuteTutorial {

  public static void main(String[] args) {

    // Create Activiti process engine
    ProcessEngine processEngine = ProcessEngineConfiguration
      .createStandaloneProcessEngineConfiguration()
      .buildProcessEngine();

    // Get Activiti services
    RepositoryService repositoryService = processEngine.getRepositoryService();
    RuntimeService runtimeService = processEngine.getRuntimeService();

    // Deploy the process definition
    repositoryService.createDeployment()
      .addClasspathResource("FinancialReportProcess.bpmn20.xml")
      .deploy();

    // Start a process instance
    String procId = runtimeService.startProcessInstanceByKey("financialReport").getId();

    // Get the first task
    TaskService taskService = processEngine.getTaskService();
    List<Task> tasks = taskService.createTaskQuery().taskCandidateGroup("accountancy").list();
    for (Task task : tasks) {
      System.out.println("Following task is available for accountancy group: " + task.getName());

      // claim it
      taskService.claim(task.getId(), "fozzie");
    }

    // Verify Fozzie can now retrieve the task
    tasks = taskService.createTaskQuery().taskAssignee("fozzie").list();
    for (Task task : tasks) {
      System.out.println("Task for fozzie: " + task.getName());

      // Complete the task
      taskService.complete(task.getId());
    }

    System.out.println("Number of tasks for fozzie: "
            + taskService.createTaskQuery().taskAssignee("fozzie").count());

    // Retrieve and claim the second task
    tasks = taskService.createTaskQuery().taskCandidateGroup("management").list();
    for (Task task : tasks) {
      System.out.println("Following task is available for management group: " + task.getName());
      taskService.claim(task.getId(), "kermit");
    }

    // Completing the second task ends the process
    for (Task task : tasks) {
      taskService.complete(task.getId());
    }

    // verify that the process is actually finished
    HistoryService historyService = processEngine.getHistoryService();
    HistoricProcessInstance historicProcessInstance =
      historyService.createHistoricProcessInstanceQuery().processInstanceId(procId).singleResult();
    System.out.println("Process instance end time: " + historicProcessInstance.getEndTime());
  }

}
```

