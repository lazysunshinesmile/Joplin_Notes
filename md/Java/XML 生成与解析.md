XML 生成与解析

XML 生成与解析

# 生成xml文件
## 相关变量初始化
```
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document document = builder.newDocument();
TransformerFactory tff = TransformerFactory.newInstance();
Transformer tf = tff.newTransformer();
```
## 添加文件相关属性
```java
// 设置是否自动生成头，设置为true，不自动生成头，<?xml version="1.0" encoding="UTF-8" standalone="no"?>
tf.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION, "yes");
//设置是否自动换行，只能为yes或者no，yes表示自动换行。
tf.setOutputProperty(OutputKeys.INDENT, "yes");
//设置xml版本
tf.setOutputProperty(OutputKeys.VERSION, "1.0");
//设置是否生成独立的文件说明，只能为yes或者no，设置为yes即没有dtd和schema作为该XML的说明文档，且不显示该属性，该属性还有待考究
tf.setOutputProperty(OutputKeys.STANDALONE, "yes");
//其他属性未测试，自己google（/javax/xml/transform/OutputKeys.java）
    public static final String METHOD = "method";
    public static final String DOCTYPE_PUBLIC = "doctype-public";
    public static final String DOCTYPE_SYSTEM = "doctype-system";
    public static final String CDATA_SECTION_ELEMENTS = "cdata-section-elements";
    public static final String MEDIA_TYPE = "media-type";
```

## 生成Document
```
Element phonebook = document.createElement("phonebook");
phonebook.setAttribute("type", "mycreate");
Element contact = document.createElement("contact");
contact.setTextContent("sunxiang");
phonebook.appendChild(contact);
// 这句话一定要加上，关联各个节点的
document.appendChild(phonebook);
```
*生成以下xml文件*
```
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<phonebook type="mycreate">
<contact>sunxiang</contact>
</phonebook>
```

# 解析xml文件
## 相关变量初始化
```
DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = builderFactory.newDocumentBuilder();
InputStream fis = null;
BufferedReader br = null;
Document document;
```
## 开始解析
```
fis = new FileInputStream(file);
br= new BufferedReader(new InputStreamReader(fis)); 
InputSource inputSource = new InputSource(br);
inputSource.setEncoding(sConfiguration.getCharset());
document = builder.parse(inputSource);  
NodeList rootNodeList = document.getChildNodes();
for(int i=0; i<rootNodeList.getLength(); i++) {
    Node node = rootNodeList.item(i);
    //具体的NodeType有哪些，查看org.w3c.dom.Node类。
    System.out.println("node type:" + node.getNodeType());
    System.out.println("node name:" + node.getNodeName());
}
```


















