public void initiateCardProSubmission(Long kycId) {

        appLogger.info("Orchestration Async process for certifier initiated with kycId: {}", kycId);
        Gson gson = new GsonBuilder().serializeSpecialFloatingPointValues().serializeNulls().create();

        KYCInformationDTO kycInformation = kycInformationUtilityService.getKycInformation(kycId);
        appLogger.info("KycInformation within new async for certifier thread with kycId: {} and kycInformation: {}", kycId, gson.toJson(kycInformation));
        String ecid = kycInformation.getEcid();
        appLogger.info("ECID {} With kycId for certifier: {}", ecid, kycId);

        try {
            CmlCardBaseDTO cmlCardBaseDTO = generatePayloadForPDFOrCardProSubmission(kycId, kycInformation, null, null, true);

            List<KYCClientUploadedDocumentDTO> kycClientUploadedDocumentList = kycClientUploadedDocumentService.getKYCClientUploadedDocumentsDto(kycId);

            if (!CollectionUtils.isEmpty(kycClientUploadedDocumentList)) {
                List<KYCClientUploadedDocumentDTO> filteredKycClientUploadedDocumentList  = kycClientUploadedDocumentList.stream()
                        .filter(doc -> (doc.getDocumentType().equalsIgnoreCase(COLTConstants.INTERNAL_DOCTYPE)) ||
                                doc.getDocumentType().equalsIgnoreCase(COLTConstants.ACCELERATOR_CARD_CCA_DOCUMENT_TYPE))
                        .collect(Collectors.toList());

                if (CollectionUtils.isEmpty(cmlCardBaseDTO.getDocumentId())) {
                    cmlCardBaseDTO.setDocumentId(filteredKycClientUploadedDocumentList.stream().map(KYCClientUploadedDocumentDTO::getDocumentId).collect(Collectors.toList()));
                }
                CmlCardMapper.mapClientUploadedDocumentList(cmlCardBaseDTO, filteredKycClientUploadedDocumentList);

                appLogger.info("Calling submitToCardPro for certifier with kycId: {}", kycId);
                submitToCardPro(kycId, cmlCardBaseDTO, gson);
                submitContract(kycId);
            }

        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }
