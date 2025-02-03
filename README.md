import { baseUrl, IS_LOCAL, kycProxy, platformProxy, wait } from "./constants";
import {
    companyInfo,
    getFormById,
    mockBankerDetails,
    mockEmployeeDetails,
    mockGetCertifierDetails,
    mockGetCompanyOverview,
    mockGetExistingRequests,
    mockGetProducts,
    mockGetRequestDetails,
    mockSpendLimit
} from "./mockData";
import { getV2OnboardingFeatureFlag } from "_util/commonUtil";

const apiCall = async (path, mockData, forceMock?, method?, payload?, notData?) => {
    return new Promise(async (resolve, reject) => {
        if (IS_LOCAL || forceMock) {
            // below ternary is beacuse dashboard call doesn't return in {data: ...}
            // wait greater than 4999 cause issues for unit test
            notData
                ? resolve(wait(4000).then(() => mockData))
                : resolve(wait(2000).then(() => mockData.data));
        } else {
            const res = await fetch(`${baseUrl}${path}`, {
                method: method || "GET",
                credentials: "include",
                headers: new Headers({ "content-type": "application/json" }),
                redirect: "follow",
                body: payload !== undefined ? JSON.stringify(payload) : null
            })
                .then((response) => response.json())
                .catch((err) => {
                    console.log("Err:", err);
                    reject(res);
                });
            // below ternary is beacuse dashboard call doesn't return in {data: ...}
            notData ? resolve(res) : resolve(res.data);
        }
    });
};

export const bankerDetails = async (sid): Promise<any> => {
    const bankerDetailsResponse: any = await apiCall(
        `${kycProxy}/employee/search?searchData=${sid}`,
        mockEmployeeDetails,
        false
    );
    if (IS_LOCAL) {
        return mockBankerDetails.data.at(0);
    } else {
        return bankerDetailsResponse;
    }
};

export const cancelApplication = async (value: string, canceledReason: string): Promise<any> => {
    const cancelResponse: any = await apiCall(
        `${kycProxy}/cmlcard/${value}/cancel-application`,
        {},
        false,
        "POST",
        { canceledReason }
    );
    if (IS_LOCAL) {
        return {
            messsage: "Successfully canceled application with KYCid "
        };
    } else {
        return cancelResponse;
    }
};

export const companySearch = async (value): Promise<any> => {
    const response: any = await apiCall(
        `${kycProxy}/client/clientSearch?ecid=${value}`,
        companyInfo,
        false
    );
    if (IS_LOCAL) {
        return response.filter(
            (e) => e.ecidName.toLowerCase().includes(value.toLowerCase()) || e.ecidId === value
        );
    } else {
        return response;
    }
};

export const employeeSearch = async (value): Promise<any> => {
    const employeeSearchResponse: any = await apiCall(
        `${kycProxy}/employee/search?searchData=${value}`,
        mockEmployeeDetails,
        false
    );
    if (IS_LOCAL) {
        return employeeSearchResponse.filter(
            (e) =>
                e.firstName.toLowerCase().includes(value.toLowerCase()) || e.employeeSID === value
        );
    } else {
        return employeeSearchResponse;
    }
};

export const getCompanyOverview = async (ecid: string): Promise<any> => {
    const companyOverview: any = await apiCall(
        `${kycProxy}/client/clientDetail?ecid=${ecid}`,
        mockGetCompanyOverview(ecid)
    );
    return companyOverview.at(0);
};

export const getCreditLimit = async (ecid: string): Promise<any> => {
    const creditDetails: any = await apiCall(
        `${kycProxy}/cmlcard/${ecid}/eci-credit-details`,
        mockSpendLimit,
        false,
        undefined,
        undefined,
        true
    );
    return creditDetails;
};

export const getExistingRequests = async (ecid: string, payload: any): Promise<any> => {
    const getExistingRequestsResponse: any = await apiCall(
        `api/cdd/v1/internal/dashboard`,
        mockGetExistingRequests(ecid),
        false,
        "POST",
        payload,
        true
    );

    return getExistingRequestsResponse;
};

export const getProducts = async (ecid): Promise<any> => {
    const getProductsData: any = await apiCall(
        `${kycProxy}/client/product/warnings?ecid=${ecid}`,
        mockGetProducts,
        false
    );
    return getProductsData;
};

export const getForm = async (formId): Promise<any> => {
    return getFormById(formId).data;
};

export const sendApplication = async (payload): Promise<any> => {
    const apiUrl = getV2OnboardingFeatureFlag() ? `${kycProxy}/internal/onboarding` : `${platformProxy}/createProfile`;

    const sendApplicationResponse: any = await apiCall(
        apiUrl,
        mockGetCompanyOverview(payload),
        false,
        "POST",
        payload,
        true
    );

    return sendApplicationResponse;
};

export const getRequestDetails = async (kycId: string) => {
    const response = await apiCall(
        `${kycProxy}/cmlcard/${kycId}/details`,
        mockGetRequestDetails,
        false,
        "GET",
        undefined,
        true
    );
    return response;
};

export const getCertifierDetails = async (kycId: string) => {
    const response = await apiCall(
        `${kycProxy}/cmlcard/${kycId}/certifier-details`,
        mockGetCertifierDetails,
        false,
        "GET",
        undefined,
        true
    );
    return response;
};
