# Jsp

# include 和 jsp include

先看demo

test.jsp文件源代码:

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Test</title>
</head>
<body>
<%
    request.setAttribute("s1", "s1");
%>
<%@include file="m1.jsp" %>
<h1>这是个测试页面</h1>

<jsp:include page="m2.jsp"></jsp:include>
</body>
</html>

```


m1.jsp 文件源代码:
```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>m1</title>
</head>
<body>
<div>静态态导入m1文件-${requestScope['s1']}</div>
</body>
</html>
```

m2.jsp 文件源代码:

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>m2</title>
</head>
<body>
    <div>动态导入m2文件-${requestScope['s1']}</div>
</body>
</html>

```

<div style="color:red">下面查看源代码</div>

```java
 static {
    _jspx_dependants = new java.util.ArrayList<String>(1);
    _jspx_dependants.add("/WEB-INF/page/m1.jsp"); // 静态类中还有
  }

  public void _jspService(HttpServletRequest request, HttpServletResponse response)
        throws java.io.IOException, ServletException {

    PageContext pageContext = null;
    HttpSession session = null;
    ServletContext application = null;
    ServletConfig config = null;
    JspWriter out = null;
    Object page = this;
    JspWriter _jspx_out = null;
    PageContext _jspx_page_context = null;

    try {
      response.setContentType("text/html;charset=UTF-8");
      pageContext = _jspxFactory.getPageContext(this, request, response,
      			null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;
      _jspx_resourceInjector = (org.glassfish.jsp.api.ResourceInjector) application.getAttribute("com.sun.appserv.jsp.resource.injector");

      out.write("\n");
      out.write("\n");
      out.write("<html>\n");
      out.write("<head>\n");
      out.write("    <title>Test</title>\n");
      out.write("</head>\n");
      out.write("<body>\n");

    request.setAttribute("s1", "s1");

      out.write('\n');
      out.write("\n");
      out.write("\n");
      out.write("<html>\n");
      out.write("<head>\n");
      out.write("    <title>m1</title>\n");
      out.write("</head>\n");
      out.write("<body>\n");
      out.write("<div>静态态导入m1文件-");
      out.write((java.lang.String) org.apache.jasper.runtime.PageContextImpl.evaluateExpression("${requestScope['s1']}", java.lang.String.class, (PageContext)_jspx_page_context, null));
      out.write("</div>\n");
      out.write("</body>\n");
      out.write("</html>\n");
      out.write("\n");
      out.write("<h1>这是个测试页面</h1>\n");
      out.write("\n");
      org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "m2.jsp", out, false);
      out.write("\n");
      out.write("</body>\n");
      out.write("</html>\n");
    } catch (Throwable t) {
      if (!(t instanceof SkipPageException)){
        out = _jspx_out;
        if (out != null && out.getBufferSize() != 0)
          out.clearBuffer();
        if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
        else throw new ServletException(t);
      }
    } finally {
      _jspxFactory.releasePageContext(_jspx_page_context);
    }
  }

```




1. include 是一个指令， 而jsp include是一个标准动作
