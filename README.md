export const saveBusinessOperations = async (onboardingId, values) => {
    delete values?.accountToolName;
    delete values?.accountOtherVendor;
    const request = {
        ...values,
        kycId: onboardingId,
        expsToolName: values?.expsToolName === 'Other' ? values?.otherVendor : values?.expsToolName,
        totalAnnualRevenueAmount: values?.totalAnnualRevenueAmount
            ? Number(values?.totalAnnualRevenueAmount)
            : null,
    };
    delete request?.otherVendor;

    return await apiCall('post', `/kyc/${onboardingId}/bus-ops`, request);
};
