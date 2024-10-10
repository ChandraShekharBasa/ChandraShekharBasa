@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class EmployeeSearch {
    private Query query;
}

EmployeeSearch employeeSearchReq = new EmployeeSearch();
employeeSearchReq.getQuery().getBulk().setField(CommonConstants.ALL);
employeeSearchReq.getQuery().getBulk().setValues(searchQuery);

java.lang.NullPointerException: Cannot invoke "net.jpmchase.common.services.domain.request.employeesearch.Query.getBulk()" because the return value of "net.jpmchase.common.services.domain.request.employeesearch.EmployeeSearch.getQuery()" is null
	at net.jpmchase.common.services.service.impl.EmployeeSearchServiceImpl.getEmployeeDetails(EmployeeSearchServiceImpl.java:45) ~[classes/:na]
	at net.jpmchase.common.services.controller.EmployeeSearchController.getEmployeeDetails(EmployeeSearchController.java:29) ~[classes/:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:568) ~[na:na]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205) ~[spring-web-6.0.13.jar:6.0.13]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:150) ~[spring-web-6.0.13.jar:6.0.13]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:118) ~[spring-webmvc-6.0.13.jar:6.0.13]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:884) ~[spring-webmvc-6.0.13.jar:6.0.13]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:797) ~[spring-webmvc-6.0.13.jar:6.0.13]


