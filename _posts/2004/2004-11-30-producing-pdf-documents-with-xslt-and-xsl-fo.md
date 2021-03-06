---
layout: default
title: Producing PDF documents with XSLT and XSL:FO
tags: xsl xslt xsl:fo xml pdf
comments: true
---
# Producing PDF documents with XSLT and XSL:FO

[XSLT](https://www.w3.org/TR/xslt) and [XSL:FO](https://www.w3.org/TR/xsl/) are W3C standards that can be used in conjunction to create production quality PDF documents on the fly. The document generation process can be depicted using a simple flow diagram

```mermaid
graph LR
  xml[XML Data]
  xsl[XSL Template]
  xslt[XSL Transformation]
  style xslt fill:#caffca
  xslfo[XSL:FO Document]
  fop[XSLF:FO Processing]
  style fop fill:#caffca
  pdf[PDF Document]
  xml --> xslt
  xsl --> xslt
  xslt --> xslfo
  xslfo --> fop
  fop --> pdf
```

The document generation process occurs in the following steps

1. XSL Transformation

    In this step an XSL template containing XSLT instructions is used to process XML data and generate output in a technology independent formatting scheme called XSL:FO. XSL:FO contains enough content and formatting information to produce documents in a variety of different formats such as PDF, PostScript, RTF, and so on. This step is carried out by an XSLT processor such as [Xalan Java](https://xml.apache.org/xalan-j/) or MSXML.

2. XSL:FO processing

    The XSL:FO document generated by the preceding step is by itself not enough to generate printed output, mainly because few printers support it (as opposed to PostScript). This means that the XSL:FO document must be further processed to generate a document that can be easily distributed and printed. PDF is obviously the de facto standard in this space. This step is carried out by an XSL:FO processor such as [Apache FOP](https://xmlgraphics.apache.org/fop/) or [RenderX XEP Engine](http://services.renderx.com/Content/tools/xep.html).

We will now demonstrate the process and the tools required to develop and transform an XSL template to a finished PDF document.

## The XML data

The first step in developing an XSL template is to understand how the input XML data is organized. Having a stable well-defined XML schema of the input document will go a long way in creating a good XSL template. The XML data itself can be generated by several means

1. An application can serialize an object tree using an XML marshalling framework such as [Castor XML](http://castor.exolab.org/xml-framework.html).

2. An application can generate an XML document programmatically using XML DOM (Document Object Model) API or by directly writing to a text file.

3. An end user can produce XML documents using applications such as Microsoft Office, OpenOffice.org, Altova Authentic, Altova XML Spy, etc.

In our example, we will use the following XML document

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Order reference="12343-AHSHE-314159">
<Client>
        <Name>Jean Smith</Name>
        <Address>2000, Alameda de las Pulgas, San Mateo, CA 94403</Address>
    </Client>
    <Item reference="RF-0001">
        <Description>Lamb Chops</Description>
        <Quantity>10</Quantity>
        <UnitPrice>8.95</UnitPrice>
        <image>RF-0001.jpg</image>
    </Item>
    <Item reference="RF-0034">
        <Description>Chocolate</Description>
        <Quantity>5</Quantity>
        <UnitPrice>28.50</UnitPrice>
        <image>RF-0034.jpg</image>
    </Item>
    <Item reference="RF-3341">
        <Description>Cookie</Description>
        <Quantity>30</Quantity>
        <UnitPrice>0.85</UnitPrice>
        <image>RF-3341.jpg</image>
    </Item>
</Order>
```

This document corresponds to the following object model

```mermaid
classDiagram
    Client "1" --> "many" Order : places
    Order "1" --> "many" Item : has
    Order : string reference
    Client : string name
    Client : string address
    Item : string reference
    Item : string description
    Item : int quantity
    Item : double unitPrice
    Item : string image
```

## The XSL Template

The following XSL template transforms the XML document shown previously into an XSL:FO document, which can then be transformed to a PDF document using FOP.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" 
    xmlns:fo="http://www.w3.org/1999/XSL/Format">
  <xsl:param name="imagePath">file:///C:\data\work\java\fo\docs\examples\images\</xsl:param>
  <xsl:template match="/Order">
    <fo:root xmlns:fo="http://www.w3.org/1999/XSL/Format">
      <xsl:call-template name="pageLayout"/>
      <xsl:call-template name="pageContent"/>
    </fo:root>
  </xsl:template>
  <xsl:template name="pageLayout">
    <fo:layout-master-set>
      <fo:simple-page-master master-name="pagemaster" page-width="21cm" 
          page-height="29.7cm" margin-top="0.1cm" margin-bottom="0.1cm">
        <fo:region-body region-name="body" margin-left="1cm" margin-top="1.5cm" 
            margin-right="1cm" margin-bottom="1cm"/>
        <fo:region-before extent="1cm"/>
        <fo:region-after extent="1cm"/>
      </fo:simple-page-master>
    </fo:layout-master-set>
  </xsl:template>
  <xsl:template name="pageContent">
    <fo:page-sequence master-reference="pagemaster">
      <xsl:call-template name="pageHeader"/>
      <xsl:call-template name="pageFooter"/>
      <xsl:call-template name="pageBody"/>
    </fo:page-sequence>
  </xsl:template>
  <xsl:template name="pageHeader">
    <fo:static-content flow-name="xsl-region-before" display-align="after" 
        margin-left="1cm" margin-right="1cm" font-family="ZapfDingbats">
      <fo:block>
        <fo:table>
          <fo:table-column/>
          <fo:table-column/>
          <fo:table-body>
            <fo:table-row>
              <fo:table-cell>
                <fo:block>Order Detail</fo:block>
              </fo:table-cell>
              <fo:table-cell>
                <fo:block text-align="end">Page <fo:page-number/>
                </fo:block>
              </fo:table-cell>
            </fo:table-row>
          </fo:table-body>
        </fo:table>
      </fo:block>
      <fo:block>
        <fo:leader leader-length="100%" leader-pattern="rule" 
            rule-thickness="0.5pt" color="black"/>
      </fo:block>
    </fo:static-content>
  </xsl:template>
  <xsl:template name="pageFooter">
    <fo:static-content flow-name="xsl-region-after" display-align="after" 
        margin-left="1cm" margin-right="1cm">
      <fo:block>
        <fo:leader leader-length="100%" leader-pattern="rule" 
            rule-thickness="0.5pt" color="black"/>
      </fo:block>
      <fo:block>Reference # <xsl:value-of select="@reference"/>
      </fo:block>
    </fo:static-content>
  </xsl:template>
  <xsl:template name="pageBody">
    <fo:flow flow-name="body">
      <xsl:call-template name="clientDetail"/>
      <xsl:call-template name="orderDetail"/>
    </fo:flow>
  </xsl:template>
  <xsl:template name="clientDetail">
    <fo:block>
      <xsl:value-of select="Client/Name"/>
    </fo:block>
    <fo:block padding-after="1cm">
      <xsl:value-of select="Client/Address"/>
    </fo:block>
  </xsl:template>
  <xsl:template name="orderDetail">
    <fo:block padding-after="1cm">
      <xsl:call-template name="itemDetail"/>
    </fo:block>
  </xsl:template>
  <xsl:template name="itemDetail">
    <fo:table>
      <fo:table-column column-width="2cm"/>
      <fo:table-column column-width="4cm"/>
      <fo:table-column column-width="3cm"/>
      <fo:table-column column-width="3cm"/>
      <fo:table-column column-width="4cm"/>
      <xsl:call-template name="itemDetailTableHeader"/>
      <xsl:call-template name="itemDetailTableBody"/>
    </fo:table>
  </xsl:template>
  <xsl:template name="itemDetailTableHeader">
    <fo:table-header>
      <fo:table-row>
        <fo:table-cell border-style="solid">
          <fo:block>Ref #</fo:block>
        </fo:table-cell>
        <fo:table-cell border-style="solid">
          <fo:block>Description</fo:block>
        </fo:table-cell>
        <fo:table-cell border-style="solid">
          <fo:block text-align="center">Quantity</fo:block>
        </fo:table-cell>
        <fo:table-cell border-style="solid">
          <fo:block text-align="center">Unit Price</fo:block>
        </fo:table-cell>
        <fo:table-cell border-style="solid">
          <fo:block>Image</fo:block>
        </fo:table-cell>
      </fo:table-row>
    </fo:table-header>
  </xsl:template>
  <xsl:template name="itemDetailTableBody">
    <fo:table-body>
      <xsl:for-each select="Item">
        <xsl:call-template name="itemDetailTableRow"/>
      </xsl:for-each>
    </fo:table-body>
  </xsl:template>
  <xsl:template name="itemDetailTableRow">
    <xsl:element name="fo:table-row">
      <xsl:if test="boolean(position() mod 2)">
        <xsl:attribute name="background-color">silver</xsl:attribute>
      </xsl:if>
      <fo:table-cell border-style="solid">
        <fo:block>
          <xsl:value-of select="@reference"/>
        </fo:block>
      </fo:table-cell>
      <fo:table-cell border-style="solid">
        <fo:block>
          <xsl:value-of select="Description"/>
        </fo:block>
      </fo:table-cell>
      <fo:table-cell border-style="solid">
        <fo:block text-align="center">
          <xsl:value-of select="Quantity"/>
        </fo:block>
      </fo:table-cell>
      <fo:table-cell border-style="solid">
        <fo:block text-align="center">
          <xsl:value-of select="UnitPrice"/>
        </fo:block>
      </fo:table-cell>
      <fo:table-cell border-style="solid">
        <fo:block margin-left="2pt">
          <xsl:element name="fo:external-graphic">
            <xsl:attribute name="src">
              <xsl:value-of select="$imagePath"/><xsl:value-of select="image"/>
            </xsl:attribute>
          </xsl:element>
        </fo:block>
      </fo:table-cell>
    </xsl:element>
  </xsl:template>
</xsl:stylesheet>
```

The execution of an XSL template starts at the first `xsl:template` element that matches an element in the XML data. The match constraints are specified in the match attribute using an [XPath](https://www.w3.org/TR/xpath) expression. XPath is also used in the select attribute of an `xsl:value-of` element, in the test attribute of the `xsl:if` element, in the select attribute of `xsl:for-each` element, etc. XPath expression language is a rich language for selecting nodes for processing, specifying conditions for different ways of processing a node, and generating text to be inserted into the resulting document.

The `pageLayout` template sets the layout of the output page. The `pageContent` template generates the content of the document.

Page number citations, to build a table of content for example, can be added using the `fo:page-number-citation` element.

FOP supports the following typefaces by default - Courier, Helvetica, Symbol, Times, and ZapfDingbats. These typefaces also render without any problem in Adobe Acrobat Reader. Additional [fonts](https://xmlgraphics.apache.org/fop/1.0/fonts.html) can be added to FOP.

[FOP extensions](https://xmlgraphics.apache.org/fop/1.0/extensions.html) can be used to add special features like bookmarks to PDF documents.

## Producing PDF

If you have FOP configured and running on your machine, the PDF document can be generated by issuing the following command

```cmd
fop -xsl template.xsl -xml data.xml -pdf out.pdf
```

The `out.pdf` file as seen in Adobe Acrobat Reader is shown below.

![Output PDF](/assets/img/apache-fop-pdf.png)

A sample Java class to produce the PDF document using the FOP API is shown below.

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;

import javax.xml.transform.Result;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.sax.SAXResult;
import javax.xml.transform.sax.SAXSource;
import javax.xml.transform.stream.StreamSource;

import org.apache.fop.apps.Driver;
import org.xml.sax.InputSource;

public class GeneratePDF {

    public static void main(String[] args) {
        if (args.length == 4) {
            xslToPDF(args[0], args[1], args[2], args[3]);
        }
        else if (args.length == 2) {
            foToPDF(args[0], args[1]);
        }
        else {
            System.err.println("Usage:");
            System.err.println("\tjava GeneratePDF "
                    + "<xml file> <xsl template> <path to images> <pdf file>");
            System.err.println("OR");
            System.err.println("\tjava GeneratePDF <fo file> <pdf file>");
        }
    }

    public static void xslToPDF(String xml, String xsl, String imagePath,
            String pdf) {
        try {
            // FOP Driver setup
            Driver driver = new Driver();
            driver.setRenderer(Driver.RENDER_PDF);
            FileOutputStream pdfStream = new FileOutputStream(pdf);
            driver.setOutputStream(pdfStream); // PDF output file
            Result saxResult = new SAXResult(driver.getContentHandler());
            // XSL transformer setup
            TransformerFactory factory = TransformerFactory.newInstance();
            FileInputStream xslStream = new FileInputStream(xsl);
            Transformer transformer = factory.newTransformer(new StreamSource(
                    xslStream));
            transformer.setParameter("imagePath", imagePath);
            // XML data
            FileInputStream xmlDataStream = new FileInputStream(xml);
            InputSource inputSource = new InputSource(xmlDataStream);
            SAXSource saxSource = new SAXSource(inputSource);
            // Do XSL Transform driving the results to FOP
            transformer.transform(saxSource, saxResult);
        }
        catch (Exception e) {
            e.printStackTrace(System.err);
        }
    }

    public static void foToPDF(String fo, String pdf) {
        try {
            Driver driver = new Driver(new InputSource(fo),
                    new FileOutputStream(pdf));
            driver.setRenderer(Driver.RENDER_PDF);
            driver.run();
        }
        catch (Exception e) {
            e.printStackTrace(System.err);
        }
    }
}
```

## Advantages and Limitations

XSL tools are still evolving. Tools like FOP still [do not support](https://xmlgraphics.apache.org/fop/compliance.html) everything that the specification has to offer.
