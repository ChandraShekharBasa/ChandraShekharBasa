package com.scw.directoryService.service.xmlService;

import com.scw.directoryService.model.sqlEntity.Report;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.xml.sax.SAXException;

import javax.xml.XMLConstants;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.File;
import java.io.IOException;
import java.util.Optional;
import java.util.logging.Level;
import java.util.logging.Logger;

public class XmlReportService {

    private static final Logger logger = Logger.getLogger(XmlReportService.class.getName());

    public Optional<Report> getReportFromXml(File file) {
        try {
            DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
            factory.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "all");

            DocumentBuilder builder = factory.newDocumentBuilder();
            Document document = builder.parse(file);

            document.getDocumentElement().normalize();
            Element element = document.getDocumentElement();

            return Optional.of(new Report(
                element.getElementsByTagName("author").item(0).getTextContent(),
                element.getElementsByTagName("title").item(0).getTextContent(),
                element.getElementsByTagName("theses").item(0).getTextContent(),
                element.getElementsByTagName("text").item(0).getTextContent()
            ));

        } catch (SAXException | IOException | ParserConfigurationException exception) {
            logger.log(Level.SEVERE, "error while creating report at xml format \n", exception);
        }
        return Optional.empty();
    }


    public boolean createXMLReportFromObject(Report report, String path,
                                             String fileName) {
        try {
            Document document = DocumentBuilderFactory
                .newInstance()
                .newDocumentBuilder()
                .newDocument();
            Element reportTag = document.createElement("report");
            document.appendChild(reportTag);

            Element author = document.createElement("author");
            author.setTextContent(report.getAuthor());
            reportTag.appendChild(author);

            Element title = document.createElement("title");
            title.setTextContent(report.getTitle());
            reportTag.appendChild(title);

            Element theses = document.createElement("theses");
            theses.setTextContent(report.getTheses());
            reportTag.appendChild(theses);

            Element text = document.createElement("text");
            text.setTextContent(report.getText());
            reportTag.appendChild(text);

            Transformer transformer = TransformerFactory.newInstance().newTransformer();
            DOMSource source = new DOMSource(document);
            StreamResult result = new StreamResult(new File(path + "\\" + fileName));
            transformer.transform(source, result);

            return true;

        } catch (ParserConfigurationException | TransformerException exception){
            logger.log(Level.SEVERE, "incorrectly create xml-report from .bin file \n", exception);
            return false;
        }
    }

}
