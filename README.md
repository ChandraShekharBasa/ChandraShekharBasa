// Initialize the Query object if it's null
if (employeeSearchReq.getQuery() == null) {
    employeeSearchReq.setQuery(new Query());
}

// Initialize the Bulk object if it's null
if (employeeSearchReq.getQuery().getBulk() == null) {
    employeeSearchReq.getQuery().setBulk(new Bulk());
}
