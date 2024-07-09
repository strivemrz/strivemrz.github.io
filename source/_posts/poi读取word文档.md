---
updated: 2023-02-28
created: 2023-02-28
title: poi操作word文档
categories: poi
tags:
  - poi
excerpt: ''
cover: https://img.bizhizu.com/2014/0701/20140701023021648.jpg
---

#### 导入依赖

~~~xml
<!--  apache  poi-->
<!-- poi -->
		<!--word操作工具类-->
		<dependency>
			<groupId>com.deepoove</groupId>
			<artifactId>poi-tl</artifactId>
			<version>1.6.0-beta1</version>
		</dependency>

		<!-- POI-word文件处理需要 -->
		<!-- WordToHtml .doc .odcx  poi  -->
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-scratchpad</artifactId>
			<version>4.1.2</version>
<!--			<version>3.17</version>-->
		</dependency>

		<!-- 操作excel的库 注意版本保持一致 poi poi-ooxml  poi-scratchpad -->
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi</artifactId>
			<version>4.1.2</version>
<!--			<version>3.17</version>-->
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml -->
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml</artifactId>
			<version>4.1.2</version>
<!--			<version>3.17</version>-->
		</dependency>

		<dependency>
			<groupId>fr.opensagres.xdocreport</groupId>
			<artifactId>fr.opensagres.poi.xwpf.converter.xhtml</artifactId>
			<version>2.0.2</version>
		</dependency>
~~~

#### 工具类代码

~~~java
package com.lq.file.word;
 
/**
 * <p>Description:POIUtil 工具类</p>
 * <p>Copyright: Copyright (c)2019</p>
 * <p>Company: Tope</p>
 * <P>@version 1.0</P>
 */
import org.apache.poi.POIXMLDocument;
import org.apache.poi.POIXMLTextExtractor;
import org.apache.poi.hwpf.extractor.WordExtractor;
import org.apache.poi.openxml4j.opc.OPCPackage;
import org.apache.poi.xwpf.extractor.XWPFWordExtractor;
 
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;
 
public class POIUtil {
 
	/**
	* @Description: POI 读取  word
	* @create: 2019-07-27 9:48
	* @update logs
	* @throws Exception
	*/
	public static List<String> readWord(String filePath) throws Exception{
		
		List<String> linList = new ArrayList<String>();
		String buffer = "";
		try {
			if (filePath.endsWith(".doc")) {
				InputStream is = new FileInputStream(new File(filePath));
				WordExtractor ex = new WordExtractor(is);
				buffer = ex.getText();
				ex.close();
				
				if(buffer.length() > 0){
					//使用回车换行符分割字符串
					String [] arry = buffer.split("\\r\\n");
					for (String string : arry) {
						linList.add(string.trim());
					}
				}
			} else if (filePath.endsWith(".docx")) {
				OPCPackage opcPackage = POIXMLDocument.openPackage(filePath);
				POIXMLTextExtractor extractor = new XWPFWordExtractor(opcPackage);
				buffer = extractor.getText();
				extractor.close();
				
				if(buffer.length() > 0){
					//使用换行符分割字符串
					String [] arry = buffer.split("\\n");
					for (String string : arry) {
						linList.add(string.trim());
					}
				}
			} else {
				return null;
			}
			
			return linList;
		} catch (Exception e) {
			System.out.print("error---->"+filePath);
			e.printStackTrace();
			return null;
		}
	}
 
}
~~~

#### doc/docx转html格式

```java
package com.wedowork.common.utils;

import cn.hutool.core.io.FileUtil;
import fr.opensagres.poi.xwpf.converter.xhtml.XHTMLConverter;
import fr.opensagres.poi.xwpf.converter.xhtml.XHTMLOptions;
import lombok.Cleanup;
import lombok.Getter;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;
import org.apache.poi.hwpf.HWPFDocument;
import org.apache.poi.hwpf.converter.WordToHtmlConverter;
import org.apache.poi.hwpf.usermodel.Paragraph;
import org.apache.poi.hwpf.usermodel.Range;
import org.apache.poi.ss.formula.functions.T;
import org.apache.poi.xwpf.usermodel.XWPFDocument;
import org.apache.poi.xwpf.usermodel.XWPFParagraph;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.multipart.MultipartFile;
import org.w3c.dom.Document;

import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.*;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

@Slf4j
@Setter
@Getter
public class WordUtils {

    //在静态变量使用@Value 加@Component注解，@Value写在set方法上
    @Value("${file.tempPath}")
    private static String tempPath;


    public static File inputstreamToFile(InputStream input){
        File file = new File("/word");
        try {
            OutputStream os = new FileOutputStream(file);
            int bytesRead = 0;
            byte[] buffer = new byte[8192];
            while ((bytesRead = input.read(buffer, 0, 8192)) != -1) {
                os.write(buffer, 0, bytesRead);
            }
            os.close();
            input.close();
            return file;
        }catch (Exception e){
            log.error("转换出现错误：",e);
        }
        return file;
    }
    public static String WordToHtml(MultipartFile file) throws IOException {
        String originalFilename = file.getOriginalFilename();
        String extension=originalFilename.substring(originalFilename.lastIndexOf("."));
        InputStream ips = file.getInputStream();
        File file1=null;
            try {
                if (ips == null) {
                    return "";
                }
                //判断文档为doc或docx，分别进行处理转换
                if (extension.equals(".doc") || extension.equals(".DOC")) {
                    HWPFDocument wordDocument = new HWPFDocument(ips);
                    WordToHtmlConverter wordToHtmlConverter = new WordToHtmlConverter(
                            DocumentBuilderFactory.newInstance().newDocumentBuilder()
                                    .newDocument());
                    //该部分为doc中的图片操作（如doc中存在图片），可将图片上传服务器和获取图片URL的逻辑放在重写的位置
                    //参数说明：savePicture(图片byte, 图片type, 图片名, 图片宽度, 图片高度)
                    wordToHtmlConverter.processDocument(wordDocument);

                    Document htmlDocument = wordToHtmlConverter.getDocument();
                    ByteArrayOutputStream out = new ByteArrayOutputStream();
                    DOMSource domSource = new DOMSource(htmlDocument);
                    StreamResult streamResult = new StreamResult(out);
                    TransformerFactory tf = TransformerFactory.newInstance();
                    //设置转换参数
                    Transformer serializer = tf.newTransformer();
                    serializer.setOutputProperty(OutputKeys.ENCODING, "UTF-8");
                    serializer.setOutputProperty(OutputKeys.INDENT, "yes");
                    serializer.setOutputProperty(OutputKeys.METHOD, "html");
                    serializer.transform(domSource, streamResult);
                    String result = new String(out.toByteArray(),StandardCharsets.UTF_8);
                    ips.close();
                    out.close();
                    wordDocument.close();
                    result = result.replaceAll("\\r\\n", "<br/>");
                    return result;
                }else if (extension.equals(".docx") || extension.equals(".DOCX")){
                    file1=inputstreamToFile(ips);
                    //poi操作docx文件只能转成FileInputStream流
                    // 1)加载word文档生成XWPFDocument对象
                    XWPFDocument document = new XWPFDocument(new FileInputStream(file1));

                    // 2)解析XHTML配置
                    XHTMLOptions options = XHTMLOptions.create();
                    options.setIgnoreStylesIfUnused(false);
                    options.setFragment(true);
                    // 3)将XWPFDocument转换成XHTML
                    @Cleanup ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    XHTMLConverter.getInstance().convert(document, baos, options);
                    String html = new String(baos.toByteArray(), StandardCharsets.UTF_8);
                    baos.close();
                    document.close();
                    file1.delete();
                    return html;
                }
            } catch (Exception e){
                log.error("word转html错误:{}");
                e.printStackTrace();
            }
        return "";
    }

    public static<T> List<T> getParagraph(MultipartFile file) throws IOException {
        String originalFilename = file.getOriginalFilename();
        String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
        if(Objects.requireNonNull(extension.equals(".doc"))||extension.equals(".DOC")){
            // 读取doc，将doc加载进对象 T为Paragraph
            InputStream inputStream = file.getInputStream();
            File file1 = inputstreamToFile(inputStream);

            HWPFDocument document = new HWPFDocument(new FileInputStream(file1));
            Range range = document.getRange();
            List<T> paragraphs = new ArrayList<>();
            for(int i=0;i<range.numParagraphs();i++){
                paragraphs.add((T)range.getParagraph(i));
            }
            return paragraphs;
        }else if(Objects.requireNonNull(extension.equals(".docx"))||extension.equals(".DOCX")){
            // 读取docx，将docx加载进对象 T为String
            InputStream inputStream = file.getInputStream();

            List<T> paragraphText =new ArrayList<>();
            XWPFDocument document = new XWPFDocument(inputStream);

            List<XWPFParagraph> paragraphs = document.getParagraphs();
            for (XWPFParagraph paragraph:paragraphs){
                paragraphText.add((T)paragraph.getParagraphText());
            }
            return paragraphText;
        }
        return null;
    }
}
```

#### 检索h1~h6目录

```java
public String covertToLawsCatalog(Long id, String lawText) {
    StringBuilder lawContent = new StringBuilder(lawText);
    //删除原有的目录
    deletePreCatalogByLawId(id);
    // begTag=<h1> endTag=</h1>
    //提取目录
    List<AnalogyCaseCatalogue> list=new ArrayList<>();
    int index=0;
    int end=0;
    String begStr = "<h[1-6]{1}>";
    String endStr="</h[1-6]{1}>";
    Pattern begin = Pattern.compile(begStr);
    Pattern ends = Pattern.compile(endStr);
    Matcher begMatch = begin.matcher(lawContent.toString());
    Matcher endMatch = ends.matcher(lawContent.toString());

    while (begMatch.find(index)){
        AnalogyCaseCatalogue lawsCatalogue = new AnalogyCaseCatalogue();
        //设置law_id
        lawsCatalogue.setLawsId(id);

        index=begMatch.start();
        boolean b = endMatch.find(end);
        int end2=endMatch.start();
        String substring = lawContent.substring(index+4, end2);

        lawsCatalogue.setCatalogueName(substring);
        String random = UUID.randomUUID().toString();
        lawsCatalogue.setCatalogueSign(random);
        lawContent=lawContent.insert(index+3," id=\""+random+"\"");
        begMatch = begin.matcher(lawContent.toString());
        endMatch = ends.matcher(lawContent.toString());
        list.add(lawsCatalogue);

        index=index+random.length()+substring.length()+15;
        end=end+random.length()+substring.length()+15;
    }

    return lawContent.toString();
}
```