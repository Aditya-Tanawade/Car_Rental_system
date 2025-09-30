Caused by: java.sql.SQLException: ORA-17067: Invalid Oracle URL specified
https://docs.oracle.com/error-help/db/ora-17067/
	at oracle.jdbc.driver.PhysicalConnection.parseUrl(PhysicalConnection.java:2028) ~[ojdbc11-23.7.0.25.01.jar:23.7.0.25.01]
	at oracle.jdbc.driver.PhysicalConnection.readConnectionProperties(PhysicalConnection.java:1611) ~[ojdbc11-23.7.0.25.01.jar:23.7.0.25.01]
	at oracle.jdbc.driver.PhysicalConnection.<init>(PhysicalConnection.java:1013) ~[ojdbc11-23.7.0.25.01.jar:23.7.0.25.01]
	at oracle.jdbc.driver.T4CConnection.<init>(T4CConnection.java:810) ~[ojdbc11-23.7.0.25.01.jar:23.7.0.25.01]
	at oracle.jdbc.driver.T4CDriverExtension.getConnection(T4CDriverExtension.java:63) ~[ojdbc11-23.7.0.25.01.jar:23.7.0.25.01]
	at oracle.jdbc.driver.OracleDriver.connect(OracleDriver.java:887) ~[ojdbc11-23.7.0.25.01.jar:23.7.0.25.01]
	at oracle.jdbc.driver.OracleDriver.connect(OracleDriver.java:694) ~[ojdbc11-23.7.0.25.01.jar:23.7.0.25.01]
	at com.zaxxer.hikari.util.DriverDataSource.getConnection(DriverDataSource.java:144) ~[HikariCP-6.3.3.jar:na]
	at com.zaxxer.hikari.pool.PoolBase.newConnection(PoolBase.java:370) ~[HikariCP-6.3.3.jar:na]
	at com.zaxxer.hikari.pool.PoolBase.newPoolEntry(PoolBase.java:207) ~[HikariCP-6.3.3.jar:na]
	at com.zaxxer.hikari.pool.HikariPool.createPoolEntry(HikariPool.java:488) ~[HikariCP-6.3.3.jar:na]
	at com.zaxxer.hikari.pool.HikariPool.checkFailFast(HikariPool.java:576) ~[HikariCP-6.3.3.jar:na]
	at com.zaxxer.hikari.pool.HikariPool.<init>(HikariPool.java:97) ~[HikariCP-6.3.3.jar:na]
	at com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:111) ~[HikariCP-6.3.3.jar:na]
	at org.springframework.jdbc.datasource.DataSourceUtils.fetchConnection(DataSourceUtils.java:160) ~[spring-jdbc-6.2.11.jar:6.2.11]
	at org.springframework.jdbc.datasource.DataSourceUtils.doGetConnection(DataSourceUtils.java:118) ~[spring-jdbc-6.2.11.jar:6.2.11]
	at org.springframework.jdbc.datasource.DataSourceUtils.getConnection(DataSourceUtils.java:81) ~[spring-jdbc-6.2.11.jar:6.2.11]
	... 90 common frames omitted


Process finished with exit code 0
