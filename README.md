<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
      lang="en">
<head>
    <script type="text/javascript"
            th:src="@{/webjars/jquery/3.4.1/jquery.min.js/}"></script>
    <link rel="stylesheet" th:href="@{../css/main.css}"/>
    <title>ANALYSIS</title>
</head>
<body>
<div class="container">
    FILTER SEARCH : <span th:utext="${filter}"/>
</div>
<div class="container">
    <h3>ANALYSIS REQUESTS RESULT SEARCH</h3>
    <table class="table-bordered">
        <thead class="tab-header-area">
        <tr>
            <th> num </th>
            <th> analysis name </th>
            <th> patient</th>
            <th> analysis date</th>
        </tr>
        </thead>
        <tbody>
        <tr th:each="result : ${results}" th:id="id+${result.num}">
            <td>
                <input type="text" th:name="num"
                       th:value="${result.num}" readonly>
            </td>
            <td>
                <input type="text" th:name="analysisName"
                       th:value="${result.analysisName}" readonly/>
            </td>
            <td>
                <input type="text" th:name="patientName"
                       th:value="${result.patientName}" readonly/>
            </td>
            <td>
                <input type="text" th:name="analysisDate"
                       th:value="${result.analysisDate}" readonly/>
            </td>
        </tr>
        </tbody>
    </table>
</div>
<div class="container" id="js-user-page">
    <a href="/user-page">user page</a>
</div>
</body>
</html>
