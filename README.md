public CmlCardPaymentDTO getRebateAccountInfo(String kycId) {
        try {
            CmlCardPaymentDTO rebateAccountDTO;
            KYCPaymentInfoEntity dbKYCPaymentInfoEntity =
                    kycPaymentInfoRepository.findRebateDetailsForKycId(Long.parseLong(kycId));
            if (Objects.nonNull(dbKYCPaymentInfoEntity)) {
                rebateAccountDTO =
                        CmlCardPaymentInfoEntityToDTOMapper.mapKYCPaymentInfoEntityToCmlCardPaymentDTO(
                                dbKYCPaymentInfoEntity);
            } else {
                log.error("CmlCardPaymentInfoServiceImpl: payment info entity is empty with kycId: {}",
                        kycId);
                //todo: refactor
                return null;
            }
            log.info("CML Card services getting payment info with kycId: {}", kycId);
            return rebateAccountDTO;
        } catch (Exception e) {
            MDC.put(CommonConstants.ERROR_CODE, LoggingErrorCodes.DATABASE_ERROR);
            log.error("An exception occurred trying to get payment info with "
                            + "kycId: {}",
                    kycId, e);
            throw new OnboardingDatabaseException(UIErrorCode.DB_ERROR,
                    "An exception occurred trying to get payment info "
                            + "with kycId: "
                            + kycId, MDC.get(CommonConstants.TRACEID));
        }
    }


Not covered by tests sonar

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
