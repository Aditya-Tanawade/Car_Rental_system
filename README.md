
        <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>


<!--    &lt;!&ndash; OpenHTMLToPDF (converts HTML -> PDF) &ndash;&gt;-->
    <dependency>
        <groupId>com.openhtmltopdf</groupId>
        <artifactId>openhtmltopdf-pdfbox</artifactId>
        <version>1.0.10</version>
    </dependency>
    <!-- optional: for supporting fonts -->
    <dependency>
        <groupId>com.openhtmltopdf</groupId>
        <artifactId>openhtmltopdf-java2d</artifactId>
        <version>1.0.10</version>
    </dependency>


com.openhtmltopdf.load INFO:: SAX XMLReader in use (parser): com.sun.org.apache.xerces.internal.jaxp.SAXParserImpl$JAXPSAXParser
com.openhtmltopdf.load INFO:: Loaded document in ~4ms
com.openhtmltopdf.load INFO:: TIME: parse stylesheets 0ms
com.openhtmltopdf.match INFO:: media = print
com.openhtmltopdf.match INFO:: Matcher created with 171 selectors
com.openhtmltopdf.general INFO:: Using fast-mode renderer. Prepare to fly.
org.springframework.mail.MailSendException: Mail server connection failed. Failed messages: org.eclipse.angus.mail.util.MailConnectException: Couldn't connect to host, port: smtp.example.com, 587; timeout -1;
  nested exception is:
	java.net.UnknownHostException: smtp.example.com; message exception details (1) are:
Failed message 1:
org.eclipse.angus.mail.util.MailConnectException: Couldn't connect to host, port: smtp.example.com, 587; timeout -1;
  nested exception is:
	java.net.UnknownHostException: smtp.example.com
	at org.eclipse.angus.mail.smtp.SMTPTransport.openServer(SMTPTransport.java:2243)
	at org.eclipse.angus.mail.smtp.SMTPTransport.protocolConnect(SMTPTransport.java:729)
	at jakarta.mail.Service.connect(Service.java:345)
	at org.springframework.mail.javamail.JavaMailSenderImpl.connectTransport(JavaMailSenderImpl.java:480)
	at org.springframework.mail.javamail.JavaMailSenderImpl.doSend(JavaMailSenderImpl.java:399)
	at org.springframework.mail.javamail.JavaMailSenderImpl.send(JavaMailSenderImpl.java:350)
	at org.springframework.mail.javamail.JavaMailSender.send(JavaMailSender.java:101)
	at com.finalproject.hrportal.service.impl.OfferLetterService.sendOfferEmail(OfferLetterService.java:63)
	at com.finalproject.hrportal.service.impl.OfferLetterService.generateAndSend(OfferLetterService.java:67)
	at com.finalproject.hrportal.controller.OfferLetterController.sendOffer(OfferLetterController.java:22)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:258)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:191)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:118)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:991)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:896)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1089)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:979)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1014)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:914)
	at jakarta.servlet.http.HttpServlet.service(HttpServlet.java:590)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:885)
	at jakarta.servlet.http.HttpServlet.service(HttpServlet.java:658)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:195)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:51)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:167)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:90)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:483)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:116)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:93)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:344)
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:398)
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:63)
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:903)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1776)
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:52)
	at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:975)
	at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:493)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:63)
	at java.base/java.lang.Thread.run(Thread.java:842)
Caused by: java.net.UnknownHostException: smtp.example.com
	at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:572)
	at java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)
	at java.base/java.net.Socket.connect(Socket.java:633)
	at java.base/java.net.Socket.connect(Socket.java:583)
	at org.eclipse.angus.mail.util.SocketFetcher.createSocket(SocketFetcher.java:368)
	at org.eclipse.angus.mail.util.SocketFetcher.getSocket(SocketFetcher.java:243)
	at org.eclipse.angus.mail.smtp.SMTPTransport.openServer(SMTPTransport.java:2193)
	... 59 more


<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8" />
    <title>Offer Letter</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; color: #222; }
        .header { text-align: center; border-bottom: 2px solid #e6e6e6; padding-bottom: 10px;
       margin-bottom: 20px; }
        .company { font-size: 20px; font-weight: 700; }
        .sub { font-size: 12px; color: #666; }
        .content { margin-top: 20px; line-height: 1.5; font-size: 14px; }
        .section { margin-top: 16px; }
        .signature { margin-top: 40px; }
        .details-table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        .details-table td { padding: 6px 8px; vertical-align: top; }
        .details-table .label { font-weight: 600; width: 200px; color: #333; }
        .footer { margin-top: 30px; font-size: 12px; color: #666; border-top: 1px dashed #ddd;
       padding-top: 12px; }
    </style>
</head>
<body>
<div class="header">
    <div class="company">Your Company Name Pvt. Ltd.</div>
    <div class="sub">Registered Office • Address line 1 • City</div>
</div>
<div class="content">
    <div>Date: <span th:text="${date}">01 Jan 2025</span></div>
    <div class="section">
        <strong>To,</strong><br/>
        <span th:text="${fullName}">Candidate Name</span><br/>
        <span th:text="${email}">candidate@example.com</span>
    </div>
    <div class="section">
        <p>Dear <span th:text="${fullName}">Candidate</span>,</p>
        <p>We are pleased to offer you the position of <strong th:text="${profileRole}">Software Engineer</strong> at <strong>Your Company Name</strong>. The details of your offer are below:
        </p>
        <table class="details-table">
            <tr>
                <td class="label">Position</td>
                <td th:text="${profileRole}">Software Engineer</td>
            </tr>
            <tr>
                <td class="label">Expected CTC</td>
                <td th:text="${expectedCtc}">600000</td>
            </tr>
            <tr>
                <td class="label">Total Experience</td>
                <td th:text="${totalExperience}">3</td>
            </tr>
            <tr>
                <td class="label">Notice Period</td>
                <td th:text="${noticePeriod}">60</td>
            </tr>
            <tr>
                <td class="label">Skills</td>
                <td th:text="${skills}">Java, Spring Boot</td>
            </tr>
            <tr>
                <td class="label">Remarks</td>
                <td th:text="${remarks}">-</td>
            </tr>
        </table>
        <div class="section">
            <p>Please note that this offer is subject to verification of your documents and a
                successful background check. If you accept this offer, please sign and return a scanned copy of
                this letter.</p>
        </div>
        <div class="signature">
            <p>Sincerely,</p>
            <p><strong>HR Team</strong><br/>Your Company Name</p>
        </div>
    </div>
    <div class="footer">
        This offer letter is computer generated and does not require a physical signature.
    </div>
</div>
</body>
</html>

package com.finalproject.hrportal.service.impl;


import com.finalproject.hrportal.dto.OfferLetterRequest;
import com.openhtmltopdf.pdfboxout.PdfRendererBuilder;
import jakarta.mail.internet.MimeMessage;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.ByteArrayResource;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;
import org.thymeleaf.context.Context;
import org.thymeleaf.spring6.SpringTemplateEngine;

import java.io.ByteArrayOutputStream;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

@Service
public class OfferLetterService {
    private final SpringTemplateEngine templateEngine;
    private final JavaMailSender mailSender;
    private final String fromAddress;
    public OfferLetterService(SpringTemplateEngine templateEngine,
                              JavaMailSender mailSender,
                              @Value("${app.mail.from}") String fromAddress) {
        this.templateEngine = templateEngine;
        this.mailSender = mailSender;
        this.fromAddress = fromAddress;
    }
    public byte[] generateOfferPdf(OfferLetterRequest req) throws Exception {
        Context ctx = new Context();
        ctx.setVariable("fullName", req.getFullName());
        ctx.setVariable("email", req.getEmail());
        ctx.setVariable("profileRole", req.getProfileRole());
        ctx.setVariable("expectedCtc", req.getExpectedCtc());
        ctx.setVariable("totalExperience", req.getTotalExperience());
        ctx.setVariable("noticePeriod", req.getNoticePeriod());
        ctx.setVariable("skills", req.getSkills());
        ctx.setVariable("remarks", req.getRemarks());
        ctx.setVariable("date", LocalDate.now().format(DateTimeFormatter.ofPattern("dd MMMM yyyy")));
                String html = templateEngine.process("offer-letter", ctx);
        try (ByteArrayOutputStream os = new ByteArrayOutputStream()) {
            PdfRendererBuilder builder = new PdfRendererBuilder();
            builder.withHtmlContent(html, null);
            builder.toStream(os);
            builder.run();
            return os.toByteArray();
        } catch (Exception e) {
            throw new Exception("Failed to generate PDF", e);
        }
    }
    public void sendOfferEmail(OfferLetterRequest req, byte[] pdfBytes) throws Exception {
        MimeMessage msg = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(msg, true, "UTF-8");
        helper.setFrom(fromAddress);
        helper.setTo(req.getEmail());
        helper.setSubject("Offer Letter - " + req.getProfileRole());
        String body = String.format("Dear %s,\n\nPlease find attached your offer letter.\n\nRegards,\nHR Team", req.getFullName());
                helper.setText(body, false);
        helper.addAttachment("OfferLetter_" + req.getFullName().replaceAll("\\s+","_") + ".pdf",
                new ByteArrayResource(pdfBytes));
        mailSender.send(msg);
    }
    public void generateAndSend(OfferLetterRequest req) throws Exception {
        byte[] pdf = generateOfferPdf(req);
        sendOfferEmail(req, pdf);
    }

}


package com.finalproject.hrportal.controller;

import com.finalproject.hrportal.dto.OfferLetterRequest;
import com.finalproject.hrportal.service.impl.OfferLetterService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/offers")
@RequiredArgsConstructor
public class OfferLetterController {

    private final OfferLetterService service;

    @PostMapping("/send")
    public ResponseEntity<?> sendOffer(@RequestBody OfferLetterRequest req) {
        try {
            service.generateAndSend(req);
            return ResponseEntity.ok().body("Offer letter sent to " + req.getEmail());
        } catch (Exception e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Failed to send offer letter: " +
                    e.getMessage());
        }
    }

}
