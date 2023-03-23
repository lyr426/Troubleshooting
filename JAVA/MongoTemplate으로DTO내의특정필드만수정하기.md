# MongoTemplate 으로 DTO 내의 특정 필드만 수정하기

> “샘플 데이터 줄테니 Nosql로 한 번에 수정 가능한 api 만들어주세요 ~ “
> 

객체 데이터의 뎁스가 최대 3인 샘플 데이터를 원하는 값만 골라서 수정할 수 있는 api는 어떻게 만들어야 할까 ?

받았던 샘플데이터는 이렇게 생겨먹었었다 

```java
{
	"a1" : "qwwE",
	"a2" : {
			"b1" : "asd", 
			"b2" : {
					"c2" : "dasd",
					"c3" : "@#", 
....}
```

여기서 a1의 값과 c2의 값만 수정하고 싶을 때 api로 요청하는 json의 형태는 다음과 같을 것이다. 

```java
{
	"a1" : "qqq",
	"a2" : { 
			"b2" : {
					"c2" : "WWW" 
			}
	}
}
```

### 문제

1. 그냥 MongoRepository에서 save를 해버리면 바꾸려 하지 않았던 값이 전부 null로 변경되어 버린다.
2. NativeQuery를 작성하자니 너무 복잡해지고 가독성에 안좋을 거 같았다. 

### 문제 해결

1. RequestBody를 `Map<String, Object>` 로 받자 ! 
    - 계속 삽질했던 이유 중 하나가 샘플데이터를 감싸고 있는 DTO로 요청을 받아버리니 클라이언트 측에서 전달하지 않았던 값은 전부 null로 채워졌다.
2. MongoTemplate을 사용하자 ! 
    - 여러가지 방법을 사용하다가 복잡한 데이터를 처리할 때는(특히 쿼리문을 써야한다면) MongoTemplate이 좀 더 해결하는데 좋을 거 같다.
3. ModelMapper를 이용하자 ! 
    - ModelMapper를 이용하여 복잡한 객체를 전부 get, set 하지 않고 해당하는 key를 가진 값을 전부 복사 해준다.

- Service

```java
// tableDTO = service의 수정 메소드의 인자 (클라이언트가 수정하고자 하여 넘긴 데이터) 

// ----------- map 형태로 요청받은 데이터만 BOMTableDTO에 입력 -------------
Query query = new Query(Criteria.where("id").is(tableDTO.get("id")));
ModelMapper modelMapper = new CustomModelMapper();
modelMapper.getConfiguration().setSkipNullEnabled(true);
TableDTO updateTable = mongoTemplate.findById(tableDTO.get("id"), TableDTO.class);

modelMapper.map(bomTableDTO, updateTable);

        
// ----------------- 특정 필드가 아닌 DTO 전체를 Update -------------------

Document doc = new Document();
mongoTemplate.getConverter().write(updateTable, doc);
Update update = Update.fromDocument(doc);
mongoTemplate.updateMulti(query, update, TableDTO.class);

return mongoTemplate.findById(tableDTO.get("id"), TableDTO.class);
```