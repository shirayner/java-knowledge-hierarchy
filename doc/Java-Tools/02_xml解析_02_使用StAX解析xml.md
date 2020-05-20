[toc]



# 前言

## 推荐阅读

> - [Using the Streaming API for XML (StAX)](https://docs.oracle.com/cd/E17904_01/web.1111/e13724/intro.htm#XMLPG143)
> - [Java Read XML with StAX Parser – Cursor & Iterator APIs](https://howtodoinjava.com/xml/read-xml-stax-parser-cursor-iterator/)
> - [Java StAX Parser - Parse XML Document](https://www.tutorialspoint.com/java_xml/java_stax_parse_document.htm)
> - [Parsing an XML File Using StAX](https://www.baeldung.com/java-stax)
> - [StAX---基于事件的拉式XML解析](https://segmentfault.com/a/1190000006975725)



其他：

> - [JDom,Dom4j,JAXB,XPath](https://www.jianshu.com/p/604fb4748abe)
> - 





# 一、使用StAX 解析xml





# 二、入门实例

## 1.代码实例

### 1.1 xml

- sample.tmx

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tmx version="1.4">
	<header adminlang="en" />
	<body>
		<tu>
			<tuv xml:lang="zh-cn">
				<seg>
					玄奘，中国最伟大的翻译家
					<ph>
						<b>
							bold 1111
							bold 1111
							bold 1111
						</b>
						<b>
							bold 1111
						</b>
						<b>
							bold 1111
						</b>
					</ph>
					dfsdfkjs
					<ph>
						<b>
							bold 2222
						</b>
					</ph>
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
					玄奘，中国最伟大的翻译家222
					<ph>
						<b>
							bold dsfsf
						</b>
					</ph>
					dfsdfkjs
				</seg>
			</tuv>
			<tuv xml:lang="en">
				<seg>
					Xuan Zang, Possibly China's Greatest Translator222
				</seg>
			</tuv>
			<tuv xml:lang="de">
				<seg>
					Xuan Zang - möglicherweise Chinas größter Übersetzer222
				</seg>
			</tuv>
			<tuv xml:lang="es">
				<seg>
					Xuan Zang, probablemente el traductor más importante de China222
				</seg>
			</tuv>
			<tuv xml:lang="it">
				<seg>
					Xuan Zang, probabilmente il più grande traduttore cinese222
				</seg>
			</tuv>
			<tuv xml:lang="ko">
				<seg>
					현장법사(玄奬法師), 중국 최고의 번역가222
				</seg>
			</tuv>
		</tu>
	</body>
</tmx>

```





### 1.2 常量与实体类

（1）TmxElementConstant

```java
public interface TmxElementConstant {
    String TMX = "tmx";
    String HEADER = "header";
    String ADMIN_LANG = "adminlang";
    String BODY = "body";
    String TU = "tu";
    String TUV = "tuv";
    String XML_LANG = "xml:lang";
    String SEG = "seg";
    String PH = "ph";
}

```







```java
@Data
public class Tmx {
    TmxHeader header;
    TmxBody body;
}
```



```java
@Data
public class TmxHeader {
    private String adminLang;
}

```





```java
@Data
public class TmxBody {
    private List<Tu> tus;
}
```





```java
@Data
public class Tu {
    private List<Tuv> tuvs;
}

```





```java
@Data
public class Tuv {
    public static final Pattern PATTERN_PLACEHOLDER= Pattern.compile("\\$\\{\\d+\\}");
    public static final Pattern PATTERN_SPACE= Pattern.compile("\\n+\\t+");

    private String lang;
    private String seg;
    private String segText;
    private List<String> phList;
    private Map<Integer, String> phInnerTextMap;


    public String getDisplayText(){
        Map<Integer, String> paramMap = IntStream.range(0, phList.size()).boxed().collect(toMap(i -> i+1 , i ->{
            List<String> placeHolderSymbolList = getPlaceHolderSymbolList(phList.get(i));
            StringBuilder phBlockInnerText = new StringBuilder();
            for(String placeHolderSymbol: placeHolderSymbolList){
                int index = Integer.parseInt(placeHolderSymbol.substring(2, 3));
                // replace space
                Matcher matcher = PATTERN_SPACE.matcher(phInnerTextMap.get(index));
                String eventString = matcher.replaceAll("");
                phBlockInnerText.append(eventString);
            }
            return phBlockInnerText.toString();
        }));

        return getRealText(segText, paramMap);
    }


    private static List<String> getPlaceHolderSymbolList(String textWithPlaceHolder){
        List<String> placeHolderSymbolList = new ArrayList<>();
        Matcher matcher = PATTERN_PLACEHOLDER.matcher(textWithPlaceHolder);
        while (matcher.find()) {
            placeHolderSymbolList.add(matcher.group(0));
        }
        return placeHolderSymbolList;
    }


    public static String getRealText(String textWithPlaceHolder, Map<Integer,String> paramMap){
        List<String> placeHolderList = getPlaceHolderSymbolList(textWithPlaceHolder);

        for(String placeHolder: placeHolderList){
            int index = Integer.parseInt(placeHolder.substring(2, 3));
            textWithPlaceHolder =  textWithPlaceHolder.replace(placeHolder, paramMap.get(index));
        }

        return textWithPlaceHolder;
    }

}
```





### 1.3 Tmx解析器

```java
public interface TmxParser {

    Tmx parse(InputStream inputStream);

    String toXmlString(Tmx tmx);

}

```



```java
@Slf4j
public class SimpleTmxParser implements TmxParser {

    private boolean parsingSeg = false;
    private boolean parsingPh = false;
    private boolean parsingPhInnerElementText = false;

    @Override
    public Tmx parse(InputStream inputStream) {
        Tmx tmx = null;

        XMLEventReader eventReader = getEventReader(inputStream);
        while (eventReader.hasNext()) {
            XMLEvent event = null;
            try {
                event = eventReader.nextEvent();
            } catch (XMLStreamException e) {
                log.error("[[type=SimpleTmxParser]] error occurs when parsing xml", e);
                return null;
            }

            switch (event.getEventType()) {
                case XMLStreamConstants.START_ELEMENT:
                    StartElement startElement = event.asStartElement();
                    String qName = startElement.getName().getLocalPart();
                    switch (qName.toLowerCase()) {
                        case TMX:
                            tmx = new Tmx();
                            break;
                        case HEADER:
                            TmxHeader header = new TmxHeader();
                            Iterator<Attribute> iterator = startElement.getAttributes();
                            while (iterator.hasNext()) {
                                Attribute attribute = iterator.next();
                                if (ADMIN_LANG.equals(attribute.getName().getLocalPart())) {
                                    header.setAdminLang(attribute.getValue());
                                }
                            }
                            tmx.setHeader(header);
                            break;
                        case BODY:
                            TmxBody tmxBody = new TmxBody();
                            tmx.setBody(tmxBody);
                            break;
                        case TU:
                            TmxBody body = tmx.getBody();
                            List<Tu> tus = body.getTus();
                            if (tus == null) {
                                tus = new ArrayList<>();
                            }
                            Tu tu = new Tu();
                            tus.add(tu);

                            body.setTus(tus);
                            break;
                        case TUV:
                            body = tmx.getBody();
                            tus = body.getTus();
                            tu = tus.get(tus.size() - 1);
                            List<Tuv> tuvs = tu.getTuvs();
                            if (tuvs == null) {
                                tuvs = new ArrayList<>();
                            }

                            Tuv tuv = new Tuv();
                            iterator = startElement.getAttributes();
                            while (iterator.hasNext()) {
                                Attribute attribute = iterator.next();
                                String xmlLang = attribute.getName().getPrefix() + ":" + attribute.getName().getLocalPart();
                                if (XML_LANG.equalsIgnoreCase(xmlLang)) {
                                    tuv.setLang(attribute.getValue());
                                }
                            }
                            tuvs.add(tuv);

                            tu.setTuvs(tuvs);
                            break;
                        case SEG:
                            parsingSeg = true;
                            continue;
                        default:
                            break;
                    }
                    break;

                case XMLStreamConstants.END_ELEMENT:
                    EndElement endElement = event.asEndElement();
                    qName = endElement.getName().getLocalPart();
                    if (SEG.equalsIgnoreCase(qName)) {
                        parsingSeg = false;
                    }
                    break;
                default:
                    break;

            }

            if (parsingSeg) {
                parseSeg(event, tmx);
            }

        }
        return tmx;
    }


    @SneakyThrows
    private XMLEventReader getEventReader(InputStream inputStream) {
        XMLInputFactory factory = XMLInputFactory.newInstance();
        return factory.createXMLEventReader(new InputStreamReader(inputStream));
    }

    private void parseSeg(XMLEvent event, Tmx tmx) {
        TmxBody body = tmx.getBody();
        List<Tu> tus = body.getTus();
        Tu tu = tus.get(tus.size() - 1);
        List<Tuv> tuvs = tu.getTuvs();
        Tuv tuv = tuvs.get(tuvs.size() - 1);
        // populate seg
        String formerSeg = tuv.getSeg() == null ? "" : tuv.getSeg();
        String seg = formerSeg + event.toString();
        tuv.setSeg(seg);

        if (event.getEventType() == XMLStreamConstants.CHARACTERS) {
            if (!parsingPh) {
                // populate segText
                String formerSegText = tuv.getSegText() == null ? "" : tuv.getSegText();
                String segText = formerSegText + event.toString().trim();
                tuv.setSegText(segText);
            }
        } else if (event.getEventType() == XMLStreamConstants.START_ELEMENT) {
            String qName = event.asStartElement().getName().getLocalPart();
            if (PH.equalsIgnoreCase(qName)) {
                // init phList
                parsingPh = true;
                List<String> phList = tuv.getPhList();
                if (phList == null) {
                    phList = new ArrayList<>();
                }
                phList.add("");
                tuv.setPhList(phList);

                //segText place holder
                String formerSegText = tuv.getSegText() == null ? "" : tuv.getSegText();
                String segText = formerSegText + "${" + phList.size() + "}";
                tuv.setSegText(segText);
                return;
            }

        } else if (event.getEventType() == XMLStreamConstants.END_ELEMENT) {
            String qName = event.asEndElement().getName().getLocalPart();
            if (PH.equalsIgnoreCase(qName)) {
                parsingPh = false;
                return;
            }
        }

        if (parsingPh) {
            // populate phList
            processHpBlock(event, tuv);
        }
    }


    public void processHpBlock(XMLEvent event, Tuv tuv){
        List<String> phList = tuv.getPhList();

        if(event.getEventType() == XMLStreamConstants.CHARACTERS){
            if(StringUtils.isNotBlank(event.toString())){
                Map<Integer, String> phInnerTextMap = tuv.getPhInnerTextMap();
                if(phInnerTextMap == null){
                    phInnerTextMap = new HashMap<>();
                }

                if(!parsingPhInnerElementText){
                    // start parsing Ph Inner Element text
                    parsingPhInnerElementText = true;
                    // phInnerTextMap size ++
                    phInnerTextMap.put(phInnerTextMap.size()+1, "");
                    // phList: ph inner block placeholder, e.g  <b>${1}</b>
                    String currentPhContent= phList.get(phList.size() - 1)  + "${" + phInnerTextMap.size() + "}";
                    phList.set(phList.size() - 1, currentPhContent);

                }

                // phInnerTextMap: put phInnerText
                String tempInnerText = phInnerTextMap.get(phInnerTextMap.size()) +"" + event.toString();
                phInnerTextMap.put(phInnerTextMap.size(),  tempInnerText);
                tuv.setPhInnerTextMap(phInnerTextMap);

            }
        }else{
            String phContent = phList.get(phList.size() - 1) + event.toString();
            phList.set(phList.size() - 1, phContent);
            tuv.setPhList(phList);
        }

        if(event.getEventType() == XMLStreamConstants.END_ELEMENT && parsingPhInnerElementText){
            parsingPhInnerElementText = false;
        }

    }

    @Override
    public String toXmlString(Tmx tmx) {

        try {
            StringWriter stringWriter = new StringWriter();
            XMLStreamWriter xmlStreamWriter = getXmlStreamWriter(stringWriter);
            xmlStreamWriter.writeStartDocument();
            xmlStreamWriter.writeStartElement(TMX);
            xmlStreamWriter.writeAttribute("version", "1.4");

            // header
            TmxHeader header = tmx.getHeader();
            xmlStreamWriter.writeStartElement(HEADER);
            if(StringUtils.isNotBlank(header.getAdminLang())){
                xmlStreamWriter.writeAttribute(ADMIN_LANG, header.getAdminLang());
            }
            xmlStreamWriter.writeEndElement();

            // body
            xmlStreamWriter.writeStartElement(BODY);
            TmxBody body = tmx.getBody();
            List<Tu> tus = body.getTus();
            if (tus != null) {
                for (Tu tu : tus) {
                    xmlStreamWriter.writeStartElement(TU);
                    List<Tuv> tuvs = tu.getTuvs();
                    if (tuvs != null) {
                        for (Tuv tuv : tuvs) {
                            xmlStreamWriter.writeStartElement(TUV);
                            if (!StringUtils.isEmpty(tuv.getLang())) {
                                xmlStreamWriter.writeAttribute(XML_LANG, tuv.getLang());
                            }
                            if (!StringUtils.isEmpty(tuv.getSeg())) {
                                xmlStreamWriter.writeStartElement(SEG);
                                String segText = tuv.getSegText();
                                List<String> phList = tuv.getPhList();

                                if (phList != null) {
                                    String[] segTextSplit = segText.split("\\$\\{\\d+\\}");
                                    for (int i = 0; i < phList.size(); i++) {
                                        xmlStreamWriter.writeCharacters(segTextSplit[i]);
                                        String phBlock = Tuv.getRealText(phList.get(i), tuv.getPhInnerTextMap());
                                        writePhToXml(xmlStreamWriter, stringWriter, phBlock);
                                    }
                                } else {
                                    xmlStreamWriter.writeCharacters(segText);
                                }
                                xmlStreamWriter.writeEndElement();
                            }
                            xmlStreamWriter.writeEndElement();
                        }
                    }
                    xmlStreamWriter.writeEndElement();
                }
            }
            xmlStreamWriter.writeEndElement();
            xmlStreamWriter.writeEndDocument();
            xmlStreamWriter.flush();
            xmlStreamWriter.close();

            String xmlString = stringWriter.getBuffer().toString();
            stringWriter.close();
            return xmlString;

        } catch (XMLStreamException | IOException e) {
            log.error("[[type=SimpleTmxParser]] error occurs when parsing xml", e);
            return null;
        }

    }


    private XMLStreamWriter getXmlStreamWriter(StringWriter stringWriter) throws XMLStreamException {
        XMLOutputFactory xmlOutputFactory = XMLOutputFactory.newInstance();
        return xmlOutputFactory.createXMLStreamWriter(stringWriter);
    }


    private void writePhToXml(XMLStreamWriter xmlStreamWriter, StringWriter stringWriter, String phText) throws XMLStreamException {
        xmlStreamWriter.writeStartElement(PH);
        xmlStreamWriter.writeCharacters("");
        stringWriter.append(phText);
        xmlStreamWriter.writeEndElement();
    }


}

```







### 1.4 门面类

```java
@Slf4j
public class TmxHelper {
    TmxParser tmxParser;

    public TmxHelper(){
         this.tmxParser =  new SimpleTmxParser();
    }
    public TmxHelper(TmxParser tmxParser){
        this.tmxParser =  tmxParser;
    }

    public Tmx parse(InputStream inputStream) {
        return this.tmxParser .parse(inputStream);
    }

    public String toXmlString(Tmx tmx) {
        return this.tmxParser .toXmlString(tmx);
    }


}
```



### 1.5 测试类

```groovy
class TmxHelperSpec extends Specification {

    def "Parse"() {
        given: "an InputStream"
        InputStream inputStream = this.getClass().getResourceAsStream("/sample.tmx")

        and: "a TmxHelper"
        TmxHelper tmxHelper = new TmxHelper()

        when: "parse object from inputStream"
        def tmx = tmxHelper.parse(inputStream)
        println tmx

        then:
        tmx.body != null

    }


    def "toXmlString"() {
        given: "an Tmx instance"
        Tmx tmx = getTmx()

        and: "a TmxHelper"
        TmxHelper tmxHelper = new TmxHelper()

        when: "parse object to String"
        def str = tmxHelper.toXmlString(tmx)
        println str

        then:
        str != null

    }


    private Tmx getTmx(){
        InputStream inputStream = this.getClass().getResourceAsStream("/sample.tmx")
        TmxHelper tmxHelper = new TmxHelper()
        Tmx tmx = tmxHelper.parse(inputStream)
        return tmx
    }
}

```



















