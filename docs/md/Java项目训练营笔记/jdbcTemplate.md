```java
 @Nullable
    private <T> T execute(StatementCallback<T> action, boolean closeResources) throws DataAccessException {
        Assert.notNull(action, "Callback object must not be null");
        Connection con = DataSourceUtils.getConnection(this.obtainDataSource());
        Statement stmt = null;

        Object var12;
        try {
            stmt = con.createStatement();
            this.applyStatementSettings(stmt);
            T result = action.doInStatement(stmt);
            this.handleWarnings(stmt);
            var12 = result;
        } catch (SQLException var10) {
            String sql = getSql(action);
            JdbcUtils.closeStatement(stmt);
            stmt = null;
            DataSourceUtils.releaseConnection(con, this.getDataSource());
            con = null;
            throw this.translateException("StatementCallback", sql, var10);
        } finally {
            if (closeResources) {
                JdbcUtils.closeStatement(stmt);
                DataSourceUtils.releaseConnection(con, this.getDataSource());
            }

        }

        return var12;
    }
```
```java
public void execute(final String sql) throws DataAccessException {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Executing SQL statement [" + sql + "]");
        }

        class ExecuteStatementCallback implements StatementCallback<Object>, SqlProvider {
            ExecuteStatementCallback() {
            }

            @Nullable
            public Object doInStatement(Statement stmt) throws SQLException {
                stmt.execute(sql);
                return null;
            }

            public String getSql() {
                return sql;
            }
        }

        this.execute(new ExecuteStatementCallback(), true);
    }
```
使用了回调函数处理不同的sql语句
