## 스프링 부트 테스트
```
@SpringBootTest(
    properties = [
        "spring.config.location=" +
                "file:\${APP_HOME}/properties/application.yml" +
                ", file:\${APP_HOME}/properties/database.yml"
    ]
)
```

## XML Mapper
```
class ServiceTest {
    val easyXml = """
        <response>
            <person>
                <no>1</no>
                <name>seongjin</name>
                <age>28</age>
            </person>
        </response>
    """.trimIndent()
    data class EasyXml(val person: Person){

            data class Person(
                val no:Int,
                val name:String,
                val age:Int
            )
    }
    @Test
    fun easyXml(){
        val xmlMapper: XmlMapper = XmlMapper().registerKotlinModule() as XmlMapper
        val easyObject = xmlMapper.readValue<EasyXml>(easyXml)
        println("EASY: $easyObject")
        Assertions.assertNotNull(easyObject)
    }
}
```
- response 내의 태그(person)와 클래스 생성자 프로퍼티(person)가 같아야 함..