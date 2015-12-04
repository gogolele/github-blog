title: Java Util Function
date: 2015-08-19 07:36:34
tags: java
categories: dev
---
Util.java

```java
package com.tonywutao.util;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.StringReader;
import java.io.StringWriter;
import java.net.Inet4Address;
import java.net.UnknownHostException;
import java.security.MessageDigest;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.SimpleTimeZone;
import java.util.TimeZone;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.transform.OutputKeys;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import javax.xml.xpath.XPath;
import javax.xml.xpath.XPathConstants;
import javax.xml.xpath.XPathExpression;
import javax.xml.xpath.XPathFactory;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.w3c.dom.Document;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import org.xml.sax.InputSource;
import com.google.common.io.Files;
import com.tonywutao.datafeed.config.CommonValue;

/**
 * Provide SHA Hashcode convert function, etc.
 *
 * @author tao wu
 *
 */
public class Util {
	private static final Log LOG = LogFactory.getLog(Util.class);
	final static String TIME_FORMAT = "yyyy-MM-dd hh:mm:ss";

	public static void main(String[] args) {
		Date d = new Date();
		System.out.print(date2String2(d));
	}

	/**
	 * HASH KEY GENERATOR
	 *
	 * @param item
	 * @return
	 * @throws Exception
	 */
	public static String gethashCode(String item) throws Exception {
		MessageDigest digest = MessageDigest.getInstance("SHA-256");
		byte[] itemEncoded = item.getBytes("UTF-8");
		byte[] hash = digest.digest(itemEncoded);
		int[] arr = new int[hash.length];

		for (int i = 0; i < hash.length; i++) {
			byte t = hash[i];
			int n = t < 0 ? t & 0xFF : t;
			arr[i] = n;
		}

		StringBuilder result = new StringBuilder();
		for (int b : arr) {
			result.append(FillStringToLength(Integer.toString(b, 16), "0", 2, true));
		}
		return result.toString();
	}

	public static String FillStringToLength(String sourceStr, String fillStr, int length, boolean frontFlag) {
		String tempStr = sourceStr;
		while (tempStr.length() < length) {
			if (frontFlag) {
				tempStr = fillStr + tempStr;
			} else {
				tempStr = tempStr + fillStr;
			}
		}

		if (tempStr.length() > length) {
			if (frontFlag) {
				tempStr = tempStr.substring(tempStr.length() - length, length);
			} else {
				tempStr = tempStr.substring(0, length);
			}
		}

		return tempStr;
	}

	// PDT -- UTC - 7
	// EDT -- UTC - 4

	private static void testTimeZone() throws ParseException {
		int timeZoneOffset = -4;

		if (timeZoneOffset > 13 || timeZoneOffset < -12) {
			timeZoneOffset = 0;
		}
		TimeZone timeZone;
		String[] ids = TimeZone.getAvailableIDs(timeZoneOffset * 60 * 60 * 1000);
		if (ids.length == 0) {
			// if no ids were returned, something is wrong. use default TimeZone
			timeZone = TimeZone.getDefault();
		} else {
			timeZone = new SimpleTimeZone(timeZoneOffset * 60 * 60 * 1000, ids[0]);
		}

		SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		SimpleDateFormat format2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		format.setTimeZone(timeZone);
		String ds = "2014-04-09 14:58:57";
		String ds2 = (new Date()).toString();
		String dateString = format.format(format2.parse(ds));
		// String dateString = format.format(new Date());
		System.out.println(dateString);
	}

	private static void testConvertTimeByZone() {
		String dateString = "2014-04-09 14:58:57";
		int timeZoneOffset = -7;
		try {
			String result = convertTimeByZone(dateString, timeZoneOffset);
			System.out.println(result);
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

	/**
	 * 1. time string, 2. e3 timezone(utc -7)
	 *
	 * @throws ParseException
	 */
	private static String convertTimeByZone(String strDate, int timeZoneOffset) throws ParseException {

		if (timeZoneOffset > 13 || timeZoneOffset < -12) {
			timeZoneOffset = 0;
		}
		TimeZone timeZone;
		String[] ids = TimeZone.getAvailableIDs(timeZoneOffset * 60 * 60 * 1000);
		if (ids.length == 0) {
			// if no ids were returned, something is wrong. use default TimeZone
			timeZone = TimeZone.getDefault();
		} else {
			timeZone = new SimpleTimeZone(timeZoneOffset * 60 * 60 * 1000, ids[0]);
		}

		// format for timezoneoffset
		SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		format.setTimeZone(timeZone);

		// format for datestring to Date
		SimpleDateFormat format2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		String ds = "2014-04-09 14:58:57";

		return format.format(format2.parse(ds));

	}

	public static void printLog(String str) {
//		System.out.println(str);
		LOG.info(str);
	}

	// time stamp convertor for Date
	public static String timestampToDate(long timestamp) {
		SimpleDateFormat sdf = new SimpleDateFormat("MM/dd/yyyy HH:mm:ss");
		java.util.Date dt = new Date(timestamp);
		String sDateTime = sdf.format(dt);
		return sDateTime;
	}

	public static int ip2Int(String ip) {
		Inet4Address inetAddress = null;
		try {
			inetAddress = (Inet4Address) Inet4Address.getByName(ip);
		} catch (UnknownHostException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		int result = 0;
		for (byte b : inetAddress.getAddress()) {
			result = result << 8 | (b & 0xFF);
		}
		return result;
	}

	public static long ip2Long(String ip) {
		Inet4Address inetAddress = null;
		try {
			inetAddress = (Inet4Address) Inet4Address.getByName(ip);
		} catch (UnknownHostException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		long result = 0;
		for (byte b : inetAddress.getAddress()) {
			result = result << 8 | (b & 0xFF);
		}
		return result;
	}

	public static String long2Ip(long ll) {
		StringBuffer sb = new StringBuffer("");
		sb.append(String.valueOf((ll >>> 24)));
		sb.append(".");
		sb.append(String.valueOf((ll & 0x00FFFFFF) >>> 16));
		sb.append(".");
		sb.append(String.valueOf((ll & 0x0000FFFF) >>> 8));
		sb.append(".");
		sb.append(String.valueOf((ll & 0x000000FF)));
		return sb.toString();
	}

	public static String wipeQuote(String a) {
		if (a.startsWith("\"") && a.endsWith("\"")) {
			a = a.replaceAll("\"", "");
		}
		return a;
	}

	public static String date2String(Date d) {
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
		return sdf.format(d);
	}

	public static String date2String2(Date d) {
		SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
		return sdf.format(d);
	}

	public static void writeToFile(String filename, List<String> lines) {
		if (lines == null || lines.size() == 0)
			return;
		FileWriter writer = null;
		try {
			writer = new FileWriter(filename, true);
			for (String s : lines) {
				writer.write(s + "\n");
			}
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			try {
				writer.close();
			} catch (Exception ex) {
			}
		}
	}
	public static void writeToFile(String filePath, String line) {
		FileWriter writer = null;
		try {
			writer = new FileWriter(filePath, true);
			writer.write(line+ "\n");
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			try {
				writer.close();
			} catch (Exception ex) {
			}
		}
	}

	public static void appendToXmlFile(String filePath, List<String> lines) {
		if(lines == null || lines.isEmpty() ) return;
		String destFile = filePath;
		String srcFile = getXmlTemplatePath(filePath);
		String nodePath = "";
		List<Node> lstNode = new ArrayList<Node>();
		for(String s : lines){
			Node node = string2Node(s);
			if(node != null){
				lstNode.add(node);
			}
		}
		if(!lstNode.isEmpty())appendXmlNode(srcFile, destFile,nodePath, lstNode);

	}

	public static long convert2long(String date) {
		try {
			SimpleDateFormat sf = new SimpleDateFormat(TIME_FORMAT);
			return sf.parse(date).getTime();
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return 0l;
	}

	public static Date str2Date(String dateStr) {
		Date d = null;
		SimpleDateFormat format = new SimpleDateFormat(TIME_FORMAT);

		try {
			format.setTimeZone(TimeZone.getTimeZone("America/Los_Angeles"));
			d = format.parse(dateStr);
		} catch (Exception e) {
			e.printStackTrace();
			Util.printLog("time string convert date error, " + e.getMessage());
		}
		return d;
	}

	public static String getFileNameByPath(String filePath) {
		if (filePath == null || filePath.length() == 0)
			return null;
		String[] arr = filePath.split("/");
		return arr[arr.length - 1];
	}

	public static String getFodlerByPath(String filePath) {
		if (filePath == null || filePath.length() == 0)
			return null;
		String[] arr = filePath.split("/");
		String s = "";
		for(int i=0; i<arr.length-1; i++){
			s += arr[i] + "/";
		}
		return s;
	}

	public static boolean checkFolderExist(String directoryPath) {
		boolean result = false;
		File theDir = new File(directoryPath);
		if (!theDir.exists()) {
			printLog("creating directory: " + directoryPath);
			try {
//				theDir.mkdir();
				theDir.mkdirs();
				result = true;
			} catch (SecurityException se) {
				printLog("error while creating directory " + directoryPath + ", e=" + se.getMessage());
			}
		}
		return result;
	}

	public static boolean moveFile(File srcPath, File descPath) {
		boolean b = false;
		try {
			Files.move(srcPath,  descPath);
			b = true;
		} catch (IOException e) {
			// TODO Auto-generated catch block
			printLog("moveFile Error, " + e.getMessage());
		}
		return b;
	}

	public static String getXmlTemplatePath(String xmlFilePath) {
		Util.checkFolderExist(CommonValue.PATH_DATAFEED_TEMPLATE);
		String templatePath = CommonValue.PATH_DATAFEED_TEMPLATE+"/" + Util.getFileNameByPath(xmlFilePath);
		return templatePath;
	}

	public static boolean getXmlTemplate(String xmlFilePath) {
		boolean b = false;
		try {
			DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
			Document document = dbf.newDocumentBuilder().parse(new File(xmlFilePath));
			XPathFactory xpf = XPathFactory.newInstance();
			XPath xpath = xpf.newXPath();
			XPathExpression expression = xpath.compile("/*/Message/Inventory/Item");
			// XPathExpression expression = xpath.compile("/*/MessageType");
			NodeList itemNode = (NodeList) expression.evaluate(document, XPathConstants.NODESET);
			for(int i=0; i< itemNode.getLength(); i++){
				itemNode.item(i).getParentNode().removeChild(itemNode.item(i));
			}
			TransformerFactory tf = TransformerFactory.newInstance();
			Transformer t = tf.newTransformer();
			String templatePath = getXmlTemplatePath(xmlFilePath);
			StreamResult result = new StreamResult(templatePath);
			t.transform(new DOMSource(document), result);
			b = true;
		} catch (Exception e) {
			Util.printLog(" template xml file generate error, e=" + e.getMessage());
		}
		return b;
	}

	public static void appendXmlNode(String srcPath, String descPath,  String nodePath, List<Node> lstNode) {
		try {
			DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
			Document document = dbf.newDocumentBuilder().parse(new File(srcPath));
			XPathFactory xpf = XPathFactory.newInstance();
			XPath xpath = xpf.newXPath();
			XPathExpression expression = xpath.compile("/*/Message/Inventory");
			// XPathExpression expression = xpath.compile("/*/MessageType");
			Node hostNode = (Node) expression.evaluate(document, XPathConstants.NODE);
			for(Node n : lstNode ){
				Node newNode = document.importNode(n, true);
				hostNode.appendChild(newNode);
			}
			TransformerFactory tf = TransformerFactory.newInstance();
			Transformer t = tf.newTransformer();
			StreamResult result = new StreamResult(descPath);
			t.transform(new DOMSource(document), result);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}


	public static String node2String(Node node) {
		StringWriter sw = new StringWriter();
		try {
			Transformer t = TransformerFactory.newInstance().newTransformer();
			t.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION, "yes");
			t.setOutputProperty(OutputKeys.INDENT, "yes");
			t.transform(new DOMSource(node), new StreamResult(sw));
		} catch (TransformerException te) {
			System.out.println("nodeToString Transformer Exception");
		}
		return sw.toString();
	}

	public static Node string2Node(String s){
//		String s = "<Item><SellerPartNumber>7500033888</SellerPartNumber><SellingPrice>229.99</SellingPrice><Inventory>34</Inventory><FulfillmentOption>Merchant</FulfillmentOption><ActivationMark>True</ActivationMark><MSRP>321.99</MSRP></Item>";
		javax.xml.parsers.DocumentBuilderFactory b = javax.xml.parsers.DocumentBuilderFactory.newInstance();
		b.setNamespaceAware(false);
		javax.xml.parsers.DocumentBuilder db = null;
		try {
			db = b.newDocumentBuilder();
			 Node node = db.parse(
				        new InputSource(new StringReader(s)))
				        .getDocumentElement();
			return node;
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			return null;
		}
	}
}
```

xml parse example
```java
private boolean loadXmlFile() {
		// NeweggEnvelope--root
		// Message --L1
		// Inventory --L2
		// Item --L3
		// Fields --L4
		boolean r = false;
		long startTime, endTime;
		startTime = System.currentTimeMillis();
		FileEntity fileEntity = new FileEntity();
		fileEntity.setFileId(DataFeedReader.getFileEntityId());
		fileEntity.setFilePath(this.filePath);
		fileEntity.setProcessStartTime(startTime);
		DataFeedReader.FILE_SET.put(fileEntity.getFileId(), fileEntity);

		MktItemEntity entity = null;
		NodeList nodesL1 = null;
		NodeList nodesL2 = null;
		NodeList nodesL3 = null;
		NodeList nodesL4 = null;
		try {
			DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
			DocumentBuilder builder = factory.newDocumentBuilder();
			Document doc = builder.parse(this.filePath);
			Element root = doc.getDocumentElement();
			nodesL1 = root.getChildNodes();

			for (int i = 0; i < nodesL1.getLength(); i++) {
				if (nodesL1.item(i).getNodeName().equals("Message")) {
					nodesL2 = nodesL1.item(i).getChildNodes();
					for (int k = 0; k < nodesL2.getLength(); k++) {
						if (nodesL2.item(k).getNodeName().equals("Inventory")
								|| nodesL2.item(k).getNodeName().equals("Price")) {
							nodesL3 = nodesL2.item(k).getChildNodes();
							for (int t = 0; t < nodesL3.getLength(); t++) {
								nodesL4 = nodesL3.item(t).getChildNodes();
								rowCount++;
								entity = transformXmlNode(nodesL4);
								entity.setFileEntityId(fileEntity.getFileId());
								entity.setLine(Util.node2String(nodesL3.item(t))); // to
																					// xml
								// string?
								if (entity != null && !entity.getSpn().equals(CommonValue.DEFAULT_STR)) {
									DataFeedReader.ITEM_QUEUE.put(entity);
								}
								entity = null;
								if (rowCount % 1000 == 0) {
									Util.printLog("filePath=" + Util.getFileNameByPath(filePath) + ", rowCount="
											+ rowCount + " rows," + " TOTAL="
											+ DataFeedReader.getAtomicCounter(DataFeedReader.CounterType.TOTAL)
											+ ", INVALID="
											+ DataFeedReader.getAtomicCounter(DataFeedReader.CounterType.INVALID)
											+ ", READ QUEUE = " + DataFeedReader.ITEM_QUEUE.size() + ", FILE QUEUE="
											+ DataFeedReader.FILE_QUEUE.size());
								}
							}
						}
					}
				}
			}
			Util.printLog("[read completed] filePath=" + Util.getFileNameByPath(filePath) + ", rowCount=" + rowCount
					+ " rows," + " TOTAL=" + DataFeedReader.getAtomicCounter(DataFeedReader.CounterType.TOTAL)
					+ ", INVALID=" + DataFeedReader.getAtomicCounter(DataFeedReader.CounterType.INVALID)
					+ ", READ QUEUE = " + DataFeedReader.ITEM_QUEUE.size() + ", FILE QUEUE="
					+ DataFeedReader.FILE_QUEUE.size());
			r = true;
		} catch (Exception e) {
			r = false;
			e.printStackTrace();
			Util.printLog("file process error, [" + this.filePath + "], " + e.getMessage());
		} finally {
			endTime = System.currentTimeMillis();
			DataFeedReader.FILE_SET.get(fileEntity.getFileId()).setProcessEndTime(endTime);
			DataFeedReader.FILE_SET.get(fileEntity.getFileId()).setProcessSuccess(r);
			return r;
		}
	}

	private boolean loadExcelFile() {
		boolean b = true;
		// need to implement
		return b;
	}

	private MktItemEntity transformXmlNode(NodeList nodeList) {
		if (nodeList == null || nodeList.getLength() == 0)
			return null;

		try {
			DataFeedReader.increAtomicCounter(DataFeedReader.CounterType.TOTAL);
			MktItemEntity entity = new MktItemEntity();
			entity.setLine(nodeList.toString());
			Node node = null;
			String nodeName = null;
			String nodeValue = null;
			for (int i = 0; i < nodeList.getLength(); i++) {
				node = nodeList.item(i);
				nodeName = node.getNodeName().trim();
				nodeValue = node.getTextContent().trim();
				entity.setSchemaValue(nodeName, nodeValue);
			}
			return entity;
		} catch (Exception e) {
			DataFeedReader.increAtomicCounter(DataFeedReader.CounterType.INVALID);
			e.printStackTrace();
			return null;
		}
	}
}
```

-EOF-

TAO WU@CA, USA
