https://docs.google.com/document/d/1peFQLjeUdK2jvvkm8sADhnylQ0KLzGznrTEvYNbOe9w/edit?tab=t.0




  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.5.6)

2025-10-05T16:20:52.699+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] c.f.hrportal.HrportalnewApplication      : Starting HrportalnewApplication using Java 17.0.11 with PID 20616 (D:\Hr-portal-backend-main\target\classes started by ADMIN in D:\Hr-portal-backend-main)
2025-10-05T16:20:52.708+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] c.f.hrportal.HrportalnewApplication      : No active profile set, falling back to 1 default profile: "default"
2025-10-05T16:20:52.853+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] o.s.b.devtools.restart.ChangeableUrls    : The Class-Path manifest attribute in C:\Users\ADMIN\.m2\repository\com\oracle\database\jdbc\ojdbc11\23.7.0.25.01\ojdbc11-23.7.0.25.01.jar referenced one or more files that do not exist: file:/C:/Users/ADMIN/.m2/repository/com/oracle/database/jdbc/ojdbc11/23.7.0.25.01/oraclepki.jar
2025-10-05T16:20:52.854+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2025-10-05T16:20:52.854+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2025-10-05T16:20:54.339+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JDBC repositories in DEFAULT mode.
2025-10-05T16:20:54.419+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 64 ms. Found 0 JDBC repository interfaces.
2025-10-05T16:20:55.442+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8081 (http)
2025-10-05T16:20:55.471+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2025-10-05T16:20:55.472+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.46]
2025-10-05T16:20:55.524+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2025-10-05T16:20:55.526+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 2668 ms
Google Calendar not initialized: Address already in use: bind
2025-10-05T16:20:56.442+05:30  WARN 20616 --- [hrportalnew] [  restartedMain] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'interviewController' defined in file [D:\Hr-portal-backend-main\target\classes\com\finalproject\hrportal\controller\InterviewController.class]: Unsatisfied dependency expressed through constructor parameter 0: Error creating bean with name 'interviewSchedulerService' defined in file [D:\Hr-portal-backend-main\target\classes\com\finalproject\hrportal\service\InterviewSchedulerService.class]: Unsatisfied dependency expressed through constructor parameter 0: No qualifying bean of type 'com.google.api.services.calendar.Calendar' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}
2025-10-05T16:20:56.449+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] o.apache.catalina.core.StandardService   : Stopping service [Tomcat]
2025-10-05T16:20:56.477+05:30  INFO 20616 --- [hrportalnew] [  restartedMain] .s.b.a.l.ConditionEvaluationReportLogger : 

Error starting ApplicationContext. To display the condition evaluation report re-run your application with 'debug' enabled.
Google Calendar not initialized: Address already in use: bind
2025-10-05T16:20:56.534+05:30 ERROR 20616 --- [hrportalnew] [  restartedMain] o.s.b.d.LoggingFailureAnalysisReporter   : 

***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in com.finalproject.hrportal.service.InterviewSchedulerService required a bean of type 'com.google.api.services.calendar.Calendar' that could not be found.

The following candidates were found but could not be injected:
	- User-defined bean method 'googleCalendarService' in 'GoogleCalendarConfig' ignored as the bean value is null


Action:

Consider revisiting the entries above or defining a bean of type 'com.google.api.services.calendar.Calendar' in your configuration.


Process finished with exit code 0


