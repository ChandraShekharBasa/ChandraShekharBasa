private void submitContract(Long kycId) {
        String response = StringUtils.EMPTY;
        try {
            appLogger.info("Initiating orchestration service for submitting contract for kycId: {}", kycId);
            List<Product> products = cmlCardProductService.getProductsForKYC(kycId);
            if (isContractAPIEnabled && Objects.nonNull(products) && COLTConstants.ACCELERATOR_CARD.equalsIgnoreCase(products.get(0).getType())) {
                KYCInformationEntity kycInformationEntity = kycInformationUtilityService.getKycInformationEntity(kycId);
                DocumentMetaDataEntity documentMetaDataEntity = docManagementService.getCCADocForAcceleratorCard(kycId);
                ContractBaseRequest contractBaseRequest = CmlCardMapper.mapContractTemplateDTO(kycInformationEntity, products, documentMetaDataEntity);
                response = orchestrationService.submitContract(contractBaseRequest);
                appLogger.info("response from orchestration service for submitting contract for kycId: {} -- {}", kycId, response);

                // Store contract submission success status in DB
                if (Objects.nonNull(response) && response.equals(CommonConstants.CONTRACT_API_SUBMIT_SUCCESS)) {
                    appLogger.info("Updating KycInformation with contract submission status: {} for kycId {}",
                            CommonConstants.CONTRACT_API_SUBMIT_SUCCESS, kycId);
                    kycInformationEntity.setContractSubmitStatus(response);
                    kycInformationUtilityService.saveKycInformation(kycInformationEntity);
                }
            }
        } catch (Exception ex) {
            emailService.sendEmailForServiceFailure(kycId,CommonConstants.CONTRACT_API);

            // Store contract submission failure status in DB
            if (response.isEmpty()) {
                KYCInformationEntity kycInformationEntity = kycInformationUtilityService.getKycInformationEntity(kycId);
                appLogger.info("Updating KycInformation with contract submission status: {} for kycId {}",
                        CommonConstants.CONTRACT_API_SUBMIT_FAILED, kycId);
                kycInformationEntity.setContractSubmitStatus(response);
                kycInformationUtilityService.saveKycInformation(kycInformationEntity);
            }

            throw new DownstreamServicesException(UIErrorCode.UNEXPECTED_ERROR,
                    "An exception occurred when sending request to Orchestration Service for Contract Submission", MDC.get(CommonConstants.TRACEID), ex);
        }
    }

