# 本质上是使用了ThreadLocal

MapperFactoryBean继承了SqlSessionDaoSupport Spring中的sqlSession变成了SqlSessionTemplate对象
Mybatis中的SqlSessionTemplate继承了SqlSession

SqlSessionTemplate使用了jdk动态代理

SqlSessionTemplate中的内部类SqlSessionInterceptor
```java
public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) {
    Assert.notNull(sessionFactory, "No SqlSessionFactory specified");
    Assert.notNull(executorType, "No ExecutorType specified");
    SqlSessionHolder holder = (SqlSessionHolder)TransactionSynchronizationManager.getResource(sessionFactory);
    SqlSession session = sessionHolder(executorType, holder);
    if (session != null) {
        return session;
    } else {
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Creating a new SqlSession");
        }

        session = sessionFactory.openSession(executorType);
        registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);
        return session;
    }
}
```
```java
private static void registerSessionHolder(SqlSessionFactory sessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator, SqlSession session) {
	if (TransactionSynchronizationManager.isSynchronizationActive()) {
		Environment environment = sessionFactory.getConfiguration().getEnvironment();
		if (environment.getTransactionFactory() instanceof SpringManagedTransactionFactory) {
			if (LOGGER.isDebugEnabled()) {
				LOGGER.debug("Registering transaction synchronization for SqlSession [" + session + "]");
			}

			SqlSessionHolder holder = new SqlSessionHolder(session, executorType, exceptionTranslator);
			TransactionSynchronizationManager.bindResource(sessionFactory, holder);
			TransactionSynchronizationManager.registerSynchronization(new SqlSessionUtils.SqlSessionSynchronization(holder, sessionFactory));
			holder.setSynchronizedWithTransaction(true);
			holder.requested();
		} else {
			if (TransactionSynchronizationManager.getResource(environment.getDataSource()) != null) {
				throw new TransientDataAccessResourceException("SqlSessionFactory must be using a SpringManagedTransactionFactory in order to use Spring transaction synchronization");
			}

			if (LOGGER.isDebugEnabled()) {
				LOGGER.debug("SqlSession [" + session + "] was not registered for synchronization because DataSource is not transactional");
			}
		}
	} else if (LOGGER.isDebugEnabled()) {
		LOGGER.debug("SqlSession [" + session + "] was not registered for synchronization because synchronization is not active");
	}

}
```
