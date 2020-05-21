[toc]



# 前言

## 推荐阅读

> - [Guide to JAXB](https://www.baeldung.com/jaxb)
> - [XML 与 javabean 的转换](https://www.cnkirito.moe/javabean-xml/)
> - [JAXB相关注解](https://docs.oracle.com/javaee/7/api/javax/xml/bind/annotation/package-summary.html)
> - 





# 一、使用JAXB解析xml



## 1.xml文件

```xml
<?xml version="1.0"?>
<tmx version="1.4">
	<header adminlang="en">
	</header>
	<body>
		<tu>
			<tuv xml:lang="zh-cn">
				<seg>
					玄奘，中国最伟大的翻译家 dfsdfkjs
				</seg>
			</tuv>
			<tuv xml:lang="en">
				<seg>
					Xuan Zang, Possibly China's Greatest Translator
				</seg>
			</tuv>
			<tuv xml:lang="de">
				<seg>
					Xuan Zang - möglicherweise Chinas größter Übersetzer
				</seg>
			</tuv>
			<tuv xml:lang="es">
				<seg>
					Xuan Zang, probablemente el traductor más importante de China
				</seg>
			</tuv>
			<tuv xml:lang="it">
				<seg>
					Xuan Zang, probabilmente il più grande traduttore cinese
				</seg>
			</tuv>
			<tuv xml:lang="ko">
				<seg>
					현장법사(玄奬法師), 중국 최고의 번역가
				</seg>
			</tuv>
		</tu>
		<tu>
			<tuv xml:lang="zh-cn">
				<seg>
					玄奘，中国最伟大的翻译家 dfsdfkjs
				</seg>
			</tuv>
			<tuv xml:lang="en">
				<seg>
					Xuan Zang, Possibly China's Greatest Translator
				</seg>
			</tuv>
			<tuv xml:lang="de">
				<seg>
					Xuan Zang - möglicherweise Chinas größter Übersetzer
				</seg>
			</tuv>
			<tuv xml:lang="es">
				<seg>
					Xuan Zang, probablemente el traductor más importante de China
				</seg>
			</tuv>
			<tuv xml:lang="it">
				<seg>
					Xuan Zang, probabilmente il più grande traduttore cinese
				</seg>
			</tuv>
			<tuv xml:lang="ko">
				<seg>
					현장법사(玄奬法師), 중국 최고의 번역가
				</seg>
			</tuv>
		</tu>
	</body>
</tmx>

```



## 2.实体类

（1）tmx

```java
@XmlRootElement(name = "tmx")// <1>
@XmlAccessorType(XmlAccessType.FIELD)// <1>
@Data
public class Tmx {
    @XmlElement(name = "header")
    TmxHeader header;

    @XmlElement(name = "body")
    TmxBody body;
}
```



(2) TmxHeader

```java
@XmlAccessorType(XmlAccessType.FIELD)
@Data
public class TmxHeader {

    @XmlAttribute(name = "adminlang")
    private String adminLang;

    @XmlAttribute(name = "creationtool")
    private String creationTool = "Tuma";

    @XmlAttribute(name = "creationtoolversion")
    private String creationToolVersion = "1.0.1";

    @XmlAttribute(name = "creationdate")
    private String creationDate;

    @XmlAttribute(name = "datatype")
    private String datatype;

}

```



（3）TmxBody

```java
@Data
@XmlAccessorType(XmlAccessType.FIELD)
public class TmxBody {

    @XmlElement(name = "tu")
    private List<Tu> tus;

}
```



（3）Tu

```java
@Data
@XmlAccessorType(XmlAccessType.FIELD)
public class Tu {
    @XmlAttribute(name = "creationid")
    private String creationId;

    @XmlAttribute(name = "creationdate")
    private String creationDate;

    @XmlAttribute(name = "tuid")
    private String tuid;

    @XmlElement(name = "tuv")
    private List<Tuv> tuvs;
}

```



（4）Tuv

```java
@Data
@XmlAccessorType(XmlAccessType.FIELD)
public class Tuv {
    @XmlAttribute(name = "lang", namespace = javax.xml.XMLConstants.XML_NS_URI)
    private String lang;

    @XmlElement(name = "seg")
    private String seg;
}

```







## 3.xml解析器

- 接口

```java
public interface XmlParser {

    <T> T parseObject(String xmlContent, Class<T> claaz);

    <T> T parseObject(InputStream inputStream, Class<T> claaz);

    String toXmlString(Object obj);

}

```



- 实现类

  ```java
  @Slf4j
  public class JAXBXmlParser implements XmlParser{
  
      @Override
      public <T> T parseObject(String xmlContent, Class<T> claaz) {
          Object result = null;
  
          try {
              JAXBContext context = JAXBContext.newInstance(claaz);
              Unmarshaller unmarshaller = context.createUnmarshaller();
              result = unmarshaller.unmarshal(new StringReader(xmlContent));
          } catch (JAXBException e) {
              log.error("[[type=JAXBXmlParser]] can not parse xml content to class:{}, xml content: \n {}", claaz.toString(), xmlContent, e);
          }
          return (T) result;
      }
  
      @Override
      public <T> T parseObject(InputStream inputStream, Class<T> claaz) {
          Object result = null;
  
          try {
              JAXBContext context = JAXBContext.newInstance(claaz);
              Unmarshaller unmarshaller = context.createUnmarshaller();
              result = unmarshaller.unmarshal(inputStream);
          } catch (JAXBException e) {
              log.error("[[type=JAXBXmlParser]] can not parse xml content to class:{}", claaz.toString(), e);
          }
          return (T) result;
      }
  
      @Override
      public String toXmlString(Object obj) {
          String result = null;
        try {
              JAXBContext context = JAXBContext.newInstance(obj.getClass());
              Marshaller marshaller = context.createMarshaller();
              StringWriter writer = new StringWriter();
              marshaller.marshal(obj, writer);
              result = writer.toString();
          } catch (Exception e) {
              log.error("[[type=JAXBXmlParser]] can not transform Object:{} to String, Object:{}", obj.getClass(), obj, e);
          }
          return result;
      }
  }
  
  ```
  
  

## 4.Spock测试用例

```java
class JAXBXmlParserSpec extends Specification {

    def "ParseObject"() {
        given: "a xmlContent"
        String xmlContent = getXmlContent()

        and: "a XmlParser"
        XmlParser xmlParser = new JAXBXmlParser();

        when: "parse object from string"
        Tmx tmx = xmlParser.parseObject(xmlContent, Tmx.class);

        then:
        tmx.header.creationTool == "Heartsome TM Server"
    }

    def "ToXmlString"() {

        given: "an object"
        def obj = getObject()

        and: "a XmlParser"
        XmlParser xmlParser = new JAXBXmlParser()

        when: "convert object to string"
        def xmlContent = xmlParser.toXmlString(obj)

        then:
        xmlContent != null
    }


    private String getXmlContent(){
        InputStream inputStream = this.getClass().getResourceAsStream("/sample.tmx")
        String xmlContent = IOUtils.toString(inputStream, "UTF-8")
        return xmlContent;
    }

    private Object getObject(){
        String xmlContent = getXmlContent()
        XmlParser xmlParser = new JAXBXmlParser()
        Tmx tmx = xmlParser.parseObject(xmlContent, Tmx.class)
        return tmx
    }

}

```









