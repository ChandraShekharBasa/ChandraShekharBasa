java.lang.NullPointerException: Cannot invoke "java.lang.Boolean.booleanValue()" because "isExpenseToolUsed" is null

Boolean isExpenseToolUsed = Objects.isNull(expenseReportingToolInfo.getIsThirdPartyExpTool()) ? false : expenseReportingToolInfo.getIsThirdPartyExpTool();
