### 개요
스프링 프레임워크에서 제공하는 [AbstractRoutingDataSource](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/datasource/lookup/AbstractRoutingDataSource.html)를 사용해 코드수정 없이 읽기 전용 트랜잭션을 Slave 데이터베이스에서 처리하도록 한다.

참고
* [dynamic-datasource-routing](http://spring.io/blog/2007/01/23/dynamic-datasource-routing/)
* [Anyframe Routing DataSource Plugin](http://dev.anyframejava.org/docs/anyframe/plugin/optional/routingdatasource/1.0.0/reference/htmlsingle/routingdatasource.html)

### 동작방식

[MultitenantDataSourceRouter](https://github.com/iyboklee/boot-multitenant-datasource/blob/master/src/main/java/com/github/iyboklee/config/mtds/MultitenantDataSourceRouter.java)는 Service 객체에서 @Transactional 어노테이션을 사용한는 public 메소드에 대한 Around Advice를 구현한다.

* @Transactional 어노테이션이 읽기 전용 속성을 지닌다면, 트랜잭션이 Slave 노드에서 처리될 수 있도록 정보를 ThreadLocal 저장소에 설정한다. 
* 그 외 모든 트랜잭션은 쓰기가 가능한 Master 노드에서 처리된다.
* Service 메소드가 리턴할 때, ThreadLocal 저장소에 저장된 데이터베이스 선택 정보를 초기화한다.

### 제약사항

* AbstractRoutingDataSource를 통해 통합되는 DataSource는 동일한 데이터베이스 스키마를 가져야한다.
  * 일반적으로 Master, Slave 노드는 동일한 데이터베이스를 지니기 때문에 큰 문제는 아니다.
* 한번 트랜잭션이 시작되면 중간에 데이터베이스를 변경할 수 없다. 즉, 트랜잭션이 읽기 전용으로 Slave 노드에서 시작되었다면, 중간에 쓰기연산을 위해 Master 노드로 변경할 수 없다.
* 데이터베이스 노드 선택을 위한 정보를 ThreadLocal 저장소에 저장하기 때문에 모든 작업 종료 후 반드시 ThreadLocal 저장소를 초기화 해야한다.