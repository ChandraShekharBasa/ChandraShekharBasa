EmployeeSearch employeeSearchReq = new EmployeeSearch();
            employeeSearchReq.setQuery(new Query());
            employeeSearchReq.getQuery().setImpersonate("V841274");
            employeeSearchReq.getQuery().setBulk(new Bulk());
            employeeSearchReq.getQuery().getBulk().setField(CommonConstants.ALL);
            employeeSearchReq.getQuery().getBulk().setValues(searchQuery);

            log.info("Employee Search request: {}", employeeSearchReq);

            HttpEntity<EmployeeSearch> httpEntity = new HttpEntity<>(employeeSearchReq, httpHeaders);

            empSearchResponse = empSearchRestTemplate.exchange(url, HttpMethod.POST, httpEntity,
                    EmployeeSearchResponse.class);





Employee Search request: EmployeeSearch(query=Query(impersonate=null, criteria=null, bulk=Bulk(field=_all, values=[basa])))
{
  "query": {
    "impersonate": null,
    "criteria": null,
    "bulk": {
      "field": "_all",
      "values": ["basa"]
    }
  }
}
