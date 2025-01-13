 private Product getKYCProduct(Long kycId, String requestType) {
        List<Product> products = cmlCardProductService.getProductsByKycAndType(kycId, requestType);
        return products.stream()
                .filter(product -> COLTConstants.ACCELERATOR_CARD.equalsIgnoreCase(product.getType()))
                .findAny()
                .orElse(null);
    }
