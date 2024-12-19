static private void mapProducts(List<Product> products, JobSchedulerEmailNotificationReportDTO report) {
        if (!CollectionUtils.isEmpty(products)) {
            products.forEach(pro -> {

                if (!Objects.nonNull(pro)) {
                    return;
                }

                String productType = pro.getType();

                if (CERTIFIER.equalsIgnoreCase(productType)) {
                    productType = SIGNER_CERTIFICATION;
                } else if (ONE_CARD_REWARD.equalsIgnoreCase(pro.getType())) {
                    if (Objects.nonNull(pro.getProductDetails())) {
                        Map<String, Object> map = pro.getProductDetails();
                        if (Objects.nonNull(map.get("settlementTerms"))) {
                            String settlementTerm = map.get("settlementTerms").toString();
                            if (StringUtils.isNotEmpty(settlementTerm)) {
                                report.setSettlementTerms(settlementTerm);
                                if (StringUtils.isNotEmpty(settlementTerm)) {
                                    report.setRequestType("30/1".equalsIgnoreCase(settlementTerm) ? DYNAMIC_CARD : ONE_CARD_WITH_REWARDS);
                                    return;
                                }
                            }
                        }
                    }
                } else if (COLTConstants.ACCELERATOR_CARD.equalsIgnoreCase(productType)) {
                    productType = COLTConstants.ACCELERATOR_CARD;
                } else if (COLTConstants.ONE_CARD_REBATES.equalsIgnoreCase(productType)) {
                    productType = COLTConstants.ONE_CARD_REBATES;
                }

                report.setRequestType(productType);
            });
        }
    }
