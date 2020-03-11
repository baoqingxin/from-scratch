# DOM
## 什么是DOM?
文档对象模型(Document Object Model简写DOM)是HTML和XML文档的编程接口。它提供了对文档的结构化的表达，并定义了一种方式可以使从程序中对该结构进行访问，从而改变文档的结构，样式和内容。DOM将文档解析为一个由节点和对象(包含属性和方法的对象)组成的结构集合。简言之，它会将web页面和叫脚本或程序语言连接起来。

## 节点类型
DOM可以将任何HTML或XML文档描绘成一个由多层次节点构成的结构。它有下列几种类型：
1. Node类型
DOM1级定义了一个Node接口，该接口将由DOM中的所有节点类型实现。这个Node接口在JS中是作为Node类型实现的，而且JS中的所有节点类型都继承自Node类型，也因此所有节点类型都共享着相同的基本属性和方法。  
节点类型共12种：
* Node.ELEMENT_NODE(1) 一个元素节点，例如`<p>`和`<div>`。
* Node.ATTRIBUTE_NODE(2) 元素的耦合属性在DOM4规范里Node接口将不再实现这个元素属性
* Node.TEXT_NODE(3) Element 或者 Attr 中实际的  文字
* Node.CDATA_SECTION_NODE(4) 	一个 CDATASection，例如`<!CDATA[[ … ]]>`。(XML中)
* Node.PROCESSING_INSTRUCTION_NODE(7) 一个用于XML文档的 ProcessingInstruction ，例如 <?xml-stylesheet ... ?> 声明。
* Node.COMMENT_NODE(8) 一个 Comment 节点。(注释)
* Node.DOCUMENT_NODE(9) 一个 Document 节点。
* Node.DOCUMENT_TYPE_NODE(10) 描述文档类型的 DocumentType 节点。例如 <!DOCTYPE html>  就是用于 HTML5 的。
* Node.DOCUMENT_FRAGMENT_NODE(11) 一个 DocumentFragment 节点
* Node.ENTITY_REFERENCE_NODE(5) 一个 XML 实体引用节点。 在 DOM4 规范里被移除。
* Node.ENTITY_NODE(6) 一个 XML <!ENTITY ...>  节点。 在 DOM4 规范中被移除。
* Node.NOTATION_NODE(12) 一个 XML <!NOTATION ...> 节点。 在 DOM4 规范里被移除.
 
## DOM操作
### 获取节点
1. document
* getElementById
* getElementByName
* getELementByTagName
* querySelector
* querySelectAll

2. 节点指针
* firstChild
* lastChild
* childNodes
* previousSibling
* nextSibling
* parentNode

### 节点操作
1. 创建节点
* createElement
* createAttribute
* createTextNode

2. 插入节点
* appendChild
* insertBefore

3. 替换节点
* replaceChild

4. 复制节点 
* cloneNode

5. 删除节点
* removeChild

### 属性操作
1. 获取属性
* getAttribute

2. 设置属性
* setAttribute
3. 删除属性
* removeAttribute

### 文本操作
* insertData
* appendData
* deleteData
* replaceData
* splitData
* substring