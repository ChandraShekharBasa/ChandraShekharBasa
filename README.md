UPDATED LATEST FROM CHATGPT:
[=============================

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.slf4j.Logger;

import java.util.Collections;
import java.util.List;

@ExtendWith(MockitoExtension.class)
class YourServiceTest {

    @InjectMocks
    private YourService yourService;

    @Mock
    private CMLCardProductService cmlCardProductService;

    @Mock
    private KYCInformationUtilityService kycInformationUtilityService;

    @Mock
    private DocManagementService docManagementService;

    @Mock
    private OrchestrationService orchestrationService;

    @Mock
    private EmailService emailService;

    @Mock
    private Logger appLogger;

    @Mock
    private KYCInformationEntity kycInformationEntity;

    @Mock
    private DocumentMetaDataEntity documentMetaDataEntity;

    @Mock
    private Product product;

    private final Long kycId = 123L;

    @BeforeEach
    void setUp() {
        // Basic setup for mocks, assuming accelerator card type
        when(product.getType()).thenReturn(COLTConstants.ACCELERATOR_CARD);
        when(cmlCardProductService.getProductsForKYC(kycId)).thenReturn(Collections.singletonList(product));
        when(kycInformationUtilityService.getKycInformationEntity(kycId)).thenReturn(kycInformationEntity);
        when(docManagementService.getCCADocForAcceleratorCard(kycId)).thenReturn(documentMetaDataEntity);
    }

    @Test
    void testSubmitContract_SuccessWithinInitiateCardProSubmission() {
        // Arrange
        when(orchestrationService.submitContract(any(ContractBaseRequest.class))).thenReturn(CommonConstants.CONTRACT_API_SUBMIT_SUCCESS);

        // Act
        yourService.initiateCardProSubmission(kycId);

        // Assert
        verify(orchestrationService).submitContract(any(ContractBaseRequest.class));
        verify(kycInformationUtilityService).saveKycInformation(kycInformationEntity);
        verify(appLogger).info("Updating KycInformation with contract submission status: {} for kycId {}",
                CommonConstants.CONTRACT_API_SUBMIT_SUCCESS, kycId);
        assertEquals(CommonConstants.CONTRACT_API_SUBMIT_SUCCESS, kycInformationEntity.getContractSubmitStatus());
    }

    @Test
    void testSubmitContract_FailureWithinInitiateCardProSubmission() {
        // Arrange
        when(orchestrationService.submitContract(any(ContractBaseRequest.class))).thenThrow(new DownstreamServicesException());

        // Act
        assertThrows(RuntimeException.class, () -> yourService.initiateCardProSubmission(kycId));

        // Assert
        verify(emailService).sendEmailForServiceFailure(kycId, CommonConstants.CONTRACT_API);
        verify(kycInformationUtilityService).saveKycInformation(kycInformationEntity);
        verify(appLogger).info("Updating KycInformation with contract submission status: {} for kycId {}",
                CommonConstants.CONTRACT_API_SUBMIT_FAILED, kycId);
    }

    @Test
    void testSubmitContract_ProductsListIsNullWithinInitiateCardProSubmission() {
        // Arrange
        when(cmlCardProductService.getProductsForKYC(kycId)).thenReturn(null);

        // Act
        yourService.initiateCardProSubmission(kycId);

        // Assert
        verify(orchestrationService, never()).submitContract(any(ContractBaseRequest.class));
        verify(emailService, never()).sendEmailForServiceFailure(anyLong(), anyString());
    }
}

