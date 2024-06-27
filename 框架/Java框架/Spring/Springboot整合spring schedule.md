# Springboot开启定时任务Spring Schedule(内涵业务场景代码)

## 一、简介

**Spring Schedule**是Spring框架提供的一种轻量级的任务调度框架，可以**用来执行定时任务和周期性任务**。在很多应用场景中，我们需要定时执行某些任务，**比如定时备份数据库、定时清理缓存、定时发送邮件等等**。Spring Schedule提供了很方便的任务调度解决方案，可以很容易地实现这些定时任务。

Spring Boot是Spring框架的一个子项目，它提供了很多开箱即用的特性，可以帮助我们更快速、更便捷地开发Spring应用。在Spring Boot应用中整合Spring Schedule也非常简单，本文将介绍如何在Spring Boot中整合Spring Schedule。

## 二、添加依赖

引入springboot依赖

## 三、配置定时任务

在Spring Boot中配置定时任务有两种方式：基于注解和基于XML配置。我们这里选择基于注解的方式。

首先，**在启动类上添加@EnableScheduling注解**，启用Spring Schedule：

```java
@SpringBootApplication
@EnableScheduling
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

然后，在需要执行定时任务的方法上添加@Scheduled注解，指定定时任务的执行时间：

```java
@Service
public class MyService {
    @Scheduled(cron = "0 0 0 * * ?")
    public void backupDatabase() {
        // 备份数据库的代码
    }
}
```

这个例子中，我们**定义了一个名为backupDatabase的方法，并且使用@Scheduled注解指定了它每天0点执行一次**。

其中，cron参数是一个Cron表达式，用来指定任务的执行时间。

## 四、任务并发执行问题

默认情况下，Spring Schedule是单线程执行任务的，也就是说，如果一个任务还没执行完，下一个任务就必须等待。如果任务的执行时间较长，那么会影响其他任务的执行，甚至会导致任务积压，最终导致整个应用崩溃。

为了避免这种情况的发生，我们需要在配置文件中添加以下配置：

```properties
spring.task.scheduling.pool.size=10
```

一个大小为10的线程池来执行任务，也就是说，最多可以并发执行10个任务。这样就能避免任务积压和应用崩溃的问题了。

## 五、动态调整定时任务

有时候，我们需要动态调整定时任务的执行时间，比如说，某个任务需要紧急执行，我们就需要把它的执行时间提前。**在Spring Boot中，我们可以使用@Scheduled注解的fixedRate和fixedDelay属性来实现动态调整定时任务。**

**fixedRate属性指定任务执行的频率，单位是毫秒**。如果任务的执行时间大于fixedRate，那么任务会在执行完后立即再次执行；如果任务的执行时间小于fixedRate，那么任务会在fixedRate时间间隔后再次执行。

**fixedDelay属性指定任务执行的延迟时间，单位也是毫秒**。如果任务的执行时间大于fixedDelay，那么任务会在执行完后延迟fixedDelay时间后再次执行；如果任务的执行时间小于fixedDelay，那么任务会在fixedDelay-executionTime时间间隔后再次执行。

下面是一个动态调整定时任务执行时间的例子

```java
@Service
public class MyService {

    private long fixedRate = 60 * 1000; // 默认执行间隔为1分钟

    @Scheduled(fixedRate = "#{myService.fixedRate}")
    public void doTask() {
        // 执行任务的代码
    }

    public void updateFixedRate(long fixedRate) {
        this.fixedRate = fixedRate;
    }

}
```

这个例子中，我们定义了一个名为doTask的方法，并且使用@Scheduled注解指定了它的执行频率为fixedRate。同时，我们还定义了一个名为updateFixedRate的方法，用来动态调整定时任务的执行频率。

在updateFixedRate方法中，我们可以更新fixedRate属性的值，从而改变定时任务的执行频率。注意，我们在@Scheduled注解的fixedRate属性中使用了SpEL表达式来获取fixedRate属性的值，这样就能实现动态调整定时任务的执行频率了。

## 六、业务场景

### 6.1 从数据库中获取cron表达式，启动定时任务

**可以通过在Spring Boot项目中定义一个 `ScheduledTaskRegistrar` bean，并在其中注册一个 `Runnable` 对象，以实现在项目启动后手动创建 Spring Schedule 定时任务**。

同时，**使用 `JdbcTemplate` 从 MySQL 数据库中获取 cron 表达式并将其传递给 `CronTrigger` 对象以定义定时任务的调度时间**。

示例demo

```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.addTriggerTask(
                new Runnable() {
                    @Override
                    public void run() {
                        // 这里是您的定时任务执行的逻辑
                    }
                },
                new Trigger() {
                    @Override
                    public Date nextExecutionTime(TriggerContext triggerContext) {
                        String cronExpression = getCronExpressionFromDatabase();
                        if (cronExpression == null || cronExpression.isEmpty()) {
                            return null; // 如果 cron 表达式为空，禁止执行定时任务
                        }
                        CronTrigger cronTrigger = new CronTrigger(cronExpression);
                        return cronTrigger.nextExecutionTime(triggerContext);
                    }
                }
        );
    }

    private String getCronExpressionFromDatabase() {
        // 从 MySQL 数据库中获取 cron 表达式
        // 使用 JdbcTemplate 执行 SQL 查询，返回 cron 表达式字符串
        // 示例代码：
        String sql = "SELECT cron_expression FROM my_table WHERE task_name = 'my_task'";
        String cronExpression = jdbcTemplate.queryForObject(sql, String.class);
        return cronExpression;
    }

}
```

**在上面的代码中，`ScheduledTaskRegistrar` bean 会自动在项目启动后被创建，并通过 `configureTasks` 方法注册了一个定时任务。该任务使用 `Runnable` 对象定义了任务执行的逻辑，使用 `Trigger` 对象定义了任务的调度时间。**

**在 `Trigger` 对象中，我们通过调用 `getCronExpressionFromDatabase` 方法从 MySQL 数据库中获取 cron 表达式，并使用 `CronTrigger` 对象来定义该定时任务的调度时间。在 `nextExecutionTime` 方法中，`TriggerContext` 参数可以提供有关上一次执行时间和上下文信息的详细信息。**

**请注意，`ScheduledTaskRegistrar` bean 的 `@EnableScheduling` 注解用于启用 Spring Schedule 功能。**

### 6.2 从数据库中获取cron表达式，启动定时任务，任务执行时间超过下次执行时间,继续执行

在6.1的基础上继续修改

```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.addFixedDelayTask(
                new Runnable() {
                    @Override
                    public void run() {
                        // 这里是您的定时任务执行的逻辑
                    }
                },
                new Trigger() {
                    @Override
                    public Date nextExecutionTime(TriggerContext triggerContext) {
                        String cronExpression = getCronExpressionFromDatabase();
                        CronTrigger cronTrigger = new CronTrigger(cronExpression);
                        Date nextExecutionTime = cronTrigger.nextExecutionTime(triggerContext);
                        if (nextExecutionTime != null) {
                            // 如果下一次执行时间不为空，计算执行时间与当前时间之间的差值
                            long delay = nextExecutionTime.getTime() - System.currentTimeMillis();
                            if (delay < 0) {
                                // 如果下一次执行时间已经过去，设置延迟时间为 0
                                delay = 0;
                            }
                            return new Date(System.currentTimeMillis() + delay);
                        } else {
                            // 如果下一次执行时间为空，返回 null
                            return null;
                        }
                    }
                }
        );
    }

    private String getCronExpressionFromDatabase() {
        // 从 MySQL 数据库中获取 cron 表达式
        // 使用 JdbcTemplate 执行 SQL 查询，返回 cron 表达式字符串
        // 示例代码：
        String sql = "SELECT cron_expression FROM my_table WHERE task_name = 'my_task'";
        String cronExpression = jdbcTemplate.queryForObject(sql, String.class);
        return cronExpression;
    }

}
```

### 6.3 从数据库中获取定时任务,按照任务配置cron表达式启动,并监听数据库是否有新的定时任务生成

在6.2的基础上修改,原有的定时任务肯定还是不能动,那么就增加一个默认的定时任务来帮忙

+ 获取定时任务

+ 增加默认轮训定时任务
  
  + 去重,追加新的定时任务

+ 增加定时任务

```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {

    @Autowired
    private JdbcTemplate jdbcTemplate;
	
	/**
     * 当前正在执行的定时任务(数据库可能突然插入)
     */
    private List<Task> registerTaskList;

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		// 待执行定时任务
		List<Task> tasks = findTasksToExecute();
		if (CollUtil.isEmpty(registerTaskList)) {
            registerTaskList = new ArrayList<>();
        }
        // 追加定时任务列表
        synchronized (registerTaskList) {
            registerTaskList.addAll(tasks);
        }
		// 轮询追加定时任务
		taskRegistrar.addFixedDelayTask(
                new Runnable() {
                    @Override
                    public void run() {
						// 追加新的定时任务(数据库插入新的定时任务,自动追加执行)
						addNewScheduleTask(taskRegistrar);
                    }
                },
                new Trigger() {
                    @Override
                    public Date nextExecutionTime(TriggerContext triggerContext) {
						// 设置当前时间和时区
						Calendar nextExecutionTime = Calendar.getInstance();
						TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai"); // 设置为您所在的时区
						nextExecutionTime.setTimeZone(timeZone);
						Date lastCompletionTime = triggerContext.lastCompletionTime();
						if (lastCompletionTime != null) {
							nextExecutionTime.setTime(lastCompletionTime);
						}
						// 延迟1分钟启动
						nextExecutionTime.add(Calendar.MINUTE, 1);
						// 设置定时任务的触发规则(延迟1分钟启动,每5s执行一次)
						CronTrigger cronTrigger = new CronTrigger("5 * * * * ?", timeZone);
						return cronTrigger.nextExecutionTime(triggerContext);
                    }
                }
        );
        if (!CollectionUtil.isEmpty(tasks)) {
			for(Task task : tasks) {
				String cronExpression = task.getCronExpression();
				taskRegistrar.addFixedDelayTask(
						new Runnable() {
							@Override
							public void run() {
								// 业务执行逻辑
								// executeScheduleTask(task);
							}
						},
						new Trigger() {
							@Override
							public Date nextExecutionTime(TriggerContext triggerContext) {
								CronTrigger cronTrigger = new CronTrigger(cronExpression);
								Date nextExecutionTime = cronTrigger.nextExecutionTime(triggerContext);
								if (nextExecutionTime != null) {
									// 如果下一次执行时间不为空，计算执行时间与当前时间之间的差值
									long delay = nextExecutionTime.getTime() - System.currentTimeMillis();
									if (delay < 0) {
										// 如果下一次执行时间已经过去，设置延迟时间为 0
										delay = 0;
									}
									return new Date(System.currentTimeMillis() + delay);
								} else {
									// 如果下一次执行时间为空，返回 null
									return null;
								}
							}
						}
				);
			}

        }
    }

	/**
     * 追加新的定时任务.
     *
     * @param taskRegistrar 定时任务注册器
     */
    private void addNewScheduleTask(ScheduledTaskRegistrar taskRegistrar) {
        // 数据库中新的定时任务
        List<Task> newTaskList = findNewTasksToExecute();
        if (CollUtil.isNotEmpty(newTaskList)) {
            // 存在新的定时任务
            synchronized (registerTaskList) {
                registerTaskList.addAll(newTaskList);
            }
            for (Task newTask : newTaskList) {
                String newCronExpression = newTask.getCronExpression();
                taskRegistrar.addTriggerTask(
                        () -> {
                            // 定时任务的逻辑
                            executeScheduleTask(newTask);
                        },
                        triggerContext -> {
                            if (newCronExpression == null || newCronExpression.isEmpty()) {
                                // 如果 cron 表达式为空，禁止执行定时任务
                                return null;
                            }
                            CronTrigger cronTrigger = new CronTrigger(newCronExpression);
                            Date nextExecutionTime = cronTrigger.nextExecutionTime(triggerContext);
                            if (nextExecutionTime != null) {
                                // 如果下一次执行时间不为空，计算执行时间与当前时间之间的差值
                                long delay = nextExecutionTime.getTime() - System.currentTimeMillis();
                                if (delay < 0) {
                                    // 如果下一次执行时间已经过去，设置延迟时间为 0
                                    delay = 0;
                                }
                                return new Date(System.currentTimeMillis() + delay);
                            } else {
                                // 如果下一次执行时间为空，返回 null
                                return null;
                            }
                        });
                // 重新加载任务
                taskRegistrar.afterPropertiesSet();
            }
        }
    }
	
	/**
     * 执行定时任务.
	 *
     * @param task 定时任务
     */
	void executeScheduleTask(Task task) {
		// ....
	}
	
	/**
     * 获取定时任务.
     *
     * @return 定时任务
     */
    private List<Task> findTasksToExecute() {
		String sql = "select * from task";
		List<Task> queryForList = jdbcTemplate.query(sql, new TaskRowMapper());
		return queryForList;
    }
	
	/**
     * 获取数据库中新的定时任务.
     *
     * @return 定时任务集合
     */
    private List<Task> findNewTasksToExecute() {
        List<Task> tasks = findTasksToExecute();
        if (CollUtil.isEmpty(tasks)) {
            return new ArrayList<>();
        }
        // 通过主键进行区分
        Map<Integer, Task> registerTaskMap = registerTaskList.stream().collect(Collectors.toMap(Task::getId, Function.identity()));
        tasks = tasks.stream().filter(e -> !registerTaskMap.containsKey(e.getKeyId())).collect(Collectors.toList());
        return tasks;
    }

}
```

### 6.4 启动定时任务,任务结束时,关闭定时任务

关闭定时任务

+ 通过`ScheduledTaskRegistrar.destroy()`方法
  
  + **关闭`ScheduledTaskRegistrar`中注册的所有定时任务**(一键关闭)

+ 通过`ScheduledFuture.cancel(true)`方法关闭当前定时任务
  
  + **关闭`ScheduledFuture`注册的定时任务**(选择关闭)

#### 基于`ScheduledTaskRegistrar.destroy()`

基于代码6.1进行改造

```java
@@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		Task task = new Task();
        taskRegistrar.addTriggerTask(
                new Runnable() {
                    @Override
                    public void run() {
                        // 这里是您的定时任务执行的逻辑
						excute(task);
						if (isClose(task)) {
							// 符合关闭条件,关闭定时任务
							taskRegistrar.destroy();
						}
                    }
                },
                new Trigger() {
                    @Override
                    public Date nextExecutionTime(TriggerContext triggerContext) {
                        String cronExpression = getCronExpressionFromDatabase();
                        if (cronExpression == null || cronExpression.isEmpty()) {
                            return null; // 如果 cron 表达式为空，禁止执行定时任务
                        }
                        CronTrigger cronTrigger = new CronTrigger(cronExpression);
                        return cronTrigger.nextExecutionTime(triggerContext);
                    }
                }
        );
    }

    private String getCronExpressionFromDatabase() {
        // 从 MySQL 数据库中获取 cron 表达式
        // 使用 JdbcTemplate 执行 SQL 查询，返回 cron 表达式字符串
        // 示例代码：
        String sql = "SELECT cron_expression FROM my_table WHERE task_name = 'my_task'";
        String cronExpression = jdbcTemplate.queryForObject(sql, String.class);
        return cronExpression;
    }

}
```

#### 基于`ScheduledFuture.cancel(true)`

基于代码6.1改造

定义`TaskScheduler`

```java
@Configuration
@EnableScheduling
public class TaskSchedulerConfig {

    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        // 设置线程池大小
        taskScheduler.setPoolSize(15);
        // 设置线程名前缀
        taskScheduler.setThreadNamePrefix("MyTaskScheduler-");
        // 设置等待线程池关闭的时间
        taskScheduler.setAwaitTerminationSeconds(3600);
        // 在关闭时等待所有任务完成
        taskScheduler.setWaitForTasksToCompleteOnShutdown(true);
        // 其他定制配置...
        return taskScheduler;
    }
}
```

定义定时任务

```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {

    @Autowired
    private JdbcTemplate jdbcTemplate;
	
	/**
     * 定时任务(用于关闭某项定时任务)
     */
    @Autowired
    private TaskScheduler taskScheduler;
	/**
     * 定时任务(用于关闭某项定时任务)
     */
	private ScheduledFuture<?> scheduledFuture;

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		taskRegistrar.setTaskScheduler(taskScheduler);
		Task task = new Task();
        scheduledFuture = taskScheduler.schedule(
                new Runnable() {
                    @Override
                    public void run() {
                        // 这里是您的定时任务执行的逻辑
						excute(task);
						if (isClose(task)) {
							// 符合关闭条件,关闭定时任务
							scheduledFuture.close(true);
						}
                    }
                },
                new Trigger() {
                    @Override
                    public Date nextExecutionTime(TriggerContext triggerContext) {
                        String cronExpression = getCronExpressionFromDatabase();
                        if (cronExpression == null || cronExpression.isEmpty()) {
                            return null; // 如果 cron 表达式为空，禁止执行定时任务
                        }
                        CronTrigger cronTrigger = new CronTrigger(cronExpression);
                        return cronTrigger.nextExecutionTime(triggerContext);
                    }
                }
        );
    }

    private String getCronExpressionFromDatabase() {
        // 从 MySQL 数据库中获取 cron 表达式
        // 使用 JdbcTemplate 执行 SQL 查询，返回 cron 表达式字符串
        // 示例代码：
        String sql = "SELECT cron_expression FROM my_table WHERE task_name = 'my_task'";
        String cronExpression = jdbcTemplate.queryForObject(sql, String.class);
        return cronExpression;
    }

}
```

通过`taskScheduler.schedule`创建任务,依据判断条件`isClose()`关闭定时任务

### 6.5 从数据库中获取定时任务,按照任务配置cron表达式启动,并监听数据库是否有新的定时任务生成,任务执行完毕后,关闭当前定时任务.

基于6.3,6.4代码进行改造!!!!

由于我们采用的是一次定义多任务,但关闭时要求只关闭当前任务,所以不能采用

`ScheduledTaskRegistrar.destroy()`,它会将所有任务一起关闭,源码如下:

```java
	@Override
	public void destroy() {
	    for (ScheduledTask task : this.scheduledTasks) {
			task.cancel();
		}
		if (this.localExecutor != null) {
			this.localExecutor.shutdownNow();
		}
	}
```

`ScheduledTask.cancel` 源码如下:

```java
	public void cancel() {
		ScheduledFuture<?> future = this.future;
		if (future != null) {
			future.cancel(true);
		}
	}
```

与我们的`ScheduledFutur.cancel一致`,定点删除,因此我们开始正题

定义`TaskScheduler`

```java
@Configuration
@EnableScheduling
public class TaskSchedulerConfig {

    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        // 设置线程池大小
        taskScheduler.setPoolSize(15);
        // 设置线程名前缀
        taskScheduler.setThreadNamePrefix("MyTaskScheduler-");
        // 设置等待线程池关闭的时间
        taskScheduler.setAwaitTerminationSeconds(3600);
        // 在关闭时等待所有任务完成
        taskScheduler.setWaitForTasksToCompleteOnShutdown(true);
        // 其他定制配置...
        return taskScheduler;
    }
}
```

定义定时任务

```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {

    @Autowired
    private JdbcTemplate jdbcTemplate;
	
	/**
     * 当前正在执行的定时任务(数据库可能突然插入)
     */
    private List<Task> registerTaskList;
	
	/**
     * 定时任务(用于关闭某项定时任务)
     */
    @Autowired
    private TaskScheduler taskScheduler;

    /**
     * 存储所有的定时任务map<唯一标识,定时任务>
     */
    private Map<String, ScheduledFuture> scheduledFutureMap;


    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		// 待执行定时任务
		List<Task> tasks = findTasksToExecute();
		if (CollUtil.isEmpty(registerTaskList)) {
            registerTaskList = new ArrayList<>();
        }
        // 追加定时任务列表
        synchronized (registerTaskList) {
            registerTaskList.addAll(tasks);
        }
		
		taskRegistrar.setTaskScheduler(taskScheduler);
		
		// 轮询追加定时任务
		ScheduledFuture<?> defaultScheduledFuture = taskScheduler.schedule(
                new Runnable() {
                    @Override
                    public void run() {
						// 追加新的定时任务(数据库插入新的定时任务,自动追加执行)
						addNewScheduleTask(taskRegistrar);
                    }
                },
                new Trigger() {
                    @Override
                    public Date nextExecutionTime(TriggerContext triggerContext) {
						// 设置当前时间和时区
						Calendar nextExecutionTime = Calendar.getInstance();
						TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai"); // 设置为您所在的时区
						nextExecutionTime.setTimeZone(timeZone);
						Date lastCompletionTime = triggerContext.lastCompletionTime();
						if (lastCompletionTime != null) {
							nextExecutionTime.setTime(lastCompletionTime);
						}
						// 延迟1分钟启动
						nextExecutionTime.add(Calendar.MINUTE, 1);
						// 设置定时任务的触发规则(延迟1分钟启动,每5s执行一次)
						CronTrigger cronTrigger = new CronTrigger("5 * * * * ?", timeZone);
						return cronTrigger.nextExecutionTime(triggerContext);
                    }
                }
        );
		// 管理默认任务
		scheduledFutureMap.put("default", defaultScheduledFuture);
        if (!CollectionUtil.isEmpty(tasks)) {
			for(Task task : tasks) {
				String cronExpression = task.getCronExpression();
				ScheduledFuture<?> schedule = taskScheduler.schedule(
						new Runnable() {
							@Override
							public void run() {
								// 业务执行逻辑
								// executeScheduleTask(task);
							}
						},
						new Trigger() {
							@Override
							public Date nextExecutionTime(TriggerContext triggerContext) {
								CronTrigger cronTrigger = new CronTrigger(cronExpression);
								Date nextExecutionTime = cronTrigger.nextExecutionTime(triggerContext);
								if (nextExecutionTime != null) {
									// 如果下一次执行时间不为空，计算执行时间与当前时间之间的差值
									long delay = nextExecutionTime.getTime() - System.currentTimeMillis();
									if (delay < 0) {
										// 如果下一次执行时间已经过去，设置延迟时间为 0
										delay = 0;
									}
									return new Date(System.currentTimeMillis() + delay);
								} else {
									// 如果下一次执行时间为空，返回 null
									return null;
								}
							}
						}
				);
				// 通过主键作为唯一标识,退出时获取scheduledFuture退出
				String identity = String.valueOf(task.getId());
				scheduledFutureMap.put(identity, schedule);
			}
        }
    }

	/**
     * 追加新的定时任务.
     *
     * @param taskRegistrar 定时任务注册器
     */
    private void addNewScheduleTask(ScheduledTaskRegistrar taskRegistrar) {
        // 数据库中新的定时任务
        List<Task> newTaskList = findNewTasksToExecute();
        if (CollUtil.isNotEmpty(newTaskList)) {
            // 存在新的定时任务
            synchronized (registerTaskList) {
                registerTaskList.addAll(newTaskList);
            }
            for (Task newTask : newTaskList) {
                String newCronExpression = newTask.getCronExpression();
                ScheduledFuture<?> schedule = taskScheduler.schedule(
                        () -> {
                            // 定时任务的逻辑
                            executeScheduleTask(newTask);
                        },
                        triggerContext -> {
                            if (newCronExpression == null || newCronExpression.isEmpty()) {
                                // 如果 cron 表达式为空，禁止执行定时任务
                                return null;
                            }
                            CronTrigger cronTrigger = new CronTrigger(newCronExpression);
                            Date nextExecutionTime = cronTrigger.nextExecutionTime(triggerContext);
                            if (nextExecutionTime != null) {
                                // 如果下一次执行时间不为空，计算执行时间与当前时间之间的差值
                                long delay = nextExecutionTime.getTime() - System.currentTimeMillis();
                                if (delay < 0) {
                                    // 如果下一次执行时间已经过去，设置延迟时间为 0
                                    delay = 0;
                                }
                                return new Date(System.currentTimeMillis() + delay);
                            } else {
                                // 如果下一次执行时间为空，返回 null
                                return null;
                            }
                        });
				// 通过主键作为唯一标识,退出时获取scheduledFuture退出
				String identity = String.valueOf(task.getId());
				scheduledFutureMap.put(identity, schedule);
                // 重新加载任务
                taskRegistrar.afterPropertiesSet();

            }
        }
    }
	
	/**
     * 执行定时任务.
	 *
     * @param task 定时任务
     */
	void executeScheduleTask(Task task) {
		// 定时任务
		
		// 判断是否关闭当前定时任务
		if (isClose(task)) {
			String identity = String.valueOf(task.getId());
			if (scheduledFutureMap.containsKey(identity)) {
				ScheduledFuture scheduledFuture = scheduledFutureMap.get(identity);
				// 取消当前任务
				scheduledFuture.cancel(true);
				scheduledFutureMap.remove(identity);
				return;
			}
		}
	}
	
	/**
     * 获取定时任务.
     *
     * @return 定时任务
     */
    private List<Task> findTasksToExecute() {
		String sql = "select * from task";
		List<Task> queryForList = jdbcTemplate.query(sql, new TaskRowMapper());
		return queryForList;
    }
	
	/**
     * 获取数据库中新的定时任务.
     *
     * @return 定时任务集合
     */
    private List<Task> findNewTasksToExecute() {
        List<Task> tasks = findTasksToExecute();
        if (CollUtil.isEmpty(tasks)) {
            return new ArrayList<>();
        }
        // 通过主键进行区分
        Map<Integer, Task> registerTaskMap = registerTaskList.stream().collect(Collectors.toMap(Task::getId, Function.identity()));
        tasks = tasks.stream().filter(e -> !registerTaskMap.containsKey(e.getKeyId())).collect(Collectors.toList());
        return tasks;
    }

}
```

## 七、总结

通过本文的介绍，我们可以看到，在Spring Boot应用中整合Spring Schedule非常简单，只

需要添加依赖、配置定时任务，就能实现定时任务的执行。同时，我们还介绍了如何解决任

务并发执行的问题，以及如何动态调整定时任务的执行时间,动态关闭定时任务。

希望这篇文章对您有所帮助。
