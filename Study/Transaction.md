# Transaction 
DB 상태변경에 대한 최소한의 작업 단위


## 성질
1. 원자성 : 모두 성공(commit)? 또는 모두 실패(rollback)? 둘 중에 하나여야한다.
2. 일관성 : 트랜잭션은 일관성있는 데이터베이스 상태가 이뤄져야한다.
3. 격리성 : 트랜잭션이 실행되고 있을때 그 트랜잭션에 다른 트랜잭션이 끼어들 수 없다. 
4. 영속성 : 트랜잭션이 완료되면 데이터베이스에 영구 저장되어야함.

## 과거 Transaction  방법
- JDBC 코드를 이용해 자바 소스 코드에서 직접 처리
```
String insertTableSQL = "INSERT INTO DBUSER (USER_ID, USERNAME, CREATED_BY, CREATED_DATE) VALUES (?,?,?,?)";
String updateTableSQL = "UPDATE DBUSER SET USERNAME =? WHERE USER_ID = ?";

try {
    dbConnection = getDBConnection();
    dbConnection.setAutoCommit(false);

    preparedStatementInsert = dbConnection.prepareStatement(insertTableSQL);
    [...]
    preparedStatementInsert.executeUpdate();

    preparedStatementUpdate = dbConnection.prepareStatement(updateTableSQL);
    [...]
    preparedStatementUpdate.executeUpdate();

    dbConnection.commit();
} catch (SQLException e) {
    dbConnection.rollback();
} finally {
    [...]
}
```

- DAO에 대한 재사용성을 높이기 위한 처리
```
public class AnswerDao {
    public void insert(Answer answer) throws SQLException {
        try {
            String sql = "INSERT INTO ANSWERS (writer, contents, createdDate, questionId) VALUES (?, ?, ?, ?)";
            [...]
            pstmt.executeUpdate();
        } finally {
            [...]
        }       
    }
}

public class QuestionDao {
    public void updateCommentCount(long questionId) throws SQLException {
        try {
            String countplussql = "update QUESTIONS set countOfComment = countOfComment + 1 where questionId = ?";
            [...]
            pstmt.executeUpdate();
        } finally {
            [...]
        }       
    }    
}
```

```
public class QnaService {
    […]
    public void insert(Answer answer) throws SQLException {
        Connection con = null; 
        try {
            con = ConnectionManager.getConnection();
            con.setAutoCommit(false);
            
            answerDao.insert(con, answer);
            questionDao.updateCommentCount(con, answer.getQuestionId());
            
            con.commit();
        } catch (SQLException e) {
            con.rollback();
        } finally {
            […]
        }
    }
}
```

- Spring - xml 설정 파일을 활용한 transaction 처리
```
<?xml version="1.0" encoding="UTF-8"?>
<beans [...]>

    <aop:config>
        <aop:pointcut id="txPointcut" expression="execution(* *..MemberDao.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>
    
    <tx:advice id="txAdvice">
        <tx:attributes>
            <tx:method name="get*" read-only="true" propagation="MANDATORY"/>
            <tx:method name="*" />
        </tx:attributes>
    </tx:advice>  
    
    <bean id="memberDao" class="member.dao.MemberDaoImpl" 
        p:dataSource-ref="dataSource" />
    
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager" 
        p:dataSource-ref="dataSource" />
    
    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        [...]
    </bean>
</beans>
```

## 현재 Transaction 방법
- Spring @Transactional 애노테이션을 활용한 transaction 처리
```
@Transactional
public static class MemberDaoImpl extends JdbcDaoSupport implements MemberDao {
    SimpleJdbcInsert insert;

    public void initTemplateConfig() {
        insert = new SimpleJdbcInsert(getDataSource()).withTableName("member");
    }
    
    public void add(Member m) {
        insert.execute(new BeanPropertySqlParameterSource(m));
    }
    
    public void deleteAll() {
        getJdbcTemplate().execute("delete from member");
    }
    
    @Transactional(readOnly=true)
    public long count() {
        return getJdbcTemplate().queryForObject("select count(*) from member", Long.class).longValue();
    }
}
```

## 상황에 맞는 Transaction 설정?
- @Transactional 애노테이션을 사용하지만, 상황에 맞게 사용자가 설정해서 사용해야한다.
- 예를 들어 A,B,C 작업이 순서대로 하나의 트랜잭션에서 작업되는데, C작업 도중에 에러가 발생해 Rollback 되더라도 A,B는 진행되어야 하는 상황이 발생하기 때문이다. 
즉, 모든 작업을 RollBack 시키지 말아야할 경우도 있다.


## 설정 내용은 크게 다음과 같다.
```
- readOnly=true
- 격리 레벨(isolation level)
- 전달 행위(propagation behavior)
- Checked Exception(compile time exception)과 Unchecked Exception(runtime exception)에 따른 transaction 처리
```

